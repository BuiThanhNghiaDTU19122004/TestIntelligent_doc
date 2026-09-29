# Phân Tích Điểm Nghẽn (Pain Points) Trụ Cột 3: Black-Box AI Testing & RAG Evaluation

> **Mục tiêu tài liệu**: Phân tích chi tiết các thách thức, hạn chế kỹ thuật và điểm nghẽn kiến trúc mà Trụ cột 3 (Black-box RAG Evaluation) của QA Cat đang gặp phải trong môi trường kiểm thử thực tế.  
> **Tài liệu liên quan**: [`01_Tong_Quan_QA_Agent.md`](file:///D:/Doc/QA-Agent/01_Tong_Quan_QA_Agent.md), [`02_Luong_Chay_Chi_Tiet_QA_Agent.md`](file:///D:/Doc/QA-Agent/02_Luong_Chay_Chi_Tiet_QA_Agent.md).

---

## 1. Bản Chất Của Trụ Cột 3 Trong QA Cat Hiện Tại

Trong phiên bản hiện hành của QA Cat, Trụ cột 3 chịu trách nhiệm kiểm thử các ứng dụng RAG (Retrieval-Augmented Generation) Chatbot theo mô hình **Hộp đen (Black-box)**:
- **Playwright DOM Driving**: Tự động mở trình duyệt, gõ câu hỏi từ dataset vào khung chat của ứng dụng mục tiêu, đợi text streaming hoàn tất và cào (scrape) câu trả lời từ DOM.
- **RAGAS Metric Evaluation**: Sử dụng AWS Bedrock Opus để chấm 3 chỉ số cốt lõi: Relevancy, Correctness và Faithfulness.

Dù tiếp cận này mang lại tính tất định cao (không phụ thuộc vào LLM Agent trong lúc lái browser), nó bộc lộ **6 điểm nghẽn chí mạng (Pain Points)** khi triển khai ở cấp độ doanh nghiệp.

---

## 2. Chi Tiết 6 Điểm Nghẽn Kỹ Thuật (Key Pain Points)

### 2.1. Điểm Nghẽn 1: Sự Lệch Pha Ngữ Cảnh Trong Kiểm Thử Hộp Đen (Context Reconstruction Dilemma)
- **Vấn đề**: Để chấm được chỉ số **Faithfulness** (độ trung thực, không bị ảo giác), LLM Judge bắt buộc phải có thông tin `context` (các đoạn văn bản mà hệ thống RAG đã tham khảo để sinh câu trả lời).
- **Hạn chế hộp đen**: Vì là kiểm thử hộp đen qua giao diện web, QA Cat **không thể can thiệp hay nhìn thấy** vector database, pipeline retrieval hoặc danh sách chunks thực sự mà Chatbot mục tiêu đã lấy ra.
- **Hệ quả**: QA Cat buộc phải tự xây dựng một pipeline retrieval song song: Người kiểm thử upload tài liệu PDF vào QA Cat, hệ thống tự chunk và nhúng qua Amazon Titan Embed v2 để tái tạo context. Khi thuật toán tìm kiếm của QA Cat (top-k, reranking) khác với Chatbot thật, **context được cấp cho LLM Judge bị lệch hoàn toàn so với context mà Chatbot đã đọc** $\rightarrow$ Dẫn đến tình trạng báo động giả (False-Negative): Chatbot trả lời hoàn toàn đúng với dữ liệu nội bộ của nó, nhưng vẫn bị LLM Judge phạt 0 điểm vì context của QA Cat không chứa thông tin đó.

### 2.2. Điểm Nghẽn 2: Ràng Buộc Miền Nghiệp Vụ Quá Hẹp (Domain Lock-in & Wizard Coupling)
- **Vấn đề**: Luồng khởi tạo kiểm thử (Wizard UI) của QA Cat được thiết kế gắn chặt (hard-coupled) với mô hình RAG Chatbot truyền thống:
  - Bắt buộc phải có tab Source Docs (tải PDF).
  - Bắt buộc phải cấu hình Chunk Recipe (Chunk Size, Chunk Overlap).
- **Hệ quả**: Hệ thống **không thể tái sử dụng** để kiểm thử các loại mô hình AI khác rất phổ biến hiện nay:
  - Các mô hình Foundation Model thuần túy (Completion / Reasoning LLMs).
  - Các LLM chuyên sinh mã (Code Generation / Code Completion).
  - Các AI Agent tự chủ gọi Tool / API nhiều bước (Autonomous Multi-step Tool Calling).
  - Các API phân loại văn bản, trích xuất thực thể có cấu trúc (Information Extraction / Named Entity Recognition).

### 2.3. Điểm Nghẽn 3: Bộ Tiêu Chí Đánh Giá Quá Cũ & Thiếu Chuẩn Xác Thực (Metric Limitations)
- **Vấn đề**: RAGAS chỉ tập trung vào ngữ nghĩa văn bản tự do (unstructured text overlap & semantic similarity):
  - *Answer Relevancy*: Câu trả lời có bám theo câu hỏi.
  - *Faithfulness*: Câu trả lời có nằm trong context.
  - *Answer Correctness*: Câu trả lời có giống expected answer.
- **Điểm mù trong thực tế sản xuất**:
  - Không thể xác thực tính tuân thủ cấu trúc dữ liệu (**JSON Schema Validation**).
  - Không kiểm tra được độ chính xác của chữ ký hàm khi gọi Tool (**Tool Call Arguments & Signature Accuracy**).
  - Không đo lường và cưỡng chế cam kết hiệu năng thời gian thực (**SLA P95/P99 latency constraints**).
  - Không kiểm thử được khả năng tuân thủ phủ định (**Negative Prompt Adherence** - ví dụ: *"Không được tiết lộ thông tin nội bộ"*).

### 2.4. Điểm Nghẽn 4: Khóa Chặt Một Nhà Cung Cấp & Thiên Lệch Thẩm Định (Provider Lock-in & Judge Bias)
- **Vấn đề**: Module adapter thẩm định ([`bedrock_judge_adapter.py`](file:///D:/Doc/diagram/redraw_qa_pentest.py#L148)) được viết cứng để chỉ giao tiếp với **AWS Bedrock Converse API**.
- **Hệ quả**:
  - **Judge Self-Preference Bias**: Khi Chatbot mục tiêu cũng dùng Claude (ví dụ Claude 3.5 Sonnet) và LLM Judge là Claude Opus, mô hình thẩm phán có xu hướng đánh giá thiên vị hơn so với khi kiểm thử các mô hình họ GPT hay Gemini.
  - Không thể thực hiện đánh giá đối chuẩn đa nhà cung cấp (Multi-provider Benchmark): Không thể cùng lúc gọi GPT-4o, Gemini 2.5 Pro, Claude 3.7 và các mô hình mã nguồn mở cục bộ (Ollama / vLLM / DeepSeek) để lấy điểm đồng thuận (Consensus Scoring).

### 2.5. Điểm Nghẽn 5: Nợ Kỹ Thuật Chưa Tự Động Hóa (The "Pending Wiring" Technical Debt)
- **Thực trạng**: Hiện tại, tính năng nạp tự động tài liệu từ giao diện (`Source tab -> KB Docs`) vào tham số `context` của engine đánh giá vẫn đang trong trạng thái *Pending*.
- **Hậu quả vận hành**:
  - Khi chạy Playwright tự động, `context` mặc định bị bỏ trống $\rightarrow$ Chỉ số **Faithfulness không thể tự động chấm**.
  - Muốn chấm Faithfulness, kỹ sư QA phải gọi thủ công qua REST API bằng cURL: `POST /api/ragas/test-runs/{id}/judge` kèm payload context. Điều này phá vỡ hoàn toàn tính tự động hóa khép kín (End-to-end CI/CD).

### 2.6. Điểm Nghẽn 6: Độ Dễ Vỡ Khi Cào Dữ Liệu Trình Duyệt (DOM Scraping Fragility)
- **Vấn đề**: Thay vì kiểm thử trực tiếp qua API endpoint của Chatbot, việc dùng Playwright để gõ và cào DOM rất dễ gặp sự cố:
  - Token streaming từ Server-Sent Events (SSE) hoặc WebSocket khiến DOM thay đổi liên tục, gây ra hiện tượng bắt thiếu chữ (premature capture) nếu selector timeout không chuẩn.
  - Bất kỳ thay đổi nhỏ nào về CSS class hoặc HTML hierarchy của khung chat phía frontend sẽ làm gãy toàn bộ kịch bản test.

---

## 3. Bảng Tóm Tắt & Giải Pháp Đề Xuất Từ Test Intelligent (TI)

| Điểm Nghẽn Trụ Cột 3 | Tác Động Tiêu Cực | Hướng Giải Quyết Đề Xuất Qua TI |
| :--- | :--- | :--- |
| **Context Mismatch** | False-Negative cao, phạt oan câu trả lời đúng | TI S07 chuẩn bị sẵn `PinnedContext` (SHA-256 locked) nạp trực tiếp vào phiên test. |
| **Domain Lock-in** | Không test được Agent, Code LLM, JSON API | Tách riêng tầng đánh giá: Hỗ trợ cả Native API Driver lẫn Headless Browser. |
| **Metric Limitations** | Bỏ sót lỗi JSON vỡ, sai tham số Tool, vi phạm SLA | Bổ sung bộ Metric chuẩn công nghiệp: Schema Compliance, Tool Accuracy, P95 Latency. |
| **Provider Lock-in** | Thiên lệch thẩm định, khó đối chuẩn thị trường | Tích hợp MLflow Native Evaluation Harness hỗ trợ Bedrock, OpenAI, Anthropic, Ollama. |
| **Pending Wiring** | Đứt gãy quy trình CI/CD tự động | Tự động hóa qua TI Job Dispatcher: Runner nhận trọn gói config, dataset và KB context. |
| **DOM Fragility** | Test flaky, chi phí bảo trì script cao | Ưu tiên kiểm thử cấp API/SSE streaming; chỉ dùng DOM cho kiểm thử trải nghiệm người dùng cuối. |
