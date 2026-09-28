# FEEDBACK: Đánh giá Báo cáo Đối soát của Gemini & Khuyến nghị Hành động cho Task 1–4

| Thuộc tính | Giá trị |
| --- | --- |
| Đối tượng review | `BAO_CAO_DOI_SOAT_MISMATCH_VA_DONG_BO_TI.md` (do Gemini sinh, 25/09/2026) |
| Đối chiếu với | Biên bản họp TI 23/09/2026 · Task 1 (Trang) · Task 2 (Hoàng, v0.2) · Task 3 (Nghĩa, v2.0.0) · Task 4 (Hùng, v1.2.0) |
| Phương pháp | Đọc trực tiếp từng file gốc + grep từ khóa đối chiếu + web search xác minh 2 claim công nghệ (CodeGuru, Bedrock model) |
| Kết luận 1 câu | Báo cáo Gemini **đúng hướng ở phần kỹ thuật nhưng lạc hậu ~40% so với thực tế team đã làm**, và **bỏ lọt lỗ hổng nặng nhất**: kiến trúc security của Task 3 & Task 4 đang dựa vào một dịch vụ AWS đã ngừng hoạt động hoàn toàn. |

---

## 0. Tóm tắt 1 phút

Báo cáo Gemini có giá trị thật (đặc biệt là gợi ý Smart Dispatching Matrix, NoSQL gap, Faithfulness), nhưng có **2 lỗi hệ thống** khiến nó không thể dùng làm căn cứ "chốt" ngay:

1. **Dùng dữ liệu cũ hơn báo cáo tự khai**: Gemini tự ghi nguồn là "Task 2 v0.2" nhưng không phản ánh đúng nội dung v0.2 — Hoàng đã tự sửa 3/10 điểm "mismatch" (bỏ Network Firewall, thêm domain Database D2.b, vẽ lại sơ đồ dispatch ngang hàng) **trước khi** Gemini viết báo cáo này, và Hùng (Task 4 v1.2.0) cũng đã đồng bộ theo. Gemini trình bày lại các điểm này như "phát hiện mới" và giao lại thành action-item cho Hoàng — tức là **giao lại việc đã xong**.
2. **Bỏ lọt lỗ hổng nghiêm trọng nhất**: `Amazon CodeGuru Security` — dịch vụ mà Task 3 và Task 4 dùng làm **Tầng 0** của kiến trúc bảo mật — đã bị AWS khai tử hoàn toàn từ 20/11/2025 (chưa từng ra khỏi bản Preview). Đây không phải là "lệch pha giữa 2 tài liệu" như Gemini mô tả nhẹ ở Mismatch 7 — đây là **một thành phần kiến trúc trỏ vào dịch vụ không còn tồn tại**. Chi tiết ở Mục 3.1.

Phần dưới đây trả lời 3 câu hỏi bạn hỏi: **(1)** đúng/sai của từng điểm trong báo cáo Gemini so với đề xuất thật của từng người, **(2)** feedback cụ thể cho từng Task, **(3)** lỗ hổng/công nghệ lỗi thời Gemini chưa vá.

---

## 1. Đối soát từng điểm "Mismatch" của Gemini với thực tế 4 Task

Chú thích cột **Verdict**: ✅ Đúng & còn giá trị · ⚠️ Đúng nhưng đã bị fix một phần (báo cáo lạc hậu) · 🔴 Overclaim/thiếu nuance · 🆕 Gemini bỏ sót phần quan trọng nhất của vấn đề.

