# Xora Banking: Business Needs, Agentic Architecture, and Terminology Library Overview

**XORA BANKING — NHU CẦU KINH DOANH, KIẾN TRÚC AGENTIC VÀ THƯ VIỆN THUẬT NGỮ**

**Bản chuẩn hóa đề xuất v1 — 25/09/2026**

Tài liệu này thống nhất cách mô tả sản phẩm và thiết kế Xora cho ngân hàng, dùng chung cho SME, Product, Architecture, Data, Agentic Development, QA, DevOps/MLOps và Delivery.

Nội dung tổng hợp từ Banking Use Case Playbook, workbook phát triển–kiểm thử Xora và các phân tích về incident investigation, VTB, agents, memory, KB và XBrain. Các cấu trúc dưới đây là baseline thiết kế đề xuất; trạng thái sẵn sàng của từng năng lực phải được xác nhận bằng hồ sơ qualification tương ứng.

Các bảng định nghĩa trong tài liệu tạo thành **thư viện thuật ngữ dùng chung**. Thuật ngữ tiếng Anh được giữ làm tên tham chiếu trong thiết kế, code, contract và hồ sơ nghiệm thu.

---

1. **Mỗi use case phải bắt đầu từ một kết quả kinh doanh có thể xác minh**

Một use case đủ rõ cần trả lời được:

- Ai đang gặp vấn đề?
- Công việc hoặc quyết định nào bị ảnh hưởng?
- Hậu quả là gì: chậm, sai, thiếu, tốn công hay gián đoạn?
- Dữ liệu nào chứng minh vấn đề và kết quả?
- Ai có thẩm quyền xác nhận?
- Cải thiện được đo trên phạm vi và khoảng thời gian nào?

Ví dụ, yêu cầu “xây agent điều tra incident” cần được cụ thể hóa thành:

> Với một IT service được chọn, hệ thống thu thập evidence từ các nguồn được phép, đưa ra giả thuyết có căn cứ, nhận biết phần thiếu và hỗ trợ người vận hành xác minh nguyên nhân. Hiệu quả được đánh giá qua thời gian có evidence, độ đúng của kết luận và effort điều tra.

**Danh mục khởi đầu cho ngân hàng**

| Mã | Nhu cầu kinh doanh | Đầu ra cung cấp | Dữ liệu chính | Cách xác nhận giá trị |
| --- | --- | --- | --- | --- |
| **D1 — Data Platform & Data Quality** | Dữ liệu đến chậm, thiếu, sai hoặc khó đối soát | Pipeline và data product đáng tin cậy cho một domain | Nguồn nghiệp vụ, schema, mapping, business date, control totals | Đầy đủ, đúng hạn, đối soát đúng, giảm effort vận hành |
| **M1 — MLOps for Selected Models** | Triển khai và vận hành model còn thủ công | Quản lý phiên bản, deployment, monitoring và rollback cho model được chọn | Model artifact, feature contract, prediction, evaluation, deployment logs | Tái lập deployment, phát hiện lỗi, rollback và đáp ứng yêu cầu vận hành |
| **G1 — Banking Knowledge Assistant** | Nhân viên khó tìm quy trình đúng và còn hiệu lực | Câu trả lời có nguồn, đúng quyền truy cập | Tài liệu được duyệt, metadata, quyền và ngày hiệu lực | Đúng nội dung, đúng nguồn, đúng phiên bản, giảm thời gian tìm kiếm |
| **G2 — Banking Document Processing** | Nhập liệu và kiểm tra hồ sơ tốn công | Phân loại, trích xuất, kiểm tra và chuyển hồ sơ cho người review | Chứng từ, document schema, validation rules, correction | Độ đúng từng trường, phát hiện thiếu/mâu thuẫn, effort sửa |
| **G3 — GenAI for Data Engineering & Analytics** | Engineering mất nhiều thời gian hiểu và tạo SQL, mapping, tài liệu, test | Artifact kỹ thuật có thể review và kiểm thử | Schema, semantic definitions, mapping, code, expected results | Đúng logic, vượt kiểm thử độc lập, giảm tổng effort gồm cả sửa lỗi |
| **A1 — AIOps / Xora Resolve** | Thu thập evidence và điều tra sự cố mất nhiều thời gian | Incident workspace, timeline, hypothesis, recommendation và review | Metrics, logs, traces, topology, changes, incident history | Evidence coverage, độ đúng RCA đã xác minh, thời gian và effort điều tra |
| **BSA-P1 — Payment Completion Assurance** | Chưa biết giao dịch đã hoàn tất hay đang ở trạng thái chưa xác định | Trạng thái nghĩa vụ thanh toán và hàng đợi ngoại lệ có evidence | Instruction, posting, phản hồi đối tác, xác nhận cuối cùng, return/reversal | Hoàn tất đúng hạn, pending/unknown count, value và age |
| **BSA-P2 — Batch & Cutoff Assurance** | File hoặc job báo thành công nhưng còn thiếu phần việc | Theo dõi từng đơn vị cần hoàn tất trước cutoff | Danh sách kỳ vọng, dòng batch, approval, processing và final outcomes | Đủ số lượng, đúng hạn, phần còn thiếu và giá trị chưa hoàn tất |
| **BSA-P3 — Posting & Reconciliation Assurance** | Trạng thái hoặc số liệu không khớp giữa các hệ thống | Kết quả matching và discrepancy queue có người chịu trách nhiệm | Posting, ledger, settlement, amount, currency, value date | Chênh lệch, tuổi ngoại lệ, thời gian xử lý và kết quả đối soát |

Các nhóm FinOps, SecOps, AI/ML Governance, Contact Center/CX, Compliance và Risk là các hướng mở rộng trong danh mục Xora. Mỗi nhóm cần được xác định use case, nguồn dữ liệu và qualification riêng trước khi đưa vào cam kết cung cấp.

Ba pattern BSA có thể áp dụng cho nhiều domain: chuyển tiền, QR/bill/wallet, thẻ, ATM, tiền gửi, tín dụng, payroll, thanh toán quốc tế, treasury, trade finance, custody, onboarding, thực thi kiểm soát fraud/AML, EOD và xử lý tranh chấp.

**Sức khỏe kỹ thuật và kết quả nghiệp vụ phải được đo riêng.** API trả thành công, container restart thành công hoặc job kết thúc thành công chưa tự xác nhận người nhận đã nhận tiền, batch đã đủ dòng hay báo cáo đã được bên nhận chấp thuận.

Với BSA, SME ngân hàng phải xác định trước:

- Business object và nghĩa vụ cần hoàn tất.
- Tập đối tượng kỳ vọng để làm mẫu số.
- Nguồn trạng thái có thẩm quyền.
- Quy tắc định danh, matching và xử lý retry.
- Business date, cutoff và độ trễ được phép.
- Người có quyền xử lý ngoại lệ và xác nhận kết quả.

---

1. **Thiết kế Xora cần được nhìn theo nhiều chiều đồng thời**

Một sơ đồ runtime không thể thay thế toàn bộ thiết kế sản phẩm. Team cần duy trì các góc nhìn liên kết sau:

