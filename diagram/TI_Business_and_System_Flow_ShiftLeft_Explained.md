# TESTING INTELLIGENCE (TI) — BẢN GIẢI THÍCH LUỒNG FLOW END-TO-END SHIFT-LEFT
## CẦU NỐI GIỮA LUỒNG NGHIỆP VỤ (BUSINESS FLOW), CÂU HỎI QA/QC VÀ KIẾN TRÚC KỸ THUẬT (v2.1 CANDIDATE)

> **Mục tiêu:** Giúp toàn bộ các bên liên quan (Developer, QA/QC, DevOps, Solution Architect, Product Owner) cùng nhìn chung **một bức tranh duy nhất**:
> - Từ lúc Developer có một thay đổi code/PR $\rightarrow$ TI can thiệp ở đâu?
> - QA/QC cần trả lời câu hỏi gì về chất lượng?
> - Hệ thống tạo ra bằng chứng gì để chứng minh?
> - Ai sử dụng kết quả đó và ảnh hưởng ra sao tới quyết định phát hành (Release)?
>
> ⚠️ **RANH GIỚI SỰ THẬT (ADR-0002 / Law 18):**
> Bản vẽ và luồng dưới đây là **THIẾT KẾ ĐÍCH (CANDIDATE)**. Hiện trạng đo kiểm thực tế (`OBSERVED 22/09`): Hệ thống vẫn chạy trên 1 EC2 duy nhất (API :8000 + Portal :8001), kết quả ghi ổ EBS. Quy tắc cốt lõi: `CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · `completed ≠ PASS`.

---

## 1. BỨC TRANH TOÀN CẢNH TRONG 60 GIÂY (BUSINESS & TECHNICAL MAPPING)

```text
  [DEVELOPER / CI-CD]
         │ (Changeset: Git diff, OpenAPI, Flyway SQL)
         ▼ [01] POST /v2/artifact-jobs
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ GIAI ĐOẠN 1: TIẾP NHẬN & CÁP LỆNH KHÔNG CHẶN (TI Control Plane — Account A)                      │
│ • CloudFront + WAF ──► TI API v2 phản hồi ngay HTTP 202 Accepted (<1s) kèm job_id               │
│ • Lưu trạng thái ban đầu: queued vào RDS PostgreSQL; bàn giao cho Job Controller                │
└────────────────────────────────┬─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼ [06] Chạy trước trong Sandbox
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ GIAI ĐOẠN 2: THẨM ĐỊNH TĨNH & SUY LUẬN AI (Shift-Left Pre-Scan & Semantic Brain)                │
│ • [06-07] Trục 1 Sandbox: Semgrep OSS + Trivy + Gitleaks quét tĩnh SAST/CVE/Secrets (1 lần duy nhất)│
│ • [07A/B] Xuất SARIF lên S3, trả SecurityFindings thô về Job Controller                         │
│ • [08A/B] Job Controller chuyển context sang Account B; Claude Sonnet 5 suy luận Blast Radius    │
│   (Opus 5 chỉ kích hoạt khi Risk==CRITICAL để Threat Modeling)                                   │
│ • [09A-C] Bedrock Evaluations chấm điểm độc lập (Groundedness ≥0.80, Faithfulness ≥0.85)         │
│   ──► Trả về Verified ImpactSet + ToolIntent JSON (Symbolic IDs, KHÔNG chứa secret/URL thật)     │
└────────────────────────────────┬─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼ [10] Smart Dispatch: Verified ImpactSet ∩ TargetBinding
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ GIAI ĐOẠN 3: THỰC THI KIỂM THỬ CÔ LẬP THEO PHẠM VI ẢNH HƯỞNG (Sandbox Execution VPC)             │
│ • Job Controller là "Workflow Authority": Tra cứu TenantBinding nội suy URL thật & cấp STS ngắn hạn│
│ • Kích hoạt ECS Fargate RunTask đúng các Trục bị tác động; các trục khác: SKIPPED (Tiết kiệm 100%)│
│   ┌────────────────┬────────────────┬────────────────┬────────────────┬──────────────────────┐   │
│   │ Trục 2: API    │ Trục 3: UI     │ Trục 4: DB     │ Trục 5: Perf   │ Trục 6: D5b DAST     │   │
│   │ Schemathesis   │ Playwright/axe │ Aurora v2/Dynam│ k6 (3 Khóa)    │ ZAP (Wave 3 Staging) │   │
│   └────────────────┴────────────────┴────────────────┴────────────────┴──────────────────────┘   │
│ • [11A] Direct-to-S3 Offloading: Runners đẩy raw logs, video MP4, traces thẳng S3 Object Lock    │
│ • [11B] Envelope ~2KB: Runners chỉ gửi mã băm SHA-256 Digest và exit code về Job Controller     │
└────────────────────────────────┬─────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼ [12A-D] Phán quyết chất lượng
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ GIAI ĐOẠN 4: RA PHÁN QUYẾT GATE & BÀN GIAO CHO QUYẾT ĐỊNH RELEASE (Account A & Consumers)        │
│ • S09 Decision Gate chạy mã tất định (Critical==0, p95<SLA, Faithfulness>=0.85)                  │
│ • Cập nhật trạng thái: state=completed VÀ gate_result tách biệt hoàn toàn (Law 18: completed≠PASS)│
│   ┌───────────────────────────┬─────────────────────────────────┬────────────────────────────┐   │
│   │ 🟢 PASS: An toàn tuyệt đối│ 🟡 HOLD + Waiver: QA Lead thẩm  │ 🔴 DO_NOT_PASS: Hard-stop  │   │
│   │    Đủ tiêu chuẩn kiểm thử │    định và ghi nhận ngoại lệ    │    Chặn đứng merge PR ngay │   │
│   └───────────────────────────┴─────────────────────────────────┴────────────────────────────┘   │
│ • [12C] Tri thức GOLDEN lưu vào S10 Memory (UNVERIFIED) · [12D] CI/CD poll kết quả qua API v2    │
│ • Bàn giao Evidence Pack cho Release Authority (XoraOps) ra quyết định triển khai Production     │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. BỐI CẢNH NGHIỆP VỤ & 6 CÂU HỎI CHẤT LƯỢNG CỦA QA/QC

