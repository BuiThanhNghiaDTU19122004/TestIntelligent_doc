# TÀI LIỆU ĐẶC TẢ KIẾN TRÚC TỔNG THỂ HỆ THỐNG TESTING INTELLIGENCE (TI)
## BẢN THIẾT KẾ HỢP NHẤT HẠ TẦNG CLOUD, CƠ CHẾ PHÂN TẦNG AI VÀ KHUNG ĐIỀU PHỐI KIỂM THỬ ĐA MIỀN
### Master Architecture Blueprint — Version 1.0 (Sign-off Ready)

---

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Hệ thống** | Testing Intelligence (TI) Platform — Enterprise Test Execution & Evaluation Spine |
| **Ngày hoàn thiện** | 25 tháng 9 năm 2026 |
| **Các bên đóng góp** | • **Hà Tây Nguyên** (DevOps): Triết lý Ports & Adapters, Plugin & Interface Abstraction.<br>• **Hoàng** (Task 2 — Architecture) & **Trang** (Task 1 — Tooling): Hạ tầng AWS 2 Accounts, Fargate Sandbox, VPC Endpoints & FinOps.<br>• **Nghĩa** (Task 3 — AI Model) & **Hùng** (Task 4 — QA Strategy): Chuỗi S01–S10, Model Tiering, ToolIntent Handshake, Tiêu chuẩn ISTQB CT-AI/GenAI. |
| **Phê duyệt bởi** | Tan.Thai (Product Architect) · Anh Quang (Project Lead / Delivery) |
| **Trạng thái tài liệu** | 🟢 **APPROVED / MASTER BLUEPRINT** *(Duyệt thiết kế kiến trúc đích CANDIDATE; KHÔNG đồng nghĩa nghiệm thu hệ thống — theo dõi cổng #97. Hiện trạng đo 22/09 ghim 8a61cf66: 1 EC2 chứa cả API :8000 + Portal :8001)* |

---

## MỤC LỤC

1. [TỔNG QUAN CHIẾN LƯỢC & NGUYÊN TẮC HỢP NHẤT](#1-tổng-quan-chiến-lược--nguyên-tắc-hợp-nhất)
2. [MÔ HÌNH HỢP NHẤT 3 TẦNG KIẾN TRÚC TI (3-TIER ARCHITECTURAL FRAMEWORK)](#2-mô-hình-hợp-nhất-3-tầng-kiến-trúc-ti-3-tier-architectural-framework)
   - [2.1. Tầng Trừu Tượng Hóa & Nguyên Lý Kiến Trúc: Ports & Adapters và Interface IsolatedRunner (Law 23)](#21-tầng-trừu-tượng-hóa--nguyên-lý-kiến-trúc-ports--adapters-và-interface-isolatedrunner-law-23)
   - [2.2. Tầng Hạ Tầng Điện Toán Đám Mây & An Ninh Mạng: AWS 2 Accounts, Fargate Sandbox & Tối ưu Mạng D9](#22-tầng-hạ-tầng-điện-toán-đám-mây--an-ninh-mạng-aws-2-accounts-fargate-sandbox--tối-ưu-mạng-d9)
   - [2.3. Tầng Trí Tuệ Nhân Tạo & Điều Phối Đánh Giá Nghiệp Vụ: Chuỗi S01–S10, Bedrock Model Tiering & ToolIntent Handshake](#23-tầng-trí-tuệ-nhân-tạo--điều-phối-đánh-giá-nghiệp-vụ-chuỗi-s01s10-bedrock-model-tiering--toolintent-handshake)
3. [SƠ ĐỒ KIẾN TRÚC TỔNG THỂ (C4 MODEL DIAGRAMS)](#3-sơ-đồ-kiến-trúc-tổng-thể-c4-model-diagrams)
   - [3.1. Sơ đồ C4 Level 2: Container & Deployment Topology (Toàn cảnh 3 vùng)](#31-sơ-đồ-c4-level-2-container--deployment-topology-toàn-cảnh-3-vùng)
   - [3.2. Sơ đồ C4 Level 3: Zoom sâu bên trong Job Controller (Workflow Authority)](#32-sơ-đồ-c4-level-3-zoom-sâu-bên-trong-job-controller-workflow-authority)
   - [3.3. Sơ đồ Sequence: Vòng đời xử lý một Job từ Tiếp nhận đến Gate S09](#33-sơ-đồ-sequence-vòng-đời-xử-lý-một-job-từ-tiếp-nhận-đến-gate-s09)
4. [CHI TIẾT CÁC MIỀN THIẾT KẾ KỸ THUẬT (DOMAIN SPECIFICATIONS)](#4-chi-tiết-các-miền-thiết-kế-kỹ-thuật-domain-specifications)
   - [4.1. Domain D2: Fargate Sandbox Task-per-Job](#41-domain-d2-fargate-sandbox-task-per-job)
   - [4.2. Domain D2.b: Database Testing với Aurora Serverless v2 Clone](#42-domain-d2b-database-testing-với-aurora-serverless-v2-clone)
   - [4.3. Domain D5a & D5b: Chiến lược An ninh Đa tầng (Hybrid Defense)](#43-domain-d5a--d5b-chiến-lược-an-ninh-đa-tầng-hybrid-defense)
   - [4.4. Domain D9: Tối ưu Hóa Chi phí Mạng Egress Sandbox](#44-domain-d9-tối-ưu-hóa-chi-phí-mạng-egress-sandbox)
5. [HỢP ĐỒNG GIAO DIỆN DỮ LIỆU (CONTRACT SPECIFICATIONS)](#5-hợp-đồng-giao-diện-dữ-liệu-contract-specifications)
   - [5.1. Cấu trúc ToolIntent JSON (AgentCore $\rightarrow$ Job Controller)](#51-cấu-trúc-toolintent-json-agentcore--job-controller)
   - [5.2. Cấu trúc TenantBinding Server-Side (Job Controller $\rightarrow$ Runner)](#52-cấu-trúc-tenantbinding-server-side-job-controller--runner)
6. [BÀI TOÁN KINH TẾ HẠ TẦNG (FINOPS & TCO ESTIMATION)](#6-bài-toán-kinh-tế-hạ-tầng-finops--tco-estimation)
7. [LỘ TRÌNH TRIỂN KHAI THEO WAVE (W0 — W4) & GATES](#7-lộ-trình-triển-khai-theo-wave-w0--w4--gates)

---

## 1. TỔNG QUAN CHIẾN LƯỢC & NGUYÊN TẮC HỢP NHẤT

Testing Intelligence (TI) Platform được thiết kế nhằm giải quyết bài toán lớn: **Chuyển đổi từ một hệ thống đánh giá thụ động (Evaluation-only) sang Nền tảng Tự động hóa Thực thi và Đánh giá Kiểm thử Đa miền Toàn diện (Active Test Execution & Evaluation)**.

Hệ thống tuân thủ nghiêm ngặt **24 Architecture Laws** của TI, trong đó tập trung vào các nguyên lý cốt lõi:
1. **Law 4.3 (Workflow Authority)**: `Job Controller` là cơ quan giữ sổ cái trạng thái và điều phối workflow duy nhất; AI Model / AgentCore chỉ là đơn vị tư vấn suy luận, không phải system of record.
2. **Law 5 & 7 (Deterministic Measurement & Zero AI Self-Declaration)**: Mọi kết quả pass/fail đều phải đo bằng công cụ định lượng thật; AI Model tuyệt đối không được tự phán `PASS` cho chính mình.
3. **Law 10.1 & 13 (Server-Owned Binding & Untrusted Artifact)**: `ToolIntent` từ AI không bao giờ được chứa credential, URL thật hoặc câu lệnh shell/SQL tùy ý; mọi đích đến vật lý đều do server tra cứu tại `TenantBinding`.
4. **Law 16 (Immutable Evidence Digest)**: Bằng chứng thô phải được chuẩn hóa, băm SHA-256 và lưu trữ bất biến trên S3 Object Lock.
5. **Law 18 (`completed ≠ PASS`)**: Job chạy xong trạng thái completed chỉ có nghĩa là hoàn thành kỹ thuật, quyết định Gate đạt chuẩn hay không thuộc về luật đánh giá và người có thẩm quyền.
6. **Law 23 (Provider Port & Replacement Contract)**: Mọi module thực thi đều gắn qua Interface chuẩn; thay đổi công nghệ bên dưới không được làm đục lõi điều phối.

---

## 2. MÔ HÌNH HỢP NHẤT 3 TẦNG KIẾN TRÚC TI (3-TIER ARCHITECTURAL FRAMEWORK)

> 📊 **Bản vẽ trực quan Draw.io:** [TI_3Tier_Architecture.drawio](images/TI_3Tier_Architecture.drawio) · *Master Workbook Tab 1:* [TI_Master_Architecture.drawio](images/TI_Master_Architecture.drawio)

Bản thiết kế này chuẩn hóa và tích hợp toàn diện đóng góp từ cả 4 nhóm chuyên trách thành 3 phân tầng kiến trúc chính thức:

```
                  ┌────────────────────────────────────────────────────────┐
                  │  TẦNG TRÍ TUỆ NHÂN TẠO & ĐÁNH GIÁ NGHIỆP VỤ (Task 3&4) │
                  │  • Bedrock Claude Model Tiering 3 cấp                  │
                  │  • Chuỗi S01-S10 & ToolIntent Handshake (Law 10.1)     │
                  │  • Khung đo lường chất lượng ISTQB CT-AI / CT-GenAI    │
                  └───────────────────────────┬────────────────────────────┘
                                              │ (ToolIntent JSON - Law 10.1)
                                              ▼
┌─────────────────────────────────────────────┴─────────────────────────────────────────────┐
│                 TẦNG TRỪU TƯỢNG HÓA & NGUYÊN LÝ KIẾN TRÚC (DevOps / Nguyên)               │
│                 • Triết lý Ports & Adapters (Hexagonal Architecture)                      │
│                 • Cơ chế Plugin & Interface Chuẩn hóa IsolatedRunner (Law 23)             │
│                 • Tách biệt Control Plane vs Execution Engine                             │
└─────────────────────────────────────────────┬─────────────────────────────────────────────┘
                                              │ (Dispatch có Lease - Law 23)
                                              ▼
                  ┌────────────────────────────────────────────────────────┐
                  │   TẦNG HẠ TẦNG ĐIỆN TOÁN & AN NINH MẠNG (Task 1 & 2)   │
                  │   • Hạ tầng AWS 2 Accounts Riêng Biệt                  │
                  │   • ECS Fargate Task-per-Job (Sandbox Cô Lập VM)       │
                  │   • Mạng D9 No-Internet & VPC Endpoints                │
                  │   • Amazon Aurora Serverless v2 Clone (<60s)           │
                  └────────────────────────────────────────────────────────┘
```

### 2.1. Tầng Trừu Tượng Hóa & Nguyên Lý Kiến Trúc: Ports & Adapters và Interface IsolatedRunner (Law 23)
- **Đóng góp từ Hà Tây Nguyên (DevOps)**:
  - Áp dụng mẫu kiến trúc **Hexagonal (Ports and Adapters)** kết hợp **Plugin Architecture**.
  - **Tách bạch Control Plane và Execution Engine**: Control Plane (`Job Controller`) nắm giữ logic nghiệp vụ, quản lý vòng đời tác vụ, nhưng hoàn toàn mù (agnostic) đối với công nghệ kiểm thử cụ thể.
  - **Interface `IsolatedRunner` (Cổng giao tiếp chuẩn)**:
    ```typescript
    interface IsolatedRunner {
      executeJob(input: {
        jobId: string;
        imageDigest: string;           // Image container đã scan bảo mật
        command: string[];             // Lệnh thực thi được whitelist
        tenantBindingRef: string;      // ID cấu hình bí mật (server-resolved)
        resourceLimits: {              // Giới hạn CPU / RAM / Timeout
          cpu: number;
          memoryGiB: number;
          timeoutSeconds: number;
        };
      }): Promise<{
        exitCode: number;
        rawResultPath: string;         // Đường dẫn kết quả thô trên S3
        executionLogsPath: string;     // Log thực thi chi tiết
        sha256Digest: string;          // Mã băm bằng chứng
      }>;
    }
    ```
  - **Lợi ích**: Khi cần thay thế Playwright bằng framework khác hoặc nâng cấp hạ tầng từ ECS Fargate sang EKS+Karpenter (khi throughput tăng cao), phần lõi Job Controller không bị thay đổi bất kỳ dòng code nào.

### 2.2. Tầng Hạ Tầng Điện Toán Đám Mây & An Ninh Mạng: AWS 2 Accounts, Fargate Sandbox & Tối ưu Mạng D9
- **Đóng góp từ Hoàng (Task 2) & Trang (Task 1)**:
  - **Phân tách 2 AWS Accounts độc lập**:
    - **Account A (`ap-southeast-1` - Singapore)**: Backend TI, tiếp nhận API, quản trị danh tính và lưu trữ trạng thái có thẩm quyền.
    - **Account B (`us-east-1` - N. Virginia)**: Cụm AgentCore Runtime và Bedrock Models nơi đặt các cụm mô hình AI tiên tiến nhất của AWS.
  - **Vùng Sandbox Task-per-Job (Domain D2)**:
    - Bác bỏ mô hình chạy trực tiếp trên EC2 hay Testcontainers.
    - Mỗi job kiểm thử chạy trong một **ECS Fargate task riêng biệt** (cô lập mức VM, tự hủy sau khi xong, trả tiền theo giây).
  - **Đột phá Tối ưu Chi phí Mạng D9 (Feedback v0.2)**:
    - Bỏ AWS Network Firewall (~$280/tháng/AZ) $\rightarrow$ Chuyển sang: **Private Subnet không route Internet + Security Group Deny All + VPC Endpoints (S3, ECR, CloudWatch Logs)**. Chi phí giảm từ ~$320/tháng xuống còn **~$22/tháng**.

### 2.3. Tầng Trí Tuệ Nhân Tạo & Điều Phối Đánh Giá Nghiệp Vụ: Chuỗi S01–S10, Bedrock Model Tiering & ToolIntent Handshake
- **Đóng góp từ Nghĩa (Task 3) & Hùng (Task 4)**:
  - **Xương sống 10 chặng xử lý (S01–S10)**: Phân định rõ ràng chặng nào dùng Mã cứng Deterministic (S01, S02, S08, S09 hard barrier), chặng nào dùng AI suy luận (S03 semantic, S04 threat modeling, S05 planning, S06 generation), và chặng nào do Tool đo lường (S07 execution).
  - **Chiến lược Dual-Model Tiering 2 cấp (Hợp nhất Sonnet 5 + Opus 5, Loại bỏ Haiku 4.5 theo DEF-X-M03 / M-02)**:
    - `Claude Sonnet 5`: Đảm nhiệm toàn bộ việc xử lý template, trích xuất JSON, lập kế hoạch kiểm thử (S05) và sinh kịch bản candidate chi tiết (S06) với vai trò model chủ lực mặc định.
    - `Claude Opus 5`: Chỉ kích hoạt khi `Risk Tier == CRITICAL` tại S04 để phân tích Threat Modeling và lỗ hổng logic nghiệp vụ tinh vi.
    - *Ghi chú chuẩn hóa*: Loại bỏ hoàn toàn Claude Haiku 4.5 khỏi kiến trúc nhằm đơn giản hóa điều phối, tránh rủi ro vỡ schema JSON và thống nhất chất lượng sinh test theo quyết định DEF-X-M03 / M-02.
  - **Giao thức ToolIntent Handshake an toàn (Law 10.1 & 13)**:
    - Model chỉ phát `ToolIntent JSON` với các tham số biểu tượng (Symbolic Params).
    - Job Controller chặn lại, kiểm tra Allowlist, tra cứu `TenantBinding` bí mật, cấp STS token ngắn hạn rồi mới dispatch trực tiếp sang Sandbox.

---

## 3. SƠ ĐỒ KIẾN TRÚC TỔNG THỂ (C4 MODEL DIAGRAMS)

> 📐 **KHO BẢN VẼ DRAW.IO CHÍNH THỨC (EDITABLE & ENTERPRISE GRADE):**  
> Toàn bộ các sơ đồ dưới đây đã được số hóa và chuẩn hóa sang định dạng Draw.io XML (`.drawio`) tương tác, tích hợp đầy đủ **bộ icon AWS chính hãng** (CloudFront, WAF, ECS, Fargate, RDS PostgreSQL, S3 Object Lock, Bedrock AI, Aurora Serverless v2, STS, IAM, PrivateLink VPC Endpoints), có thể xem và chỉnh sửa trực tiếp trên [draw.io / diagrams.net](https://app.diagrams.net) hoặc VS Code Draw.io Integration:
> - 🌟 **Master Workbook (4-in-1 Tabs):** [TI_Master_Architecture.drawio](images/TI_Master_Architecture.drawio)
> - 📄 **1. Mô hình 3 tầng kiến trúc:** [TI_3Tier_Architecture.drawio](images/TI_3Tier_Architecture.drawio) *(AWS Bedrock, Organizations, Fargate, PrivateLink, Aurora)*
> - 📄 **2. C4 Level 2 Deployment Topology:** [TI_C4_L2_Topology.drawio](images/TI_C4_L2_Topology.drawio) *(CloudFront, WAF, ECS, RDS, S3 Lock, Bedrock, Fargate Tasks, PrivateLink, Aurora)*
> - 📄 **3. C4 Level 3 Job Controller Deep Dive:** [TI_C4_L3_JobController.drawio](images/TI_C4_L3_JobController.drawio) *(RDS, STS, ECS/Fargate Tasks, S3 Lock)*
> - 📄 **4. Sequence Vòng đời tương tác & Gate S09:** [TI_Sequence_Lifecycle.drawio](images/TI_Sequence_Lifecycle.drawio) *(Lifelines gắn icon ECS, RDS, Bedrock, Fargate, S3 Lock)*

### 3.1. Sơ đồ C4 Level 2: Container & Deployment Topology (Toàn cảnh 3 vùng)

> 📊 **Bản vẽ trực quan Draw.io:** [TI_C4_L2_Topology.drawio](images/TI_C4_L2_Topology.drawio) · *Master Workbook Tab 2:* [TI_Master_Architecture.drawio](images/TI_Master_Architecture.drawio)

```mermaid
flowchart TB
    %% ========================================================
    %% CALLERS & EXTERNAL
    %% ========================================================
    subgraph EXT["BÊN GỌI NGOÀI (EXTERNAL CONSUMERS)"]
        CI["CI/CD Pipeline\n(GitHub Actions / GitLab CI)"]
        PRT["Developer / QA Web Portal\n(Next.js Dashboard)"]
        CLI["TI CLI Client\n(scripts/verify-artifact-api.py)"]
    end

    %% ========================================================
    %% ACCOUNT A: BACKEND & WORKFLOW AUTHORITY (ap-southeast-1)
    %% ========================================================
    subgraph ACC_A["ACCOUNT A — BACKEND & WORKFLOW AUTHORITY (ap-southeast-1) [CANDIDATE]"]
        CF["AWS CloudFront CDN\n(/v1/*, /v2/* Routes)"]
        API["TI API v2 (FastAPI Engine)\n• Xác thực Entra ID / GitHub OIDC\n• Validate Schema & SHA-256 Digest\n• Trả mã 202 Accepted + job_id"]
        
        subgraph JC["JOB CONTROLLER — Account A (Law 4.3 Giữ Sổ Cái & Thẩm Quyền)"]
            direction TB
            ADM["Admission Gate & Rate Limiter"]
            STATE["State Machine (PostgreSQL)\n• Quản lý Trạng thái Job & Step"]
            LEASE["Lease & Heartbeat Coordinator\n• Cấp lease, gia hạn, recovery"]
            RESOLVER["TenantBinding Resolver (Law 13)\n• Tra cứu endpoint thật & STS grant"]
        end

        DB_PG[("Amazon RDS PostgreSQL\n(Job Store Bền Vững - NFR §23)\n• Sổ cái State, Leases, Quotas")]
        S3_EVI[("Amazon S3 + Object Lock\n(Evidence Store Bất Biến - Law 16)\n• Raw Logs, Traces, Screenshots\n• Băm SHA-256 đối soát")]
    end

    %% ========================================================
    %% ACCOUNT B: AGENTCORE AI RUNTIME (us-east-1)
    %% ========================================================
    subgraph ACC_B["ACCOUNT B — AGENTCORE RUNTIME & AI REASONING (us-east-1) [CANDIDATE]"]
        direction TB
        HAR["TI Harness / TIJobRunner\n• Vòng lặp suy luận có giới hạn token\n• KHÔNG phải system of record"]
        
        subgraph TIER["Bedrock Dual-Model Tiering Engine (Task 3 — Sonnet 5 & Opus 5)"]
            M_SONNET["Claude Sonnet 5\n(Planning S05, Candidate S06 & JSON Parsing)"]
            M_OPUS["Claude Opus 5\n(Threat Modeling when Risk == CRITICAL)"]
        end

        GATEWAY["AgentCore Gateway (MCP / IAM)\n• Chặn xuất ToolIntent không hợp lệ"]
        POLICY["Policy Engine\n• Giám sát ENFORCE rào chắn an ninh"]
        MEM[("AgentCore Knowledge Memory [UNVERIFIED]\n(Chỉ lưu tri thức đã qua duyệt GOLDEN · Chưa nghiệm thu)")]
    end

    %% ========================================================
    %% EXTENSION ZONE: ISOLATED RUNNER SANDBOX (Task 2 & DevOps)
    %% ========================================================
    subgraph SANDBOX_VPC["VÙNG MỞ RỘNG — SANDBOX VPC THỰC THI KIỂM THỬ (CANDIDATE)"]
        direction TB

        subgraph NET_D9["Hạ Tầng Mạng An Ninh D9 (~$22/tháng)"]
            SG_DENY["Security Group: DENY ALL EGRESS\nPrivate Subnet (Không NAT, Không IGW)"]
            VPCE["VPC Endpoints (AWS PrivateLink)\n• S3 Endpoint • ECR Endpoint • CloudWatch Logs • STS"]
        end

        subgraph RUNNERS["Cụm Task Fargate Riêng Biệt (Interface: IsolatedRunner - Law 23)"]
            T_SAST["D5a SAST Task (W1)\n(Semgrep OSS + Trivy + Gitleaks)\nQuét PR Diff & Secret"]
            T_API["D2 API Functional Task (W1)\n(Schemathesis + Playwright API)\nKiểm thử Hợp đồng & Fuzzing"]
            T_UI["D3 Web UI Task (W2)\n(Playwright Headless + axe-core)\nChạy E2E & Đo Accessibility"]
            T_PERF["D4 Performance Task (W2)\n(AWS DLT + k6 Engine)\nBơm tải phân tán & Đo SLA p95\n⚠ Cần DevOps duyệt trước"]
            T_DB["D2.b Database Task (W1/W2)\n(Flyway + SQLAlchemy read-only)\nChạy trên Aurora Clone"]
            T_DAST["D5b DAST Task (W3)\n(OWASP ZAP / nuclei)\nQuét web Staging sống"]
        end

        AURORA_CLONE[("Amazon Aurora Serverless v2 Clone\n• Tạo snapshot clone sạch < 60s\n• Tự hủy sau khi test xong")]
    end

    %% ========================================================
    %% KẾT NỐI LUỒNG DỮ LIỆU
    %% ========================================================
    CI & PRT & CLI -->|"1. POST /v2/artifact-jobs\n(Artifact + Digest SHA-256)"| CF
    CF --> API
    API -->|"2. Enqueue Job"| ADM
    ADM --> STATE
    STATE <--> DB_PG

    %% Handshake giữa Job Controller và AI Harness
    ADM -->|"3. Gửi Task Context (Law 10.1)"| HAR
    HAR --> TIER
    HAR --> POLICY
    HAR -->|"4. Phát ToolIntent JSON (Không credential)"| GATEWAY
    GATEWAY -->|"5. Trả ToolIntent"| JC

    %% Job Controller tra cứu binding và dispatch ngang hàng
    LEASE --> RESOLVER
    RESOLVER -->|"6. Direct Dispatch có lease\n(Kèm short-lived STS credentials)"| T_SAST
    RESOLVER -->|"6. Direct Dispatch có lease"| T_API
    RESOLVER -->|"6. Direct Dispatch có lease"| T_UI
    RESOLVER -->|"6. Direct Dispatch có lease"| T_PERF
    RESOLVER -->|"6. Direct Dispatch có lease"| T_DB
    RESOLVER -->|"6. Direct Dispatch có lease"| T_DAST

    %% Kết nối DB Clone
    T_DB -.->|"Kiểm tra schema & migration"| AURORA_CLONE

    %% Rào chắn mạng
    RUNNERS -.-> SG_DENY
    RUNNERS -.-> VPCE

    %% Ghi bằng chứng thô và cập nhật trạng thái
    T_SAST & T_API & T_UI & T_PERF & T_DB & T_DAST -->|"7. Normalized Raw Evidence\n(Logs, traces, HAR, screenshots)"| S3_EVI
    S3_EVI -.->|"8. Báo cáo hoàn tất + SHA-256 Hash"| STATE

    %% Cập nhật tri thức
    S3_EVI -.->|"9. Review GOLDEN (S10 - UNVERIFIED)"| MEM

    %% Polling kết quả
    CI & PRT & CLI -.->|"10. Poll GET /v2/artifact-jobs/{id}\n(completed != PASS - Law 18)"| CF
```

---

### 3.2. Sơ đồ C4 Level 3: Zoom sâu bên trong Job Controller (Workflow Authority)

> 📊 **Bản vẽ trực quan Draw.io:** [TI_C4_L3_JobController.drawio](images/TI_C4_L3_JobController.drawio) · *Master Workbook Tab 3:* [TI_Master_Architecture.drawio](images/TI_Master_Architecture.drawio)

Sơ đồ thể hiện rõ cách Job Controller thực hiện phân giải và dispatch ngang hàng, **loại bỏ hoàn toàn hiểu lầm kiến trúc Fargate lồng Fargate**:

```mermaid
flowchart TD
    subgraph IN_BOUNDARY["1. Tiếp nhận & Xác thực (Admission Layer)"]
        R_IN["Request từ API v2 / ToolIntent từ AgentCore"]
        VAL_SCHEMA["Kiểm tra JSON Schema hợp lệ"]
        VAL_ALLOW["Kiểm tra Tool Allowlist (Law 15)"]
        VAL_BUDGET["Kiểm tra Token & Compute Quota"]
        R_IN --> VAL_SCHEMA --> VAL_ALLOW --> VAL_BUDGET
    end

    subgraph CORE_STATE["2. Sổ cái Trạng thái Bền vững (Job Store PostgreSQL)"]
        INIT_JOB["Khởi tạo Job State: QUEUED"]
        STEP_CTRL["Step State Transition: RUNNING"]
        FAIL_CTRL["Timeout / Retry / Recovery Management"]
        TERM_CTRL["Final State: COMPLETED / FAILED"]
        VAL_BUDGET --> INIT_JOB --> STEP_CTRL
        STEP_CTRL -.-> FAIL_CTRL
        STEP_CTRL --> TERM_CTRL
    end

    subgraph DISPATCH_ENGINE["3. Động cơ Phân giải & Dispatch Ngang Hàng"]
        TB_MAP["TenantBinding Resolver\n• Map URL thật của Staging/Target\n• Gọi AWS STS cấp Short-lived Credentials"]
        LEASE_GRANT["Cấp Worker Lease có thời hạn (Heartbeat TTL)"]
        DIRECT_CALL["Gọi ECS RunTask API (Fargate LaunchType)"]
        TB_MAP --> LEASE_GRANT --> DIRECT_CALL
    end

    subgraph PEER_WORKERS["4. Cụm Worker Ngang Hàng (Fargate Task-per-Job)"]
        W_SAST["Fargate Task: Semgrep SAST"]
        W_API["Fargate Task: Schemathesis / Playwright API"]
        W_UI["Fargate Task: Playwright UI"]
        W_PERF["Fargate Task: k6 Load Test"]
        W_DB["Fargate Task: Flyway DB Clone"]
        W_DAST["Fargate Task: ZAP DAST"]
    end

    subgraph EVIDENCE_COLLECT["5. Thu thập Bằng chứng & Hoàn tất (Law 16)"]
        COL_RAW["Tiếp nhận Raw Results & Artifacts"]
        HASH_SHA["Băm SHA-256 & Normalize Kết quả"]
        STORE_LOCK["Ghi vào Amazon S3 Object Lock"]
        REVOKE["Thu hồi STS Short-lived Credentials"]
        COL_RAW --> HASH_SHA --> STORE_LOCK --> REVOKE
    end

    STEP_CTRL --> TB_MAP
    DIRECT_CALL --> W_SAST & W_API & W_UI & W_PERF & W_DB & W_DAST
    W_SAST & W_API & W_UI & W_PERF & W_DB & W_DAST --> COL_RAW
    REVOKE --> TERM_CTRL
```

---

### 3.3. Sơ đồ Sequence: Vòng đời xử lý một Job từ Tiếp nhận đến Gate S09

> 📊 **Bản vẽ trực quan Draw.io:** [TI_Sequence_Lifecycle.drawio](images/TI_Sequence_Lifecycle.drawio) · *Master Workbook Tab 4:* [TI_Master_Architecture.drawio](images/TI_Master_Architecture.drawio)

```mermaid
sequenceDiagram
    autonumber
    actor Caller as CI / Developer (Caller)
    participant API as TI API v2 (Account A)
    participant JC as Job Controller (Account A)
    participant DB as RDS PostgreSQL (Job Store)
    participant Harness as Bedrock Harness (Account B)
    participant Fargate as Fargate Sandbox (Isolated VPC)
    participant S3 as S3 Object Lock (Evidence)
    actor QA as QA Lead / Authority

    Caller->>API: POST /v2/artifact-jobs (Artifact + sha256)
    API->>JC: Enqueue Job Request
    JC->>DB: Ghi trạng thái QUEUED
    API-->>Caller: 202 Accepted {job_id, poll_url}

    Note over JC,Harness: Chặng Phân Tích Ngữ Nghĩa & Sinh Kịch Bản (S01 - S06)
    JC->>Harness: InvokeHarness (Artifact context, Token Budget)
    Harness->>Harness: S05 Planning & S06 Candidate Gen (Sonnet 5, Opus 5 nếu Threat Critical)
    Harness-->>JC: Trả ToolIntent JSON (Symbolic IDs, KHÔNG credentials)
    JC->>JC: S03 Impact Engine & S04 Risk Engine (State Authority tính toán Target Runners)

    Note over JC,Fargate: Chặng Thực Thi Cô Lập (S07 Orchestration)
    JC->>JC: Tra cứu TenantBinding & STS cấp Token tạm
    JC->>DB: Cập nhật Step State: RUNNING (Lease TTL 5 mins)
    JC->>Fargate: ECS RunTask (Launch Fargate Task theo domain tương ứng)
    
    loop Heartbeat (chu kỳ 60s, Worker Lease TTL 5m)
        Fargate-->>JC: Gửi Heartbeat gia hạn lease
    end

    Fargate->>Fargate: Thực thi trong Private Subnet (SG Deny All, VPC Endpoints)
    Fargate->>S3: Ghi Raw Result (Logs, traces, screenshots)
    Fargate-->>JC: Báo hoàn tất tác vụ (Task Finished + S3 Key)
    
    Note over JC,S3: Chặng Đóng Bằng Chứng & Phán Quyết Gate S09 (S08 - S09)
    JC->>S3: Băm SHA-256 digest & Khóa Object Lock (WORM 90 ngày)
    JC->>JC: S09 Deterministic Code Barrier (Critical==0, Secrets==0, Faithfulness>=0.85 +-0.03)
    JC->>DB: Cập nhật Job State: COMPLETED & Gate Result: PASS / HOLD / DO_NOT_PASS (Law 18)
    
    loop Polling
        Caller->>API: GET /v2/artifact-jobs/{job_id}
        API-->>Caller: 200 OK {state: COMPLETED, gate_result: HOLD / PASS / DO_NOT_PASS}
    end

    opt Nếu Gate Result là HOLD (Khuyến nghị chờ duyệt ngoại lệ - KHÔNG tự động pass)
        QA->>API: POST /v2/operations/{id}/actions (Submit Approved Waiver)
        API->>DB: Cập nhật Final Release Decision: PASS (Đủ điều kiện Deploy)
    end
```

---

## 4. CHI TIẾT CÁC MIỀN THIẾT KẾ KỸ THUẬT (DOMAIN SPECIFICATIONS)

### 4.1. Domain D2: Fargate Sandbox Task-per-Job
- **Cơ chế hoạt động**: Mỗi bài kiểm thử được khởi tạo thành một Fargate Task độc lập với CPU và RAM được cấp phát riêng biệt.
- **Ranh giới cô lập**: Mức máy ảo (MicroVM do AWS Nitro hypervisor quản lý). Loại bỏ hoàn toàn nguy cơ rò rỉ bộ nhớ hoặc dữ liệu chéo giữa các tenant.
- **Vòng đời tác vụ**: Khởi tạo $\rightarrow$ Kéo image từ ECR nội bộ $\rightarrow$ Thực thi script test $\rightarrow$ Đẩy bằng chứng thô về S3 $\rightarrow$ Task tự hủy hoàn toàn.

### 4.2. Domain D2.b: Database Testing với Aurora Serverless v2 Clone
- **Vấn đề đã giải quyết**: Bác bỏ phương án Testcontainers trên máy chủ EC2 backend (vốn gây tốn RAM/CPU, làm tăng chi phí EC2 thêm $30–80/tháng và khó nạp DB lớn).
- **Giải pháp chuẩn hóa**:
  1. Khi nhận bài test database, Job Controller gọi AWS RDS API tạo một **Aurora Clone** từ database staging của tenant. Cơ chế Copy-on-Write cho phép tạo bản clone đầy đủ dữ liệu trong **< 60 giây** mà không tốn chi phí lưu trữ ban đầu.
  2. Fargate Task chạy **Flyway** để test migration kịch bản mới trên bản clone.
  3. Fargate Task chạy script **SQLAlchemy** (read-only) kiểm tra cấu trúc schema và ràng buộc toàn vẹn.
  4. Sau khi test xong, hủy bản Aurora Clone lập tức $\rightarrow$ Không ảnh hưởng đến dữ liệu production, chi phí chỉ tính trong vài phút tồn tại của clone.
- ⚠️ **Lưu ý Vận hành & Quota:** Trạng thái phương án Aurora Clone là `CANDIDATE (PENDING tham vấn Team Data về quota snapshot và tần suất dọn dẹp clone)` theo Báo cáo Đối soát P3.

### 4.3. Domain D5a & D5b: Chiến lược An ninh Đa tầng (Hybrid Defense)
Khắc phục sự lệch pha giữa SAST (quét code) và DAST (quét web sống):
- **Tầng 1 — D5a SAST / SCA / Secret Scanner (Wave 1 — PR-time)**:
  - Công cụ: **Semgrep OSS** (quét SAST), **Trivy** (quét lỗ hổng thư viện phụ thuộc), **Gitleaks** (quét lộ mật khẩu/API key).
  - Vị trí: Chạy ngay khi lập trình viên mở PR (không cần ứng dụng phải deploy).
- **Tầng 2 — Base & Native Inspection (Wave 1 & W2)**:
  - **Amazon Inspector**: Tự động rà quét lỗ hổng ECR container image trước khi chạy task.
  - **Pre-scan S02→S04**: Semgrep OSS (ruleset rút gọn) kết hợp Gitleaks chạy pre-scan AST diff và secret để cung cấp input cho S03/S04. *(Lưu ý: Amazon CodeGuru Security đã chính thức EOL ngừng hoạt động từ 20/11/2025 theo GLOSSARY_TI và Báo cáo Đối soát §3.1 — KHÔNG dùng trong TI).*
- **Tầng 3 — D5b DAST Runner (Wave 3 — Runtime Scanning)**:
  - Công cụ: **OWASP ZAP** / **nuclei** chạy trong Fargate sandbox.
  - Điều kiện kích hoạt: **Chỉ chạy khi có Staging URL sống** do tenant cung cấp.
- **Tầng 4 — AI Threat Modeling (Chỉ kích hoạt khi Cần thiết)**:
  - Khi Tầng 1 hoặc Tầng 2 phát hiện lỗ hổng `CRITICAL`, hệ thống tự động gán `Risk Tier: CRITICAL`.
  - Lúc này, **Claude Opus 5** được huy động để phân tích logic nghiệp vụ sâu (IDOR, race conditions, phân quyền bypass). Nếu không có lỗ hổng Critical, Opus 5 sẽ không được gọi $\rightarrow$ Tiết kiệm ngân sách token tối đa.

> 📝 **Ghi chú Kỹ thuật & Rào chắn (Footnote Gap - Checklist F09):**
> 1. *Phạm vi quét SAST/SCA:* Semgrep/Trivy ở giai đoạn này chủ yếu quét mã nguồn ứng dụng (source code), chưa quét toàn diện các tệp kiểm thử động (testing artifacts). Đây là gap đã được ghi nhận tại Biên bản 23/09 §3 và sẽ được mở rộng ở Wave 2.
> 2. *Rủi ro Prompt Injection:* Trong kiến trúc TI không có UI Chat người dùng cuối trực tiếp; rủi ro Prompt Injection được quản lý theo góc độ phát hiện payload độc hại lọt vào source code / PR diff qua rào chắn allowlist và AgentCore Gateway.

### 4.4. Domain D9: Tối ưu Hóa Chi phí Mạng Egress Sandbox
- **Nguyên tắc**: Tuyệt đối không cho phép container đang chạy test kết nối ra Internet để phòng chống rò rỉ mã nguồn và dữ liệu tenant.
- **Cấu hình hạ tầng**:
  - Task chạy trong Private Subnet không gắn Internet Gateway (IGW) và không có NAT Gateway.
  - Security Group: Thiết lập Outbound Rule là `DENY ALL` (0.0.0.0/0).
  - Giao tiếp với dịch vụ AWS qua **AWS VPC Endpoints (PrivateLink)**:
    - *com.amazonaws.ap-southeast-1.s3* (Gateway Endpoint — Miễn phí).
    - *com.amazonaws.ap-southeast-1.ecr.api* & *ecr.dkr* (Interface Endpoints).
    - *com.amazonaws.ap-southeast-1.logs* (CloudWatch Logs Interface Endpoint).
    - *com.amazonaws.ap-southeast-1.sts* (AWS STS Interface Endpoint cho short-lived tokens).
- **Hiệu quả kinh tế**: Giảm chi phí mạng từ **$288/tháng** (AWS Network Firewall) xuống còn **~$22/tháng** (3 VPC interface endpoints $\times$ $7.3/tháng).

### 4.5. Domain D4: Performance & Load Testing với Distributed k6 Engine
- **Bộ 3 khóa an toàn (3 Safety Interlocks)**:
  1. *Ngưỡng trần tải (VU Ceiling)*: Giới hạn tối đa 500 Virtual Users trong môi trường Staging.
  2. *Circuit Breaker*: Tự động ngắt bài test khi tỷ lệ lỗi HTTP 5xx vượt quá 5% hoặc latency p95 > 2.000ms.
  3. *Tự hủy có giám sát*: Task Fargate tự động hủy khi hết thời gian chạy tối đa (timeout 15 phút).
- ⚠️ **Cảnh báo Rủi ro Hạ tầng & Quy định Phê duyệt (Biên bản 23/09 §3 / Checklist B10, F08)**:
  > **CẢNH BÁO:** Việc bơm tải lớn có nguy cơ làm cạn kiệt tài nguyên hệ thống (tự DoS dịch vụ nội bộ) hoặc bị AWS phòng vệ kích hoạt ban IP / rate-limit tài khoản AWS.  
  > **RÀO CHẮN BẮT BUỘC:** Mọi bài test tải D4 chỉ được phép thực thi trong dải IP/VPC được chỉ định và **bắt buộc phải có phê duyệt trước từ DevOps Team** thông qua TenantBinding Allowlist.

### 4.6. Quản Lý Kịch Bản Kiểm Thử & Lịch Sử Nền Tảng
- **Kho lưu trữ kịch bản (Test Scripts Repository - Checklist F07)**:
  - Toàn bộ kịch bản kiểm thử (Playwright E2E, k6 scripts, Flyway migrations) được quản lý tập trung tại kho mã nguồn `ti-test-packs/` (Đề xuất — `CANDIDATE`).
  - Phân phiên bản chặt chẽ theo **Git Tag** (ví dụ: `v1.2.0-crm-smoke`) gắn kèm digest băm SHA-256 đối soát.
- **Image Container Tiền Đóng Gói (Pre-baked ECR Images - Checklist D04)**:
  - 4 container images chuẩn hóa được đóng gói sẵn và quét an ninh trên Amazon ECR: `ti-runner-sast`, `ti-runner-api`, `ti-runner-browser`, `ti-runner-perf`.
  - Quy trình giao hàng: `GitHub Actions → Amazon ECR → AWS SSM RunCommand/ECS` (Đường khai báo nét đứt, chưa đo).
- **Mốc Lịch Sử AgentCore Harness (Checklist A10, D08)**:
  - `AgentCore Runtime 18` — Xác nhận triển khai qua receipt ngày 14/09/2026 (source `fbdd8dfc`, container image `efabf57dcd8c`) làm mốc tham chiếu lịch sử kiến trúc nền tảng.

---

## 5. HỢP ĐỒNG GIAO DIỆN DỮ LIỆU (CONTRACT SPECIFICATIONS)

### 5.1. Cấu trúc ToolIntent JSON (AgentCore $\rightarrow$ Job Controller)
Tuân thủ nghiêm ngặt **Architecture Law 10.1**: Không bao giờ chứa thông tin đăng nhập, URL hoặc câu SQL tùy ý.

```json
{
  "$schema": "https://ti.internal/schemas/tool-intent-v2.json",
  "intent_id": "intent_7f8a9b0c-1234-5678-abcd-ef0123456789",
  "job_id": "job_3a4b5c6d-9876-5432-dcba-fe9876543210",
  "domain": "API_TESTING",
  "action": "EXECUTE_TEST_PACK",
  "target_binding_ref": "binding_crm_staging_api",
  "execution_plan": {
    "pack_id": "pkg_api_schemathesis_v1",
    "test_suite_ref": "candidate_ts_882910",
    "allowed_methods": ["GET", "POST"],
    "rate_limit_rps": 10,
    "timeout_seconds": 180
  },
  "constraints": {
    "egress_mode": "RESTRICTED_SANDBOX",
    "risk_tier": "MEDIUM",
    "budget_ceiling_usd": 0.50
  }
}
```

### 5.2. Cấu trúc TenantBinding Server-Side (Job Controller $\rightarrow$ Runner)
Được tra cứu hoàn toàn tại server phía TI (Account A), dữ liệu bảo mật được inject trực tiếp qua biến môi trường ngắn hạn:

```json
{
  "binding_id": "binding_crm_staging_api",
  "tenant_id": "tenant_enterprise_xora",
  "resolved_endpoint": "https://staging-api.crm.internal.xora.com/v1",
  "auth_type": "AWS_STS_SHORT_LIVED",
  "role_arn": "arn:aws:iam::123456789012:role/TITestRunnerAssumeRole",
  "allowed_ip_range": "10.100.20.0/24",
  "credential_secret_ref": "arn:aws:secretsmanager:ap-southeast-1:123456789012:secret:tenant_xora_key-9a8b"
}
```

---

## 6. BÀI TOÁN KINH TẾ HẠ TẦNG (FINOPS & TCO ESTIMATION)

Bảng phân tích chi phí dựa trên đơn giá chính thức của AWS (với giả định khối lượng thử nghiệm: **500 jobs/tháng**, thời gian trung bình 5 phút/job):

| Hạng mục Hạ tầng | Phương án Cũ / Lý thuyết | Đề xuất Master Blueprint | Chi phí Ước tính / Tháng | Nhãn Sự Thật |
| :--- | :--- | :--- | :--- | :--- |
| **Compute Sandbox** | Chạy EC2 liên tục ($80/tháng) | **ECS Fargate task-per-job** (2 vCPU, 4GB RAM) | ~$5.00 – $15.00 | `INFERRED` (Đơn giá chính thức AWS Fargate) |
| **Bảo mật Mạng Egress** | AWS Network Firewall | **Private Subnet + SG Deny All + VPC Endpoints** | ~$22.00 | `CANDIDATE` (7.3$/endpoint/AZ) |
| **Job Store Database** | SQLite DEV | **Amazon RDS PostgreSQL (db.t4g.micro)** | ~$15.00 – $25.00 | `CANDIDATE` (Hiện trạng đo 22/09: SQLite trên 1 EC2 ghim 8a61cf66) |
| **Lưu trữ Bằng chứng** | S3 Standard không khóa | **S3 Standard + Object Lock (Compliance Mode)** | ~$3.00 – $5.00 | `CANDIDATE` (Hiện trạng đo 22/09: lưu ổ EBS trên EC2) |
| **Database Testing** | Testcontainers trên EC2 (tăng cấu hình) | **Aurora Serverless v2 Clone** (chỉ tính theo phút) | ~$10.00 – $20.00 | `CANDIDATE` |
| **Chi phí AI Token** | Toàn bộ bằng Claude Opus 5 ($15/1M token) | **Claude Dual-Model Tiering (Sonnet 5 / Opus 5)** | Giảm từ $120 $\rightarrow$ ~$35.00 | `INFERRED` (Tối ưu 65–75% chi phí token, bỏ Haiku 4.5 theo M-02) |
| **TỔNG CỘNG HẠ TẦNG** | **~$450 – $600 / tháng** | **TIẾT KIỆM TỐI ĐA** | **~$90.00 – $122.00 / tháng** | Tiết kiệm ~75% ngân sách |

---

## 7. LỘ TRÌNH TRIỂN KHAI THEO WAVE (W0 — W4) & GATES

| Lộ trình | Miền Kiểm Thử Bao Phủ | Mục Tiêu & Công Cụ Chính | Gate Kiểm Soát | Claim Cho Phép |
| :---: | :--- | :--- | :---: | :--- |
| **Wave W0** | **L0**: Static / Artifact / Contract | Củng cố TI API v2, kiểm tra artifact digest & PinnedContext | **Gate G3** | Artifact evaluation running `OBSERVED` |
| **Wave W1** | **L1**: Unit Testing<br>**L2**: API Testing<br>**L6 (SAST)**: Security PR-time | • Fargate Sandbox D2 đầu tiên.<br>• Dual-mode API (Schemathesis + Playwright API).<br>• Semgrep OSS + Trivy + Gitleaks trong sandbox. | **Gate G4** | API execution + SAST PR-time qualified trên dữ liệu mẫu |
| **Wave W2** | **L3**: Web UI E2E<br>**L8**: Accessibility (a11y)<br>**L5**: Performance & Load | • Playwright headless browser farm trên Fargate.<br>• axe-core quét chuẩn tiếp cận WCAG.<br>• DLT trên AWS với k6 engine. | **Gate G4–G5** | Browser & Load execution qualified |
| **Wave W3** | **L4**: Mobile Testing<br>**L6 (DAST)**: Dynamic Security<br>**L11**: Infra Testing | • Tích hợp AWS Device Farm cho Mobile.<br>• OWASP ZAP / nuclei quét Staging URL sống.<br>• Checkov / OPA quét IaC Terraform. | **Gate G5** | Managed device + DAST qualified |
| **Wave W4** | **L7**: Chaos & Resilience<br>**L9**: Data Quality | • Chaos Engineering (Fault Injection Simulator).<br>• Great Expectations / dbt tests kiểm tra Data. | **Gate G6** | Sẵn sàng Cutover Production |

---

## 8. KẾT LUẬN & HƯỚNG DẪN BẢO TRÌ KIẾN TRÚC

1. **Tính hoàn chỉnh của Bản thiết kế**: Tài liệu này đã khóa chặt ranh giới kỹ thuật giữa **Hạ tầng DevOps**, **Động cơ AI** và **Chiến lược Kiểm thử Chất lượng**, triệt tiêu mọi mâu thuẫn trước đây giữa Task 1, 2, 3 và 4.
2. **Quy tắc bảo trì**: Bất kỳ sự thay đổi nào đối với công cụ chạy test trong tương lai phải được thực hiện thông qua việc viết thêm một Adapter mới tuân thủ interface `IsolatedRunner` (Law 23), tuyệt đối không sửa đổi mã nguồn điều phối của `Job Controller`.
3. **Bước kế tiếp**: Kích hoạt **Spike P4** (ngân sách < $200) để đo kiểm độ trễ Fargate Cold Start và kiểm chứng thực tế interface `IsolatedRunner` trên môi trường AWS trước khi bước vào Wave 1.
