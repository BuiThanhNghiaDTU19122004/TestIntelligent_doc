# TI — LUỒNG HỆ THỐNG ĐI TỪ ĐẦU ĐẾN CUỐI (giải thích cho người đọc sơ đồ v2.1)

> **Mục đích:** giúp bạn đọc hiểu **một tài vụ kiểm thử đi qua hệ thống Testing Intelligence (TI) như thế nào**, ai quyết gì, dữ liệu nào băng qua biên nào, và **chỗ nào trong hình còn là đề xuất chứ chưa phải sự thật**.
> **Nguồn hình:** `diagram/TI_System_Architecture_v2.1_CANDIDATE.drawio` + `.drawio.png` (đã đọc trực tiếp XML, không chỉ ảnh) · `diagram/images/TI_Master_Architecture.drawio` (5 tab) · `diagram/Hung/Workflow_Hung.pdf`.
> **Nguồn luật/nghiệp vụ:** `Research/Task_1…Task_4`, `Research/BAO_CAO_DOI_SOAT_MISMATCH_VA_DONG_BO_TI.md`, `Research/GLOSSARY_TI.md`, Biên bản họp TI 23/09/2026.
> **Trạng thái:** tài liệu **đọc-hiểu** (bản đồ chữ), **không** phải bằng chứng nghiệm thu. Mọi con số/sự thật phải tra ngược về nguồn có nhãn `OBSERVED`/`INFERRED`/`CANDIDATE`.
> **Ngày soạn:** 28/09/2026.

## 0. QUY ƯỚC NHÃN SỰ THẬT (đọc trước khi tin bất cứ điều gì trong hình)

| Nhãn | Nghĩa | Ví dụ trong hồ sơ TI |
| :--- | :--- | :--- |
| `OBSERVED` | Đã **đo được** trên bản đang chạy, tại một ngày cụ thể | Hiện trạng 22/09: `API :8000` + `Portal :8001` **trên cùng 1 EC2**; evidence ghi xuống EBS |
| `INFERRED` | Suy ra từ mã/cấu hình, **chưa đo** | Chi phí token tiết kiệm ~65–75% nhờ model tiering |
| `CANDIDATE` | **Đề xuất**, chưa chạy | **Toàn bộ** v2.1: Fargate task-per-job, 6 trục runner, Direct-to-S3, Gate S09 tự động |
| `UNVERIFIED` | Có nhắc tới nhưng **không tra được bằng chứng** | Memory reuse (S10) — Task 3 ghi *"ACTIVE — chưa nghiệm thu reuse"* |
| Nét liền / nét đứt | Liền = đã đo; đứt = khai báo/chưa đo | Theo `GLOSSARY_TI.md` §A — **legend v2.1 hiện định nghĩa khác, cần sửa** (xem §6, mục số 1) |

> Ba ranh giới **không được xóa**: `CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · `completed ≠ PASS (Law 18)`.

---

## 1. HIỂU TRONG 60 GIÂY

TI là một **dây chuyền thẩm định artifact (PR/changeset)**, chia làm **4 vùng** và **3 vai**:

```
VÙNG 1 — BÊN GỌI (CI/CD · Dev/QA · TI CLI)
   │  [01] POST /v2/artifact-jobs  (changeset + SHA-256)
   ▼
VÙNG 2 — ACCOUNT A (ap-southeast-1) = "BỘ NÃO HÀNH CHÍNH"
   CloudFront → WAF → TI API v2 (:8000)  →  Job Controller (ECS/Fargate, Law 4.3)
   Job Controller giữ: State Machine · TenantBinding Resolver · Lease+Heartbeat · Port Launcher
   Kho: RDS PostgreSQL (Job Store)  ·  S3 + Object Lock (Evidence, WORM 90 ngày)
   Thêm: S09 Decision Gate · Web Portal :8001 · Secrets Manager (tenant secrets + STS)
   │  [08A] Cross-Account IAM (không mang secret)
   ▼
VÙNG 3 — ACCOUNT B (us-east-1) = "BỘ NÃO AI"
   AgentCore Harness → Bedrock (Sonnet 5 mặc định · Opus 5 khi Risk=CRITICAL)
     → Bedrock Evaluations (chấm Groundedness/Faithfulness) → trả ToolIntent JSON
   Knowledge Base S10 Memory (GOLDEN only — chưa nghiệm thu)
   │  [10] Smart Dispatch: ImpactSet ∩ TargetBinding  →  ECS RunTask
   ▼
VÙNG 4 — SANDBOX VPC (ap-southeast-1, Domain D2)
   Private subnet KHÔNG IGW/NAT · SG DENY ALL EGRESS · VPC Endpoints (S3 Gateway $0, ECR/Logs/STS ~$22/tháng)
   Trục 1 D5a Security (Semgrep+Trivy+Gitleaks) · Trục 2 API Fuzzing · Trục 3 UI/A11y
   Trục 4 DB Dual Isolation · Trục 5 Performance (k6 + 3 khóa) · Trục 6 D5b DAST (W3)
   Runners đẩy raw evidence THẲNG lên S3, chỉ trả envelope ~2KB về Job Controller