| # | Mismatch của Gemini | Verdict | Bằng chứng đối chiếu trực tiếp |
| :---: | :--- | :---: | :--- |
| 1 | Execution platform: CodeBuild/Synthetics (Task 1) vs Fargate (Task 2) | ✅ | Đúng 100%. Task 1 §2.2 vẫn vẽ `S07 → CloudWatch Synthetics` cho UI và CodeBuild cho API/Security; Task 2 đã chọn Fargate task-per-job từ v0.1. Đây là mismatch thật, Trang cần cập nhật. |
| 2 | Evidence storage / nguy cơ ngộp EC2 | ⚠️ | **Đã được giải quyết một phần**: sơ đồ §8 của Task 2 (v0.2) đã vẽ raw result đi **thẳng từ Fargate task → S3 Object Lock**, không qua Job Controller/EC2 (`SCNA & BRF & LOD & SCNB & DBT --> OBJ`). Cái còn thiếu thật là: Task 1 vẫn giả định backend chạy trên EC2 + EBS gp3 (§2.1) và chưa cập nhật theo kiến trúc Direct-to-S3 này. Vấn đề không phải "chưa ai giải quyết" mà là "Trang chưa đọc/áp bản vẽ mới nhất của Hoàng". |
| 3 | NoSQL testing hoàn toàn thiếu | ✅ | Xác nhận bằng grep: **0 lần** xuất hiện "NoSQL/DynamoDB/DocumentDB/MongoDB" trong Task 2, 3, 4 (Task 1 chỉ nhắc tên công cụ trong danh mục liệt kê ở Phần 1, không có trong phần lựa chọn Phần 2/3 dành cho TI). Đây là gap thật, nghiêm trọng, đúng như biên bản họp nêu. Đề xuất TTL/Ephemeral table của Gemini hợp lý về mặt kỹ thuật. |
| 4 | Orchestration chạy toàn bộ 6 runner dù PR chỉ đổi 1 phần | ✅ | Xác nhận: Task 1 dòng "S07 --> R_API / R_UI / R_DB / R_Perf" không có nhánh điều kiện. Task 4 dòng 191: "Risk Tier CRITICAL → Chạy TẤT CẢ 6 domain runners" — xác nhận đúng. **Nhưng Gemini gọi giải pháp này là "Smart Dispatching Matrix" như một phát minh mới** — thực ra đây chính là năng lực **S03 Impact Engine** đã có sẵn trong tài liệu kiến trúc gốc (`ImpactSet` là canonical output của S03). Xem Mục 3.4. |
| 5 | Nơi lưu Test Script (Playwright/K6) chưa rõ | ✅ | Đúng, không task nào chốt vị trí lưu trữ kịch bản tĩnh vs kịch bản AI sinh. Đề xuất "2-tier storage" của Gemini hợp lý, nhưng cần Product Architect duyệt vì đụng vào ranh giới S06/S08 (candidate vs published knowledge). |
| 6 | K6 tự DoS / AWS ban | ✅ | Đúng là chưa ai đề xuất private routing. Nhưng lưu ý: Task 4 R03 **đã có** risk register cho self-DoS (`MAX_VUS=100, MAX_DURATION=300s, server-enforced`) — Gemini đề xuất siết chặt hơn (`MAX_VUS=20/60s`) là hợp lý nhưng nên trình bày là "siết lại ngưỡng đã có" thay vì "đề xuất mới hoàn toàn". |
| 7 | Security & Prompt Injection trong hệ headless | 🆕 | Đây là điểm Gemini **đánh nhẹ nhất trong khi lẽ ra phải là nghiêm trọng nhất**. Gemini chỉ nói "Task 4 vẫn còn liệt kê CodeGuru Security trong bảng In-Scope" như một lỗi đặt tên nhỏ (mức 🟡 Trung bình). Thực tế: `Amazon CodeGuru Security` là dịch vụ **đã ngừng hỗ trợ hoàn toàn từ 20/11/2025, chưa từng rời bản Preview** — và nó không chỉ nằm trong 1 bảng của Task 4, mà là **Tầng 0** của toàn bộ mô hình Hybrid Defense 4 tầng (Task 4 §4.2.5, đổi trong changelog v1.2.0 mục 3) và cũng được Task 3 đưa vào Phụ lục công cụ Security. Xem phân tích đầy đủ ở Mục 3.1. |
| 8 | Non-determinism GenAI (chạy 2 lần ra 2 điểm) | ✅ | Đúng: grep xác nhận `temperature`/`top_p` không xuất hiện trong bất kỳ file nào. Giải pháp Gemini đề xuất (temperature=0.0 + cache theo SHA-256 + tolerance band) đúng hướng, nhưng **temperature=0.0 không đảm bảo determinism tuyệt đối** trên hạ tầng inference GPU quy mô lớn (batching, MoE routing vẫn có thể gây lệch nhỏ) — Gemini viết "triệt tiêu triệt để" là overclaim, nên sửa thành "giảm mạnh, không loại bỏ 100%" và giữ nguyên phần cơ chế tolerance-band + median-of-3 làm lớp phòng thủ thứ hai. |
| 9 | Thiếu Faithfulness trong HOLD, chưa gộp 6 đặc tính | ✅ | Đúng, xác nhận: Task 4 Mục 6.2 (Exit Criteria) chỉ có `GroundednessScore < 0.80`, không có Faithfulness. Bảng gộp Gemini đề xuất (6 trục Trang × 6 đặc tính Hùng) là ý tưởng hợp lý và nên giữ lại. |
| 10 | Cost "ảo", cần tính lại | 🔴 | Đúng với Task 1 & Task 3 (số liệu quá lạc quan, xác nhận: Task 1 tính CodeBuild $0.0008/lượt, Synthetics $0.0012/lượt — quá thấp). **Sai khi áp dụng đều cho cả team**: Task 2 (v0.2) đã tự tính lại bằng đơn giá Fargate chính thức, có công thức, có ngày tra giá, có nhãn `OBSERVED/INFERRED/CANDIDATE`, và Task 4 §4.2.5 dòng 381 **đã dẫn số của Task 2** ("giảm từ ~$320/tháng xuống <$15/tháng"). Trớ trêu là **bảng chi phí của chính Gemini bỏ hẳn kỷ luật OBSERVED/INFERRED/CANDIDATE** mà Task 2 đã lập ra — tức là báo cáo "sửa lỗi cost ảo" lại viết theo cách kém nghiêm ngặt hơn tài liệu nó đang phê bình. Xem Mục 3.6. |

