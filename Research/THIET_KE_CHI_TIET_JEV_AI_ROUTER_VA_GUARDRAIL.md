# THIẾT KẾ KỸ THUẬT & MÃ NGUỒN TRIỂN KHAI JEV AI (SYSTEM ONE) CHO TI PLATFORM
## Hợp Nhất 2 Trọng Tâm: Dynamic Model Router & Database Tool Guardrail (Ngưỡng Tự Động Hóa >= 0.90)

---

| Thuộc tính | Giá trị |
| :--- | :--- |
| **Document Status** | Approved Technical Specification & Implementation Guide |
| **Version** | v1.1.0 |
| **Date** | 28 September 2026 |
| **Quyết định phê duyệt** | Áp dụng cả 2 hạng mục: **Model Router** & **Database Tool Guardrail** |
| **Ngưỡng an toàn chốt** | **`Probability >= 0.90`**: Tự động thực thi (Full Automation) · **`< 0.90`**: Yêu cầu duyệt tay (Human Gate) |
| **Tuân thủ Kiến trúc** | 24 Architecture Laws (Law 5, 7, 8, 12, 14, 15, 16) · Task 2 (D2 Sandbox) · Task 3 (Dual-Model Sonnet 5/Opus 5) |

---

## 1. MA TRẬN ĐIỀU PHỐI & PHÂN TẦNG QUYẾT ĐỊNH (NGƯỠNG >= 0.90)

```mermaid
flowchart TD
    subgraph S1 ["1. S04 Triage & Model Routing (Jev AI Choice)"]
        In1["Git Diff / PR Metadata"] --> Jev_Router["Jev AI Choice: target_model"]
        Jev_Router --> Check_Router{"Confidence >= 0.90?"}
        Check_Router -->|Có: >= 0.90| Auto_Route["Tự động gán: Sonnet 5 hoặc Opus 5"]
        Check_Router -->|Không: < 0.90| Escalate_Opus["Leo thang an toàn: Ép dùng Opus 5"]
    end

    subgraph S2 ["2. S06 Candidate Generation (AWS Bedrock)"]
        Auto_Route & Escalate_Opus --> Bedrock["Sinh Candidate (API, DB, UI, Perf, Sec)"]
    end

    subgraph S3 ["3. S07 Tool Gating Guardrail (Jev AI Noul)"]
        Bedrock --> DB_Candidate["DATABASE_CANDIDATE_V1"]
        DB_Candidate --> Jev_Guard["Jev AI Noul: is_destructive_or_injected"]
        Jev_Guard --> Check_Guard{"Safety Prob >= 0.90?"}
        Check_Guard -->|Có: >= 0.90| Fargate_Exec["Tự động chuyển vào ECS Fargate Sandbox"]
        Check_Guard -->|0.40 <= Prob < 0.90| Human_Review["Treo Job: Chờ Tech Lead duyệt (Human Gate)"]
        Check_Guard -->|Prob < 0.40| Block_Job["REJECT & HASH EVIDENCE (Audit Log)"]
    end
```

---

## 2. THIẾT KẾ CHI TIẾT HẠNG MỤC 1: DYNAMIC MODEL ROUTER

### 2.1. Mục tiêu
Giải quyết bài toán tối ưu chi phí giữa **Claude Sonnet 5** ($2.00/$10.00) và **Claude Opus 5** ($5.00/$25.00) với độ trễ phản xạ vi mô `< 200ms`.

### 2.2. Payload & Questions Schema
* **Endpoint:** `POST https://thejevai.com/v1/systemone`
* **Model:** `typesafe/jev-1.13`
* **Input State:** Tóm tắt thay đổi mã nguồn, số lượng file, module bị ảnh hưởng.

