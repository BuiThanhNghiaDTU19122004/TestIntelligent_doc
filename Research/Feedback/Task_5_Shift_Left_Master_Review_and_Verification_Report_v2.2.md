# BÁO CÁO THẨM ĐỊNH KIẾN TRÚC & ĐÓNG GAP SHIFT-LEFT TOÀN DIỆN (v2.2)
## ĐỐI SOÁT HỒ SƠ KIẾN TRÚC TESTING INTELLIGENCE (TI) — HÙNG, TRANG, HOÀNG & ĐỀ XUẤT ÁP DỤNG XORA

> **Người thực hiện:** QA Strategy & Shift-Left Verification Team (Tester #4 / Lead QA)  
> **Ngày lập:** 28/09/2026 · **Phiên bản:** v2.2.0-Master-ShiftLeft  
> **Nguồn đối soát chính:**
> 1. `diagram/Hung/TI_Workflow_Hungdz.md` + `diagram/Hung/Final_workflow.png` (Hùng — Workflow Phase 1–4)
> 2. `diagram/TI_Detailed_System_Architecture.md` (Trang — Đặc tả kỹ thuật chi tiết [01]→[12D])
> 3. `diagram/TI_Master_Architecture_Blueprint.md` + `diagram/images/TI_Master_Architecture.drawio` (Hoàng — Master Blueprint & C4 Diagrams)
> 4. `diagram/TI_AI_Flow_Short_Review_Questions.md` & `diagram/TI_System_Flow_End_to_End_v2.1_Explained.md` (Luồng AI & End-to-End v2.1)
> 5. Các ý kiến phản biện & định hướng: `reference/Feedback từ Nguyễn Thành Đạt.md`, `Research/Feedback/Task_5_Connect_4Parts_Short_Report_2026-09-28.md`, `Research/Feedback/Task_5_Gap_Closure_Review_v2.2_2026-09-28.md`, `Research/Feedback/TI_Bao_Cao_ShiftLeft_va_Ap_Dung_Vao_Xora.md`.
>
> ⚠️ **RANH GIỚI SỰ THẬT TỐI THƯỢNG (ADR-0002 / ADR-0003 / Law 18):**
> - **CANDIDATE ≠ ĐÃ CHẠY:** Toàn bộ thiết kế ECS Fargate Sandbox task-per-job, 6 trục Runner, Direct-to-S3 Offloading, S09 Decision Gate là **THIẾT KẾ ĐÍCH (CANDIDATE)**.
> - **HIỆN TRẠNG ĐO KIỂM THỰC TẾ (OBSERVED 22/09/2026):** Hệ thống đang chạy trên **1 máy EC2 duy nhất** (API cổng `:8000` + Portal cổng `:8001`), dữ liệu kết quả lưu trên ổ đĩa EBS, cổng `#97` còn mở, Memory reuse mang nhãn `UNVERIFIED`.
> - **KHUYẾN NGHỊ ≠ PHÊ DUYỆT:** Phán quyết của TI Gate chỉ là khuyến nghị đầu vào độc lập, quyết định phát hành cuối cùng thuộc về Release Authority / XoraOps.
> - **Law 18:** `completed ≠ PASS` (Trạng thái công việc kết thúc không đồng nghĩa chất lượng đạt chuẩn).

---

## 1. TỔNG QUAN TRIẾT LÝ SHIFT-LEFT: TỪ GÓC NHÌN QA/QC (ĐỊNH HƯỚNG NGUYỄN THÀNH ĐẠT)

Một trong những hạn chế lớn nhất của các phiên bản thiết kế ban đầu là **tập trung quá sâu vào giải pháp hạ tầng** (*"Nên dùng Fargate, k6, Playwright hay S3 như thế nào?"*) mà chưa làm rõ **giá trị cốt lõi từ góc nhìn Đảm bảo Chất lượng (QA/QC)**. 

Theo định hướng của anh Nguyễn Thành Đạt, Shift-Left thực chất là đưa tư duy kiểm thử vào ngay từ khâu thiết kế kiến trúc thông qua chuỗi giá trị 7 bước khép kín:

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 1. BUSINESS FLOW: Luồng nghiệp vụ từ khi Dev commit code/PR đến khi sẵn sàng triển khai Production      │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 2. PAIN POINT & QUALITY RISK: Những rủi ro đe dọa trực tiếp tính ổn định, bảo mật và trải nghiệm        │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 3. QUALITY QUESTIONS: Các câu hỏi cốt tử mà QA/QC và Release Authority bắt buộc phải trả lời            │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 4. EVIDENCE CONTRACT: Bằng chứng cụ thể, định lượng và bất biến cần thu thập để trả lời câu hỏi         │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 5. TEST STRATEGY: Chiến lược kiểm thử, kỹ thuật sinh test và các công cụ thực thi tương ứng             │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 6. PLATFORM REQUIREMENTS: Các yêu cầu phi chức năng về cô lập, bảo mật mạng, ngân sách token & hạ tầng │
└────────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 7. SYSTEM ARCHITECTURE: Bản thiết kế kiến trúc kỹ thuật (4 phân vùng, 6 trục, 10 chặng S01–S10)        │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

Nhờ chuỗi giá trị này, **bản vẽ kiến trúc không bị xáo trộn hay xóa bỏ**, mà ngược lại trở nên **cực kỳ vững chắc và có thể bảo vệ (defend) dễ dàng trước Hội đồng Kiến trúc (SA/DevOps)** vì mỗi container, mỗi VPC Endpoint, hay mỗi quy tắc Gate đều truy vết trực tiếp về một câu hỏi chất lượng cụ thể.

---

## 2. MA TRẬN TRUY XUẤT NGUỒN GỐC CHẤT LƯỢNG (4-WAY SHIFT-LEFT TRACEABILITY MATRIX)

Bảng dưới đây gắn kết trực tiếp giữa **Rủi ro Nghiệp vụ**, **Câu hỏi QA/QC**, **Thành phần Kiến trúc TI** và **Bằng chứng / Gate Phán Quyết**:

| # | Rủi ro Chất lượng Nghiệp vụ (Quality Risk) | Câu hỏi Cốt tử QA/QC Cần Trả Lời (Quality Question) | Chiến Lược & Trục Runner TI Thực Thi | Bằng Chứng Sinh Ra (Evidence Contract) | Gate Phán Quyết & Ngưỡng Chấp Nhận |
|:-:|:---|:---|:---|:---|:---|
| **R1** | Rò rỉ Secret / API Key hoặc lọt lỗ hổng CVE nghiêm trọng lên môi trường dùng chung | *Changeset mới có làm lộ credential hoặc chứa thư viện có lỗ hổng bảo mật đã biết không?* | **Trục 1: D5a Security**<br>(Gitleaks + Trivy + Semgrep OSS) chạy pre-scan sớm tại `[06]` | Báo cáo chuẩn hóa **SARIF v2.1** tải trực tiếp lên S3 Object Lock (`[07A]`) | **Hard-stop `DO_NOT_PASS`** nếu:<br>`SECRETS_LEAKED > 0` hoặc `CVE_CRITICAL > 0` |
| **R2** | Vỡ hợp đồng dữ liệu API (Breaking Contract) gây sập ứng dụng Client / Frontend | *Các API bị thay đổi có vi phạm OpenAPI spec hoặc sập khi gặp payload dị thường không?* | **Trục 2: API Functional & Fuzzing**<br>(Schemathesis Fuzzing + Playwright API Stateful) | Exit code, HTTP Error Matrix, Playwright Trace ZIP (`[11A]`) | **Gate S09:** `5xx_Rate == 0%`, 100% Contract Schema Validated |
| **R3** | Hỏng giao diện người dùng E2E hoặc vi phạm pháp lý về khả năng tiếp cận người khuyết tật | *Luồng người dùng cốt lõi có bị gián đoạn và giao diện có đạt chuẩn tiếp cận quốc tế không?* | **Trục 3: UI Web & Accessibility**<br>(Playwright Headless + axe-core WCAG 2.1 AA) | Video MP4, Screenshot PNG khi lỗi, axe-core Violation JSON (`[11A]`) | **Gate S09:** `E2E_Pass == 100%`, `axe_violations (Critical/Serious) == 0` |
| **R4** | Lỗi migration cơ sở dữ liệu làm hỏng schema hoặc rò rỉ dữ liệu giữa các tenant | *Script Flyway migration có chạy thành công, rollback an toàn và cô lập tuyệt đối dữ liệu không?* | **Trục 4: DB Dual Isolation**<br>(Aurora Serverless v2 Clone + DynamoDB Ephemeral) | Migration Execution Log, Diff Schema Report, Rollback Verification (`[11A]`) | **Gate S09:** Migration Success, Rollback Verified, Hook CleanUp OK |
| **R5** | Suy giảm hiệu năng hệ thống hoặc nghẽn cổ chai khi tải thực tế tăng cao | *Thời gian phản hồi p95 có vượt SLA cam kết và hệ thống có tự phục hồi dưới tải lớn không?* | **Trục 5: Performance Testing**<br>(AWS DLT + k6 Engine + Bộ 3 Khóa An Toàn) | k6 Summary Metrics JSON, Response Time Distribution, Error Rate Graph | **Gate S09:** `p95 < SLA (ví dụ <500ms)`, `5xx_Error < 2%`, không trigger Circuit Breaker |
| **R6** | Lỗ hổng an ninh ứng dụng động (SQLi, XSS, CSRF) khi triển khai Staging | *Ứng dụng khi chạy thực tế có tồn tại điểm yếu bảo mật động có thể bị khai thác không?* | **Trục 6: D5b DAST Task**<br>(OWASP ZAP Active Scan + nuclei — Wave 3) | ZAP Alert Summary, nuclei Finding SARIF, Proof-of-Concept Payloads | **Gate S09:** `High/Critical DAST Findings == 0` (chỉ chạy khi Staging URL sống) |
| **R7** | AI tự suy diễn ảo giác, sinh ra kịch bản test sai lệch hoặc field không tồn tại | *Kịch bản kiểm thử do AI sinh ra có bám sát mã nguồn và phản ánh trung thực nghiệp vụ không?* | **Bedrock Evaluations**<br>(Đo lường độc lập tại Account B, `temp=0.0`) | Báo cáo kiểm định chất lượng AI: Groundedness & Faithfulness Scores | **Evaluations Gate:**<br>`Groundedness >= 0.80`, `Faithfulness >= 0.85` (Dưới $\rightarrow$ `HOLD`) |

---

## 3. TỔNG HỢP REVIEW CHÉO 3 HỒ SƠ (HÙNG, TRANG, HOÀNG) & SƠ ĐỒ LUỒNG

### 3.1. Đánh Giá Hồ Sơ của Hùng (`diagram/Hung/TI_Workflow_Hungdz.md` + `Final_workflow.png`)
* **Điểm mạnh:**
  - Cấu trúc 4 Phase (Phân tích $\rightarrow$ AI Suy luận $\rightarrow$ Điều phối thực thi $\rightarrow$ Bằng chứng & Phán quyết) rất mạch lạc, trực quan, dễ hiểu đối với QA và Developer.
  - Đã loại bỏ hoàn toàn chữ `CodeGuru` (EOL), thay thế chuẩn xác bằng `Amazon Inspector + Semgrep/Trivy/Gitleaks`.
  - Đã tích hợp bộ 3 nhánh Gate (`PASS`, `HOLD + Waiver`, `DO_NOT_PASS`) và nguyên tắc `completed ≠ PASS (Law 18)`.
* **Điểm đã tinh chỉnh trong lượt Shift-Left này:**
  - **Khắc phục G-17:** Sửa đổi định nghĩa Waiver. Trước đây ghi "cưỡng chế duyệt phát hành (`PASS`)", nay đã chuẩn hóa thành: *Ghi nhận bản ghi `WaiverDecision` riêng để tạm gỡ trạng thái giữ trong TI; quyền phê duyệt phát hành cuối cùng thuộc về Release Authority / XoraOps*.
  - Khẳng định rõ Trục 1 Pre-scan chạy đúng 1 lần (dùng chung container image D5a với ruleset rút gọn).

### 3.2. Đánh Giá Hồ Sơ của Trang (`diagram/TI_Detailed_System_Architecture.md`)
* **Điểm mạnh:**
  - Bản đặc tả kỹ thuật chi tiết nhất về 12 bước tuần tự `[01]` đến `[12D]`, có hợp đồng JSON `ToolIntent` và `TenantBinding`.
  - Phân tích sâu 6 Trục Runner Fargate và cơ chế `Direct-to-S3 Offloading`.
* **Điểm đã sửa đổi toàn diện trong đợt Shift-Left này (Đóng triệt để 12 Gaps):**
  - **G-03:** Xóa bỏ 100% các đường dẫn máy cá nhân `file:///c:/Users/T14S/...`, thay bằng link tương đối.
  - **G-04:** Gắn Truth Banner `CANDIDATE (Thiết kế đích)` kèm hiện trạng đo kiểm `OBSERVED 22/09: 1 EC2 ghim 8a61cf66`.
  - **G-05:** Sửa đổi cơ chế WAF (WAF xử lý OWASP + Rate limit; HMAC được xác thực tại API / Lambda@Edge).
  - **G-06:** Loại bỏ thuật ngữ sai `--network none`, thay bằng chuẩn Fargate: `Private Subnet không IGW/NAT + SG DENY ALL EGRESS + VPC Endpoints`.
  - **G-07:** Định vị chính xác VPC Endpoints nằm trong Sandbox Execution VPC (không thuộc Account A Control Plane).
  - **G-08:** Chuyển Secrets Manager & STS Tokens về Account A (Law 13).
  - **G-09:** Bổ sung hộp Gate 3 nhánh đầy đủ, tích hợp bộ 6 chỉ số GenAI (Tolerance `±0.03`), nhánh `Faithfulness < 0.85 → HOLD`, chính sách Retry $\le N$ lần.
  - **G-10:** Thống nhất trạng thái khởi tạo là `queued` (khớp với bộ 4 trạng thái của Portal).
  - **G-11:** Sửa luồng Polling thành `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` (không truy cập trực tiếp RDS).
  - **G-12:** Bổ sung cột `Nhãn Sự Thật` vào bảng FinOps, ghi chú SLA là mục tiêu thiết kế `CANDIDATE`.
  - **G-13:** Bổ sung trọn vẹn 8 khối vận hành nền tảng (Section 7).
  - **G-14:** Chuẩn hóa tên model `Claude Sonnet 5` (`anthropic.claude-sonnet-5`, `us-east-1`).

### 3.3. Đánh Giá Hồ Sơ của Hoàng (`diagram/TI_Master_Architecture_Blueprint.md` & C4 Drawio)
* **Điểm mạnh:**
  - Bản thiết kế kiến trúc chuẩn C4 Level 2, Level 3 và mô hình 3 tầng với bộ icon AWS enterprise.
  - Phân tích FinOps sâu sắc, chỉ rõ mức tiết kiệm từ ~$450-$600/tháng xuống còn ~$90-$122/tháng nhờ Fargate per-second và VPC Endpoints.
* **Điểm đã tinh chỉnh trong lượt Shift-Left này:**
  - **G-01:** Sửa dòng L127: Làm rõ S03/S04 là tất định tại Job Controller (Account A - System of Record); Harness chỉ đóng vai trò bộ não AI đề xuất giả thuyết `ImpactSet`.
  - **G-02:** Thêm nhãn `CANDIDATE` vào hàng Tổng chi phí FinOps (L508).
  - **G-15:** Ghi chú rõ công thức `Target = Verified ImpactSet ∩ TargetBinding` trên các cạnh Dispatch của sơ đồ C4.
  - **G-16:** Bổ sung nhánh DynamoDB Local / Ephemeral NoSQL vào Trục 4 Database Testing.
  - **G-17:** Chuẩn hóa nhánh Waiver trong Sequence Diagram.

---

## 4. BẢNG TỔNG HỢP ĐÓNG TOÀN BỘ 23 GAPS & DEFECT LOG (G-01 ➔ G-23)

Bảng tổng hợp dưới đây ghi nhận trạng thái khắc phục chính thức của toàn bộ các Defect từ Vòng 1 và các Gap mới phát hiện:

| Mã Gap | Mức Độ | Tóm Tắt Vấn Đề Lệch Trước Đây | Giải Pháp Xử Lý & Chuẩn Hóa | Trạng Thái Hiện Tại |
|:---:|:---:|:---|:---|:---:|
| **G-01** | 🟠 P1 | Blueprint L127 xếp S03/S04 là "AI suy luận", đá với sequence L350 | Sửa Blueprint L127: S03/S04 tất định tại Job Controller (Account A); Harness chỉ đề xuất gợi ý | ✅ **ĐÃ ĐÓNG** |
| **G-02** | 🟠 P1 | Blueprint §6 Hàng tổng FinOps thiếu cột `Nhãn Sự Thật` | Thêm nhãn `CANDIDATE (hiện trạng đo 22/09 ghim 8a61cf66 vẫn 1 EC2)` | ✅ **ĐÃ ĐÓNG** |
| **G-03** | 🔴 P0 | Detailed chứa đường dẫn máy cá nhân `file:///c:/Users/T14S/...` | Đã xóa 100% các chuỗi cá nhân, thay bằng relative link nội bộ repo | ✅ **ĐÃ ĐÓNG** |
| **G-04** | 🟠 P1 | Detailed claim tuyệt đối *"Bám sát 100%"* và thiếu Truth Banner | Đã thay bằng Banner `CANDIDATE` và dẫn chứng hiện trạng `OBSERVED` | ✅ **ĐÃ ĐÓNG** |
| **G-05** | 🟠 P1 | Detailed L35/L77 ghi *"WAF kiểm tra chữ ký HMAC"* (WAF không verify HMAC) | Sửa thành: WAF chặn OWASP + Rate limit; HMAC verify tại TI API v2 / Lambda@Edge | ✅ **ĐÃ ĐÓNG** |
| **G-06** | 🟠 P1 | Detailed dùng `--network none` (sai thuật ngữ Fargate) | Thay bằng câu chuẩn: `Private Subnet không IGW/NAT + SG DENY ALL EGRESS + VPCE` | ✅ **ĐÃ ĐÓNG** |
| **G-07** | 🟠 P1 | Detailed §1 đặt VPC Endpoints ở Account A thay vì Sandbox VPC | Đã chuyển khối VPC Endpoints sang Sandbox Execution VPC | ✅ **ĐÃ ĐÓNG** |
| **G-08** | 🔴 P0 | Detailed L53 đặt Secrets Manager & STS Tokens ở Account B (vi phạm Law 13) | Chuyển Secrets Manager & STS Tokens về Account A (Control Plane) | ✅ **ĐÃ ĐÓNG** |
| **G-09** | 🔴 P0 | Detailed gộp `state` với `gate_result`, thiếu hộp 3 nhánh, thiếu `±0.03` | Đã tách riêng `state=completed` và `gate_result`; bổ sung hộp 3 nhánh, `±0.03` và 6 chỉ số | ✅ **ĐÃ ĐÓNG** |
| **G-10** | 🟡 P2 | Trạng thái khởi tạo dùng `PENDING` đá với Portal (`queued`) | Thống nhất dùng từ **`queued`** duy nhất trên toàn bộ các tài liệu | ✅ **ĐÃ ĐÓNG** |
| **G-11** | 🟡 P2 | Sơ đồ vẽ CI/CD poll trực tiếp từ RDS PostgreSQL | Sửa thành: `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` | ✅ **ĐÃ ĐÓNG** |
| **G-12** | 🟠 P1 | Bảng FinOps của Detailed thiếu cột Nhãn, SLA thiếu nguồn | Thêm cột `Nhãn Sự Thật` (CANDIDATE/INFERRED), đánh dấu SLA là mục tiêu thiết kế | ✅ **ĐÃ ĐÓNG** |
| **G-13** | 🟠 P1 | Detailed thiếu 6 khối vận hành (AgentCore Gateway, ECR pre-baked, DoS...) | Đã bổ sung trọn vẹn Section 7 gồm 8 khối vận hành và bảo vệ nền tảng | ✅ **ĐÃ ĐÓNG** |
| **G-14** | 🟡 P2 | Tên model *"Claude 5.0 Sonnet"* không chuẩn | Chuẩn hóa: `Claude Sonnet 5` (`anthropic.claude-sonnet-5`, `us-east-1`) + Opus 5 if CRITICAL | ✅ **ĐÃ ĐÓNG** |
| **G-15** | 🔴 P0 | Blueprint vẽ dispatch như thể PR nào cũng chạy cả 6 runner | Bổ sung điều kiện bắt buộc: $\mathbf{Target = Verified\ ImpactSet \cap TargetBinding}$ | ✅ **ĐÃ ĐÓNG** |
| **G-16** | 🟠 P1 | Blueprint thiếu nhánh kiểm thử cơ sở dữ liệu NoSQL | Bổ sung `DynamoDB Local / Ephemeral Table` với TTL 1h và Hook CleanUp | ✅ **ĐÃ ĐÓNG** |
| **G-17** | 🔴 P0 | Waiver bị hiểu nhầm thành lệnh phê duyệt phát hành tự động | Làm rõ: Waiver chỉ tạo bản ghi `WaiverDecision`; quyền Release thuộc Release Authority | ✅ **ĐÃ ĐÓNG** |
| **G-18** | 🔴 P0 | Mâu thuẫn quyền lực S03/S04 giữa Account A và Account B | Chốt nguyên tắc: Harness gợi ý $\rightarrow$ Evaluations kiểm $\rightarrow$ Job Controller (Account A) chốt | ✅ **ĐÃ ĐÓNG** |
| **G-19** | 🟠 P1 | Chưa định rõ dung sai `±0.03` và chính sách retry khi Eval thất bại | Đã định lượng: Sai số chấp nhận $\pm 0.03$; retry tối đa $N \le 2$ lần, quá $N \rightarrow$ chuyển `HOLD` | ✅ **ĐÃ ĐÓNG** |
| **G-20** | 🟠 P1 | Bảng FinOps tính thiếu số lượng VPC Interface Endpoints | Chuẩn hóa: 4 endpoints (`ecr.api`, `ecr.dkr`, `logs`, `sts`) $\times$ $7.3/tháng $\approx$ $29.2/tháng | ✅ **ĐÃ ĐÓNG** |
| **G-21** | 🟠 P1 | An toàn mạng Egress và quy chế bảo vệ dữ liệu xuyên vùng | SG chỉ mở prefix-list VPCE; dữ liệu context sang `us-east-1` bắt buộc synthetic/redacted | ✅ **ĐÃ ĐÓNG** |
| **G-22** | 🟡 P2 | Trích dẫn số Law và số Gate chưa đồng bộ với Architecture Overview | Chuẩn hóa ánh xạ Law và Wave sang Gate G3 $\rightarrow$ G6 | ✅ **ĐÃ ĐÓNG** |
| **G-23** | 🟡 P2 | Lẫn lộn giữa hai tên gọi bản vẽ: "V2/v2.0" và "v2.1" | Thống nhất tên gọi chuẩn: **`v2.1 CANDIDATE`** trên toàn bộ tài liệu | ✅ **ĐÃ ĐÓNG** |

---

## 5. KẾ HOẠCH ÁP DỤNG THỰC TẾ: ĐƯA TI VÀO HỆ SINH THÁI XORA

### 5.1. Định Vị TI trong Kiến Trúc Xora (Banking / XBrain / Resolve)
Trong hệ sinh thái Xora (Xora Platform, Xora Resolve, XBrain, XoraOps), TI đóng vai trò là **Evaluation Runner & Bằng Chứng Độc Lập (Evidence Engine)**, hoàn toàn không dẫm chân lên vai trò ra quyết định của XoraOps:

```text
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ XORA PLATFORM (Identity, TenantContext, Tool Gateway, Telemetry)                                 │
└─────────────────────────────────┬────────────────────────────────────────────────────────────────┘
                                  │ Cung cấp ngữ cảnh & TenantBinding
                                  ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ TESTING INTELLIGENCE (TI) — EVALUATION RUNNER                                                    │
│  1. Tiếp nhận Changeset / Package từ XBrain hoặc Xora Resolve                                    │
│  2. Thực thi kiểm thử tất định (Contract, Security, Performance, UI) qua 6 Trục Fargate Sandbox  │
│  3. Thu thập Bằng chứng thô Direct-to-S3 + Băm mã SHA-256 Digest                                 │
│  4. Phát phán quyết Gate Recommendation (PASS / HOLD / DO_NOT_PASS)                              │
└─────────────────────────────────┬────────────────────────────────────────────────────────────────┘
                                  │ Xuất Evidence Bundle + Gate Recommendation
                                  ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ XORAOPS (Release Authority & Policy Enforcement)                                                 │
│  • Tiếp nhận bằng chứng bất biến từ TI                                                           │
│  • Thẩm định nghiệp vụ ngân hàng & chấp thuận Waiver (nếu có)                                    │
│  • Ra Quyết Định Cuối Cùng: Release Approval / Activation lên Môi Trường Thực Tế                 │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2. Ba Rào Cản An Toàn Khi Áp Dụng Cho Dữ Liệu Ngân Hàng (Banking Constraints)
1. **Ranh giới Dữ liệu Xuyên Vùng (Cross-Region Boundary):** Ngân hàng nghiêm cấm đưa dữ liệu khách hàng thật (PII / raw customer data) ra khỏi lãnh thổ hoặc sang region `us-east-1`. Vì vậy, dữ liệu gửi sang Bedrock trong chặng S03–S06 bắt buộc phải là **dữ liệu tổng hợp (synthetic)** hoặc đã qua xử lý **che mờ (redacted)**.
2. **AI Judge chỉ là Tín hiệu Phụ trợ:** Điểm số do LLM chấm (`Faithfulness`, `Groundedness`) chỉ dùng để phân loại cảnh báo hoặc đưa vào nhánh `HOLD`. **Quyết định chặn đứng (`DO_NOT_PASS`) bắt buộc phải dựa trên các quy tắc tất định 100% (Deterministic Hard Gates)**: Rò rỉ secret, vỡ schema contract, hoặc sập hệ thống.
3. **Mô hình Cắt Chuyển Từng Bước (Cutover Invariants):** Không triển khai ồ ạt; áp dụng lộ trình 3 pha:
   - *Pha 1 (TIEF Baseline):* Chạy thẩm định tĩnh contract và scan an ninh trên dữ liệu mẫu (Cổng G4).
   - *Pha 2 (Tenant & Memory Binding):* Áp dụng Tenant Pack cho các hồ sơ tích hợp (Profile A/B), kiểm tra vòng đời Memory publish $\rightarrow$ reuse $\rightarrow$ revoke (Cổng G5).
   - *Pha 3 (Cắt chuyển sang Xora Platform):* Chuyển toàn diện sang Xora Platform provider port, đạt chứng nhận hoàn tất kiểm thử (Cổng G6).

---

## 6. KẾT LUẬN & KIẾN NGHỊ NGHIỆM THU

1. **Kết quả Shift-Left:** Toàn bộ **23 Gaps (bao gồm 6 Blocking P0, 12 Critical P1, và 5 Minor P2)** đã được xử lý triệt để, đồng bộ hóa 100% giữa tài liệu của Hùng, Trang, Hoàng và sơ đồ luồng chi tiết.
2. **Đủ Điều Kiện Chuyển Vòng 2:** Bộ hồ sơ thiết kế kiến trúc hiện tại đã đạt độ tin cậy tuyệt đối về mặt lý thuyết và quy chuẩn kỹ thuật tĩnh (Đạt 100% Checklist Vòng 1).
3. **Bước Kế Tiếp:** Đề nghị Lead QA (Nghĩa) phê chuẩn mở **Vòng 2 (Evidence đối chiếu môi trường thực tế)** và chuẩn bị kịch bản đo kiểm Spike P4 trên hạ tầng AWS Fargate.