**Tóm lại**: 7/10 điểm là phát hiện đúng và còn giá trị (1, 3, 4, 5, 6, 8, 9). 2/10 điểm đúng nhưng đã lạc hậu vì team tự fix trước rồi (2, 10 một phần). 1/10 điểm (7 — security) đúng nhưng bị đánh giá thấp hơn mức độ nghiêm trọng thật.

---

## 2. Feedback riêng cho từng thành viên / Task

### 2.1. Trang — Task 1 (Tool & Framework)

**Điểm mạnh**: Bảng so sánh ưu/nhược từng nhóm công cụ (Phần 3) rất tốt, có lý do loại/chọn rõ ràng, đúng tinh thần "problem-first" — đặc biệt phần đánh giá CodeGuru EOL (dòng 320–321) là phát hiện đúng và có giá trị mà chính Gemini/Task 3/Task 4 chưa cập nhật theo (xem Mục 3.1).

**Cần sửa (theo thứ tự ưu tiên)**:
1. **Đồng bộ với quyết định D2 = Fargate của Task 2** (§2.2, §3.7): thay `AWS CodeBuild + CloudWatch Synthetics` bằng 4 image ECR mà Gemini/Task 2 đã thống nhất (`ti-runner-api-fuzz`, `ti-runner-ui-web`, `ti-runner-perf`, `ti-runner-security`). Đây là mismatch thật, cần sửa.
2. **Xoá giả định EC2/EBS gp3 lưu evidence** (§2.1): cập nhật theo luồng Direct-to-S3 mà Task 2 §8 đã vẽ — không phải đề xuất "mới", chỉ là đọc lại bản vẽ mới nhất của Hoàng.
3. **Bổ sung mục NoSQL testing** (DynamoDB Local/Ephemeral + TTL cleanup, hoặc DocumentDB scoped-per-job) — gap có thật, chưa ai làm.
4. **Cập nhật lại Bảng 3.7** với công thức của Task 2 (đơn giá Fargate chính thức, có ngày tra) thay vì số liệu tự đoán (CodeBuild $0.0008, Synthetics $0.0012).
5. **Tự mâu thuẫn nội bộ cần soát lại**: mục 3.5 đã đúng khi loại OWASP ZAP ("Không chọn làm lõi"), nhưng Task 2 (v0.2) và Task 3 đã đưa ZAP/nuclei vào làm D5b (Wave 3, chỉ chạy khi có Staging URL). Trang cần quyết định lại: giữ nguyên "không chọn" (và nói rõ lý do khác với Hoàng/Nghĩa), hoặc cập nhật theo hướng "chọn có điều kiện" để 3 tài liệu khớp nhau.

