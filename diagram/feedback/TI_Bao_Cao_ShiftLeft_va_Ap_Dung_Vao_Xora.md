# BÁO CÁO: KIỂM TRA LẠI BLOCKING SHIFT-LEFT VÀ ĐỀ XUẤT ÁP DỤNG TI VÀO XORA

**Ngày:** 28/09/2026 · **Trạng thái:** CANDIDATE (đề xuất, chưa triển khai)
**Nguồn:** Gap Closure v2.2, ReReview v1.2, Connect 4 Parts, Checklist Task 5, Blueprint/Detailed/Hùng (bản trong thư mục), hai tài liệu Xora (Banking v1, Sprint 2 v1.0 — 25/09).

---

## 1. Kết luận nhanh

1. Vòng 2 vẫn **CHƯA ĐẠT**. Gap Closure v2.2 liệt kê 14 gap (Blueprint 2, Detailed 12).
2. Tôi đối chiếu lại các file hiện có với checklist và thấy **thêm các Blocking mà v2.2 chưa liệt kê** (mục 2.2).
3. Có **một điểm v2.2 nay đã lỗi thời**: `TI_Workflow_Hungdz.md` không còn CodeGuru (đã dùng Inspector + Semgrep/Trivy/Gitleaks). Có thể bỏ điều kiện "Hùng sửa L28".
4. Về Xora: TI khớp nhất với vai trò **Evaluation Runner + nguồn evidence cho XoraOps**, không phải nơi ra quyết định release (mục 4).

---

## 2. Blocking: còn thiếu những gì?

### 2.1. Đã có trong Gap Closure v2.2 (giữ nguyên, thứ tự fill: G-03 → G-08 → G-09 → G-12 → G-13 → G-14 → G-01/02 → còn lại)

P0: G-03 (link máy cá nhân), G-08 (Secrets ở Account B), G-09 (Gate thiếu Waiver/±0.03/6 chỉ số). P1: G-01, G-02, G-04, G-05, G-06, G-07, G-12, G-13. P2: G-10, G-11, G-14.

### 2.2. Chưa có trong v2.2 (đề xuất bổ sung)

| ID đề xuất | Mức | Vấn đề (kiểm tra trên file hiện tại) | Mã checklist | Cách sửa |
|---|:--:|---|---|---|
| G-15 | P0 | **Blueprint không có `ImpactSet ∩ TargetBinding`** (0 hit). Sơ đồ C4 vẽ 6 cạnh "Direct Dispatch" giống nhau (L240–245), tức đọc như PR nào cũng chạy cả 6 runner. Detailed có công thức này (L120). | B09, F06, C02 | Ghi điều kiện trên cạnh dispatch: chỉ tạo runner thuộc `Verified ImpactSet ∩ TargetBinding`, ngoài tập là `SKIPPED`. |
| G-16 | P1 | **Blueprint không có nhánh NoSQL** (0 hit). Detailed có DynamoDB Local (TTL 1h). | B06, F05, D11 | Thêm DynamoDB Local / Ephemeral Table + hook cleanup vào Trục DB. |
| G-17 | P0 | **Waiver ghi thành quyết định release.** Blueprint L377: `Final Release Decision: PASS`. Hùng: "cưỡng chế duyệt phát hành (PASS)". `Final_workflow.png` bước 38: `Cập nhật Final Gate: PASS`. Trái "KHUYẾN NGHỊ ≠ PHÊ DUYỆT". | Ranh giới §0, C05 | Ghi bản ghi `WaiverDecision` riêng (người duyệt, lý do, thời điểm). Giữ `gate_result = HOLD`. Quyết định release thuộc Release Authority/XoraOps. |
| G-18 | P0 | **S03/S04 vẫn mâu thuẫn giữa các file.** Connect 4 Parts §0 nói Sonnet ở Account B làm S03 Impact và Opus làm S04. G-01, Detailed L100–102 và Hùng nói S03/S04 tất định tại Job Controller (Account A). Ngoài ra Hùng gọi Harness 2 lần (bước 9 và 13). | A11, DEF-X-006 | Chốt một câu: *"Harness đề xuất giả thuyết ImpactSet/RiskTier; Evaluations kiểm; Job Controller (Account A) chốt Verified ImpactSet/RiskTier là system of record."* Sửa Hùng còn 1 lần gọi Harness. |
| G-19 | P1 | **Chỉ số GenAI chưa đủ ở cả 3 file.** Blueprint có `Faithfulness ≥0.85 ±0.03` nhưng không có node Evaluations, temp 0.0, bộ 6 chỉ số. Detailed có temp 0.0 nhưng không có ±0.03 và 6 chỉ số. Chưa nói ±0.03 nghĩa là gì và Eval fail thì retry mấy lần (C-3). | F10, C07, C08 | Gộp một hộp Gate dùng chung: nguồn chỉ số, thời điểm đo, ±0.03, retry ≤ N rồi HOLD. Điểm do LLM chấm ghi nhãn INFERRED. |
| G-20 | P1 | **FinOps chưa tính lại số**, không chỉ thiếu nhãn (G-02/G-12). Network Firewall ghi $280 / $320 / $288; "3 endpoints × $7.3" nhưng danh sách có 4 interface endpoint (`ecr.api`, `ecr.dkr`, `logs`, `sts`). | F01, A14, B14 | Tính lại theo đơn giá công bố, ghi rõ số AZ và giả định số task/job. |
| G-21 | P1 | **Mạng và dữ liệu.** (a) "SG DENY ALL": Security Group chỉ có rule allow, nên viết "không có egress rule ngoài prefix list VPCE và CIDR tenant được allowlist". (b) Chưa vẽ đường mạng cho heartbeat về Job Controller và cho runner tới Staging URL/Aurora clone. (c) Chưa có quyết định phân loại dữ liệu khi source/diff sang `us-east-1`. | B07, A07 | Bổ sung đường mạng; chỉ dùng dữ liệu synthetic/redacted đến khi có quyết định cross-region. |
| G-22 | P2 | Trích dẫn "Law 4.3 / 10.1" và số cổng G-x. Trong TI Architecture Overview v0.1, Law 4 = Policy/Gateway, Law 10 = Publication ≠ Reuse; G4 = E2E, G5 = Knowledge, G6 = Platform Integrated. | A05 | Ghi nguồn đánh số, sửa ánh xạ Wave↔Gate. |
| G-23 | P2 | Ảnh chuẩn có hai tên: "V2 / v2.0" (Connect) và "v2.1" (Gap, Detailed). | A11 | Chọn một tên và tên file. |


