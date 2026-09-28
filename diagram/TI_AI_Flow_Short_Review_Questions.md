# TI — FLOW NGẮN CỦA PHẦN AI (Account B) + CÂU HỎI REVIEW TƯƠNG ỨNG

> **Mục đích:** cho bạn một **flow 1 trang** của phần AI trong TI, mỗi mắt xích gắn **nút mờ `❓`**; từ đúng nút mờ đó ⇒ tra ra **câu hỏi cần hỏi team AI** (§3) — hỏi có trọng tâm, không hỏi lan man.
> **Nguồn:** `diagram/TI_System_Architecture_v2.1_CANDIDATE.drawio` · `Research/Task_3_AI-Prompt_Research.md` §1.1 (hiện trạng AI tự khai) · `Research/Task_4_Test_Plan_Strategy_Evaluation.md` §5.3/§6.
> **Ngày:** 28/09/2026 · Ngắn gọn có chủ đích: chỉ bàn **phần AI**, không lặp lại hạ tầng.

---

## 1. FLOW NGẮN (đường đi của AI trong 1 lượt thẩm định)

```
 [VÀO] Changeset (git diff · OpenAPI · SQL migration · UI bundle)
       +  [07B] ImpactSet / RiskTier  +  SARIF (Semgrep/Trivy/Gitleaks)
                    │
                    │ [08A] Cross-Account IAM: context + Token Budget
                    │       (TUYỆT ĐỐI không mang secret/URL thật)
                    ▼
 ┌──────────────────────── ACCOUNT B (us-east-1) ────────────────────────┐
 │ AgentCore Harness (TIJobRunner)                          ❓1 runtime nào? │
 │   S03 Impact Engine      (ai chạy: A hay B?)              ❓2           │
 │   S04 Risk Tiering       (ai chạy: A hay B?)              ❓2           │
 │        │                                                                │
 │        ▼   S05 Planning          ← model nào?              ❓3 tiering?  │
 │            S06 Candidate Gen     ← Opus 5 khi nào?         ❓4 điều kiện │
 │        │        │ TestPlan + candidate testcases                        │
 │        │        ▼                                                       │
 │        │   Bedrock Evaluations ── Groundedness ≥0.80                   │
 │        │                          Faithfulness ≥0.85      ❓5 ai đo? tên │
 │        │        ├── ĐẠT ──▶ [09B] lưu candidate → S3 (Evidence)         │
 │        │        └── KHÔNG ĐẠT ──▶ ??? (retry? HOLD?)      ❓6 chưa vẽ   │
 │        │                                                                │
 │        └─ phát ToolIntent JSON (chỉ symbolic IDs) ── ai validate?  ❓7  │
 │                                     │ [09C] Quality OK                  │
 └─────────────────────────────────────┼───────────────────────────────────┘
                                       ▼
        Job Controller (Account A): allowlist → TenantBinding → STS → ECS RunTask
                                       │
                        6 trục sandbox chạy → [11A] raw → S3
                                       │      [11B] envelope ~2KB
                                       ▼
        S09 Gate (mã cứng): PASS │ HOLD │ DO_NOT_PASS
                                       │ PASS
                                       ▼
        [12C] tri thức GOLDEN → AgentCore Memory (S10)      ❓8 reuse chưa nghiệm thu
```

**Đọc 1 câu:** AI nhận *ngữ cảnh + ngân sách token* (không có secret) → suy luận S03–S06 → tự chấm bằng Evaluations → **chỉ phát "ý định" (ToolIntent)** rồi **dừng**; muốn chạy gì phải để Job Controller dispatch; kết quả PASS mới được ghi vào Memory để lần sau học lại.

---

## 2. BẢNG 8 BƯỚC — AI NHẬN GÌ / TRẢ GÌ / AI KIỂM

| Bước | Nhận vào | AI/công cụ làm gì | Trả ra (artifact) | Ai kiểm chứng | Trạng thái thật |
| :--: | :--- | :--- | :--- | :--- | :--- |
| **S03** Impact | Changeset + dependency graph | Xác định miền bị ảnh hưởng | `ImpactSet` | Job Controller (system of record) | ❓ tranh chấp A/B |
| **S04** Risk | `ImpactSet` + SARIF/CVE | Gán `Risk Tier` LOW/MED/HIGH/CRITICAL | `RiskTier` (quyết định có gọi Opus 5 hay không) | Job Controller | ❓ tranh chấp A/B |
| **S05** Planning | `RiskTier` + context + `Evaluation Pack` | Lập kế hoạch kiểm thử (chạy domain nào, tiêu chí gì) | `TestPlan` | Evaluations (bước sau) | CANDIDATE (model tiering chưa có) |
| **S06** Candidate | `TestPlan` | Sinh candidate testcase/schema (boundary, null, stateful flow…) | bộ `candidate testcases` | **Bedrock Evaluations** + Gate sau này | CANDIDATE |
| **Eval** | TestPlan + candidates | Chấm `Groundedness` / `Faithfulness` (+ bộ 6 chỉ số GenAI) | điểm số + quyết định đạt/không | Ngưỡng do Evaluation Pack | ❓ chỉ nêu 2/6 chỉ số |
| **Hand-off** | candidates đạt | **Không chạy gì cả** — phát `ToolIntent JSON` (symbolic) | `ToolIntent` (không URL/secret/SQL tự do) | Job Controller allowlist (Law 15) | ❓ validator ở đâu |
| **Exec** | `ToolIntent` | (Job Controller mới là bên chạy thật) | 6 trục runner → raw → S3 | S09 Gate | CANDIDATE |
| **S10** Learn | Gate = `PASS` | Ghi tri thức "GOLDEN" vào Memory để lượt sau dùng lại | vector/memory record | Người duyệt GOLDEN (?) | `UNVERIFIED` — chưa nghiệm thu reuse |

