---
title: "Luồng Chạy Chi Tiết Hệ Thống QA Cat (qa-agent)"
aliases: ["QA Cat Workflow", "Luồng chạy QA Cat", "qa-agent execution flow"]
tags:
  - workflow
  - execution-flow
  - qa-cat
  - multi-agent
  - bedrock
  - playwright
created: 2026-09-29
author: TechX Corp / Architecture & QA Team
status: LIVE / SANDBOX
---

# Luồng Chạy Chi Tiết Hệ Thống QA Cat (`qa-agent`)

Tài liệu này đặc tả chi tiết toàn bộ vòng đời thực thi (Execution Lifecycle) của nền tảng **QA Cat**, từ khâu khởi tạo dự án, phân rã mục tiêu, điều khiển trình duyệt, vòng lặp đánh giá của các sub-agents cho đến khâu xuất bằng chứng và giám sát qua MLflow.

---

## 1. Sơ Đồ Khái Niệm Tổng Thể (Architecture Flow)

```mermaid
flowchart TD
    User(["QA Engineer"]) -->|1. Nhập URL & Mục tiêu tự nhiên| UI["Web UI (React 18 + Zustand)"]
    UI -->|2. WebSocket v2 (/api/ws/agent/v2)| Supervisor["Supervisor Orchestrator\n(FastAPI :8000)"]
    
    subgraph MultiAgent_Loop ["Vòng lặp Multi-Agent (AWS Bedrock)"]
        Supervisor -->|3. Yêu cầu lập kế hoạch| Planner["Planner Agent\n(Claude Opus 4.7)"]
        Planner -->|4. Test Plan v1 (Tasks)| Supervisor
        Supervisor -->|5. Giao Task hiện tại| Executor["Executor Agent\n(Claude Sonnet 4)"]
        
        Executor -->|6. Lệnh điều khiển CDP| Browser{"Browser Backend\n(AgentCore / Playwright)"}
        Browser -->|7. DOM, Screenshot, Logs| Executor
        Executor -->|8. Task Execution Result| Supervisor
        
        Supervisor -->|9. Thẩm định kết quả| Critic["Critic Agent\n(Claude Opus 4.7)"]
        Critic -->|10. Phán quyết:\nAPPROVE / RETRY / REPLAN / SKIP| Supervisor
    end

    Supervisor -->|11. Replan nếu fail| Planner
    Supervisor -->|12. Tracing & Metrics| MLflow["MLflow Observability\n(sqlite:///mlflow.db)"]
    Supervisor -->|13. Stream trạng thái & Video live| UI
    Supervisor -->|14. Lưu trữ Session & Artifacts| SQLite[("SQLite Database\n(qa-agent-data/)")]
```

---

## 2. Các Giai Đoạn Vận Hành Chi Tiết (Step-by-Step Lifecycle)

### Giai đoạn 1: Thiết lập & Khởi tạo Ngữ cảnh (Project Setup)
Mọi tác vụ trong QA Cat đều được đóng gói theo phạm vi **Project** (Tenant Scope):
1. **Khởi tạo qua Wizard 5 bước (`/projects/new`):**
   - **Bước 1 - Identity:** Nhập Tên dự án, Target URL cần kiểm thử.
   - **Bước 2 - Knowledge Base (KB):** Tải lên tài liệu nghiệp vụ (PDF, DOCX, spec) hoặc link SharePoint.
   - **Bước 3 - Chunk Recipe:** Cấu hình chiến lược phân mảnh văn bản (bot-aligned chunking) phục vụ tìm kiếm ngữ nghĩa với `Titan Embed v2`.
   - **Bước 4 - Guardrail & Persona:** Thiết lập các chốt chặn an toàn và vai trò của agent.
   - **Bước 5 - Test Plan:** Nạp test suite có sẵn (CSV/JSON) hoặc chọn chế độ tự động sinh.
2. **Quản lý phiên làm việc (Session & URL State):**
   - URL trạng thái hỗ trợ khôi phục tiến độ: `/projects/new?resume=<id>&step=<n>`.
   - Toàn bộ metadata, lịch sử chat, các plan version được lưu bền vững trong SQLite tại `./qa-agent-data/`.