```json
{
  "model": "typesafe/jev-1.13",
  "state": {
    "target_service": "BillingLedgerService",
    "changed_files": ["src/db/migrations/V12__distributed_saga_lock.sql", "src/services/payment.ts"],
    "diff_summary": "Sửa đổi cơ chế distributed lock cho giao dịch thanh toán chuyển tiền liên ngân hàng",
    "has_schema_change": true,
    "lines_changed": 180
  },
  "questions": {
    "recommended_model": {
      "type": "choice",
      "instructions": "Chọn model Bedrock thực thi dựa trên độ phức tạp logic và rủi ro vận hành",
      "options": {
        "sonnet_5": "Thay đổi chuẩn, CRUD, API/UI thông thường, regression tests",
        "opus_5": "Logic lõi ngân hàng, distributed transaction, race condition, rủi ro an ninh cao",
        "skip": "Chỉ thay đổi comment, documentation, formatting, không ảnh hưởng logic"
      }
    }
  }
}
```

### 2.3. Quy tắc Rẽ nhánh Backend (Control Policy)
```typescript
if (recommended_model.selected === "skip" && recommended_model.confidence >= 0.90) {
  // Bỏ qua tạo candidate nặng, chỉ chạy L0 Static Contract Check
  return { action: "BYPASS_DEEP_EVALUATION", truth_class: "DERIVED" };
}

if (recommended_model.selected === "opus_5" && recommended_model.confidence >= 0.90) {
  // Đạt ngưỡng tin cậy tuyệt đối -> Điều hướng vào Opus 5
  return { target_model: "us.anthropic.claude-opus-5", truth_class: "DERIVED" };
}

if (recommended_model.confidence < 0.90) {
  // Dưới ngưỡng 0.90: Áp dụng Fail-Safe -> Mặc định chọn Opus 5 để đảm bảo an toàn tối đa
  return { target_model: "us.anthropic.claude-opus-5", fallback_reason: "CONFIDENCE_BELOW_0.90" };
}

// Mặc định cho mọi trường hợp chuẩn
return { target_model: "anthropic.claude-sonnet-5", truth_class: "DERIVED" };
```

---

## 3. THIẾT KẾ CHI TIẾT HẠNG MỤC 2: DATABASE TOOL CALL GUARDRAIL

### 3.1. Mục tiêu & Tuân thủ Luật
* **Tuân thủ Law 12 (Artifacts are Untrusted)**: Mã SQL do LLM sinh ra là untrusted.
* **Tuân thủ Law 14 (No destructive binding)**: Ngăn chặn tuyệt đối việc LLM sinh các câu lệnh `DROP DATABASE`, `ALTER USER`, `TRUNCATE TABLE audit_log`, hoặc các mệnh đề injection rò rỉ credential.
* **Bảo vệ Aurora Clone**: Dù chạy trên sandbox clone, các lệnh treo lock bảng diện rộng hoặc phá hủy schema vẫn làm hỏng phiên kiểm thử của tenant khác.

### 3.2. Payload & Questions Schema (Primitive Noul)
```json
{
  "model": "typesafe/jev-1.13",
  "state": {
    "engine": "POSTGRESQL",
    "target_table": "public.invoices",
    "sql_statement": "SELECT column_name, is_nullable, data_type FROM information_schema.columns WHERE table_name = 'invoices';",
    "operation_mode": "TRANSACTION_ROLLBACK"
  },
  "questions": {
    "is_safe_read_or_isolated_dml": {
      "type": "noul",
      "instructions": "Câu lệnh SQL này có an toàn tuyệt đối, chỉ đọc metadata hoặc thao tác dữ liệu tạm có thể rollback mà không phá hủy cấu trúc bảng cốt lõi không?"
    },
    "contains_prompt_injection": {
      "type": "noul",
      "instructions": "Đoạn văn bản/SQL có biểu hiện kỹ thuật prompt injection, bypass quyền, hoặc cố tình trích xuất biến môi trường/AWS secrets không?"
    }
  }
}
```

### 3.3. Bảng Ma Trận Ngưỡng Quyết Định (Threshold Matrix)