Theo phản biện của anh **Nguyễn Thành Đạt**, để hệ thống TI thực sự mang lại giá trị thực tế, mỗi component trên kiến trúc phải trả lời được một **câu hỏi chất lượng cụ thể** nhằm phục vụ quyết định của con người:

| Bước Luồng Nghiệp Vụ | Câu Hỏi Cốt Tử Của QA/QC | Bằng Chứng Cần Thu Thập | Component Kiến Trúc Đảm Nhiệm | Hỗ Trợ Quyết Định Nào? |
|:---|:---|:---|:---|:---|
| **1. Khi Dev push PR** | *Thay đổi này có vô tình làm lộ API Key, Secret hay chứa thư viện có lỗ hổng CVE nghiêm trọng không?* | File báo cáo chuẩn **SARIF v2.1** (chứa CWE, CVE, Secret path) | **Trục 1: D5a Security**<br>(Semgrep + Trivy + Gitleaks) | Nếu có Secret/Critical $\rightarrow$ **DO_NOT_PASS (Chặn PR ngay lập tức)** |
| **2. Phân tích tác động** | *Thay đổi chạm vào những dịch vụ nào? AI đề xuất kịch bản test có bị ảo giác hay bịa đặt không?* | Báo cáo Bedrock Evaluations (`Groundedness >= 0.80`, `Faithfulness >= 0.85`) | **Account B AI Brain**<br>(Claude Sonnet 5 + Bedrock Eval) | Nếu `Faithfulness < 0.85` $\rightarrow$ **HOLD (Tạm giữ kịch bản, không dispatch bừa)** |
| **3. Kiểm thử API** | *Các thay đổi backend có làm sai lệch hợp đồng dữ liệu hoặc sập server khi gặp payload dị thường không?* | Danh sách Schema validation errors, crash payload log, exit code | **Trục 2: API Functional**<br>(Schemathesis Fuzzing + Playwright) | Nếu có lỗi HTTP 5xx $\rightarrow$ **DO_NOT_PASS** |
| **4. Kiểm thử UI & A11y** | *Trải nghiệm người dùng có bị gián đoạn và giao diện có vi phạm tiêu chuẩn tiếp cận người khuyết tật không?* | Video quay lại màn hình MP4, screenshot lúc lỗi, axe-core WCAG JSON | **Trục 3: UI & Accessibility**<br>(Playwright + axe-core WCAG) | Nếu flow chính vỡ $\rightarrow$ **DO_NOT_PASS** |
| **5. Nâng cấp Database** | *Script Flyway migration có làm mất dữ liệu, khóa bảng hoặc rollback thất bại khi có sự cố không?* | Log thực thi migration, schema diff snapshot, rollback verification | **Trục 4: DB Dual Isolation**<br>(Aurora Serverless v2 + DynamoDB) | Nếu rollback fail $\rightarrow$ **DO_NOT_PASS** |
| **6. Đánh giá Hiệu năng** | *Thời gian phản hồi p95 có đáp ứng SLA dưới tải lớn mà không làm nghẽn hạ tầng dùng chung không?* | k6 Summary JSON, biểu đồ phân bổ độ trễ, số liệu lỗi HTTP 5xx | **Trục 5: Performance Testing**<br>(AWS DLT + k6 Engine) | Nếu `p95 > SLA` $\rightarrow$ **HOLD (Chờ QA Lead duyệt Waiver)** |

