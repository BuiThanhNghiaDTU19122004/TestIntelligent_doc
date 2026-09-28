# TASK 5 — KẾ HOẠCH XÁC MINH & CHECKLIST SHIFT-LEFT CHO 3 SƠ ĐỒ

**Dự án:** Testing Intelligence (TI)
**Tác giả:** Nghĩa — Tester #4 (phụ trách Shift-left testing, theo Biên bản họp TI 23/09/2026, Mục 5)
**Ngày tạo:** 25/09/2026
**Đối tượng xác minh:** 3 sơ đồ sẽ được vẽ sau họp 23/09/2026 — của Hoàng (kiến trúc tổng thể), Hùng (workflow), Trang (sơ đồ kiến trúc chi tiết gắn service/tool)
**Trạng thái:** ACTIVE — sẵn sàng chạy khi nhận draft đầu tiên
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

## 7. CHECKLIST XÁC MINH

> Cách dùng: mỗi mục chấm **✅ Pass / ❌ Fail**. Cột *Nguồn đối chiếu* là bắt buộc khi Fail — phải trích được.

### CHECKLIST A — Mục chung cho cả 3 sơ đồ

| ID | Nội dung kiểm tra (Pass nếu = Có/Đúng) | Nguồn đối chiếu | P/F |
| :--: | :--- | :--- | :--: |
| A01 | Sơ đồ có: tên, **ngày đo/ngày tạo**, người vẽ, nhãn nhóm (kiến trúc / chức năng / hạ tầng / lưu trữ) | Quy ước portal `/diagrams` | ☐ |
| A02 | Mọi khẳng định gắn nhãn `OBSERVED` / `INFERRED` / `CANDIDATE` — không trộn lẫn | Đối soát §3.9 (kỷ luật nhãn sự thật) | ☐ |
| A03 | Quy ước **nét liền = đo được trên live**, **nét đứt = khai báo/chưa đo** được giải thích bằng legend ngay trên sơ đồ | Portal — Cách đọc | ☐ |
| A04 | Không có câu nào tuyên bố **"đã nghiệm thu"** khi #97 còn mở | Portal — Theo dõi nghiệm thu #97 | ☐ |
| A05 | Thuật ngữ dùng đúng `GLOSSARY_TI.md` — **giống hệt** cách dùng trong 2 sơ đồ kia | `D:\Doc\Research\GLOSSARY_TI.md` | ☐ |
| A06 | **Không** xuất hiện dịch vụ AWS đã EOL (đặc biệt Amazon CodeGuru Security — ngừng 20/11/2025) | Đối soát §3.1 (P0) | ☐ |
| A07 | Hai tài khoản AWS vẽ đúng ranh giới: backend `ap-southeast-1` ↔ AgentCore Harness/Memory `us-east-1` | Portal — Hạ tầng; Task 2 §8 | ☐ |
| A08 | Trạng thái phụ (`UNVERIFIED`, `FROZEN`, `chưa nghiệm thu`, `READY/ACTIVE`) khớp giá trị thật tại ngày review | `/evidence` (ADR-0002 ưu tiên 1) | ☐ |
| A09 | Route/URL dẫn trên sơ đồ **tồn tại thật** (`/diagrams`, `/api`, `/evidence`, `/docs`, `/runs`, `/knowledge`) | Gọi thử portal/API | ☐ |
| A10 | Tham chiếu kỹ thuật (commit, image digest, Runtime) khớp portal tại ngày review — hoặc ghi rõ "theo lượt đo <ngày>" | `/evidence`, receipt | ☐ |
| A11 | **Nhất quán với 2 sơ đồ kia**: cùng tên component, cùng hướng luồng, cùng trạng thái | So sánh chéo 3 draft | ☐ |
| A12 | Đọc được: legend đủ, không chồng lấn, mũi tên có nhãn, màu có chú nghĩa | Review trực quan | ☐ |
| A13 | **Traceability**: mỗi thành phần/luồng truy ngược được về ≥1 nguồn (biên bản / Task 1–4 / portal) — không có thành phần "tự nhiên mà có" | Ma trận §10 | ☐ |
| A14 | Mọi con số **chi phí** có nhãn, không bị coi là "hơi ảo" — đối chiếu đơn giá Fargate công bố | Biên bản §1; Task 2 Phụ lục A; Đối soát §3.9 | ☐ |
| A15 | Draft nộp đúng nơi quy định, có file nguồn chỉnh sửa được (không chỉ ảnh chụp) | Quy ước team | ☐ |