### 2.2. Hoàng — Task 2 (Architecture)

**Điểm mạnh**: Đây là tài liệu có kỷ luật cao nhất trong 4 task — dùng nhãn `OBSERVED/INFERRED/CANDIDATE`, ghi rõ falsifiability, có ADR + decision log + trade-off table, đã tự chạy một vòng review (v0.1 → v0.2 theo feedback mentor 22/09) và tự sửa đúng 3 trong 4 điểm mentor yêu cầu. Đây là **chuẩn nên áp dụng cho cả 3 task còn lại**, không phải ngược lại.

**Cần làm tiếp (không phải "sửa lỗi", mà là hoàn thiện)**:
1. **Chạy Spike P4** như đã lên kế hoạch ở §11 — mọi số liệu hiện tại vẫn là `CANDIDATE`, cần hoá đơn thật để chuyển `OBSERVED`. Đây là việc quan trọng nhất, không phải việc Gemini gợi ý (network firewall, direct-to-S3) vì 2 việc đó **đã làm rồi**.
2. **Đối soát số $22/tháng (Task 2) vs "<$15/tháng" (Task 4 dòng 381)** cho chi phí VPC Endpoints — Task 2 tự nhận số của mentor "hơi thấp" nhưng Task 4 vẫn đang trích số cũ. Cần 1 số chốt duy nhất trước khi ký ADR-08.
3. **Ký ADR-01 (Q1=Executor) và ADR-03 (D2=Fargate)** — đây vẫn đang ⏳ chờ ký, là one-way door thật. Nên tách rõ trong mọi trao đổi nhóm: "đề xuất kỹ thuật" (đã đủ chín) khác với "quyết định có thẩm quyền" (chưa có chữ ký Product Architect). Gemini's report vô tình viết như thể mọi thứ đã chốt — cần cẩn thận không lấy đó làm căn cứ triển khai trước khi Tan.Thai ký.
4. **Xác nhận lại D2.b (Database Testing) với Task 1 & Task 3** — bản thân Task 2 v0.2 ghi rõ "⚠️ chờ xác nhận lại", nên đây vẫn là việc mở, không phải đã xong hoàn toàn.

### 2.3. Nghĩa — Task 3 (AI Model & Prompting)

**Điểm mạnh**: Model tiering 3 tầng (Haiku/Sonnet/Opus theo độ phức tạp task) là thiết kế hợp lý, tiết kiệm chi phí thật (không giống ước tính "vài cent" ngây thơ của Task 1). Capability Manifest mẫu tuân đúng khung 24 laws.

**Cần sửa — mức độ khẩn cấp cao**:
1. **Gỡ `Amazon CodeGuru Security` khỏi Phụ lục công cụ Security** (dòng 1039) và khỏi mọi sơ đồ nhắc đến nó (dòng 163). Đây không phải lỗi chính tả — dịch vụ này đã ngừng hoạt động từ 20/11/2025. Xem Mục 3.1 để biết thay bằng gì.
2. **Bổ sung cơ chế kiểm soát non-determinism**: khai báo `temperature` và `top_p` trong Capability Manifest (hiện 3 manifest mẫu ở Phần 5 hoàn toàn không có 2 field này).
3. **Đối soát version**: file hiện tại tự ghi "v2.0.0" ở đầu trang, nhưng Task 4 (dòng 668) nói "Task 3 đã hoàn thành đồng bộ v2.1.0". Cần xác nhận: nếu v2.1.0 đã tồn tại, phải là bản chính thức lưu hành (không phải bản v2.0.0 này); nếu chưa, Task 4 cần sửa lại changelog cho đúng, tránh cả team làm việc trên 2 "sự thật" khác nhau về trạng thái đồng bộ.
4. **Bổ sung phần NoSQL** vào schema `DATABASE_CANDIDATE_V1` (thêm field `engine_type`) như Gemini đề xuất — hợp lý, nên làm.