---

### Giai đoạn 2: Tiếp nhận Mục tiêu & Phân rã Kế hoạch (Planning Phase)

```mermaid
sequenceDiagram
    autonumber
    actor QA as QA Engineer
    participant UI as Web UI (React)
    participant WS as WebSocket Handler (/v2)
    participant Sup as Supervisor Orchestrator
    participant Plan as Planner (Opus 4.7)
    participant ML as MLflow Tracer

    QA->>UI: Nhập mục tiêu tự nhiên & Bấm "Open Browser / Run"
    UI->>WS: Gửi payload init_session(goal, target_url, project_id)
    WS->>Sup: Tạo phiên làm việc mới & Session ID
    Sup->>ML: Khởi tạo Tracing Span (bedrock.autolog)
    
    Sup->>Plan: Gọi Bedrock Converse API kèm System Prompt & KB Context
    Note over Plan: Phân tích mục tiêu người dùng,<br/>đối chiếu cấu trúc trang,<br/>chia nhỏ thành danh sách Task nguyên tử
    Plan-->>Sup: Trả về cấu trúc Plan Spec (Tasks 1..N)
    Sup->>UI: Phát sự kiện WS: `plan_created` (hiển thị danh sách Task bên ChatPanel)
```

- **Mô hình AI sử dụng:** **Claude Opus 4.7** (được chọn vì năng lực suy luận sâu, tránh thiếu sót các bước phụ thuộc).
- **Đặc điểm Plan:** Plan được đánh số version bất biến (`plan_v1`). Mỗi task có:
  - `task_id`: Định danh duy nhất.
  - `description`: Mô tả hành động cụ thể cần làm.
  - `expected_outcome`: Điều kiện nghiệm thu thành công của task.

---

### Giai đoạn 3: Thực thi Trình duyệt & Vòng lặp Multi-Agent (Execution Loop)

Đây là trái tim của hệ thống QA Cat, vận hành theo cơ chế khép kín **Executor $\rightarrow$ Browser $\rightarrow$ Critic**:

```mermaid
sequenceDiagram
    autonumber
    participant Sup as Supervisor
    participant Exec as Executor (Sonnet 4)
    participant Brw as Browser (AgentCore / Playwright)
    participant Crit as Critic (Opus 4.7)
    participant UI as Web UI (BrowserPanel)

    loop Cho từng Task trong Plan
        Sup->>Exec: Yêu cầu thực thi Task[i]
        
        loop Tương tác chi tiết đến khi Task hoàn tất
            Exec->>Brw: Gửi lệnh CDP (Click selector, Type text, Scroll, Navigate)
            Brw-->>Exec: Trả về DOM Tree, Visual Snapshot, Console Logs
            Brw-->>UI: Truyền hình ảnh thời gian thực (DCV 1080p hoặc MJPEG stream)
            Exec->>Exec: Tự đánh giá tiến độ tương tác
        end
        
        Exec-->>Sup: Gửi kết quả thực thi Task kèm bằng chứng (Screenshots, Logs)
        
        Sup->>Crit: Chuyển bằng chứng + Expected Outcome cho Critic
        Note over Crit: So sánh màn hình thực tế và DOM<br/>với mục tiêu kỳ vọng của Task
        Crit-->>Sup: Trả về Phán quyết: (APPROVE / RETRY / REPLAN / SKIP)
        
        alt Phán quyết là APPROVE
            Sup->>UI: Cập nhật Task[i] trạng thái SUCCESS (Xanh)
        else Phán quyết là RETRY
            Sup->>Exec: Yêu cầu chạy lại Task[i] với chỉ dẫn sửa sai từ Critic
        else Phán quyết là REPLAN
            Sup->>Plan: Yêu cầu Planner điều chỉnh lại toàn bộ Plan (Sinh plan_v2)
            Plan-->>Sup: Cập nhật Plan mới
            Sup->>UI: Cập nhật lại Task List trên giao diện
        else Phán quyết là SKIP
            Sup->>UI: Đánh dấu Task[i] bị bỏ qua (Vàng) kèm lý do
        end
    end
```

