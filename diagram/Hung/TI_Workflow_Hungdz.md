# TÀI LIỆU : SƠ ĐỒ LUỒNG THỰC THI (WORKFLOW) TESTING INTELLIGENCE (TI)
**Người trình bày:** Hùng (QA Strategy)
**Mục tiêu:** Giúp toàn bộ team (DevOps, Architecture, AI, Tooling) hiểu rõ bức tranh toàn cảnh về cách một tác vụ kiểm thử chạy thực tế trong hệ thống TI, từ lúc tiếp nhận đến khi ra quyết định cuối cùng.

> ⚠️ **LƯU Ý:** Đây là THIẾT KẾ ĐÍCH (CANDIDATE, CHƯA TRIỂN KHAI). Hiện trạng (OBSERVED, đo 22/09): API :8000 + Portal :8001 trên 1 EC2.

---

## 1. TỔNG QUAN
Bản vẽ được đúc kết từ **Master Architecture Blueprint** mới nhất, tổng hợp bài làm của tất cả các mảng: Hạ tầng (Hoàng), AI (Nghĩa) và Tooling (Trang). 

Mục đích của sơ đồ này là để chúng ta thấy rõ: AI đóng vai trò gì, Job Controller điều phối ra sao, và kết quả được lưu trữ như thế nào để đảm bảo tính minh bạch, bảo mật tuyệt đối.

---

## 2. SƠ ĐỒ WORKFLOW TỔNG THỂ

[Xem Sơ đồ Workflow (PDF)](Workflow_Hung.pdf)

*(Lưu ý: Bạn cần update và xuất lại file PDF để khớp với các quy chuẩn kỹ thuật cập nhật dưới đây nha)*

---

## 3. ĐẶC TẢ CHI TIẾT TỪNG GIAI ĐOẠN

### Khởi tạo: Tiếp nhận Yêu cầu
Quá trình bắt đầu khi Developer (hoặc CI/CD pipeline) gọi request `POST /v2/artifact-jobs` đính kèm mã nguồn/artifact và mã băm `sha256`. Hệ thống API hoạt động theo cơ chế bất đồng bộ, phản hồi ngay mã HTTP `202 Accepted` kèm theo một `job_id`. Job được đưa vào hàng đợi với trạng thái `queued` trong cơ sở dữ liệu. Client sẽ sử dụng cơ chế Polling để chủ động theo dõi trạng thái tiếp theo.

### Phase 1: Phân tích ban đầu (S01 - S04)
Giai đoạn này thực hiện phân tích cơ học và tĩnh (Deterministic) chủ yếu tại Account A:
- **S01 & S02**: Hệ thống API ghim ngữ cảnh (Target Registry) và bóc tách tự động những dòng code thay đổi (Change Detector).
- **Security Scanners (Pre-scan)**: Mã nguồn lập tức được đẩy qua công cụ quét tĩnh. Sử dụng chung image D5a với ruleset rút gọn, chạy đúng 1 lần bằng **Amazon Inspector (ECR image scan) + Semgrep/Trivy/Gitleaks** để nhận diện lỗ hổng bảo mật cơ bản.
  > *Gap & Kế hoạch (Prompt-Injection): Semgrep/Trivy hiện chỉ scan source code — chưa đáp ứng hướng scan của Testing. Rủi ro prompt-injection phải nhìn dưới góc quét payload độc hại lọt vào source code.*
- **S03 & S04 (Job Controller - Account A)**: Các bước xác định ImpactSet và Risk Tier là deterministic và được chạy tại Job Controller (Account A). Hệ thống AI (Harness) ở Account B chỉ tham gia trả về gợi ý ImpactSet.

### Phase 2: AI Suy luận & Sinh Test (S05 - S06)
Đây là giai đoạn xử lý suy luận (Reasoning) do AI Harness ở Account B đảm nhiệm:
- **Chiến lược Model Tiering (Dual-Model 2 Cấp)**: Hệ thống phân bổ mô hình AI tối ưu hóa chi phí:
  - Sử dụng **Claude Sonnet 5** làm mô hình mặc định.
  - *Chỉ khi* đánh giá Risk Tier ở S04 là `CRITICAL`, mô hình **Claude Opus 5** mới được gọi để phân tích mô hình đe dọa (Threat Modeling).
