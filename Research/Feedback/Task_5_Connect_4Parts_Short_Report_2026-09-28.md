# BÁO CÁO NGẮN: NỐI 4 PHẦN — 3 TASK DIAGRAM + SHIFT-LEFT (28/09/2026)

> **Ảnh chuẩn:** `diagram/TI_System_Architecture-Architecture_V2.drawio.png` (System Architecture v2.0)
> **3 task diagram:** S1-Hoàng (kiến trúc tổng thể) · S2-Hùng (workflow Phase 1–4) · S3-Trang/Blueprint (chi tiết + hợp đồng) + **Shift-left** (Task 5, Tester #4).
> **Trạng thái ảnh:** `CANDIDATE` — thiết kế đích, chưa phải hệ đang chạy (hiện trạng OBSERVED 22/09: 1 EC2 API :8000 + Portal :8001, evidence trên EBS, ghim `8a61cf66`, cổng #97 mở).

## 0. QUYẾT ĐỊNH CHỐT (ADOPTED — không mở lại tranh luận S03/S04 ở acc nào)

**Luồng chốt theo ảnh V2, áp cho mọi tài liệu:**

```
[06] ECS Trục 1 (D5a: Semgrep + Trivy + Gitleaks) chạy TRƯỚC trong Sandbox
  → [07A] SARIF → S3  +  [07B] SecurityFindings (SAST/CVE/Secrets) → Job Controller (A)
  → [08A] Context + SecurityFindings (Cross-Account IAM, KHÔNG secret) → Account B
  → [08B] Bedrock AI Review: Sonnet (S03 Impact + S05 Plan + S06 Gen), Opus 5 chỉ khi CRITICAL (S04 Threat)
  → [09A/B/C] Evaluations (Groundedness ≥0.80, Faithfulness ≥0.85, temp=0.0) → Verified ImpactSet
  → [10] Smart Dispatch: Verified ImpactSet ∩ TargetBinding → ECS RunTask Trục 2–6
```

- Trục 1 là **máy quét pattern-matching**: chỉ trả `SecurityFindings` thô, không suy diễn ImpactSet nghiệp vụ (Detailed L101–102 đã ghi đúng).
- Bedrock Account B là **bộ não suy luận**: nhận Context + SecurityFindings rồi mới sinh `ImpactSet`/`RiskTier`/TestPlan/Cases (Detailed L107–109, ảnh `[08B]`).
- Job Controller Account A giữ **workflow authority** (Law 4.3): allowlist ToolIntent → TenantBinding Resolver → STS ngắn hạn → dispatch (ảnh `TenantBinding Resolver`, Blueprint §5.2).
- Pre-scan D5a chạy **đúng 1 lần** (ruleset rút gọn, chung image D5a); S07 runner full-scale là chuyện Wave triển khai, không phải chạy lại 2 lần (đóng DEF-S3-011 theo hướng a2).
## 1. SHIFT-LEFT ĐÃ GIẢI QUYẾT ĐƯỢC GÌ (đối chiếu ảnh V2)

| # | Vấn đề trước đây | Ảnh V2 + 4 tài liệu nay trả lời ra sao |
|---|---|---|
| 1 | Chạy cả 6 runner dù PR chỉ chạm 1 domain | `[10] Target = Verified ImpactSet ∩ TargetBinding`; ngoài ImpactSet → `SKIPPED`; DAST Wave 3 nét đứt, chỉ khi Staging URL sống |
| 2 | Evidence nghẽn EC2/EBS | `[07A]/[09B]/[11A] Direct-to-S3` (Gateway $0) + WORM 90 ngày + SHA-256; Runner chỉ trả envelope ~2KB `[11B]` |
| 3 | AI tự phán PASS / ảo giác | Evaluations `[09A–C]` (temp 0.0, Groundedness 0.80, Faithfulness 0.85) + Gate S09 deterministic + `completed ≠ PASS` (Law 18) |
| 4 | Secret/URL lọt vào prompt AI | `[08A]` chỉ mang Context + SecurityFindings qua STS AssumeRole; URL/secret do TenantBinding Resolver nội suy server-side (Law 13) |
| 5 | Sandbox mở Internet | `DENY ALL EGRESS`, Private Subnet không IGW/NAT, chỉ ra qua VPCE (ECR/Logs/STS ~$22/mo, S3 Gateway $0) |
| 6 | Trộn `state` với `gate_result` | `[12B] Update state` tách khỏi Gate PASS/HOLD/DO_NOT_PASS; HOLD → Waiver QA Lead (Task 4 §6.2) |
| 7 | Không biết file nào để tra | v2.1 CANDIDATE (bản đồ `[01]→[12D]`) · Blueprint (ToolIntent/TenantBinding/FinOps) · Detailed (spec từng bước) · Hung workflow (Phase 1–4) |
## 2. CÒN PHẢI CHỐT / CẢI THIỆN (không chạm S03/S04-acc nữa)

| # | Việc | Chốt 1 câu | Owner |
|---|---|---|---|
| C-1 | `queued` hay `pending` | Dùng **`queued`** duy nhất (Checklist C11); `gate_result` là cột riêng, không gộp vào `state` | Trang (Detailed L83/L187/L236) |
| C-2 | Pre-scan mấy lần | **Đúng 1 lần** (image D5a ruleset rút gọn); cập nhật Task 4 §3.1.1 cho khớp | Hùng + Hoàng |
| C-3 | Eval fail → retry mấy lần → HOLD khi nào | Thêm nhánh fail: retry ≤ N (log S3) → quá N thì `HOLD`; bổ sung `Faithfulness<0.85→HOLD`, `±0.03`, bộ 6 chỉ số (Task 4 §5.3) | Nghĩa + Hùng |
| C-4 | Poll result vẽ tắt | Sửa thành `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` | Hoàng (drawio) + Trang (chữ) |
| C-5 | FinOps thiếu nhãn + SLA treo | Thêm cột `Nhãn` (mặc định `CANDIDATE`); SLA là *mục tiêu thiết kế*; TOTAL Blueprint ghi `CANDIDATE (ghim 8a61cf66 vẫn 1 EC2)` | Hoàng (Blueprint L508) + Trang (Detailed §5) |
| C-6 | Thuật ngữ mạng + HMAC + Secrets + link máy cá nhân | Câu chuẩn Blueprint L421 (Private Subnet + SG Deny All + VPCE); HMAC verify tại API/Lambda@Edge; Secrets về Account A (Law 13); link tương đối sang v2.1 | Trang |
| C-7 | Thiếu khối vận hành trên Detailed | Copy Blueprint §4.5/§4.6 + footnote gap: Gateway/Policy, Memory `UNVERIFIED`, ECR pre-baked + digest + đường Actions→ECR→SSM nét đứt, `ti-test-packs/` + git tag, DoS/AWS-ban + xin phép DevOps, Aurora `PENDING Team Data`, gap scan SAST, Runtime 18 receipt | Trang |
| C-8 | Tên model + tiering là CANDIDATE | Thống nhất `Claude Sonnet 5` (`anthropic.claude-sonnet-5`, `us-east-1`); tiering ghi `CANDIDATE` | Nghĩa + Trang |
## 3. ĐỌC NHANH ẢNH V2 (ECS trả report security → Bedrock mới quyết)

1. `[01–05]`: CI POST artifact + SHA-256 → CloudFront → WAF → TI API v2 trả `202 Accepted <1s` → RDS ghi `queued`.
2. `[06]`: Job Controller gọi ECS RunTask Trục 1 (D5a) trong Sandbox DENY ALL EGRESS.
3. `[07A/B]`: Trục 1 đẩy SARIF thẳng S3, trả `SecurityFindings` gọn về Job Controller.
4. `[08A/B]`: Job Controller gửi Context + SecurityFindings sang Account B; Sonnet review Blast Radius sinh ImpactSet + TestPlan/Cases; Opus 5 chỉ khi CRITICAL.
5. `[09A/B/C]`: Evaluations chấm độc lập → Quality OK + Verified ImpactSet về Account A.
6. `[10]`: Smart Dispatch `Verified ImpactSet ∩ TargetBinding` → chỉ tạo Trục 2–6 bị ảnh hưởng.
7. `[11A/B]`: Runner đẩy raw → S3 WORM, trả envelope 2KB + SHA-256.
8. `[12A/B/C/D]`: Gate S09 deterministic → RDS `COMPLETED + gate_result` → GOLDEN PASS → Memory (UNVERIFIED) → CI poll qua API.

> Ghi chú trung thực (Task_5 §9): đạt checklist = *bản vẽ đáng tin làm căn cứ*. Không đồng nghĩa đã nghiệm thu — #97 mở, `runtime_binding` UNVERIFIED (Law 18).
| C-9 | Khai tử drawio cũ | Gắn nhãn `SUPERSEDED — dùng v2.1` cho `TI_System_Architecture.drawio` + sửa mọi link Detailed sang v2.1 | Hoàng + Trang |