### 2.4. Hùng — Task 4 (Testing Strategy & Evaluation)

**Điểm mạnh**: Vận dụng 3 tầng ISTQB (CTFL/CT-AI/CT-GenAI) có hệ thống, Risk Register đầy đủ, đã chủ động đồng bộ với feedback mentor của Task 2 (dòng 381: quyết định Private Subnet đã khớp với Task 2 v0.2) — cho thấy Hùng đang theo dõi Task 2 sát hơn Gemini.

**Cần sửa**:
1. **Bổ sung `FaithfulnessScore < 0.85` vào điều kiện HOLD ở Mục 6.2** — hiện chỉ có Groundedness. Đây là gap thật, đúng như biên bản họp và Gemini nêu.
2. **Gỡ vai trò lõi của CodeGuru Security khỏi Hybrid Defense 4 tầng (§4.2.5, Tầng 0)** — mức độ ưu tiên cao nhất trong toàn bộ 4 task, vì đây là runner **không thể chạy được** trên thực tế. Cần thiết kế lại Tầng 0 (xem đề xuất ở Mục 3.1).
3. **Làm rõ lại quy tắc dispatch ở §4.1**: thay "CRITICAL → chạy TẤT CẢ 6 domain runners" bằng công thức giao của Impact Domain ∩ Tenant Enabled ∩ Risk Tier — nhưng ghi rõ đây là hiện thực hoá capability **S03 Impact Engine đã có sẵn** trong kiến trúc gốc, cần Product Architect xác nhận cách map, không tự quyết trong Task 4.
4. **Bổ sung domain DAST (ZAP/nuclei, Wave 3)** vào bảng In-Scope/Out-of-Scope §3.1 — hiện Task 4 không nhắc đến DAST ở đâu cả, trong khi Task 2 và Task 3 đã có (mismatch nội bộ giữa 3 tài liệu, Gemini không phát hiện).
5. **Gộp bảng 6 đặc tính CT-AI với 6 trục công nghệ của Trang** như đề xuất — hợp lý, nên làm sau khi Trang cập nhật Task 1 theo Mục 2.1 trên.

---

## 3. Lỗ hổng & Công nghệ lỗi thời (tính đến 09/2026) mà báo cáo Gemini chưa vá

### 3.1. 🔴 Nghiêm trọng nhất: `Amazon CodeGuru Security` không còn tồn tại

Đã xác minh qua tìm kiếm thực tế (không chỉ dựa vào Task 1):

| Sản phẩm | Trạng thái thật (đã xác minh) | Task nào đang dùng nó ra sao |
| --- | --- | --- |
| **Amazon CodeGuru Security** (Preview) | **Ngừng hỗ trợ hoàn toàn từ 20/11/2025** — chưa từng đạt GA, bị AWS khai tử thẳng từ bản Preview. Không có phiên bản "duy trì cho khách hàng cũ". | Task 3 (Phụ lục công cụ) và Task 4 (Tầng 0 của Hybrid Defense 4 tầng, dùng làm pre-runner S02→S04) đang coi đây là dịch vụ đang hoạt động. |
| **Amazon CodeGuru Reviewer** | Chuyển sang **Maintenance Mode từ 07/11/2025**: khách hàng mới **không thể** tạo repository association mới; khách cũ vẫn dùng được association đã có. AWS khuyến nghị khách mới dùng Amazon Q Developer (review) + Amazon Inspector (security scan). | Task 1 gộp chung với Profiler và nói "EOL từ 20/11/2025" — **sai ngày và sai mức độ** (đây là ngày EOL của CodeGuru *Security*, không phải Reviewer). |
| **Amazon CodeGuru Profiler** | **Vẫn hoạt động bình thường**, không nằm trong diện khai tử — có trang pricing được cập nhật gần đây. | Task 1 liệt kê Profiler là "đã EOL" — **sai**, Profiler không bị ảnh hưởng. |