| Chiều thiết kế | Câu hỏi cần trả lời | Đối tượng hoặc artifact chính |
| --- | --- | --- |
| **Kinh doanh** | Giải quyết vấn đề gì, cho ai, giá trị nào? | Use Case Brief, offering, business outcome, KPI |
| **Năng lực sản phẩm** | Năng lực nào có thể tái sử dụng? | Capability family, mental model, capability contract |
| **Đóng gói giải pháp** | Cần kết hợp những năng lực nào? | UC package, journey, dependency manifest |
| **Thực thi** | Service, agent, model và con người phối hợp thế nào? | Workflow, agent specification, tool manifest, authority |
| **Dữ liệu và tri thức** | Hệ thống biết gì, nhớ gì, lấy bằng chứng ở đâu? | Data contract, ontology, KB, memory, evidence |
| **Model** | Model làm nhiệm vụ gì, được thích nghi và đánh giá thế nào? | Model configuration, model card, training/evaluation protocol |
| **Tenant và deployment** | Chạy ở đâu, dùng nguồn nào, ai được truy cập? | Profile, TenantContext, tenant binding, deployment manifest |
| **Learning và chất lượng** | Cải thiện điều gì, chứng minh bằng cách nào? | Learning Item, WorkPackage, dataset, evaluation, qualification receipt |
| **Vận hành và chi phí** | Ai quản trị, phát hành, thu hồi và chịu chi phí? | XoraOps records, telemetry, usage attribution, L1–L9 |

Một thay đổi có thể tác động nhiều chiều. Ví dụ, đổi model provider có thể ảnh hưởng chất lượng đầu ra, nơi xử lý dữ liệu, latency, chi phí và giá trị của qualification evidence trước đó.

---

1. **Thư viện đơn vị thiết kế: từ nghiệp vụ đến một lần thực thi**

| Thuật ngữ chuẩn | Định nghĩa dùng trong Xora | Ví dụ hoặc quan hệ |
| --- | --- | --- |
| **Business Domain** | Miền nghiệp vụ có khái niệm, quy tắc và người sở hữu tương đối thống nhất | Payments, Lending, Data Operations |
| **Business Service** | Dịch vụ tạo kết quả có ý nghĩa cho người dùng hoặc tổ chức | Chuyển tiền liên ngân hàng |
| **Technical Service** | Thành phần phần mềm hoặc hạ tầng phục vụ một chức năng kỹ thuật | Payment API, middleware, database |
| **Business Object** | Đối tượng nghiệp vụ được theo dõi xuyên quy trình | Payment instruction, hồ sơ vay, dòng payroll |
| **Business Outcome** | Kết quả nghiệp vụ cần được xác minh | Instruction hoàn tất theo định nghĩa đã thống nhất |
| **Offering** | Phạm vi giá trị và dịch vụ/sản phẩm được đóng gói để cung cấp | AIOps cho một IT service |
| **Use Case — UC** | Tình huống công việc cụ thể: actor, trigger, input, xử lý, output và tiêu chí thành công | Điều tra nguyên nhân tăng lỗi giao dịch |
| **Design Win** | Việc giải pháp được khách hàng lựa chọn cho một phạm vi đã xác nhận, theo tiêu chí ghi nhận của tổ chức | Cần hồ sơ lựa chọn; tên nhóm trong roadmap chưa phải bằng chứng đã thắng |
| **Capability** | Khả năng thực hiện một nhiệm vụ với contract xác định | Đọc startup status; kiểm tra evidence freshness |
| **Capability Family** | Đơn vị catalogue tập hợp năng lực liên quan, có ngữ nghĩa, quy tắc và cách đánh giá chung | Evidence Assessment |
| **Mental Model** | Mô hình khái niệm và lập luận giúp biểu diễn domain: thực thể, quan hệ, điều kiện và quy tắc | Phân biệt symptom, impacted service và causal dependency |
| **UC Package** | Gói thực thi kết hợp capability, workflow, tool, kiến thức và kiểm thử để giải quyết UC | IT Service Investigation |
| **Journey** | Chuỗi công việc xuyên package, hệ thống hoặc vai trò để đạt outcome | Detect → investigate → review → recover → verify |
| **Profile** | Cấu hình đại diện tạo ra yêu cầu phát triển hoặc kiểm thử khác biệt | Công nghệ, provider, quyền, topology, phiên bản |
| **Tenant** | Phạm vi khách hàng/tổ chức được quản lý và cách ly dữ liệu, cấu hình, quyền | Một ngân hàng hoặc phạm vi được định nghĩa trong hợp đồng |
| **Tenant Binding** | Ánh xạ package/profile với nguồn, identity, policy, model và kiến thức thực tế của tenant | Binding cho iPay UAT |
| **Case** | Hồ sơ công việc hoặc vấn đề được theo dõi qua nhiều lần xử lý | Một incident đang điều tra |
| **Run / Execution** | Một lần thực thi workflow hoặc agent trên input và phiên bản xác định | Chạy lại điều tra sau khi có thêm evidence |
| **Session** | Phạm vi duy trì tương tác/ngữ cảnh theo runtime | Một phiên trao đổi; quan hệ với run phải được thiết kế rõ |

Trong workbook, “mental model/family” được dùng gần nhau để mô tả đơn vị đầu tư năng lực. Khi triển khai, nên dùng **Family** làm đối tượng catalogue và tham chiếu **Mental Model** như một artifact có phiên bản của năng lực đó. Cách chuẩn hóa này không tạo thêm một đơn vị đếm sản phẩm.

Các quan hệ chính:

- Một package có thể sử dụng nhiều family.
- Một family có thể phục vụ nhiều package.
- Một journey có thể kết hợp nhiều package.
- Một package có thể được dùng trong nhiều journey.
- Một case có thể có nhiều run.
- Profile cần ghi rõ phạm vi: family, package hay journey.
- Tenant binding chọn đúng phiên bản và cấu hình được phép triển khai.

Nhu cầu và Use CaseUC PackageJourneyCapability FamilyMental Model và KnowledgeService, Agent hoặc ModelTenant BindingProfile có phiên bảnCase và RunEvidence và OutcomeKết hợpTái sử dụng

Các mục tiêu như 250 family, 350 package và 32 journey trong workbook là các inventory kế hoạch khác nhau. Chúng không xác định trực tiếp số lượng agent, foundation model hoặc môi trường triển khai.

---

1. **Agents là thành phần thực thi có nhiệm vụ và quyền giới hạn**

Một agent nhận mục tiêu, sử dụng context và model để lựa chọn hoặc thực hiện các bước trong phạm vi được giao. Các hành vi của agent phải được ràng buộc bởi workflow, tool contract và authority.

| Thuật ngữ | Định nghĩa | Quy tắc thiết kế |
| --- | --- | --- |
| **Agent** | Thành phần thực thi nhiệm vụ, có khả năng dùng context và chọn hành động/tool phù hợp | Có input/output, quyền, giới hạn và điều kiện dừng |
| **Agent Role** | Trách nhiệm logic được giao cho agent | Planner, Analyst, Recommendation, Verifier |
| **Workflow** | Quy trình có trạng thái, bước chuyển, điều kiện và xử lý lỗi | Có thể chứa code, agent và human task |
| **Supervisor** | Thành phần điều phối và phân công tác vụ giữa các worker | Chỉ phân công trong phạm vi authority đã cấp |
| **Arbiter** | Thành phần xử lý kết quả cạnh tranh hoặc bất đồng theo tiêu chí | Có thể yêu cầu bổ sung hoặc chuyển người phân xử |
| **Skill** | Phương pháp thực hiện một loại nhiệm vụ, có điều kiện áp dụng và kiểm thử | Nêu bước, nhánh quyết định, evidence và stop rule |
| **Tool** | Thao tác có thể gọi qua giao diện xác định | Ví dụ đọc queue status |
| **Connector** | Thành phần kết nối với hệ thống nguồn/đích | Quản lý giao thức, xác thực và trao đổi dữ liệu |
| **Adapter** | Thành phần ánh xạ contract chung sang giao diện hoặc ngữ nghĩa cụ thể | Mapping capability đọc startup sang nguồn của tenant |
| **Tool Gateway** | Điểm kiểm soát truy cập, định tuyến và thực thi tool | Kiểm tra identity, quyền, schema, giới hạn và audit |
| **HITL — Human-in-the-loop** | Bước con người tham gia review hoặc quyết định trong workflow | Ghi rõ quyết định nào, người nào và evidence cần xem |
| **Runtime** | Môi trường thực thi agent/workflow | Quản lý tài nguyên, phiên, lỗi và telemetry |