### CHECKLIST B — Sơ đồ của Hoàng · Kiến trúc tổng thể (S1)

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | P/F |
| :--: | :--- | :--- | :--: |
| B01 | Khớp **Task 2 §8 v0.2**: Job Controller nhận `ToolIntent JSON` → tra `TenantBinding` → **dispatch ngang hàng** (mỗi job = 1 Fargate task riêng) — **không** vẽ mô hình "Fargate lồng Fargate" | Task 2 §8 + ghi chú v0.2 | ☐ |
| B02 | Job Controller hiện diện với **lease / heartbeat / recovery** | Task 2 §8 (node JC) | ☐ |
| B03 | Luồng bằng chứng: runner → **S3 + Object Lock trực tiếp (direct-to-S3)** — **không** vẽ "lưu bằng chứng trên EC2" (ngộp storage — điểm mở của biên bản §2) | Đối soát §3.4 (P1); Task 2 §8 | ☐ |
| B04 | Nền tảng thực thi = **ECS Fargate task-per-job, 4 image ECR pre-baked, interface `IsolatedRunner`** — không phải CodeBuild/Lambda (câu hỏi mở biên bản §2 đã chốt) | Đối soát §3.3 (P1 — "Chốt Fargate task-per-job") | ☐ |
| B05 | Các domain vẽ đúng wave & công cụ: **D5a** Semgrep+Trivy (W1) · **D5b** ZAP/nuclei (W3, chỉ khi có Staging URL sống) · **D2.b** Aurora Clone + Flyway · **D3** Playwright · **D4** k6 (DLT on AWS) | Task 2 §6.0; Đối soát §3.7 (P2) | ☐ |
| B06 | **NoSQL** không bị bỏ trống: có phương án DynamoDB Local/Ephemeral Table (TTL 1h + hook cleanup) — biên bản §2 đòi xử lý rollback NoSQL | Đối soát §3.5 (P1); Biên bản §2 | ☐ |
| B07 | Egress: **private subnet không route internet + SG Deny All + VPC Endpoints** (S3, ECR, CloudWatch Logs) — đúng ADR-08 v0.2 | Task 2 §8 (node EGR) | ☐ |
| B08 | `ToolIntent JSON` **không chứa credential / URL thật / SQL tùy ý** (Law 12–15) | Task 2 §8 (nhãn cạnh mũi tên) | ☐ |
| B09 | Trả lời được câu mở biên bản §3: **input của các runner còn lại lấy từ đâu** — thể hiện `S03 ImpactSet ∩ S01 TargetBinding` quyết định chạy phần nào; **không** vẽ "PR sửa API → kích hoạt cả 6 domain runner" | Biên bản §3; Đối soát §3.6 (P0) | ☐ |
| B10 | K6/Load test có cảnh báo **rủi ro AWS ban / self-DoS** ("tự DoS người nhà") và ranh giới chỉ chạy trên vùng được phép | Biên bản §3 (trang 2) | ☐ |
| B11 | DB: **Aurora Serverless v2 clone cho SQL** + ghi chú *"chờ tham vấn Team Data về quota snapshot"* | Biên bản §2; Đối soát dòng P3 (điểm 6) | ☐ |
| B12 | Model vẽ đúng: **Claude trong sơ đồ = model Bedrock được Harness dùng**; Claude/Cursor trên máy dev **không** phải bộ chạy production | Portal — Cách đọc; Task 2 §8 | ☐ |
| B13 | Mỗi quyết định có **exit note (Law 23)** hoặc gắn trạng thái `CANDIDATE` — không có đường một chiều ngầm | Task 2 §12 (bảng ADR-01…ADR-09) | ☐ |
| B14 | Không claim vượt số liệu: chi phí / latency / coverage đều có nguồn + ngày đo | Task 2 Phụ lục A; Đối soát §3.9 | ☐ |
| B15 | Nhất quán với **sơ đồ kiến trúc 22/09 đang public** — chỗ nào khác phải giải thích được *vì sao* khác (đề xuất khác ≠ hiện trạng) | Portal `/diagrams` (ưu tiên 1) | ☐ |