---

## 2.3. Thứ tự sửa đề xuất

G-17 → G-18 → G-15 → G-08 → G-09/G-19 → G-03 → G-16 → G-12/G-20 → G-13 → phần còn lại. Lý do: bốn mục đầu chạm ba ranh giới không được xóa hoặc câu mở của biên bản 23/09.

---

## 3. Xora nói gì (tóm tắt từ 2 tài liệu)

- **Thành phần:** Xora Platform (identity, authority, orchestration, Tool Gateway, context, telemetry), Xora Resolve (workflow incident), XBrain (phát triển và cải tiến năng lực), XoraOps (policy, qualification, release, deployment), Tenant Adapter.
- **Luồng chuẩn:** Intent → TenantContext → Authority → Workflow → Capability → Tenant Bindings → Execution → Evidence/Outcome. Agent không tự chọn tenant hay tự nâng quyền.
- **Đánh giá:** đối tượng đánh giá là *cả cấu hình* (model + prompt + skill + retrieval + KB + memory + tools + workflow + binding). Model judge chỉ là tín hiệu phụ trợ; **hard gates** kiểm bằng contract/policy test.
- **Bốn hồ sơ tách nhau:** qualification, acceptance, deployment, activation. Case có nhiều chiều trạng thái độc lập.
- **Sprint 2:** một package điều tra chạy trên 2 integration profile; đánh giá A (baseline) / B (cải tiến) / C (thêm memory); 12 hard gates; Release Manifest; 8 gói bàn giao; kế hoạch 10 ngày; 8 work package S2-01…S2-08.
- **Ràng buộc dữ liệu:** không đưa raw customer data về TechX; chỉ nhận structured escalation và feedback trong phạm vi cho phép.

---

## 4. Áp dụng TI vào Xora như thế nào

### 4.1. Vị trí của TI

| Xora | TI đóng vai trò | Ghi chú |
|---|---|---|
| **Evaluation Runner** (component trong Sprint 2) | TI là hiện thực hóa: nhận artifact/release, chạy check tất định, sinh evidence | Xora: đánh giá A/B/C do QA/Evaluation sở hữu. TI chạy hard gates và evidence, không thay dataset chẩn đoán |
| **Xora Platform** (provider) | TI dùng identity, TenantContext, Tool Gateway, memory binding | TIEF là nền tạm; TI Architecture đã có cutover invariant (một admission, một tool path, không dual-write) |
| **XBrain** | Consumer #1: gửi Work Package/artifact; nhận Gate Recommendation | `PACK.TI.XBRAIN` |
| **Xora Resolve** | Consumer #2: gửi Release Composition, capability manifest, tenant binding, fixture | `PACK.TI.RESOLVE` |
| **XoraOps** | Nhận Evidence Handoff, giữ quyền `Valid/Frozen`, release, activation | TI không tự ra quyết định release |
| **Tenant Adapter / integration profile** | Tenant Pack (`PACK.TI.TENANT.*`): criteria, fixtures, tool policy, ngưỡng | Không fork code, không chứa secret |

### 4.2. Ánh xạ contract và thuật ngữ