---

## 3. ĐỐI CHIẾU 1-1 VỚI SƠ ĐỒ KỸ THUẬT `[01]` ĐẾN `[12D]`

| Mã Bước | Hành Động Kỹ Thuật | Phân Vùng | Trách Nhiệm Xử Lý & Ranh Giới An Toàn |
|:---:|:---|:---:|:---|
| **`[01]`** | `POST /v2/artifact-jobs` | External $\rightarrow$ Account A | CI/CD gửi git diff, OpenAPI, SQL migration kèm mã băm SHA-256 digest. |
| **`[02]`** | Forward qua CloudFront & WAF | Edge Group | WAF áp dụng OWASP rules & rate limit; HMAC được verify tại API / Lambda@Edge. |
| **`[03]`** | HTTP `202 Accepted` < 1s | TI API v2 | Trả ngay phản hồi kèm `job_id` và URL polling; không bắt CI runner phải treo máy. |
| **`[04]`** | Khởi tạo state: `queued` | Amazon RDS | Ghi sổ bản ghi kiểm thử mới với trạng thái chuẩn `queued` (khớp Portal). |
| **`[05]`** | Enqueue Job vào Controller | Job Controller | Chuyển trạng thái sang `running`; Job Controller nắm quyền điều phối (Law 4.3). |
| **`[06]`** | ECS RunTask Trục 1 (D5a) | Sandbox VPC | Chạy pre-scan Semgrep + Trivy + Gitleaks đúng 1 lần trong môi trường cấm Internet. |
| **`[07A]`**| Direct-to-S3: SARIF Báo Cáo | Trục 1 $\rightarrow$ S3 | Đẩy file kết quả kiểm toán an ninh tĩnh trực tiếp lên S3 qua Gateway Endpoint ($0). |
| **`[07B]`**| Gửi `SecurityFindings` thô | Trục 1 $\rightarrow$ Controller | Trả danh sách lỗ hổng về Controller; Trục 1 không tự ý suy diễn nghiệp vụ. |
| **`[08A]`**| STS AssumeRole sang AI Brain | Account A $\rightarrow$ B | Chỉ gửi Context và SecurityFindings; tuyệt đối không mang Secret/URL thật. |
| **`[08B]`**| Claude Sonnet 5 Review | Account B | Phân tích Blast Radius sinh `ImpactSet` + TestPlan; Opus 5 chỉ gọi khi CRITICAL. |
| **`[09A]`**| Gửi sang Bedrock Evaluations | Bedrock $\rightarrow$ Eval | Đo bộ 6 chỉ số GenAI (Groundedness ≥0.80, Faithfulness ≥0.85, Dung sai ±0.03). |
| **`[09B]`**| Lưu Candidate Test lên S3 | Eval $\rightarrow$ S3 | Lưu kịch bản kiểm thử hợp lệ vào kho bằng chứng S3 Evidence. |
| **`[09C]`**| Trả về `Verified ImpactSet` | Eval $\rightarrow$ Controller | Báo cáo Quality OK kèm `Verified ImpactSet` và `ToolIntent JSON` biểu tượng. |
| **`[10]`** | **Smart Dispatch** | Controller $\rightarrow$ Sandbox | Kích hoạt đúng tập hợp: $\mathbf{Target = Verified\ ImpactSet \cap TargetBinding}$. |
| **`[11A]`**| Direct-to-S3: Raw Evidence | Runners $\rightarrow$ S3 | Trút toàn bộ video MP4, logs, Playwright traces lên S3 Object Lock (WORM 90 ngày). |
| **`[11B]`**| Envelope Metadata ~2KB | Runners $\rightarrow$ Controller | Chỉ gửi exit code và SHA-256 digest về Controller; triệt tiêu 100% nghẽn ổ đĩa. |
| **`[12A]`**| Bàn giao số liệu vào Gate | Controller $\rightarrow$ Gate | Kiểm tra tính toàn vẹn SHA-256 và chuyển metrics sang S09 Decision Gate. |
| **`[12B]`**| Phán quyết Gate 3 Nhánh | S09 Decision Gate | Tách biệt `state=completed` và `gate_result` (`PASS` / `HOLD+Waiver` / `DO_NOT_PASS`). |
| **`[12C]`**| Index tri thức GOLDEN | Gate $\rightarrow$ S10 Memory | Test case đạt `PASS` được đưa vào Knowledge Base Memory (nhãn `UNVERIFIED`). |
| **`[12D]`**| CI/CD Polling Kết Quả | CI $\rightarrow$ API v2 $\rightarrow$ RDS | CI pipeline định kỳ gọi API lấy phán quyết cuối cùng để merge hoặc block PR. |