### CHECKLIST C — Sơ đồ của Hùng · Workflow (S2)

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | P/F |
| :--: | :--- | :--- | :--: |
| C01 | Đủ **10 lane S01–S10**, đúng thứ tự và đúng chủ trì từng lane (EC2 giữ auth/persist/governance; TIJobRunner trên AgentCore Runtime tính pipeline S02–S09; API xác minh & lưu ở S08) | Portal — "Chức năng một lượt đánh giá"; Task 4 §3 | ☐ |
| C02 | **Tenant Binding quyết định chạy phần nào** — không vẽ "PR sửa API → chạy cả 6 domain runner"; input các runner lấy từ `S03 ImpactSet ∩ S01 TargetBinding` | Biên bản §3; Đối soát §3.6 (P0) | ☐ |
| C03 | **Không có vòng tự thử lại** cho lỗi thực thi (retry phải vẽ tường minh khác vòng lặp, hoặc ghi policy riêng) | Portal — "tiếp nhận và xử lý artifact: lỗi thực thi không được vẽ thành vòng tự thử lại" | ☐ |
| C04 | `completed ≠ PASS` (Law 18) — "completed là xử lý xong, không phải mọi check đều pass" được diễn đạt đúng | Portal — "vòng đời tác vụ"; Task 4 §2.2 | ☐ |
| C05 | Ba ranh giới thể hiện rõ: **`CANDIDATE ≠ ĐÃ CHẠY` · `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` · review/publish/revoke tách quyền** | Portal — "Ba ranh giới không được xóa" | ☐ |
| C06 | Entry/Exit criteria khớp **Task 4 §6**: PASS / DO_NOT_PASS (hard stop: Critical, secret lộ, hash mismatch, rollback fail) / HOLD — vẽ đúng nhánh | Task 4 §6.1–§6.2 | ☐ |
| C07 | HOLD có tiêu chí **Faithfulness** (`FaithfulnessScore < 0.85 → HOLD`) bên cạnh Groundedness ≥ 0.80 | Đối soát §3.9 (P1 — thiếu Faithfulness); Biên bản §4 | ☐ |
| C08 | **6 chỉ số GenAI** (Groundedness, Precision…, gồm Faithfulness sau khi gộp) + cơ chế **temperature 0.0 + tolerance ±0.03** cho 2 lần chạy khác điểm | Task 4 §5.3; Đối soát §3.9; Biên bản §4 | ☐ |
| C09 | "6 đặc tính đánh giá AI" của Hùng và của Trang **đã gộp làm một** (không còn 2 bộ song song) | Biên bản §4 | ☐ |
| C10 | Outbox: **cần receiver ACK (HTTP 2xx)**, publisher idempotent, hai tenant K1/K2 có bảng chứng riêng | Portal — "Receipt triển khai 14/09 · Nét đạt" | ☐ |
| C11 | Vòng đời job vẽ đủ 4 trạng thái `queued / running / completed / failed`; **từ chối tiếp nhận xảy ra TRƯỚC khi có job** | Portal — "vòng đời tác vụ" (Lưu trữ V1 · 06) | ☐ |
| C12 | Phân biệt rõ **luồng artifact job (POST /v2/artifact-jobs)** với **`answer_change` kế thừa** — không trộn hai hợp đồng | Portal — "answer_change kế thừa" | ☐ |
| C13 | Human review có quyền lực thật: **reviewer kiểm soát candidate decision** (Law 9), người duyệt quyết định review/publish | Task 4 §2.2; Portal — "một lượt đánh giá" | ☐ |
| C14 | **Prompt injection** được thể hiện đúng góc độ của TI: quét mã độc/payload độc hại lọt vào **source code** (TI không có UI Chat) | Biên bản §3 | ☐ |
| C15 | Nhất quán với sơ đồ **luồng chức năng S01–S10** đang public (nét đứt = phần chưa có lượt đo live: Gateway/ToolIntent, outbox chờ ACK, Memory reuse) | Portal `/diagrams` (ưu tiên 1) | ☐ |