**Các vai trò agent có thể tái sử dụng**

| Vai trò | Đầu ra chính |
| --- | --- |
| **Triage / Evidence Planner** | Phạm vi, kế hoạch thu thập và phần evidence còn thiếu |
| **Domain Analyst** | Phân tích theo domain, kết luận hoặc giả thuyết có nguồn |
| **Recommendation** | Đề xuất bước xử lý, điều kiện, tác động và cách xác minh |
| **Verifier** | Kết quả kiểm tra tính đúng, nhất quán và mức đủ của evidence |
| **Learning Curator** | Memory/KB/skill candidate hoặc Learning Item cho XBrain |

Một workflow đơn giản có thể chỉ dùng một agent. Một vai trò cũng có thể được thực hiện bằng code khi quy tắc đã rõ.

Các bước tính toán số tiền, đối soát, xác thực quyền, cập nhật trạng thái và kiểm tra contract cần có logic xác định. Agent hỗ trợ giải thích, điều tra và đề xuất trong phạm vi phù hợp.

**Agent Specification tối thiểu**

Mỗi agent cần khai báo nhiệm vụ; input/output; tool được phép; memory/KB được phép; model configuration; evidence requirements; timeout/budget; điều kiện dừng; escalation; test suite và version.

**Skill Specification tối thiểu**

Mỗi skill cần có điều kiện áp dụng và loại trừ; capability bắt buộc; giả thuyết cần phân biệt; các bước kiểm tra; cách diễn giải kết quả; tiêu chí kết luận; giới hạn quyền; output contract và test fixtures.

---

1. **Memory, KB và dữ liệu nguồn có chức năng khác nhau**

Trạng thái giao dịch, cấu hình đang triển khai và quyền truy cập hiện hành phải được lấy từ nguồn có thẩm quyền. Memory và KB bổ sung ngữ cảnh, kinh nghiệm và hướng dẫn.

**Phân loại memory theo thời gian và nội dung**

“Ngắn hạn/dài hạn” mô tả thời gian sử dụng. “Episodic/semantic/procedural” mô tả loại nội dung. Đây là hai chiều phân loại khác nhau.

| Loại | Nội dung | Cách sử dụng |
| --- | --- | --- |
| **Working Memory / STM** | Mục tiêu hiện tại, evidence đang dùng, giả thuyết, bước đã thử | Duy trì tiến trình của run/case |
| **Episodic Memory** | Kinh nghiệm từ một lần xử lý: bối cảnh, bước thực hiện, kết quả và giới hạn | Tìm case tương tự và so sánh điều kiện |
| **Semantic Memory** | Những thông tin hoặc đặc điểm đã học được về môi trường | Hỗ trợ hiểu bối cảnh; kiểm tra lại khi môi trường thay đổi |
| **Procedural Memory** | Kinh nghiệm về cách thực hiện nhiệm vụ | Nội dung đã chuẩn hóa nên được phát hành qua Skill Registry |
| **Preference Memory** | Sở thích tương tác được phép ghi nhận | Cá nhân hóa cách trình bày; không thay đổi quyền hoặc quy tắc nghiệp vụ |
| **Failure Memory** | Kinh nghiệm về bước thất bại, diễn giải sai, giới hạn của tool | Là nhóm nội dung của memory; luôn giữ điều kiện thất bại |


Memory còn cần phạm vi: cá nhân, nhóm, tenant, domain hoặc shared theo quyền được phê duyệt.

“Escalation memory” là cách gọi phần ngữ cảnh được giữ để tiếp tục xử lý escalation. Nó có thể chứa các kết luận chưa xác minh; trạng thái đó phải được giữ nguyên.

Mỗi memory dài hạn cần:

`Nội dung + phạm vi + nguồn + trạng thái xác minh + điều kiện áp dụng + thời gian hiệu lực + phiên bản + quyền sử dụng`.

Một case chưa có RCA vẫn có thể cung cấp memory hữu ích về các bước đã thử hoặc nguồn chưa đọc được. Phần nguyên nhân tiếp tục giữ `Unknown`.

**Phân loại KB theo phạm vi quản trị**

| Lớp KB | Nội dung | Chủ sở hữu nội dung |
| --- | --- | --- |
| **Shared Technical KB** | Tài liệu công nghệ và cơ chế lỗi được phép tái sử dụng | Technical/domain owner |
| **Domain / Family KB** | Khái niệm, quan hệ, quy tắc và kiến thức của family | SME và family owner |
| **Package KB** | Kiến thức cần cho cách phối hợp và đầu ra của package | Package owner |
| **Tenant KB** | Runbook, mapping, kiến trúc, quy trình và business rule riêng | Knowledge owner của tenant |
| **Skill Registry** | Phương pháp thực hiện đã có version và evaluation | Skill owner và release authority |

KB cần quản lý cả nội dung và metadata: nguồn, owner, phiên bản, ngày hiệu lực, quyền truy cập, phạm vi công nghệ và trạng thái thu hồi.

**Các thuật ngữ truy xuất tri thức**

| Thuật ngữ | Định nghĩa |
| --- | --- |
| **Document** | Đơn vị tài liệu nguồn được quản lý và có provenance |
| **Chunk** | Phần tài liệu được chia để xử lý/truy xuất; phải giữ liên kết về nguồn và vị trí |
| **Embedding** | Biểu diễn số phục vụ so sánh hoặc tìm kiếm; vẫn chịu chính sách dữ liệu |
| **Vector Index** | Chỉ mục phục vụ tìm các biểu diễn gần nhau |
| **Retrieval** | Tìm và lấy thông tin phù hợp từ nguồn được phép |
| **Reranking** | Xếp hạng lại các kết quả đã tìm để ưu tiên nội dung phù hợp hơn |
| **RAG** | Kết hợp nội dung truy xuất với model sinh câu trả lời |
| **Grounding** | Gắn phát biểu với evidence hoặc nguồn hỗ trợ phù hợp |
| **Context Window** | Dung lượng input/output model có thể xử lý trong một lần gọi, theo giới hạn của model |
| **Context Engineering** | Chọn, tổ chức và giới hạn thông tin đưa vào model cho nhiệm vụ cụ thể |

