# TASK 5 — KẾ HOẠCH XÁC MINH & CHECKLIST SHIFT-LEFT CHO 3 SƠ ĐỒ

**Dự án:** Testing Intelligence (TI)
**Tác giả:** Nghĩa — Tester #4 (phụ trách Shift-left testing, theo Biên bản họp TI 23/09/2026, Mục 5)
**Ngày tạo:** 25/09/2026 · **Cập nhật:** 28/09/2026 (v1.1 — điền kết quả Vòng 1)
**Đối tượng xác minh:** 3 sơ đồ sẽ được vẽ sau họp 23/09/2026 — của Hoàng (kiến trúc tổng thể), Hùng (workflow), Trang (sơ đồ kiến trúc chi tiết gắn service/tool)
**Trạng thái:** ACTIVE — Vòng 1 đã điền sẵn (28/09/2026), **chờ Nghĩa xác nhận & ký**; Vòng 2 chưa mở
**Phương pháp:** ADR-0001 · **Nguồn sự thật:** ADR-0002 · **Thang Pass/Fail:** ADR-0003 · **Cấu trúc tài liệu:** ADR-0004

> *"Shift-left testing"* ở đây nghĩa là: **xác minh sơ đồ NGAY TẠI KHÂU VẼ (draft)** — trước khi sơ đồ được dùng để thiết kế test case, trước khi triển khai, trước khi họp nghiệm thu. Phát hiện lỗi trên sơ đồ rẻ hơn 10–100× so với để sai sót đi vào hạ tầng thật (nguyên tắc Shift Left — CTFL, Task 4 §2.1).

---

## 0. Tóm tắt điều hành