### CHECKLIST D — Sơ đồ của Trang · Chi tiết service & tool (S3)

| ID | Nội dung kiểm tra (Pass nếu = Đúng) | Nguồn đối chiếu | P/F |
| :--: | :--- | :--- | :--: |
| D01 | Mỗi thành phần ghi rõ **tên service AWS thật + region** (`ap-southeast-1` / `us-east-1`) — không có ô chung chung "database" hay "compute" | Portal — Hạ tầng 22/09; Task 1 | ☐ |
| D02 | Cổng & giao thức đúng hiện trạng: CloudFront → API `:8000` và portal `:8001` trên **cùng một EC2** (hai container không chung ổ: `/app/data` chỉ gắn ở API và portal) | Portal — Hạ tầng (đo 22/09, nguồn ghim `8a61cf66`) | ☐ |
| D03 | Tool versions & **nơi lưu trữ script K6/Playwright được chỉ rõ** (điểm mở của biên bản §3: "cần làm rõ nơi lưu trữ các kịch bản K6 và Playwright") | Biên bản §3 | ☐ |
| D04 | 4 image **ECR pre-baked** + digest đúng; đường giao hàng `GitHub Actions → ECR → SSM` vẽ **nét đứt** (đường khai báo, chưa đo live) | Portal — Hạ tầng; Đối soát §3.3 | ☐ |
| D05 | Nền tảng thực thi phản ánh chốt **Fargate task-per-job** — nếu còn vẽ CodeBuild/Lambda cho runner thì Fail (cold start 45–90s + tải Chromium mỗi lần = lãng phí) | Đối soát §3.3 (P1); Biên bản §2 | ☐ |
| D06 | **Semgrep + Trivy** được đặt đúng chỗ S02 và bao quát đúng hướng Testing (biên bản: hiện chỉ tập trung scan source code, chưa đáp ứng hướng scan của Testing) — nếu chưa đủ thì phải ghi là gap, không được vẽ như đã làm | Biên bản §3; Task 2 §6.0 (D5a) | ☐ |
| D07 | Sơ đồ **Bedrock thuần đã được chẻ lại cho rõ** (biên bản §4 yêu cầu) — tách bạch: model Bedrock / AgentCore Harness dùng model nào / pipeline Bedrock Evaluations | Biên bản §4; Portal — Cách đọc | ☐ |
| D08 | AgentCore **Harness / Gateway / Runtime** đúng version theo receipt được trích (Runtime 18 @ receipt 14/09) và ghi rõ receipt nào | Receipt 14/09; `/evidence` | ☐ |
| D09 | Không có credential/secret nào vẽ trong prompt hay ToolIntent; **secrets do backend giữ, tool chỉ nhận URL đã binding** (Law 12–15) | Task 2 §4 (mục 10.1); Task 4 §2.2 | ☐ |
| D10 | Evidence path đầy đủ: **normalize + hash SHA-256 (Law 16)** → `raw_result_s3_uri` → S08; **không** vẽ raw result quay lại storage trên EC2 | Task 4 §5.4; Đối soát §3.4 | ☐ |
| D11 | DB testing vẽ đủ **hai nhánh**: SQL = Aurora Serverless v2 clone + Flyway; NoSQL = DynamoDB Local/Ephemeral (TTL 1h + cleanup) | Đối soát §3.5 | ☐ |
| D12 | Backup/backup scope **trung thực**: versioning ≠ WORM; phạm vi sao lưu chỉ backend `ap-southeast-1`; 4 lệnh backup trả kết quả rỗng = "đã hỏi và được trả lời là không có" | Portal — Hạ tầng | ☐ |
| D13 | Không vẽ service đã EOL (**CodeGuru Security** — thay bằng Semgrep + Trivy hoặc Amazon Inspector) | Đối soát §3.1 (P0) | ☐ |
| D14 | Mọi con số (dung lượng, giá, latency, %) kèm **nguồn + ngày** và nhãn `OBSERVED/INFERRED/CANDIDATE` | Đối soát §3.9; Task 2 Phụ lục A | ☐ |
| D15 | Nhất quán với sơ đồ **hạ tầng đang public** và với 2 sơ đồ của Hoàng/Hùng (cùng tên service, cùng port, cùng region) | Portal `/diagrams` + A11 | ☐ |