- **Kiểm định chất lượng**: Kịch bản sinh ra phải qua Bedrock Evaluations với **bộ 6 chỉ số GenAI (Tolerance ±0.03)**. Đặc biệt, nếu **Faithfulness < 0.85 → kích hoạt HOLD**. Điều này chặn triệt để hiện tượng ảo giác.
- Kết quả đầu ra là **ToolIntent JSON** - định dạng an toàn chỉ chứa các định danh, tuyệt đối không chứa URL thật, mật khẩu hay tham số chạy code tùy ý.

### Phase 3: Điều phối & Thực thi (S07)
Giai đoạn thực thi (Execution) được kiểm soát nghiêm ngặt bởi Job Controller (Account A):
- **Workflow Authority**: Job Controller tiếp nhận `ToolIntent JSON`, thực hiện đối chiếu quyền hạn (Allowlist).
- **TenantBinding & Security**: Job Controller tra cứu cấu hình nội suy URL đích, gọi AWS STS để cấp Token ngắn hạn.
- **Dispatch Ngang Hàng (Wave 1 & 2)**: Các lệnh thực thi được phân phối song song (par) tới các container ECS Fargate Sandbox: **D5a (SAST), API Functional & Fuzzing (Trục 2), D3 (UI), D4 (Perf), và D2.b (DB Clone)**.
- **Dispatch D5b DAST (Wave 3)**: Công cụ quét bảo mật động D5b (ZAP/Nuclei) được tách riêng ở Wave 3 và **chỉ chạy khi có Staging URL sống**.
- **Cô lập mạng & Heartbeat**: Mọi Sandbox Fargate sử dụng kiến trúc mạng **Private Subnet không IGW/NAT + SG DENY ALL + VPC Endpoints (ECR+Logs+STS)**. Các Sandbox gửi `Heartbeat` định kỳ **60 giây** về Job Controller (với trần **Lease TTL là 5 phút**).

### Phase 4: Bằng chứng & Ra Quyết định (S08 - S10)
Giai đoạn thu thập và đối soát cuối cùng:
- **Thu thập & Băm**: Kết quả thô được đẩy lên S3. Job Controller thu thập và băm **SHA-256 Digest**.
- **Tính Bất Biến (Law 16)**: Bằng chứng được khóa cứng bằng **S3 Object Lock**.
- **Phán quyết Gate 3 Nhánh (S09)**: `gate_result` được tách biệt hoàn toàn khỏi trạng thái `state`. Các nhánh gồm:
  1. **PASS**: Cập nhật kết quả an toàn.
  2. **HOLD + Waiver**: Tạm giữ để duyệt thủ công (QA Lead).
  3. **DO_NOT_PASS (Hard-stop)**: Kích hoạt khi `Critical > 0`, `SECRETS_LEAKED > 0`, SHA-256 mismatch, hoặc rollback fail.
- **Quy tắc Law 18**: `completed ≠ PASS` (Trạng thái job hoàn tất không có nghĩa là Gate đã thông qua. Khuyến nghị không phải là phê duyệt cuối cùng).
- **S10 - Production Learning**: Lưu tri thức. Mọi node Memory hiện đều mang nhãn `UNVERIFIED — chưa nghiệm thu` và nguyên tắc là chỉ lưu tri thức đã qua thẩm định `GOLDEN`.

### Yếu tố Con người (Waiver / Override)
Trong quy trình kiểm soát rủi ro (Human-in-the-loop), nếu Gate xếp loại `HOLD`, người có thẩm quyền (QA Lead) có quyền xem xét bằng chứng và gửi lệnh `Approved Waiver` thông qua API `/actions` để cưỡng chế duyệt phát hành (`PASS`).

---

## 4. KẾT LUẬN KIẾN TRÚC
- **Phân tách Trách nhiệm Rõ Ràng**: AI chỉ đóng vai trò tư duy (Reasoning); quyền thực thi và chốt RiskTier (Account A) nằm ở Job Controller.
- **Hiệu Quả Kinh Tế**: Việc áp dụng Model Tiering và tách luồng xử lý giúp tối ưu ngân sách hạ tầng và token.
- **Bảo Mật Kép**: Sự kết hợp giữa Sandbox VPC Endpoints, Token STS Ngắn hạn và S3 Object Lock đảm bảo an toàn tuyệt đối.