```

**3 câu chốt để nhớ:**
1. **AI chỉ tư duy viết kế hoạch/test candidate** (Account B) — **AI không được chạy gì và không được giữ secret**; nó chỉ phát `ToolIntent JSON` toàn tham số biểu tượng.
2. **Job Controller là "workflow authority"** (Account A): tra `TenantBinding` → nội suy URL thật → cấp STS ngắn hạn → **mới** dispatch ECS RunTask. Quyết định chạy phần nào = `ImpactSet (S03) ∩ TargetBinding.EnabledDomains (S01)`.
3. **Bằng chứng là bất biến**: raw đi thẳng S3 + Object Lock + băm SHA-256; job "chạy xong" ≠ "đạt" — `S09 Gate` trả `PASS / HOLD / DO_NOT_PASS`, và HOLD chỉ được chuyển PASS bằng **Waiver có chữ ký người có thẩm quyền**.

---

## 2. AI LÀM GÌ — AI **KHÔNG ĐƯỢC** LÀM GÌ (bảng quyền hạn)

| Thành phần | Vùng | Nhiệm vụ chính | **Bị cấm / không giữ** | Bằng chứng vị trí trong v2.1 |
| :--- | :--- | :--- | :--- | :--- |
| **CI/CD · Dev/QA · TI CLI** | Vùng 1 | Gửi changeset + SHA-256; poll kết quả; duyệt Waiver (QA Lead) | Không tự chạy test, không tự đọc raw evidence trong sandbox | `zone_ext` L13; `lbl_01` L451 |
| **CloudFront + WAF** | A | TLS termination, chặn OWASP/rate-limit, định tuyến `/v1` `/v2` | WAF **không** xác thực chữ ký HMAC (việc này thuộc API/Lambda@Edge) | `cloudfront` L31, `waf` L34 |
| **TI API v2 (FastAPI :8000)** | A | Validate schema, trả `202 Accepted + job_id` < 1s (không block CI) | Không giữ ngữ cảnh tenant, không dispatch runner | `ti_api` L85 |
| **Job Controller** (State Machine · TenantBinding Resolver · Lease+Heartbeat Coordinator · Port Launcher) | A | Sổ cái duy nhất (Law 4.3); allowlist `ToolIntent`; nội suy URL thật; cấp STS; dispatch ngang hàng; giữ lease/recovery; băm SHA-256; gọi Gate | Không chạy mã test; **không bao giờ nhận secret từ prompt AI** | `jc_box` L40, `jc_resolver` L46, `jc_lease` L49, `port_launcher` L173 |
| **Cross-Account IAM (AssumeRole)** | A→B | Chuyển *task context + token budget* sang Account B | Không chuyển credential tenant | `e_jc_ac` L429 `[08A]` |
| **AgentCore Harness + Bedrock** | B | (S03–S06) Impact/Risk → Planning → sinh candidate; gọi Evaluations; phát `ToolIntent JSON` | Không phải *system of record*; không chạy tool; không có URL/secret | `agentcore`, `sonnet` L138, `opus` L141 |
| **Bedrock Evaluations** | B | Chấm `Groundedness`/`Faithfulness`; đạt thì lưu candidate lên S3; trả "Quality OK" | Không được tự tuyên bố PASS (Law 7) | `bed_eval`, `e_bedrock_eval` L155 `[09A]`, `e_eval_s3` L334 `[09B]` |
| **Sandbox 6 trục (ECS Fargate task-per-job)** | 4 | Chạy test trong MicroVM cô lập, **NO INTERNET + VPC Endpoints**; đẩy raw lên S3; trả envelope ~2KB; heartbeat giữ lease | Không ra Internet; không nhận URL chưa binding; không chạm DB gốc | `sandbox` L164; `r1_box`…`r6_box` L181–L252 |
| **S3 + Object Lock / RDS PostgreSQL** | A | Evidence WORM 90 ngày + băm SHA-256; Job Store (state, lease, quota) | RDS **không** phải nơi chứa raw evidence | `s3_evi`, `rds_pg` |
| **S09 Decision Gate** | A | Mã cứng đối soát luật → `PASS / HOLD / DO_NOT_PASS` | **Không** cho AI override; `completed ≠ PASS` | `s09_gate` L61, `[12A]` L73, `[12B]` L76 |
| **QA Lead / Authority** | 1 | Xem evidence, ký **Waiver** để chuyển `HOLD → PASS` trở thành *release decision* | Không được override `DO_NOT_PASS` (Critical/secret lộ/hash mismatch) | ⚠️ **v2.1 đang thiếu nhánh này** — xem §6.3 |

---

## 3. LUỒNG CHI TIẾT `[01] → [12D]` (đọc kèm hình)

### 3.1. Giai đoạn 1–2: nhận việc & phân tích (`[01]`–`[07B]`)

| Bước | Ai → Ai | Dữ liệu qua biên | Ghi chú kỹ thuật / ranh giới | Nhãn |
| :--: | :--- | :--- | :--- | :--: |
| `[01]` | CI/CD (hoặc Dev/QA, TI CLI) → CloudFront | `POST /v2/artifact-jobs` + **changeset (git diff, OpenAPI, SQL migration, UI bundle) + SHA-256 digest** | Endpoint `/v2` là đường mới; `/v1/testing/changes` còn sống nhưng **FROZEN** | CANDIDATE (đo được: route tồn tại) |
| `[02]` | CloudFront → WAF → TI API v2 (:8000) | HTTPS request đã lọc OWASP/rate-limit | ⚠️ Không được viết "WAF verify HMAC" | CANDIDATE |
| `[03]` | TI API v2 → Caller | `202 Accepted {job_id}` trong **< 1s** → CI/CD được giải phóng, không bị block | Đây là điểm "async admission" | CANDIDATE |
| `[04]` | TI API v2 → RDS PostgreSQL (Job Store) | Bản ghi job trạng thái khởi tạo + quota/timeout | ⚠️ v2.1 ghi `PENDING`, portal/Hùng dùng `QUEUED`/`queued` → **phải chốt 1 từ** (Ch. C11) | CANDIDATE |
| `[05]` | TI API v2 → Job Controller | Bàn giao `job_id` + context | JC = `ECS/Fargate State Machine (Law 4.3)` | CANDIDATE |
| `[06]` | Job Controller → **Trục 1 (D5a)** | Chạy **pre-scan**: Semgrep (SAST diff 2–5s) · Trivy (CVE) · Gitleaks (secrets) | v2.1 tự ghi chú: *"Trục 1 chạy SỚM ở [06] (pre-scan S02→S04); [10] chỉ khởi tạo Trục 2–6"*. ⚠️ Còn xung đột Task 4 §3.1.1 (Semgrep+Trivy = S07 runner) → xem §6.4 | CANDIDATE |
| `[07A]` | Trục 1 → S3 (Object Lock) | Báo cáo **chuẩn SARIF** (CWE/CVE/secrets) | Đẩy thẳng S3 qua **Gateway Endpoint $0** — raw **không** quay về backend | CANDIDATE |
| `[07B]` | Trục 1 → Job Controller | `ImpactSet` (S03) + `Risk Tier` (S04) | ⚠️ Chính chỗ này đang tranh chấp: **Account A hay Account B** mới là nơi tính S03/S04 (xem §6.2) | CANDIDATE |

### 3.2. Giai đoạn 2–3: não AI & điều phối (`[08A]`–`[10]`)

| Bước | Ai → Ai | Dữ liệu qua biên | Ghi chú kỹ thuật / ranh giới | Nhãn |
| :--: | :--- | :--- | :--- | :--: |
| `[08A]` | Job Controller → AgentCore Harness (Account B) | **Cross-Account IAM AssumeRole**: task context + **Token Budget** | Không mang credential tenant; Harness **không** phải system of record | CANDIDATE |
| `[08B]` | Harness → Bedrock | `Sonnet 5` mặc định (S05 plan + S06 sinh candidate); `Opus 5` **chỉ khi** `Risk Tier == CRITICAL` (threat modeling sâu) | ⚠️ Hiện trạng theo Task 3: runtime **đang pin cứng Opus 5, chưa có tiering** → phần này là đề xuất | CANDIDATE |
| `[09A]` | Sonnet → Bedrock Evaluations | TestPlan + danh sách candidate | Đây là **chốt chặn chống ảo giác** | CANDIDATE |
| `[09B]` | Evaluations → S3 | Candidate đạt chuẩn được lưu (điều kiện: `Groundedness ≥ 0.80`, `Faithfulness ≥ 0.85`, `temperature 0.0`) | ⚠️ v2.1 **chưa** nêu `±0.03`, chưa nêu 6 chỉ số, chưa nêu nhánh khi **không** đạt | CANDIDATE |
| `[09C]` | Evaluations → Job Controller | Tín hiệu "Quality OK" | Nếu **không** OK thì sao (retry? HOLD?) → hiện chưa vẽ | CANDIDATE |
| `[10]` | Job Controller (TenantBinding Resolver) → Port Launcher → **ECS RunTask** | **`Target Runners = ImpactSet ∩ TargetBinding.EnabledDomains`** → tạo **Trục 2…6** | Đây là câu trả lời cho điểm nóng Biên bản §3: *"PR chỉ sửa API thì KHÔNG chạy cả 6 trục"*. Trục 6 (D5b DAST) chỉ chạy khi **W3 + có Staging URL sống** (nét đứt) | CANDIDATE |

### 3.3. Giai đoạn 3–4: chạy sandbox, đóng bằng chứng, phán quyết (`[11A]`–Waiver)

| Bước | Ai → Ai | Dữ liệu qua biên | Ghi chú kỹ thuật / ranh giới | Nhãn |
| :--: | :--- | :--- | :--- | :--: |
| **Sandbox chạy** | Mỗi runner (Fargate task riêng) | Trục 1: Semgrep/Trivy/Gitleaks → SARIF · Trục 2: Schemathesis (fuzzing OpenAPI → HTTP 500) + Playwright API (HAR/trace) · Trục 3: Playwright headless (PNG/MP4/trace) + axe-core WCAG 2.1 AA · Trục 4: Aurora v2 Clone (<60s, Flyway) + DynamoDB Local (TTL 1h + DeleteTable) · Trục 5: AWS DLT + k6 (Internal ALB, trần 500 VUs, `abortOnFail` nếu 5xx > 2%) · Trục 6: ZAP + nuclei | Mạng: `Private Subnet không IGW/NAT + SG DENY ALL EGRESS + VPC Endpoints`. Mỗi job = 1 task, **tự hủy**, trả tiền theo giây | CANDIDATE |
| **Heartbeat** | Runner → Job Controller | `Heartbeat` định kỳ gia hạn **Worker Lease TTL** | Hồ sơ đang ghi **2 mức khác nhau** (60s ở Sequence vs "TTL 5 phút" ở Hùng/Blueprint) ⇒ cần chốt (ghi rõ "TTL 5 phút, heartbeat 60s") | CANDIDATE |
| `[11A]` | Runner → S3 (Object Lock) | Raw artifacts nặng: log k6, video MP4, PNG, Playwright trace, SARIF, DAST report… | **Direct-to-S3** qua S3 Gateway Endpoint (**$0**). Dữ liệu nặng **không bao giờ** đi qua Job Controller ⇒ chống ngộp ổ EBS/EC2 backend | CANDIDATE |
| `[11B]` | Runner → Job Controller | **Envelope ~2KB**: `exit_code`, S3 URI, `sha256Digest` | Nhẹ nên không gây nghẽn I/O backend. ⚠️ v2.1 hiện **chỉ vẽ envelope từ Trục 2** — phải vẽ cho **mọi** runner | CANDIDATE |
| `[12A]` | Job Controller → S09 Gate | Bảng metrics tổng hợp + SHA-256 | Gate = **mã cứng** (deterministic), không AI | CANDIDATE |
| `[12B]` | S09 Gate → RDS | Cập nhật **state** và **gate result**: `PASS` / `HOLD` / `DO_NOT_PASS` | Phải **tách** `state` (queued/running/completed/failed) khỏi `gate_result`; ghi rõ `completed ≠ PASS` (Law 18) | CANDIDATE |
| `[12C]` | S09 Gate → Knowledge Base S10 Memory (Account B) | Chỉ đẩy **tri thức GOLDEN** (testcase đã thẩm định) làm cơ sở cho lượt sau | ⚠️ Phải gắn nhãn `UNVERIFIED — chưa nghiệm thu reuse` (Task 3 §1.1 + portal) | CANDIDATE / UNVERIFIED |
| `[12D]` | Caller → **TI API v2** → RDS | `GET /v2/artifact-jobs/{id}` → `200 OK {state, gate}` | ⚠️ v2.1 vẽ **RDS → CI/CD thẳng** (bỏ qua API) ⇒ phải sửa thành `CI → API v2 → RDS` | CANDIDATE |
| **Live report** | Dev/QA → Web Portal :8001 | Xem báo cáo trực quan, video tái hiện lỗi, tải gói bằng chứng đã khóa | Portal là nơi *xem*, không phải nơi *quyết* | CANDIDATE |
| **Waiver (người)** | QA Lead/Authority → `POST /v2/operations/{id}/actions` | **Approved Waiver** → chuyển `HOLD` thành *release decision* `PASS` | Nhánh này có trong `images/TI_Sequence_Lifecycle.drawio` + `Blueprint §3.3` + `Workflow_Hung.pdf` — **nhưng thiếu trong v2.1 và Detailed** ⇒ phải bổ sung | CANDIDATE |

### 3.4. Ba cửa Gate phải nhớ (chuẩn Task 4 §6 — v2.1 hiện **chưa vẽ đủ**)

| Kết quả | Điều kiện | Có được override? |
| :--- | :--- | :--- |
| **`DO_NOT_PASS`** (hard stop) | `CRITICAL_COUNT > 0` **OR** `SECRETS_LEAKED > 0` · P95 vượt ngưỡng · migration rollback fail · **SHA-256 hash mismatch** | ❌ **Không** — không waiver, không exception |
| **`HOLD`** | `HIGH_COUNT > 0` · P99 vượt nhưng P95 OK · `Groundedness < 0.80` cho ≥ 30% candidate · `Faithfulness < 0.85` · runner timeout không có lý do SKIP | ✅ Chỉ khi có **Waiver Document ký bởi Architecture Authority** |
| **`PASS`** | Deterministic assertions OK · `HIGH_COUNT = 0` (hoặc có Waiver) · P95 ≤ ngưỡng Evaluation Pack · 0 console error · 0 vi phạm WCAG 2.1 AA · `Groundedness ≥ 0.80` mọi candidate | Đây vẫn chỉ là **GATE_RECOMMENDATION** — không phải "xác nhận tuyệt đối" |

---

## 4. SƠ ĐỒ CHỮ 1 TRANG (tiện copy vào chat/slide)

```
[CI/CD · Dev/QA · TI CLI]
   │[01] POST /v2/artifact-jobs (changeset + SHA-256)
   ▼