### CHECKLIST E — Đối chiếu evidence (Vòng 2)

> Mỗi mục **bắt buộc** có Evidence Pack: `URL + timestamp + kết quả/screenshot`.

| ID | Nội dung kiểm tra (Pass nếu = Khớp) | Nguồn đối chiếu | P/F |
| :--: | :--- | :--- | :--: |
| E01 | Truy cập `/diagrams` live tại ngày review, ghi lại **tiêu đề + ngày đo + nhóm** của từng sơ đồ → khớp claim của 3 draft | `https://d1tibdarzmw3jq.cloudfront.net/diagrams` | ☐ |
| E02 | Đọc `/evidence` → **revision đang phục vụ NGAY BÂY GIỜ** khớp (hoặc được giải thích đúng) với tham chiếu trên draft | `/evidence` | ☐ |
| E03 | Claim "V2 / receipt 14/09" đối chiếu đúng **source `fbdd8dfc…`, image `efabf57dcd8c`, AgentCore Runtime 18** và ghi rõ đây là **mốc lịch sử**, không phải bản đang chạy | Receipt 14/09 (Portal — Receipt) | ☐ |
| E04 | Claim liên quan 23/09 đối chiếu receipt `0409cb5a` (ảnh ECR `sha256:b717be38…`) — nhớ: receipt này **chỉ đọc lại 4 tuyến công khai, không đo lại kiến trúc** | Portal — Lượt triển khai 23/09 | ☐ |
| E05 | Tham chiếu commit: nếu draft nói "theo lượt đo 22/09" thì phải là `b55fed16…` (ảnh `58abe918…`); **không được trộn** với lượt `7dca7c0a` / `253e702e` (CI run `35694235318`) cùng ngày | Portal — Lần đo kiến trúc 22/09 | ☐ |
| E06 | Hợp đồng API: `OpenAPI 3.1.0 · info.version 1.0.0`, policy `artifact-v1` — đường dẫn **v1 `/v1/testing/changes` LIVE nhưng FROZEN**, job mới là `POST /v2/artifact-jobs` | Portal — API; `/api` | ☐ |
| E07 | Các claim "đã đo được / nét liền" được chứng minh bằng **gọi thật**: `GET /docs · /runs · /knowledge · /diagrams` trả 200 (portal ghi rõ 200 toàn bộ, `diagrams` 280 từ) | Gọi API trực tiếp | ☐ |
| E08 | Claim về mã lỗi (`422` đủ điều kiện; `401 AUTHORITY_INSUFFICIENT` cho caller thiếu quyền) được **thử lại** đúng kịch bản | Gọi API + portal — Nét liên = đã đo được hơn này | ☐ |
| E09 | Trạng thái nghiệm thu: **#97 còn mở** → bất kỳ đâu viết "đã nghiệm thu/đủ bằng chứng" = **Fail** | Portal — Theo dõi nghiệm thu #97 | ☐ |
| E10 | Mọi `runtime_binding` / Memory reuse / outbox ACK vẽ **nét đứt + nhãn UNVERIFIED** đúng hiện trạng | Portal — Nét đứt là phần chưa có lượt đo | ☐ |

### CHECKLIST F — 10 điểm nóng của Biên bản 23/09/2026 (áp cho cả 3 sơ đồ)

