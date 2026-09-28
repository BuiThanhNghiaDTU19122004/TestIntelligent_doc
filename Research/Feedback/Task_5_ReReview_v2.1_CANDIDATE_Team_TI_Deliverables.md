# PHIẾU RE-REVIEW (VÒNG 1 SAU GÓP Ý) — BỘ HỒ SƠ TI v2.1 CANDIDATE CỦA TEAM

> **Đối tượng re-review (5 hồ sơ mới nộp):**
> 1. `D:\Doc\diagram\Hung\TI_Workflow_Hungdz.md` (mtime 28/09/2026 08:46)
> 2. `D:\Doc\diagram\Hung\Workflow_Hung.pdf` (mtime 27/09/2026 19:42 — **bản cũ, chưa vẽ lại**)
> 3. `D:\Doc\diagram\images\TI_Master_Architecture.drawio` (+ 4 tab: `TI_3Tier_Architecture`, `TI_C4_L2_Topology`, `TI_C4_L3_JobController`, `TI_Sequence_Lifecycle` — mtime 28/09/2026 08:50)
> 4. `D:\Doc\diagram\TI_Detailed_System_Architecture.md` (mtime **27/09/2026 19:42** — *trước* phiếu góp ý 20:04)
> 5. `D:\Doc\diagram\TI_System_Architecture_v2.1_CANDIDATE.drawio` + `.drawio.png` (mtime 28/09/2026 09:04 / 08:33)
>
> **Chuẩn chấm:** `Task_5_Shift_Left_Verification_Plan_and_Checklist.md` (Checklist A–F, Pass/Fail nhị phân, cần 100% Pass) · **Phiếu góp ý gốc:** `Task_5_ShiftLeft_Review_Feedback_TI_Architecture_Diagrams.md` (11 Blocking + 16 Minor) · **Phương pháp:** ADR-0001 (2 vòng) · **Nguồn sự thật:** ADR-0002 · **Thang:** ADR-0003.
> **Ngày re-review:** 28/09/2026 · **Người chấm:** Cline (AI reviewer) — **chốt Pass/Fail cuối vẫn thuộc Nghĩa (Tester #4)**.
>
> ## ⛔ KẾT LUẬN NHANH: **CHƯA ĐẠT** — chưa đủ điều kiện chuyển Vòng 2 (Evidence).
> - **Blocking:** 11/11 vẫn **OPEN** dưới dạng "chưa đóng ở ≥1 hồ sơ" → 4 mục **ĐÃ SỬA** trong `v2.1 CANDIDATE.drawio`, nhưng **3 hồ sơ còn lại chưa được cập nhật** nên tổ hợp chưa đạt.
> - **Regression mới (P0):** chữ **CodeGuru (EOL 20/11/2025)** xuất hiện trở lại ở **2 hồ sơ mới/vừa rà** (`Hung/TI_Workflow_Hungdz.md`, `Workflow_Hung.pdf`) và **vẫn còn** ở `TI_Master_Architecture_Blueprint.md` §4.3.
> - **Mismatch mới:** 10 điểm (M-01 → M-10, §5) — nặng nhất là **S03/S04 chạy ở Account A hay Account B** (2 hồ sơ nói A, 3 hồ sơ nói B) và **số runner điều phối 5 hay 6**.

---

## 1. NGUỒN ĐỐI CHIẾU ĐÃ DÙNG (EVIDENCE PACK — ADR-0002)

| # | Nguồn | Cách lấy | Giá trị dùng để chấm |
| :--: | :--- | :--- | :--- |
| EV-1 | Portal live `/diagrams`, `/evidence` (nguồn sự thật ưu tiên 1) | Theo phiếu góp ý gốc (27/09/2026) — **chưa gọi lại trong lượt này** | Hiện trạng: EC2 1 máy (`API :8000` + `Portal :8001`), evidence chưa sang S3, Memory reuse chưa nghiệm thu, #97 còn mở |
| EV-2 | File nguồn chỉnh sửa được | Đọc trực tiếp XML `.drawio` (không chỉ ảnh PNG) | Xác minh **từng** nhãn/nút/cạnh theo line-number |
| EV-3 | `Workflow_Hung.pdf` | Trích text bằng `pypdf 6.19.0` (1 trang, 2.479 ký tự) | 32 nhãn node + 32 bước tương tác của Hùng |
| EV-4 | `Biên bản cuộc họp TI - 23_09_2026.pdf` | Trích text bằng `pypdf` (2 trang, 3.253 ký tự) | 10 điểm nóng F01–F10 + phân công RACI (Hoàng/Hùng/Trang/Nghĩa) |
| EV-5 | Git history | `git log --oneline` | Xác định hồ sơ nào **thực sự** được sửa sau phiếu góp ý |

**Bằng chứng then chốt về "ai đã sửa":** commit `f714085` (*"doc: update architecture diagram version 2.1"*) và `4d3d4c8` (*"updated diagram to nice"*) → **chỉ nhóm ảnh/drawio của Hoàng (images/*) và file `v2.1 CANDIDATE` được ghi mới**. `TI_Detailed_System_Architecture.md` giữ nguyên mtime **27/09 19:42** (trước phiếu góp ý 20:04) ⇒ **tài liệu này chưa nhận một dòng sửa nào**. `TI_Master_Architecture_Blueprint.md` **đã bị sửa ngay giữa lúc tôi đang review** (mtime `28/09 09:08:54`, khi tôi bắt đầu đọc là `08:50`) nhưng bản sửa **không chạm bất kỳ lỗi nào đã bị chấm Fail**: vẫn `CodeGuru` ở §4.3 (**L400** — số dòng đổi từ 402), vẫn `🟢 APPROVED / MASTER BLUEPRINT` (L13) không kèm dòng *"duyệt tài liệu ≠ nghiệm thu hệ thống"*, vẫn `[OBSERVED]` cả 2 zone (L165/L184), vẫn chỉ 5 runner. Thay đổi duy nhất: **Model Tiering 3 cấp → "Dual-Model Tiering 2 cấp (Hợp nhất Sonnet 5 + Opus 5, Loại bỏ Haiku 4.5)"** (L128 + sơ đồ L188) — và điều này lại **đá nhau với chính bảng FinOps cùng file** (L478 vẫn ghi *"Claude Model Tiering (Haiku/Sonnet/Opus)"*) ⇒ xem **M-02**.

---

## 2. TRẠNG THÁI TỪNG HỒ SƠ

| # | Hồ sơ | Có cập nhật sau góp ý? | Nhận xét 1 dòng |
| :--: | :--- | :--: | :--- |
| 1 | `Hung/TI_Workflow_Hungdz.md` | ⚠️ Có (28/09 08:46) nhưng **nội dung vẫn lỗi cũ** | Vẫn `CodeGuru`; thiếu API runner; không nhãn/legend; S03/S04 đặt Account B |
| 2 | `Hung/Workflow_Hung.pdf` | ❌ **Không** (27/09 19:42) | Node **"Security (CodeGuru/Semgrep)"**; SAST chạy 2 lần (bước 7 & 19); 5 runner; `COMPLETED (Kèm Gate)` |
| 3 | `images/TI_Master_Architecture.drawio` (+4 tab) | ✅ Có (28/09 08:50) | **Đã bổ sung** Haiku 4.5 + AgentCore Gateway + legend bar + sandbox CANDIDATE; **còn** `[OBSERVED]` ở cả 2 zone; **5 runner** (thiếu API) |
| 4 | `TI_Detailed_System_Architecture.md` | ❌ **Không** | Chưa nhận dòng sửa nào ⇒ giữ nguyên các Blocking/Minor thuộc hồ sơ này (DEF-X-004/005/006/007/008/009/010, DEF-S3-011 + M02/M07/M08/M09/M10/M12/M13/M14/M15/M16) |
| 5 | `TI_System_Architecture_v2.1_CANDIDATE.drawio(.png)` | ✅ Có (28/09 08:33–09:04) | **Hồ sơ tiến bộ nhất**: 4 Blocking đã sửa thật; còn 7 Blocking dạng "chưa đủ" |

### 2.1. `v2.1 CANDIDATE.drawio` — những gì ĐÃ SỬA (ghi nhận tích cực)

| Hạng mục | Bằng chứng (line trong `TI_System_Architecture_v2.1_CANDIDATE.drawio`) |
| :--- | :--- |
| Bỏ `--network none` → câu chữ kỹ thuật đúng | L164: `Sandbox Execution VPC … Private Subnet không IGW/NAT · SG DENY ALL EGRESS · VPC Endpoints: ECR+Logs+STS ~$22/mo · S3 Gateway $0` |
| VPC Endpoints về đúng Sandbox | L167–L171 (icon `mxgraph.aws4.endpoints` + text trong swimlane `sandbox`) |
| Secrets Manager về **Account A** | L110: `Secrets Manager / Tenant Secrets & STS / (Account A — Law 13)` |
| Bỏ mọi dấu vết CodeBuild | L43: `ECS/Fargate State Machine Law 4.3`; L173: `ECS RunTask / IsolatedRunner / Law 23` |
| Gate tách khỏi state | L61: node `Decision Gate PASS/HOLD/DO_NOT_PASS`; L73 `[12A] Metrics+SHA256`; L76 `[12B] Update state` (đi vào RDS, không gộp chuỗi `COMPLETED`) |
| Thêm **legend + nhãn sự thật** | L10: legend + `⚠ Đây là THIẾT KẾ ĐÍCH … CANDIDATE, CHƯA TRIỂN KHAI` + `Hiện trạng (OBSERVED, đo 22/09): API :8000 + Portal :8001 trên 1 EC2` |
| Nhãn chuỗi `[01]…[12D]` đầy đủ | `lbl_01` L451, `lbl_02` L454, `lbl_05` L124, `[03]` L306, `[04]` L93, `[06]` L317, `[07A]` L292, `lbl_07B` L457, `[08A]` L429, `[08B]` L150, `[09A]` L155, `[09B]` L334, `[09C]` L345, `lbl_10` L460, `lbl_11B` L463, `[12A]` L73, `[12B]` L76, `[12C]` L113, `[12D]` L402 |
| Thêm node **CI/CD Pipeline** trong External | L22 (`mxgraph.ios7.icons.settings` + label `CI/CD Pipeline`) |
| D5b DAST tách Wave 3 | L242 `Trục 6: D5b DAST Task (W3)`; L274 cạnh `dashed` nhãn `Wave 3` |
| Ghi chú vòng đời Trục 1 | L303: `Trục 1 (D5a): chạy SỚM ở [06] (pre-scan S02→S04) / [10] Smart Dispatch khởi tạo Trục 2–Trục 6`; **không còn** `e_pl_r1` (grep = 0 hit) |

---

## 3. ĐỐI CHIẾU 11 DEFECT **BLOCKING** (SỬA ≤ 24H)

> Ký hiệu: ✅ **FIXED** · 🟡 **PARTIAL** (đúng chỗ này, thiếu chỗ khác) · ❌ **OPEN** · 🔴 **REGRESSION** (lỗi đã bị bác nay quay lại)

| ID | Nội dung | v2.1 CANDIDATE | images/*.drawio | Detailed .md | Hung .md/.pdf | Trạng thái tổng |
| :-- | :--- | :--: | :--: | :--: | :--: | :--: |
| DEF-X-001 | CodeGuru Security (EOL) | ✅ | ✅ | ✅ | 🔴 **CÓ** | 🔴 **REGRESSION** |
| DEF-X-002 | Thiếu nhãn CANDIDATE + legend | 🟡 (có legend nhưng sai định nghĩa nét đứt; ghi "v2.0") | 🟡 (có legend, 2 zone vẫn `[OBSERVED]`) | ❌ | ❌ | 🟡 PARTIAL |
| DEF-X-003 | Thiếu nhãn `[01][02][05]` + node CI/CD | ✅ **FIXED** | 🟡 (hệ số "1..10") | ❌ (vẫn claim "khớp 100%") | 🟡 (hệ số 1–32) | 🟡 PARTIAL |
| DEF-X-004 | `--network none` | ✅ **FIXED** (L164) | 🟡 (file `TI_System_Architecture.drawio` cũ vẫn còn L163) | ❌ (L50, L136, L140) | 🟡 (`NO-INTERNET`) | 🟡 PARTIAL |
| DEF-X-005 | VPC Endpoints sai chỗ | ✅ **FIXED** | 🟡 (`vpce_node` đúng sandbox) | ❌ (§1 Account A ↔ §3 Sandbox) | ❌ (không nêu) | 🟡 PARTIAL |
| DEF-X-006 | S03/S04 chạy ở đâu (P0) | ❌ (ngầm Account A, **không có node S03/S04**) | ❌ (Account B — `msg6`) | ❌ (Account A — `[07B]`) | ❌ (Account B — Phase 2) | ❌ **OPEN (P0)** |
| DEF-X-007 | Secrets Manager ở Account B | ✅ **FIXED** (L110) | 🟡 (không có node Secrets) | ❌ (§1 L55 vẫn trong Account B) | 🟡 (chỉ nói STS) | 🟡 PARTIAL |
| DEF-X-008 | "CodeBuild/Fargate" | ✅ **FIXED** | ✅ | ❌ (L463) | ✅ (chỉ Fargate) | 🟡 PARTIAL |
| DEF-X-009 | Gate S09 thiếu nhánh | 🟡 (có node gate, tách `[12A]/[12B]`) | 🟡 (có `opt HOLD→Waiver`) | ❌ (gộp state+gate) | 🟡 (có Waiver, gộp COMPLETED) | 🟡 PARTIAL |
| DEF-X-010 | FinOps không nhãn / số "ảo" | 🟡 (`~$22/mo` không nhãn) | 🟡 (Blueprint §6 có nhãn ✔) | ❌ (vẫn `$0.0008`, không cột nhãn) | ✅ | ❌ OPEN |
| DEF-S3-011 | Trục 1 dispatch 2 lần + gap scan | 🟡 (chốt hướng (a) ✔ nhưng **xung đột Task 4 §3.1.1**; thiếu footnote gap) | ❌ (`t_sast` là runner duy nhất) | ❌ (`[06]` + `e_pl_r1`) | ❌ (SAST chạy 2 lần: PDF bước 7 & 19) | ❌ **OPEN** |

### 3.1. Việc cần làm chính xác cho từng Blocking

**DEF-X-001 · CodeGuru (P0 — sửa TRƯỚC tiên)**
- Bằng chứng: `Hung/TI_Workflow_Hungdz.md` L28 (*"…SAST như Semgrep và **CodeGuru**"*); `Workflow_Hung.pdf` node `Security (CodeGuru/Semgrep)`; `TI_Master_Architecture_Blueprint.md` **L400** (*"**Amazon CodeGuru Security**: Phân tích AST diff của PR bằng Machine Learning"*) — lưu ý bản sửa 09:08 của Blueprint **không chạm dòng này**.
- Sửa: xóa CodeGuru ở **3 vị trí**; dùng `Amazon Inspector (ECR image scan) + Semgrep/Trivy/Gitleaks`; nếu nhắc lịch sử phải kèm `(EOL 20/11/2025 — KHÔNG dùng trong TI)`.
- Rủi ro: hồ sơ workflow mới nhất đang "dạy" team dùng dịch vụ không còn tồn tại → Fail Checklist A06 (P0 Đối soát §3.1).

**DEF-X-002 · Nhãn sự thật + legend**
- v2.1 làm tốt banner CANDIDATE + ghi chú hiện trạng 22/09 (cần **nhân bản** sang 3 hồ sơ còn lại).
- Lỗi còn: L10 định nghĩa *"Nét đứt = phản hồi / bằng chứng gián tiếp / điều kiện kích hoạt"* — **trái quy ước** portal + `GLOSSARY_TI.md` §A (nét liền = đã đo; nét đứt = khai báo/chưa đo) theo Checklist A03; legend ghi *"THIẾT KẾ ĐÍCH **v2.0**"* trong khi tiêu đề là **v2.1**.
- `images/*.drawio` L120+L184 (master) / L36+L85 (C4 L2) vẫn dán `[OBSERVED]` cho **cả Account A và B** → đúng lỗi **B15 "claim vượt evidence"** đã bị Fail.
- Sửa: (1) đổi định nghĩa nét đứt theo GLOSSARY; (2) "v2.0" → "v2.1"; (3) hạ `[OBSERVED]` ở 2 zone A/B; (4) thêm banner CANDIDATE vào `Detailed .md` + `Hung .md`.

**DEF-X-006 · Chốt S03/S04 (P0 — escalate Tan.Thai nếu lần 2 vẫn không thống nhất)**
- **Nói Account A (Job Controller):** `v2.1` (nhãn `[07B] ImpactSet + RiskTier` phát ra từ khu vực Trục 1 → Job Controller) và `Detailed .md` §[07B] (*"Trục 1 gửi … về Job Controller để S03 Impact Engine … S04 Risk Engine"*).
- **Nói Account B (Harness):** `Blueprint .md` L346–347 (*"Harness->>Harness: S03 Impact & S04 Risk Tiering"*), `images/TI_Sequence_Lifecycle.drawio` L113 (`msg6`), `Hung .md` Phase 2 (*"S03 & S04 … do Account B đảm nhiệm"*), `Workflow_Hung.pdf` (bước 10–11 nằm trong lane Account B sau `InvokeHarness`).
- Hệ quả: không chốt được ma trận `Changeset → Runner` (Đối soát §7 — P0). **Phải chốt đúng 1 câu** (đề xuất: *"S03/S04 là deterministic → chạy tại Job Controller (Account A); Harness chỉ trả gợi ý ImpactSet"*) rồi sửa đồng bộ **5 hồ sơ**.
- Kèm: `v2.1` **không có node S03/S04** nào (chỉ có nhãn `[07B]` là text rời) → truy vết A13 không đạt ⇒ cần thêm node `S03 Impact Engine` / `S04 Risk Engine` vào đúng vùng và đúng account.

**DEF-X-009 · Gate S09 — 5 điểm còn thiếu (C04, C06, C07, C08, C13, F10)**
1. **Hard-stop `DO_NOT_PASS`** chưa định nghĩa: `Critical > 0`, `SECRETS_LEAKED > 0`, SHA-256 mismatch, rollback fail (Task 4 §6.2) — hiện chỉ có tên trạng thái trên node.
2. **`Faithfulness < 0.85 → HOLD`** chưa xuất hiện ở bất kỳ sơ đồ nào (C07 — P1 Đối soát §3.9).
3. **Nhánh HOLD → Waiver / Human review:** `v2.1` grep `Waiver` = **0 hit**, không có actor `QA Lead`; `Detailed .md` cũng không có. Chỉ `images/TI_Sequence_Lifecycle.drawio` L284–289 + `Blueprint .md` L373–376 có `opt HOLD→Waiver` (Hùng đã tham chiếu đúng đoạn này — cần copy vào v2.1 + Detailed).
4. **Tolerance ±0.03 + bộ 6 chỉ số GenAI:** grep `0.03` trên **toàn bộ** `*.drawio` = **0 hit**.
5. **`completed ≠ PASS` (Law 18):** `v2.1` **không có chữ `COMPLETED`**; `Detailed .md` §[12B] gộp *"Cập nhật `COMPLETED` + PASS/HOLD/DO_NOT_PASS"*; `Workflow_Hung.pdf` bước 27 *"Cập nhật Job State: COMPLETED (Kèm Gate)"*; `images` sequence bước 18 y như vậy.
- Sửa: thêm **hộp 3 nhánh Gate** (PASS / HOLD+Waiver / DO_NOT_PASS hard-stop) + tách `state` ↔ `gate_result` + dòng `completed ≠ PASS (Law 18)` + dòng "khuyến nghị ≠ phê duyệt" trên **cả 4 sơ đồ**.

**DEF-S3-011 · Trục 1 chạy mấy lần + gap scan**
- v2.1 đã chốt hướng (a) và ghi rõ tại L303 — điểm cộng. Nhưng hướng (a) **xung đột Task 4 §3.1.1**: Task 4 xếp Semgrep+Trivy+Gitleaks là **S07 runner chính thức**, còn chặng `S02→S04` theo Task 4 là **CodeGuru + Inspector** — mà CodeGuru đã bị loại (P0) ⇒ **"cái gì chạy pre-scan S02→S04" đang trống**.
- Sửa (chọn 1): (a1) giữ Trục 1 = S07 runner và tách "pre-scan nhanh" thành bước riêng có tên công cụ riêng; **hoặc** (a2) chốt *"pre-scan dùng cùng image D5a, ruleset rút gọn, chạy **đúng 1 lần**"* và **cập nhật Task 4 §3.1.1** cho khớp. Không được để 2 tài liệu nói 2 luật.
- Kèm **footnote gap bắt buộc** (đang thiếu ở mọi hồ sơ): *"Semgrep/Trivy hiện chỉ scan source code — chưa đáp ứng hướng scan của Testing (Biên bản §3); rủi ro prompt-injection phải nhìn dưới góc quét payload độc hại lọt vào source code (TI không có UI Chat)"*.

### 3.2. Ba hồ sơ chưa sửa gì — nhắc lại mức độ Fail theo Checklist (không đổi so với vòng 1)

| Hồ sơ | Mục Fail giữ nguyên | Ghi chú |
| :--- | :--- | :--- |
| `TI_Detailed_System_Architecture.md` | **A10, D02, D03, D04, D05, D06, D07, D08, D14, D15, C04, C06, C07, C08, C13, E10, F01, F02, F07, F08, F09, F10** | Không nhận dòng sửa nào; vẫn trỏ `file:///c:/Users/T14S/...`, vẫn `--network none` (L50/136/140), vẫn `CodeBuild/Fargate` (L463), vẫn claim *"Bám sát 100%"* (L7), vẫn `HMAC` (L73) |
| `TI_Master_Architecture_Blueprint.md` | **A06 (CodeGuru), B10, B11, B13, B15** | Vẫn `🟢 APPROVED / MASTER BLUEPRINT` (L13) không kèm dòng *"duyệt tài liệu ≠ nghiệm thu hệ thống"*; vẫn `[OBSERVED]` 2 zone |
| `Hung/TI_Workflow_Hungdz.md` + `Workflow_Hung.pdf` | **A01, A02, A03, A05, A06, A11, A12, A13, A15, C02, C04, C06, C07, C08, C11, C12, C14, F05, F07, F09, F10** | Xem §4 để biết mục nào nay đã đỡ hơn (Waiver có, C13 ✅) |


---

## 4. ĐỐI CHIẾU 16 DEFECT **MINOR** (SỬA ≤ 3 NGÀY)

| ID | Nội dung Fail (vòng 1) | Trạng thái nay | Bằng chứng / việc cần làm |
| :-- | :--- | :--: | :--- |
| `DEF-S3-M01` | PNG thiếu ngày tạo/ngày đo/người vẽ/nhóm | 🟡 PARTIAL | v2.1 **đã có** "Hiện trạng (OBSERVED, đo 22/09)" trong legend; nhưng **thiếu ngày vẽ + tác giả + nhóm** và tên file chưa mang nhãn sự thật ⇒ đề nghị `TI_System_Architecture_v2.1_CANDIDATE_2026-09-28.png` + 1 dòng metadata |
| `DEF-X-M02` | Trạng thái job lệch chuẩn (`PENDING/QUEUED` vs portal `queued/running/completed/failed`) | ❌ OPEN | `v2.1` L93 `[04] PENDING`; `Detailed .md` L79/461 `PENDING`; `Workflow_Hung.pdf` bước 3 `QUEUED`; `images` sequence `QUEUED` ⇒ **3 cách viết khác nhau**, chưa map về bộ 4 trạng thái của portal |
| `DEF-X-M03` | Model tiering lệch pha + tên model + thiếu AgentCore Gateway | 🔴 REGRESSION | `images/*` có `AgentCore Gateway (MCP/IAM)` (master L225) ✔ nhưng: `images/*` **vẫn còn node `m_haiku`** (master L202, C4L2 L98) trong khi **bản Blueprint sửa lúc 09:08 đã "Loại bỏ Haiku 4.5"** (L128/L188) mà **bảng FinOps cùng file vẫn ghi Haiku** (L478); `v2.1` vẫn 2 tầng **thiếu Gateway** và dùng tên **"Claude 5.0 Sonnet"** ≠ `Claude Sonnet 5` (Task 3/Glossary) ⇒ xem **M-02** |
| `DEF-X-M04` | 5 runner (thiếu API Functional) vs 6 trục | 🟡 PARTIAL | `v2.1` ✔ 6 trục (r1–r6); `Detailed .md` ✔ 6 trục; **nhưng** `images/*` chỉ 5 (`t_sast`, `t_ui`, `t_perf`, `t_db`, `t_dast` — master L264/271/278/285/292) và `Blueprint §3.1/§3.2` 5 (`T_SAST…T_DAST`, L240–244, L300–304) và `Workflow_Hung.pdf` 5 (bước 19: D5a/D3/D4/D2.b/D5b) ⇒ 3 hồ sơ còn thiếu Trục 2 |
| `DEF-S3-M05` | Nhãn `[11A]/[11B]` sai vùng; cạnh thiếu arrowhead | ❌ OPEN | `v2.1`: `[11B]` vẫn **chỉ phát từ Trục 2** (`e_r2_jc` L392 `value=""`); **5 cạnh →S3 đều `endArrow=none, endFill=0`** (L353, L365, L374, L384, L440) và chỉ `e_r4_s3` (L440) có nhãn `[11A]`; các nhãn `[01][02][07B][10][11B]` là **text rời** không neo vào cạnh ⇒ gộp 1 cạnh S3 + 1 nhãn `[11A]`, thêm arrowhead, gắn nhãn vào cạnh |
| `DEF-S3-M06` | Không đối chiếu hiện trạng EC2 (nguồn ghim `8a61cf66`) | 🟡 PARTIAL | v2.1 legend có 1 dòng hiện trạng ✔ nhưng **thiếu commit ghim `8a61cf66`** và chi tiết 2 container (`/app/data` chỉ gắn API + portal); `Detailed .md` vẫn **không có** footnote hiện trạng |
| `DEF-S3-M07` | Chưa chỉ nơi lưu script k6/Playwright + cách version (F07) | ❌ OPEN | grep `ti-test-packs` toàn repo = **0 hit**; không hồ sơ nào nêu ⇒ thêm 1 dòng vào mỗi sơ đồ: *"Script: repo `ti-test-packs/` (đề xuất — CANDIDATE), version theo git tag"* |
| `DEF-S3-M08` | Thiếu cảnh báo AWS ban / "tự DoS người nhà" + xin phép DevOps (B10, F08) | ❌ OPEN | `v2.1` có "BỘ 3 KHÓA AN TOÀN" (L239) ✔ nhưng **không có** dòng cảnh báo ban / self-DoS / bước xin phép DevOps (grep `AWS ban`/`tự DoS`/`xin phép` = 0 hit) |
| `DEF-X-M09` | Aurora Clone thiếu ghi chú "chờ tham vấn Team Data về quota snapshot" | ❌ OPEN | grep `Team Data` toàn repo = 0 hit; `v2.1` `r4_summary` (L218) thậm chí còn **claim tuyệt đối mới**: *"an toàn 100% DB gốc"* ⇒ thêm footnote `PENDING tham vấn Team Data` + bỏ chữ "100%" |
| `DEF-S3-M10` | Chưa nêu gap Semgrep/Trivy + góc prompt-injection-in-source (F09) | ❌ OPEN | Không hồ sơ nào có (xem DEF-S3-011) |
| `DEF-S3-M11` | Thiếu 4 image ECR pre-baked + digest + đường `GitHub Actions → ECR → SSM` (nét đứt) | ❌ OPEN | grep `pre-baked` = 0 hit; `v2.1` chỉ ghi "VPC Endpoints: ECR+Logs+STS" (L164) — **không có node ECR, không có digest, không có đường giao hàng** ⇒ Fail D04 |
| `DEF-S3-M12` | Link `file:///c:/Users/T14S/...` trỏ máy cá nhân | ❌ OPEN | `Detailed .md` L5, L6, L27… vẫn nguyên ⇒ đổi sang link tương đối `../Research/...`, `./TI_System_Architecture_v2.1_CANDIDATE.drawio` |
| `DEF-S3-M13` | Poll vẽ `RDS → CI/CD` bỏ qua API | ❌ OPEN | `v2.1` L402 `e_rds_ci` source=`rds_pg` → target CI/CD (nhãn `[12D] Poll result`) ⇒ phải vẽ `CI → TI API v2 → RDS` (`GET /v2/artifact-jobs/{id}`) |
| `DEF-S3-M14` | "WAF kiểm tra chữ ký HMAC" | 🟡 PARTIAL | `v2.1` chỉ ghi "WAF" — **tránh được lỗi** ✔; nhưng `Detailed .md` L73 vẫn khẳng định WAF verify HMAC ⇒ sửa thành *"WAF (OWASP rules + rate limit); xác thực HMAC tại TI API v2 / Lambda@Edge"* |
| `DEF-X-M15` | Claim AgentCore Harness thiếu receipt/ngày (D08, A10) | ❌ OPEN | grep `Runtime 18`/`receipt`/`8a61cf66` trong `diagram/**` = 0 hit ⇒ thêm block *"AgentCore Runtime 18 — receipt 14/09 (`fbdd8dfc`/`efabf57dcd8c`) — **mốc lịch sử**"* |
| `DEF-S3-M16` | Memory reuse chưa nghiệm thu nhưng không gắn nhãn `UNVERIFIED` (E10) | ❌ OPEN | `v2.1` `s10_mem` (L147) = `Knowledge Base / S10 Memory / GOLDEN only` — **không có** `UNVERIFIED / chưa nghiệm thu`; `images` `mem_node` = *"Chỉ lưu tri thức đã qua thẩm định GOLDEN"*; `Detailed` §[12C] tương tự; `Hung` = *"S10: Production Learning (Lưu vào Memory - Human-Controlled)"* (còn lệch nghĩa) |

**Tổng kết Minor:** ✅ **0 FIXED** · 🟡 **4 PARTIAL** (M01, M04, M06, M14) · 🔴 **1 REGRESSION** (M03 — tranh chấp Haiku) · ❌ **11 OPEN**.

> **Bổ sung sau khi rà lần cuối (09:11):** hồ sơ `TI_Master_Architecture_Blueprint.md` đã bị **sửa giữa lúc review** (09:08:54) để "loại bỏ Haiku 4.5" mà **không** sửa bảng FinOps cùng file, **không** sửa `images/*`, **không** sửa Task 3/4 ⇒ tạo **xung đột P0 mới (M-02)**. Đây là ví dụ điển hình cho việc cần **một người giữ "single source of truth"** cho phần AI/model thay vì mỗi hồ sơ sửa một kiểu.


---

## 5. MISMATCH **MỚI** PHÁT HIỆN GIỮA CÁC HỒ SƠ (CẦN CHỐT TRƯỚC KHI SỬA TIẾP)

| ID | Chủ đề | Hồ sơ nói A | Hồ sơ nói B | Mức | Việc cần làm |
| :-- | :--- | :--- | :--- | :--: | :--- |
| **M-01** | **S03/S04 chạy ở đâu** | Account A: `v2.1` (`[07B]`), `Detailed .md` | Account B: `Blueprint .md` L346, `images` sequence L113, `Hung .md` Phase 2, `Hung .pdf` bước 10–11 | 🔴 **P0** | Chốt 1 câu + sửa 5 hồ sơ (xem DEF-X-006) |
| **M-02** | **Model tiering & tên model** (⚠️ **phát sinh trong lúc review**) | **2 tầng (bỏ Haiku)**: `Blueprint` bản mới L128 *"Dual-Model Tiering 2 cấp … Loại bỏ Haiku 4.5"* + L188, `v2.1`, `Hung .md` | **3 tầng (còn Haiku 4.5)**: **chính bảng FinOps L478 của Blueprint** (vẫn ghi `Haiku/Sonnet/Opus`), `images/*` (node `m_haiku` master L202 / C4L2 L98), Task 3 + Task 4 (tiering có Haiku 4.5); riêng `v2.1` còn dùng tên *"Claude **5.0** Sonnet"* | 🔴 **P0 mới** | Phải **ra quyết định chính thức**: (i) nếu **bỏ Haiku** → xóa node `m_haiku` ở `images/*`, sửa FinOps L478 + Task 3/Task 4; (ii) nếu **giữ Haiku** → revert L128/L188. Không được để 1 file tự mâu thuẫn (L128 ↔ L478). Đồng thời thống nhất tên `Claude Sonnet 5` (Glossary A05) |
| **M-03** | **Số runner điều phối trong 1 job** | 6 trục: `v2.1` (r1–r6), `Detailed .md` | 5 runner: `images/*` (thiếu Trục 2 API), `Blueprint §3.1/§3.2`, `Hung .pdf` bước 19 | 🟠 P1 | Bổ sung runner **API Functional & Fuzzing** vào images/Blueprint/Hùng |
| **M-04** | **Trạng thái khởi tạo job** | `PENDING`: `v2.1` L93, `Detailed .md` L79 | `QUEUED`: `Hung .md` L23, `Hung .pdf` bước 3, `images` sequence | 🟡 P2 | Map về `queued/running/completed/failed` (portal) và dùng **một** từ duy nhất |
| **M-05** | **Lease TTL / nhịp heartbeat** | **5 phút**: `Hung .md` L44 (*"mỗi 5 phút (Lease TTL)"*), `Hung .pdf` bước 18, `Blueprint` L352 | **60 giây**: `images` sequence (`loop [Heartbeat định kỳ 60s]`) | 🟠 P1 | Chốt 1 con số (đề xuất TTL 5 phút + heartbeat 60s **phải ghi rõ là 2 tham số khác nhau**) |
| **M-06** | **Điều kiện chạy D5b DAST** | **W3 + chỉ khi có Staging URL sống**: `v2.1` (L242/L251/L274), `Blueprint §4.3`, `images` (`t_dast` W3) | **Không điều kiện** (chạy trong `par` như 5 trục khác): `Hung .md` L43, `Hung .pdf` bước 19 | 🟠 P1 | Sửa Hùng: `D5b` phải là nhánh nét đứt có điều kiện *"W3 & Staging URL sống"* (Checklist B05/F05) |
| **M-07** | **Câu chữ cô lập mạng** | `Private Subnet không IGW/NAT + SG DENY ALL + VPC Endpoints`: `v2.1`, `Blueprint §4.4` | `NO-INTERNET` (Hùng), `--network none` (`Detailed .md`, `TI_System_Architecture.drawio` L163) | 🟠 P1 | Dùng **một** câu chuẩn của v2.1 ở mọi hồ sơ; xóa/nhãn "superseded" cho file drawio cũ |
| **M-08** | **Ai là pre-runner S02→S04 sau khi CodeGuru bị loại** | `v2.1`: Trục 1 (D5a) chạy sớm (pre-scan) | `Task 4 §3.1.1`: D5a là **S07 runner**; pre-runner S02→S04 là CodeGuru+Inspector (đã EOL) | 🔴 **P0** | Xem DEF-S3-011: chốt lại giữa Hùng (Task 4) + Hoàng + Trang, cập nhật Task 4 cho khớp |
| **M-09** | **Trạng thái Memory (S10)** | *"GOLDEN only"*: `v2.1`, `Detailed .md` | *"đã qua thẩm định GOLDEN"* (`images`); *"Human-Controlled"* (`Hung`); **"Memory reuse chưa được nghiệm thu"** (portal EV-1) | 🟡 P2 | Thêm nhãn `UNVERIFIED — chưa nghiệm thu` cho mọi node Memory + giải thích trong legend (E10) |
| **M-10** | **Rác hồ sơ cũ chưa khai tử** | `v2.1` là bản mới | `TI_System_Architecture.drawio` (cũ) **vẫn còn trong repo** và `Detailed .md` **vẫn trỏ vào bản cũ** này | 🟠 P1 | Ghi `_(SUPERSEDED — dùng v2.1)_` vào file cũ + sửa mọi link trong `Detailed .md` sang v2.1 |

### 5.1. Ba "claim tuyệt đối" cần quét lại (Checklist F02 — "đọc lại từng dòng")

| Vị trí | Câu chữ đang có | Vấn đề |
| :--- | :--- | :--- |
| `v2.1` L218 (`r4_summary`) | *"Hook dọn sạch sau test, **an toàn 100% DB gốc**"* | Claim tuyệt đối không evidence → đổi thành *"không ghi vào DB gốc (clone bị hủy sau test)"* |
| `Detailed .md` L7 + tiêu đề §4 | *"Bám sát **100%**…/ **Đồng bộ 100%** với sơ đồ Draw.io"* | Claim tuyệt đối, lại trỏ bản drawio **cũ** → đổi thành *"khớp theo bản v2.1 ngày …; các điểm lệch liệt kê ở §…"* |
| `v2.1` legend (L10) | *"Nét liền = luồng dữ liệu chính (CANDIDATE — thiết kế đã chốt)"* | Nét liền bị dùng cho cả phần **chưa triển khai** → trái A03; phải ghi đúng: nét liền = đã đo (OBSERVED), nét đứt = CANDIDATE/khai báo |


---

## 6. YÊU CẦU SỬA / BỔ SUNG — CHIA THEO CHỦ SỞ HỮU

### 6.1. 🔴 Blocking (phải xong trước re-review lần 2 — T+24h)

| # | Việc | Owner | Ghi chú kỹ thuật |
| :--: | :--- | :--- | :--- |
| 1 | Xóa `CodeGuru` ở `Hung .md` L28, node PDF `Security (CodeGuru/Semgrep)`, `Blueprint .md` **L400** | Hùng + Hoàng | Thay bằng `Inspector (ECR image scan) + Semgrep/Trivy/Gitleaks` — **A06** |
| 1b | **Chốt dứt khoát Haiku 4.5 còn hay bỏ** rồi đồng bộ: `Blueprint` L128/L188 ↔ FinOps L478 ↔ `images/*` (`m_haiku`) ↔ Task 3/4 | Hoàng + Nghĩa | **M-02 (P0 mới)** — hiện 1 file tự mâu thuẫn |
| 2 | Chốt **S03/S04** + sửa đồng bộ 5 hồ sơ + thêm node S03/S04 vào v2.1 | Hoàng + Nghĩa → escalate Tan.Thai | P0 Đối soát §7 — **DEF-X-006 / M-01** |
| 3 | Chốt lại **ai chạy pre-scan S02→S04** sau khi bỏ CodeGuru; cập nhật Task 4 §3.1.1 cho khớp v2.1 | Hùng + Hoàng + Trang | **DEF-S3-011 / M-08** |
| 4 | Bổ sung **hộp 3 nhánh Gate** (PASS / HOLD+Waiver / DO_NOT_PASS hard-stop) + `Faithfulness<0.85→HOLD` + `±0.03` + 6 chỉ số + tách `state` ↔ `gate_result` + `completed ≠ PASS` | Hoàng (v2.1, images) + Hùng (PDF) + Trang (Detailed) | **DEF-X-009 / C04,C06,C07,C08,C13,F10** |
| 5 | Thêm nhánh **HOLD → QA Lead/Authority Waiver** vào v2.1 + Detailed (copy từ Sequence hiện có) | Hoàng + Trang | C13 — "Ba ranh giới" |
| 6 | Cập nhật `Detailed .md`: bỏ `--network none` (L50/136/140), bỏ `CodeBuild/Fargate` (L463), sửa `HMAC` (L73), bỏ `e_pl_r1`, tách state/gate §[12B], thêm banner CANDIDATE | Trang | **DEF-X-004/005/006/008/009 + M14** |
| 7 | Đánh dấu `TI_System_Architecture.drawio` (bản cũ còn `--network none` L163, `e_pl_r1`, `PENDING`) là **SUPERSEDED** và sửa link trong Detailed sang v2.1 | Trang + Hoàng | **M-10** — tránh 2 nguồn sự thật |
| 8 | Sửa `[12D]` thành `CI → TI API v2 → RDS`; gộp cạnh →S3 + 1 nhãn `[11A]`; thêm arrowhead; neo nhãn `[01][02][07B][10][11B]` vào cạnh | Hoàng | **DEF-S3-M05/M13** |
| 9 | Sửa Hùng: D5b DAST phải là nhánh **W3 + có Staging URL sống** (không nằm trong `par`); thêm Trục 2 (API) vào danh sách; dùng câu chuẩn "Private Subnet không IGW/NAT + SG DENY ALL + VPC Endpoints" | Hùng | **M-06/M-07/M-03** |
| 10 | Thêm nhãn `UNVERIFIED — chưa nghiệm thu` cho **mọi node Memory/S10** + giải thích trong legend | Hoàng + Trang + Hùng | **E10 / M16 / M-09** |

### 6.2. 🟡 Bổ sung bắt buộc (Minor — ≤ 3 ngày)

- **Quy ước đọc sơ đồ:** sửa định nghĩa nét liền/nét đứt theo `GLOSSARY_TI.md` §A ở v2.1; hạ `[OBSERVED]` ở 2 zone Account A/B trong `images/*`; sửa "v2.0"→"v2.1"; thêm ngày vẽ + tác giả + nhóm vào v2.1; đổi tên file PNG kèm nhãn sự thật.
- **Hạ tầng hiện trạng:** thêm footnote *"Hiện trạng (OBSERVED, đo 22/09 — nguồn ghim `8a61cf66`): API :8000 + Portal :8001 trên 1 EC2; 2 container cách ly, `/app/data` chỉ gắn API + portal"* vào Detailed + v2.1.
- **ECR pre-baked:** thêm node `Amazon ECR (4 image pre-baked + image digest)` + đường **nét đứt** `GitHub Actions → ECR → SSM` ghi rõ *"đường khai báo, chưa đo"* (D04/M11).
- **Kho script test:** ghi rõ `ti-test-packs/` (CANDIDATE) + version theo git tag (D03/F07/M07).
- **Load test an toàn:** thêm hộp cảnh báo *"rủi ro AWS ban / tự DoS người nhà — chỉ bắn trong vùng được phép, xin phép DevOps trước"* (B10/F08/M08).
- **Aurora Clone:** thêm `PENDING tham vấn Team Data (quota snapshot)`; bỏ câu "an toàn 100%" (B11/M09 + F02).
- **Gap scan & prompt-injection:** thêm 2 dòng "Gap & Kế hoạch" vào khối D5a của **mọi** sơ đồ (D06/F09/M10).
- **AgentCore:** thêm block tham chiếu *"AgentCore Runtime 18 — receipt 14/09 (`fbdd8dfc`/`efabf57dcd8c`) — mốc lịch sử"* (D08/A10/M15) + bổ sung node **AgentCore Gateway/Policy** + tier **Haiku 4.5** vào v2.1 cho khớp `Blueprint`/`images` (D07/M03).
- **FinOps:** thêm cột **Nhãn** (`CANDIDATE` mặc định) cho mọi con số; tính lại `$0.0008/lượt quét diff` theo đơn giá thật; đánh dấu `<1s` / `2–5s` là *mục tiêu thiết kế* (A14/F01/D14).
- **Sửa link `file:///c:/Users/T14S/...`** → link tương đối trong Detailed (A15/M12).


---

## 7. KẾT LUẬN & ĐIỀU KIỆN CHUYỂN VÒNG 2 (EVIDENCE)

**Kết luận vòng re-review 1:** 🔴 **CHƯA ĐẠT** cho cả 5 hồ sơ (cần 100% Pass theo ADR-0003).

- **Điểm tiến bộ rõ nhất:** `TI_System_Architecture_v2.1_CANDIDATE.drawio` — đã sửa **thật** 4 Blocking (DEF-X-003, 004, 005, 008) và tách Gate khỏi state, thêm legend + nhãn CANDIDATE + ghi chú hiện trạng. Đây là hồ sơ đúng hướng nhất, **nên lấy làm "bản gốc"** để 3 hồ sơ khác đồng bộ theo.
- **Nút thắt lớn nhất:** **tài liệu Detailed chưa được sửa một dòng nào** (`mtime 27/09 19:42` < giờ góp ý `20:04`), trong khi đây chính là hồ sơ "sơ đồ kiến trúc chi tiết gắn service/tool" của Trang ⇒ 9/11 Blocking thuộc nhóm D vẫn nguyên.
- **Rủi ro lớn nhất:** `CodeGuru (EOL)` quay lại trong hồ sơ **workflow của Hùng** (cả `.md` và `.pdf`) — tức bản đang "giúp team dễ hình dung luồng chạy thực tế" lại chứa dịch vụ không thể triển khai.

**Điều kiện để mở Vòng 2 (Checklist E/F):**
1. 11 Blocking đóng xong ở **tất cả** hồ sơ (không còn 🟡/❌).
2. 10 mismatch ở §5 được chốt bằng **một câu văn bản** trong biên bản (đặc biệt M-01, M-08).
3. Mọi node `CANDIDATE`/`UNVERIFIED` gắn nhãn đúng; 3 claim tuyệt đối ở §5.1 được viết lại.
4. File drawio cũ được khai tử có ghi chú `SUPERSEDED`.

**4 câu hỏi cần team trả lời dứt khoát (để đóng P0/P1):**
1. S03/S04 chạy ở **Account A** hay **Account B**? Ai là system of record của `ImpactSet`/`RiskTier`?
2. Sau khi **CodeGuru bị loại**, tool nào đảm nhiệm **pre-scan S02→S04**, và nó chạy **mấy lần** so với Trục 1 ở S07?
3. **Trần & nhịp** lease/heartbeat là `TTL 5 phút + heartbeat 60s` hay con số khác? (2 hồ sơ đang ghi 2 kiểu)
4. **Trạng thái chuẩn** khi job vừa nhận là `queued` hay `pending` — và `gate_result` được lưu/tách khỏi `state` như thế nào?

> **Ghi chú trung thực (Task_5 §9):** Phiếu này chỉ chấm *bản vẽ + tài liệu có đáng tin làm căn cứ thiết kế hay chưa*. **Không** đồng nghĩa hệ thống đã được kiểm thử/nghiệm thu — portal `/evidence` vẫn ghi *"bằng chứng ≠ phê duyệt"*, **#97 vẫn mở**, và `runtime_binding` vẫn `UNVERIFIED`. Đúng tinh thần Law 18: *bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy.*

---

## 8. PHỤ LỤC — CÁCH TÔI ĐÃ KIỂM (để tái lập)

| Việc | Lệnh / cách làm |
| :--- | :--- |
| Đọc bản vẽ **nguồn** (không chỉ ảnh) | Đọc trực tiếp XML `TI_System_Architecture_v2.1_CANDIDATE.drawio` (477 dòng), `images/TI_Master_Architecture.drawio` (934 dòng) + 4 tab |
| Trích nội dung PDF | `python -c "from pypdf import PdfReader; ..."` cho `Workflow_Hung.pdf` (1 trang/2.479 ký tự) và `Biên bản cuộc họp TI - 23_09_2026.pdf` (2 trang/3.253 ký tự) |
| Truy chuỗi ký tự theo defect | `Select-String` trên `*.drawio` cho `CodeGuru`, `network none`, `CodeBuild`, `Haiku`, `Waiver`, `UNVERIFIED`, `0.03`, `pre-baked`, `Team Data`, `ti-test-packs`, `AWS ban`, `Runtime 18`, `receipt`, `COMPLETED`, `11B`, `12D` |
| Xác định hồ sơ nào **đã** sửa | `git log --oneline` + `git status` + so `mtime` với thời điểm phiếu góp ý (27/09 20:04) |

**CHANGELOG**

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 28/09/2026 | v1.0 | Cline (AI reviewer, theo phương pháp Task_5) | Re-review 5 hồ sơ mới (Hung `.md`+`.pdf`, `images/*.drawio`, `Detailed .md`, `v2.1 CANDIDATE` drawio+PNG) theo Checklist A–F; đối chiếu 11 Blocking + 16 Minor; kết luận **CHƯA ĐẠT** (11 Blocking còn OPEN ở ≥1 hồ sơ; 4 mục đã sửa thật trong v2.1); phát hiện **10 mismatch mới** (M-01→M-10) + 3 claim tuyệt đối; nêu 4 câu hỏi P0/P1 phải chốt trước Vòng 2 |
| 28/09/2026 | v1.1 | Cline | Cập nhật sau khi phát hiện `TI_Master_Architecture_Blueprint.md` **bị sửa giữa lúc review** (09:08:54): bổ sung ghi chú ở §1, sửa số dòng `CodeGuru` L402→**L400**, nâng **M-02 thành P0 mới** (Blueprint "loại bỏ Haiku" ↔ chính bảng FinOps cùng file ↔ `images/*` `m_haiku` ↔ Task 3/4), thêm việc **1b** vào §6.1, đổi `DEF-X-M03` sang **REGRESSION** |