**Vì sao đây nghiêm trọng hơn mức Gemini đánh giá (🟡 Trung bình)**: Task 4 dùng chính "Amazon CodeGuru Security" (không phải Reviewer/Profiler) làm **Tầng 0** của mô hình bảo mật — tức là lớp chạy sớm nhất, cung cấp risk score đầu vào cho S04 Risk Engine, ảnh hưởng đến việc gán Risk Tier cho mọi PR. Đây không phải "tên gọi cần thống nhất" mà là **một node kiến trúc không thể triển khai được trên thực tế** — nếu không sửa, khi build sẽ phát hiện ra Tầng 0 không gọi được API nào cả.

**Khuyến nghị thay thế** (để Nghĩa & Hùng áp dụng):
- Thay Tầng 0 bằng **Amazon Inspector** (code security scanning, vẫn hoạt động, được AWS chỉ định thay CodeGuru Security) kết hợp **Amazon Q Developer** cho phần review tự động nếu team chấp nhận mô hình pricing theo user ($19/dev/tháng — điều mà Task 1 đã cân nhắc và loại vì không hợp cơ chế pay-as-you-go).
- Hoặc đơn giản hoá: gộp Tầng 0 vào Tầng 1 (Semgrep + Trivy + Gitleaks đã có sẵn, chạy tại S02), bỏ hẳn khái niệm "pre-runner AWS Native ML" — giảm 1 tầng phức tạp, tiết kiệm cả effort và tiền.
- Việc này nên gắn cờ đỏ và đưa vào buổi "connect lại" mà biên bản họp yêu cầu, vì nó ảnh hưởng cả Task 3 và Task 4.

### 3.2. ⚠️ Cần re-verify: tên model Claude trên Bedrock

Task 3 chốt bảng model theo `Claude Opus 5 / Sonnet 5 / Haiku 4.5`. Tra cứu hiện tại cho thấy dòng Opus của Anthropic đã có bản kế tiếp (`Opus 5.5`) sau `Opus 5`. Vì team đang tính chi phí/token theo bảng giá cố định của một phiên bản cụ thể, **nên re-verify trực tiếp trên AWS Bedrock Model Access console** trước khi build, thay vì tin vào bảng giá đã ghi trong tài liệu — model ID/giá trên Bedrock có thể đã đổi so với lúc Nghĩa viết Task 3 (22/09/2026). Đây không phải lỗi chắc chắn, chỉ là điểm rủi ro "hàng lỗi thời nhanh" cần một dòng ghi chú "xác minh lại trước khi code" trong tài liệu.

### 3.3. ⚠️ Version drift / kiểm soát tài liệu

Task 4 (dòng 668) khẳng định "Task 3 đã hoàn thành đồng bộ v2.1.0 — loại bỏ Hurl, Testcontainers, Pixelmatch". Nhưng file Task 3 hiện có lại tự ghi header là **v2.0.0**. Hai khả năng: (a) v2.1.0 có tồn tại nhưng chưa được đưa vào bộ tài liệu đối soát lần này — nếu vậy, **toàn bộ phân tích của Gemini về Task 3 đang dựa trên bản cũ hơn cả những gì Task 4 đang tưởng là đã đồng bộ**; hoặc (b) Task 4 ghi nhận sai. Cả hai trường hợp đều cần chốt lại 1 nguồn sự thật duy nhất (single source of truth) cho version của mỗi Task trước buổi connect, nếu không nhóm sẽ tiếp tục review nhau trên các bản khác nhau.

