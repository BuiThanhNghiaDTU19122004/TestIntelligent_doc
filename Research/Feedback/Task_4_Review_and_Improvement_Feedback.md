# BÁO CÁO REVIEW VÀ ĐỀ XUẤT CẢI THIỆN TÀI LIỆU TASK 4
## MASTER TEST PLAN, TESTING STRATEGY & EVALUATION FRAMEWORK

---

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Tài liệu đánh giá** | [Research/Task_4_Test_Plan_Strategy_Evaluation.md](file:///D:/Doc/Research/Task_4_Test_Plan_Strategy_Evaluation.md) (v1.0.0) |
| **Tác giả tài liệu** | Hùng — Nhóm 4 (Testing Strategy) |
| **Ngày lập review** | 22/09/2026 |
| **Trạng thái đồng bộ** | **ĐÃ THÔNG SUỐT VỚI TASK 1, 2, 3** *(Task 3 đã hoàn tất bản vá loại bỏ Hurl, Testcontainers, Pixelmatch)* |
| **Điểm ổn định hiện tại** | **9.0 / 10** *(Tăng từ 8.0/10 sau khi Task 3 được đồng bộ)* |
| **Khuyến nghị chung** | **CHẤP THUẬN CÓ ĐIỀU CHỈNH NHẸ (PASS WITH MINOR REVISIONS)** để đạt 10/10 sẵn sàng Sign-off. |

---

## 1. TỔNG QUAN ĐÁNH GIÁ (EXECUTIVE SUMMARY)

Sau khi [Task 3 (AI Model & Prompting)](file:///D:/Doc/Research/Task_3_AI-Prompt_Research.md) được cập nhật đồng bộ với hạ tầng AWS Serverless của [Task 1](file:///D:/Doc/Research/Task_1_Research_Tool_and_Framework_for_Testing.md) và kiến trúc [Task 2](file:///D:/Doc/Research/Task2-Architecture-Report.md), **Task 4 đã có một nền tảng vững chắc và thông suốt toàn diện**:
- Task 4 là tài liệu có **tư duy phương pháp luận tốt nhất**, vận dụng nhuần nhuyễn 3 tầng ISTQB (**CTFL v4.0.1**, **CT-AI v2.0**, **CT-GenAI v1.1**) vào hệ thống Testing Intelligence.
- Bám sát **24 Architecture Laws**, đặc biệt là Law 5 (Deterministic tools đo), Law 7 (Model không tự tuyên bố PASS), Law 16 (Băm SHA-256 raw evidence) và Law 18 (`completed ≠ PASS`).
- Tách bạch độc lập hai đối tượng đánh giá: **Đối tượng A** (Artifact của Tenant) và **Đối tượng B** (Cỗ máy GenAI của TI).

Tuy nhiên, trong tài liệu Task 4 hiện vẫn còn **4 điểm lệch chi tiết nhỏ** và **1 điểm phân công công việc cần cập nhật** để đạt độ hoàn thiện tuyệt đối.

---

## 2. BỐN (04) VẤN ĐỀ HIỆN TẠI VÀ CÁCH KHẮC PHỤC

### 🔴 Vấn đề 1: Nhầm lẫn Tác giả Task 2 tại Trang bìa (Dòng 13)
* **Hiện trạng:** Bảng metadata ghi: `Review đầu vào: Task 1 (Trang) · Task 2 (Nghĩa – AI Model & Prompting) · Task 3 (Nghĩa – TL;DR)`.
* **Ảnh hưởng:** Nhầm lẫn quyền tác giả kiến trúc, bỏ sót vai trò của Hoàng (Nhóm 2).
* **Cách sửa:** Đổi thành:
  > `Review đầu vào: Task 1 (Trang – Tool & Framework) · Task 2 (Hoàng – Architecture) · Task 3 (Nghĩa – AI Model & Prompting)`

---

### 🟡 Vấn đề 2: Khoảng trống Runner thực thi API Candidate (Mục 3.1.1 & 4.2.1)
* **Hiện trạng:**
  - Mục 3.1.1 và 4.2.1 chỉ ghi công cụ chạy API là **Schemathesis**.
  - Nhưng dòng 228 lại yêu cầu Prompt S05/S06 sinh candidate theo schema `API_CANDIDATE_V1` (chứa method, body, headers, custom assertions).
  - *Vấn đề kỹ thuật:* Schemathesis chỉ tự động fuzzing từ file OpenAPI spec thô; Schemathesis **không đọc file JSON `API_CANDIDATE_V1`** do LLM sinh ra để chạy kịch bản luồng.
* **Cách sửa:** Ghi rõ cơ chế **Dual-Mode API Testing**:
  - **Mode 1 (Fuzzing tự động diện rộng):** **Schemathesis** trên AWS CodeBuild/Lambda quét 100% boundary từ OpenAPI spec (không tốn token AI).
  - **Mode 2 (Kịch bản nghiệp vụ phức tạp):** S06 sinh `API_CANDIDATE_V1`, được thực thi bằng **Playwright API (`request.newContext()`) / httpx** trên Fargate/Lambda.

---

### 🟡 Vấn đề 3: Thiếu tích hợp Amazon CodeGuru & Inspector trong Chiến lược Bảo mật (Mục 4.2.5)
* **Hiện trạng:**
  - Bảng In-Scope (Mục 3.1.1) có liệt kê CodeGuru Security + Inspector + Semgrep + Trivy.
  - Nhưng trong Mục 4.2.5 (Chiến lược Security chi tiết) và sơ đồ 3 tầng, chỉ mô tả Semgrep, Trivy, Gitleaks mà vắng bóng CodeGuru/Inspector của Task 1.
* **Cách sửa:** Bổ sung mô hình **Hybrid Defense** vào Mục 4.2.5:
  - *Tầng 1 (AWS Native):* **Amazon CodeGuru Security** (phân tích diff PR bằng ML) + **Amazon Inspector** (quét CVE thư viện tự động).
  - *Tầng 2 (Container cô lập):* **Semgrep OSS** (quét SAST offline) + **Trivy** + **Gitleaks** trong container `--network none`.
  - *Tầng 3 (AI Threat Modeling):* **Claude Opus 5** thẩm định lỗ hổng logic nghiệp vụ khi `Risk Tier == CRITICAL`.

---

### 🟡 Vấn đề 4: Lệch pha nhẹ về Tiêu chí Thoát (Exit Criteria) cho Lỗ hổng High
* **Hiện trạng:**
  - Dòng 343 (Mục 4.2.5) ghi: *"Exit Criteria (Zero Tolerance): CRITICAL_COUNT == 0, HIGH_COUNT == 0"*.
  - Nhưng dòng 502 (Mục 6.2) lại xếp `HIGH_COUNT > 0` vào trạng thái **`HOLD`** (yêu cầu review thủ công), còn chỉ có `CRITICAL` mới bị **`DO_NOT_PASS`** tuyệt đối.
* **Ảnh hưởng:** Gây lúng túng khi lập trình logic Gate tại S09: Có cho phép waiver lỗ hổng High hay chặn đứng hoàn toàn?
* **Cách sửa:** Thống nhất rõ ràng:
  - **`CRITICAL_COUNT > 0` hoặc `SECRETS_LEAKED > 0`**: Bắt buộc `DO_NOT_PASS` (Hard stop - không thể override).
  - **`HIGH_COUNT > 0`**: Kích hoạt trạng thái `HOLD`. Chỉ được phê duyệt thành `PASS` nếu có kèm approved Waiver Document có chữ ký của Architecture Authority.

---

### 🟢 Vấn đề 5: Cập nhật Action Items (Mục 8.3) do Task 3 đã vá xong
* **Hiện trạng:** Mục 8.3 đề xuất P1 là yêu cầu Nghĩa cập nhật Manifest và tool adapter.
* **Cập nhật:** Ghi nhận Task 3 đã hoàn tất cập nhật v2.1.0 (loại bỏ Hurl, Testcontainers, Pixelmatch; tích hợp Playwright API, Aurora Clone, CodeGuru). Nhóm 4 chỉ cần phối hợp chốt ngưỡng `Evaluation Pack` (P0) và xây dựng bộ `Gold Standard Repos` (P1).

---

## 📋 3. BẢNG CHECKLIST HÀNH ĐỘNG CẢI THIỆN CHO TASK 4

| STT | Vị trí trong Task 4 | Thao tác sửa | Mức ưu tiên |
| :---: | :--- | :--- | :---: |
| 1 | **Trang bìa (Dòng 13)** | Đổi tác giả Task 2 thành `Hoàng (Architecture)` | 🔴 P0 |
| 2 | **Mục 3.1.1 & 4.2.1** | Bổ sung `Playwright API` làm runner chạy `API_CANDIDATE_V1` song song với Schemathesis | 🔴 P0 |
| 3 | **Mục 4.2.5** | Đưa Amazon CodeGuru Security & Inspector vào kiến trúc Hybrid Security | 🟠 P1 |
| 4 | **Mục 4.2.5 & 6.2** | Chuẩn hóa `HIGH_COUNT > 0 -> HOLD` (có waiver) vs `CRITICAL -> DO_NOT_PASS` | 🟠 P1 |
| 5 | **Mục 8.3** | Cập nhật tiến độ: Task 3 đã hoàn thành đồng bộ | 🟡 P2 |

---

## 🎯 4. KẾT LUẬN

Bản kế hoạch kiểm thử [Task 4](file:///D:/Doc/Research/Task_4_Test_Plan_Strategy_Evaluation.md) đã đạt **9.0/10 điểm về độ ổn định và tính tương thích**. Sau khi thực hiện 5 điểm tinh chỉnh nhanh trong bảng checklist trên (ước tính mất khoảng **15–30 phút**), tài liệu sẽ đạt trạng thái **10/10 Hoàn hảo (Ready for Sign-off)** để đưa vào triển khai sprint tiếp theo.