| Xora | TI | Việc cần chốt |
|---|---|---|
| `ExecutionContext` / `TenantContext` | `TenantBinding` + execution grant (server-derived) | Dùng một tên; TenantContext lấy từ danh tính đã xác thực, không lấy từ ToolIntent |
| `EvidenceBundle` | `Evidence Envelope` (truth class OBSERVED/DERIVED/INFERRED/CANDIDATE) | Lập bảng ánh xạ trường; thêm truth class vào EvidenceBundle |
| `Release Manifest` | `Release Composition Manifest` | Hai manifest phải liên kết: TI sinh evidence tham chiếu manifest của Xora |
| Hard gates (12 mục, Sprint 2 §25) | Check `PASS/FAIL/INCONCLUSIVE` + Gate Recommendation | Gate của TI là **khuyến nghị** đầu vào cho release review |
| Trạng thái case độc lập | `state` ↔ `gate_result` ↔ review ↔ publication | Đã cùng triết lý; thống nhất bảng trạng thái |
| Memory/KB có vòng đời, revoke | S10: GOLDEN → publish → reuse → revoke | Memory chỉ dùng cho tri thức đã duyệt; reuse phải có receipt |
| 8 gói bàn giao | Deliverable của mỗi consumer pack | Dùng làm checklist Entry của TI |

### 4.3. Điểm móc vào kế hoạch 10 ngày của Sprint 2

| Ngày Sprint 2 | TI làm gì |
|---|---|
| Ngày 2 (chốt contract, fixtures) | Validate contract và capability manifest; kiểm tra secret trong prompt/config |
| Ngày 3–6 (Profile A, B; lỗi tool/quyền) | Chạy negative test (scope, tool ngoài allowlist), kiểm tra Tenant isolation giữa hai profile |
| Ngày 7–8 (candidate, memory lifecycle) | Kiểm tra publish → reuse → revoke; xác nhận revoked item không còn được truy xuất |
| Ngày 9 (holdout, regression, qualification) | Sinh evidence pack, băm SHA-256, lưu bất biến; chuyển cho XoraOps |
| Ngày 10 (release review) | Gate Recommendation là một đầu vào; quyết định do Release Authority |

### 4.4. Lộ trình đề xuất (3 pha)

1. **Pha 1 — TIEF, dữ liệu synthetic.** TI đánh giá artifact của XBrain và package Sprint 2 (contract, manifest, skill/prompt scan). Mục tiêu: một job thật có trace và evidence bất biến (cổng G4).
2. **Pha 2 — Knowledge và tenant.** Chạy chuỗi publish → reuse → revoke; Tenant Pack cho hai profile (cổng G5).
3. **Pha 3 — Cutover sang Xora Platform.** Thay từng provider port (Admission, Identity, Binding, Tool Gateway, Memory, Evidence). Chạy lại contract, negative và regression test (cổng G6).

---

## 5. Rủi ro và điểm phải chốt trước khi áp dụng

| # | Điểm | Đề xuất |
|---|---|---|
| 1 | **Ranh giới dữ liệu ngân hàng:** Xora yêu cầu không đưa raw customer data về TechX, trong khi TI gửi context sang AgentCore ở `us-east-1`. | Chỉ dùng synthetic/redacted; ra quyết định cross-region và vùng xử lý được khách cho phép trước khi chạy dữ liệu thật. |
| 2 | **Faithfulness do LLM chấm** trong khi Xora coi model judge chỉ là tín hiệu phụ trợ. | Giữ `Faithfulness < 0.85 → HOLD` (mềm). Hard-stop `DO_NOT_PASS` chỉ dùng điều kiện tất định. |
| 3 | **Trùng chức năng đánh giá:** Evaluation Runner của Xora và TI. | Phân vai: Xora QA giữ dataset chẩn đoán và A/B/C; TI giữ hard gate, evidence và gate recommendation. |
| 4 | **TIEF là nền tạm.** Thành công trên TIEF không chứng minh qualification trên Xora Platform. | Ghi `platform_integrated: false` trong mọi manifest cho đến G6. |
| 5 | **Trạng thái sẵn sàng.** Cổng #97 còn mở, `runtime_binding` UNVERIFIED. | Không dùng cụm "đã nghiệm thu" trong tài liệu hoặc slide. |

---

## 6. Việc cần bạn xác nhận

1. Chấp nhận G-15…G-23 vào Defect Log (mã DEF-... theo mẫu §8.2 của checklist)?
2. Chốt câu S03/S04 ở G-18 (đề xuất phía trên).
3. Pilot Xora: bắt đầu với XBrain Pack hay Resolve/Sprint 2 package?

*Ghi chú trung thực (Task 5 §9): tài liệu này đánh giá độ tin cậy của bản vẽ và đề xuất áp dụng; không thay bằng chứng của một lượt TI đã chạy.*