### 3.4. 🆕 "Smart Dispatching Matrix" không phải phát minh mới

Gemini trình bày công thức `Runners = Impacted Domains ∩ Tenant Enabled Domains ∩ Risk Tier` như một giải pháp mới tự nghĩ ra. Thực chất, tài liệu kiến trúc gốc của TI (`Testing_Intelligence_Architecture_Overview...md`, mục 7.1) đã định nghĩa sẵn **S03 Impact Engine** với canonical output là `ImpactSet`, và **S01 Target Registry** sở hữu `TargetBinding`. Nói cách khác, phần "phát hiện" và "giải pháp" ở Mismatch 4 chỉ là: **đội chưa nối S03/S01 vào quyết định dispatch ở S07** — một việc hiện thực hoá (implementation gap), không phải một thiết kế mới cần sáng tạo. Hệ quả thực tế: việc này chạm vào canonical capability đã được đặc tả ở tầng kiến trúc chung, nên **cần Product Architect (Tan.Thai) xác nhận cách map** trước khi Hùng/Hoàng tự triển khai riêng lẻ trong Task 4/Task 2, tránh mỗi người làm một kiểu rồi lại phải đối soát lần nữa.

### 3.5. Security D5a/D5b (SAST/DAST) vẫn chưa khớp đều 4 tài liệu

Gemini gộp việc này vào Mismatch 7 nhưng không chỉ ra rõ: **Task 1 chủ động loại OWASP ZAP** ("Không chọn làm lõi", §3.5), trong khi **Task 2 (v0.2, D5b)** và **Task 3 (Phụ lục, Wave 3)** đã đưa ZAP/nuclei vào làm DAST runner có điều kiện (chỉ chạy khi có Staging URL sống) — còn **Task 4 không đề cập ZAP/nuclei/DAST ở đâu cả** trong bảng scope. Đây là 3 trạng thái khác nhau trên 4 tài liệu, cần chốt 1 lần duy nhất.

### 3.6. Bảng chi phí của Gemini tự làm giảm kỷ luật của team

Task 2 đã lập chuẩn: mọi số liệu chưa có hoá đơn thật phải mang nhãn `OBSERVED/INFERRED/CANDIDATE` và nguồn tra giá kèm ngày. Bảng "Realistic Cost Model 2026" của Gemini (Mục 3.10) đưa ra số liệu như sự thật tuyệt đối (`$0.08–$0.13/job`, `$80–105/tháng`) mà không nhãn, không nguồn, không ngày tra giá — tức là mắc lại đúng lỗi mà biên bản họp phê bình ("cost hơi ảo"), chỉ khác là số nghe "có vẻ hợp lý hơn". Khuyến nghị: khi đưa số liệu này vào tài liệu chính thức, phải gắn nhãn `CANDIDATE` và chờ Spike P4 của Hoàng xác nhận, không dùng trực tiếp làm số chốt.

### 3.7. Governance: đừng biến "đề xuất kỹ thuật hợp lý" thành "đã có thẩm quyền ký"

Theo Authority Model của tài liệu kiến trúc gốc (mục 2.1: Product Architect giữ quyền Evaluation plan/Work Package; XBrain giữ quyền publication), các quyết định one-way-door như **Q1 = Executor** và **D2 = ECS Fargate** trong Task 2 hiện vẫn ở trạng thái ⏳ **chờ ký**. Báo cáo Gemini viết ở phần "Giải pháp chốt cho toàn team" theo văn phong khẳng định tuyệt đối ("Chuẩn hóa 100%... Loại bỏ hoàn toàn...") dễ khiến người đọc hiểu nhầm rằng các quyết định này đã có hiệu lực thi hành. Khuyến nghị: khi trình bày lại trong buổi connect, tách rõ 2 cột "Đề xuất kỹ thuật (đã chín, nên làm)" và "Cần chữ ký Product Architect trước khi triển khai" cho từng ADR.

### 3.8. Việc chưa ai đụng tới: ý kiến team Data cho Aurora Serverless Clone

