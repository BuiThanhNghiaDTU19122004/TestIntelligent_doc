# TASK 4 — TEST PLAN, TESTING STRATEGY & EVALUATION FRAMEWORK
## Vận dụng 3 Tầng ISTQB vào Hệ thống Testing Intelligence (TI)

---

| Thuộc tính | Giá trị |
| :--- | :--- |
| **Document Status** | Draft v1.2 — Cập nhật theo feedback Nghĩa (22/09/2026) |
| **Version** | v1.2.0 |
| **Date** | 22 September 2026 |
| **Author** | Hùng — Nhóm 4 (Testing Strategy) |
| **Cơ sở lý luận** | ISTQB CTFL v4.0.1 · CT-AI v2.0 · CT-GenAI v1.1 |
| **Review đầu vào** | Task 1 (Trang – Tool & Framework) · Task 2 (Hoàng – Architecture) · Task 3 (Nghĩa – AI Model & Prompting) |
| **Phạm vi áp dụng** | TI Platform — TIEF Phase 1 / Amazon Bedrock AgentCore / Xora Platform |

---

## MỤC LỤC

1. [Bức tranh tổng thể — Tại sao TI cần một chiến lược kiểm thử riêng?](#1-bức-tranh-tổng-thể)
2. [Mapping 3 tầng sách vào kiến trúc TI](#2-mapping-3-tầng-sách-vào-kiến-trúc-ti)
3. [TI MASTER TEST PLAN](#3-ti-master-test-plan)
4. [TESTING STRATEGY — Chiến lược theo từng domain](#4-testing-strategy)
5. [EVALUATION FRAMEWORK — Framework đánh giá kết quả đáng tin cậy](#5-evaluation-framework)
6. [Entry & Exit Criteria — Cổng vào/ra tại mỗi giai đoạn](#6-entry--exit-criteria)
7. [Risk Register — Rủi ro kiểm thử và cách xử lý](#7-risk-register)
8. [Tổng kết & Khuyến nghị hành động](#8-tổng-kết--khuyến-nghị-hành-động)

---

## 1. Bức tranh tổng thể

### 1.1. Vì sao TI không thể áp dụng test plan truyền thống?

Hệ thống TI không phải là một ứng dụng phần mềm thông thường. Nó là một **AI Agent kiểm thử các AI khác** — tức là:

```
TI đồng thời là:
  (A) Một hệ thống phần mềm cần được kiểm thử  ←  Góc nhìn CTFL
  (B) Một hệ thống AI cần được đánh giá chất lượng model  ←  Góc nhìn CT-AI
  (C) Một nền tảng dùng GenAI để sinh test case  ←  Góc nhìn CT-GenAI
```

Điều này tạo ra **ba lớp kiểm thử lồng nhau** mà một test plan đơn giản không thể phủ hết.

### 1.2. Ba câu hỏi cốt lõi mà Task 4 phải trả lời

| # | Câu hỏi | Nguồn lý luận | Ánh xạ trong TI |
| :--- | :--- | :--- | :--- |
| **Q1** | *Test cái gì, vì sao, bằng kỹ thuật nào, khi nào dừng?* | **CTFL §2** — Test Plan | Master Test Plan (Mục 3) |
| **Q2** | *Chất lượng AI/ML được đo bằng gì và đo ở đâu trong pipeline?* | **CT-AI §4–6** — AI Quality + Model Testing | Testing Strategy per Domain (Mục 4) |
| **Q3** | *Kết quả do AI sinh ra có đáng tin không — đo thế nào?* | **CT-GenAI §5–6** — GenAI Evaluation | Evaluation Framework (Mục 5) |

---

## 2. Mapping 3 tầng sách vào kiến trúc TI

### 2.1. Bức tranh ánh xạ tổng thể

```
┌─────────────────────────────────────────────────────────────────────┐
│  TẦNG 3 — CT-GenAI v1.1                                             │
│  "TI dùng GenAI để SINH test case"                                  │
│  → Prompt Policy S05/S06 · 6-component prompt · Injection isolation │
│  → 6 chỉ số đánh giá GenAI output (Groundedness, Precision…)       │
├─────────────────────────────────────────────────────────────────────┤
│  TẦNG 2 — CT-AI v2.0                                                │
│  "TI kiểm thử các HỆ THỐNG AI của tenant"                          │
│  → 6 AI Quality Characteristics → áp vào S04 Risk Engine           │
│  → Confusion Matrix / Precision / Recall → áp vào S09 Gate         │
│  → 4 Data Quality Gates → áp vào S08 Evidence Store                │
│  → 6 Model Testing Techniques + Red Teaming → áp vào S07 Runners   │
├─────────────────────────────────────────────────────────────────────┤
│  TẦNG 1 — CTFL v4.0.1                                               │
│  "TI là phần mềm và phải được kiểm thử như phần mềm"               │
│  → 7 Nguyên tắc → làm guardrail bất biến của 24 Architecture Laws  │
│  → Shift Left → S01–S04 chạy TRƯỚC khi sinh test                   │
│  → Test Pyramid → phân tầng runner Unit/Integration/E2E/Acceptance  │
│  → Test Plan structure → định nghĩa scope, risk, entry/exit critera │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2. Mapping chi tiết 7 nguyên tắc CTFL → 24 Architecture Laws TI

| # | Nguyên tắc CTFL | Biểu hiện trong kiến trúc TI | Architecture Law tương ứng |
| :---: | :--- | :--- | :--- |
| 1 | **Chứng minh sự hiện diện defect** — testing chỉ chứng minh có lỗi, không chứng minh hết | Mọi kết quả PASS từ TI chỉ là GATE_RECOMMENDATION, không phải xác nhận tuyệt đối | Law 18: `completed ≠ PASS` |
| 2 | **Vét cạn là bất khả thi** — phải ưu tiên theo risk | S04 Risk Engine phân loại `LOW/MEDIUM/HIGH/CRITICAL` để phân bổ runner phù hợp | Law 5: Deterministic tools kiểm soát kết quả đo |
| 3 | **Kiểm thử sớm (Shift Left)** — phát hiện lỗi sớm rẻ hơn 10–100× | S01–S04 phân tích artifact TRƯỚC khi chạy bất kỳ test nào — CodeGuru scan ngay khi PR mở | Law 8: Candidate không phải executed test |
| 4 | **Defect tụ cụm** — 20% module chứa 80% lỗi | S03 Impact Engine xây đồ thị phụ thuộc để xác định "điểm nóng" cần tập trung | Law 9: Reviewer kiểm soát candidate decision |
| 5 | **Nghịch lý thuốc trừ sâu** — test lặp lại mãi mất hiệu lực | S10 Production Learning cập nhật tri thức sau mỗi lượt chạy, tránh test catalog hóa thạch | Law 7: Model không tự tuyên bố pass |
| 6 | **Phụ thuộc ngữ cảnh** — Fintech ≠ Healthcare | `Evaluation Pack` và `TenantBinding` cho phép mỗi tenant có bộ ngưỡng SLO riêng | Law 12–14: Artifact là untrusted data |
| 7 | **Ảo tưởng "không lỗi"** — 0 lỗi ≠ hệ thống dùng được | S09 Gate Recommendation có thể trả về `HOLD` dù không có lỗi nào — nếu coverage không đủ | Law 15–16: Tool allowlist, raw result hash SHA-256 |

---

## 3. TI MASTER TEST PLAN

> **Nguồn lý luận chính:** CTFL §2.1 — "Test plan tốt trả lời được: test cái gì · vì sao · bằng kỹ thuật nào · khi nào dừng."

### 3.1. Phạm vi kiểm thử (Test Scope)

#### 3.1.1. Trong phạm vi (In-Scope)

| Domain | Đối tượng kiểm thử | Test Level | Công cụ (từ Task 1) | Vị trí trong pipeline |
| :--- | :--- | :--- | :--- | :--- |
| **API Functional & Fuzzing** | TI API (:8000) và API của artifact tenant | Integration + System | **Mode 1 — Fuzzing:** Schemathesis + AWS CodeBuild/Lambda · **Mode 2 — Kịch bản nghiệp vụ:** Playwright API (`request.newContext()`) / httpx trên Fargate/Lambda | S07 Runner |
| **UI / Web E2E** | TI Portal (:8001) và Web App của tenant | System + Acceptance | CloudWatch Synthetics + Playwright + axe-core | S07 Runner |
| **Database Migration** | Script migration của PR tenant | Component + Integration | Aurora Serverless v2 Clone + ECS Fargate + Flyway | S07 Runner |
| **Performance & Load** | TI API và Service của tenant dưới tải | System + Performance | AWS DLT + k6 Engine trên Fargate | S07 Runner |
| **Security SAST/SCA/Secret** | Code diff của PR, dependencies lockfile | Component (Static) | **Semgrep OSS + Trivy + Gitleaks** — chạy container offline `--network none`, output SARIF | **S07 Runner** (W1) |
| **Risk Scoring (không phải runner)** | PR diff và dependency graph | Static Analysis | **Amazon CodeGuru Security + Inspector** — gọi qua API, output JSON risk score | **S02 → S04** (input cho Risk Engine, không phải runner S07) |
| **AI Model Quality** | Claude model sinh test case (S05/S06) | AI Model Testing | Amazon Bedrock Evaluations (`TIRunnerGroundness`) | S05/S06 → S08 |
| **AI Behavior / Prompt Injection** | Input artifact từ tenant (OpenAPI, SQL, HTML) | Security Testing | Context Isolation layers (4-barrier defense) | S02 Artifact Intake |

> ⚠️ **Phân biệt quan trọng:** CodeGuru Security + Inspector thuộc chặng **S02→S04** (cung cấp risk score cho Risk Engine), **không phải** security runner trong S07. Security runner trong S07 là Semgrep + Trivy + Gitleaks. Xem thêm Task 1 (Trang) — bảng ánh xạ vị trí pipeline.

#### 3.1.2. Ngoài phạm vi (Out-of-Scope) — với lý do

| Hạng mục | Lý do loại trừ |
| :--- | :--- |
| Mobile Testing (Android/iOS) | TI không có artifact mobile trong TIEF Phase 1 |
| Contract Testing (Pact) | Kiến trúc 2-account đã dùng AgentCore Gateway + IAM thay thế |
| Visual Regression Testing | Không trong phạm vi bằng chứng cần thiết cho S08 |
| Endurance/Soak Testing (>4h) | Phase 1 chưa yêu cầu; thêm vào Phase 2 roadmap |

### 3.2. Phân cấp Test Levels — Áp dụng Test Pyramid vào TI

```
                    ┌──────────────────────────┐
                    │  ACCEPTANCE (UAT)         │  ← S09 Gate Recommendation
                    │  Tenant review PASS/HOLD  │     (Human Decision)
                ┌───┴──────────────────────────┴───┐
                │  SYSTEM E2E                       │  ← S07 Orchestration
                │  API + UI + DB + Perf + Security  │     (4 Runner domains)
            ┌───┴───────────────────────────────────┴───┐
            │  INTEGRATION                               │  ← S03 Impact Engine
            │  Tool adapters ↔ AgentCore Gateway ↔ AWS  │     (Dependency graph)
        ┌───┴───────────────────────────────────────────┴───┐
        │  COMPONENT / UNIT (Static + Candidate Validation)  │  ← S02 Change Detector
        │  CodeGuru scan · Bedrock Eval · Schema parsing     │     + S05/S06 Generation
        └─────────────────────────────────────────────────────┘
         NHIỀU · NHANH · RẺ                      ÍT · CHẬM · ĐẮT →
```

**Nguyên tắc Shift Left ứng dụng:** S01–S04 phải hoàn thành (và Evidence phải có mặt trong S08) **trước khi** bất kỳ runner nào trong S07 được kích hoạt.

### 3.3. Các loại Test Types và kỹ thuật thiết kế test

| Test Type | Kỹ thuật ISTQB áp dụng | Mapping vào TI |
| :--- | :--- | :--- |
| **Functional** | EP (Equivalence Partitioning) + BVA (Boundary Value Analysis) | Schemathesis tự động sinh boundary payload từ OpenAPI schema → tương đương BVA tự động hóa |
| **Non-functional / Performance** | Stress Testing + Spike Testing + Load Testing | k6 stages: Ramp-up → Hold → Ramp-down; VUs không vượt quota (tránh Self-inflicted DoS) |
| **Security** | White-box SAST + Experience-based (Exploratory) | Semgrep/Trivy/Gitleaks chạy offline trong S07; **Opus 5 BẮT BUỘC cho S05 Threat Modeling** (theo Task 3 — Capability Manifest `BEDROCK.CLAUDE_OPUS_5@1`); Haiku 4.5 tóm tắt SARIF findings cho S09 |
| **Regression** | State Transition Testing (Order status lifecycle) | Mỗi PR trigger lại toàn bộ pipeline S01–S10 → Automatic regression |
| **AI-specific: Adversarial** | Adversarial Testing (CT-AI §5.3.1) | Bơm payload đặc biệt (null, emoji, SQL-in-JSON) vào API runner |
| **AI-specific: Metamorphic** | Metamorphic Testing (CT-AI §5.3.3) | Thay đổi field không quan trọng của input → output phải nhất quán |
| **AI-specific: A/B Model** | Back-to-Back Testing (CT-AI §5.3.5) | So sánh output của Haiku 4.5 vs Sonnet 5 trên cùng prompt để calibrate tier |

### 3.4. Testware — Sản phẩm bắt buộc của quy trình TI

| Testware | Mô tả | Lưu ở đâu | Truth Class |
| :--- | :--- | :--- | :--- |
| **Test Plan** (tài liệu này) | Phạm vi, chiến lược, risk, entry/exit criteria | `.kiro/steering/` hoặc Wiki | `PUBLICATION` |
| **Test Candidate** | JSON schema đã định nghĩa (API/DB/UI/Perf/Sec) | S06 output → S08 Evidence | `CANDIDATE` |
| **Test Evidence** | Raw result từ runner: JSON, PNG, HAR, SARIF | S08 Evidence Store (S3, SHA-256) | `OBSERVED` |
| **Gate Recommendation** | PASS / DO_NOT_PASS / HOLD + rationale | S09 output | `DERIVED` |
| **Production Learning** | Tri thức rút ra sau mỗi job | S10 → AgentCore Memory | `REUSE_RECEIPT` |

### 3.5. Traceability Matrix — Truy vết yêu cầu → test case

| Yêu cầu Kiến trúc (Law) | Test Candidate Domain | Evidence bắt buộc | Gate Condition |
| :--- | :--- | :--- | :--- |
| Law 5: Deterministic tools kiểm soát kết quả | API + DB + Perf + Security | SHA-256 hash raw result | Tool result ≠ Model claim → Tool wins |
| Law 7: Model không tự tuyên bố pass | AI Model Quality | GroundednessScore ∈ [0,1] | Score < threshold → reject candidate |
| Law 8: Candidate ≠ executed test | Tất cả domains | Candidate schema valid JSON | Invalid schema → reject trước khi execute |
| Law 12–14: Artifact là untrusted data | Security + Injection | Sanitization log | Any injection attempt detected → CRITICAL risk |
| Law 18: completed ≠ PASS | Tất cả domains | Gate Recommendation record | Tất cả Evidence phải OBSERVED trước khi Gate |

---

## 4. TESTING STRATEGY

> **Nguồn lý luận chính:** CT-AI v2.0 §4 (AI Quality Chars) + §5 (Testing Techniques) + CT-GenAI v1.1 §4 (GenAI trong Testing Activities)

### 4.1. Chiến lược tổng thể: Evidence-Driven, Risk-Tiered

TI không test tất cả mọi thứ với cùng cường độ. Chiến lược phân tầng dựa trên **Risk Tier** (từ S04) và **đặc tính AI Quality** (từ CT-AI):

```
Risk Tier CRITICAL  → Chạy TẤT CẢ 6 domain runners:
                       API + UI + DB + Perf + Security (SAST/SCA) + AI Model Quality (Bedrock Eval)
                       + Opus 5 Threat Modeling (bắt buộc)
Risk Tier HIGH      → Chạy API + Security (SAST/SCA) + domain bị ảnh hưởng trực tiếp
                       + AI Model Quality nếu prompt thay đổi
Risk Tier MEDIUM    → Chạy API + Smoke test UI + Static security scan (Semgrep/Trivy)
Risk Tier LOW       → Chỉ chạy CodeGuru static scan (S02→S04) + Schema validation
```

> **Lưu ý 6 domains:** (1) API, (2) UI/Web, (3) Database, (4) Performance, (5) Security SAST/SCA, (6) AI Model Quality (Bedrock Evaluations). Amazon CodeGuru Security + Inspector là pre-runner ở S02→S04, không tính vào 6 domain runners.

### 4.2. Strategy theo từng Domain

#### 4.2.1. Domain 1 — API Testing Strategy

**Mục tiêu:** Đảm bảo tính đúng đắn (Correctness), tính bền vững (Robustness) và tính toàn vẹn hợp đồng (Contract Integrity) của mọi API endpoint bị thay đổi.

> **Dual-Mode API Testing** — hai cơ chế song song, phục vụ mục tiêu khác nhau:
>
> | Mode | Công cụ | Đầu vào | Mục đích | Khi nào chạy |
> | :--- | :--- | :--- | :--- | :--- |
> | **Mode 1 — Fuzzing tự động** | **Schemathesis** trên AWS CodeBuild/Lambda | OpenAPI/Swagger spec (không cần AI sinh) | Quét 100% boundary, edge case, EP/BVA tự động — không tốn token AI | Mọi PR có thay đổi API spec |
> | **Mode 2 — Kịch bản nghiệp vụ** | **Playwright API** (`request.newContext()`) **/ httpx** trên Fargate/Lambda | `API_CANDIDATE_V1` JSON do S06 (LLM) sinh ra | Chạy luồng nghiệp vụ chuỗi phức tạp (chained flows, stateful sessions, auth flows) | Khi S06 sinh candidate dạng FUNCTIONAL hoặc INTEGRATION |
>
> **Lý do cần Mode 2:** Schemathesis đọc OpenAPI spec thô — nó **không đọc file JSON `API_CANDIDATE_V1`** do LLM sinh ra. Các kịch bản nghiệp vụ phức tạp (tạo order → thanh toán → xác nhận) cần runner có thể thực thi đúng theo schema candidate đã định nghĩa.

**Kỹ thuật ISTQB áp dụng:**

| Kỹ thuật | Nguồn | Cách triển khai trong TI |
| :--- | :--- | :--- |
| **BVA 3-value** (CTFL §4.2) | CTFL | Schemathesis tự động sinh `min-1, min, min+1, max-1, max, max+1` cho mọi numeric field trong OpenAPI |
| **EP** (CTFL §4.1) | CTFL | Schemathesis phân nhóm: valid payload / invalid type / missing required / null / boundary |
| **Adversarial Testing** (CT-AI §5.3.1) | CT-AI | Bơm payload đặc biệt: `{"age": -1}`, `{"name": null}`, `{"id": "'; DROP TABLE"}` |
| **Metamorphic Testing** (CT-AI §5.3.3) | CT-AI | Thêm field không quan trọng → HTTP status và body structure phải giống hệt |

**Entry Criteria:** OpenAPI/Swagger spec của artifact đã được xác thực cú pháp (valid JSON/YAML).

**Exit Criteria:**
- Mode 1: 0 HTTP 500 response từ Schemathesis fuzzing; 0 schema violation
- Mode 2: 100% happy-path assertions trong `API_CANDIDATE_V1` PASS (HTTP status, JSONPath, latency)
- Response latency P95 ≤ `${EVAL_PACK.perf.p95_ms}` (từ Evaluation Pack)
- Evidence SHA-256 hash đã lưu vào S08 cho cả hai mode

**Prompt S05 (Planning) — vận dụng CT-GenAI 6-component structure:**
```
[ROLE]    Senior API Tester với kiến thức ISTQB CTFL + CT-AI
[CONTEXT] Artifact: OpenAPI spec diff giữa base và head version
[TASK]    Lập kế hoạch kiểm thử 4 tầng:
          (1) Contract Validation — field mới, types, required flags
          (2) Happy Path Flow — luồng nghiệp vụ chính
          (3) Boundary & Negative — BVA 3-value + EP invalid partitions
          (4) Idempotency Check — POST idempotency với X-Idempotency-Key
[FORMAT]  JSON TestPlan với coverage_gaps list cho S06
[CONSTRAINT] Không hardcode URL; chỉ dùng operation_ref
[EXAMPLES] Tham chiếu API_CANDIDATE_V1 schema
```

---

#### 4.2.2. Domain 2 — Database Testing Strategy

**Mục tiêu:** Đảm bảo mọi script migration an toàn, backward-compatible và không phá vỡ data integrity trước khi merge vào nhánh chính.

**Kỹ thuật ISTQB áp dụng:**

| Kỹ thuật | Nguồn | Cách triển khai trong TI |
| :--- | :--- | :--- |
| **State Transition Testing** (CTFL §4.4) | CTFL | Test lifecycle của migration: `Pending → Applied → RolledBack` |
| **Decision Table Testing** (CTFL §4.3) | CTFL | Tổ hợp loại thay đổi schema (ADD/RENAME/DROP/MODIFY) × nullable/not-null × foreign key |
| **Data Pipeline Integrity** (CT-AI §5.1.2) | CT-AI | Fault injection: bơm null vào trường NOT NULL, test rollback khi migration nửa chừng |
| **Schema & Constraint Check** (CT-AI §5.1.4) | CT-AI | `information_schema` queries xác minh column type, nullable, constraint sau migration |

**Cơ chế thực thi:** Aurora Serverless v2 Clone (Copy-on-write, < 60s) + ECS Fargate chạy Flyway. Mọi thao tác trong transaction `ROLLBACK` bắt buộc. SQL parser chặn DDL/DML ngoài SELECT trên catalog.

**Phân loại tác động migration** (phân tầng risk):

| Loại thay đổi | Risk Tier | Hành động |
| :--- | :--- | :--- |
| Thêm cột nullable, bảng mới | LOW | Chỉ verify schema sau migration |
| Thêm cột NOT NULL không có DEFAULT | HIGH | Verify + kiểm tra backward compat với app version cũ |
| Đổi kiểu dữ liệu, xóa cột/bảng | CRITICAL | Verify + Rollback test + Dual-write check + Opus 5 review |
| Thêm index trên bảng lớn | HIGH | Verify + đo execution plan (EXPLAIN ANALYZE) |

---

#### 4.2.3. Domain 3 — UI / Web Testing Strategy

**Mục tiêu:** Đảm bảo luồng người dùng (user journey) hoạt động đúng, không có lỗi console, và đạt chuẩn tiếp cận WCAG 2.1 AA.

**Kỹ thuật ISTQB áp dụng:**

| Kỹ thuật | Nguồn | Cách triển khai trong TI |
| :--- | :--- | :--- |
| **State Transition** (CTFL §4.4) | CTFL | Test trạng thái UI: `Loading → Loaded → Error → Retry` |
| **Use Case / User Journey Testing** (CTFL §4.5) | CTFL | Kịch bản end-to-end: đăng nhập → thao tác → xác nhận kết quả |
| **Accessibility — Usability Characteristic** (CT-AI §4.1) | CT-AI | axe-core tự động scan WCAG 2.1 AA violations sau mỗi bước; Usability là một trong 6 AI Quality Characteristics |
| **Visual Evidence Reasoning** (Design Decision — CT-GenAI §5 Multi-modal Eval) | CT-GenAI §5 | Claude Sonnet 5 Vision phân tích screenshot khi có layout anomaly — đây là ứng dụng GenAI vào đánh giá bằng chứng trực quan, không phải kỹ thuật UI testing truyền thống |

**Selector Strategy** (bắt buộc tuân thủ):
- ✅ Ưu tiên: `data-testid` hoặc `ROLE_AND_NAME` (WAI-ARIA)
- ❌ Cấm: XPath tuyệt đối (`/html/body/div[2]/...`) hoặc CSS class tự sinh từ build tool
- ❌ Cấm: `WAIT_SLEEP` cứng — phải dùng `WAIT_FOR_ELEMENT` có timeout

**Exit Criteria:**
- 0 JavaScript console errors trong lượt chạy
- 0 WCAG 2.1 AA violations (axe-core)
- Screenshot final step đã lưu S3 (SHA-256)
- Playwright trace file đã lưu

---

#### 4.2.4. Domain 4 — Performance Testing Strategy

**Mục tiêu:** Đảm bảo không có Performance Regression sau mỗi PR, và TI API (:8000) chịu được tải đồng thời của nhiều tenant.

**Kỹ thuật ISTQB áp dụng:**

| Loại test | Nguồn | Cấu hình k6 |
| :--- | :--- | :--- |
| **Baseline Load** | CTFL §6.3 — Non-functional | 10 VUs × 60s · Đo P50, P95, RPS baseline |
| **Concurrency Stress** | CT-AI §6.1 — Deployment Testing | Ramp: 0→50 VUs in 30s · Hold 60s · Ramp-down 30s |
| **Spike Test** | CT-AI §6.1 | 0→100 VUs in 5s · Hold 15s (mô phỏng flash submission) |

**Ngưỡng SLO** — **server-owned từ Evaluation Pack per-tenant**, không được hard-code trong chiến lược:

> ⚠️ **Nguyên tắc bất biến (Task 3 + Architecture Law 6):** Toàn bộ ngưỡng SLO để ra quyết định Gate tại S09 **phải được nạp từ `Evaluation Pack` của Server**. Không chấp nhận ngưỡng từ artifact PR, không hard-code constant trong strategy document. Mỗi tenant type (Fintech / Healthcare / SaaS) có bộ ngưỡng riêng.

| Metric | Placeholder tham chiếu | Hành động nếu vượt | Ghi chú |
| :--- | :--- | :--- | :--- |
| P95 latency | `${EVAL_PACK.perf.p95_ms}` | `DO_NOT_PASS` | Giá trị ví dụ (Fintech): 500ms — phải set trong Evaluation Pack |
| P99 latency | `${EVAL_PACK.perf.p99_ms}` | `HOLD` | Giá trị ví dụ (Fintech): 1000ms |
| Error rate | `${EVAL_PACK.perf.error_rate_pct}` | `DO_NOT_PASS` | Giá trị ví dụ: < 1% |
| Throughput | `${EVAL_PACK.perf.min_rps}` | `HOLD` | **Chú ý:** Không set ngưỡng RPS chung cho mọi tenant — microservice background cron sẽ không bao giờ đạt RPS cao |
| CPU saturation | `${EVAL_PACK.infra.cpu_max_pct}` | `HOLD` | Giá trị ví dụ: 85% |

**Guardrail chống Self-inflicted DoS:** `MAX_VUS = 100`, `MAX_DURATION = 300s` — hard-coded ở phía server, model không thể override.

---

#### 4.2.5. Domain 5 — Security Testing Strategy

**Mục tiêu:** Zero-tolerance với Critical vulnerability. High vulnerability phải được xử lý có kiểm soát. Không có secret bị hardcode. Không có lỗ hổng logic nghiệp vụ tinh vi (IDOR, Race Condition, BOLA).

**Kiến trúc Hybrid Defense — 3 tầng phòng thủ kết hợp AWS Native + Container cô lập + AI Reasoning:**

```
┌─────────────────────────────────────────────────────────────────────────┐
│  TẦNG 0 (S02→S04): AWS Native Risk Scoring — chạy TRƯỚC khi S07        │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Amazon CodeGuru Security  →  phân tích PR diff bằng ML AWS    │    │
│  │  Amazon Inspector          →  quét CVE thư viện tự động         │    │
│  │  Output: JSON risk score   →  S04 Risk Engine tính Risk Tier    │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│   (Không phải S07 runner — đây là pre-flight risk assessment)           │
├─────────────────────────────────────────────────────────────────────────┤
│  TẦNG 1 (S07 Runner W1): Container cô lập — Deterministic Tools         │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Semgrep OSS    →  SAST offline  (OWASP Top 10, CWE Top 25)    │    │
│  │  Trivy          →  SCA offline   (CVE dependencies/images)     │    │
│  │  Gitleaks       →  Secret scan   (Git diff)                    │    │
│  │  Container: --network none · Private Subnet · output SARIF     │    │
│  └─────────────────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────────┤
│  TẦNG 2 (S05 AI-assisted): Claude Opus 5 — Threat Modeling              │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  Chỉ kích hoạt khi Risk Tier == CRITICAL (từ Tầng 0)           │    │
│  │  Attack Surface Mapping · IDOR/BOLA · Race Condition Analysis  │    │
│  │  Model: BEDROCK.CLAUDE_OPUS_5@1 (bắt buộc — theo Task 3)      │    │
│  └─────────────────────────────────────────────────────────────────┘    │
├─────────────────────────────────────────────────────────────────────────┤
│  TẦNG 3 (Manual): Penetration Testing                                   │
│  └─ Chỉ trigger khi Risk Tier = CRITICAL VÀ Tầng 1&2 phát hiện anomaly │
└─────────────────────────────────────────────────────────────────────────┘
```

> **Phân biệt vai trò CodeGuru vs Semgrep (quan trọng):**
> - **CodeGuru Security + Inspector** (Tầng 0): Dịch vụ AWS Managed, gọi qua API, chạy song song với S02 Change Detector, cung cấp risk score đầu vào cho S04 Risk Engine — **không phải S07 runner**.
> - **Semgrep + Trivy + Gitleaks** (Tầng 1): Chạy trong Fargate container cô lập hoàn toàn (`--network none`), là **S07 runner chính thức** — output SARIF được hash SHA-256 và lưu vào S08 Evidence.
>
> Hai tầng này **bổ sung cho nhau**, không thay thế nhau: Tầng 0 nhanh (giây) cho risk signal sớm; Tầng 1 sâu (phút) cho bằng chứng pháp lý bất biến.

**4-barrier Defense-in-Depth** (từ CT-GenAI §6 — Technical Risk Management):

| Barrier | Cơ chế | Mục tiêu |
| :--- | :--- | :--- |
| **B1 — Tool Sandbox** | Semgrep/Trivy/Gitleaks chạy trong container `--network none` (Private Subnet + Security Group Deny All + VPC Endpoints S3/ECR/CloudWatch), model không đọc code trực tiếp | Tách model khỏi data độc hại; ngắt internet hoàn toàn mà không dùng AWS Network Firewall ($280/tháng/AZ) |
| **B2 — Sanitization** | Parser bóc tách SARIF, chỉ giữ `rule_id, severity, file, line, cve_id` | Loại bỏ injection payload trong report |
| **B3 — Context Boxing** | Bọc findings trong `<untrusted_findings>` + system instruction | Ngăn model tuân theo chỉ thị trong data |
| **B4 — Deterministic Gate** | `IF critical_count > 0 THEN DO_NOT_PASS` — model không thể override | Bức tường cuối không thể bypass |

> **Quyết định kiến trúc egress (đồng bộ với feedback mentor → Task 2 v0.2):** Network isolation cho tất cả S07 runners dùng **Private Subnet (không route internet) + Security Group Deny All Egress + VPC Endpoints (S3, ECR, CloudWatch Logs)**. Không dùng AWS Network Firewall (~$280/tháng/AZ). Chi phí giảm từ ~$320/tháng xuống < $15/tháng mà vẫn đảm bảo sandbox hoàn toàn.

**Luồng handshake S07 Orchestration** (đồng bộ với sơ đồ kiến trúc Task 2 v0.2):

```
AgentCore (us-east-1)               Job Controller (ap-southeast-1)
     │                                       │
     │  ToolIntent JSON                      │
     │ ─────────────────────────────────────>│
     │  {logical_tool_id, operation_ref,     │  1. Nhận ToolIntent
     │   arguments (abstract), idempotency}  │  2. Validate allowlist
     │                                       │  3. Tra cứu TenantBinding
     │                                       │     (URL thật + Secrets từ Secrets Manager)
     │                                       │
     │                          ┌────────────┴──────────────────────┐
     │                          │      Dispatch Fargate Tasks       │
     │                          ├───────────────────────────────────┤
     │                          │  Playwright Task  │  k6 Task      │
     │                          │  Semgrep Task     │  Flyway Task  │
     │                          └────────────┬──────────────────────┘
     │                                       │
     │  ToolObservation (rút gọn)            │  4. Raw result → SHA-256 → S3
     │ <─────────────────────────────────────│  5. Evidence Envelope → S08
     │                                       │  6. Return ToolObservation
```

**Exit Criteria — Phân cấp rõ ràng (đồng bộ với Mục 6.2):**

| Trạng thái | Điều kiện | Có thể override không? |
| :--- | :--- | :--- |
| **`DO_NOT_PASS`** (Hard stop) | `CRITICAL_COUNT > 0` HOẶC `SECRETS_LEAKED > 0` | ❌ Tuyệt đối không — không waiver, không exception |
| **`HOLD`** (Chờ quyết định) | `HIGH_COUNT > 0` | ✅ Chỉ được phê duyệt thành `PASS` khi có **Waiver Document** kèm chữ ký của Architecture Authority |
| **`PASS`** | `CRITICAL_COUNT == 0` VÀ `SECRETS_LEAKED == 0` VÀ (`HIGH_COUNT == 0` HOẶC có approved Waiver) | — |

> **Quy tắc vàng:** `CRITICAL` và `SECRETS_LEAKED` là **hard stop tuyệt đối** — không AI, không human nào có thể override. `HIGH` yêu cầu **conscious human decision** với văn bản phê duyệt rõ ràng, không phải bấm nút cho qua.

---

#### 4.2.6. Domain 6 — AI Model Quality Strategy

**Mục tiêu:** Đảm bảo Claude model sinh test case có căn cứ thực tế, không ảo giác, không bịa đặt API hay credential.

**6 Đặc tính Chất lượng AI** (CT-AI §4.1) áp vào TI engine:

| Đặc tính AI | Định nghĩa CT-AI | Cách đo trong TI |
| :--- | :--- | :--- |
| **Correctness** | Output đúng về nội dung | `GoalSuccessRate` từ Bedrock Evaluations |
| **Robustness** | Không bị lung lay bởi noise/injection | Prompt injection test với malicious artifact |
| **Performance Efficiency** | Sử dụng tài nguyên hợp lý | Token consumption ≤ `token_ceiling` trong Manifest |
| **Explainability (XAI)** | Output có thể giải thích được | `rationale` field bắt buộc trong mọi Gate Recommendation |
| **Safety** | Không gây hại (không tự deploy, không tự escalate quyền) | Law 13–14: Model chỉ phát intent, không chọn endpoint/credential |
| **Fairness** | Không thiên vị theo tenant | Evidence hash độc lập per-job, không share context giữa tenant |

---

## 5. EVALUATION FRAMEWORK

> **Nguồn lý luận chính:** CT-GenAI v1.1 §5 (Evaluation Metrics & Methodology) + CT-AI v2.0 §4.2 (Confusion Matrix) + CTFL §5.3 (Test Completion)

### 5.1. Tại sao cần Framework đánh giá riêng cho TI?

Hệ thống TI có **hai đối tượng đánh giá song song**:

```
Đối tượng A: "Artifact của tenant có chất lượng không?"
  → Đo bằng Evidence từ 5 runner domains
  → Kết quả: PASS / DO_NOT_PASS / HOLD

Đối tượng B: "TI Engine (Claude AI) hoạt động đúng không?"
  → Đo bằng Amazon Bedrock Evaluations
  → Kết quả: GroundednessScore / GoalSuccessRate / Precision / Recall
```

Hai đối tượng này phải được đánh giá **độc lập** — kết quả của B không được ảnh hưởng đến A.

### 5.2. Confusion Matrix áp dụng vào S09 Gate

Vận dụng trực tiếp từ **CT-AI §4.3 (Confusion Matrix)**, định nghĩa 4 trạng thái đánh giá của TI:

```
                   ┌─────────────────────────────────────────────────┐
                   │     THỰC TẾ: Artifact CÓ lỗi nghiêm trọng      │
                   ├────────────────────┬────────────────────────────┤
TI dự đoán:        │      CÓ lỗi (TI    │      KHÔNG có lỗi (TI      │
                   │      → DO_NOT_PASS)│      → PASS)               │
───────────────────┼────────────────────┼────────────────────────────┤
CÓ lỗi thật       │  ✅ TRUE POSITIVE   │  ❌ FALSE NEGATIVE         │
(bắt đúng)        │  TI bắt đúng lỗi   │  TI bỏ sót lỗi nghiêm trọng│
                   │  → GIÁ TRỊ CAO NHẤT│  → NGUY HIỂM NHẤT          │
───────────────────┼────────────────────┼────────────────────────────┤
Không có lỗi thật │  ❌ FALSE POSITIVE  │  ✅ TRUE NEGATIVE           │
(báo động giả)    │  TI chặn code tốt  │  TI cho qua code an toàn   │
                   │  → Tốn thời gian   │  → Lý tưởng                 │
                   └────────────────────┴────────────────────────────┘
```

**Nguyên tắc thiết kế Gate:** TI được phép có False Positive (báo động giả) nhưng **tuyệt đối không được có False Negative** (bỏ sót lỗi nghiêm trọng). Do đó:

- Gate ngưỡng **nghiêng về Recall cao** (bắt hết lỗi) hơn là Precision cao (chỉ báo khi chắc chắn)
- Khi không chắc chắn → trả `HOLD` (yêu cầu human review) thay vì `PASS`

### 5.3. 6 Chỉ số Đánh giá Kết quả GenAI

Vận dụng từ **CT-GenAI §5 (Evaluation Metrics & Methodology)**:

| # | Chỉ số | Định nghĩa | Công thức / Đo bằng | Ngưỡng TI |
| :---: | :--- | :--- | :--- | :--- |
| 1 | **GroundednessScore** | Test candidate có bám sát artifact không? | Amazon Bedrock Evaluations | ≥ 0.80 |
| 2 | **GoalSuccessRate** | Candidate có đạt được mục tiêu kiểm thử không? | Bedrock Evaluations + Human spot-check | ≥ 0.75 |
| 3 | **Precision (Gate)** | Trong số lần TI báo lỗi, bao nhiêu % là lỗi thật? | TP / (TP + FP) | ≥ 0.70 |
| 4 | **Recall (Gate)** | Trong số lỗi thật, TI bắt được bao nhiêu %? | TP / (TP + FN) | ≥ **0.95** (ưu tiên) |
| 5 | **HallucinationRate** | Tỷ lệ candidate bịa đặt URL/endpoint không có trong artifact | Tổng candidate vi phạm Law 13–14 / Tổng candidate | ≤ 0.02 (2%) |
| 6 | **InjectionResistanceRate** | Tỷ lệ payload injection bị chặn thành công bởi 4-barrier | Blocked / Total injection attempts | ≥ 0.99 (99%) |

### 5.4. 4 Data Quality Gates áp dụng vào S08 Evidence Store

Vận dụng từ **CT-AI §5.1 (Input Data Testing)** — áp cho Evidence chứ không chỉ cho training data:

| Gate | Vị trí trong Pipeline TI | Tiêu chí đạt | Hành động khi không đạt |
| :--- | :--- | :--- | :--- |
| **G1 — Evidence Schema** | Sau khi runner trả raw result | Evidence JSON phải validate theo Evidence Envelope Schema (định nghĩa bên dưới) | Reject, yêu cầu runner chạy lại |
| **G2 — Evidence Completeness** | Trước khi S09 chạy | 100% domain runner đã có Evidence trong S08 (hoặc runner explicitly skipped với lý do) | Block S09 |
| **G3 — Evidence Integrity** | Tại S09 | SHA-256 hash của raw result khớp với hash trong Evidence Envelope | Invalidate Evidence, reject job |
| **G4 — Evidence Freshness** | Tại S09 | Evidence không quá 24h tuổi (stale evidence không được dùng) | Yêu cầu re-run |

**Evidence Envelope Schema** — Interface contract cho Nhóm 1 (Trang) implement G1:

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "TI Evidence Envelope v1",
  "type": "object",
  "required": ["envelope_id", "job_id", "domain", "runner_tool", "truth_class",
               "raw_result_s3_uri", "raw_result_sha256", "collected_at_utc",
               "deterministic_assertions", "gate_eligible"],
  "properties": {
    "envelope_id":          { "type": "string", "pattern": "^EVD-[A-Z]+-[0-9]{14}-[A-Z0-9]{6}$" },
    "job_id":               { "type": "string" },
    "domain":               { "type": "string", "enum": ["API", "UI", "DATABASE", "PERFORMANCE", "SECURITY", "AI_MODEL"] },
    "runner_tool":          { "type": "string" },
    "truth_class":          { "type": "string", "enum": ["OBSERVED"] },
    "raw_result_s3_uri":    { "type": "string", "pattern": "^s3://" },
    "raw_result_sha256":    { "type": "string", "pattern": "^[a-f0-9]{64}$" },
    "collected_at_utc":     { "type": "string", "format": "date-time" },
    "deterministic_assertions": {
      "type": "array",
      "minItems": 1,
      "items": {
        "required": ["assertion_id", "result", "actual_value"],
        "properties": {
          "assertion_id": { "type": "string" },
          "result":       { "type": "string", "enum": ["PASS", "FAIL"] },
          "actual_value": {}
        }
      }
    },
    "gate_eligible":        { "type": "boolean" },
    "skip_reason":          { "type": "string" }
  }
}
```

> **Quy ước đặt tên:** `EVD-API-20260922143000-A3X7K2`. Mọi runner phải emit JSON khớp schema này trước khi ghi vào S08. Nhóm 1 dùng schema JSON này làm contract cứng — không tự suy diễn schema.

### 5.5. Ground Truth Benchmark Loop

> Vận dụng **CT-GenAI §5 — "Bài học về Độc lập Đo lường"**: Không bao giờ tin vào số liệu benchmark tự công bố bên ngoài.

TI phải xây dựng vòng lặp đối chứng nội bộ:

```
┌─────────────────────────────────────────────────────────────────┐
│                  GROUND TRUTH BENCHMARK LOOP                    │
│                                                                 │
│  1. Chọn tập Repository Chuẩn (Gold Standard Repos)             │
│     → 10-20 repo từ TechX/Xora đã biết trước defect thật        │
│                                                                 │
│  2. Chạy TI tự động trên Gold Standard Repos                    │
│     → Thu thập Gate Recommendation và Evidence                  │
│                                                                 │
│  3. So sánh với Manual Testing Ground Truth                     │
│     → Tính Precision, Recall, HallucinationRate thực tế         │
│                                                                 │
│  4. Điều chỉnh Prompt Policy S05/S06 nếu metrics lệch          │
│     → Cập nhật Capability Manifest (token_ceiling, thresholds)  │
│                                                                 │
│  5. Lặp lại mỗi Sprint hoặc khi thay đổi model tier            │
└─────────────────────────────────────────────────────────────────┘
```

**Tần suất chạy Ground Truth Loop:**
- Khi deploy model tier mới (ví dụ: nâng Sonnet 5 → Opus 5 cho S05)
- Khi thay đổi Prompt Policy S05/S06
- Định kỳ mỗi 2 sprint (4 tuần)

---

## 6. Entry & Exit Criteria

> **Nguồn lý luận:** CTFL §5.2 — "Test plan tốt xác định tiêu chí vào/ra rõ ràng"

### 6.1. Entry Criteria — Điều kiện để bắt đầu pipeline TI

| Level | Điều kiện bắt buộc | Kiểm tra bởi |
| :--- | :--- | :--- |
| **Job Level** | Artifact đã được submit qua TI API, PinnedContext đã xác nhận | S01 Target Registry |
| **Test Planning** | S02 Change Detector đã trích xuất Changeset thành công | S02 output |
| **Test Execution** | Risk Tier đã được xác định (S04), Test Candidate đã được sinh (S06), Candidate truth_class = CANDIDATE | S04 + S06 output |
| **Gate Evaluation** | Tất cả runner trong S07 đã trả Evidence (OBSERVED) hoặc SKIP với lý do | S08 completeness check |

### 6.2. Exit Criteria — Điều kiện để kết thúc và ra quyết định

#### Điều kiện PASS:
- [ ] Tất cả deterministic assertions của 6 domains đều PASS
- [ ] `CRITICAL_COUNT == 0` và `SECRETS_LEAKED == 0` (Security — hard stop)
- [ ] `HIGH_COUNT == 0` HOẶC có approved Waiver Document từ Architecture Authority
- [ ] P95 latency ≤ `${EVAL_PACK.perf.p95_ms}` (Performance)
- [ ] `BROWSER_CONSOLE_ERRORS == 0` (UI)
- [ ] `ACCESSIBILITY_VIOLATIONS == 0` (UI)
- [ ] `GroundednessScore ≥ 0.80` cho mọi candidate được dùng

#### Điều kiện DO_NOT_PASS (cứng — tuyệt đối không thể override):
- [ ] `CRITICAL_COUNT > 0` — bất kỳ lỗ hổng Critical nào
- [ ] `SECRETS_LEAKED > 0` — bất kỳ secret bị lộ
- [ ] P95 latency > `${EVAL_PACK.perf.p95_ms}`
- [ ] Migration rollback thất bại
- [ ] SHA-256 hash mismatch (Evidence tampered)

#### Điều kiện HOLD (yêu cầu human review + Waiver Document nếu muốn PASS):
- [ ] `HIGH_COUNT > 0` — lỗ hổng High chưa có Critical; **chỉ chuyển PASS khi có Waiver Document ký bởi Architecture Authority**
- [ ] P99 latency vượt `${EVAL_PACK.perf.p99_ms}` nhưng P95 OK
- [ ] `GroundednessScore < 0.80` cho ≥ 30% candidate
- [ ] Bất kỳ runner nào timeout mà không có SKIP lý do có căn cứ

---

## 7. Risk Register

> **Nguồn lý luận:** CTFL §5.5 — Risk-based Testing + CT-GenAI §6 — Technical Risk Management

| # | Rủi ro | Xác suất | Tác động | Chiến lược xử lý | Law / Principle |
| :--- | :--- | :---: | :---: | :--- | :--- |
| R01 | Model AI bịa đặt test candidate (Hallucination) | Trung bình | Cao | GroundednessScore gate ≥ 0.80; Back-to-Back testing giữa các model tier | CT-GenAI §5 |
| R02 | Artifact chứa Prompt Injection độc hại | Trung bình | Rất cao | 4-barrier Defense-in-Depth; artifact là UNTRUSTED | CT-GenAI §6, Law 12 |
| R03 | Performance test tự gây nghẽn TI API (Self-DoS) | Thấp | Rất cao | Hard quota: MAX_VUS=100, MAX_DURATION=300s, server-enforced | Law 5 |
| R04 | Migration chạy trên DB Production thật | Rất thấp | Thảm họa | Bắt buộc dùng Aurora Clone; không bao giờ trỏ vào production connection | Law 14 |
| R05 | Evidence bị giả mạo (tampered result) | Thấp | Rất cao | SHA-256 hash tại S08; `OBSERVED` truth class bất biến | Law 16 |
| R06 | Model tier không phù hợp với task phức tạp | Cao | Trung bình | Phân tầng Haiku/Sonnet/Opus theo task; Ground Truth Benchmark Loop | CT-GenAI §2 |
| R07 | Stale evidence dùng cho Gate sai | Trung bình | Cao | Evidence Freshness Gate (G4): không quá 24h | CTFL §5.2 |
| R08 | Test candidate lặp lại mãi mất hiệu lực | Cao | Trung bình | S10 Production Learning cập nhật catalog; Pesticide Paradox prevention | CTFL Nguyên tắc 5 |

---

## 8. Tổng kết & Khuyến nghị hành động

### 8.1. Ba nguyên tắc bất biến (kế thừa từ 3 tầng sách)

```
NGUYÊN TẮC 1 — TỪ CTFL: "SHIFT LEFT — PHÁT HIỆN SỚM, SỬA RẺ"
  → S01–S04 chạy trước S07. Evidence tích lũy từng bước.
  → Lỗi phát hiện ở S02 rẻ hơn 30× so với phát hiện ở production.

NGUYÊN TẮC 2 — TỪ CT-AI: "TOOL ĐO, KHÔNG PHẢI MODEL PHÁN"
  → Deterministic tools (Semgrep, k6, Playwright, Flyway) ra Evidence.
  → Model (Claude) chỉ sinh ý định (Intent) và tóm tắt lý do (Rationale).
  → Model KHÔNG được tự tuyên bố PASS hay override kết quả đo của tool.

NGUYÊN TẮC 3 — TỪ CT-GENAI: "ĐÁNH GIÁ AI BẰNG SỐ — KHÔNG BẰNG CẢM TÍNH"
  → GroundednessScore, GoalSuccessRate, Recall, HallucinationRate
  → Đo bằng Amazon Bedrock Evaluations, không đo bằng "có vẻ đúng"
  → Ground Truth Benchmark Loop nội bộ mỗi 4 tuần
```

### 8.2. Bảng tóm tắt Framework đáng tin cậy

| Lớp | Công cụ đảm bảo | Chỉ số đo | Nguồn sách |
| :--- | :--- | :--- | :--- |
| **Test Planning** | Master Test Plan + Risk Tier | Risk Tier distribution | CTFL §2 |
| **Test Design** | EP + BVA + State Transition + Decision Table | Coverage per domain | CTFL §4 |
| **AI Quality** | 6 Quality Chars + Confusion Matrix | Recall ≥ 0.95, FN → 0 | CT-AI §4 |
| **Data Quality** | 4 Quality Gates (G1–G4) | SHA-256 + Freshness | CT-AI §5.1 |
| **Model Quality** | Bedrock Evaluations + Back-to-Back | GroundednessScore ≥ 0.80 | CT-GenAI §5 |
| **Security** | 4-barrier Defense + Zero Tolerance Gate | CRITICAL==0, SECRETS==0 | CT-GenAI §6 |
| **Continuous Improvement** | Ground Truth Benchmark Loop | Precision/Recall trend | CT-GenAI §5 + CTFL Nguyên tắc 5 |

### 8.3. Recommended Actions — Việc cần làm tiếp theo

> **Cập nhật tiến độ (22/09/2026):** Task 3 (Nghĩa) đã hoàn thành đồng bộ **v2.1.0** — loại bỏ Hurl, Testcontainers, Pixelmatch; tích hợp Playwright API, Aurora Clone, CodeGuru theo đúng stack đã thống nhất với Task 1 và Task 2. Các action items dưới đây đã được điều chỉnh tương ứng.

| Priority | Action | Owner đề xuất | Trạng thái | Input từ |
| :--- | :--- | :--- | :--- | :--- |
| 🔴 P0 | Định nghĩa và freeze `Evaluation Pack` (ngưỡng SLO **per tenant type** — không hard-code) | Hùng + Mentor | ⏳ Chờ | Task 4 này |
| 🔴 P0 | Triển khai 4 Evidence Quality Gates vào S08 — dùng Evidence Envelope Schema v1 trong Mục 5.4 làm contract | Trang (Nhóm 1) | ⏳ Chờ | Task 1 + Task 4 §5.4 |
| 🔴 P0 | Task 2 (Hoàng): Cập nhật v0.2 theo 4 điểm mentor — Semgrep+Trivy vào W1; DB domain (Aurora Clone+Flyway); bỏ Network Firewall → Private Subnet+SG; vẽ lại sơ đồ handshake | Hoàng (Nhóm 2) | ⏳ Chờ | Feedback mentor Task 2 |
| 🟠 P1 | Xây dựng Gold Standard Benchmark Repo (10 repos chuẩn với known defects) để chạy Ground Truth Loop | Hùng | ⏳ Chờ | Task 4 §5.5 |
| 🟠 P1 | Phối hợp với Nghĩa chốt ngưỡng `Evaluation Pack` cho từng tenant type (Fintech / Healthcare / SaaS) | Hùng + Nghĩa | ⏳ Chờ | Task 3 v2.1.0 + Task 4 |
| ✅ P1 | ~~Cài đặt `token_ceiling`, `guardrail_policy`, `model_profile` vào 3 Capability Manifests~~ | Nghĩa (Nhóm 3) | **DONE — Task 3 v2.1.0** | Task 3 đã hoàn tất |
| 🟡 P2 | Calibrate Model Tier thực tế (Haiku vs Sonnet vs Opus) trên Gold Standard Repos sau khi xây xong | Hùng + Nghĩa | ⏳ Chờ | Task 4 §5.5 |
| 🟢 P3 | Tích hợp Pesticide Paradox prevention vào S10 Production Learning | Hoàng (Nhóm 2) | ⏳ Chờ | Task 4 §7 R08 |

---

## CHANGELOG

| Version | Ngày | Thay đổi |
| :--- | :--- | :--- |
| v1.0.0 | 22/09/2026 | Draft ban đầu |
| v1.2.0 | 22/09/2026 | Cập nhật theo feedback Nghĩa (5 items): (1) Sửa tác giả Task 2 = Hoàng (Architecture); (2) Bổ sung Dual-Mode API Testing (Schemathesis fuzzing + Playwright API/httpx cho API_CANDIDATE_V1); (3) Nâng cấp Security 4.2.5 thành Hybrid Defense 4-tầng (Tầng 0 AWS Native CodeGuru+Inspector → Tầng 1 Semgrep+Trivy+Gitleaks → Tầng 2 Opus 5 → Tầng 3 Manual Pentest); (4) Chuẩn hóa CRITICAL→DO_NOT_PASS (hard stop) vs HIGH→HOLD (cần Waiver Document) tại 4.2.5 và 6.2; (5) Cập nhật 8.3 ghi nhận Task 3 v2.1.0 đã hoàn thành đồng bộ |

---
  
*Document: Task_4_Test_Plan_Strategy_Evaluation.md*
*Author: Hùng — Nhóm 4 (Testing Strategy)*
*Date: 22 September 2026 | Version: v1.1.0*
*References: ISTQB CTFL v4.0.1 · CT-AI v2.0 · CT-GenAI v1.1 · Task 1 (Trang) · Task 2 (Nghĩa) · Task 3 (Nghĩa) · Task_2_Architecture_Review_Feedback.md (Mentor)*
