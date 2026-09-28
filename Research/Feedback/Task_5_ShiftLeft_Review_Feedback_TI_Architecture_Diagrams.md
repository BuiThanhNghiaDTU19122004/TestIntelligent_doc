# PHIẾU GÓP Ý SHIFT-LEFT — 2 TÀI LIỆU KIẾN TRÚC + 1 SƠ ĐỒ DRAWIO/PNG CỦA TI

> **Đối tượng đánh giá:**
> 1. `D:\Doc\diagram\TI_Master_Architecture_Blueprint.md` (v1.0 — ánh xạ Checklist **B** · S1 kiến trúc tổng thể)
> 2. `D:\Doc\diagram\TI_Detailed_System_Architecture.md` (v2.0.0-Master-Sync — ánh xạ Checklist **D** + các mục **C** áp dụng)
> 3. `D:\Doc\diagram\TI_System_Architecture.drawio` + `TI_System_Architecture.drawio (1).png` (cùng bộ với #2)
>
> **Phương pháp:** ADR-0001 (2 vòng: Review tĩnh + Đối chiếu evidence) · **Nguồn sự thật:** ADR-0002 · **Thang:** ADR-0003 (Pass/Fail nhị phân; Fail tách `Blocking` ≤24h / `Minor` ≤3 ngày) · **Cấu trúc:** ADR-0004
> **Ngày lập:** 27/09/2026 · **Bên góp ý:** AI reviewer (tổng hợp) — **Chốt Pass/Fail cuối:** Nghĩa (Tester #4) theo Task_5 §2 RACI
> **Trạng thái:** 🔴 **CHƯA ĐẠT VÒNG 1** — 11 Blocking + 16 Minor. Chưa đủ điều kiện nộp vòng evidence (Task_5 §9).

---

## 1. PHẠM VI, ÁNH XẠ CHECKLIST & GIẢ ĐỊNH

| Đối tượng | Ánh xạ Task_5 | Ghi chú |
| :--- | :---: | :--- |
| Master Blueprint (2 sơ đồ Mermaid C4 + 8 mục) | **A + B + F** | Tương ứng S1 (kiến trúc tổng thể — Hoàng) |
| Detailed System Architecture (3 sơ đồ Mermaid + 7 mục) + drawio/PNG | **A + D + F** + phần **C** (C04, C06, C07, C08, C13) | Tương ứng S3 (chi tiết service/tool — Trang) |
| Cả 3 | **E** (Evidence Pack) | Vòng 2 — đã chạy trước 1 phần để phục vụ review tĩnh |

- **Giả định A1′:** 3 file này được coi là draft cho 3 sơ đồ của Biên bản 23/09 (S1/S2/S3). Nếu phân công khác, chỉ cần đổi cột "Ánh xạ", nội dung defect không đổi.
- **Giả định A2′:** Checklist C (workflow S01–S10 của Hùng) chỉ chấm các mục thực sự xuất hiện trong 2 tài liệu; bộ sơ đồ 10 lane S01–S10 chưa có trong scope lần này.
- **Ngoài phạm vi:** sửa nội dung — tác giả sửa, tester chỉ ghi defect (Task_5 §1.3).

---

## 2. NGUỒN SỰ THẬT ĐÃ DÙNG (EVIDENCE PACK — ADR-0002)

| Pack | Nguồn (ưu tiên) | Thời điểm | Trích dẫn then chốt |
| :--: | :--- | :--- | :--- |
| **EV-1** | `GET https://d1tibdarzmw3jq.cloudfront.net/diagrams` (HTTP 200, 68.953 bytes) | 27/09/2026 | "Kiến trúc triển khai và chức năng theo **lượt đo 2026-09-22 trên nguồn b55fed16** — ảnh chụp có ngày, **không** phải bản đang phục vụ"; "Hạ tầng **Đo 2026-09-22** … **nguồn ghim 8a61cf66**: CloudFront → **API :8000 và portal :8001 trên CÙNG MỘT EC2** ở ap-southeast-1; **kết quả ghi xuống ổ bằng chứng EBS**"; "Đường giao hàng GitHub Actions → ECR → SSM — **vẽ nét đứt vì là đường khai báo**"; spec 007 gắn nhãn "**ĐỀ XUẤT · NOT_DEPLOYED**"; "**Memory reuse chưa được nghiệm thu**"; job mới = `POST /v2/artifact-jobs`, `/v1/testing/changes` LIVE nhưng **FROZEN** |
| **EV-2** | `GET https://d1tibdarzmw3jq.cloudfront.net/evidence` (HTTP 200, 103.981 bytes) | 27/09/2026 | Manifest **2026-09-23.2** (tổng hợp 2026-09-23T11:40:00Z); bản phát hành xác minh độc lập gần nhất: nguồn `1af743a8` · ảnh backend `e0a035974077` · gói UI `6105ac882064`; đọc lại tại edge 2026-09-23T08:09:35Z: `/`, `/docs`, `/diagrams`, `/diagrams/architecture-v2` = 200; "trang này báo cáo bằng chứng; **KHÔNG phải phê duyệt, KHÔNG nghiệm thu**" |
| **EV-3** | `D:\Doc\Research\GLOSSARY_TI.md` · `BAO_CAO_DOI_SOAT…§3.9` · Biên bản 23/09 (trích trong Task_5) | 25/09/2026 | Từ cấm dùng (CodeGuru EOL), kỷ luật nhãn, 10 điểm nóng P0–P3 |

> **Kết luận sự thật then chốt từ EV-1/EV-2:** hiện trạng **đang chạy** = EC2 1 máy (API :8000 + portal :8001), evidence chưa sang S3. Toàn bộ nội dung Fargate / Job Controller lease / 6 trục / Direct-to-S3 trong 3 đối tượng review là **thiết kế đích** → bắt buộc gắn nhãn `CANDIDATE` (xem DEF-X-002).

---

## 3. TỔNG KẾT PASS/FAIL (ADR-0003 — nhị phân)

| Nhóm checklist | Tổng mục | Pass | Fail | Ghi chú |
| :---: | :--: | :--: | :--: | :--- |
| **A** — Chung (cả 3) | 15 | 5 | **10** | A04, A07, A09, A10, A15 Pass |
| **B** — Blueprint (S1) | 15 | 11 | **4** | B10, B11, B13, B15 Fail |
| **D** — Detailed + drawio/PNG (S3) | 15 | 6 | **9** | — |
| **C** — Mục workflow áp dụng | 5 | 0 | **5** | Chấm theo Detailed (C04, C06, C07, C08, C13); riêng Blueprint Pass C13 |
| **E** — Evidence | 10 | 9 | **1** | E10 Fail; E03–E05/E07–E08 Pass-kèm-vì-không-claim |
| **F** — 10 điểm nóng Biên bản | 10 | 4 | **6** | F03, F04, F05, F06 Pass |

**Kết luận:** cả Blueprint và bộ Detailed+drawio đều **CHƯA ĐẠT** (cần 100% Pass). Theo §8.1: 11 defect **Blocking** phải sửa ≤24h trước re-review; 16 defect **Minor** ≤3 ngày làm việc.

---

## 4. ĐIỂM LÀM TỐT (CẦN PHÁT HUY — đúng nguyên lý TI)

1. **Law 16 — Bằng chứng bất biến:** Direct-to-S3 + Object Lock (WORM 90 ngày) + SHA-256 digest; envelope ~2KB tách khỏi raw artifacts → đóng đúng điểm nóng EC2 storage ngộp (B03 ✅, D10 ✅, F04 ✅).
2. **Law 23 — Provider Port:** `IsolatedRunner` + ECS RunTask Port Launcher, task-per-job, không mô hình "Fargate lồng Fargate" (B04 ✅).
3. **Điểm nóng P0 Đối soát §3.6:** công thức `Target Runners = ImpactSet (S03) ∩ TargetBinding.EnabledDomains (S01)` xuất hiện nhất quán ở cả 2 tài liệu và label `[10]` trên drawio (B09 ✅, F06 ✅) — không có cảnh "PR sửa API → chạy cả 6 trục".
4. **Laws 10.1/13 — Hợp đồng sạch:** ToolIntent JSON chỉ chứa symbolic params; TenantBinding server-side, credential qua STS ngắn hạn, không lọt vào prompt (B08 ✅, D09 ✅).
5. **Đóng đúng các điểm nóng P1/P2 của Đối soát:** D5a (Semgrep+Trivy+Gitleaks) = W1, D5b (ZAP/nuclei) = W3 chỉ khi có Staging URL sống (đóng P2 điểm 8); DB Dual SQL Aurora Clone + NoSQL DynamoDB TTL 1h + DeleteTable hook (đóng P1 điểm 5, F05 ✅); mô hình 3 tầng, Opus 5 chỉ khi CRITICAL; `temperature: 0.0` + `Faithfulness ≥ 0.85` (đóng một phần P1 điểm 9).
6. **Hướng mạng D9 đúng bản chất:** bỏ Network Firewall → private subnet + SG Deny-All + VPC Endpoints (B07 ✅ ở Blueprint) — trừ cách diễn đạt `--network none` (DEF-X-004).
7. **Bộ sơ đồ Mermaid 4.1–4.3** trong Detailed rất hữu ích cho review (topology + sequence + pipeline) — nên giữ làm "bản đồ chữ" kèm drawio.

---

## 5. DEFECT LOG — BLOCKING (SỬA ≤ 24H — Task_5 §8.1)

### DEF-X-001 · `A06`, `D13` — Amazon CodeGuru Security (đã EOL) trong Blueprint
- **Vị trí:** `TI_Master_Architecture_Blueprint.md` §4.3 "Tầng 2 — AWS Native Inspection": *"**Amazon CodeGuru Security**: Phân tích AST diff của PR bằng Machine Learning"*.
- **Vì sao Fail:** CodeGuru Security **ngừng hoạt động 20/11/2025** — GLOSSARY_TI mục "Từ cấm dùng" + Đối soát §3.1 (**P0**) + Task_5 A06/D13. Research Task 1 cũng "khai tử" công cụ này.
- **Sửa:** Xóa CodeGuru; giữ **Amazon Inspector** (ECR image scan) + Semgrep/Trivy (pre-runner S02→S04 theo Task 4 §3.1.1). Nếu buộc nhắc lịch sử phải ghi "(EOL 20/11/2025 — KHÔNG dùng trong TI)".
- **Owner:** Hoàng (Blueprint) · **Deadline:** T+24h.

### DEF-X-002 · `A02`, `A03`, `B15`, `D02`, `D15` — Thiếu nhãn sự thật + legend; trộn thiết kế đích với hiện trạng
- **Vị trí:** PNG (không legend, không nhãn), drawio, cả 2 tài liệu.
- **Vì sao Fail:** EV-1: hiện trạng live (đo 22/09, ghim `8a61cf66`) = **API :8000 + portal :8001 trên cùng 1 EC2, evidence ghi xuống EBS**. Sơ đồ vẽ Fargate, Job Controller lease, 6 trục, Direct-to-S3 = **thiết kế đích** nhưng không gắn nhãn `CANDIDATE`, không legend nét liền/đứt (Đối soát §3.9; portal dùng đúng mẫu "ĐỀ XUẤT · NOT_DEPLOYED"). Blueprint còn gắn `[OBSERVED]` cả 2 vùng Account A/B → **claim vượt evidence** (lease/heartbeat chưa có lượt đo live).
- **Sửa:** (1) Banner trên PNG + tiêu đề 2 tài liệu: `THIẾT KẾ ĐÍCH v2.0 — CANDIDATE · chưa triển khai (chờ Spike P4 / Wave 1)`; (2) thêm **Legend**: nét liền = thiết kế đã chốt, nét đứt = khai báo/chưa đo; ghi chú hiện trạng EC2 "đo 22/09 — ghim 8a61cf66"; (3) bỏ hoặc hạ cấp `[OBSERVED]` từng node.
- **Owner:** Hoàng + Trang · **Deadline:** T+24h.

### DEF-X-003 · `A12`, `A13`, `F02` — Claim "khớp 100% [01]→[12D]" nhưng drawio thiếu nhãn [01], [02], [05]
- **Vị trí:** Detailed header ("Bám sát **100%** cấu trúc… đối soát trực tiếp") + §2; drawio: cạnh WAF→TI API `value=""`, cạnh API⇄Job Controller không nhãn, cạnh CloudFront↔icon settings 2 chiều không nhãn; External zone **thiếu actor "CI/CD Pipeline"** (chỉ icon settings vô danh).
- **Vì sao Fail:** Không truy vết được 3 bước đầu của chuỗi chuẩn tắc (A13); icon vô tên (A12); claim "100%" là claim vượt hiện trạng (F02).
- **Sửa:** Thêm nhãn `[01] POST /v2/artifact-jobs`, `[02] Forward qua WAF`, `[05] Enqueue job`; thêm node "CI/CD Pipeline (GitHub Actions / GitLab CI)" vào External; hoặc sửa claim thành "khớp [03]→[12D]; [01][02][05] đang được bổ sung".
- **Owner:** Trang (drawio) · **Deadline:** T+24h.

### DEF-X-004 · `B07` — `--network none` mâu thuẫn kỹ thuật với chính luồng Evidence
- **Vị trí:** drawio title Sandbox `· --network none`; Detailed §1 "DENY ALL EGRESS (--network none)" và §3 header (kèm "3 VPC Endpoints ~$22/mo").
- **Vì sao Fail:** Container Fargate `--network none` **không có network interface** → không thể push S3 `[07A]/[11A]`, không kéo ECR, không gửi envelope `[11B]`, không chạm VPC Endpoint — vô hiệu chính luồng mà tài liệu mô tả. Vi phạm Law 5/7 (diễn đạt phải đúng kỹ thuật).
- **Sửa:** Thay bằng cụm chính xác: `Private Subnet (không IGW/NAT) + Security Group DENY ALL EGRESS + VPC Endpoints (S3 Gateway $0; ECR/Logs/STS Interface ~$22/th)`.
- **Owner:** Trang + Hoàng · **Deadline:** T+24h.

### DEF-X-005 · `A11`, `A13` — Nơi đặt VPC Endpoints không nhất quán + cạnh vô nghĩa
- **Vị trí:** Detailed **§1** đặt "3 Interface Endpoints + S3 Gateway" trong **Account A**; **§3** đặt trong **Sandbox**; drawio đặt node trong swimlane `acc_a` và có cạnh dashed **VPC Endpoints → Knowledge Base S10 (Account B)** không rõ nghĩa.
- **Vì sao Fail:** Endpoint chỉ có ý nghĩa trong VPC dùng nó (Sandbox cần ECR/Logs/S3 để kéo image & đẩy bằng chứng); 2 chỗ trong cùng tài liệu mâu thuẫn; cạnh → KB không truy vết được nguồn (A13).
- **Sửa:** Đặt endpoints vào Sandbox (đúng D9); nếu Account A có VPC riêng thì ghi rõ 2 nhóm tách biệt; xóa cạnh VPC Endpoints → Knowledge Base.
- **Owner:** Hoàng · **Deadline:** T+24h.

### DEF-X-006 · `A11` (P0 Đối soát §7) — S03/S04 chạy ở đâu? Hai tài liệu nói hai nơi
- **Vị trí:** Blueprint §3.3 sequence: *"Harness→Harness: **S03 Impact & S04 Risk Tiering** (Opus 5 if Critical)"* = Account B; Detailed §2 `[07B]`: *"Trục 1 gửi … về **Job Controller** để: **S03** Impact Engine xuất ImpactSet, **S04** Risk Engine tính Risk Tier"* = Account A. Blueprint §2.3 lại phân loại S03 là "AI suy luận (semantic)".
- **Vì sao Fail:** Mâu thuẫn xuyên tài liệu (A11); Law 4.3 (Job Controller là system of record) không ăn khớp cách phân loại; Đối soát §7 (**P0**) đòi 1 ma trận mapping Changeset → Runner duy nhất.
- **Sửa:** Chốt 1 nơi chủ trì S03/S04 (đề xuất: **deterministic → Account A / Job Controller**; nếu semantic ở Account B thì chỉ trả `ImpactSet` về) và sửa đồng bộ cả 2 tài liệu + drawio.
- **Owner:** Hoàng + Nghĩa (Task 3) · **Deadline:** T+24h · Không đồng ý → escalate Tan.Thai (Task_5 §8.1).

### DEF-X-007 · `B08`, `D09` (Law 13) — Secrets Manager vẽ sai tài khoản AWS
- **Vị trí:** drawio node `secrets` thuộc swimlane **`acc_b` (us-east-1)** "Tenant Secrets & STS Tokens"; Detailed §6.2 ghi secret tra cứu/giữ ở **Account A** với ARN `arn:aws:secretsmanager:**ap-southeast-1**:…`.
- **Vì sao Fail:** Law 13 — server Account A sở hữu binding & credential; drawio đặt ở Account B trái mô tả trong §6.2 → mơ hồ chỗ sở hữu secret (rủi ro hiểu nhầm credential đi qua AI Brain).
- **Sửa:** Vẽ **Tenant Secrets & STS → Account A**; nếu Account B có khóa riêng (model API key) tách node "Model Credentials (Account B)".
- **Owner:** Trang · **Deadline:** T+24h.

### DEF-X-008 · `D05`, `B04` — Dòng "CodeBuild/Fargate" tái mở câu hỏi đã chốt P1
- **Vị trí:** Detailed §5 (Bảng ma trận FinOps) dòng `[06]`: "Thành phần AWS: **Job Controller ➔ CodeBuild/Fargate**".
- **Vì sao Fail:** Đối soát §3.3 (**P1 — "Chốt Fargate task-per-job"**) + Task_5 D05: còn nêu CodeBuild/Lambda cho runner = Fail (cold start 45–90s + setup Playwright mỗi lần = lãng phí); mâu thuẫn luôn với Port Launcher/Fargate elsewhere trong cùng tài liệu.
- **Sửa:** Chỉ còn **ECS Fargate (qua Port Launcher)**; xóa CodeBuild.
- **Owner:** Trang · **Deadline:** T+24h.

### DEF-X-009 · `C04`, `C06`, `C07`, `C08`, `C13`, `F10` — Gate S09 chưa đủ nhánh theo Task 4 §6
- **Vị trí:** Detailed §2 `[12B]` (boolean `Critical==0 & p95<SLA & Groundedness>=0.80 & Faithfulness>=0.85`), `[09B]`, bảng FinOps `[12B]` "Cập nhật COMPLETED + PASS/HOLD/DO_NOT_PASS".
- **Vì sao Fail (5 điểm):**
  1. Thiếu **hard-stop `DO_NOT_PASS`**: secret lộ, hash SHA-256 mismatch, rollback fail (Task 4 §6.2);
  2. Không gán **`Faithfulness < 0.85 → HOLD`** (C07 — Đối soát §3.9 P1); không có nhánh HOLD → **Waiver + Human review** (Blueprint có `opt HOLD→QA`, Detailed không → A11); ranh giới **KHUYẾN NGHỊ ≠ PHÊ DUYỆT** bị mờ;
  3. Không có nhánh **fail của `[09B]`** (Groundedness < 0.80 → tái tạo hay HOLD?) và không nói retry tường minh (C03 — portal: lỗi không được vẽ thành vòng tự thử lại ngầm);
  4. **Thiếu tolerance ±0.03** cho 2 lần chạy khác điểm (C08/F10);
  5. Chỉ nêu **2/6 chỉ số GenAI** (Groundedness, Faithfulness) — thiếu bộ chỉ số đã gộp (F10).
  - Kèm đó: `[12B]` gộp `COMPLETED` (state) với `PASS/HOLD/DO_NOT_PASS` (gate) → trái Law 18 `completed ≠ PASS` (Blueprint có ghi Law 18, Detailed không).
- **Sửa:** Bảng nhánh Gate 3 trạng thái đầy đủ + tách `state` khỏi `gate_result`; ghi rõ HOLD = khuyến nghị chờ người duyệt.
- **Owner:** Hoàng + Nghĩa (Task 4 §6) · **Deadline:** T+24h.

### DEF-X-010 · `A14`, `F01`, `F02` — Bảng FinOps Detailed: số không nhãn, dùng lại số đã bị chốt là "ảo"
- **Vị trí:** Detailed §5 — cột "Chi phí FinOps Ước tính" (19 dòng) **không có cột nhãn**; dòng `[06]` "**~$0.0008 / lượt quét diff**"; `[11B]` "Xóa sổ **100%** rủi ro"; `[03]`/`[06]` in đậm SLA "< 1s", "2–5s" như đã đo.
- **Vì sao Fail:** (1) Đối soát §3.9 **điểm 10 (P2)** chính là chấm số `$0.0008/build` là "**quá thấp, phi thực tế**" — nay bị dùng lại y nguyên; (2) toàn bộ số chi phí thiếu nhãn `CANDIDATE/INFERRED` trong khi Blueprint §6 **có** nhãn → không nhất quán (A11); (3) "100%", "khớp 100%" = claim tuyệt đối không evidence (F02).
- **Sửa:** Thêm cột Nhãn (mặc định `CANDIDATE`, chuyển `INFERRED/OBSERVED` sau Spike P4); tính lại $0.0008 theo đơn giá thật; đổi "Xóa sổ 100%" → "loại bỏ hẳn nhóm rủi ro ngộp ổ EBS"; đánh dấu SLA là *mục tiêu thiết kế*.
- **Owner:** Trang (bảng) · **Deadline:** T+24h.

### DEF-S3-011 · `B09`, `C02`, `D06`, `F09` — Trục 1 bị dispatch 2 lần + mập mờ pre-runner vs S07 runner
- **Vị trí:** drawio có **cả hai**: `e_jc_sast` `[06]` Job Controller → Trục 1 **và** `e_pl_r1` Port Launcher → Trục 1 (sau `[10]`); Detailed §2 `[06]` (chạy **trước** AI `[08]`) nhưng `[10]` lại liệt kê `e_pl_r1` một lần nữa.
- **Vì sao Fail:** Task 4 §3.1.1: Semgrep+Trivy+Gitleaks là **S07 runner chính thức**; Detailed lại dùng làm pre-runner cấp input S03/S04 → không rõ Trục 1 chạy **1 hay 2 lần**, ranh giới S02→S04 vs S07 mờ (Task_5 D06, F09). Song song đó, cả 2 tài liệu **chưa ghi gap**: Semgrep/Trivy hiện scan source code, chưa đáp ứng trọn hướng scan của Testing (Biên bản §3) và chưa nêu góc **prompt injection = quét payload độc hại lọt vào source code** (TI không có UI Chat).
- **Sửa (chốt 1 trong 2):** (a) `[06]` = pre-scan S02→S04 và `[10]` **không dispatch lại Trục 1** (chỉ r2–r6); hoặc (b) gộp Trục 1 vào batch `[10]` và dời AI `[08]` về sau — nhưng khi đó phải vẽ lại thứ tự. Kèm footnote gap scan + prompt-injection scope.
- **Owner:** Hoàng + Trang · **Deadline:** T+24h.

---

## 6. DEFECT LOG — MINOR (SỬA ≤ 3 NGÀY LÀM VIỆC)

| ID | Checklist | Vị trí | Nội dung Fail | Đề xuất sửa |
| :--: | :---: | :--- | :--- | :--- |
| `DEF-S3-M01` | A01 | PNG title | Chỉ có tên + v2.0; **thiếu ngày tạo/ngày đo, người vẽ, nhóm nhãn** | Thêm dòng: ngày · tác giả · nhóm (kiến trúc/hạ tầng) |
| `DEF-X-M02` | A05, C04, C11 | Cả 2 tài liệu | Trạng thái job lệch chuẩn: Detailed dùng `PENDING/COMPLETED`, Blueprint dùng `QUEUED`, portal chuẩn `queued/running/completed/failed`; `[12B]` gộp state + gate | Dùng đúng bộ state của portal/Glossary; tách `state` và `gate_result` (kèm DEF-X-009) |
| `DEF-X-M03` | A05, A11, D07 | Blueprint §2.3 vs Detailed `[08B]` vs drawio | Model tiering lệch pha: Blueprint 3 cấp (**Haiku 4.5**/Sonnet 5/Opus 5); Detailed + drawio chỉ 2 cấp; tên "**Claude 5.0 Sonnet**" ≠ "**Claude Sonnet 5**" (`anthropic.claude-sonnet-5` — Task 3); drawio Account B thiếu node **AgentCore Gateway/Policy** (Blueprint có; Biên bản §4 đòi "chẻ rõ Bedrock") | Thống nhất tên model theo Glossary/Task 3; thêm Haiku tier + Gateway/Policy vào Account B |
| `DEF-X-M04` | A11, B05 | Blueprint §3.1 C4 | Blueprint chỉ vẽ **5 runner** (thiếu API Functional runner — Trục 2) so với 6 trục của Detailed; đánh số luồng "1..10" khác hệ "[01]..[12D]" | Thêm runner API; chuẩn hóa đánh số hoặc chú giải ánh xạ 2 hệ số |
| `DEF-S3-M05` | A12 | PNG/drawio | Label `[10]` và `[11B]` hiển thị **trong swimlane Account B** (sai vùng); `[11B]` chỉ vẽ từ Trục 2 trong khi doc nói mọi runner gửi envelope; 5/6 cạnh →S3 không nhãn `[11A]` (chỉ Trục 4 có), vài cạnh `endArrow=none` | Di chuyển label về đúng cạnh/vùng; gộp cạnh S3 + 1 label; thêm arrowhead |
| `DEF-S3-M06` | D02 | Detailed/drawio | Không ghi đối chiếu hiện trạng: "đang chạy 1 EC2 (đo 22/09 — ghim `8a61cf66`), 2 container cách ly `/app/data`" → không thấy được vì sao khác bản vẽ | Footnote "Hiện trạng (OBSERVED) → Thiết kế đích (CANDIDATE)" |
| `DEF-S3-M07` | D03, F07 | Detailed §3 | **Chưa chỉ đích danh nơi lưu script k6/Playwright** + cách version (điểm mở Biên bản §3) | Ghi "repo `ti-test-packs/` (đề xuất — CANDIDATE), version theo git tag" hoặc đánh dấu gap + owner |
| `DEF-S3-M08` | B10, F08 | Detailed §3.5 Trục 5 | Bộ 3 khóa ✓ nhưng **thiếu cảnh báo AWS ban / "tự DoS người nhà"** + bước xin phép DevOps/ranh giới vùng chạy | Thêm hộp "Chỉ bắn trong vùng được phép — xin phép DevOps trước (Biên bản §3)" |
| `DEF-X-M09` | B11 | Detailed §3.4 / Blueprint §4.2 | Aurora Clone **thiếu ghi chú "chờ tham vấn Team Data về quota snapshot"** (Đối soát dòng P3) | Thêm footnote status `PENDING tham vấn` |
| `DEF-S3-M10` | D06, F09 | Detailed §3.1 | Chưa nêu **gap** Semgrep/Trivy (chưa đáp ứng hướng scan của Testing) và góc prompt-injection-in-source | Thêm 2 dòng "Gap & Kế hoạch" (đúng cách ghi gap mà ADR-0003 cho phép) |
| `DEF-S3-M11` | D04 | Detailed/drawio | Thiếu **4 image ECR pre-baked + image digest** và đường `GitHub Actions → ECR → SSM` (phải vẽ **nét đứt** — EV-1) | Thêm node ECR + đường giao hàng nét đứt + chú thích "khai báo, chưa đo" |
| `DEF-S3-M12` | A15 | Detailed header/footer | Link `file:///c:/Users/T14S/TI/TestIntelligent_doc/...` trỏ máy cá nhân → hỏng khi share repo (`D:\Doc`) | Đổi sang link tương đối `../Research/...`, `./TI_System_Architecture.drawio` |
| `DEF-S3-M13` | A12 | Detailed §2 `[12D]` + drawio | Poll vẽ **RDS → CI/CD thẳng** (bỏ qua API); thực tế CI gọi `GET /v2/artifact-jobs/{id}` → API → RDS | Vẽ `CI → API v2 → RDS` (đúng hợp đồng — EV-1/E06) |
| `DEF-S3-M14` | F02 | Detailed §2 `[02]` | "WAF kiểm tra **chữ ký HMAC**" — WAF managed rules **không verify HMAC** (chỉ OWASP/rate limit); verify HMAC thuộc API/Lambda@Edge | Sửa: "WAF (OWASP rules + rate limit); xác thực HMAC tại TI API v2 / Lambda@Edge" |
| `DEF-X-M15` | D08, A10 | Cả 2 tài liệu | Claim về AgentCore Harness **không kèm receipt/ngày**: nên trích "AgentCore Runtime 18 — receipt 14/09 (`fbdd8dfc`/`efabf57dcd8c`) — mốc lịch sử" hoặc ghi "theo lượt đo <ngày>" | Thêm block Tham chiếu nguồn (receipt + ngày) theo A10 |
| `DEF-S3-M16` | E10 | drawio `s10_mem` / Detailed `[12C]` | Memory: EV-1 ghi "**Memory reuse chưa được nghiệm thu**" — node chỉ ghi "GOLDEN only", 1 cạnh nét đứt, **không nhãn** `chưa nghiệm thu / UNVERIFIED` | Thêm nhãn trạng thái node + giải thích nét đứt trong legend |

> **Ghi chú A04:** cả 2 tài liệu **không** dùng từ "đã nghiệm thu" → Pass. Tuy Blueprint header `🟢 APPROVED / MASTER BLUEPRINT` nên kèm 1 dòng phụ "*= duyệt tài liệu, ≠ nghiệm thu hệ thống (cổng theo dõi còn mở)*" — gộp sửa cùng DEF-X-002.

---

## 7. BẢNG CHẤM CHI TIẾT (ADR-0003 — ✅ Pass / ❌ Fail)

### 7.1. Checklist A — Chung (cả 3 đối tượng)

| ID | Kết luận | ID | Kết luận | ID | Kết luận |
| :--: | :--- | :--: | :--- | :--: | :--- |
| A01 | ❌ (M01) | A06 | ❌ (001) | A11 | ❌ (005/006 + M03/M04) |
| A02 | ❌ (002) | A07 | ✅ 2 account đúng ranh giới | A12 | ❌ (003 + M05/M13) |
| A03 | ❌ (002 — không legend) | A08 | ❌ (M16 — Memory chưa ghi trạng thái) | A13 | ❌ (003/005) |
| A04 | ✅ (kèm ghi chú M) | A09 | ✅ route `/v2/artifact-jobs` khớp EV-1 | A14 | ❌ (010) |
| A05 | ❌ (M02/M03) | A10 | ✅ (không trích commit — kèm M15) | A15 | ✅ có file nguồn .drawio/.md |

### 7.2. Checklist B — Master Blueprint (S1): 11 Pass / 4 Fail

| Pass | Fail |
| :--- | :--- |
| B01 (dispatch ngang hàng, không lồng Fargate) · B02 (lease/heartbeat) · B03 (direct-to-S3) · B04 (Fargate + IsolatedRunner) · B05 (D5a W1/D5b W3/D2.b/D3/D4 đúng wave) · B06 (DynamoDB TTL) · B07 (D9 wording đúng ở Blueprint) · B08 (ToolIntent sạch) · B09 (`ImpactSet ∩ TargetBinding`) · B12 (Claude = model Bedrock) · B14 (FinOps có nhãn) | **B10** (thiếu cảnh báo AWS ban/self-DoS — M08) · **B11** (thiếu ghi chú tham vấn Team Data — M09) · **B13** (thiếu exit note/CANDIDATE per quyết định — 002) · **B15** (nhất quán với sơ đồ 22/09: gắn `[OBSERVED]` vượt evidence — 002) |

### 7.3. Checklist D — Detailed + drawio/PNG (S3): 6 Pass / 9 Fail

| Pass | Fail |
| :--- | :--- |
| D01 (tên service + region) · D09 (không credential trong prompt/ToolIntent) · D10 (normalize + SHA-256 → S3, không quay lại EC2) · D11 (DB hai nhánh SQL/NoSQL) · D12 (không claim sai về backup) · D13 (Detailed/drawio không có CodeGuru) | **D02** (002/M06 — không đối chiếu hiện trạng EC2) · **D03** (M07 — nơi lưu script) · **D04** (M11 — ECR pre-baked) · **D05** (008 — CodeBuild) · **D06** (011/M10 — gap scan) · **D07** (M03 — Bedrock chưa chẻ đủ) · **D08** (M15 — thiếu receipt) · **D14** (010 — số không nhãn/nguồn) · **D15** (002/M06 — lệch sơ đồ hạ tầng public) |

### 7.4. Checklist C — Mục workflow áp dụng: 0 Pass / 5 Fail (chấm theo Detailed; riêng Blueprint **Pass C13** vì sequence có `opt HOLD→QA`)

| ID | Kết luận | Lý do |
| :--: | :---: | :--- |
| C04 | ❌ | Detailed không diễn đạt `completed ≠ PASS`; `[12B]` gộp state + gate |
| C06 | ❌ | Thiếu hard-stop DO_NOT_PASS (secret lộ/hash mismatch/rollback fail) |
| C07 | ❌ | Có ngưỡng Faithfulness nhưng **chưa gán `< 0.85 → HOLD`** |
| C08 | ❌ | Có temp 0.0 nhưng **thiếu tolerance ±0.03**; thiếu 6 chỉ số |
| C13 | ❌ | Detailed không có đường HOLD → Waiver/Human review (Blueprint có → A11) |

### 7.5. Checklist E — Evidence: 9 Pass / 1 Fail

| Kết luận | Chi tiết |
| :--- | :--- |
| ✅ E01, E02 | Đã truy cập `/diagrams` + `/evidence` — Evidence Pack EV-1/EV-2 (§2) |
| ✅ E03–E05 | 2 tài liệu **không claim** commit/receipt nào → không mâu thuẫn (nhưng nên thêm trích dẫn — M15) |
| ✅ E06 | Hợp đồng `POST /v2/artifact-jobs`, `/v1` FROZEN khớp EV-1 |
| ✅ E07, E08 | Không claim HTTP 200/422/401 → ngoài phạm vi claim của 2 tài liệu |
| ✅ E09 | Không có câu "đã nghiệm thu" (xem ghi chú A04) |
| ❌ **E10** | **M16** — Memory reuse "chưa nghiệm thu" (EV-1) nhưng sơ đồ không gắn nhãn UNVERIFIED/chưa nghiệm thu |

### 7.6. Checklist F — 10 điểm nóng Biên bản 23/09: 4 Pass / 6 Fail

| ID | Kết luận | ID | Kết luận |
| :--: | :--- | :--: | :--- |
| F01 Chi phí "hơi ảo" | ❌ (010) | F06 Orchestrator vs Tenant Binding | ✅ (`ImpactSet ∩ TargetBinding`) |
| F02 Tổng kết đúng từng dòng | ❌ (003/M14 — "khớp 100%", HMAC) | F07 Nơi lưu script K6/Playwright | ❌ (M07) |
| F03 Lý do chọn Fargate | ✅ (Task 4/Biên bản §2 đã chốt, tài liệu nêu rõ) | F08 K6 AWS ban / self-DoS | ❌ (M08) |
| F04 EC2 storage ngộp | ✅ (Direct-to-S3 + WORM 90d) | F09 Scope Semgrep/Trivy + prompt injection | ❌ (011/M10) |
| F05 NoSQL rollback | ✅ (DynamoDB TTL 1h + DeleteTable) | F10 GenAI (6 chỉ số, ±0.03, chẻ Bedrock) | ❌ (009/M03) |

---

## 8. KẾT LUẬN & QUY TRÌNH RE-REVIEW

**Kết luận vòng 1 (Review tĩnh):** 🔴 **CHƯA ĐẠT** cho cả 3 đối tượng (cần 100% Pass theo ADR-0003).

| Hạng mục | Số lượng | Nội dung |
| :--- | :--: | :--- |
| **Blocking** (sửa ≤24h) | **11** | DEF-X-001 → DEF-X-010, DEF-S3-011 |
| **Minor** (sửa ≤3 ngày) | **16** | DEF-*-M01 → M16 |
| Tổng mục Fail (A+B/D+C+E+F) | **25** (Blueprint: A10+B4+C4+E1+F6) / **31** (Detailed+drawio: A10+D9+C5+E1+F6) | Chi tiết §7 · riêng C13 Blueprint Pass (có `opt HOLD→QA`) |

**Thứ tự sửa đề xuất (theo thứ tự thăng trầm rủi ro):**

1. **Đúng nguyên lý TI, sửa trước (T+24h):** DEF-X-001 (CodeGuru EOL — P0) → DEF-X-002 (nhãn `CANDIDATE` + legend — kỷ luật sự thật) → DEF-X-006 (chốt S03/S04 — P0 Đối soát) → DEF-X-007 (Law 13 secrets) → DEF-X-009 (Gate Law 18/Task 4 §6).
2. **Mâu thuẫn kỹ thuật nội bộ (T+24h):** DEF-X-003, DEF-X-004, DEF-X-005, DEF-X-008, DEF-X-010, DEF-S3-011.
3. **Góp ý hoàn thiện (T+3 ngày):** 16 mục Minor — sửa đợt, gộp re-review.

**Luồng re-review (Task_5 §4.3):** người vẽ sửa → tester chỉ chấm lại các mục từng Fail + rà nhanh Checklist A → nếu 100% Pass → chuyển **Vòng 2 Evidence** (Checklist E đầy đủ với Evidence Pack) → Nghĩa (Tester #4) ký **ĐẠT/CHƯA ĐẠT** ở mẫu §8.3. Escalation: 2 lần Fail cùng mục → họp Hoàng/Hùng/Trang + Nghĩa; chưa đồng ý → Tan.Thai (đặc biệt DEF-X-001/006 — Đối soát P0).

**Đề xuất quy ước đặt lại tên file drawio PNG:** `TI_System_Architecture_v2.0_CANDIDATE_2026-09-27.png` để chính tên file đã mang nhãn sự thật (giống mẫu portal "ĐỀ XUẤT · NOT_DEPLOYED").

---

## 9. GHI CHÚ TRUNG THỰC (Task_5 §9)

> Đạt phiếu góp ý này = *bản vẽ + 2 tài liệu đáng tin làm căn cứ thiết kế*. **Không** đồng nghĩa hệ thống đã được kiểm thử hay đã nghiệm thu — portal `/evidence` ghi rõ "bằng chứng ≠ phê duyệt", và các cổng qualification vẫn còn hàng `PENDING/BLOCKED/UNAPPROVED`. Đúng tinh thần Law 18: *bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy.*

---

## CHANGELOG

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 27/09/2026 | v1.0 | AI reviewer (tổng hợp theo Task_5) | Review tĩnh 2 tài liệu + drawio/PNG theo Checklist A–F; 2 Evidence Pack EV-1/EV-2 từ portal live; 11 Blocking + 16 Minor; chấm A(5/15) · B(11/15) · D(6/15) · C(0/5) · E(9/10) · F(4/10) — chờ Nghĩa (Tester #4) chốt Pass/Fail cuối |