#### Hai chế độ Browser Backend:
1. **`agentcore` (Production / Demo):**
   - Trình duyệt điều khiển từ xa trên đám mây AWS thông qua **AWS AgentCore Browser**.
   - Kết nối bằng giao thức Chrome DevTools Protocol (CDP) có ký chứng thực **AWS SigV4**.
   - Hiển thị trực tiếp lên Web UI thông qua iframe **DCV Web Viewer** độ phân giải full HD (1920×1080).
   - Hỗ trợ tính năng **"Take Control"**: Người dùng có thể bấm nút để trực tiếp can thiệp chuột/phím vào trình duyệt, sau đó trả quyền lại cho Agent.
2. **`playwright_local` (Dev / Local CI):**
   - Chạy Playwright headless Chromium ngay trong container/local.
   - Stream khung hình qua chuẩn **MJPEG** kèm lớp phủ (overlay) tương tác.

---

### Giai đoạn 4: Đánh Giá Chất Lượng GenAI & Tracing (MLflow Observability)

Bên cạnh luồng UI Browser, QA Cat còn vận hành song song luồng **RAGAS / MLflow Evaluation** để kiểm định Chatbot:
1. **Thu thập dữ liệu:** Đọc bộ câu hỏi kiểm thử từ file CSV/JSON hoặc trực tiếp từ tương tác của bot trên UI.
2. **Gọi Bedrock Judge Adapter (`bedrock_judge_adapter.py`):**
   - Khắc phục lỗi tương thích của MLflow Gateway với AWS Bedrock bằng cách định tuyến trực tiếp qua `boto3 Converse API`.
   - Chấm điểm 4 metric cốt lõi:
     - **Faithfulness:** Câu trả lời có căn cứ từ tài liệu KB không?
     - **Answer Relevancy:** Câu trả lời có giải quyết đúng trọng tâm câu hỏi không?
     - **Context Precision:** Các đoạn trích dẫn có chuẩn xác không?
     - **Context Recall:** Có bỏ sót thông tin quan trọng nào trong tài liệu không?
3. **Ghi nhận Tracing (Non-breaking Opt-in):**
   - Mọi lượt gọi LLM được ghi nhận span: Token input/output, Latency (p50/p90/p99), Cost ước tính.
   - Dữ liệu lưu tại `./qa-agent-data/mlflow.db`. Người dùng có thể theo dõi qua tab **Lab** trên giao diện hoặc gõ lệnh CLI `qa-agent mlflow` để mở MLflow UI đầy đủ.

---

## 3. Các Trạng Thái & Xử Lý Ngoại Lệ (Error Handling)

| Tình huống lỗi | Hành vi xử lý của hệ thống |
| :--- | :--- |
| **Element không tìm thấy trên DOM** | Executor tự động thử các chiến lược fallback: tìm theo text, aria-label, hoặc chụp screenshot dùng Vision để xác định tọa độ click. |
| **Task lặp lại quá số lần quy định** | Nếu Critic từ chối `RETRY` vượt quá `max_attempts` (mặc định 3 lần) $\rightarrow$ Tự động chuyển sang `REPLAN` hoặc đánh dấu `FAILED`. |
| **Mất kết nối WebSocket / Trình duyệt** | Supervisor giữ trạng thái trong SQLite. Khi người dùng refresh trang, UI tự động kết nối lại và khôi phục đúng bước đang chạy. |
| **Lỗi ghi Tracing MLflow** | Tự động degrade về no-op handle, tuyệt đối không làm gián đoạn luồng test chính của trình duyệt. |

---

## 4. Liên Kết Trong Obsidian Vault
- [[QA_Agent_QA_Cat|Hồ sơ Tổng quan Agent QA Cat]]
- [[Luong_Chay_Chi_Tiet_Pentest|Luồng Chạy Chi Tiết Hệ Thống Pentest]]
- [[TI_Integration_Architecture|Bản Thiết Kế Tích Hợp Tổng Thể vào TI]]
- [[QA_Cat_AWS_Detail.drawio|Sơ Đồ Kiến Trúc Draw.io Chi Tiết QA Cat]]
