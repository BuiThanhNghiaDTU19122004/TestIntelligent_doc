# REVIEW ĐÓNG GAP KIẾN TRÚC TI — v2.2 (28/09/2026)

> **Phạm vi:** `diagram/TI_Master_Architecture_Blueprint.md` (mtime 28/09 10:08:23) + `diagram/TI_Detailed_System_Architecture.md` (mtime 28/09 10:51:57).
> **Nền:** `Research/Feedback/Task_5_ReReview_v2.1_CANDIDATE_Team_TI_Deliverables.md` v1.2 §9 (vừa bổ sung cùng ngày).
> **Chuẩn chấm:** `Research/Task_5_Shift_Left_Verification_Plan_and_Checklist.md` (A–F, Pass/Fail nhị phân, cần 100% Pass) · **Thang nhãn:** `Research/GLOSSARY_TI.md` §A (`OBSERVED` = đã đo kèm ngày/nguồn; `CANDIDATE` = thiết kế đích chưa triển khai; `UNVERIFIED` = chưa nghiệm thu) · **Ba ranh giới:** `CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · `completed ≠ PASS` (Law 18).
> **Kết luận nhanh: ⛔ CHƯA ĐẠT Vòng 2** — Blueprint còn **2 gap P1** (G-01, G-02); Detailed còn **12 gap** (G-03 → G-14, trong đó 3 P0 + 6 P1 + 3 P2). Thứ tự fill đề xuất: **G-03 → G-08 → G-09 → G-12 → G-13 → G-14 → G-01/G-02 → G-04/G-05/G-06/G-07/G-10/G-11**.

---

## 1. BẢNG GAP CÒN LẠI (COPY–PASTE CHO OWNER)

| ID | Mức | File : dòng | Hiện trạng lệch | Cách fill (ghi đúng câu này) | Owner |
| :--- | :--: | :--- | :--- | :--- | :--- |
| G-01 | 🟠 P1 | Blueprint §2.3 L127 | Ghi "S03 semantic, S04 threat modeling" là **AI suy luận**, trong khi sequence L350 chốt S03/S04 chạy tại **JC (Account A, deterministic)** + Detailed L100–L102 cũng ghi Account A | Sửa L127 thành: *"S03/S04 là deterministic chạy tại Job Controller (Account A); Harness chỉ trả gợi ý ImpactSet"* (khớp đề xuất DEF-X-006) | Nghĩa + Hoàng (escalate Tan.Thai nếu giữ ý AI) |
| G-02 | 🟠 P1 | Blueprint §6 TOTAL L508 | Hàng tổng `~$90–$122/tháng / Tiết kiệm ~75%` **không có cột Nhãn** (6 hàng trên đã có `INFERRED`/`CANDIDATE`) | Thêm nhãn `CANDIDATE (tổng hợp từ các hàng trên; hiện trạng đo 22/09 ghim 8a61cf66 vẫn chạy 1 EC2)` | Hoàng |
| G-04 | 🟠 P1 | Detailed L7 + §4 tiêu đề | Claim tuyệt đối *"Bám sát 100% / Đồng bộ 100%"* + vẫn trỏ drawio **cũ** | Viết lại: *"khớp theo bản v2.1 CANDIDATE ngày 28/09/2026; các điểm lệch liệt kê ở §…"* + trỏ sang file v2.1 (DEF-S3-M05/F02) | Trang |
| G-05 | 🟠 P1 | Detailed L35 + L77 | *"WAF … HMAC Verify / kiểm tra chữ ký HMAC"* — WAF managed rules **không verify HMAC** | Sửa cả 2 chỗ: *"WAF (OWASP rules + rate limit); xác thực HMAC tại TI API v2 / Lambda@Edge"* (DEF-S3-M14) | Trang |
| G-06 | 🟠 P1 | Detailed L48, L137, L141, L197 | Cụm `--network none` (thuật ngữ Docker, sai nền Fargate) | Thay toàn file bằng câu chuẩn Blueprint L421: *"Private Subnet không IGW/NAT + SG DENY ALL EGRESS + VPC Endpoints (ECR/Logs/STS ~$22/mo, S3 Gateway $0)"* (DEF-X-004/M-07) | Trang |
| G-07 | 🟠 P1 | Detailed §1 L42 ↔ §3 | §1 đặt *"3 Interface Endpoints (ECR, Logs, STS)"* ở **Account A**, §3 lại đặt ở **Sandbox** | Sửa §1: VPCE thuộc **Sandbox VPC** (copy v2.1 L164/L167–L171); Account A chỉ giữ CloudFront/WAF/API/JC/RDS/S3 (DEF-X-005) | Trang |
| G-08 | 🔴 P0 | Detailed L53 | `AWS Secrets Manager & STS Tokens` nằm trong **Account B** (trái Law 13) | Chuyển sang Account A: *"Secrets Manager / Tenant Secrets & STS (Account A — Law 13)"* (copy v2.1 L110; đối chiếu Blueprint §5.2 L479) (DEF-X-007) | Trang |
| G-09 | 🔴 P0 | Detailed L217, L250 + thiếu Waiver/±0.03 | Gộp `COMPLETED + PASS/HOLD/DO_NOT_PASS` một chỗ; **không có** nhánh HOLD→Waiver, `Faithfulness<0.85→HOLD`, `±0.03`, bộ 6 chỉ số GenAI, dòng `completed ≠ PASS` | Copy nguyên hộp Gate Blueprint L367–L372 + `opt HOLD→Waiver` L375–L378; tách `state` ↔ `gate_result`; thêm `completed ≠ PASS (Law 18)` (DEF-X-009) | Trang + Hoàng |
| G-10 | 🟡 P2 | Detailed L83–L85, L187, L236 | Trạng thái khởi tạo `PENDING` ≠ portal (`queued/running/completed/failed`) + Hùng/Blueprint dùng `QUEUED` | Thống nhất **một từ**: `queued` (Checklist C11); sửa L83/L187/L236 + ánh xạ đủ 4 trạng thái (DEF-X-M02/M-04) | Trang |
| G-11 | 🟡 P2 | Detailed L219, L252 (`[12D]`) | Vẽ poll `RDS → CI/CD` trực tiếp (bỏ qua API) | Sửa thành `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` (DEF-S3-M13) | Hoàng (drawio) + Trang (chữ) |
| G-12 | 🟠 P1 | Detailed §5 (L231–L252) | Bảng FinOps **không có cột Nhãn**, số `$0.085/GB`, `<1s`, `2–5s` không ghi nguồn | Thêm cột `Nhãn` (mặc định `CANDIDATE`); đánh dấu SLA là *mục tiêu thiết kế*; copy mẫu Blueprint §6 L500–L508 (DEF-X-010/A14/F01/D14) | Trang |
| G-13 | 🟠 P1 | Detailed (toàn file, 0 hit) | Thiếu 6 khối: AgentCore Gateway/Policy, Memory `UNVERIFIED`, ECR pre-baked + digest + đường `GitHub Actions→ECR→SSM` nét đứt, `ti-test-packs/` + git tag, cảnh báo DoS/AWS-ban + xin phép DevOps, Aurora `PENDING Team Data`, footnote gap scan SAST + prompt-injection, Runtime 18 receipt | Copy nguyên Blueprint §4.5 L435–L437 (DoS), §4.6 L440–L447 (ti-test-packs/pre-baked/Runtime 18), §4.2 L397 (Team Data), §4.3 L414–L416 (gap scan) + node Gateway (Blueprint L194–L195) + nhãn `UNVERIFIED` (Blueprint L196) | Trang |
| G-14 | 🟡 P2 | Detailed L49–L50, L106–L108, L242 | Tên model *"Claude 5.0 Sonnet"* ≠ Glossary/Task 3 (*"Claude Sonnet 5"*); thiếu Model ID Bedrock + region | Thống nhất `Claude Sonnet 5` (`anthropic.claude-sonnet-5`, `us-east-1`) + `Claude Opus 5` khi `Risk==CRITICAL`; ghi tiering là `CANDIDATE` (Task 3 tự khai chưa triển khai) (A05) | Trang + Nghĩa |

---

## 2. ĐIỂM ĐÃ ĐÓNG (KHÔNG CẦN SỬA LẠI — GHI NHẬN ĐỂ KHỎI REGRESSION)

1. **Blueprint DEF-X-001:** L406 footnote CodeGuru EOL đúng chuẩn (pre-scan = Semgrep ruleset rút gọn + Gitleaks). Lỗi CodeGuru còn duy nhất ở `Hung/TI_Workflow_Hungdz.md` L28 (ngoài phạm vi 2 file nhưng chặn Vòng 2).
2. **Blueprint M-02:** L128 ↔ L507 đã khớp Dual-Model Sonnet 5/Opus 5 (bỏ Haiku). Còn lệch liên-file với `images/*` node `m_haiku` → Hoàng xóa/sync.
3. **Blueprint M-03:** đủ 6 runner (L212 `T_API`, L302 `W_API`); Detailed đủ 6 trục (L50–L56/L143–L160).
4. **Blueprint M-05:** TTL 5 phút + heartbeat 60s ghi rõ là 2 tham số (L354/L357) — mẫu để Detailed/Hùng copy.
5. **Blueprint M-16/E10:** MEM `[UNVERIFIED]` (L196) + cạnh `Review GOLDEN (S10 - UNVERIFIED)` (L259).
6. **Blueprint D03/D04/D08/B10/B11:** `ti-test-packs/` (L441), pre-baked ECR (L443–L445), Runtime 18 receipt (L447), cảnh báo DoS + phê duyệt DevOps (L435–L437), Aurora `PENDING Team Data` (L397), footnote gap SAST/prompt-injection (L414–L416).
7. **Detailed (bản 10:51):** S03/S04 tại JC Account A (L100–L102, khớp Blueprint L350), S09 có Faithfulness (L129), công thức `Target = ImpactSet ∩ TargetBinding` (L120), Direct-to-S3 + Envelope 2KB (L124–L125).
8. **Cả 2 file:** mạng Sandbox đúng (Blueprint L123/L206/L421); D5b DAST có điều kiện Staging URL sống (Blueprint L407–L409; Detailed L160).

---

## 3. 4 CÂU HỎI PHẢI CHỐT TRƯỚC VÒNG 2 (GIỮ NGUYÊN TỪ PHIẾU v1.1 §7)

1. **S03/S04 chạy ở Account A hay Account B?** Ai là system of record của `ImpactSet`/`RiskTier`? (Đề xuất: deterministic → JC Account A; Harness chỉ trả gợi ý.)
2. **Sau khi CodeGuru bị loại, tool nào chạy pre-scan S02→S04, chạy mấy lần** so với Trục 1 ở S07? (Đề xuất: cùng image D5a, ruleset rút gọn, **đúng 1 lần** + cập nhật Task 4 §3.1.1.)
3. **Lease/heartbeat:** `TTL 5 phút + heartbeat 60s` (mẫu Blueprint L357) hay con số khác?
4. **Trạng thái chuẩn khi job vừa nhận:** `queued` hay `pending` — và `gate_result` tách khỏi `state` như thế nào? (Checklist C11 + Law 18.)

---

## 4. CHECKLIST NGHIỆM THU FILE NÀY (100% PASS MỚI MỞ VÒNG 2)

- [ ] G-03: không còn chuỗi `file:///c:/Users/T14S` trong `diagram/**` (`Select-String` = 0 hit).
- [ ] G-05/G-06/G-08: không còn `HMAC Verify` ở WAF, `--network none`, Secrets ở Account B.
- [ ] G-10: chỉ còn đúng 1 từ trạng thái khởi tạo (`queued`), ánh xạ đủ 4 trạng thái portal.
- [ ] G-09: Detailed có hộp 3 nhánh PASS/HOLD+Waiver/DO_NOT_PASS + `±0.03` + 6 chỉ số + `completed ≠ PASS`.
- [ ] G-12: mọi con số FinOps có cột Nhãn (mặc định `CANDIDATE`).
- [ ] G-13: đủ 6 khối bổ sung (Gateway, UNVERIFIED, ECR pre-baked, ti-test-packs, DoS/DevOps, Team Data, gap scan, Runtime 18).
- [ ] G-01/G-02: Blueprint sửa L127 (S03/S04) + hàng TOTAL có nhãn.
- [ ] Drawio cũ `TI_System_Architecture.drawio` gắn nhãn `SUPERSEDED` + mọi link Detailed trỏ sang v2.1.

> **Ghi chú trung thực (Task_5 §9):** File này chỉ chấm *bản vẽ + tài liệu có đáng tin làm căn cứ thiết kế hay chưa*. **Không** đồng nghĩa hệ thống đã kiểm thử/nghiệm thu — cổng **#97 vẫn mở**, `runtime_binding` vẫn `UNVERIFIED`. Đúng Law 18: *bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy.*

**CHANGELOG**

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 28/09/2026 | v2.2 | Cline (AI reviewer) | Review đóng gap sau Blueprint 10:08 + Detailed 10:51: 2 gap Blueprint (G-01/G-02) + 12 gap Detailed (G-03→G-14); 8 điểm đã đóng; 4 câu hỏi P0/P1; checklist 8 dòng nghiệm thu Vòng 2 |
