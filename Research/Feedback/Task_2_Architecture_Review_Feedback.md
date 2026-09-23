# PHIẾU ĐÁNH GIÁ & GÓP Ý KIẾN TRÚC TASK 2 (ARCHITECTURE REVIEW FEEDBACK)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Đối tượng đánh giá** | [Task2-Architecture-Report.md](file:///D:/Doc/Research/Task2-Architecture-Report.md) (Phiên bản v0.1-draft) |
| **Tác giả đề xuất** | Hoàng (Task 2 — Architecture) |
| **Bên rà soát** | Product Architect & Hội đồng Kỹ thuật TI |
| **Ngày lập** | 22/09/2026 |
| **Kết luận chung** | 🟡 **CHẤP THUẬN CÓ ĐIỀU KIỆN (CONDITIONAL PASS)** — Cho phép chạy Spike P4; Yêu cầu sửa đổi các điểm lệch pha để nâng cấp lên v0.2 trước khi ký chính thức. |

---

## 1. ĐÁNH GIÁ TỔNG QUAN (EXECUTIVE SUMMARY)

Bản đề xuất của Hoàng thể hiện tư duy kiến trúc hệ thống và quản trị rủi ro rất bài bản, bám sát các nguyên tắc cốt lõi (24 Laws) của TI. Việc mở rộng TI từ "Evaluator thuần" sang "Executor" thông qua **Sandbox ECS Fargate task-per-job** là định hướng chính xác để giải quyết các hạn chế hiện tại.

Tuy nhiên, tài liệu **chưa đồng bộ với kết quả nghiên cứu của Task 1 và Task 3**, sơ đồ kiến trúc còn trừu tượng và có giải pháp hạ tầng chưa tối ưu về mặt chi phí ở giai đoạn đầu.

---

## 2. NHỮNG ĐIỂM LÀM TỐT (ĐIỂM CỘNG CẦN PHÁT HUY)

1. **Tuân thủ xuất sắc Architecture Law 23 (Provider Port)**:
   - Đưa ra interface `IsolatedRunner` giúp cô lập lõi điều phối với engine hạ tầng. Nếu sau này Fargate không tối ưu, có thể đổi sang EKS+Karpenter mà không phải đập đi xây lại hệ thống.
2. **Tuân thủ nguyên tắc an ninh (Laws 12–16)**:
   - Cơ chế sandbox ngắt internet mặc định, băm SHA-256 raw result lưu vào S3 Object Lock để bảo toàn tính toàn vẹn của bằng chứng.
3. **Nâng cấp kịp thời hạ tầng Job Store**:
   - Đề xuất thay thế SQLite DEV bằng **Amazon RDS PostgreSQL** (hỗ trợ lease, heartbeat, worker recovery) theo chuẩn NFR §23.
4. **Phương pháp nghiên cứu khoa học**:
   - Gắn nhãn dữ liệu (`OBSERVED`, `INFERRED`, `CANDIDATE`), có tiêu chuẩn phản nghiệm (*falsifiability*) rõ ràng và đề xuất chạy Spike P4 với kinh phí tiết kiệm (< $200).

---

## 3. BỐN (04) ĐIỂM BẮT BUỘC PHẢI ĐIỀU CHỈNH (ACTION ITEMS)

### 🔴 Điểm 1: Đồng bộ Ngăn xếp Bảo mật với Task 1 và Task 3 (Domain D5)
* **Hiện trạng của Hoàng**: Đề xuất dùng **OWASP ZAP / nuclei container** và đẩy vào Wave 3 (W3).
* **Vấn đề**: ZAP và nuclei là công cụ **DAST** (chỉ quét được web đang chạy). Khi dev tạo PR, code chưa deploy thì không thể quét được. Trong khi Task 1 và Task 3 cần công cụ quét mã nguồn tĩnh (SAST) ngay tại Phase 1 để tính điểm rủi ro (*Risk Tier*).
* **Yêu cầu sửa đổi**:
  - Đưa **Semgrep OSS + Trivy** vào **Wave 1 (W1)** làm công cụ quét SAST, SCA và Secret mặc định trong sandbox.
  - Đổi tên D5 thành **DAST Runner (ZAP/nuclei)** và giữ ở W3 (chỉ kích hoạt khi có Staging URL sống).

---

### 🔴 Điểm 2: Bổ sung Miền Database Testing vào Kiến trúc
* **Hiện trạng của Hoàng**: Bảng D1–D14 bỏ quên hoàn toàn miền Database Testing.
* **Yêu cầu sửa đổi**:
  - Bổ sung domain **D2.b (Database Testing)**: Tích hợp giải pháp **Amazon Aurora Serverless v2 Clone + Fargate chạy Flyway / SQLAlchemy read-only** (giải pháp đã được Task 1 và Task 3 thống nhất).

---

### 🟡 Điểm 3: Tối ưu Chi phí Mạng D9 (Bỏ AWS Network Firewall)
* **Hiện trạng của Hoàng**: Dùng NAT Gateway + AWS Network Firewall để chặn internet sandbox.
* **Vấn đề**: AWS Network Firewall có chi phí cố định tối thiểu **~$280/tháng/AZ**. Với quy mô thử nghiệm 500 job/tháng (~$30 compute), việc chi trả $280 cho firewall là lãng phí nghiêm trọng.
* **Yêu cầu sửa đổi**:
  - Thay thế bằng: **Private Subnet (không route ra internet) + Security Group Deny All + VPC Endpoints (S3, ECR, CloudWatch Logs)**.
  - Chi phí giảm từ **~$320/tháng** xuống còn **chưa đến $15/tháng** mà vẫn đảm bảo ngắt internet 100%.

---

### 🟡 Điểm 4: Sửa lại Sơ đồ Kiến trúc & Làm rõ Luồng Giao tiếp (Handshake)
* **Hiện trạng của Hoàng**: Sơ đồ Mục 8 vẽ `Job Controller --> Sandbox --> Browser/Load/Security` gây hiểu lầm là mô hình Fargate task lồng Fargate task. Chưa có luồng nhận kết quả từ AI (Task 3).
* **Yêu cầu sửa đổi**:
  - Vẽ lại sơ đồ: Thể hiện rõ Job Controller là bên nhận `ToolIntent JSON` từ AgentCore (Task 3), tra cứu `TenantBinding` (URL thật + Secrets), rồi dispatch trực tiếp đến từng Task Fargate chuyên biệt tương ứng (`Playwright Task`, `k6 Task`, `Semgrep Task`).

---

## 4. KẾT LUẬN & ĐỀ XUẤT PHÊ DUYỆT

1. **Phê duyệt ngân sách Spike P4 (< $200)**: Đồng ý cho Hoàng chạy thử nghiệm đo độ trễ Fargate Cold Start và kiểm chứng interface `IsolatedRunner`.
2. **Kế hoạch cập nhật**: Hoàng ngồi lại với đại diện Task 1 và Task 3 trong 1 buổi làm việc để khớp nối 4 điểm trên, xuất bản bản **v0.2** trước ngày 24/09/2026 để tiến hành ký duyệt chính thức (*Sign-off*).