CloudFront → WAF → TI API v2 (:8000) ──[03] 202 Accepted <1s──▶ Caller
   │[04] state khởi tạo              │[05] enqueue
   ▼                                 ▼
RDS PostgreSQL ◀───[12B]─── S09 Decision Gate ◀──[12A]── Job Controller (Law 4.3)
(Job Store)                  │[12C]                        ├ State Machine
                             ▼                             ├ TenantBinding Resolver
              Knowledge Base S10 Memory (GOLDEN only,     ├ Lease + Heartbeat
              UNVERIFIED) [Account B]                      └ Port Launcher
                                                                    │
   ┌────────────────────────────────────────────────────────────────┤
   │[06] pre-scan Trục 1                                    [08A] Cross-Account IAM
   ▼                                                                    ▼
Trục 1 D5a (Semgrep+Trivy+Gitleaks, W1)                     AgentCore Harness [Account B]
   │[07A] SARIF → S3 ───────────┐                                 │[08B] Sonnet 5 (Opus 5 nếu CRITICAL)
   │[07B] ImpactSet+RiskTier ───┼──▶ Job Controller              ▼
   └───────────────────────────┼──▶                    Bedrock Evaluations (Groundedness/Faithfulness)
                               │                                │[09A] plan + cases
                               │                                │[09B] Eval OK → S3 (candidate đạt)
                               │                                └[09C] Quality OK → Job Controller
   [10] Smart Dispatch = ImpactSet ∩ TargetBinding → ECS RunTask (task-per-job)
   ▼
