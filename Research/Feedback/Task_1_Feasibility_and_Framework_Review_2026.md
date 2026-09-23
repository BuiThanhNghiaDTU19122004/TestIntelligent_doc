# BÁO CÁO THẨM ĐỊNH TÍNH KHẢ THI & ĐÁNH GIÁ CÔNG NGHỆ BẢN ĐỀ XUẤT TASK 1 (BẢN CẬP NHẬT SAU REVISION)
## RÀ SOÁT KHẢ NĂNG TRIỂN KHAI THỰC TẾ, ĐỘ PHÙ HỢP CÁC FRAMEWORK VÀ CẬP NHẬT KỸ THUẬT ĐẾN THÁNG 09/2026

---

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Đối tượng thẩm định** | [Task_1_Research_Tool_and_Framework_for_Testing.md](file:///D:/Doc/Research/Task_1_Research_Tool_and_Framework_for_Testing.md) |
| **Tác giả đề xuất** | Đồng nghiệp phụ trách Nhóm 1 (Testing Tools & Frameworks) |
| **Trạng thái tài liệu** | **REVISED (Đã được cập nhật và sửa đổi ngày 22/09/2026)** |
| **Bên thực hiện đánh giá** | Chuyên gia Kiến trúc Hệ thống & AI Testing (Độc lập rà soát cho Nhóm 3) |
| **Mốc thời gian kỹ thuật** | **Tháng 09/2026 (Active baseline)** |
| **Hệ thống mục tiêu** | Testing Intelligence (TI) Platform (Hạ tầng 2 AWS Accounts: `ap-southeast-1` & `us-east-1`) |
| **Kết luận thẩm định mới** | 🟢 **ĐÃ ĐẠT ĐỘ HOÀN THIỆN VÀ KHẢ THI RẤT CAO (~92%). ĐÃ XỬ LÝ TRIỆT ĐỂ ĐIỂM NGHẼN OUTDATE (CODEGURU EOL).** |

---

## 1. TỔNG QUAN ĐÁNH GIÁ SAU KHI ĐỒNG NGHIỆP CẬP NHẬT (POST-REVISION VERDICT)

Sau khi kiểm tra toàn diện các thay đổi trong tài liệu Task 1, ghi nhận: **Đồng nghiệp của bạn đã tiếp thu phản hồi và thực hiện sửa đổi cực kỳ xuất sắc, có tư duy kỹ thuật rất vững vàng.**

1. **Xử lý triệt để lỗi Outdate**: 
   - Đã gỡ bỏ hoàn toàn **Amazon CodeGuru Security** (dịch vụ đã bị AWS khai tử ngày 20/11/2025).
   - Đã phân tích thấu đáo lý do không chọn **Amazon Q Developer** (mô hình $19/user/tháng hướng IDE, không phù hợp cho headless pipeline máy-với-máy).
   - Đã thay thế bằng **AWS CodeBuild / Lambda + Semgrep & Trivy**.
2. **Đồng bộ hóa hoàn hảo với Nhóm 3**:
   - Việc chuyển sang **Semgrep** (quét SAST diff) và **Trivy** (quét CVE dependencies & secrets) giúp Task 1 và Task 3 đạt được sự **thống nhất 100% về công cụ bảo mật**.
3. **Bảo toàn nguyên tắc kiến trúc Serverless**:
   - Chạy Semgrep và Trivy trên serverless container của AWS (CodeBuild/Lambda) giúp giữ đúng tôn chỉ của Nhóm 1: Không tốn chi phí nuôi máy ảo, chỉ trả tiền vài giây khi quét diff PR.

---

## 2. BẢNG MA TRẬN THẨM ĐỊNH 6 TRỤC KIỂM THỬ (SAU KHI FIX)

| STT | Trục kiểm thử (Domain) | Giải pháp sau hiệu chỉnh của Task 1 | Trạng thái kỹ thuật (T9/2026) | Khả năng triển khai | Mức độ phù hợp với TI Platform |
| :---: | :--- | :--- | :---: | :---: | :--- |
| **1** | **API Testing** | AWS CodeBuild / Lambda + Schemathesis & Playwright API | 🟢 **KHẢ THI** | **8.5/10** | Fuzzing OpenAPI tự động rất mạnh; phối hợp tốt với Playwright API cho các luồng nghiệp vụ chuỗi. |
| **2** | **UI / Web Testing** | CloudWatch Synthetics + Playwright & axe-core | 🟢 **KHẢ THI** | **9.5/10** | Tối ưu hàng đầu: Serverless hoàn toàn (`syn-nodejs-playwright-2.0`), chụp ảnh, quay video, quét WCAG chuẩn tiếp cận. |
| **3** | **Security & Risk** | **AWS CodeBuild / Lambda + Semgrep & Trivy** *(ĐÃ FIX)* | 🟢 **XUẤT SẮC** | **9.5/10** | **Điểm sáng lớn nhất**: Tốc độ quét 2–5s, bản quyền $0, tránh vendor lock-in, khớp 100% với Nhóm 3. |
| **4** | **Database Testing** | Aurora Serverless v2 Clone + ECS Fargate (Flyway) | 🟡 **CÓ ĐIỀU KIỆN** | **8.0/10** | Copy-on-write cực nhanh cho DB Aurora Staging; cần lưu ý điều kiện target DB phải dùng Aurora và quota 15 clones. |
| **5** | **Performance Testing** | AWS Distributed Load Testing (DLT) + k6 trên Fargate | 🟢 **KHẢ THI** | **8.5/10** | DLT v4.2.x duy trì tích cực, k6 cực nhẹ. Rất tốt cho Stress test định kỳ và PR rủi ro cao. |
| **6** | **AI Model Evaluation**| Amazon Bedrock Evaluations (`TIRunnerGroundness`) | 🟢 **KHẢ THI** | **7.5/10** | Đo Groundedness chuẩn xác; phù hợp nhất cho chu trình CI/CD nội bộ (Ground Truth Loop). |

---

## 3. CHI TIẾT ĐÁNH GIÁ CÁC ĐIỂM ĐÃ SỬA CỦA ĐỒNG NGHIỆP

### 3.1. Điểm Sửa Đổi Trọng Tâm: Trục An Ninh Bảo Mật (Mục 2.2, 2.3.3, 3.5, 3.7)
* **Những gì đồng nghiệp đã thay đổi**:
  - Tại sơ đồ luồng Mermaid (dòng 235, 263, 280): Đã cập nhật thành `Tool_Sec["★ 1. AWS CodeBuild / Lambda + Semgrep & Trivy"]` kết nối từ S02 $\rightarrow$ S04 $\rightarrow$ S08.
  - Tại bảng ánh xạ Pipeline (dòng 289): Khẳng định xuất file JSON danh sách lỗ hổng định lượng cho `Risk Tier`.
  - Tại Mục 2.3.3: Bổ sung phân tích lý do CodeGuru EOL từ 20/11/2025 và giải thích tại sao Amazon Q Developer không phù hợp với backend TI.
  - Tại Bảng so sánh 3.5 & Bảng tổng kết 3.7: Cập nhật chi phí thực tế: Semgrep/Trivy bản quyền $0, CodeBuild chỉ tốn ~\$0.0008 cho mỗi lượt quét diff 10s.
* **Đánh giá chuyên gia**:
  - **Sửa đổi rất thông minh và thực tế**: Không chỉ giải quyết được bài toán EOL của AWS mà còn giúp giảm chi phí từ $10/100K dòng code (giá cũ của CodeGuru) xuống gần như bằng 0.
  - Tốc độ quét Git diff bằng Semgrep (2–5 giây) nhanh hơn gấp nhiều lần so với việc chờ SaaS engine của AWS phân tích, giúp pipeline của TI phản hồi kết quả cực nhanh cho lập trình viên.

---

## 4. HAI ĐIỂM TINH CHỈNH CUỐI CÙNG (MINOR POLISH) ĐỂ HOÀN THIỆN 100%

Để hai tài liệu Task 1 và Task 3 hoàn toàn khớp nhau như hai nửa của một khối lego, bạn chỉ cần góp ý nhẹ với đồng nghiệp 2 chi tiết sau:

### 1. Đồng bộ chuỗi văn bản phiên bản Model Bedrock tại Mục 2.1 (dòng 211):
* **Hiện trạng trong Task 1**: Vẫn ghi nhận `Bedrock Model (Claude Opus / 3.5)`.
* **Thực tế của Nhóm 3**: Đã nâng cấp lên hệ **Claude thế hệ 5 & Haiku 4.5** (`Claude Haiku 4.5`, `Claude Sonnet 5`, `Claude Opus 5`).
* **Đề xuất**: Đồng nghiệp chỉ cần sửa nhẹ cụm `Claude Opus / 3.5` thành `Claude Sonnet 5 / Haiku 4.5 / Opus 5 (Model Tiering)` để đồng bộ nhận thức kiến trúc trên toàn dự án.

### 2. Bật cờ Quét Secrets trong Trivy:
* Task 3 có đề xuất thêm công cụ **Gitleaks** để quét credentials trong commit history.
* Vì đồng nghiệp đã chọn **Trivy**, Trivy có sẵn tính năng quét secret. Chỉ cần cấu hình cờ `--scanners vuln,secret` trong lệnh chạy của Trivy là bao hàm toàn bộ công năng của Gitleaks mà không cần cài thêm tool mới.

---

## 5. KẾT LUẬN CHUNG CHO HAI NHÓM

Bản tài liệu [Task_1_Research_Tool_and_Framework_for_Testing.md](file:///D:/Doc/Research/Task_1_Research_Tool_and_Framework_for_Testing.md) sau khi sửa đổi đã đạt chất lượng kỹ thuật rất cao:
* **Không còn lỗi outdate**.
* **Ngăn xếp công nghệ hiện đại, bền vững đến cuối 2026 và các năm tiếp theo**.
* **Đã hoàn toàn khớp nối với thiết kế Prompting & Capability Manifest của Nhóm 3**.

Hai bạn hiện đã có một bộ tài liệu nghiên cứu hoàn chỉnh, sẵn sàng tự tin trình bày và bảo vệ trước Product Architect (Tan.Thai) và Tech Lead (anh Quang).