---

## 3. TỪ MỖI NÚT MỜ ❓ ⇒ CÂU HỎI CẦN HỎI (ánh xạ 1–1)

| Nút | Nút mờ ở đâu | Câu hỏi để hỏi team AI | Câu trả lời "đạt" phải có |
| :--: | :--- | :--- | :--- |
| ❓1 | **Runtime Harness** | Task 3 ghi *"Harness (runtime **v5**)"*, còn receipt/checklist ghi *"AgentCore **Runtime 18**"*. **Bản đang chạy là runtime nào**, Harness version mấy? | Số runtime + version + receipt/ngày |
| ❓2 | **S03/S04 ở đâu** | S03–S04 chạy ở **Account A (JC)** hay **Account B (Harness)**? Ai là *system of record* — và Harness trả về là **gợi ý** hay **quyết định**? | **Một câu chốt** + danh sách file/sơ đồ phải sửa |
| ❓3 | **Model tiering** | Hình vẽ *"Sonnet 5 mặc định"* — nhưng Task 3 tự khai *"**chưa triển khai** multi-model/model tiering động"*. Vậy tiering **đã chạy chưa hay còn CANDIDATE**? | Có/không + kế hoạch Wave nào |
| ❓4 | **Điều kiện gọi Opus 5** | Sơ đồ ghi `Risk == CRITICAL` → gọi Opus. Vậy `Risk == CRITICAL` được tính **ở bước nào, bằng công thức gì**, và ai là người đặt điều kiện đó? | Ngưỡng + công thức + owner |
| ❓5 | **Ai chấm eval** | `Faithfulness ≥ 0.85` đo bằng gì — Bedrock Evaluations thật hay LLM-as-judge tự viết? Module tên `TIRunnerGroundness` có đúng không? Bộ **6 chỉ số GenAI** chốt là gì? | Tên module thật + công thức Faithfulness + bảng 6 chỉ số |
| ❓6 | **Eval fail thì sao** | `Groundedness < 0.80`: re-generate **mấy lần**, ghi log ở đâu, sau bao lần thì **HOLD**? | Policy retry bằng văn bản (không có vòng thử lại ngầm) |
| ❓7 | **Ai validate ToolIntent** | `ToolIntent` được validate ở **Harness / AgentCore Gateway / Job Controller**? Schema version mấy, log validate ở đâu? | URL schema + version + nơi log |
| ❓8 | **Memory reuse** | Tiêu chí nào là `GOLDEN`, **ai duyệt**, lượt sau AI **có đọc lại** không, chống "ô nhiễm tri thức" bằng gì? | Tiêu chí + người duyệt + cơ chế reuse + nhãn `UNVERIFIED` trên sơ đồ |

---

## 4. NẾU CHỈ HỎI 4 CÂU (P0 — hỏi theo thứ tự này)

1. **❓2 — S03/S04 ở account nào?** *(chặn toàn bộ ma trận Changeset→Runner; 2 hồ sơ nói A, 3 hồ sơ nói B)*
2. **❓1+❓3 — Runtime đang pin model nào, tiering chạy chưa?** *(hình vẽ Sonnet-mặc-định nhưng hiện trạng khai "pin Opus 5, chưa có tiering")*
3. **Haiku 4.5 bỏ hay giữ?** *(Blueprint sửa hôm nay ghi "loại bỏ Haiku" nhưng bảng FinOps cùng file + `images/*` + Task 3 vẫn giữ — xem chi tiết Nhóm B trong tài liệu dài)*
4. **❓6 — Eval fail → retry mấy lần → HOLD khi nào?** *(Checklist cấm vòng thử lại ngầm; nếu AI tự retry vô hạn thì không bao giờ xong)*

## 5. "HIỆN TRẠNG AI TỰ KHAI" — điểm neo để bạn đối chiếu mọi câu trả lời

> Trích `Research/Task_3_AI-Prompt_Research.md` §1.1 — đây là **hiện trạng**, mọi thứ khác là đề xuất:

- Runtime Harness **v5**, Bedrock model profile, AgentCore Gateway (MCP/IAM), AgentCore Memory **ACTIVE nhưng chưa nghiệm thu reuse**
- Harness điều phối **8 inline tools**: `change · impact · risk · plan · generate · production · evaluate · assemble_result`
- Chỉ **2 tool adapter** đã có: `http-request` (API qua Lambda), `ti-playwright` (UI qua MCP/Browser CDP) · **chưa có** adapter cho DB / Performance / Security
- Runtime **kết nối cố định `us.anthropic.claude-opus-5`** — *chưa có tiering động*
- Input: `text/plain, text/markdown, application/json`; giới hạn **256 KiB/artifact, 1 MiB tổng**

## CHANGELOG

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 28/09/2026 | v1.0 | Cline | Flow ngắn 1 trang của phần AI (Account B): sơ đồ chữ + bảng 8 bước vào/ra + ánh xạ 8 nút mờ ❓→ câu hỏi + 4 câu P0 + hiện trạng AI tự khai |