SANDBOX VPC (Private subnet · SG DENY ALL · VPC Endpoints)  [NO INTERNET]
  Trục 2 API Fuzzing │ Trục 3 UI/A11y │ Trục 4 DB Dual │ Trục 5 Perf (k6 + 3 khóa) │ Trục 6 DAST (W3 + Staging URL)
        ├──[11A] raw (PNG/MP4/HAR/trace/SARIF) Direct-to-S3 ──▶ S3 + Object Lock (WORM 90d)
        └──[11B] Envelope ~2KB (exit_code, S3 URI, SHA-256) ──▶ Job Controller
                                                                    │
                                                            S09 Gate (mã cứng)
                                                     PASS │ HOLD │ DO_NOT_PASS
                                                                    │
                              HOLD ──▶ QA Lead/Authority ký Waiver ──▶ PASS (release decision)
```

---

## 5. BA TÀI SẢN "CHỈ MỘT NƠI ĐƯỢC GIỮ" (hiểu để không hỏi sai)

| Tài sản | Ai giữ | Vì sao quan trọng |
| :--- | :--- | :--- |
| **Workflow authority** (state, lease, quota, dispatch) | **Job Controller / RDS (Account A)** | Nếu AI giữ state ⇒ không audit được, không recover được khi worker chết |
| **Secret & URL thật** (`TenantBinding`) | **Account A** (Secrets Manager + STS ngắn hạn) | AI chỉ thấy **symbolic IDs**; nếu AI từng thấy secret ⇒ lộ là lộ vĩnh viễn trong prompt/log |
| **Bằng chứng bất biến** (raw + SHA-256 + WORM 90 ngày) | **S3 Object Lock (Account A)** | *"Bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy"* |

---

## 6. ĐỌC HÌNH v2.1 CẦN BIẾT: 8 KHOẢNG TRỐNG (đừng tưởng hình đã đầy đủ)

> Đây là các điểm **có thật trong file**, tôi đã kiểm trực tiếp trong XML `.drawio`. Khi bạn giải thích cho người khác, hãy nói luôn "cái này hình chưa vẽ".

| # | Khoảng trống | Chi tiết kỹ thuật |
| :--: | :--- | :--- |
| 1 | **Legend định nghĩa nét đứt sai quy ước** | Legend ghi *"Nét đứt = phản hồi / bằng chứng gián tiếp / điều kiện kích hoạt"* — trái quy ước portal + `GLOSSARY_TI.md`: **nét liền = đã đo (OBSERVED), nét đứt = khai báo/chưa đo**. Ngoài ra legend ghi *"THIẾT KẾ ĐÍCH **v2.0**"* trong khi tên/title là **v2.1** |
| 2 | **Không có node S03/S04** | Chỉ có nhãn rời `[07B] ImpactSet + RiskTier`; **không vẽ** node `S03 Impact Engine` / `S04 Risk Engine` ⇒ không truy vết được và đang tranh chấp Account A hay B |
| 3 | **Thiếu toàn bộ nhánh Gate/HOLD/Waiver** | Không có `DO_NOT_PASS` hard-stop, không có `Faithfulness < 0.85 → HOLD`, không có `HOLD → Waiver (QA Lead)`, không có dòng `completed ≠ PASS`, không có `±0.03` và bộ **6 chỉ số GenAI** |
| 4 | **Trục 1 pre-scan đang xung đột tài liệu** | v2.1 ghi `[06]` = pre-scan (Trục 1 chạy sớm) — nhưng `Task 4 §3.1.1` lại xếp Semgrep+Trivy+Gitleaks là **S07 runner** và giao pre-scan S02→S04 cho **CodeGuru+Inspector** (CodeGuru đã EOL) ⇒ cần chốt lại |
| 5 | **Envelope `[11B]` & các cạnh S3 chưa chuẩn** | `[11B]` chỉ vẽ **từ Trục 2**; 5 cạnh runner → S3 đều **không có đầu mũi** (`endArrow=none`) và chỉ **1 cạnh** có nhãn `[11A]` |
| 6 | **`[12D]` vẽ sai đường poll** | Đang vẽ **RDS → CI/CD** trực tiếp trong khi hợp đồng thật là `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` |
| 7 | **Thiếu hạ tầng phục vụ chính nó** | Không có **4 image ECR pre-baked + image digest**, không có đường `GitHub Actions → ECR → SSM` (nét đứt) ⇒ Checklist D04 vẫn Fail. Cũng chưa có node **AgentCore Gateway** và tier **Haiku** (đang tranh chấp — xem §7 nhóm B) |
| 8 | **Memory chưa gắn trạng thái** | Node chỉ ghi `Knowledge Base / S10 Memory / GOLDEN only` — **thiếu** nhãn `UNVERIFIED — chưa nghiệm thu reuse`, dù Task 3 §1.1 ghi *"AgentCore Memory (ACTIVE — chưa nghiệm thu reuse)"* |

---

## 7. BỘ CÂU HỎI GỬI TEAM AI (Nhóm 3 — Nghĩa) — cách hỏi để lấy được BẰNG CHỨNG

### 7.0. Ba quy tắc đặt câu hỏi (áp cho mọi câu dưới đây)

1. **Hỏi "hiện đang chạy cái gì", đừng hỏi "có làm được không".** Câu trả lời phải kèm nhãn: `OBSERVED` (đã đo) / `CANDIDATE` (đề xuất) / `UNVERIFIED`.
2. **Mỗi câu phải đòi 1 trong 4 bằng chứng:** (a) số liệu + ngày đo; (b) đường dẫn file/log/S3 URI; (c) receipt/commit/image digest; (d) tên + version dịch vụ thật. Trả lời kiểu *"đã cấu hình ổn"* **không tính là trả lời**.
3. **Ghi mọi câu trả lời vào bảng log (§8)** — vì mỗi câu trả lời sẽ thành điểm neo cho test case và cho Gate sau này.

### 7.1. Nhóm A — Ranh giới & nơi chạy (quan trọng nhất, đang mâu thuẫn)

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| A1 | **S03 (Impact) và S04 (Risk) chạy ở đâu: Account A (Job Controller) hay Account B (Harness)?** Ai là *system of record* ghi `ImpactSet`/`RiskTier`? Nếu Harness tính thì Harness trả về dạng gì (gợi ý hay quyết định)? | Hình v2.1: nhãn `[07B]` phát từ Trục 1 → Job Controller (ngụ ý Account A). Nhưng `Blueprint §3.3` + `Sequence` + `Workflow_Hung.pdf` + Task 3 lại cho **Harness self-loop S03/S04** ở Account B | Trả lời 1 câu chốt + file/sơ đồ cần sửa; kèm ai phê duyệt |
| A2 | Trong dải S01→S06, bước nào là **mã cứng (deterministic)** và bước nào là **model reasoning**? Liệt kê theo bảng `bước → loại → model/tool` | `Blueprint §2.3` xếp S03 là *"AI suy luận (semantic)"*, S04 là *"threat modeling"*; nhưng hình v2.1 **không có node S03/S04** để kiểm chứng | Bảng 6 dòng, có cột `OBSERVED/CANDIDATE` |
| A3 | Harness có được **tự gọi tool/DB/Internet** không, hay **mọi** hành động phải quay về Job Controller để allowlist rồi mới dispatch? | Law 4.3 (workflow authority) + `AgentCore Gateway (MCP/IAM)` chỉ có ở images/Blueprint nhưng **thiếu ở v2.1** | Mô tả đường đi: model → `ToolIntent` → ai validate → ai gọi ECS RunTask |
| A4 | `ToolIntent` được **validate ở đâu** — trong Harness, tại AgentCore Gateway, hay tại Job Controller? Có schema/JSON Validator chạy thật không, log ở đâu? | Điểm nóng Đối soát §3.6 (P0) + `Task 3` nói candidate phải pass **JSON Schema Validator** | Đường dẫn schema + nơi ghi log validate |

### 7.2. Nhóm B — Model & chi phí (đang có xung đột nội bộ hồ sơ)

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| B1 | **Runtime đang pin model nào?** Task 3 §1.1 ghi *"Runtime v5 kết nối cố định `us.anthropic.claude-opus-5`; **chưa triển khai multi-model hay model tiering động**"*. Vậy sơ đồ *"Sonnet 5 mặc định + Opus 5 khi CRITICAL"* đã chạy chưa hay còn là `CANDIDATE`? | Đây là mâu thuẫn lớn nhất giữa hình v2.1 và hiện trạng AI tự khai | Có/không + nếu chưa: kế hoạch Wave nào, ai làm |
| B2 | **Haiku 4.5 còn hay bỏ?** `Blueprint` (bản sửa 28/09 09:08) ghi *"Dual-Model Tiering 2 cấp — Loại bỏ Haiku 4.5"*, nhưng **bảng FinOps cùng file** vẫn ghi `Haiku/Sonnet/Opus`, `images/*.drawio` vẫn vẽ node `m_haiku`, Task 3 ghi *"Thay thế Tính năng Haiku 4.5"* | Nếu mỗi tài liệu nói một kiểu, không thể viết test case/không thể chốt ngân sách token | **Một quyết định duy nhất** + danh sách file phải sửa |
| B3 | Tên model chuẩn để dùng thống nhất là gì: `Claude Sonnet 5` (Glossary/Task 3) hay `Claude 5.0 Sonnet` (v2.1)? **Model ID Bedrock thật** là gì (kiểu `us.anthropic.claude-sonnet-5`…)? | Checklist A05 (thuật ngữ phải giống hệt), 3 hồ sơ đang ghi 2 kiểu | Model ID đầy đủ + khu vực (`us-east-1`?) |
| B4 | `token_ceiling` chuẩn là bao nhiêu: **6000** (Task 3 §capability manifest) hay **8000** (Task 3 phần guardrail)? Có phải per-tenant không? Vượt trần thì job ra sao (`HOLD`/fail)? | 2 con số khác nhau trong cùng 1 tài liệu → không thể test | 1 con số + hành vi khi vượt trần + nơi log |
| B5 | Các con số mới thêm ở Blueprint (§2.3): **Prompt Caching `$0.20/1M input`** và **Batch API `$1.00/$5.00`** — lấy từ bảng giá nào, ngày nào, và có tính vào ước tính **65–75% tiết kiệm token** không? | Checklist A14/D14/F01: mọi con số chi phí phải có nhãn + nguồn (Đối soát từng chấm số `$0.0008` là "quá thấp, phi thực tế") | Link bảng giá AWS + ngày + nhãn `INFERRED/CANDIDATE` |

### 7.3. Nhóm C — Đo chất lượng GenAI (đang thiếu 3/6 chỉ số + cơ chế ổn định)

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| C1 | **Bộ 6 chỉ số GenAI chốt cuối là gì?** Task 4 §5.3 hiện liệt kê: `GroundednessScore ≥ 0.80`, `GoalSuccessRate ≥ 0.75`, `Precision ≥ 0.70`, `Recall ≥ 0.95`, `HallucinationRate ≤ 2%`, `InjectionResistanceRate ≥ 0.99`. Bộ này có phải bản chốt? | Biên bản §4 yêu cầu **gộp** "6 đặc tính của Hùng" vào "6 đặc tính của Trang"; hình v2.1 **chỉ nêu 2 chỉ số** (Groundedness/Faithfulness) | Danh sách 6 chỉ số + ngưỡng + ai sở hữu |
| C2 | **`Faithfulness ≥ 0.85`** được đo bằng gì — Bedrock Evaluations hay LLM-as-judge tự viết? Chỉ số này **không có** trong bảng 6 chỉ số của Task 4 §5.3 nhưng lại xuất hiện ở cả v2.1 và Detailed — vậy nó nằm ở đâu trong bộ đo? | Biên bản §4: *"Trạng thái HOLD đang bị thiếu tiêu chí Faithfulness"* → yêu cầu bổ sung; hiện 2 tài liệu nói khác nhau | Công thức + nguồn đo + ngưỡng chính thức |
| C3 | **Dải dung sai `±0.03`** áp cho những chỉ số nào, đo trên **bao nhiêu lần chạy**, và nếu lệch quá 0.03 thì làm gì (re-run mấy lần? chuyển `HOLD`?) | Biên bản §4: *"6 chỉ số GenAI cho ra mức điểm khác nhau ở 2 lần chạy liên tiếp dù môi trường và code giữ nguyên"*; Task 3 §"độ ổn định" nói áp cho Groundedness + Faithfulness | Quy trình cụ thể: số lần chạy, ngưỡng lệch, hành động khi lệch, ai chịu trách nhiệm |
| C4 | `temperature 0.0` + `top_p 0.01` được **enforce ở tầng code** (request wrapper tới Bedrock) hay chỉ là **hướng dẫn trong prompt**? Làm sao chứng minh — có log request chứa 2 tham số này không? | V2.1/Detailed đều ghi `temperature: 0.0` như "đã cố định"; cần bằng chứng để không thành claim rỗng | Ảnh/log request thật (có `temperature`, `top_p`, `model_id`) + ngày |
| C5 | Khi eval **không đạt** (`Groundedness < 0.80`): hệ thống **re-generate bao nhiêu lần**, có ghi lại lần fail vào evidence không, sau bao nhiêu lần thì chuyển `HOLD`? Quy tắc retry nằm ở file/policy nào? | Checklist C03: *"lỗi thực thi không được vẽ thành vòng tự thử lại ngầm"* — retry phải **tường minh**; Task 4 §6 nói `HOLD` khi Groundedness < 0.80 cho ≥ 30% candidate | Policy retry bằng văn bản + số lần + nơi lưu log |
| C6 | **Bedrock Evaluations có thật đang bật không**, ở account nào, và module/tên thật là gì (`TIRunnerGroundness` trong Detailed viết đúng chưa)? Ai sở hữu ngưỡng theo tenant (Evaluation Pack)? | Detailed gọi module là `TIRunnerGroundness` (nghi ngờ sai chính tả/"Groundness"); ngưỡng P95/P99/error-rate lại nằm trong `EVAL_PACK.perf.*` | Tên module + ARN/nơi cấu hình + chủ sở hữu ngưỡng |

### 7.4. Nhóm D — Hợp đồng dữ liệu (để test được "Zero secret")

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| D1 | `ToolIntent JSON` schema hiện ở **version nào**, ai là owner, công bố ở URL nào, các field **bắt buộc** là gì? | Detailed §6.1 ghi `$schema: …/tool-intent-v2.json`; Blueprint §5.1 cũng vẽ contract → cần xác nhận có thật | URL schema + version + owner |
| D2 | **Làm sao chứng minh AI không bao giờ thấy secret/URL thật?** Có cơ chế/test tự động nào (kiểm tra prompt đã redact, chặn secret pattern) hay chỉ là quy ước? | Law 13 + Checklist B08/D09; Biên bản yêu cầu *"Zero-Leak Secret"* | Cơ chế cụ thể (filter/regex/policy ở đâu) + log/test chứng minh |
| D3 | Ai **nội suy URL thật** và cấp **STS token**? TTL bao lâu? Sau khi job xong có **thu hồi** token không (Blueprint §3.2 có bước *"Thu hồi STS"*)? | Law 13 — server-owned binding; nếu token không thu hồi thì không đạt "short-lived" | TTL + bằng chứng revoke + ai gọi STS |
| D4 | `[11B] Envelope` gồm đúng những field nào, có **schema version** không, và **kích thước ~2KB đã đo thật chưa**? | Detailed §[11B] ghi *"~2 KB"*; v2.1 chỉ vẽ envelope từ Trục 2 → cần xác nhận hợp đồng chung | Bảng field + số byte đo được + ngày |

### 7.5. Nhóm E — AgentCore · Memory · Prompt Injection

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| E1 | **Version runtime thật là gì?** Task 3 §1.1 ghi *"AgentCore Harness (runtime **v5**)"*, còn checklist/receipt ghi *"AgentCore **Runtime 18** — receipt 14/09"*. Cái nào mô tả bản đang chạy, receipt nào, ngày nào? | Checklist A10/D08/E03: mọi tham chiếu Runtime/receipt phải khớp portal tại ngày review | Số runtime + ngày + receipt/commit |
| E2 | Hiện **chỉ có 2 tool adapter** (`http-request`, `ti-playwright`), **chưa có adapter cho DB / Performance / Security**. Vậy 6 trục runner trong v2.1 có đường triển khai chưa (Wave nào, ai làm, bao giờ)? | Task 3 §1.1 nói rõ hiện trạng chỉ 2 adapter; hình v2.1 vẽ 6 trục như thể đã có | Lộ trình: trục nào W1/W2/W3, owner, điều kiện mở |
| E3 | **Memory S10**: hiện ghi cái gì, tiêu chí nào thì một testcase được coi là `GOLDEN`, ai **duyệt**, và lượt sau AI **có đọc lại** memory không? Nếu có thì chống "ô nhiễm tri thức" (pesticide paradox) bằng cách nào? | Task 3 §1.1: *"AgentCore Memory (ACTIVE — **chưa nghiệm thu reuse**)"*; Checklist E10 đòi gắn nhãn `UNVERIFIED` | Tiêu chí GOLDEN + người duyệt + cơ chế reuse + nhãn đúng trên sơ đồ |
| E4 | **4-barrier Prompt Injection** đã implement hay mới là thiết kế? Có bộ payload test nào để đo `InjectionResistanceRate ≥ 0.99` chưa, kết quả hiện tại bao nhiêu? | Task 4 §5.3 yêu cầu đo `InjectionResistanceRate ≥ 0.99`; Biên bản §3: *"TI không có UI Chat → prompt injection phải nhìn dưới góc quét payload độc hại lọt vào source code"* | Bộ payload + kết quả + ngày; barrier nào đang chạy thật |

### 7.6. Nhóm F — Bằng chứng · Retry/Idempotency · Quota

| # | Câu hỏi (dùng nguyên văn) | Vì sao hỏi (điểm neo) | Trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| F1 | Ứng với mỗi bước AI (`[08B]`, `[09A]`, `[09B]`), **artifact nào được lưu**, ở path/S3 URI nào, **ai băm SHA-256**, và có lưu **prompt + response** để audit không? | Checklist D10/E08: bằng chứng phải normalize + hash → S3, không quay lại ổ EC2 | Bảng `bước → artifact → path → ai hash`; nêu rõ prompt/response có redact hay không |
| F2 | Nếu Harness/task **chết giữa chừng**, job resume thế nào? Lease TTL và nhịp heartbeat thực tế là bao nhiêu (**60s** hay **5 phút**)? Có recovery/kill task treo không? | 2 hồ sơ ghi 2 mức khác nhau (Sequence "Heartbeat 60s" vs Hùng/Blueprint "Lease TTL 5 mins"); Checklist B02 đòi lease/heartbeat/recovery | 1 con số thống nhất + mô tả recovery + bằng chứng test (nếu có) |
| F3 | Vượt `token_ceiling` hoặc vượt quota compute thì job ra sao (fail cứng, `HOLD`, hay cắt ngắn)? Có log/evidence cho việc đó không? | Task 3 guardrail `token_ceiling`; v2.1 ghi `[04]` "quota/timeout" nhưng không nói hành vi | Hành vi rõ ràng + nơi ghi log + mã lỗi |

### 7.7. Tin nhắn mẫu để gửi (copy–paste)

> Chào Nghĩa/Nhóm 3, mình đang rà lại bộ sơ đồ kiến trúc v2.1 (`TI_System_Architecture_v2.1_CANDIDATE.drawio`) để chuẩn bị Vòng 2 đối chiếu evidence. Mình cần chốt **hiện trạng đang chạy** vs **đề xuất**, nên nhờ bạn trả lời 6 nhóm câu hỏi (A–F) trong `diagram/TI_System_Flow_End_to_End_v2.1_Explained.md §7`. Quy ước trả lời:
> 1. Mỗi câu ghi rõ nhãn `OBSERVED` (đã đo — kèm ngày) / `CANDIDATE` (đề xuất) / `UNVERIFIED`;
> 2. Kèm bằng chứng dạng: số liệu + ngày, đường dẫn file/log/S3 URI, hoặc receipt/commit/image digest;
> 3. Câu nào chưa biết thì ghi thẳng "chưa có bằng chứng" — **không cần** trả lời cho đủ.
> Ưu tiên 4 câu P0: **A1 (S03/S04 ở account nào), B1 (runtime đang pin model nào), B2 (Haiku bỏ hay giữ), C5 (retry eval)**. Trả lời xong mình đối chiếu và cập nhật bảng §8.

---

## 8. BẢNG GHI NHẬN CÂU TRẢ LỜI (để thành bằng chứng, không phải lời nói)

| # | Câu hỏi | Người trả lời | Ngày | Bằng chứng kèm theo | Kết luận (Pass/Fail) |
| :--: | :--- | :--- | :--- | :--- | :--: |
| A1 | S03/S04 ở account nào | | | | ☐ |
| A4 | ToolIntent validate ở đâu | | | | ☐ |
| B1 | Runtime đang pin model nào | | | | ☐ |
| B2 | Haiku 4.5 bỏ hay giữ | | | | ☐ |
| C1 | Bộ 6 chỉ số GenAI chốt | | | | ☐ |
| C3 | Cơ chế `±0.03` | | | | ☐ |
| C5 | Retry khi eval fail | | | | ☐ |
| E1 | Runtime version + receipt | | | | ☐ |
| E3 | Memory GOLDEN & reuse | | | | ☐ |
| F2 | TTL lease / heartbeat | | | | ☐ |

> **Cách chấm:** câu trả lời **có bằng chứng và khớp hồ sơ** ⇒ `Pass/FIXED`; trả lời lệch sơ đồ ⇒ sinh 1 defect + sửa sơ đồ; **không có bằng chứng** ⇒ theo ADR-0003 mục đó = **Fail**, buộc đổi cách diễn đạt trên sơ đồ (nét đứt / nhãn `CANDIDATE` / `UNVERIFIED`).

---

## 9. NGUỒN THAM CHIẾU ĐỂ TRA LẠI (khi cần tranh luận)

| Chủ đề | File · vị trí |
| :--- | :--- |
| Luồng & nhãn `[01]→[12D]` của hình v2.1 | `diagram/TI_System_Architecture_v2.1_CANDIDATE.drawio` — L10 legend · L40–L61 Job Controller & Gate · L85–L110 API/Secrets · L138–L155 Bedrock/Evaluations · L164–L252 Sandbox 6 trục · L303 ghi chú Trục 1 · L334 `[09B]` · L402 `[12D]` |
| Chuẩn Gate 3 nhánh | `Research/Task_4_Test_Plan_Strategy_Evaluation.md` §6.1–§6.2 (L602–L613) và §5.3 bộ 6 chỉ số (L482–L493) |
| Hiện trạng AI (adapter, model, memory) | `Research/Task_3_AI-Prompt_Research.md` §1.1 (L70–L81); phần ổn định `temperature 0.0` / `top_p 0.01` / `±0.03` |
| Vì sao chọn Fargate thay CodeBuild/Lambda + mạng D9 | `Research/BAO_CAO_DOI_SOAT_MISMATCH_VA_DONG_BO_TI.md` §3.3 (L146–L158), L306, L317; `Research/Task_2-Architecture-Report.md` L141–L169 + ADR-03 (L300) |
| Quy ước nhãn sự thật & 3 ranh giới | `Research/GLOSSARY_TI.md` §A/§B; `Research/Task_5_Shift_Left_Verification_Plan_and_Checklist.md` A01–A15, E10 |
| Điểm nóng biên bản 23/09 | `PDF/Biên bản cuộc họp TI - 23_09_2026.pdf` |
| Phiếu re-review 5 hồ sơ (11 Blocking + 16 Minor + 10 mismatch) | `Research/Feedback/Task_5_ReReview_v2.1_CANDIDATE_Team_TI_Deliverables.md` |

## CHANGELOG

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 28/09/2026 | v1.0 | Cline (soạn theo yêu cầu) | Tài liệu đọc-hiểu luồng TI end-to-end theo v2.1: quy ước nhãn sự thật, bảng quyền hạn 11 thành phần, luồng `[01]→[12D]` chi tiết, sơ đồ chữ 1 trang, 3 tài sản "một nơi giữ", 8 khoảng trống của v2.1, **bộ 26 câu hỏi (A–F) gửi team AI** + mẫu tin nhắn + bảng ghi nhận bằng chứng |