---

## 4. QUY TRÌNH CON NGƯỜI: WAIVER VÀ QUYẾT ĐỊNH PHÁT HÀNH (HUMAN-IN-THE-LOOP)

Một ranh giới kiến trúc cực kỳ quan trọng được nhấn mạnh trong phiên bản v2.1 CANDIDATE là sự phân định giữa **Khuyến nghị Kiểm thử (Gate Recommendation)** và **Quyết định Phát hành (Release Decision)**:

```text
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ KẾT QUẢ GATE S09 LÀ "HOLD" (Ví dụ: p95=520ms vượt SLA 500ms, hoặc Faithfulness=0.83 tiệm cận)    │
└─────────────────────────────────┬────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ THẨM ĐỊNH NGOẠI LỆ (QA Lead / Security Authority)                                                │
│ • QA Lead kiểm tra Evidence Pack trên S3 và đối chiếu rủi ro thực tế                             │
│ • Nếu chấp thuận rủi ro: Gửi POST /v2/operations/{id}/actions (Submit Approved Waiver)           │
│ • Hệ thống TI ghi nhận bản ghi kiểm toán "WaiverDecision" (Người duyệt, Lý do, Thời điểm)         │
│ • Trạng thái gate_result vẫn được bảo lưu là "HOLD" kèm cờ "Waiver_Granted"                      │
└─────────────────────────────────┬────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│ QUYẾT ĐỊNH PHÁT HÀNH (Release Authority / XoraOps)                                               │
│ • Release Authority căn cứ vào Evidence Pack từ TI + biên bản WaiverDecision của QA Lead        │
│ • Ban hành quyết định chính thức: Triển khai (Deploy) hoặc Hủy bỏ (Abort) lên Production         │
│ ⚠️ TI KHÔNG TỰ ĐỘNG CHUYỂN TRẠNG THÁI RELEASE THÀNH "PASS" KHI CHƯA CÓ QUYỀN HẠN PHÁT HÀNH       │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 5. TỔNG KẾT: 3 ĐIỀU MỌI THÀNH VIÊN CẦN GHI NHỚ

1. **AI chỉ là Bộ Não Tư Duy, Không Có Quyền Lực Thực Thi:** Mô hình AI ở Account B chỉ đề xuất kế hoạch và kịch bản dưới dạng biểu tượng (`ToolIntent JSON`). Job Controller ở Account A mới là **Cơ quan thẩm quyền tối cao (Workflow Authority)** đối soát và dispatch.
2. **Tiết Kiệm Tối Đa Nhờ Smart Dispatch:** PR chạm vào module nào thì chỉ chạy đúng Trục Runner của module đó ($\mathbf{ImpactSet \cap TargetBinding}$). Không bao giờ chạy dàn trải cả 6 runner một cách lãng phí.
3. **Bằng Chứng Bất Biến & Tách Biệt Trạng Thái:** Toàn bộ kết quả kiểm thử được khóa bằng `S3 Object Lock` (WORM 90 ngày) và băm `SHA-256`. Trạng thái job kết thúc (`completed`) hoàn toàn độc lập với phán quyết chất lượng (`gate_result`), tuân thủ nghiêm ngặt **Law 18: `completed ≠ PASS`**.