| ID | Điểm nóng (Biên bản) | Câu hỏi Pass/Fail | P/F |
| :--: | :--- | :--- | :--: |
| F01 | **Quản lý chi phí "hơi ảo"** (§1) | Mọi con số chi phí trên sơ đồ đã được tính lại theo đơn giá thật + gắn nhãn `CANDIDATE` (chưa `OBSERVED`) chưa? | ☐ |
| F02 | **Tổng kết báo cáo chưa đúng** (§1) | Các nhãn/tiêu đề/summary trên sơ đồ đã được đọc lại từng dòng cho đúng chưa? | ☐ |
| F03 | **CodeBuild/Lambda — setup Playwright mỗi lần có lãng phí?** (§2) | Sơ đồ đã nêu rõ *lý do chọn* nền tảng (Fargate task-per-job + image pre-baked) thay vì để ngỏ câu hỏi chưa? | ☐ |
| F04 | **EC2 storage ngộp vì evidence** (§2) | Luồng evidence có đi thẳng ra object storage (S3 + Object Lock) thay vì chồng lên EC2 không? Dung lượng/retention có ghi rõ (hoặc ghi "chưa đo")? | ☐ |
| F05 | **NoSQL rollback khi test pass/fail** (§2) | Sơ đồ có phương án NoSQL (DynamoDB ephemeral + TTL + cleanup) chứ không chỉ Aurora clone cho SQL? | ☐ |
| F06 | **Orchestrator không điều phối được, Tenant Binding mới quyết định** (§3) | Luồng điều phối thể hiện đúng `ImpactSet ∩ TargetBinding` và input của từng runner chưa? | ☐ |
| F07 | **Nơi lưu script K6/Playwright chưa rõ** (§3) | Sơ đồ chi tiết (Trang) đã chỉ đích danh nơi lưu + cách version các script này chưa? | ☐ |
| F08 | **K6 dễ bị AWS ban / "tự DoS người nhà"** (§3) | Có ranh giới/ cảnh báo vùng chạy load test và bước xin phép DevOps chưa? | ☐ |
| F09 | **Semgrep/Trivy chưa đáp ứng hướng scan của Testing; Prompt injection = quét payload trong source** (§3) | Sơ đồ có thể hiện đúng phạm vi scan hiện tại **và** gap cần đóng, đúng góc độ prompt-injection của TI không? | ☐ |
| F10 | **GenAI: evidence chưa đủ thuyết phục, 2 lần chạy 2 kết quả, thiếu Faithfulness, gộp 6 đặc tính Hùng+Trang, sơ đồ Bedrock chẻ lại** (§4) | Sơ đồ workflow/chi tiết đã: gộp 6 đặc tính; thêm Faithfulness; nêu cơ chế ổn định điểm (temp 0, tolerance ±0.03); chẻ rõ pipeline Bedrock Evaluations/AgentCore chưa? | ☐ |

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

## 9. Tiêu chí nghiệm thu tổng thể (Exit Criteria)

Một sơ đồ được đóng dấu **ĐẠT** khi **tất cả** điều kiện sau đúng:

- [ ] Checklist A (15 mục) = 100% Pass.
- [ ] Checklist B hoặc C hoặc D (15 mục, theo chủ sơ đồ) = 100% Pass.
- [ ] Checklist E (10 mục) = 100% Pass — mọi claim đã có Evidence Pack.
- [ ] Checklist F (10 mục) = 100% Pass — 10 điểm nóng biên bản đã đóng hoặc đã ghi rõ "gap + kế hoạch" trên sơ đồ.
- [ ] Defect Log không còn mục `OPEN`.
- [ ] Đã qua **2 vòng** (draft → góp ý → re-review) và Nghĩa ký kết luận.

**Kết luận nghiệm thu 3 sơ đồ** ghi vào 1 dòng biên bản báo cáo (G5): trạng thái từng sơ đồ, số mục Pass/Fail, link Defect Log.

> **Ghi chú trung thực:** Đạt checklist này = *sơ đồ đáng tin làm căn cứ*. **Không** đồng nghĩa hệ thống đã được kiểm thử hay đã nghiệm thu (#97 vẫn có thể còn mở) — đúng tinh thần "bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy".

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

**Tổng số mục:** 15 + 15 + 15 + 15 + 10 + 10 = **80 mục** (mỗi sơ đồ phải qua **60 mục**: A + nhóm riêng + E + F).

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