| Hạng mục | Nội dung |
| :--- | :--- |
| **Mục tiêu** | 3 sơ đồ phải **đúng với hiện trạng đo được**, **đúng biên bản 23/09**, **đúng tài liệu Task 1–4**, và **nhất quán với nhau** — trước khi bất kỳ ai dùng chúng làm căn cứ |
| **Vòng 1 — Review tĩnh** | Đọc draft sơ đồ, chấm **Pass/Fail từng mục Checklist A–D**, góp ý trong SLA 24 giờ |
| **Vòng 2 — Đối chiếu evidence** | Từng claim của sơ đồ được đối chiếu với nguồn thật (portal live `/diagrams`, `/evidence`, receipt, API, Research docs) — **Checklist E–F** |
| **Thang** | Pass/Fail **nhị phân từng mục**; tổng thể phải **100% Pass** mới đóng dấu nghiệm thu (ADR-0003) |
| **Quyền chốt** | Nghĩa (Tester #4) chốt Pass/Fail cuối cùng sau vòng re-review |
| **Sản phẩm đầu ra** | Phiếu đánh giá mỗi sơ đồ + Defect Log + dấu nghiệm thu (xem §8–§9) |

**Ba ranh giới không được xóa** (kế thừa tiêu chí nghiệm thu hiện có của portal):
`CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · `completed ≠ PASS` (Law 18).

---

## 0A. KẾT QUẢ ĐIỀN VÒNG 1 — 28/09/2026 *(bản điền sẵn, chờ Nghĩa xác nhận & ký)*

**Ký hiệu:** ✅ Pass · ❌🔴 Fail **Blocking** · ❌🟡 Fail **Minor** · ⏳ chờ Vòng 2 (cần gọi portal live — chưa tính Pass/Fail, nhưng **chặn ký**) · ➖ N/A có lý do (**chỉ hợp lệ khi Nghĩa xác nhận bằng chữ**, ADR-0003 §5).

| Sơ đồ | Tổng mục | ✅ | ❌🔴 | ❌🟡 | ⏳ | ➖ | Kết luận |
| :--- | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| **S1 — Hoàng (Blueprint)** | 50 | 35 | 5 | 7 | 3 | 0 | ⛔ CHƯA ĐẠT |
| **S2 — Hùng (workflow md + Final_workflow.png)** | 50 | 23 | 4 | 17 | 4 | 2 | ⛔ CHƯA ĐẠT |
| **S3 — Trang (Detailed md + drawio)** | 50 | 21 | 11 | 16 | 2 | 0 | ⛔ CHƯA ĐẠT |

> Mỗi sơ đồ phải qua **50 mục** = A(15) + nhóm riêng B/C/D(15) + E(10) + F(10). *(§10 bản v1.0 ghi "60 mục" là sai số học — đã sửa.)*

**Phạm vi chấm trong lượt này — đọc trước khi tin số liệu:**

| Sơ đồ | Hồ sơ đã chấm (có trong lô upload) | Hồ sơ **chưa** có trong lô → chưa chấm được |
| :--- | :--- | :--- |
| **S1** | `TI_Master_Architecture_Blueprint.md` (v1.0, 529 dòng) | Ảnh chuẩn `TI_System_Architecture-Architecture_V2.drawio.png`, `images/TI_Master_Architecture.drawio` (4 tab) |
| **S2** | `TI_Workflow_Hungdz.md` + `Final_workflow.png` (sequence 38 bước) | `Workflow_Hung.pdf` (md tự ghi "chưa export lại") |
| **S3** | `TI_Detailed_System_Architecture.md` + `TI_System_Architecture.drawio` (**bản cũ — chưa gắn nhãn SUPERSEDED**) | `TI_System_Architecture_v2.1_CANDIDATE.drawio(.png)` |

→ Nếu V2 PNG / v2.1 drawio đã sửa các mục ❌ ở S1/S3, gửi file để chấm lại; các mục chỉ dựa vào hình (A12, A13, B09…) có thể đổi.

**5 điều quan trọng nhất từ lượt điền này:**

1. **S03/S04 vẫn là mâu thuẫn P0 chưa đóng — và phiếu v2.2 đã đọc sai Detailed.** Detailed L63/L107/L318 đặt S03/S04 ở **Account B (Bedrock sinh ImpactSet)**, khớp Connect Report §0 và ảnh V2; nhưng Blueprint L350 và Hùng ghi **Job Controller (Account A) deterministic**. Gap Closure v2.2 (điểm đã đóng #7, G-01) ghi Detailed L100–102 là "S03/S04 tại JC" — trong bản Detailed của lô upload, dòng đó chỉ nói Trục 1 *không* suy diễn ImpactSet (trừ khi bản trong repo khác bản upload — cần chạy lại `Select-String`). Cần **chốt 1 câu** (đề xuất ở §7A.3, Q1).
2. **Hùng dispatch D5a 2 lần và không có `ImpactSet ∩ TargetBinding`** (PNG #7 & #21; md L32 & L48) — vừa vi phạm "pre-scan đúng 1 lần" đã ADOPTED, vừa là lỗi "PR sửa API → chạy cả 6" (P0 Đối soát §3.6).
3. **Blueprint không có công thức `ImpactSet ∩ TargetBinding` và không có NoSQL** (0 hit `DynamoDB`) — hai mục vòng 1 từng chấm ✅ (B06, B09) nay chấm ❌ vì bản Blueprint trong lô không có. Nếu bản repo có, gửi lại.
4. **Detailed gần như chưa nhận bản sửa nào của phiếu Vòng 1**: còn `--network none` (L49/138/142), HMAC ở WAF (L35/78), Secrets ở Account B (L56), `PENDING` (L86), `Claude 5.0 Sonnet`, 5 link `file:///c:/Users/T14S`, cột FinOps không nhãn, "Bám sát 100%" (L7), COMPLETED gộp gate (L130/218/251).
5. **Số liệu lạ mới phát hiện:** Blueprint L428 tính "3 interface endpoint × $7.3 ≈ $22" nhưng chính L425–427 liệt kê **4** interface endpoint (`ecr.api`, `ecr.dkr`, `logs`, `sts`) — Detailed L42/L142 lặp lại "3". Cần tính lại theo đơn giá thật và gắn nhãn `CANDIDATE`.

*Blueprint (S1) đã đóng được khá nhiều: CodeGuru→footnote EOL, Dual-Model khớp L128↔L507, đủ 6 runner, heartbeat 60s/TTL 5m, DoS+DevOps, Team Data, `ti-test-packs/`, pre-baked ECR, Runtime 18, Memory `UNVERIFIED`, zone `[CANDIDATE]`. Hùng (S2) đã đóng CodeGuru, Waiver, `queued`, D5b W3, Trục 2, gap prompt-injection.*

---

## 1. Mục tiêu & Phạm vi

### 1.1. Mục tiêu

1. Phát hiện sai lệch của sơ đồ **trước** khi nó trở thành căn cứ cho test plan / kiến trúc triển khai (shift-left).
2. Buộc mọi hình vẽ phải **tách bạch ba trạng thái sự thật**: `OBSERVED` (đo được) · `INFERRED` (suy diễn) · `CANDIDATE` (đề xuất) — kỷ luật nhãn sự thật theo Báo cáo Đối soát §3.9.
3. Đóng các điểm còn mở của Biên bản 23/09 có liên quan trực tiếp tới sơ đồ (Mục 2, 3, 4 của biên bản).

### 1.2. Trong phạm vi (In scope)

- **S1** — Sơ đồ *kiến trúc tổng thể* của Hoàng (chuẩn đối chiếu: Task 2 §8 + Phụ lục B).
- **S2** — Sơ đồ *workflow* của Hùng (chuẩn đối chiếu: Task 4 §3–§6 + luồng chức năng S01–S10 trên portal).
- **S3** — Sơ đồ *kiến trúc chi tiết, gắn service/tool* của Trang (chuẩn đối chiếu: Task 1 + portal nhóm *Hạ tầng*).
- **S4** — Tính nhất quán **giữa** 3 sơ đồ (cross-diagram consistency).
- **S5** — Đối chiếu evidence: mỗi claim rút ra được từ đâu, có đo được không.

### 1.3. Ngoài phạm vi (Out of scope)

- Chạy test thật trên hạ tầng TI (đó là vòng đời job, không phải nghiệm thu sơ đồ).
- Sửa nội dung Task 1–4 — tester chỉ **ghi nhận defect**, việc sửa thuộc về tác giả.
- 11 sơ đồ đang public trên portal `/diagrams` — chúng là **nguồn đối chiếu**, không phải đối tượng nghiệm thu lần này.

### 1.4. Giả định (assumption — chỉnh được)

| # | Giả định | Lý do |
| :--: | :--- | :--- |
| A1 | File này đặt tên `Task_5_…` để không đè `Task_4_Test_Plan_Strategy_Evaluation.md` | Chưa xác nhận được tên file khác |
| A2 | ADR và Glossary để **file riêng** trong `Research\adr\` và `Research\GLOSSARY_TI.md` | Câu hỏi xác nhận vị trí bị timeout |
| A3 | Review **từng sơ đồ ngay khi nhận** (không chờ đủ 3), SLA phản hồi 24 giờ làm việc | Shift-left = sớm nhất có thể |
| A4 | 3 sơ đồ chưa tồn tại ở ngày viết plan → plan này thiết kế cho **lần review draft đầu tiên** | Biên bản 23/09 mới giao việc |

---

## 2. Vai trò & Trách nhiệm (RACI)

| Việc | Hoàng (S1) | Hùng (S2) | Trang (S3) | **Nghĩa (Tester #4)** |
| :--- | :---: | :---: | :---: | :---: |
| Vẽ draft sơ đồ | **R** | **R** | **R** | C |
| Nộp draft + self-check nhanh bằng Checklist A | **R** | **R** | **R** | C |
| Review tĩnh (Vòng 1, Checklist A–D) | C | C | C | **R/A** |
| Đối chiếu evidence (Vòng 2, Checklist E–F) | I | I | I | **R/A** |
| Sửa defect sau góp ý | **R** | **R** | **R** | V |
| Re-review & chốt Pass/Fail cuối | I | I | I | **R/A** |
| Ghi biên bản nghiệm thu sơ đồ | C | C | C | **R** |

> **R** = Thực hiện · **A** = Phê duyệt · **C** = Tham vấn · **I** = Được thông báo · **V** = Xác minh lại

**Lưu ý xung đột lợi ích:** Nghĩa là tác giả Task 3 → khi checklist chạm nội dung Task 3, phải ghi rõ "tự xác minh" và mời Tan.Thai / Product Architect ký xác nhận (theo Đối soát §3.6 — Tan.Thai duyệt mapping Changeset → Runner).

---

## 3. Nguồn sự thật (Ground truth) & Thứ tự ưu tiên

Thứ tự tin cậy giảm dần — khi xung đột thì **nguồn đứng trước thắng** (ADR-0002):

| Ưu tiên | Nguồn | Vị trí / Cách lấy | Dùng để xác minh |
| :--: | :--- | :--- | :--- |
| **1** | **Portal live — trạng thái đang phục vụ** | `https://d1tibdarzmw3jq.cloudfront.net/diagrams` · `/evidence` · `/api` | Commit, image digest, trạng thái nghiệm thu *ngay lúc review* |
| **2** | **Receipt triển khai** | Receipt 14/09 (source `fbdd8dfc…`, image `efabf57dcd8c`, AgentCore Runtime 18); Receipt `0409cb5a` 23/09 (ảnh ECR `sha256:b717be38…` — chỉ đọc lại 4 tuyến công khai) | Claim "đã triển khai / V2" |
| **3** | **Biên bản họp TI 23/09/2026** | `D:\Doc\PDF\Biên bản cuộc họp TI - 23_09_2026.pdf` | Phạm vi 3 sơ đồ, vấn đề mở, phân công |
| **4** | **Research docs Task 1–4 + Báo cáo Đối soát** | `D:\Doc\Research\` — đặc biệt Task 2 §8, Task 4 §6, `BAO_CAO_DOI_SOAT…` (ma trận 10 điểm) | Chuẩn thiết kế, entry/exit criteria, 10 điểm P0–P3 |
| **5** | **Bản in portal 21/09** | `D:\Doc\PDF\Diagram_1..4.pdf` | Lịch sử — **không dùng làm chuẩn hiện tại** |

**Lưu ý then chốt khi đối chiếu** (trích portal, truy cập 25/09/2026):

- Kiến trúc triển khai và chức năng theo **lượt đo 2026-09-22 trên source `b55fed16…`, ảnh `58abe918…`** — ảnh chụp có ngày, **không** phải bản đang phục vụ lúc đọc.
- Cùng ngày 22/09 còn một lượt rollout trước đó (`7dca7c0a` / `253e702e`, CI run `35694235318`) — **một ngày đo không phân biệt được hai lượt**.
- Receipt 14/09 là **mốc lịch sử mở đường v2**, không mô tả bản đang chạy hôm nay.
- **Nét đứt = chưa có lượt đo live** (Gateway/ToolIntent, outbox chờ ACK, Memory reuse); `runtime_binding` = `UNVERIFIED` trên mọi check.
- Triển khai ≠ nghiệm thu; phần còn mở ghi ở `/docs · giới hạn hiện tại`; theo dõi nghiệm thu **#97**.

---

## 4. Phương pháp xác minh — 2 vòng (ADR-0001)

### 4.1. Vòng 1 — REVIEW TĨM (Static Review) · SLA 24 giờ

**Entry Criteria (điều kiện bắt đầu):**

- [ ] Draft sơ đồ nộp lên kho team (định dạng draw.io / PNG / PDF **kèm file nguồn** chỉnh sửa được).
- [ ] Kèm "phiếu tự khai" tối giản: *tên sơ đồ, người vẽ, ngày, nguồn tham khảo, các claim `CANDIDATE` còn mở*.
- [ ] Sơ đồ đã self-check bằng **Checklist A** — nếu self-check chưa xong, tester trả về ngay (không tốn vòng review).

**Thao tác:**

1. Đọc toàn bộ sơ đồ 1 lần **chưa đối chiếu** — bắt lỗi trình bày, thiếu legend, mũi tên vô hướng/nhãn mập mờ.
2. Chấm từng mục **Checklist A** (chung) + **Checklist B/C/D** tương ứng (Hoàng / Hùng / Trang). Mỗi mục = **Pass hoặc Fail**, không có "được một nửa".
3. Mỗi mục **Fail** sinh 1 dòng **Defect Log** (§8.2) kèm: *ID, mô tả, nguồn chứng minh, mức Blocking/Minor, owner*.
4. Trả phiếu đánh giá cho người vẽ + ghi deadline sửa (Blocking = 24h, Minor = 3 ngày làm việc).

### 4.2. Vòng 2 — ĐỐI CHIẾU EVIDENCE · chạy ngay sau (hoặc song song phần OBSERVED)

**Thao tác:**

1. Rút toàn bộ **claim** của sơ đồ thành danh sách (câu khẳng định kiểu "đã đo", "đã triển khai", "chạy trên X", "đạt Y").
2. Với mỗi claim → chạy **Checklist E** (portal / receipt / API) và **Checklist F** (10 điểm nóng của biên bản 23/09).
3. Ghi **Evidence Pack** cho mỗi lần kiểm: `URL + timestamp + kết quả + (screenshot hoặc đoạn trích)`. Không có evidence pack = **Fail** mục đó (không tin lời "tôi đã thấy").
4. Claim nào không tra được → gắn nhãn `UNVERIFIED`, **không được** xuất hiện ở dạng khẳng định trên sơ đồ (phải chuyển sang nét đứt / nhãn `CANDIDATE`).

### 4.3. Re-review (vòng ngắn)

- Sau khi người vẽ sửa → chỉ chấm lại các mục từng Fail + rà nhanh Checklist A để bắt lỗi phát sinh.
- Nghĩa chốt **Pass/Fail cuối** theo §9.

---

## 5. Thang đánh giá & Quy tắc nghiệm thu (ADR-0003)

| Quy tắc | Nội dung |
| :--- | :--- |
| **Đơn vị chấm** | Từng mục checklist = **Pass ✅** hoặc **Fail ❌** — nhị phân, không điểm từng phần |
| **Nghiệm thu tổng** | Sơ đồ đạt khi **tất cả** mục áp dụng = Pass → **100%**. Chỉ 1 Fail = **CHƯA ĐẠT** |
| **Không có "N/A" tự ý** | Mọi mục đều áp dụng; nếu thật sự không liên quan phải ghi lý do + tester chấp nhận bằng chữ (coi như Pass kèm ghi chú) |
| **Rủi ro của ngưỡng 100%** | Ngưỡng chặt → dễ kẹt ở mục lặt vặt. Giảm thiểu: defect tách **Blocking** (sai hiện trạng / sai biên bản) vs **Minor** (trình bày) — *cùng là Fail*, nhưng Blocking phải sửa trước re-review, Minor gộp sửa đợt. Vẫn chỉ đóng **ĐẠT** khi đủ 100% |
| **Trạng thái nghiệm thu** | `CHƯA REVIEW` → `VÒNG 1 ĐÓNG GÓP` → `VÒNG 2 EVIDENCE` → `ĐẠT` / `CHƯA ĐẠT` |
| **Không có evidence** | Không bịa Pass: mục = **Fail**, bắt buộc sơ đồ đổi cách diễn đạt (nét đứt / nhãn `CANDIDATE`) |

---

## 6. Lịch trình & SLA

| Giai đoạn | Thời điểm | Đầu ra |
| :--- | :--- | :--- |
| G0 — Nhận draft | T+0 (ngay khi từng sơ đồ nộp) | Thông báo + phiếu tự khai |
| G1 — Vòng 1 review tĩnh | T+24 giờ làm việc | Phiếu Checklist A–D + Defect Log |
| G2 — Sửa defect | T+24h (Blocking) / T+3 ngày (Minor) | Draft v2 + ghi chú đã sửa |
| G3 — Vòng 2 evidence | T+48h kể từ G1 | Checklist E–F + Evidence Pack |
| G4 — Re-review & chốt | T+72h kể từ G1 | Dấu **ĐẠT / CHƯA ĐẠT** của Nghĩa |
| G5 — Báo cáo | Cùng ngày chốt | 1 dòng biên bản: trạng thái 3 sơ đồ, số mục Pass/Fail, link Defect Log |

---

## 7. CHECKLIST XÁC MINH — **BẢN ĐÃ ĐIỀN (Vòng 1, 28/09/2026)**

> Cách dùng: ✅ Pass · ❌🔴 Blocking · ❌🟡 Minor · ⏳ chờ Vòng 2 · ➖ N/A có lý do. Cột *Ghi chú / căn cứ* trích dòng/bước cụ thể để tra lại. **Đây là bản điền sẵn; Pass/Fail cuối do Nghĩa chốt** (RACI §2).

### CHECKLIST A — Mục chung cho cả 3 sơ đồ

| ID | Nội dung kiểm tra (Pass nếu = Có/Đúng) | Nguồn đối chiếu | S1 Hoàng | S2 Hùng | S3 Trang | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--: | :--: | :--- |
| A01 | Sơ đồ có: tên, **ngày đo/ngày tạo**, người vẽ, nhãn nhóm (kiến trúc / chức năng / hạ tầng / lưu trữ) | Quy ước portal `/diagrams` | ✅ | ❌🟡 | ❌🟡 | S2: PNG không có ngày/người vẽ/nhóm nhãn (md có tác giả, không ngày). S3: Detailed L1–4 có phiên bản+ngày nhưng thiếu người vẽ/nhóm nhãn; drawio cũ không metadata (M01). |
| A02 | Mọi khẳng định gắn nhãn `OBSERVED` / `INFERRED` / `CANDIDATE` — không trộn lẫn | Đối soát §3.9 (kỷ luật nhãn sự thật) | ✅ | ❌🟡 | ❌🔴 | S1: header L13 + zone `[CANDIDATE]` (L166/L185) → DEF-X-002 đã đóng ở Blueprint. S2: md có banner ⚠️ CANDIDATE, PNG không có. S3: 0 chữ CANDIDATE/OBSERVED trong Detailed + drawio; không có ghi chú hiện trạng EC2. |
| A03 | Quy ước **nét liền = đo được trên live**, **nét đứt = khai báo/chưa đo** được giải thích bằng legend ngay trên sơ đồ | Portal — Cách đọc | ❌🟡 | ❌🟡 | ❌🔴 | S1: Mermaid không legend, dùng `-.->` cho cả poll/GOLDEN/báo cáo (không theo quy ước liền = đã đo, đứt = chưa đo). S2: UML — nét đứt = return, không legend. S3: không legend (DEF-X-002). |
| A04 | Không có câu nào tuyên bố **"đã nghiệm thu"** khi #97 còn mở | Portal — Theo dõi nghiệm thu #97 | ✅ | ✅ | ✅ | Không câu nào tuyên bố 'đã nghiệm thu'; S1 L13 ghi rõ `≠ nghiệm thu hệ thống`. |
| A05 | Thuật ngữ dùng đúng `GLOSSARY_TI.md` — **giống hệt** cách dùng trong 2 sơ đồ kia | `D:\Doc\Research\GLOSSARY_TI.md` | ✅ | ✅ | ❌🟡 | S3: 'Claude 5.0 Sonnet' (L51, L107, L243) ≠ Glossary `Claude Sonnet 5` (`anthropic.claude-sonnet-5`, `us-east-1`). |
| A06 | **Không** xuất hiện dịch vụ AWS đã EOL (đặc biệt Amazon CodeGuru Security — ngừng 20/11/2025) | Đối soát §3.1 (P0) | ✅ | ✅ | ✅ | DEF-X-001 ĐÓNG: không còn dùng CodeGuru; S1 L406 chỉ nhắc EOL kèm 'KHÔNG dùng'; Hùng đã chuyển sang Inspector + Semgrep/Trivy/Gitleaks. |
| A07 | Hai tài khoản AWS vẽ đúng ranh giới: backend `ap-southeast-1` ↔ AgentCore Harness/Memory `us-east-1` | Portal — Hạ tầng; Task 2 §8 | ✅ | ❌🟡 | ✅ | S2: chỉ ghi Account A/B, chưa có `ap-southeast-1` / `us-east-1`. |
| A08 | Trạng thái phụ (`UNVERIFIED`, `FROZEN`, `chưa nghiệm thu`, `READY/ACTIVE`) khớp giá trị thật tại ngày review | `/evidence` (ADR-0002 ưu tiên 1) | ✅ | ✅ | ❌🟡 | S3: Memory S10 chỉ ghi 'GOLDEN only', thiếu `UNVERIFIED` (M16). S1 L196/L259, S2 PNG #34 đúng. |
| A09 | Route/URL dẫn trên sơ đồ **tồn tại thật** (`/diagrams`, `/api`, `/evidence`, `/docs`, `/runs`, `/knowledge`) | Gọi thử portal/API | ⏳ | ⏳ | ✅ | Cần gọi live: `POST /v2/operations/{id}/actions` (S1 L376, S2 PNG #37) — EV-1 chỉ xác nhận `/v2/artifact-jobs`. S3 chỉ dẫn `/v2/artifact-jobs*` → khớp EV-1. |
| A10 | Tham chiếu kỹ thuật (commit, image digest, Runtime) khớp portal tại ngày review — hoặc ghi rõ "theo lượt đo <ngày>" | `/evidence`, receipt | ✅ | ✅ | ✅ | S1 L446–447 trích đúng receipt 14/09 (`fbdd8dfc`/`efabf57dcd8c`, Runtime 18) kèm nhãn 'mốc lịch sử'; S2/S3 không claim commit/digest. |
| A11 | **Nhất quán với 2 sơ đồ kia**: cùng tên component, cùng hướng luồng, cùng trạng thái | So sánh chéo 3 draft | ❌🔴 | ❌🔴 | ❌🔴 | **M-01 (P0)**: S03/S04 chạy ở đâu — S1 L127 (AI suy luận) ↔ L350 (Job Controller); S2 = Account A (JC) ↔ S3 L63/L107/L318 = Account B (Bedrock sinh ImpactSet). **DEF-S3-011**: D5a chạy 2 lần (S2 PNG #7 & #21; S3 [10] L120/L203). State: `queued` (S2) ↔ `QUEUED` (S1 L284/L343) ↔ `PENDING` (S3 L86). S1 L64 '3 cấp' ↔ L128 '2 cấp'. S3 [05] chuyển RUNNING (L90) trước pre-scan. |
| A12 | Đọc được: legend đủ, không chồng lấn, mũi tên có nhãn, màu có chú nghĩa | Review trực quan | ✅ | ✅ | ❌🟡 | S3 (drawio cũ): `[01][02][05]` không nhãn, cạnh →S3 thiếu đầu mũi, `[10]/[11B]` sai vùng (M05/M13). S1 chấm trên Blueprint md — PNG V2/v2.1 chưa có trong lô này. |
| A13 | **Traceability**: mỗi thành phần/luồng truy ngược được về ≥1 nguồn (biên bản / Task 1–4 / portal) — không có thành phần "tự nhiên mà có" | Ma trận §10 | ✅ | ✅ | ❌🟡 | S3: node/cạnh thiếu nhãn `[01][02][05]`, icon CI/CD vô danh, cạnh VPCE→KB không truy vết (DEF-X-003). |
| A14 | Mọi con số **chi phí** có nhãn, không bị coi là "hơi ảo" — đối chiếu đơn giá Fargate công bố | Biên bản §1; Task 2 Phụ lục A; Đối soát §3.9 | ❌🟡 | ✅ | ❌🔴 | S1: hàng TOTAL L508 không nhãn (G-02); `$288→$22` (L428) không nhãn; **'3 endpoint × $7.3' nhưng L425–427 liệt kê 4 interface endpoint** (ecr.api, ecr.dkr, logs, sts) → tính lại (NEW-02). S3: bảng §5 không cột Nhãn; `$0.085/GB`, `<1s`, `2–5s`, `~$0.045–0.085/job` (L243) không nguồn/ngày (DEF-X-010). S2: không có số chi phí. |
| A15 | Draft nộp đúng nơi quy định, có file nguồn chỉnh sửa được (không chỉ ảnh chụp) | Quy ước team | ✅ | ❌🟡 | ❌🟡 | S2: md L18 vẫn trỏ `Workflow_Hung.pdf` bản cũ (md tự ghi 'cần export lại'); `Final_workflow.png` chưa kèm file nguồn. S3: 5 link `file:///c:/Users/T14S/...` (L5, L6, L169, L171, L316) — M12. |

### CHECKLIST B — Sơ đồ của Hoàng · Kiến trúc tổng thể (S1) — *chấm trên Blueprint md*

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | S1 Hoàng | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--- |
| B01 | Khớp **Task 2 §8 v0.2**: Job Controller nhận `ToolIntent JSON` → tra `TenantBinding` → **dispatch ngang hàng** (mỗi job = 1 Fargate task riêng) — **không** vẽ mô hình "Fargate lồng Fargate" | Task 2 §8 + ghi chú v0.2 | ✅ | L271 + L236/L240–245: JC → ECS RunTask ngang hàng, không Fargate lồng Fargate. |
| B02 | Job Controller hiện diện với **lease / heartbeat / recovery** | Task 2 §8 (node JC) | ✅ | LEASE coordinator L174/L295; heartbeat 60s + TTL 5m ghi rõ L354/L357 (M-05 đóng). |
| B03 | Luồng bằng chứng: runner → **S3 + Object Lock trực tiếp (direct-to-S3)** — **không** vẽ "lưu bằng chứng trên EC2" (ngộp storage — điểm mở của biên bản §2) | Đối soát §3.4 (P1); Task 2 §8 | ✅ | L255 runner → S3 trực tiếp; L256 chỉ hash về state. |
| B04 | Nền tảng thực thi = **ECS Fargate task-per-job, 4 image ECR pre-baked, interface `IsolatedRunner`** — không phải CodeBuild/Lambda (câu hỏi mở biên bản §2 đã chốt) | Đối soát §3.3 (P1 — "Chốt Fargate task-per-job") | ✅ | L119–121, L210, L444 (4 image ECR pre-baked), `IsolatedRunner` L91–111. |
| B05 | Các domain vẽ đúng wave & công cụ: **D5a** Semgrep+Trivy (W1) · **D5b** ZAP/nuclei (W3, chỉ khi có Staging URL sống) · **D2.b** Aurora Clone + Flyway · **D3** Playwright · **D4** k6 (DLT on AWS) | Task 2 §6.0; Đối soát §3.7 (P2) | ✅ | D5a W1 (L211), D5b W3 + điều kiện Staging URL (L216/L409), D2.b, D3 W2, D4 W2 (L212–216). |
| B06 | **NoSQL** không bị bỏ trống: có phương án DynamoDB Local/Ephemeral Table (TTL 1h + hook cleanup) — biên bản §2 đòi xử lý rollback NoSQL | Đối soát §3.5 (P1); Biên bản §2 | ❌🔴 | **0 hit DynamoDB/NoSQL** trong Blueprint (T_DB chỉ Aurora L215/L219). Phiếu vòng 1 chấm ✅ — bản Blueprint trong lô này không có; nếu bản trong repo có, gửi lại để chấm lại. |
| B07 | Egress: **private subnet không route internet + SG Deny All + VPC Endpoints** (S3, ECR, CloudWatch Logs) — đúng ADR-08 v0.2 | Task 2 §8 (node EGR) | ✅ | L123, L206, L421–427. Lưu ý L123 liệt kê VPCE thiếu STS, §4.4 có STS — nên đồng bộ. |
| B08 | `ToolIntent JSON` **không chứa credential / URL thật / SQL tùy ý** (Law 12–15) | Task 2 §8 (nhãn cạnh mũi tên) | ✅ | L133, L454. |
| B09 | Trả lời được câu mở biên bản §3: **input của các runner còn lại lấy từ đâu** — thể hiện `S03 ImpactSet ∩ S01 TargetBinding` quyết định chạy phần nào; **không** vẽ "PR sửa API → kích hoạt cả 6 domain runner" | Biên bản §3; Đối soát §3.6 (P0) | ❌🔴 | **Không có công thức `ImpactSet ∩ TargetBinding`**; C4 L2 (L240–245) và C4 L3 (L318) RESOLVER dispatch cả 6 runner vô điều kiện = đúng kiểu 'PR sửa API → chạy cả 6' mà Đối soát §3.6 (P0) cấm. Công thức chỉ có ở Detailed L121 / ảnh V2 `[10]`. |
| B10 | K6/Load test có cảnh báo **rủi ro AWS ban / self-DoS** ("tự DoS người nhà") và ranh giới chỉ chạy trên vùng được phép | Biên bản §3 (trang 2) | ✅ | L214 (⚠ cần DevOps duyệt), L435–437. |
| B11 | DB: **Aurora Serverless v2 clone cho SQL** + ghi chú *"chờ tham vấn Team Data về quota snapshot"* | Biên bản §2; Đối soát dòng P3 (điểm 6) | ✅ | L397 `PENDING tham vấn Team Data`. |
| B12 | Model vẽ đúng: **Claude trong sơ đồ = model Bedrock được Harness dùng**; Claude/Cursor trên máy dev **không** phải bộ chạy production | Portal — Cách đọc; Task 2 §8 | ✅ | L187–197: Claude = model Bedrock của Harness (Account B). |
| B13 | Mỗi quyết định có **exit note (Law 23)** hoặc gắn trạng thái `CANDIDATE` — không có đường một chiều ngầm | Task 2 §12 (bảng ADR-01…ADR-09) | ❌🟡 | Zone `[CANDIDATE]` đã có, nhưng từng quyết định (Dual-Model tiering, nơi chạy S03/S04) chưa có exit note; Task 3 tự khai runtime đang pin Opus 5, chưa tiering (Explained §7 B1). |
| B14 | Không claim vượt số liệu: chi phí / latency / coverage đều có nguồn + ngày đo | Task 2 Phụ lục A; Đối soát §3.9 | ❌🟡 | Xem A14; thêm L526 'triệt tiêu mọi mâu thuẫn' (claim tuyệt đối) và §6 dòng TOTAL. |
| B15 | Nhất quán với **sơ đồ kiến trúc 22/09 đang public** — chỗ nào khác phải giải thích được *vì sao* khác (đề xuất khác ≠ hiện trạng) | Portal `/diagrams` (ưu tiên 1) | ✅ | L13, L166/L185 `[CANDIDATE]`, FinOps L504–505 nêu hiện trạng 22/09 ghim `8a61cf66`. |

### CHECKLIST C — Sơ đồ của Hùng · Workflow (S2) — *chấm trên md + Final_workflow.png*

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | S2 Hùng | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--- |
| C01 | Đủ **10 lane S01–S10**, đúng thứ tự và đúng chủ trì từng lane (EC2 giữ auth/persist/governance; TIJobRunner trên AgentCore Runtime tính pipeline S02–S09; API xác minh & lưu ở S08) | Portal — "Chức năng một lượt đánh giá"; Task 4 §3 | ❌🟡 | PNG chỉ gắn nhãn S01–S07 + S10; S08/S09 không có nhãn riêng; chủ trì S03/S04 mâu thuẫn Detailed (A11). |
| C02 | **Tenant Binding quyết định chạy phần nào** — không vẽ "PR sửa API → chạy cả 6 domain runner"; input các runner lấy từ `S03 ImpactSet ∩ S01 TargetBinding` | Biên bản §3; Đối soát §3.6 (P0) | ❌🔴 | **PNG #19–#21 + md L48**: dispatch song song D5a / API / D3 / D4 / D2.b **không điều kiện** — thiếu `ImpactSet ∩ TargetBinding`; D5a bị gọi lần 2 (đã pre-scan ở #7). |
| C03 | **Không có vòng tự thử lại** cho lỗi thực thi (retry phải vẽ tường minh khác vòng lặp, hoặc ghi policy riêng) | Portal — "tiếp nhận và xử lý artifact: lỗi thực thi không được vẽ thành vòng tự thử lại" | ✅ | Không có vòng retry ngầm (loop duy nhất = heartbeat). Retry khi eval fail chưa vẽ → theo dõi ở C-3 (Connect Report). |
| C04 | `completed ≠ PASS` (Law 18) — "completed là xử lý xong, không phải mọi check đều pass" được diễn đạt đúng | Portal — "vòng đời tác vụ"; Task 4 §2.2 | ✅ | PNG #33 'COMPLETED (Law 18: completed ≠ PASS)'; gate tách ở #30–#32; md 'Quy tắc Law 18'. |
| C05 | Ba ranh giới thể hiện rõ: **`CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · review/publish/revoke tách quyền** | Portal — "Ba ranh giới không được xóa" | ❌🟡 | Có CANDIDATE (md) và 'Khuyến nghị ≠ phê duyệt' (#29 + md); **thiếu** 'review/publish/revoke tách quyền'; PNG không có banner CANDIDATE. |
| C06 | Entry/Exit criteria khớp **Task 4 §6**: PASS / DO_NOT_PASS (hard stop: Critical, secret lộ, hash mismatch, rollback fail) / HOLD — vẽ đúng nhánh | Task 4 §6.1–§6.2 | ✅ | md L59: Critical>0, SECRETS_LEAKED>0, SHA mismatch, rollback fail. PNG #32 nên bổ sung 'rollback fail'. |
| C07 | HOLD có tiêu chí **Faithfulness** (`FaithfulnessScore < 0.85 → HOLD`) bên cạnh Groundedness ≥ 0.80 | Đối soát §3.9 (P1 — thiếu Faithfulness); Biên bản §4 | ❌🟡 | Có `Faithfulness < 0.85 → HOLD` (PNG #16, md) nhưng **chưa nêu `Groundedness ≥ 0.80`**. |
| C08 | **6 chỉ số GenAI** (Groundedness, Precision…, gồm Faithfulness sau khi gộp) + cơ chế **temperature 0.0 + tolerance ±0.03** cho 2 lần chạy khác điểm | Task 4 §5.3; Đối soát §3.9; Biên bản §4 | ❌🟡 | Có '6 chỉ số, ±0.03' nhưng **không liệt kê 6 chỉ số** và **thiếu `temperature 0.0`** (0 hit). |
| C09 | "6 đặc tính đánh giá AI" của Hùng và của Trang **đã gộp làm một** (không còn 2 bộ song song) | Biên bản §4 | ❌🟡 | Chưa có bằng chứng đã gộp 6 đặc tính Hùng + Trang; Task 4 §5.3 (6 chỉ số) không gồm Faithfulness (Explained §7 C1/C2). |
| C10 | Outbox: **cần receiver ACK (HTTP 2xx)**, publisher idempotent, hai tenant K1/K2 có bảng chứng riêng | Portal — "Receipt triển khai 14/09 · Nét đạt" | ➖ | Outbox/ACK không thuộc luồng artifact-job trong PNG → đề xuất N/A có lý do; **cần Nghĩa xác nhận bằng chữ** (§5 ADR-0003) hoặc thêm 1 dòng scope note. |
| C11 | Vòng đời job vẽ đủ 4 trạng thái `queued / running / completed / failed`; **từ chối tiếp nhận xảy ra TRƯỚC khi có job** | Portal — "vòng đời tác vụ" (Lưu trữ V1 · 06) | ❌🟡 | PNG: `queued` #3, `RUNNING` #20, `COMPLETED` #33 — **thiếu `failed`**; state viết HOA ↔ portal lowercase; từ chối tiếp nhận (422/401) trước khi có job chưa vẽ (Admission #18 nằm SAU 202). |
| C12 | Phân biệt rõ **luồng artifact job (POST /v2/artifact-jobs)** với **`answer_change` kế thừa** — không trộn hai hợp đồng | Portal — "answer_change kế thừa" | ❌🟡 | Chưa phân biệt `/v1/testing/changes` (FROZEN) với `/v2/artifact-jobs`. |
| C13 | Human review có quyền lực thật: **reviewer kiểm soát candidate decision** (Law 9), người duyệt quyết định review/publish | Task 4 §2.2; Portal — "một lượt đánh giá" | ✅ | PNG #37–#38 + md 'Yếu tố Con người'. Nên ghi rõ `DO_NOT_PASS` không waiver. |
| C14 | **Prompt injection** được thể hiện đúng góc độ của TI: quét mã độc/payload độc hại lọt vào **source code** (TI không có UI Chat) | Biên bản §3 | ✅ | PNG #8 + md L33: gap prompt-injection = quét payload độc trong source. |
| C15 | Nhất quán với sơ đồ **luồng chức năng S01–S10** đang public (nét đứt = phần chưa có lượt đo live: Gateway/ToolIntent, outbox chờ ACK, Memory reuse) | Portal `/diagrams` (ưu tiên 1) | ⏳ | Cần đối chiếu `/diagrams` live (nét đứt Gateway/ToolIntent, outbox, Memory reuse). |

### CHECKLIST D — Sơ đồ của Trang · Chi tiết service & tool (S3) — *chấm trên Detailed md + drawio cũ*

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | S3 Trang | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--- |
| D01 | Mỗi thành phần ghi rõ **tên service AWS thật + region** (`ap-southeast-1` / `us-east-1`) — không có ô chung chung "database" hay "compute" | Portal — Hạ tầng 22/09; Task 1 | ✅ | Detailed nêu service + region (Account A `ap-southeast-1`, B `us-east-1`, Sandbox `ap-southeast-1`). |
| D02 | Cổng & giao thức đúng hiện trạng: CloudFront → API `:8000` và portal `:8001` trên **cùng một EC2** (hai container không chung ổ: `/app/data` chỉ gắn ở API và portal) | Portal — Hạ tầng (đo 22/09, nguồn ghim `8a61cf66`) | ❌🟡 | Không có footnote hiện trạng EC2 (`:8000/:8001`, ghim `8a61cf66`); L36 ghi API 'on AWS Fargate' như đã chốt (M06). |
| D03 | Tool versions & **nơi lưu trữ script K6/Playwright được chỉ rõ** (điểm mở của biên bản §3: "cần làm rõ nơi lưu trữ các kịch bản K6 và Playwright") | Biên bản §3 | ❌🟡 | 0 hit `ti-test-packs` (M07). |
| D04 | 4 image **ECR pre-baked** + digest đúng; đường giao hàng `GitHub Actions → ECR → SSM` vẽ **nét đứt** (đường khai báo, chưa đo live) | Portal — Hạ tầng; Đối soát §3.3 | ❌🟡 | 0 hit pre-baked / ECR digest / đường `GitHub Actions → ECR → SSM` (M11). |
| D05 | Nền tảng thực thi phản ánh chốt **Fargate task-per-job** — nếu còn vẽ CodeBuild/Lambda cho runner thì Fail (cold start 45–90s + tải Chromium mỗi lần = lãng phí) | Đối soát §3.3 (P1); Biên bản §2 | ✅ | 0 hit CodeBuild/Lambda — DEF-X-008 ĐÓNG ở Detailed. |
| D06 | **Semgrep + Trivy** được đặt đúng chỗ S02 và bao quát đúng hướng Testing (biên bản: hiện chỉ tập trung scan source code, chưa đáp ứng hướng scan của Testing) — nếu chưa đủ thì phải ghi là gap, không được vẽ như đã làm | Biên bản §3; Task 2 §6.0 (D5a) | ❌🔴 | Chưa có gap Semgrep/Trivy + prompt-injection (M10); `[06]` pre-scan Trục 1 nhưng `[10]` (L120, L203) lại dispatch 'cụm 6 Trục' gồm Trục 1 → DEF-S3-011. |
| D07 | Sơ đồ **Bedrock thuần đã được chẻ lại cho rõ** (biên bản §4 yêu cầu) — tách bạch: model Bedrock / AgentCore Harness dùng model nào / pipeline Bedrock Evaluations | Biên bản §4; Portal — Cách đọc | ❌🟡 | Có Harness/Models/Evaluations nhưng **thiếu AgentCore Gateway/Policy**; tên model sai (M03). |
| D08 | AgentCore **Harness / Gateway / Runtime** đúng version theo receipt được trích (Runtime 18 @ receipt 14/09) và ghi rõ receipt nào | Receipt 14/09; `/evidence` | ❌🟡 | 0 hit Runtime 18 / receipt (M15). |
| D09 | Không có credential/secret nào vẽ trong prompt hay ToolIntent; **secrets do backend giữ, tool chỉ nhận URL đã binding** (Law 12–15) | Task 2 §4 (mục 10.1); Task 4 §2.2 | ❌🔴 | **Secrets Manager & STS Tokens nằm trong Account B** (L56; drawio) — ngược Law 13 và chính §6.2 L286 (DEF-X-007). |
| D10 | Evidence path đầy đủ: **normalize + hash SHA-256 (Law 16)** → `raw_result_s3_uri` → S08; **không** vẽ raw result quay lại storage trên EC2 | Task 4 §5.4; Đối soát §3.4 | ✅ | `[11A]/[11B]/[12A]` (L124–L129): raw → S3, envelope 2KB + SHA-256, không quay về EC2. |
| D11 | DB testing vẽ đủ **hai nhánh**: SQL = Aurora Serverless v2 clone + Flyway; NoSQL = DynamoDB Local/Ephemeral (TTL 1h + cleanup) | Đối soát §3.5 | ✅ | Trục 4: Aurora Clone + DynamoDB TTL 1h + DeleteTable (L159). (drawio còn 'an toàn 100% DB gốc' — xem F02.) |
| D12 | Backup/backup scope **trung thực**: versioning ≠ WORM; phạm vi sao lưu chỉ backend `ap-southeast-1`; 4 lệnh backup trả kết quả rỗng = "đã hỏi và được trả lời là không có" | Portal — Hạ tầng | ✅ | Không claim về backup. |
| D13 | Không vẽ service đã EOL (**CodeGuru Security** — thay bằng Semgrep + Trivy hoặc Amazon Inspector) | Đối soát §3.1 (P0) | ✅ | Không CodeGuru. |
| D14 | Mọi con số (dung lượng, giá, latency, %) kèm **nguồn + ngày** và nhãn `OBSERVED/INFERRED/CANDIDATE` | Đối soát §3.9; Task 2 Phụ lục A | ❌🔴 | Xem A14. |
| D15 | Nhất quán với sơ đồ **hạ tầng đang public** và với 2 sơ đồ của Hoàng/Hùng (cùng tên service, cùng port, cùng region) | Portal `/diagrams` + A11 | ❌🔴 | Lệch Hùng/Blueprint về S03/S04 (A11); VPCE §1 L42 (Account A) ↔ §3 L142 (Sandbox) — DEF-X-005; không đối chiếu sơ đồ hạ tầng 22/09. |

### CHECKLIST E — Đối chiếu evidence (Vòng 2)

> ⚠️ Vòng 2 chỉ chính thức mở khi Vòng 1 đạt 100%. Các mục dưới đây là **chấm trước** những phần không cần gọi live; E01/E02 (và A09, C15) phải chạy lại ngày ký kèm Evidence Pack `URL + timestamp + kết quả`.

| ID | Nội dung kiểm tra | Nguồn đối chiếu | S1 Hoàng | S2 Hùng | S3 Trang | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--: | :--: | :--- |
| E01 | Truy cập `/diagrams` live tại ngày review, ghi lại **tiêu đề + ngày đo + nhóm** của từng sơ đồ → khớp claim của 3 draft | `https://d1tibdarzmw3jq.cloudfront.net/diagrams` | ⏳ | ⏳ | ⏳ | Phiên này **không gọi được portal live** (chỉ mở được URL có trong kết quả search). Chạy lại ngày ký; EV-1 (27/09) giữ làm tham chiếu. |
| E02 | Đọc `/evidence` → **revision đang phục vụ NGAY BÂY GIỜ** khớp (hoặc được giải thích đúng) với tham chiếu trên draft | `/evidence` | ⏳ | ⏳ | ⏳ | Như E01 — EV-2 manifest 2026-09-23.2 là của 27/09, chưa refresh. |
| E03 | Claim "V2 / receipt 14/09" đối chiếu đúng **source `fbdd8dfc…`, image `efabf57dcd8c`, AgentCore Runtime 18** và ghi rõ đây là **mốc lịch sử**, không phải bản đang chạy | Receipt 14/09 (Portal — Receipt) | ✅ | ✅ | ✅ | S1 L447 đúng (`fbdd8dfc`, `efabf57dcd8c`, Runtime 18, mốc lịch sử); S2/S3 không claim. |
| E04 | Claim liên quan 23/09 đối chiếu receipt `0409cb5a` (ảnh ECR `sha256:b717be38…`) — nhớ: receipt này **chỉ đọc lại 4 tuyến công khai, không đo lại kiến trúc** | Portal — Lượt triển khai 23/09 | ✅ | ✅ | ✅ | Không hồ sơ nào claim receipt `0409cb5a` 23/09. |
| E05 | Tham chiếu commit: nếu draft nói "theo lượt đo 22/09" thì phải là `b55fed16…` (ảnh `58abe918…`); **không được trộn** với lượt `7dca7c0a` / `253e702e` (CI run `35694235318`) cùng ngày | Portal — Lần đo kiến trúc 22/09 | ✅ | ✅ | ✅ | S1 dùng `8a61cf66` đúng ngữ cảnh 'hiện trạng đo 22/09'; không trộn `7dca7c0a`/`253e702e`. |
| E06 | Hợp đồng API: `OpenAPI 3.1.0 · info.version 1.0.0`, policy `artifact-v1` — đường dẫn **v1 `/v1/testing/changes` LIVE nhưng FROZEN**, job mới là `POST /v2/artifact-jobs` | Portal — API; `/api` | ✅ | ✅ | ✅ | `POST /v2/artifact-jobs` khớp EV-1. |
| E07 | Các claim "đã đo được / nét liền" được chứng minh bằng **gọi thật**: `GET /docs · /runs · /knowledge · /diagrams` trả 200 (portal ghi rõ 200 toàn bộ, `diagrams` 280 từ) | Gọi API trực tiếp | ✅ | ✅ | ✅ | Không hồ sơ nào claim HTTP 200 — ngoài phạm vi claim. |
| E08 | Claim về mã lỗi (`422` đủ điều kiện; `401 AUTHORITY_INSUFFICIENT` cho caller thiếu quyền) được **thử lại** đúng kịch bản | Gọi API + portal — Nét liên = đã đo được hơn này | ✅ | ✅ | ✅ | Không hồ sơ nào claim 422/401. |
| E09 | Trạng thái nghiệm thu: **#97 còn mở** → bất kỳ đâu viết "đã nghiệm thu/đủ bằng chứng" = **Fail** | Portal — Theo dõi nghiệm thu #97 | ✅ | ✅ | ✅ | Không có câu 'đã nghiệm thu'. |
| E10 | Mọi `runtime_binding` / Memory reuse / outbox ACK vẽ **nét đứt + nhãn UNVERIFIED** đúng hiện trạng | Portal — Nét đứt là phần chưa có lượt đo | ✅ | ✅ | ❌🟡 | S3: Memory không nhãn `UNVERIFIED` (M16). S1 L196/L259, S2 PNG #34 + md. |

### CHECKLIST F — 10 điểm nóng của Biên bản 23/09/2026 (áp cho cả 3 sơ đồ)

| ID | Nội dung kiểm tra | Nguồn đối chiếu | S1 Hoàng | S2 Hùng | S3 Trang | Ghi chú / căn cứ |
| :--: | :--- | :--- | :--: | :--: | :--: | :--- |
| F01 | **Quản lý chi phí "hơi ảo"** (§1) | Mọi con số chi phí trên sơ đồ đã được tính lại theo đơn giá thật + gắn nhãn `CANDIDATE` (chưa `OBSERVED`) chưa? | ❌🟡 | ✅ | ❌🔴 | S1/S3: xem A14 (TOTAL L508 không nhãn; S3 cột Nhãn thiếu). S2 không có số. |
| F02 | **Tổng kết báo cáo chưa đúng** (§1) | Các nhãn/tiêu đề/summary trên sơ đồ đã được đọc lại từng dòng cho đúng chưa? | ❌🟡 | ❌🟡 | ❌🔴 | S1 L526 'triệt tiêu mọi mâu thuẫn'. S2 L12/L42/L71 'tuyệt đối', L41 'triệt để'. S3 L7/L122/L126/L249 '100%', L316 'đồng bộ hoàn hảo', L158 'báo sai bằng 0', L35/L78 'WAF verify HMAC' (sai kỹ thuật, M14). |
| F03 | **CodeBuild/Lambda — setup Playwright mỗi lần có lãng phí?** (§2) | Sơ đồ đã nêu rõ *lý do chọn* nền tảng (Fargate task-per-job + image pre-baked) thay vì để ngỏ câu hỏi chưa? | ✅ | ❌🟡 | ❌🟡 | S2/S3 không nêu lý do chọn Fargate task-per-job + image pre-baked (thay CodeBuild/Lambda cold start 45–90s). S1 L119–121, L444. |
| F04 | **EC2 storage ngộp vì evidence** (§2) | Luồng evidence có đi thẳng ra object storage (S3 + Object Lock) thay vì chồng lên EC2 không? Dung lượng/retention có ghi rõ (hoặc ghi "chưa đo")? | ✅ | ❌🟡 | ✅ | S2: Object Lock (#28) nhưng không ghi retention / 'chưa đo'. S1 L366 & S3 L39 ghi WORM 90 ngày. |
| F05 | **NoSQL rollback khi test pass/fail** (§2) | Sơ đồ có phương án NoSQL (DynamoDB ephemeral + TTL + cleanup) chứ không chỉ Aurora clone cho SQL? | ❌🔴 | ❌🔴 | ✅ | NoSQL: S1 (B06) không có; S2 chỉ 'D2.b (DB Clone)'. S3 L159 có DynamoDB. |
| F06 | **Orchestrator không điều phối được, Tenant Binding mới quyết định** (§3) | Luồng điều phối thể hiện đúng `ImpactSet ∩ TargetBinding` và input của từng runner chưa? | ❌🔴 | ❌🔴 | ✅ | S1/S2: không có `ImpactSet ∩ TargetBinding` (B09/C02). S3 L121–122 có + `SKIPPED`. |
| F07 | **Nơi lưu script K6/Playwright chưa rõ** (§3) | Sơ đồ chi tiết (Trang) đã chỉ đích danh nơi lưu + cách version các script này chưa? | ✅ | ➖ | ❌🟡 | S1 L441. S3: 0 hit. S2: câu hỏi chỉ định sơ đồ chi tiết của Trang → đề xuất N/A (Nghĩa xác nhận bằng chữ). |
| F08 | **K6 dễ bị AWS ban / "tự DoS người nhà"** (§3) | Có ranh giới/ cảnh báo vùng chạy load test và bước xin phép DevOps chưa? | ✅ | ❌🟡 | ❌🟡 | S1 L435–437. S2: D4 (Perf) chạy trong `par` không cảnh báo DoS/xin phép DevOps. S3: Trục 5 có 'Bộ 3 Khóa' nhưng thiếu cảnh báo (M08). |
| F09 | **Semgrep/Trivy chưa đáp ứng hướng scan của Testing; Prompt injection = quét payload trong source** (§3) | Sơ đồ có thể hiện đúng phạm vi scan hiện tại **và** gap cần đóng, đúng góc độ prompt-injection của TI không? | ✅ | ✅ | ❌🟡 | S1 L414–416; S2 PNG #8 + md L33. S3 chưa có (M10). |
| F10 | **GenAI: evidence chưa đủ thuyết phục, 2 lần chạy 2 kết quả, thiếu Faithfulness, gộp 6 đặc tính Hùng+Trang, sơ đồ Bedrock chẻ lại** (§4) | Sơ đồ workflow/chi tiết đã: gộp 6 đặc tính; thêm Faithfulness; nêu cơ chế ổn định điểm (temp 0, tolerance ±0.03); chẻ rõ pipeline Bedrock Evaluations/AgentCore chưa? | ❌🟡 | ❌🟡 | ❌🔴 | S1: có `Faithfulness ≥ 0.85 ±0.03` (L367) nhưng thiếu 6 chỉ số/`temp 0.0`/chẻ pipeline. S2: xem C07/C08. S3: chỉ 2 chỉ số, không ±0.03 (DEF-X-009). |


---

## 7A. DEFECT LOG HỢP NHẤT & VIỆC PHẢI CHỐT *(kết quả điền 28/09/2026)*

### 7A.1. Blocking — phải sửa trước re-review

| ID | Mục checklist | Hồ sơ | Hiện trạng (căn cứ) | Cách sửa | Owner | Trạng thái |
| :-- | :--: | :--: | :--- | :--- | :--- | :--: |
| DEF-X-006 / M-01 | A11, F06 | S1·S2·S3 | S03/S04: Blueprint L127 (AI) ↔ L350 (JC); Hùng = JC (Account A); Detailed L63/L107/L318 = Account B | Chốt 1 câu (Q1 §7A.3), sửa đồng bộ cả 3 + Explained + Gap v2.2 | Nghĩa + Hoàng → escalate Tan.Thai | 🔴 OPEN (P0) |
| DEF-S3-011 / M-08 | A11, C02, D06 | S2·S3·S1 | D5a chạy 2 lần: Hùng PNG #7 & #21 (md L32/L48); Detailed `[10]` L120/L203 liệt kê Trục 1 lại; Blueprint C4 L240 dispatch `T_SAST` | Chốt "đúng 1 lần ở `[06]`; `[10]` chỉ Trục 2–6" (Connect Report §0); cập nhật Task 4 §3.1.1 | Hùng + Trang + Hoàng | 🔴 OPEN |
| **NEW-01** | B09, C02, F06 | S1·S2 | Không có `ImpactSet ∩ TargetBinding`; RESOLVER/`par` dispatch tất cả runner vô điều kiện | Copy Detailed L121–122 (`Target = Verified ImpactSet ∩ TargetBinding`, ngoài tập → `SKIPPED`) vào Blueprint §3.1/§3.2 và PNG Hùng #19–#21 | Hoàng + Hùng | 🔴 OPEN |
| **NEW-02** | B06, F05 | S1·S2 | Blueprint 0 hit DynamoDB; Hùng chỉ 'D2.b (DB Clone)' | Thêm nhánh NoSQL: DynamoDB Local/Ephemeral Table, TTL 1h + hook `DeleteTable` (Detailed L159) | Hoàng + Hùng | 🔴 OPEN |
| DEF-X-002 | A02, A03, D15 | S3 | 0 CANDIDATE/legend trong Detailed + drawio; không ghi hiện trạng EC2 | Banner `THIẾT KẾ ĐÍCH — CANDIDATE`; legend liền = đã đo / đứt = chưa đo; footnote hiện trạng (đo 22/09, ghim `8a61cf66`) | Trang | 🔴 OPEN |
| DEF-X-004 | D15 | S3 | `--network none` L49, L138, L142 + drawio | Câu chuẩn Blueprint L421: *Private Subnet không IGW/NAT + SG DENY ALL EGRESS + VPC Endpoints (ECR/Logs/STS; S3 Gateway $0)* | Trang | 🔴 OPEN |
| DEF-X-005 | A11, D15 | S3 | VPCE ở Account A (L42) ↔ Sandbox (L142) | Chuyển VPCE về **Sandbox VPC** | Trang | 🔴 OPEN |
| DEF-X-007 | D09 | S3 | Secrets Manager & STS trong Account B (L56; drawio) | Chuyển sang **Account A** (Law 13; khớp §6.2 L286) | Trang | 🔴 OPEN |
| DEF-X-009 | F10, F02 | S3 | `[12B]` L130/L218/L251 gộp `COMPLETED` + gate; không Waiver, không điều kiện DO_NOT_PASS, không ±0.03, không 6 chỉ số, không `completed ≠ PASS` | Copy hộp Gate Blueprint L367–378; tách `state` ↔ `gate_result` | Trang + Hoàng | 🔴 OPEN |
| DEF-X-010 | A14, F01, D14 | S3·S1 | S3: bảng §5 không cột Nhãn, `$0.085/GB`, `<1s`, `2–5s`, `~$0.045–0.085/job`; S1: TOTAL L508 không nhãn | Thêm cột Nhãn (mặc định `CANDIDATE`); SLA = *mục tiêu thiết kế*; TOTAL ghi `CANDIDATE (hiện trạng ghim 8a61cf66 vẫn 1 EC2)` | Trang + Hoàng | 🔴 OPEN |

### 7A.2. Minor — sửa ≤ 3 ngày làm việc

| ID | Mục | Hồ sơ | Nội dung | Trạng thái |
| :-- | :--: | :--: | :--- | :--: |
| NEW-03 | A11 | S1 | Sequence L348–350: S04 (RiskTier) chạy **sau** Harness, nhưng Opus 5 chỉ gọi khi `Risk == CRITICAL` tại S04 (L130) → đảo thứ tự: S03/S04 trước, rồi S05/S06 (PNG Hùng #11–#15 đúng thứ tự này) | OPEN |
| NEW-04 | A14, B14 | S1·S3 | "3 endpoint × $7.3 ≈ $22" (Blueprint L428; Detailed L42/L142) nhưng liệt kê 4 interface endpoint; L503 ghi `$7.3/endpoint/AZ` → tính lại theo số endpoint × số AZ, đối chiếu bảng giá AWS ngày cụ thể, nhãn `CANDIDATE` | OPEN |
| NEW-05 | A11 | S1 | Sơ đồ ASCII L64 còn "Model Tiering 3 cấp" ↔ L128 "2 cấp" | OPEN |
| NEW-06 | A01, A07, C01, C05, C07, C08, C11, F04 | S2 | PNG: thiếu banner CANDIDATE, ngày/tác giả, region, nhãn S08/S09, `failed`, tên 6 chỉ số, `Groundedness ≥ 0.80`, `temperature 0.0`, retention S3, review/publish/revoke; state viết HOA | OPEN |
| NEW-07 | — | Review | Gap Closure v2.2: đọc Detailed L100–102 là "S03/S04 tại JC" (thực tế L63/L107/L318 = Account B); số dòng lệch ±3 (G-08 ghi L53, thực tế L56) → chạy lại `Select-String` trên bản trong repo trước khi phát phiếu | OPEN |
| DEF-S3-M01 | A01 | S2·S3 | Thiếu ngày/người vẽ/nhóm nhãn | PARTIAL |
| DEF-X-M02 | A11, C11 | S1·S3 | `QUEUED` (S1 L284/L343) ↔ `PENDING` (S3 L84/L86/L188/L237) ↔ `queued` (S2) → dùng `queued/running/completed/failed`, `gate_result` cột riêng | OPEN |
| DEF-X-M03 | A05, D07 | S3 (+images) | `Claude 5.0 Sonnet` (L51/L107/L243); thiếu AgentCore Gateway/Policy; `images/*` còn node `m_haiku` (ngoài lô) | OPEN |
| DEF-S3-M05 / M13 | A12, A13 | S3 | drawio cũ: `[01][02][05]` không nhãn, cạnh →S3 thiếu mũi tên, `[12D]` vẽ RDS→CI/CD (Detailed L220/L253) → `CI → GET /v2/artifact-jobs/{id} → TI API v2 → RDS` | OPEN |
| DEF-S3-M06 | D02 | S3 | Chưa có footnote hiện trạng EC2 (`:8000/:8001`, ghim `8a61cf66`) | OPEN |
| DEF-S3-M07 · M08 · M09 · M10 · M11 · M15 · M16 | D03, F08, F09, D04, D08, E10 | S3 | Thiếu `ti-test-packs/`; cảnh báo DoS/AWS-ban + xin phép DevOps; `PENDING Team Data` (Aurora); gap scan + prompt-injection; ECR pre-baked + digest + đường nét đứt; Runtime 18 receipt; Memory `UNVERIFIED` — **Blueprint đã có đủ (L397, L406–416, L435–447, L196)** → copy nguyên sang Detailed | OPEN |
| DEF-S3-M12 | A15 | S3 | 5 link `file:///c:/Users/T14S/...` (L5, L6, L169, L171, L316) → link tương đối | OPEN |
| DEF-S3-M14 | F02 | S3 | 'WAF … HMAC' (L35, L78) → *WAF (OWASP rules + rate limit); xác thực HMAC tại TI API v2 / Lambda@Edge* | OPEN |
| M-10 | A15 | S3 | `TI_System_Architecture.drawio` cũ chưa gắn `SUPERSEDED — dùng v2.1`; Detailed L5/L169/L316 vẫn trỏ bản cũ | OPEN |
| F02 claims | F02 | S1·S2·S3 | S1 L526; S2 L12/L41/L42/L71; S3 L7/L122/L126/L158/L249/L316/L320 — đổi thành câu có căn cứ (vd. "khớp theo v2.1 CANDIDATE ngày 28/09; các điểm lệch ở §…") | OPEN |

**Đã ĐÓNG (ghi nhận để khỏi regression):** DEF-X-001 (CodeGuru — cả 3 hồ sơ) · DEF-X-008 (CodeBuild — Detailed) · M-02/M-03/M-05/M-16 và D03/D04/D08/B10/B11 **trong Blueprint** · Hùng: Waiver, `queued`, D5b W3 + Staging URL, Trục 2, gap prompt-injection, câu mạng chuẩn, Faithfulness→HOLD, `completed ≠ PASS`.

### 7A.3. Sáu câu phải chốt (chỉ Nghĩa/Tan.Thai chốt được — mình chỉ đề xuất)

| # | Câu hỏi | Đề xuất (kèm lý do) | Sửa những file nào |
| :-: | :--- | :--- | :--- |
| **Q1** | **S03/S04 chạy ở đâu; ai là system of record?** | *"Harness (Account B) SINH ĐỀ XUẤT `ImpactSet`/`RiskTier` từ Context + SecurityFindings → Bedrock Evaluations chấm → trả `Verified ImpactSet`. Job Controller (Account A) là system of record: validate, ghi RDS, và tính `Target = ImpactSet ∩ TargetBinding` bằng mã cứng."* Khớp ảnh V2, Detailed, Connect Report §0 **và** Law 4.3 (AI không giữ sổ cái). Phương án còn lại (S03/S04 deterministic tại JC) buộc sửa Detailed + ảnh V2 `[08B]` + Connect Report. **Lưu ý xung đột lợi ích (§2):** Nghĩa là tác giả Task 3 → mời Tan.Thai/Product Architect đồng ký. | Blueprint L127/L350 (+ đảo thứ tự NEW-03); Hùng md L34 (Phase 1) + PNG #9–#13; Explained §3.1/§6; Gap v2.2 (G-01, điểm đã đóng #7) |
| **Q2** | Pre-scan D5a mấy lần? | **Đúng 1 lần** ở `[06]`; `[10]` chỉ dispatch Trục 2–6 (đã ADOPTED ở Connect Report §0). Cập nhật Task 4 §3.1.1. | Hùng PNG #21 + md L48; Detailed L120/L203; Blueprint C4 L240 |
| **Q3** | Lease/heartbeat | **TTL 5 phút + heartbeat 60s, ghi rõ là 2 tham số** (mẫu Blueprint L357). | Copy sang Detailed nếu có nhắc lease |
| **Q4** | Trạng thái khởi tạo & tách gate | `queued/running/completed/failed` (chữ thường, khớp portal); `gate_result` là cột riêng. | Blueprint L284/L343; Detailed L84–90/L188/L237; Hùng PNG #20/#33 |
| **Q5** | Bộ 6 chỉ số GenAI | Task 4 §5.3 liệt kê Groundedness ≥0.80, GoalSuccessRate ≥0.75, Precision ≥0.70, Recall ≥0.95, HallucinationRate ≤2%, InjectionResistanceRate ≥0.99 — **không có Faithfulness ≥0.85** nhưng Gate/HOLD đang dùng. Cần chốt: Faithfulness thay 1 chỉ số hay là chỉ số thứ 7; ±0.03 áp cho chỉ số nào. | Task 4 §5.3; PNG Hùng #16; Blueprint L367; Detailed |
| **Q6** | Eval fail → retry mấy lần → HOLD khi nào | Retry ≤ N (ghi log S3), quá N → `HOLD`; N do Nghĩa + Hùng chốt (C-3 Connect Report). Phải vẽ tường minh (C03). | Hùng PNG sau #16; Detailed `[09C]`; v2.1/V2 |

### 7A.4. Thứ tự fill đề xuất

1. **Chốt Q1 + Q2** (chặn mọi sửa còn lại).
2. **Detailed (Trang):** copy khối từ Blueprint — mạng L421, Gate L367–378, §4.5/§4.6 (L435–447), Team Data L397, gap scan L414–416 — rồi sửa 8 điểm đã liệt kê ở DEF-X-002/004/005/007/009/010 + M12/M14.
3. **Blueprint (Hoàng):** L127 + L350 theo Q1; thêm `ImpactSet ∩ TargetBinding` + NoSQL; TOTAL L508 có nhãn; L64; tính lại VPCE.
4. **Hùng:** bỏ D5a khỏi `par` (#21) + thêm điều kiện dispatch; export lại PNG/PDF kèm banner CANDIDATE, ngày/tác giả, region, 6 chỉ số + `temp 0.0`, `failed`.
5. Gắn `SUPERSEDED — dùng v2.1` cho drawio cũ; gửi lại V2 PNG + v2.1 drawio để chấm lại A12/A13/B09.
6. Chạy Vòng 2 (E01/E02/A09/C15) trên portal live rồi mới ký.

---

## 8. Quy trình xử lý Fail

### 8.1. Luồng xử lý

```
Fail phát hiện ──► Ghi Defect Log (≤ 15 phút)
       │
       ├─ Blocking ──► Báo ngay người vẽ + owner ──► Sửa ≤ 24h ──► Re-review
       │
       └─ Minor ─────► Góp ý trong phiếu ──► Sửa ≤ 3 ngày ──► Gộp re-review
```

- **Blocking** = sai hiện trạng (không khớp `/evidence`), sai biên bản 23/09, sai quyết định Task 2/4 (P0/P1 trong Đối soát), claim vượt evidence.
- **Minor** = trình bày, legend, từ ngữ chưa chuẩn — vẫn phải sửa để đạt 100%.
- **Escalation**: 2 lần Fail cùng mục → họp ngắn Hoàng/Hùng/Trang + Nghĩa; vẫn chưa đồng ý → escalate Product Architect / Tan.Thai (đặc biệt các mục P0 Đối soát §3.1, §3.6).

### 8.2. Mẫu Defect Log

| Cột | Nội dung |
| :--- | :--- |
| ID | `DEF-S1-001` (sơ đồ · số thứ tự) |
| Mục checklist | Ví dụ `B03` |
| Mô tả | Mô tả ngắn, kèm vị trí trên sơ đồ (tên node/mũi tên) |
| Chứng minh | URL + timestamp + đoạn trích (Evidence Pack) hoặc dòng tài liệu |
| Mức | `Blocking` / `Minor` |
| Owner | Hoàng / Hùng / Trang |
| Deadline | T+24h / T+3 ngày |
| Trạng thái | `OPEN` → `FIXED` → `VERIFIED` / `REJECTED` (kèm lý do) |

### 8.3. Mẫu phiếu đánh giá 1 sơ đồ (tóm tắt)

```
[SƠ ĐỒ] .................. [NGƯỜI VẼ] ......... [NGÀY NHẬN] .....
Tổng mục: ..  Pass: ..  Fail: ..   (cần 100% Pass)
Blocking: ..  Minor: ..
Kết luận vòng 1: .............................................
Kết luận cuối (Nghĩa):  ĐẠT / CHƯA ĐẠT   [ngày]  [ký]
```

---

## 9. Tiêu chí nghiệm thu tổng thể (Exit Criteria) — *trạng thái 28/09/2026*

Một sơ đồ được đóng dấu **ĐẠT** khi **tất cả** điều kiện sau đúng:

| Điều kiện | S1 Hoàng | S2 Hùng | S3 Trang |
| :--- | :--: | :--: | :--: |
| Checklist A (15 mục) = 100% Pass | ❌ (11/15 ✅, ⏳ 1) | ❌ (8/15 ✅, ⏳ 1) | ❌ (5/15 ✅) |
| Checklist B / C / D (15 mục, theo chủ sơ đồ) = 100% Pass | ❌ (11/15) | ❌ (5/15, ➖ 1, ⏳ 1) | ❌ (6/15) |
| Checklist E (10 mục) = 100% Pass — mọi claim có Evidence Pack | ⏳ (8/10 ✅, 2 chờ live) | ⏳ (8/10 ✅, 2 chờ live) | ❌ (7/10 ✅, 2 chờ live) |
| Checklist F (10 mục) = 100% Pass — điểm nóng đã đóng hoặc ghi rõ "gap + kế hoạch" | ❌ (5/10) | ❌ (2/10, ➖ 1) | ❌ (3/10) |
| Defect Log không còn `OPEN` | ❌ | ❌ | ❌ |
| Đã qua **2 vòng** và Nghĩa ký | ❌ | ❌ | ❌ |

**Kết luận nghiệm thu 3 sơ đồ (dòng biên bản G5, dự thảo):** S1 35/50 ✅ · S2 23/50 ✅ · S3 21/50 ✅ — cả ba **CHƯA ĐẠT**; chưa mở Vòng 2. *(Chưa ký — chờ Nghĩa.)*

> **Ghi chú trung thực:** Đạt checklist này = *sơ đồ đáng tin làm căn cứ*. **Không** đồng nghĩa hệ thống đã được kiểm thử hay đã nghiệm thu (#97 vẫn mở, `runtime_binding` vẫn `UNVERIFIED`) — đúng tinh thần "bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy". Bản điền này cũng **chưa** kèm Evidence Pack live của ngày 28/09.

---

## 10. Ma trận truy vết (Checklist ↔ Nguồn ↔ Sơ đồ)

| Nhóm checklist | Sơ đồ áp dụng | Nguồn chính | Loại kiểm |
| :---: | :---: | :--- | :--- |
| A01–A15 | Cả 3 | Portal `/diagrams`, `GLOSSARY_TI.md`, Đối soát §3.1/§3.9 | Tính chung + nhất quán |
| B01–B15 | Hoàng (S1) | Task 2 §8 + Phụ lục B, Đối soát §3.3–§3.6 | Kiến trúc tổng thể |
| C01–C15 | Hùng (S2) | Task 4 §3–§6, Portal luồng S01–S10, Biên bản §3–§4 | Workflow |
| D01–D15 | Trang (S3) | Portal Hạ tầng 22/09, Task 1, Biên bản §2–§3, Đối soát §3.7 | Service/tool chi tiết |
| E01–E10 | Cả 3 | Portal live `/evidence` `/api`, receipt 14/09 & 23/09 | Đối chiếu evidence (vòng 2) |
| F01–F10 | Cả 3 | Biên bản họp TI 23/09/2026 | 10 điểm nóng |

**Tổng số mục:** 15 + 15 + 15 + 15 + 10 + 10 = **80 mục** (mỗi sơ đồ phải qua **50 mục**: A(15) + nhóm riêng(15) + E(10) + F(10) — *v1.0 ghi "60" là sai số học, đã sửa ở v1.1*).

---

## 11. Cách dùng nhanh (quick start)

1. Nhận draft → kiểm **Entry Criteria** (§4.1), thiếu thì trả về.
2. In/mở file này → chấm **Checklist A** rồi **Checklist B/C/D** theo người vẽ → ghi **Defect Log**.
3. Hết vòng 1 → chuyển **Checklist E + F** (mang theo Evidence Pack).
4. Re-review → chấm lại mục từng Fail → nếu 100% Pass → ký **ĐẠT** ở §8.3.
5. Ghi 1 dòng kết quả vào biên bản họp kế tiếp.

---

## CHANGELOG

| Ngày | Phiên bản | Người | Nội dung |
| :--- | :---: | :--- | :--- |
| 25/09/2026 | v1.0 | Nghĩa (Tester #4) | Tạo đầu tiên sau buổi grilling: chốt phương pháp 2 vòng (ADR-0001), thứ tự nguồn sự thật (ADR-0002), Pass/Fail nhị phân 100% (ADR-0003), cấu trúc tài liệu (ADR-0004); 80 mục checklist A–F |
| 28/09/2026 | v1.1 | Nghĩa (Tester #4) — bản điền do Claude soạn, **chờ Nghĩa xác nhận** | Điền P/F 3 sơ đồ (S1 Blueprint · S2 Hùng md+PNG · S3 Detailed+drawio cũ) theo Checklist A–F dựa trên phiếu góp ý 27/09, re-review v2.1, Gap Closure v2.2, Connect Report và đọc lại trực tiếp từng hồ sơ; thêm §0A (kết quả), §7A (Defect Log hợp nhất + 6 câu cần chốt + thứ tự fill); sửa §9 thành bảng trạng thái, sửa §10 "60→50 mục". Phát hiện mới: NEW-01…NEW-07 (S03/S04 mâu thuẫn P0, D5a 2 lần, thiếu `ImpactSet ∩ TargetBinding`/NoSQL ở Blueprint, số VPCE, phiếu v2.2 đọc sai Detailed). Chưa có Evidence Pack live 28/09 |
