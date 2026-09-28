# BẢN THIẾT KẾ KIẾN TRÚC CHI TIẾT HỆ THỐNG TESTING INTELLIGENCE (TI) V2.0
## ĐẶC TẢ KỸ THUẬT PHÂN TÍCH TỪ ĐẦU ĐẾN CUỐI THEO SƠ ĐỒ CHUẨN TI_SYSTEM_ARCHITECTURE.DRAWIO

> **Phiên bản:** v2.0.0-Master-Sync · **Ngày cập nhật:** 2026-09-28  
> **Căn cứ đối soát trực tiếp:** [TI_System_Architecture.drawio](file:///c:/Users/T14S/TI/TestIntelligent_doc/diagram/TI_System_Architecture.drawio)  
> **Hình ảnh sơ đồ kiến trúc:** [TI_System_Architecture.png](file:///c:/Users/T14S/TI/TestIntelligent_doc/diagram/TI_System_Architecture.png)  
> **Kỷ luật kiến trúc:** Bám sát 100% cấu trúc, ký hiệu, phân vùng và chuỗi mũi tên đánh số `[01]` đến `[12D]` từ sơ đồ Draw.io gốc; chuẩn hóa luồng `ImpactSet` đi qua AI Semantic Review tại Account B trước khi Smart Dispatch.

---

## MỤC LỤC
1. [TỔNG QUAN 4 PHÂN VÙNG KIẾN TRÚC THEO SƠ ĐỒ DRAWIO](#1-tổng-quan-4-phân-vùng-kiến-trúc-theo-sơ-đồ-drawio)
2. [PHÂN TÍCH CHI TIẾT LUỒNG DỮ LIỆU TUẦN TỰ TỪ ĐẦU ĐẾN CUỐI ([01] ĐẾN [12D])](#2-phân-tích-chi-tiết-luồng-dữ-liệu-tuần-tự-từ-đầu-đến-cuối-01-đến-12d)
3. [BÓC TÁCH CHI TIẾT 6 TRỤC RUNNER THỰC THI (SANDBOX EXECUTION VPC)](#3-bóc-tách-chi-tiết-6-trục-runner-thực-thi-sandbox-execution-vpc)
4. [SƠ ĐỒ KIẾN TRÚC HỆ THỐNG HIỆN TẠI (SYSTEM ARCHITECTURE DIAGRAM)](#4-sơ-đồ-kiến-trúc-hệ-thống-hiện-tại-system-architecture-diagram)
5. [BẢNG MA TRẬN ĐẶC TẢ KỸ THUẬT & FINOPS TỪNG BƯỚC ([01] -> [12D])](#5-bảng-ma-trận-đặc-tả-kỹ-thuật--finops-từng-bước-01---12d)
6. [HỢP ĐỒNG GIAO DIỆN DỮ LIỆU CỐT LÕI (TOOLINTENT & TENANTBINDING)](#6-hợp-đồng-giao-diện-dữ-liệu-cốt-lõi-toolintent--tenantbinding)
7. [LỘ TRÌNH TRIỂN KHAI THEO WAVE (W0 — W4) & GATES](#7-lộ-trình-triển-khai-theo-wave-w0--w4--gates)
8. [KẾT LUẬN](#8-kết-luận)

---

## 1. TỔNG QUAN 4 PHÂN VÙNG KIẾN TRÚC THEO SƠ ĐỒ DRAWIO

Toàn bộ hệ thống được chia thành 4 phân vùng độc lập, cách ly nghiêm ngặt về mạng, quyền truy cập và trách nhiệm xử lý:

```text
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 1. EXTERNAL CONSUMERS: Developer / QA · TI CLI · CI/CD Pipeline (GitHub Actions / GitLab CI)           │
└───────────────────────────────────────────────────┬────────────────────────────────────────────────────┘
                                                    │ [01] POST /v2/artifact-jobs
                                                    ▼
┌────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ 2. AWS ACCOUNT A — ap-southeast-1 (Control Plane & S08 Evidence Store)                                  │
│    • Edge Group: Amazon CloudFront CDN + AWS WAF (Managed Rules & HMAC Verify)                         │
│    • TI API v2: FastAPI (:8000) on AWS Fargate (Trả 202 Accepted < 1s)                                 │
│    • Job Controller: ECS Fargate Law 4.3 (State Machine, TenantBinding Resolver, Smart Dispatch)       │
│    • Job Store: Amazon RDS for PostgreSQL (db.t4g.micro ~$15/tháng)                                    │
│    • Evidence Store: Amazon S3 + Object Lock (WORM 90 ngày, SHA-256 Digest)                            │
│    • S09 Decision Gate: Deterministic Code Engine (PASS / HOLD / DO_NOT_PASS)                          │
│    • Web Portal: Next.js (:8001) Live Reporting Dashboard                                              │
│    • VPC Endpoints: 3 Interface Endpoints (ECR, Logs, STS ~$22/mo) + 1 S3 Gateway Endpoint ($0)        │
└───────────────────────┬───────────────────────────────────────────────┬────────────────────────────────┘
                        │ [08A] Context + SecurityFindings              │ [10] Smart Dispatch
                        │       (Cross-Account IAM)                     │      (Verified ImpactSet ∩ Binding)
                        ▼                                               ▼
┌──────────────────────────────────────────────┐ ┌───────────────────────────────────────────────────────┐
│ 3. AWS ACCOUNT B — us-east-1 (AI Brain)      │ │ 4. SANDBOX EXECUTION VPC — ap-southeast-1 (Domain D2) │
│    • AgentCore Harness: TIJobRunner          │ │    • Network: DENY ALL EGRESS (--network none)        │
│    • Bedrock Models:                         │ │    • ECS RunTask Port Launcher (IsolatedRunner Law 23)│
│      - Claude 5.0 Sonnet (S03 Impact Engine +│ │    • 6 Trục Runner Thực Thi Task-per-Job:             │
│        S05 Test Plan + S06 Candidate Gen)    │ │      - Trục 1: D5a Security (Semgrep+Trivy+Gitleaks)  │
│      - Claude Opus 5 (S04 Deep Risk & Threat)│ │      - Trục 2: API Functional & Fuzzing                │
│    • Amazon Bedrock Evaluations              │ │      - Trục 3: UI Web E2E & A11y (axe-core WCAG)      │
│      (Groundedness ≥0.80, Faithfulness ≥0.85)│ │      - Trục 4: DB Dual Isolation (Aurora + DynamoDB)  │
│    • AWS Secrets Manager & STS Tokens        │ │      - Trục 5: Performance Testing (k6 + 3 Khóa)      │
│    • Knowledge Base S10 Memory (GOLDEN only) │ │      - Trục 6: D5b DAST Task (W3: ZAP + nuclei)       │
└──────────────────────────────────────────────┘ └───────────────────────────────────────────────────────┘
```

* **Phân vùng 1 (External Consumers):** Tác nhân kích hoạt kiểm thử từ bên ngoài (Lập trình viên, QA, CI/CD Pipeline).
* **Phân vùng 2 (AWS Account A — Singapore):** Tầng điều khiển trung tâm, giữ sổ cái trạng thái công việc (Job Store) và lưu trữ bằng chứng kiểm thử bất biến (Evidence Store).
* **Phân vùng 3 (AWS Account B — N. Virginia):** Tầng suy luận trí tuệ nhân tạo, đảm nhiệm **AI Semantic Review (S03 Impact Engine & S04 Risk Engine)** để xác định chính xác phạm vi lan truyền tác động (`ImpactSet`), phân tầng mô hình Claude Sonnet/Opus và giám định chất lượng kiểm thử độc lập chống ảo giác.
* **Phân vùng 4 (Sandbox Execution VPC — Singapore):** Tầng thực thi cô lập microVM Fargate, áp dụng chính sách mạng cấm ra ngoài Internet (`DENY ALL EGRESS`) để chạy 6 Trục Runner an toàn tuyệt đối.

---

## 2. PHÂN TÍCH CHI TIẾT LUỒNG DỮ LIỆU TUẦN TỰ TỪ ĐẦU ĐẾN CUỐI ([01] ĐẾN [12D])

Tiến trình vận hành trên sơ đồ Draw.io đi qua 12 nhóm bước chuẩn tắc, đánh số rõ ràng từ điểm vào ngoại vi đến điểm kết thúc:

### `[01]` Khởi Tạo & Gửi Yêu Cầu (External ➔ CloudFront)
* **Caller:** CI/CD Pipeline (GitHub Actions / GitLab CI) hoặc Developer qua TI CLI.
* **Hành động:** Khi có Pull Request mới, caller đóng gói metadata changeset (Git diff, OpenAPI spec, Flyway migration SQL) kèm chữ ký số SHA-256 digest và gửi yêu cầu `POST /v2/artifact-jobs` tới CloudFront.

### `[02]` Thẩm Định Biên & Chuyển Tiếp HTTPS (CloudFront ➔ WAF ➔ TI API v2)
* **Thành phần:** Amazon CloudFront + AWS WAF.
* **Hành động:** CloudFront chuyển tiếp lưu lượng qua AWS WAF để kiểm tra chữ ký HMAC, ngăn chặn tấn công OWASP Top 10 và áp dụng Rate Limiting. Request hợp lệ được đưa vào cổng `:8000` của **TI API v2** chạy trên AWS Fargate.

### `[03]` Phản Hồi Bất Đồng Bộ Cực Nhanh (TI API v2 ➔ Caller)
* **Thành phần:** TI API v2 (FastAPI Engine).
* **Hành động:** Validate schema của artifact và digest SHA-256. Trả ngay phản hồi HTTP `202 Accepted` kèm `job_id` trong thời gian **< 1 giây**, giải phóng hoàn toàn tiến trình CI/CD runner mà không bắt pipeline phải chờ đợi.

### `[04]` Khởi Tạo Trạng Thái PENDING (TI API v2 ➔ RDS PostgreSQL)
* **Thành phần:** TI API v2 ➔ Amazon RDS for PostgreSQL (`rds_pg`).
* **Hành động:** Ghi nhận bản ghi Job mới vào cơ sở dữ liệu với trạng thái khởi tạo `PENDING` kèm thông tin tenant, commit SHA và audit metadata.

### `[05]` Đưa Job Vào Hàng Đợi Điều Phối (TI API v2 ➔ Job Controller)
* **Thành phần:** TI API v2 ➔ Job Controller (`jc_box`).
* **Hành động:** Đưa `job_id` và ngữ cảnh artifact vào máy trạng thái của Job Controller, chuyển trạng thái Job sang `RUNNING`.

### `[06]` Kích Hoạt Quét An Ninh Tĩnh D5a (Job Controller ➔ Trục 1 Sandbox)
* **Thành phần:** Job Controller ➔ `r1_box` (Trục 1: D5a Security trong Sandbox VPC).
* **Hành động:** Kích hoạt container Fargate task-per-job chạy song song bộ ba công cụ:
  * **Semgrep OSS:** Quét SAST trực tiếp trên Git diff (thời gian cực nhanh 2–5 giây).
  * **Trivy:** Quét danh mục thư viện phụ thuộc (SCA) phát hiện mã CVE.
  * **Gitleaks:** Quét phát hiện rò rỉ Secrets, Passwords, API Keys trong changeset.

### `[07A]` & `[07B]` Xuất Báo Cáo SARIF & Bàn Giao SecurityFindings (Trục 1 ➔ S3 & Job Controller)
* **`[07A] SARIF → S3`:** Trục 1 xuất trực tiếp tệp báo cáo chuẩn hóa **SARIF** lên **Amazon S3 Evidence Store** qua S3 Gateway Endpoint miễn phí ($0) theo cơ chế Direct-to-S3 Offloading.
* **`[07B] SecurityFindings`:** Trục 1 gửi danh sách phát hiện an ninh tĩnh (`SecurityFindings`: danh mục lỗ hổng SAST, CVE packages và secret leaks) về cho Job Controller.
  * *Lưu ý kiến trúc:* Trục 1 là công cụ quét tĩnh theo pattern-matching, **không tự ý suy diễn `ImpactSet` nghiệp vụ**. Trục 1 chỉ cung cấp dữ liệu bằng chứng an ninh thô cho bước tiếp theo.

### `[08A]` & `[08B]` AI Semantic Review: Tính Toán Blast Radius & ImpactSet (Job Controller ➔ Account B Bedrock)
* **`[08A] Context + SecurityFindings`:** Job Controller chuyển giao toàn bộ ngữ cảnh PR Changeset (Git diff, OpenAPI spec, SQL migration) kèm theo `SecurityFindings` từ Trục 1 sang **AgentCore Harness (TIJobRunner)** tại Account B (`us-east-1`) qua IAM STS AssumeRole ngắn hạn.
* **`[08B] AI Review: Blast Radius & ImpactSet (Sonnet / Opus if CRITICAL)`:**
  * **Claude 5.0 Sonnet** đảm nhiệm **AI Semantic Code Review (S03 Impact Engine)**: Phân tích AST, Call Graph và Dependency Graph để xác định phạm vi lan truyền tác động thực sự, từ đó sinh ra đối tượng chuẩn tắc **`ImpactSet`** (chỉ rõ chính xác các domain và endpoints bị ảnh hưởng: API, UI, Database, hay Performance).
  * **S04 Risk Engine:** Đánh giá mức độ rủi ro tổng hợp từ `SecurityFindings` (D5a) và độ phức tạp mã nguồn để gán nhãn `Risk Tier` (`LOW`, `MEDIUM`, `HIGH`, hoặc `CRITICAL`). Nếu phát hiện lỗ hổng nghiêm trọng hoặc rò rỉ secret, kích hoạt bổ sung **Claude Opus 5** để thực hiện AI Threat Modeling chuyên sâu.
  * Sinh kế hoạch kiểm thử (**S05 Test Planning**) và kịch bản candidate (**S06 Candidate Generation**) bám sát chính xác `ImpactSet` vừa sinh.

### `[09A]`, `[09B]` & `[09C]` Giám Định Kép & Trả Về Verified ImpactSet (Bedrock ➔ Evaluations ➔ S3 & Controller)
* **`[09A] ImpactSet + TestPlan + Cases`:** Claude Sonnet gửi `ImpactSet`, kế hoạch kiểm thử và candidate test cases sang **Amazon Bedrock Evaluations** (Module `TIRunnerGroundness`).
* **`[09B] Eval OK → S3`:** Bedrock Evaluations chạy thuật toán độc lập với tham số cố định `temperature: 0.0` (Greedy Decoding) để đo:
  * $\mathbf{GroundednessScore \ge 0.80}$ (chống AI ảo giác, tự bịa API/Field không tồn tại).
  * $\mathbf{FaithfulnessScore \ge 0.85}$ (trung thực với logic nghiệp vụ).
  * Các kịch bản đạt chuẩn được lưu thẳng vào S3 Evidence Store.
* **`[09C] Quality OK + Verified ImpactSet`:** Bedrock Evaluations gửi tín hiệu xác nhận chất lượng AI hợp lệ kèm theo **`Verified ImpactSet`** và `Risk Tier` về `TenantBinding Resolver` của Job Controller (Account A).

### `[10]` Điều Phối Thông Minh Kích Hoạt Runners (Job Controller ➔ Port Launcher ➔ Runners)
* **Thành phần:** Job Controller ➔ `port_launcher` ➔ Cụm 6 Trục Runner trong Sandbox VPC.
* **Công thức Smart Dispatching:** $\mathbf{Target = Verified\ ImpactSet \cap TargetBinding}$.
* **Hành động:** Job Controller chỉ gọi ECS RunTask (`IsolatedRunner` Law 23) để kích hoạt **chính xác các Trục Runner nằm trong `Verified ImpactSet` do AI xác định**. Các miền không bị tác động được đánh dấu `SKIPPED` ngay lập tức, tiết kiệm 100% tài nguyên compute.

### `[11A]` & `[11B]` Cơ Chế Direct-to-S3 Offloading & Envelope Metadata (Runners ➔ S3 & Controller)
* **`[11A] Direct-to-S3 Offloading` (Law 16):** Toàn bộ dữ liệu artifact nặng (ảnh PNG, video MP4, Playwright traces, log k6, báo cáo DAST) được các Runner đẩy thẳng lên S3 Object Lock qua S3 Gateway Endpoint ($0).
* **`[11B] Envelope ~2KB`:** Runner chỉ gửi về Job Controller một **JSON Metadata Envelope siêu nhẹ (~2 KB)** chứa exit code, S3 URI và mã băm SHA-256 Digest $\rightarrow$ Triệt tiêu 100% rủi ro nghẽn I/O và tràn ổ đĩa backend.

### `[12A]` ➔ `[12D]` Phán Quyết Gate, Cập Nhật Trạng Thái & Hoàn Tất
* **`[12A] Metrics+SHA256`:** Job Controller chuyển envelope và metrics sang **S09 Decision Gate**.
* **`[12B] Update state`:** Decision Gate chạy mã **Deterministic Code Engine** đối soát luật ISTQB cứng (`Critical == 0 & p95 < SLA & Groundedness >= 0.80 & Faithfulness >= 0.85`), cập nhật phán quyết (`PASS`, `HOLD`, `DO_NOT_PASS`) vào RDS PostgreSQL.
* **`[12C] GOLDEN → Memory`:** Nếu phán quyết là `PASS`, test case hợp lệ được tự động index vào Bedrock Knowledge Base (S10 Memory) để tái sử dụng (Few-shot Learning).
* **`[12D] Poll result`:** CI/CD pipeline thăm dò kết quả từ RDS/API để quyết định cho phép hoặc chặn merge Pull Request. Đồng thời, Developer/QA có thể truy cập Web Portal `:8001` xem live replay và tải trọn gói audit evidence.

---

## 3. BÓC TÁCH CHI TIẾT 6 TRỤC RUNNER THỰC THI (SANDBOX EXECUTION VPC)

Mỗi trục là một container Fargate task-per-job độc lập, chạy trong Private Subnet với rào chắn mạng **DENY ALL EGRESS** (`--network none`):

```text
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│ SANDBOX EXECUTION VPC (Domain D2 · DENY ALL EGRESS · --network none · 3 VPC Endpoints ~$22/mo · S3 Gateway $0)  │
├─────────────────┬─────────────────┬─────────────────┬─────────────────┬───────────────────┬─────────────────────┤
│ TRỤC 1: D5a     │ TRỤC 2: API     │ TRỤC 3: UI WEB  │ TRỤC 4: DB DUAL │ TRỤC 5: PERF      │ TRỤC 6: D5b DAST    │
│ SECURITY (W1)   │ FUNCTIONAL (W1) │ & A11Y (W2)     │ ISOLATION (W1/2)│ TESTING (W2)      │ TASK (W3)           │
├─────────────────┼─────────────────┼─────────────────┼─────────────────┼───────────────────┼─────────────────────┤
│ • Semgrep OSS   │ • Schemathesis  │ • Playwright    │ • Aurora v2     │ • AWS DLT         │ • OWASP ZAP         │
│   (SAST 2-5s)   │   (OpenAPI Fuzz)│   (Headless Cr) │   Clone (<60s)  │   (Fargate Orch)  │   Active Scan       │
│ • Trivy         │ • Playwright    │ • axe-core      │ • DynamoDB      │ • k6 Engine       │ • nuclei            │
│   (CVE Packages)│   API (E2E Flow)│   (WCAG 2.1 AA) │   Local / TTL 1h│   (Internal ALB)  │   Template scan     │
│ • Gitleaks      │                 │                 │ • Hook          │ • Bộ 3 Khóa       │ ⚠ Cần Staging URL   │
│   (Secrets/Keys)│                 │                 │   DeleteTable   │   An Toàn         │   sống mới kích hoạt│
└─────────────────┴─────────────────┴─────────────────┴─────────────────┴───────────────────┴─────────────────────┘
```

1. **Trục 1: D5a Security Task (`r1_box`):** `Semgrep OSS` (quét AST Git diff 2–5s) + `Trivy` (quét CVE phụ thuộc) + `Gitleaks` (quét secrets). Xuất file chuẩn SARIF lên S3 (`[07A]`) và gửi `SecurityFindings` về Controller (`[07B]`).
2. **Trục 2: API Functional & Fuzzing (`r2_box`):** `Schemathesis` (fuzzing hàng nghìn payload biên từ OpenAPI spec) + `Playwright API` (thực thi luồng nghiệp vụ E2E có trạng thái).
3. **Trục 3: UI Web E2E & Accessibility (`r3_box`):** `Playwright Headless Chromium` (chạy kịch bản người dùng E2E) + `axe-core Engine` (nhúng trình duyệt quét tiêu chuẩn tiếp cận WCAG 2.1 AA, tỷ lệ báo sai bằng 0).
4. **Trục 4: Database Dual Isolation (`r4_box`):** `Amazon Aurora Serverless v2 Clone` (clone copy-on-write trong < 60s để test Flyway migration SQL, sau đó tự hủy) + `DynamoDB Local / Ephemeral Table` (bảng tạm tự hủy TTL 1 giờ).
5. **Trục 5: Performance Testing (`r5_box`):** `AWS DLT` + `k6 Engine` bắn tải trực tiếp vào Internal ALB qua mạng nội bộ. Bắt buộc tuân thủ **Bộ 3 Khóa An Toàn**: Ephemeral DB riêng, trần cứng tối đa 500 VUs trong 10 phút, và Circuit Breaker ngắt khẩn cấp nếu lỗi HTTP 5xx > 2%.
6. **Trục 6: D5b DAST Task (`r6_box`):** `OWASP ZAP Active Scan` + `nuclei`. **Chỉ kích hoạt tại Wave 3 khi có Staging URL sống** sau khi PR đã deploy thành công, quét an ninh động tìm SQLi, XSS, CSRF.

---

## 4. SƠ ĐỒ KIẾN TRÚC HỆ THỐNG HIỆN TẠI (SYSTEM ARCHITECTURE DIAGRAM)

### 4.1. Hình ảnh Sơ đồ Kiến trúc Chuẩn (Draw.io Export)

Sơ đồ được thiết kế và đồng bộ chuẩn xác tại tệp gốc [`TI_System_Architecture.drawio`](file:///c:/Users/T14S/TI/TestIntelligent_doc/diagram/TI_System_Architecture.drawio):

![Testing Intelligence System Architecture v2.0](file:///c:/Users/T14S/TI/TestIntelligent_doc/diagram/TI_System_Architecture.png)

### 4.2. Bản Đồ Trực Quan Các Phân Vùng & Chuỗi Kết Nối [01] ➔ [12D]

Bản đồ dưới đây mô phỏng chính xác cấu trúc hình học và chuỗi luồng đi qua AI Semantic Review:

```text
========================================================================================================================
[EXTERNAL CONSUMERS] (zone_ext)
Developer / QA  ·  TI CLI  ·  CI/CD Settings / Runner
       │
       ▼ [01] POST /v2/artifact-jobs (Changeset, OpenAPI, Digest SHA-256)
========================================================================================================================
[AWS ACCOUNT A: ap-southeast-1 Singapore] (zone_acct_a) — CONTROL PLANE & EVIDENCE STORE
  CloudFront (CDN) ──► WAF ──► [02] TI API v2 (:8000) ──► [03] 202 Accepted (<1s) ──► Caller
                                     │
                  ┌──────────────────┴──────────────────┐
                  ▼ [04] PENDING bản ghi                ▼ [05] Enqueue job
             RDS PostgreSQL                        Job Controller (jc_box)
             (rds_pg: db.t4g.micro)                  ├─ jc_ecs (State Machine)
                                                     ├─ jc_resolver (TenantBinding & Smart Dispatch)
                                                     └─ jc_lease (Worker Lease)
                                                        │
       ┌────────────────────────────────────────────────┼────────────────────────────────────────┐
       ▼ [06] D5a Security                              ▼ [08A] Context + SecurityFindings       ▼ [10] Smart Dispatch
=========================================  =============================================  ==============================
[SANDBOX EXECUTION VPC] (sandbox)          [AWS ACCOUNT B: us-east-1] (zone_acct_b)       [PORT LAUNCHER] (port_launcher)
Domain D2 · DENY ALL EGRESS                 AGENTCORE AI BRAIN                             ECS RunTask (Law 23)
Private Subnet · 3 VPCE + S3 Gateway $0      AgentCore Harness (agentcore)                  │
                                              │ [08B] AI Review: Blast Radius & ImpactSet   ▼
  Trục 1: D5a Security (r1_box)               Bedrock Models (Sonnet + Opus if CRITICAL)   CỤM 6 TRỤC RUNNER SANDBOX:
    • Semgrep OSS (SAST 2-5s)                 │ [09A] ImpactSet + TestPlan + Cases           • Trục 1: D5a Security
    • Trivy (CVE Packages)                    Bedrock Evaluations (bedrock_eval)             • Trục 2: API Fuzzing
    • Gitleaks (Secrets)                      (Groundedness ≥0.80, Faith. ≥0.85)             • Trục 3: UI Web & axe A11y
       │──► [07A] SARIF ──► S3 Evidence               │──► [09B] Eval OK ──► S3 Evidence     • Trục 4: Aurora v2 Clone
       └──► [07B] SecurityFindings ──► Controller     └──► [09C] Quality OK + Verified       • Trục 5: Perf k6 (3 Khóa)
                                                                 ImpactSet ──► Controller    • Trục 6: D5b DAST (ZAP)
                                                                                             │
                                                                                             ├──► [11A] Direct-to-S3 ($0)
                                                                                             └──► [11B] Envelope ~2KB ──┐
                                                                                                                        │
========================================================================================================================│
[HOÀN TẤT & PHÁN QUYẾT GATE] (Account A)                                                                                │
                                                                                                                        │
   S09 Decision Gate (s09_gate) ◄───────────────── [12A] Metrics + SHA-256 Digest ◄─────────────────────────────────────┘
   (Deterministic Code Engine: Critical==0, p95<SLA, Groundedness≥0.80)
     │
     ├────► [12B] Update State ──────────► RDS PostgreSQL (COMPLETED + PASS / HOLD / DO_NOT_PASS)
     │                                        │
     │                                        └─► [12D] Poll Result ──► CI/CD Pipeline (Merge / Block PR)
     │
     ├────► [12C] GOLDEN Test Cases ─────► S10 Knowledge Base Memory (Account B: bedrock_kb)
     │
     └────► Live Report & Video Trace ───► Web Portal (:8001 Next.js Dashboard) ──► Developer / QA
========================================================================================================================
```

---

## 5. BẢNG MA TRẬN ĐẶC TẢ KỸ THUẬT & FINOPS TỪNG BƯỚC ([01] -> [12D])

| Bước | Tên Bước Trên Draw.io | Thành phần AWS | Công nghệ / Tool | Dữ Liệu Trao Đổi & Bằng Chứng | SLA | Chi phí FinOps Ước tính |
| :---: | :--- | :--- | :--- | :--- | :---: | :---: |
| **`[01]`** | **POST Artifact Jobs** | Amazon CloudFront | HTTPS TLS 1.3 / ACM | Payload Changeset + SHA-256 Digest | < 50ms | $0.085/GB data transfer |
| **`[02]`** | **Forward qua WAF** | AWS WAF + CloudFront | AWS WAF Managed Rules | Request an toàn được gắn header xác thực | < 30ms | Đã bao gồm trong WAF |
| **`[03]`** | **202 Accepted <1s** | TI API v2 (ECS Fargate) | `FastAPI`, `uvicorn` | HTTP 202 kèm `{job_id, poll_url}` | **< 1s** | Cụm Fargate Control Plane |
| **`[04]`** | **PENDING to RDS** | Amazon RDS PostgreSQL | `asyncpg`, `SQLAlchemy` | Bản ghi trạng thái ban đầu `PENDING` | < 50ms | Cố định ~$15/tháng (db.t4g.micro) |
| **`[05]`** | **Enqueue Job** | TI API v2 ➔ Job Controller | In-VPC Private Service | Bàn giao ngữ cảnh `job_id` | < 10ms | Nội bộ VPC ($0) |
| **`[06]`** | **D5a Security Task** | Job Controller ➔ Trục 1 | **Semgrep + Trivy + Gitleaks** | Kích hoạt quét tĩnh PR-time trong Sandbox | **2 – 5s** | Compute Fargate per-second |
| **`[07A]`**| **SARIF → S3** | Trục 1 ➔ Amazon S3 | S3 Gateway Endpoint ($0) | Báo cáo chuẩn **SARIF** (CWE, CVE, Secrets) | < 500ms | S3 Standard $0.023/GB |
| **`[07B]`**| **SecurityFindings** | Trục 1 ➔ Job Controller | In-VPC JSON | Danh sách phát hiện lỗ hổng SAST/CVE/Secrets | < 100ms | Nội bộ VPC ($0) |
| **`[08A]`**| **Context + SecurityFindings** | Job Controller ➔ AgentCore | AWS STS AssumeRole | Changeset + SecurityFindings + Token Budget | < 200ms | Miễn phí IAM liên account |
| **`[08B]`**| **AI Review: Impact & TestGen**| AgentCore ➔ Bedrock | **Claude 5.0 Sonnet** (Opus nếu CRITICAL) | S03 Blast Radius ➔ `ImpactSet` + Test Plan | **5 – 15s** | Multi-turn: ~$0.045 – $0.085/job |
| **`[09A]`**| **ImpactSet + TestPlan + Cases**| Bedrock ➔ Evaluations | Bedrock Evaluations API | Chuyển `ImpactSet` + Candidate Test Cases | < 100ms | Nội bộ Bedrock |
| **`[09B]`**| **Eval OK → S3** | Evaluations ➔ S3 Evidence | S3 Gateway Endpoint | Lưu JSON TestPlan/TestCases đạt chuẩn | < 500ms | S3 Standard $0.023/GB |
| **`[09C]`**| **Quality OK + Verified ImpactSet**| Evaluations ➔ Job Controller | Cross-account API | Báo cáo Groundedness ≥0.80 + Verified `ImpactSet` | < 500ms | In-process ($0) |
| **`[10]`** | **Smart Dispatch** | Controller ➔ Port Launcher | ECS RunTask (`IsolatedRunner`) | Lệnh RunTask: `Verified ImpactSet ∩ TargetBinding` | < 1s | In-process ($0) |
| **`[11A]`**| **Direct-to-S3** | 6 Runners ➔ S3 Evidence | S3 Gateway Endpoint ($0) | Raw Artifacts (PNG, Video, Traces, Logs, DAST) | < 2s | S3 Object Lock Compliance |
| **`[11B]`**| **Envelope ~2KB** | 6 Runners ➔ Job Controller | Private Link REST / JSON | JSON Metadata Envelope (~2 KB, SHA-256) | < 100ms | Xóa sổ 100% rủi ro ngộp ổ EBS |
| **`[12A]`**| **Metrics+SHA256** | Controller ➔ Decision Gate | In-process Engine | Bảng tổng hợp chỉ số kiểm thử | < 50ms | In-process ($0) |
| **`[12B]`**| **Update State** | Decision Gate ➔ RDS | PostgreSQL Transaction | Cập nhật `COMPLETED` + `PASS / HOLD / DO_NOT_PASS` | < 50ms | Cố định RDS |
| **`[12C]`**| **GOLDEN → Memory** | Decision Gate ➔ Knowledge Base | Bedrock Knowledge Base API | Lưu vector embedding testcase verified | < 1s | Vector storage Bedrock |
| **`[12D]`**| **Poll Result** | RDS ➔ CI/CD Pipeline | GET /v2/artifact-jobs/{id} | JSON phán quyết Gate Check hoàn chỉnh | < 100ms | HTTP outbound traffic |

---

## 6. HỢP ĐỒNG GIAO DIỆN DỮ LIỆU CỐT LÕI (TOOLINTENT & TENANTBINDING)

### 6.1. Hợp đồng ToolIntent JSON (AgentCore AI Intelligence ➔ Job Controller - Law 10.1)
Mô hình AI chỉ phát các tham số biểu tượng (Symbolic Params), tuyệt đối không chứa password, token hay URL nhạy cảm:

```json
{
  "$schema": "https://ti.internal/schemas/tool-intent-v2.json",
  "intent_id": "intent_8f9a0b1c-2345-6789-bcde-fa0123456789",
  "job_id": "job_4a5b6c7d-0987-6543-edcb-0123456789ab",
  "domain": "API_TESTING",
  "action": "EXECUTE_TEST_PACK",
  "target_binding_ref": "binding_crm_staging_api",
  "execution_plan": {
    "pack_id": "pkg_schemathesis_openapi_v1",
    "test_suite_ref": "candidate_suite_991823",
    "allowed_methods": ["GET", "POST", "PUT"],
    "rate_limit_rps": 15,
    "timeout_seconds": 180
  },
  "constraints": {
    "egress_mode": "RESTRICTED_SANDBOX",
    "risk_tier": "HIGH",
    "budget_ceiling_usd": 0.35
  }
}
```

### 6.2. Hợp đồng TenantBinding Server-Side (Job Controller ➔ Runner Container - Law 13)
Được tra cứu và giải mã hoàn toàn phía server bảo mật (Account A), dữ liệu được inject trực tiếp vào Fargate Task qua biến môi trường ngắn hạn:

```json
{
  "binding_id": "binding_crm_staging_api",
  "tenant_id": "tenant_enterprise_core",
  "resolved_endpoint": "https://staging-api.internal.company.com/v1",
  "auth_type": "AWS_STS_SHORT_LIVED",
  "role_arn": "arn:aws:iam::123456789012:role/TITestRunnerIsolatedRole",
  "allowed_ip_range": "10.100.20.0/24",
  "credential_secret_ref": "arn:aws:secretsmanager:ap-southeast-1:123456789012:secret:tenant_core_token-7x1a"
}
```

---

## 7. LỘ TRÌNH TRIỂN KHAI THEO WAVE (W0 — W4) & GATES

| Lộ trình | Miền Kiểm Thử Bao Phủ | Trục Runner Tương Ứng Trên Draw.io | Gate Kiểm Soát | Mục Tiêu Nghiệm Thu |
| :---: | :--- | :--- | :---: | :--- |
| **Wave W0** | **L0**: Contract / Linting | Củng cố TI API v2, kiểm tra artifact digest & Schema validation | **Gate G3** | Tiếp nhận webhook & trả HTTP 202 < 1s đạt chuẩn |
| **Wave W1** | **L1**: Unit Testing<br>**L2**: API Testing<br>**L6 (SAST)**: Static Security | • **Trục 1:** D5a Security (`Semgrep OSS + Trivy + Gitleaks`).<br>• **Trục 2:** API Runner (`Schemathesis + Playwright API`).<br>• **Trục 4:** Database Migration SQL (`Aurora v2 Clone`). | **Gate G4** | Quét SAST diff 2-5s, API fuzzing và migration SQL ổn định |
| **Wave W2** | **L3**: UI Web E2E<br>**L8**: Accessibility (a11y)<br>**L5**: Performance Testing | • **Trục 3:** UI Runner (`Playwright Headless + axe-core WCAG`).<br>• **Trục 5:** Performance Runner (`AWS DLT + k6 + 3 Khóa An Toàn`).<br>• **Trục 4:** Bổ sung NoSQL (`DynamoDB Local / TTL 1h`). | **Gate G4–G5** | Chạy E2E Browser trơn tru, tải k6 qua Internal ALB an toàn |
| **Wave W3** | **L4**: Mobile Testing<br>**L6 (DAST)**: Dynamic Security<br>**L11**: Infra Testing | • **Trục 6:** D5b DAST Task (`OWASP ZAP Active Scan + nuclei`).<br>• Tích hợp AWS Device Farm cho Mobile & Checkov quét IaC. | **Gate G5** | Kích hoạt quét DAST trên Staging URL sống đạt chuẩn |
| **Wave W4** | **L7**: Chaos Testing<br>**L9**: Data Quality | • Chaos Engineering (AWS FIS).<br>• Great Expectations kiểm tra chất lượng dữ liệu. | **Gate G6** | Sẵn sàng Cutover Production toàn diện |

---

## 8. KẾT LUẬN

Tài liệu đặc tả kiến trúc chi tiết v2.0 này đã đồng bộ hoàn hảo với sơ đồ [TI_System_Architecture.drawio](file:///c:/Users/T14S/TI/TestIntelligent_doc/diagram/TI_System_Architecture.drawio):
1. **Khớp 1-1 với sơ đồ:** Bám sát tuần tự từ External $\rightarrow$ Account A (Control Plane) $\rightarrow$ Account B (AI Brain) $\rightarrow$ Sandbox Execution VPC (6 Trục Runner).
2. **Logic ImpactSet chuẩn xác:** Khắc phục triệt để lỗ hổng bỏ qua AI Review; Trục 1 chỉ trả về `SecurityFindings`, AI tại Account B đảm nhiệm **S03 Impact Analysis** sinh `Verified ImpactSet` trước khi Smart Dispatch.
3. **Minh bạch luồng dữ liệu:** Định danh chính xác từng mũi tên từ `[01]` đến `[12D]`, làm rõ cơ chế **Direct-to-S3 Offloading** chống tràn ổ đĩa và ranh giới mạng Sandbox DENY ALL EGRESS.
4. **Cô đọng, thực chiến:** Nhúng trực tiếp sơ đồ kiến trúc hiện tại, tập trung tuyệt đối vào giải pháp kỹ thuật và thông số vận hành.