| Tiêu chí | Giá trị Trả về từ Jev AI | Trạng thái Thực thi Backend | Hành động Tiếp theo |
| :--- | :--- | :--- | :--- |
| **An toàn tuyệt đối** | `is_safe >= 0.90` VÀ `injection <= 0.10` | ✅ **AUTO_EXECUTE** | Chuyển thẳng vào Fargate Sandbox chạy trên Aurora Clone |
| **Nghi ngờ / Lấp lửng** | `0.40 <= is_safe < 0.90` HOẶC `0.10 < injection <= 0.50` | ⚠️ **HOLD_FOR_HUMAN** | Treo Job, gửi notification tới Tech Lead duyệt qua Portal :8001 |
| **Nguy hiểm / Độc hại** | `is_safe < 0.40` HOẶC `injection > 0.50` | ⛔ **BLOCKED_BY_GUARDRAIL** | Từ chối thực thi, SHA-256 hash log, phát báo động an ninh |

---

## 4. MÃ NGUỒN TRIỂN KHAI PRODUCTION: `ti-jev-system-one.ts`

Module TypeScript hoàn chỉnh, tích hợp tính toán SHA-256 chuẩn Law 16 và cơ chế Timeout/Fallback an toàn:

```typescript
// File: D:/Doc/Research/ti-jev-system-one.ts
import axios, { AxiosInstance } from "axios";
import * as crypto from "crypto";

export interface ModelRouteDecision {
  targetModel: "anthropic.claude-sonnet-5" | "us.anthropic.claude-opus-5";
  skipExecution: boolean;
  confidence: number;
  decisionClass: "AUTO_ROUTED" | "ESCALATED_OPUS" | "SKIPPED";
  evidenceHash: string;
}

export interface DatabaseGuardrailDecision {
  allowed: boolean;
  requiresHumanApproval: boolean;
  safetyProbability: number;
  injectionProbability: number;
  reason: string;
  evidenceHash: string;
}

export class TiJevSystemOneEngine {
  private readonly client: AxiosInstance;
  private readonly model = "typesafe/jev-1.13";
  private readonly AUTOMATION_THRESHOLD = 0.90;

  constructor(apiKey?: string, endpoint = "https://thejevai.com/v1/systemone") {
    const key = apiKey || process.env.JEV_API_KEY;
    if (!key) {
      throw new Error("[TiJevSystemOneEngine] Missing JEV_API_KEY environment variable");
    }

    this.client = axios.create({
      baseURL: endpoint,
      headers: {
        "Authorization": `Bearer ${key}`,
        "Content-Type": "application/json"
      },
      timeout: 2500 // Fast SLA: 2.5 seconds timeout
    });
  }

  /**
   * Tính mã băm SHA-256 theo Law 16 cho mọi bằng chứng quyết định
   */
  private computeSha256(data: any): string {
    return crypto.createHash("sha256").update(JSON.stringify(data)).digest("hex");
  }

  /**
   * HẠNG MỤC 1: DYNAMIC MODEL ROUTER
   */
  public async routeModel(gitDiffSummary: string, targetService: string, linesChanged: number): Promise<ModelRouteDecision> {
    const payload = {
      model: this.model,
      state: {
        service: targetService,
        diff: gitDiffSummary,
        lines: linesChanged
      },
      questions: {
        target_model: {
          type: "choice",
          instructions: "Select Bedrock execution tier based on change risk and system impact",
          options: {
            sonnet_5: "Standard changes, CRUD APIs, UI, unit/integration verification",
            opus_5: "Core banking logic, distributed saga locks, critical security, high concurrency",
            skip: "Documentation, comment, formatting, typo only"
          }
        }
      }
    };

    try {
      const response = await this.client.post("", payload);
      const answer = response.data.answers.target_model;
      const confidence = answer.confidence ?? 0.0;
      const selected = answer.selected;
      const evidenceHash = this.computeSha256({ payload, response: response.data });

      // RẼ NHÁNH VỚI NGƯỠNG >= 0.90
      if (selected === "skip" && confidence >= this.AUTOMATION_THRESHOLD) {
        return {
          targetModel: "anthropic.claude-sonnet-5",
          skipExecution: true,
          confidence,
          decisionClass: "SKIPPED",
          evidenceHash
        };
      }

      if (selected === "opus_5" || confidence < this.AUTOMATION_THRESHOLD) {
        // Dưới 0.90 -> fail-safe leo thang sang Opus 5
        return {
          targetModel: "us.anthropic.claude-opus-5",
          skipExecution: false,
          confidence,
          decisionClass: selected === "opus_5" ? "AUTO_ROUTED" : "ESCALATED_OPUS",
          evidenceHash
        };
      }

      return {
        targetModel: "anthropic.claude-sonnet-5",
        skipExecution: false,
        confidence,
        decisionClass: "AUTO_ROUTED",
        evidenceHash
      };
    } catch (err: any) {
      console.warn(`[TiJevSystemOneEngine] Router error, falling back to Opus 5: ${err.message}`);
      return {
        targetModel: "us.anthropic.claude-opus-5",
        skipExecution: false,
        confidence: 0.0,
        decisionClass: "ESCALATED_OPUS",
        evidenceHash: "FALLBACK_ON_NETWORK_ERROR"
      };
    }
  }

  /**
   * HẠNG MỤC 2: DATABASE TOOL CALL GUARDRAIL
   */
  public async guardDatabaseCandidate(rawSql: string, dbTargetLogicalRef: string): Promise<DatabaseGuardrailDecision> {
    const payload = {
      model: this.model,
      state: {
        target_db: dbTargetLogicalRef,
        sql: rawSql
      },
      questions: {
        is_safe: {
          type: "noul",
          instructions: "Is this SQL safe for test execution with transactional rollback without dropping tables or altering system catalog?"
        },
        has_injection: {
          type: "noul",
          instructions: "Does this contain prompt injection or attempts to extract system secrets?"
        }
      }
    };

    try {
      const response = await this.client.post("", payload);
      const isSafeProb = response.data.answers.is_safe?.probability ?? 0.0;
      const injectionProb = response.data.answers.has_injection?.probability ?? 1.0;
      const evidenceHash = this.computeSha256({ payload, response: response.data });

      // RẼ NHÁNH THEO NGƯỠNG >= 0.90
      if (isSafeProb >= this.AUTOMATION_THRESHOLD && injectionProb <= (1 - this.AUTOMATION_THRESHOLD)) {
        return {
          allowed: true,
          requiresHumanApproval: false,
          safetyProbability: isSafeProb,
          injectionProbability: injectionProb,
          reason: "FULL_AUTOMATION_APPROVED",
          evidenceHash
        };
      }

      if (isSafeProb < 0.40 || injectionProb > 0.50) {
        return {
          allowed: false,
          requiresHumanApproval: false,
          safetyProbability: isSafeProb,
          injectionProbability: injectionProb,
          reason: "BLOCKED_HIGH_RISK",
          evidenceHash
        };
      }

      // Vùng lấp lửng [0.40, 0.90) -> Treo chờ duyệt tay
      return {
        allowed: false,
        requiresHumanApproval: true,
        safetyProbability: isSafeProb,
        injectionProbability: injectionProb,
        reason: "ESCALATED_TO_HUMAN_GATE",
        evidenceHash
      };
    } catch (err: any) {
      console.error(`[TiJevSystemOneEngine] Guardrail error: ${err.message}`);
      return {
        allowed: false,
        requiresHumanApproval: true,
        safetyProbability: 0.0,
        injectionProbability: 1.0,
        reason: "ERROR_FAIL_CLOSED_TO_HUMAN",
        evidenceHash: "FAIL_CLOSED"
      };
    }
  }
}
```

---

## 5. KẾ HOẠCH BÀN GIAO & TÍCH HỢP HỆ THỐNG

1. **Vị trí tích hợp Job Controller:** Đặt `routeModel()` tại bước tiếp nhận Job ở API `:8000` trước khi tạo lease dispatch sang Harness.
2. **Vị trí tích hợp AgentCore Harness:** Đặt `guardDatabaseCandidate()` tại trước khi gọi `ti-database-adapter` đẩy vào sandbox ECS Fargate (D2).
3. **Lưu vết Evidence Store (S08):** Toàn bộ `evidenceHash` được đính kèm vào metadata của Run Record, đảm bảo tính bất biến theo đúng Law 16.