Biên bản họp (mục 2) ghi rõ: "*Đang dùng Amazon Aurora Serverless v2 để clone database cho SQL (**cần tham khảo thêm ý kiến của team Data**)*". Cả 4 Task và cả báo cáo Gemini đều **không có bất kỳ action item nào giao việc trao đổi với team Data** — hiện tại quyết định Aurora Clone đang được xem là đã chốt kỹ thuật (Task 1, 2, 3 đều đồng thuận), nhưng chưa có xác nhận từ phía chủ sở hữu dữ liệu. Cần bổ sung 1 action item riêng, không gộp vào việc kỹ thuật.

---

## 4. Bảng hành động tổng hợp đã hiệu chỉnh (dùng thay bảng Mục 4–5 của Gemini)

| Ưu tiên | Việc cần làm | Owner | Đã làm rồi? |
| :---: | --- | :---: | :---: |
| 🔴 P0 | Gỡ "Amazon CodeGuru Security" khỏi kiến trúc lõi (Task 3 Phụ lục + Task 4 Tầng 0); thay bằng Inspector/Q Developer hoặc gộp vào Tầng 1 | Nghĩa + Hùng | ❌ Chưa |
| 🔴 P0 | Chốt 1 phiên bản duy nhất cho Task 3 (v2.0.0 hay v2.1.0) trước khi review tiếp | Nghĩa + Hùng | ❌ Chưa |
| 🔴 P0 | Nối S03 Impact Engine + S01 TargetBinding vào quyết định dispatch S07 (không tự thiết kế song song ở Task 2 và Task 4) | Hùng + Hoàng, xác nhận bởi Tan.Thai | ❌ Chưa |
| 🟠 P1 | Trang cập nhật Task 1: Fargate thay CodeBuild/Synthetics, Direct-to-S3, bổ sung NoSQL, cập nhật Bảng 3.7 theo số của Task 2 | Trang | ❌ Chưa |
| 🟠 P1 | Hùng bổ sung Faithfulness vào HOLD (§6.2) + thêm domain DAST vào scope | Hùng | ❌ Chưa |
| 🟠 P1 | Nghĩa khai báo temperature/top_p vào Capability Manifest | Nghĩa | ❌ Chưa |
| 🟡 P2 | Hoàng chạy Spike P4, chốt số $22 vs <$15 cho D9, xin ký ADR-01/03/08 | Hoàng + Product Architect | ⏳ Đang làm (đã lên kế hoạch ở §11) |
| 🟡 P2 | Gộp bảng 6 đặc tính (Hùng) × 6 trục công nghệ (Trang) | Hùng + Trang | ❌ Chưa |
| 🟢 P3 | Action item riêng: xin ý kiến team Data về Aurora Serverless Clone | PM/Hoàng | ❌ Chưa — chưa ai giao |
| 🟢 P3 | Ghi chú "re-verify model Bedrock trước khi code" vào Task 3 | Nghĩa | ❌ Chưa |

---

## 5. Đề xuất cho buổi "connect lại" (theo yêu cầu Mục 1 biên bản họp)

1. Mở đầu bằng đúng 1 slide: "4 điểm Task 2 đã tự fix trước report — khỏi làm lại", để tránh mất thời gian họp vào việc đã xong.
2. Dành ưu tiên thời gian cho đúng 1 chủ đề: **CodeGuru Security chết dịch vụ** — vì nó chạm cả Task 3 và Task 4, cần quyết định thay thế ngay, không thể để "từng người tự sửa" vì sẽ lại lệch pha lần nữa.
3. Chốt 1 người giữ "bảng version" của cả 4 Task (số version + ngày + trạng thái sync) — hiện đang là nguồn gây mismatch âm thầm nhất (Mục 3.3), nặng hơn cả các mismatch kỹ thuật.
4. Product Architect (Tan.Thai) xác nhận cách map S03/S01 vào dispatch logic trước khi Hùng/Hoàng code riêng lẻ.