RAG kết hợp tri thức trong model với thông tin truy xuất bên ngoài. Việc cập nhật corpus KB để phục vụ RAG không tự cập nhật trọng số model. [arxiv.org](https://arxiv.org/abs/2005.11401?utm_source=chatgpt.com)

**Quy trình tạo context**

1. Xác thực tenant, người dùng và mục đích.
2. Xác định service/domain, môi trường và thời điểm.
3. Lọc quyền, trạng thái phê duyệt và điều kiện áp dụng trước khi tìm kiếm.
4. Lấy dữ liệu hiện tại, KB còn hiệu lực và memory liên quan.
5. Kiểm tra mâu thuẫn, loại trùng và giới hạn dung lượng.
6. Đưa context vào model cùng provenance và phần chưa biết.

Nội dung trong log, tài liệu hoặc tool result được xử lý như dữ liệu. Nó không được tự chuyển thành quyền gọi tool hay chỉ dẫn hệ thống.

**Lưu ý về tên “memory” trong hạ tầng:** RAM của runtime, ví dụ được tính bằng GB-hour trong workbook, là tài nguyên tính toán. STM/LTM là chức năng quản lý ngữ cảnh và kinh nghiệm, có contract và vòng đời riêng.

---

1. **“Model” cần được phân loại theo đúng nghĩa**

Trong tài liệu Xora, từ “model” xuất hiện ở nhiều nghĩa. Team cần ghi tên đầy đủ.

| Thuật ngữ | Ý nghĩa |
| --- | --- |
| **Business Process Model** | Biểu diễn các bước, actor, trạng thái và điều kiện của quy trình nghiệp vụ |
| **Domain Model** | Biểu diễn các khái niệm và quan hệ trong một domain |
| **Data Model** | Cấu trúc dữ liệu, khóa, quan hệ, kiểu và ràng buộc |
| **Mental Model** | Cấu trúc khái niệm và quy tắc lập luận của năng lực |
| **AI/ML Model** | Mô hình có tham số được học để dự đoán, phân loại, biểu diễn hoặc sinh đầu ra |
| **Foundation Model** | Model được huấn luyện trên dữ liệu rộng, có thể thích nghi cho nhiều nhiệm vụ |
| **Model Configuration** | Model/version, tham số inference, endpoint, routing và giới hạn sử dụng |
| **Model Artifact / Checkpoint** | Tài sản phục vụ triển khai hoặc tiếp tục huấn luyện, gồm trọng số và cấu hình liên quan |
| **Model Card** | Hồ sơ mô tả mục đích, dữ liệu, đánh giá, giới hạn và điều kiện sử dụng model |

**Phân loại AI/ML model theo chức năng**

Các nhóm dưới đây có thể chồng lấn; một model có thể thực hiện nhiều chức năng.

| Nhóm | Chức năng | Ứng dụng trong ngân hàng/Xora |
| --- | --- | --- |
| **Statistical / Time-series Model** | Mô tả baseline, dự báo hoặc phát hiện thay đổi chuỗi thời gian | Latency, capacity, tải, thời gian hoàn tất batch |
| **Classification Model** | Gán nhóm hoặc nhãn | Phân loại tài liệu, điểm bị chặn, nhóm lỗi |
| **Regression / Forecasting Model** | Ước lượng đại lượng liên tục hoặc giá trị tương lai | Dự báo nhu cầu tài nguyên, thời gian xử lý |
| **Ranking Model** | Xếp hạng các lựa chọn | Evidence, giả thuyết RCA, bước điều tra |
| **Embedding Model** | Biểu diễn dữ liệu cho tìm kiếm hoặc so sánh | Retrieval tài liệu và case tương tự |
| **Reranker** | Đánh giá lại mức phù hợp của query–candidate | Chọn đoạn KB đúng để đưa vào context |
| **Language Model / LLM** | Hiểu và sinh ngôn ngữ hoặc nội dung có cấu trúc | Giải thích, tổng hợp evidence, sinh SQL, đề xuất |
| **Multimodal / Document Model** | Xử lý văn bản kết hợp hình ảnh hoặc cấu trúc tài liệu | Chứng từ, bảng, biểu mẫu, trích xuất |
| **Speech Model** | Nhận dạng hoặc tổng hợp tiếng nói | Contact center, transcription, voice assistant |

LLM/SLM thường mô tả nhóm model ngôn ngữ theo quy mô tương đối; số tham số như 8B, 32B hay 70B không tự chứng minh chất lượng cho một use case.

Planner, Worker, Judge và Router là **vai trò sử dụng model**. Chúng không mặc định tương ứng với các foundation model khác nhau.

**Phân loại cách thích nghi và cải thiện**

| Cách tiếp cận | Thành phần thay đổi | Khi phù hợp |
| --- | --- | --- |
| **Prompt / Instruction Update** | Chỉ dẫn và cấu trúc input | Nhiệm vụ hoặc output chưa rõ |
| **RAG / KB Update** | Nội dung được truy xuất | Thiếu hoặc sai kiến thức tham chiếu |
| **Memory Update** | Ngữ cảnh và kinh nghiệm được giữ lại | Cần tận dụng lịch sử đúng phạm vi |
| **Skill / Workflow Update** | Phương pháp thực hiện và thứ tự bước | Chưa biết kiểm tra gì hoặc xử lý nhánh lỗi |
| **Fine-tuning** | Tham số của model hoặc thành phần được huấn luyện | Có bài toán, dữ liệu và bằng chứng cần thích nghi trọng số |
| **SFT — Supervised Fine-tuning** | Học từ các ví dụ input–output được chuẩn bị | Cải thiện hành vi cho nhiệm vụ cụ thể |
| **PEFT / LoRA** | Huấn luyện một phần nhỏ tham số hoặc adapter | Thích nghi model với phạm vi tài nguyên phù hợp |
| **QLoRA** | Huấn luyện LoRA trên base model lượng tử hóa | Một cách giảm nhu cầu bộ nhớ trong quá trình thích nghi |
| **Quantization** | Biểu diễn tham số bằng độ chính xác thấp hơn | Tối ưu tài nguyên; vẫn cần đánh giá chất lượng |
| **Distillation** | Huấn luyện model học từ đầu ra hoặc tín hiệu của model khác | Tạo model phù hợp hơn với yêu cầu triển khai sau khi kiểm chứng |

Fine-tuning là nhóm phương pháp rộng. SFT mô tả cách học từ dữ liệu có giám sát; LoRA mô tả cách giới hạn phần tham số được cập nhật. Chúng có thể được sử dụng cùng nhau. LoRA giữ base weights cố định và học các thành phần cập nhật; QLoRA kết hợp cách này với base model lượng tử hóa. [arxiv.org](https://arxiv.org/abs/2106.09685?utm_source=chatgpt.com)

**Phân loại deployment là một chiều riêng**

- **Managed model API:** truy cập model qua dịch vụ được quản lý.
- **Self-hosted model:** tổ chức quản lý môi trường phục vụ model.
- **Dedicated/private deployment:** tài nguyên hoặc phạm vi truy cập được dành riêng theo thiết kế.
- **Shared deployment:** nhiều workload dùng chung hạ tầng với cơ chế cách ly phù hợp.

Model dùng trong private deployment có thể giữ nguyên trọng số. “Model riêng” phải nói rõ là **riêng endpoint, riêng tài nguyên, riêng adapter hay riêng trọng số**.

M1 có thể quản trị model dự báo hoặc phân loại sẵn có của ngân hàng. Phạm vi này cần được phân biệt với việc Xora nghiên cứu model phục vụ chính các agent của mình.

---

1. **Mỗi nhóm UC lựa chọn tổ hợp agent–memory–KB–model khác nhau**

| UC | Cách thực thi chính | Memory và KB | Model có thể sử dụng |
| --- | --- | --- | --- |
| **D1** | Pipeline, DQ và reconciliation bằng quy tắc; agent hỗ trợ điều tra ngoại lệ | Mapping, schema, data dictionary, lịch sử lỗi pipeline | Model bất thường nếu đủ dữ liệu; LLM khi có nhu cầu giải thích |
| **M1** | Registry, evaluation, deployment, monitoring và rollback | Model/feature contract, deployment history, evaluation history | Model của ngân hàng; model đánh giá hoặc phân tích khi phù hợp |
| **G1** | Retrieval có quyền và sinh câu trả lời có nguồn | Corpus được duyệt, session context, preference được phép | Embedding, reranker, LLM |
| **G2** | Intake, classify, extract, validate, human correction | Document schema, validation rules, trạng thái hồ sơ | OCR/document model, classifier, multimodal model hoặc LLM |
| **G3** | Sinh/giải thích artifact và chạy kiểm thử độc lập | Semantic definitions, code, mapping, project context | Code-capable LLM, embedding/reranker khi cần |
| **A1** | Evidence collection, hypothesis, recommendation và review | Topology, runbook, diagnostic skills, incident memory | LLM, ranking/classification; detector riêng khi đủ điều kiện |
| **BSA** | Quy tắc xác định nghĩa vụ, matching, trạng thái và cutoff; agent điều tra ngoại lệ | Business rules, authoritative source mapping, exception history | Model hỗ trợ phân loại/xếp hạng/giải thích; logic xác nhận trạng thái có quy tắc |

Model selection cần dựa trên yêu cầu chất lượng, dữ liệu được phép xử lý, latency, chi phí và khả năng đánh giá. Tên model trong bảng sizing là giả định cấu hình cần được qualification, không tự trở thành chuẩn bắt buộc cho mọi tenant.

---

1. **Dữ liệu học cần được thiết kế theo câu hỏi và đơn vị bản ghi**

**Logical dataset** là tập dữ liệu được chuẩn hóa về ý nghĩa, định danh, thời gian và quan hệ để phục vụ một mục tiêu cụ thể. Nó có thể là view tại môi trường khách hàng, không nhất thiết là một bản sao tập trung.

| Thuật ngữ | Định nghĩa và cách dùng |
| --- | --- |
| **Raw Data** | Dữ liệu nguồn trước quá trình chuẩn hóa cho use case |
| **Telemetry** | Dữ liệu quan sát hệ thống, thường gồm metrics, logs, traces |
| **Signal** | Quan sát được lựa chọn hoặc suy ra để thể hiện một trạng thái |
| **Alert** | Thông báo khi điều kiện giám sát được đáp ứng |
| **Incident** | Sự kiện gián đoạn hoặc suy giảm dịch vụ cần xử lý theo quy trình |
| **Problem Record** | Hồ sơ theo dõi nguyên nhân hoặc vấn đề cần điều tra sâu, có thể liên quan nhiều incident |
| **Evidence** | Thông tin có nguồn và bối cảnh được dùng để hỗ trợ đánh giá một phát biểu |
| **Hypothesis** | Giả thuyết cần kiểm chứng |
| **RCA — Root Cause Analysis** | Quá trình xác định cơ chế và thành phần gây ra vấn đề trong phạm vi điều tra |
| **Provenance** | Nguồn gốc và cách hình thành dữ liệu hoặc kết luận |
| **Lineage** | Quan hệ phụ thuộc qua các bước biến đổi, dataset và artifact |
| **Grain** | Ý nghĩa của một bản ghi trong dataset |
| **Feature** | Biến đầu vào được xây cho một bài toán model |
| **Label** | Nhãn/đầu ra tham chiếu gắn với bản ghi |
| **Ground Truth** | Kết quả tham chiếu được xác minh theo tiêu chí của nhiệm vụ, có thể được sửa khi có evidence mới |
| **Missingness** | Tình trạng thiếu dữ liệu và nguyên nhân thiếu |
| **Freshness** | Mức độ cập nhật của dữ liệu so với yêu cầu sử dụng |
| **Coverage** | Mức bao phủ so với tập nguồn, thực thể hoặc trường hợp cần quan sát |

**Các đơn vị học khác nhau**

| Bài toán | Grain phù hợp | Nhãn hoặc kết quả tham chiếu |
| --- | --- | --- |
| Phát hiện bất thường | Entity/operation trong một cửa sổ thời gian | Bình thường/bất thường theo phương pháp đã xác minh |
| Chẩn đoán RCA | Một incident episode và evidence sẵn có | Nguyên nhân đã xác minh hoặc chưa xác định |
| Chọn bước điều tra | Trạng thái điều tra tại thời điểm ra quyết định | Các bước hợp lệ, hữu ích và kết quả thực hiện |
| Retrieval | Query, quyền người hỏi và snapshot corpus | Tài liệu/đoạn liên quan được review |
| Trích xuất tài liệu | Document hoặc field có vị trí nguồn | Giá trị được xác minh và loại lỗi |
| Đánh giá remediation | Hành động, điều kiện trước/sau và outcome | Kết quả thực hiện và phục hồi |

Không nên gộp các grain này vào một bảng rồi coi mọi dòng là mẫu học tương đương.

**Mô hình dữ liệu tối thiểu cho incident investigation**

| Thực thể | Nội dung chính |
| --- | --- |
| `source_snapshot` | Nguồn nào đọc được, schema, retention, freshness và trạng thái truy vấn |
| `entity_mapping` | Alias/ID nguồn → canonical entity, môi trường và thời gian hiệu lực |
| `metric_point / log_event / span_event` | Tín hiệu giữ đúng grain, đơn vị và nguồn |
| `dependency_edge` | Quan hệ configured, observed hoặc inferred cùng evidence |
| `change_event` | Thay đổi triển khai/cấu hình có thực thể và thời gian hiệu lực |
| `incident_episode` | Phạm vi triệu chứng, thực thể ảnh hưởng và thời gian |
| `episode_evidence` | Quan hệ giữa episode, evidence và phát biểu được đánh giá |
| `diagnostic_step` | Bước đã chọn, tool, kết quả, lý do dừng hoặc tiếp tục |
| `outcome_label` | Kết luận có revision, người xác minh và kết quả phục hồi |

Các nguyên tắc dữ liệu bắt buộc:

- Phân biệt `event_time`, `ingest_time` và thời điểm dữ liệu sẵn có cho agent.
- Tách `impacted_entity`, `causal_entity` và `failure_mechanism`.
- Mapping thực thể cần thời gian hiệu lực.
- Thiếu metric giữ là missing; không tự đổi thành 0.
- Tool timeout giữ là lỗi thực thi; không đổi thành “không có lỗi”.
- Liên hệ theo cửa sổ thời gian phải giữ đúng là tương quan theo thời gian.
- Không đưa kết quả tương lai vào context của một quyết định trong quá khứ.

Trong hồ sơ VTB, ưu tiên phù hợp là xác minh lại nguồn, mapping, field semantics, evidence linkage và feedback. Số lượng telemetry lớn chưa chứng minh đã có đủ mẫu RCA độc lập và nhãn đáng tin cậy.

---

1. **Kiến trúc runtime và dữ liệu phải giữ đúng ranh giới tenant**

| Thành phần | Vai trò |
| --- | --- |
| **Xora Platform** | Identity, authority, orchestration, Tool Gateway, context, registry và telemetry |
| **Xora Resolve** | Domain workflow và trải nghiệm người dùng cho incident investigation |
| **XBrain** | Tổ chức phát triển và cải tiến năng lực từ Learning Item đến kiểm chứng hiệu quả |
| **XoraOps** | Quản trị policy, qualification, release, deployment, usage và quyết định vận hành |
| **Tenant Adapter** | Mapping contract chung với nguồn thực tế; kiểm soát trao đổi và xuất dữ liệu |

Luồng thực thi chuẩn:

> Intent → TenantContext → Authority Resolution → Workflow/Supervisor/Arbiter → Capability Resolution → Tenant Bindings → Execution → Evidence/Outcome.

**TenantContext** phải xuất phát từ danh tính đã xác thực. Agent không tự chọn tenant hoặc mở rộng quyền thông qua tham số do model tạo.

**Các contract giao tiếp chính**

| Contract | Vai trò |
| --- | --- |
| `ExecutionContext` | Tenant, principal, domain, case/run, quyền, policy và ngân sách |
| `EvidenceBundle` | Evidence references, thời gian, thực thể, kết quả truy vấn và giới hạn |
| `InvestigationEpisode` | Giả thuyết, bước điều tra, evidence và trạng thái |
| `EscalationRecord` | Điểm bị chặn và phần thông tin được phép chuyển tiếp |
| `DiagnosticTask` | Yêu cầu kiểm tra bổ sung với capability, giới hạn và output mong đợi |
| `OutcomeFeedback` | Kết quả xác minh chẩn đoán, hành động và phục hồi |
| `KnowledgeDeployment` | Phiên bản được phân phối, phạm vi cho phép và xác nhận kích hoạt |

Contract cần xử lý được gửi lặp, feedback đến muộn, revision và mất kết nối. Evidence reference chỉ phục vụ truy vết; quyền đọc dữ liệu gốc vẫn được kiểm tra riêng.

Với ràng buộc không đưa raw customer data về TechX:

- Raw evidence, mapping nhạy cảm và context chi tiết nằm trong vùng xử lý được khách hàng cho phép.
- Xora nhận structured escalation và feedback trong phạm vi được phép.
- Feature, embedding, summary và trace cũng phải được xét quyền sử dụng.
- XBrain có thể gửi skill hoặc bộ đánh giá xuống tenant và nhận kết quả xác minh được phép trả về.
- Inference, embedding và logging đều phải nằm trong thiết kế ranh giới xử lý dữ liệu.

SaaS, dedicated managed, customer-hosted và hybrid là các lựa chọn deployment. Trong mọi lựa chọn, tenant isolation phải bao phủ database, index, cache, queue, object storage, worker và trace.

Giai đoạn đầu có thể triển khai các thành phần logic thành module và worker trên nền tảng hiện hữu. Số lượng family không quyết định số lượng microservice.

---

**10. Learning là vòng cải thiện có kiểm chứng**

Có ba nhóm cải thiện cần được quản lý riêng:

1. **Cải thiện ngữ cảnh:** mapping, KB, memory, retrieval.
2. **Cải thiện cách thực hiện:** skill, tool, workflow, guard và output contract.
3. **Cải thiện model:** training, fine-tuning, lựa chọn hoặc thay model.

| Điểm bị chặn | Hướng xử lý ưu tiên |
| --- | --- |
| Thiếu nguồn hoặc độ chi tiết | Bổ sung observability và data access tại nguồn |
| Nguồn có nhưng không đọc được | Sửa connector, quyền hoặc tool |
| Sai thực thể hoặc dependency | Sửa mapping và ngữ nghĩa |
| Không tìm đúng kiến thức | Sửa corpus, metadata, retrieval/reranking |
| Có evidence nhưng chọn sai phép kiểm tra | Sửa diagnostic skill hoặc workflow |
| Kết luận vượt quá evidence | Sửa contract, verifier và evaluation |
| Đã biết nguyên nhân nhưng chưa được xử lý | Chuyển quy trình hành động/phê duyệt |
| Chưa xác minh recovery | Bổ sung nguồn và tiêu chí kiểm tra kết quả |

**Các thuật ngữ của vòng học**

| Thuật ngữ | Định nghĩa |
| --- | --- |
| **Learning Item** | Hồ sơ một cơ hội hoặc khoảng trống cần cải thiện |
| **Capability Gap** | Năng lực còn thiếu so với yêu cầu đã xác định |
| **WorkPackage** | Gói công việc có owner, input, dependency, artifact, review và tiêu chí hoàn tất |
| **Knowledge Candidate** | Nội dung đang được xem xét; chưa mặc nhiên được sử dụng như tri thức đã xác nhận |
| **Experiment** | Thử nghiệm có giả thuyết, cấu hình, dữ liệu và cách đánh giá |
| **Campaign** | Nhóm hoạt động thử nghiệm/qualification có mục tiêu và phạm vi xác định |
| **Fixture** | Dữ liệu hoặc môi trường có kiểm soát để chạy kiểm thử |
| **Fault Injection** | Chủ động tạo lỗi trong phạm vi thử nghiệm được phép |
| **Replay** | Chạy lại trên dữ liệu/trạng thái quá khứ với điều kiện tái dựng được |
| **Promotion** | Chuyển artifact sang trạng thái sử dụng cao hơn sau khi đạt điều kiện |
| **Revocation** | Thu hồi quyền sử dụng một phiên bản hoặc nội dung |
| **Field Validation** | Kiểm chứng hiệu quả trong môi trường sử dụng thực tế được cho phép |

Case và feedbackPhân loại khoảng trốngXBrain WorkPackageXây dựng ứng viênĐánh giá đạt?Phê duyệt phạm viTriển khai cho tenantXác minh hiệu quảĐóng gapChưa đạtĐạtCòn thiếuĐủ bằng chứng

Luồng cải tiến sản phẩm và luồng xử lý incident chạy liên kết nhưng có trạng thái độc lập. Triển khai một skill mới chưa xác nhận incident cũ đã được giải quyết.

Nếu chỉ nhận escalation thất bại, Xora có thể phân tích những điểm bị chặn trên nhóm case đó. Để đo hiệu quả toàn hệ thống, cần thêm tập đánh giá đại diện hoặc kết quả tổng hợp được phép từ tenant.

Muốn cải thiện **detection**, cần dữ liệu cả giai đoạn bình thường và sự cố, bao gồm sự cố bị bỏ sót được phát hiện độc lập. Dữ liệu escalation chủ yếu hỗ trợ **investigation** sau khi vấn đề đã được nhận biết.

---

**11. Chất lượng phải được đánh giá trên toàn bộ cấu hình thực tế**

Đối tượng cần đánh giá là tổ hợp:

> Model + prompt + skill + retrieval + KB + memory policy/context + tools + workflow + tenant binding.

**Thư viện thuật ngữ về đánh giá và phát hành**

| Thuật ngữ | Định nghĩa |
| --- | --- |
| **Baseline** | Phương án hoặc kết quả tham chiếu để so sánh |
| **Development Set** | Tập dùng để phát triển và điều chỉnh phương án |
| **Validation Set** | Tập hỗ trợ lựa chọn cấu hình hoặc tham số |
| **Holdout / Test Set** | Tập giữ riêng để đánh giá sau khi lựa chọn phương án |
| **Regression Test** | Kiểm tra năng lực đã có còn đáp ứng sau thay đổi |
| **Negative Test** | Kiểm tra input hoặc hành vi không hợp lệ và cách hệ thống xử lý |
| **Rubric** | Tiêu chí chấm có hướng dẫn diễn giải |
| **LLM-as-Judge** | Dùng model hỗ trợ chấm theo rubric; cần kiểm tra sai lệch với đánh giá phù hợp |
| **Abstention** | Chủ động chưa kết luận khi không đủ căn cứ |
| **Calibration** | Đánh giá mức phù hợp giữa độ tin cậy được báo cáo và kết quả thực tế |
| **Data Leakage** | Thông tin không được phép hoặc thông tin đáp án lọt vào quá trình học/đánh giá |
| **Drift** | Thay đổi dữ liệu hoặc hành vi khiến hiệu quả trước đây có thể không còn giữ được |
| **Qualification** | Chứng minh phiên bản đáp ứng tiêu chí trên phạm vi cấu hình xác định |
| **Acceptance** | Bên có thẩm quyền chấp nhận kết quả trong phạm vi đã thống nhất |
| **Release** | Phiên bản được phê duyệt để phân phối hoặc sử dụng theo điều kiện |
| **Deployment** | Việc đưa phiên bản vào một môi trường cụ thể |
| **Activation** | Cho phép phiên bản bắt đầu phục vụ workload |
| **Rollback** | Khôi phục cấu hình/phiên bản vận hành trước theo quy trình |

Qualification, acceptance, deployment và activation cần có hồ sơ riêng. Một artifact đạt lab test chưa mặc nhiên được chấp nhận cho production của tất cả tenant.

**Các trạng thái của case cũng cần độc lập**

| Chiều trạng thái | Ví dụ |
| --- | --- |
| Tiến độ hồ sơ | Open, Needs input, Under review, Closed |
| Chẩn đoán | Unknown, Hypothesis, Verified, Disputed |
| Phê duyệt hành động | Not requested, Pending, Approved, Rejected |
| Thực hiện hành động | Not attempted, Blocked, Failed, Succeeded |
| Phục hồi | Unknown, Partial, Verified |



Đóng case vì thiếu dữ liệu vẫn giữ chẩn đoán là `Unknown`. Review chấp nhận báo cáo không tự chuyển hành động thành đã phê duyệt hoặc recovery thành đã xác minh.

**Nguyên tắc đánh giá**

- Tách dữ liệu theo incident và nhóm kịch bản liên quan.
- Chỉ cung cấp thông tin sẵn có tại thời điểm ra quyết định.
- Cố định phiên bản và ghi nhận context đã sử dụng.
- Có case đúng mục tiêu, triệu chứng tương tự nhưng khác nguyên nhân, thiếu evidence, tool lỗi và trạng thái bình thường.
- Đánh giá theo tenant, domain, profile và độ đầy đủ dữ liệu.
- Chốt tiêu chí trước khi chấm ứng viên.
- Có reviewer phù hợp ngoài người trực tiếp tạo ứng viên. Cách tổ chức này cũng phù hợp với hướng dẫn đánh giá độc lập trong NIST AI RMF. [AIRC](https://airc.nist.gov/airmf-resources/playbook/measure/?utm_source=chatgpt.com)

**Các nhóm chỉ số chính**

| Nhóm | Cách đo cần làm rõ |
| --- | --- |
| Dữ liệu | Schema validity, mapping correctness, freshness, coverage, missingness |
| Retrieval | Kết quả đúng quyền, đúng phiên bản, đủ evidence liên quan |
| Chẩn đoán | Độ đúng trên case có ground truth, tỷ lệ kết luận thiếu căn cứ, mức bao phủ kết luận |
| Hiệu quả công việc | Thời gian, số bước, tool call không hữu ích, effort review/sửa |
| Vận hành | Runtime/tool failures, latency, retry, chi phí và rollback |
| Nghiệp vụ | Kết quả trên tập nghĩa vụ kỳ vọng, đúng cutoff và nguồn xác nhận |

KPI cần định nghĩa mẫu số, population, cửa sổ thời gian và cách xử lý unknown. Điểm “confidence” do model tự sinh chưa tự trở thành xác suất đã hiệu chỉnh.

**Release Manifest**

Manifest cần liên kết family/package/journey version, profile, agent/prompt/skill, tool contracts, model configuration, KB snapshot, memory policy, dataset/evaluation versions, kết quả, quyết định phê duyệt, tenant deployment và rollback reference.

---

**12. Ví dụ xuyên suốt: điều tra iPay và kiểm tra hoàn tất thanh toán**

Đây là kịch bản minh họa thiết kế, không phải RCA đã xác minh tại VTB.

| Cấp thiết kế | Ví dụ |
| --- | --- |
| Nhu cầu | Giảm thời gian xác định nguyên nhân giao dịch chậm |
| Business service | Chuyển tiền |
| UC kỹ thuật | Điều tra latency của service liên quan |
| Package | IT Service Investigation |
| Family | Entity Resolution, Evidence Assessment, Dependency Analysis, Hypothesis Ranking |
| Profile | OpenShift–Oracle–MQ, nguồn đã kiểm tra, quyền read-only |
| Tenant binding | Mapping, credentials, policy và KB của môi trường iPay được chọn |
| Agents | Planner, Analyst, Recommendation và Verifier theo workflow |
| Memory | Bước đã thử trong case và bài học riêng tenant còn hiệu lực |
| Model | Model phù hợp cho phân tích; các truy vấn/tính toán thực hiện qua tool |
| Evaluation | Đúng thực thể, đúng evidence, biết thiếu gì, đề xuất phép kiểm tra hữu ích |

Giả sử agent nhìn thấy latency tăng và exporter MQ không hoạt động:

1. Hệ thống ghi nhận đúng hai quan sát.
2. Evidence Planner kiểm tra phạm vi dữ liệu và nguồn còn thiếu.
3. Analyst giữ các giả thuyết phù hợp, chưa kết luận MQ là nguyên nhân.
4. Skill đề xuất kiểm tra trạng thái MQ qua capability được phép.
5. Nếu tool không thực hiện được, case giữ kết quả `Tool failed` hoặc `Not authorized`.
6. Người có quyền xem evidence tại nguồn xác minh kết luận khi có thêm dữ liệu.
7. Bài học được lưu đúng phạm vi tenant; phương pháp tái sử dụng đi qua XBrain.

Nếu ngân hàng muốn biết **một instruction đã hoàn tất hay chưa**, cần thêm BSA-P1 với mã liên kết, trạng thái có thẩm quyền và quy tắc finality của luồng đó. Telemetry kỹ thuật chỉ là một phần evidence.

Ví dụ G1 có cùng cấu trúc nhưng khác nhiệm vụ: nhân viên hỏi quy trình hiện hành; package kiểm tra quyền, truy xuất tài liệu đúng hiệu lực, model tạo câu trả lời có nguồn. Memory về câu hỏi trước đó không được làm mất hiệu lực của chính sách mới.

---

**13. Vai trò của từng nhóm và bộ bàn giao chung**

| Vai trò | Trách nhiệm chính | Đầu ra |
| --- | --- | --- |
| **SME** | Ngữ nghĩa domain, quy tắc, evidence đủ và outcome đúng | Domain definitions, rule review, label verification |
| **Product Manager** | Vấn đề, giá trị, phạm vi và ưu tiên | UC brief, acceptance criteria, outcome review |
| **Product Architect** | Capability boundaries, quan hệ đối tượng, authority và contract | Architecture baseline, capability model |
| **Data Architect** | Grain, schema, mapping, lineage và vòng đời dữ liệu | Data contract, ontology/data model |
| **Data Analyst** | Profiling, missingness, cohort và đo hiệu quả | Data quality/cohort reports |
| **Data Scientist** | Bài toán học, phương pháp, nhãn, retrieval/model và evaluation | Learning specification, experiments, evaluation protocol |
| **Data Engineer** | Adapter, pipeline, chuẩn hóa, version và data quality checks | Dataset/ingestion implementation |
| **Agentic Development / Technical Lead** | Agent, tool, workflow, integration và xử lý lỗi | Implementation, test và runtime evidence |
| **QA / Evaluation** | Kiểm thử độc lập và giới hạn khái quát hóa | Evaluation report, regression evidence |
| **DevOps / MLOps** | Môi trường, identity, CI/CD, deployment, monitoring và rollback | Deployment package, operational evidence |
| **Delivery / FDE** | Nguồn và quyền tại khách hàng, binding, feedback | Customer readiness, tenant configuration |
| **XBrain Production** | Điều phối WorkPackage, reuse, phụ thuộc và hiệu quả | Artifact manifest, tiến độ và gap closure evidence |
| **Release Authority / XoraOps** | Quyết định phát hành theo thẩm quyền và ghi nhận hồ sơ | Release decision, deployment/audit records |

Trong tài liệu và ticket, nên viết đầy đủ **Data Architect** và **Data Analyst**, tránh dùng “DA” cho cả hai.

**Bộ bàn giao tám thành phần**

| Thành phần | Nội dung |
| --- | --- |
| **Capability Card** | UC, giá trị, phạm vi, input/output, owner và tiêu chí |
| **Architecture / Flow** | Luồng thực thi, ranh giới tenant, quyền và dependency |
| **Input/Output Contract** | Schema, ngữ nghĩa, trạng thái, version và lỗi |
| **Tool Manifest** | Tool/capability, quyền, tham số, giới hạn và source mapping |
| **Config Schema** | Profile, model, retrieval, memory policy và tenant binding |
| **Deployment Guide** | Môi trường, cấu hình, activation, monitoring và rollback |
| **Test & Acceptance Pack** | Dataset, rubric, test cases, results và qualification |
| **Known Limitations & Escalation** | Điều kiện chưa đáp ứng, dấu hiệu dừng và owner tiếp nhận |

Mỗi bàn giao cần có version, người nhận, tiêu chí chấp nhận và vấn đề còn mở. Việc gửi tài liệu hoặc merge code chỉ là một phần của trạng thái hoàn tất.

---

**14. Liên kết thiết kế với chương trình phát triển L1–L9**

L1–L9 trong workbook hiện tại là các nhóm công việc và chi phí. Chúng bổ sung góc nhìn đầu tư cho kiến trúc.

| Ledger | Nội dung |
| --- | --- |
| **L1** | Phát triển và sửa lớn shared family, mental model, knowledge và implementation |
| **L2** | Regression cho shared family/profile |
| **L3** | Kiểm thử journey, tích hợp xuyên sản phẩm và recovery |
| **L4** | Phát triển, knowledge, qualification và regression riêng của package |
| **L5** | Technical enablement, showcase và POC trong phạm vi Xora |
| **L6** | Replicated configuration và binding adaptation được tiếp nhận thành product work |
| **L7** | Private runtime/model adaptation và tuning được chọn |
| **L8** | Learning mechanisms: memory/version/evidence lifecycle, promotion và rollback |
| **L9** | Hạ tầng R&D nền dùng chung |

Các quy tắc quy đổi cần được giữ:

- Family count, package count và journey count là các đơn vị danh mục khác nhau.
- Profile là đơn vị cấu hình, không phải tenant.
- POC campaign không tự tạo ra customer win.
- Worker/judge calls là workload, không phải số agent.
- Dataset được dùng lại qua version và binding; không nhân bản chỉ vì có thêm profile.
- Private deployment không tự phát sinh tuning.
- Availability, planned qualification và acceptance là các trạng thái khác nhau.
- Customer discovery/delivery và production operations giữ đúng ranh giới ngoài ngân sách dev/test của workbook.

Mỗi campaign cần liên kết được `WorkPackage_ID`, asset/profile versions, dataset, mục tiêu thử nghiệm, điều kiện dừng, evidence và ledger chịu chi phí.

---

**15. Quản trị thư viện thuật ngữ và áp dụng vào công việc hằng ngày**

Mỗi thuật ngữ trong thư viện nên có một bản ghi chuẩn:

| Trường | Nội dung |
| --- | --- |
| `Term_ID` | Định danh ổn định |
| `Canonical_Name` | Tên chuẩn dùng trong thiết kế và contract |
| `Vietnamese_Definition` | Định nghĩa tiếng Việt |
| `Scope` | Business, architecture, data, model, runtime hoặc governance |
| `Example` | Ví dụ đúng trong Xora |
| `Related_Terms` | Các thuật ngữ liên quan |
| `Common_Confusions` | Những nghĩa dễ bị nhầm |
| `Owner` | Vai trò chịu trách nhiệm |
| `Version / Status` | Proposed, Agreed hoặc Deprecated |
| `References` | Contract, diagram, schema và tài liệu liên quan |

Product Architect quản lý tính nhất quán của thư viện. SME xác nhận ngữ nghĩa nghiệp vụ; Data Architect xác nhận dữ liệu; DS xác nhận model/learning; Technical Lead xác nhận cách hiện thực. Thay đổi định nghĩa có ảnh hưởng phải được truy đến contract, dataset, test và implementation liên quan.

**Cách diễn đạt yêu cầu đủ rõ để giao việc**

| Yêu cầu còn mơ hồ | Cách viết chuẩn hóa |
| --- | --- |
| “Train mental model” | “Cập nhật family knowledge/skill” hoặc “fine-tune model” — chọn đúng loại thay đổi |
| “Thêm memory cho agent” | Nêu nội dung, scope, vòng đời, điều kiện ghi/đọc và tiêu chí đánh giá |
| “Làm model riêng cho bank” | Nêu riêng endpoint, tài nguyên, adapter hay trọng số |
| “Agent đã học lỗi này” | Nêu artifact đã thay đổi, version, evaluation và phạm vi triển khai |
| “Case đã approve” | Nêu approve báo cáo, RCA, hành động hay recovery |
| “Profile đã pass” | Nêu asset version, profile version, test suite và qualification receipt |
| “Dữ liệu đã đủ” | Nêu đủ cho nhiệm vụ nào, grain nào, nguồn nào và tiêu chí nào |
| “Deploy xong” | Nêu môi trường, version, activation và trạng thái field validation |

Một yêu cầu chuẩn có thể viết:

> Phát triển phiên bản mới của package IT Service Investigation trên profile được chọn, nhằm cải thiện việc phân biệt lỗi ứng dụng và dependency khi có triệu chứng HTTP 503. Sử dụng evidence tại tenant, giữ RCA ở trạng thái chưa xác định khi thiếu căn cứ, đánh giá trên tập case độc lập và bàn giao đầy đủ contract, skill, qualification evidence cùng deployment manifest.

Trong phase 1, Resolve và XBrain có thể vận hành độc lập theo các contract chung. Việc chuyển sang runtime dùng chung cần qualification riêng về tích hợp, isolation, xử lý lỗi, versioning, authority và rollback.
