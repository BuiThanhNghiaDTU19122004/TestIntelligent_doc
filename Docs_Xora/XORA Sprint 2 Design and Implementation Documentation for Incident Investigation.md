# XORA Sprint 2 Design and Implementation Documentation for Incident Investigation

**XORA — TÀI LIỆU THIẾT KẾ VÀ TRIỂN KHAI SPRINT 2**

**Phiên bản:** Đề xuất v1.0 — 25/09/2026  
**Phạm vi:** Incident Investigation; agent skill; memory; knowledge base; feedback; evaluation; đóng gói và phát hành.  
**Đối tượng sử dụng:** Product, SME, Product Architect, Data Architect, Data Scientist, Data Analyst, Data Engineer, Agentic Development, Technical Lead, QA, DevOps/MLOps, XBrain và XoraOps.  
**Giả định lập kế hoạch:** Sprint gồm 10 ngày làm việc. Các mã object, asset, operation và manifest trong tài liệu là thiết kế đề xuất.

---

1. **Sprint 2 tạo ra một năng lực điều tra có thể tái sử dụng, cùng một vòng cải tiến có bằng chứng.**

Mục tiêu là phát triển sản phẩm từ **business intent, scenario và đặc tả năng lực dùng chung**; sau đó áp dụng vào từng môi trường thông qua integration profile và tenant binding.

Đầu ra cuối sprint:

> Một package điều tra thực hiện được một scenario trên hai integration profile; sử dụng tools, deterministic rules và HITL hiện có; ghi lại quá trình xử lý và outcome; tạo learning candidate; kiểm thử và phát hành một phiên bản tài sản có kiểm soát.

Sprint cần trả lời được các câu hỏi:

- Agent đang xử lý đúng intent và đúng phạm vi hay chưa?
- Khi kết quả không phù hợp, có xác định được bước phát sinh vấn đề và căn cứ đánh giá không?
- Cùng một phương pháp có áp dụng được trên các profile khác nhau không?
- Memory có giúp cải thiện một quyết định cụ thể hay làm kết quả kém đi?
- Khi tri thức hoặc hệ thống thay đổi, memory và KB được cập nhật, thay thế và thu hồi thế nào?
- Một bài học được chuyển thành thay đổi trong KB, skill, rule hoặc software bằng quy trình nào?

Chất lượng và phạm vi đạt được phải được ghi trong qualification record. Kết quả lab, kiểm thử contract và kết quả trên môi trường khách hàng là các mức bằng chứng riêng.

1. **Baseline phải được ghi nhận đúng trước khi triển khai.**

Thông tin dưới đây tổng hợp từ trao đổi demo và tài liệu hiện trạng được cung cấp. Các nhận định về repository là theo tài liệu đó; tài liệu Sprint 2 này không phải kết quả kiểm tra trực tiếp mã nguồn. Pasted text(4)

| Thành phần | Baseline được mô tả | Cách sử dụng trong Sprint 2 |
| --- | --- | --- |
| **Xora AIOps** | Python, LangGraph, Bedrock; đang dùng trong POC; một số khả năng phụ thuộc feature flags | Giữ luồng hiện tại; bổ sung adapter và feedback theo contract chung |
| **Xora Resolve** | Go, AgentCore, Temporal; có sáu capability; đang construction và chạy thử nội bộ | Nền thực thi cho agent skill trong phạm vi lab được xác nhận |
| **Tools** | Đã có | Kiểm tra inventory, contract, quyền và tình trạng kết nối theo môi trường |
| **Evidence-based rules** | Đã có, chạy deterministic | Giữ vai trò kiểm tra điều kiện và ra quyết định theo baseline |
| **Agent skill** | Chưa được đóng gói theo nghĩa đã thống nhất | Xây một skill có procedure, dependencies, constraints và evaluation |
| **HITL** | Có wait, accept, reject, escalate và timeout chuyển cấp | Bổ sung outcome contract, lineage và kiểm thử trạng thái |
| **AgentCore** | Quản trị runtime/vòng đời agent; invocation/response; cơ chế memory | Tiếp tục sử dụng trong phạm vi cấu hình hiện có |
| **Temporal và workers trên ECS** | Thành phần điều phối và thực thi | Tích hợp episode, feedback, learning events và lifecycle tasks |
| **Gateway** | Có admission/policy và quan hệ với tools; tài liệu cũng đề cập AgentCore Gateway | Ngày đầu sprint xác nhận topology thực tế và trách nhiệm từng gateway |
| **Memory** | Chủ yếu ngắn hạn, session và sổ điều tra; chưa có tái sử dụng dài hạn đầy đủ | Bổ sung product experience, episode/candidate contracts và thử nghiệm memory trong lab |
| **KB** | Có tri thức tĩnh, taxonomy/câu hỏi SRE và phương pháp RCA bản nháp | Chọn một phần để chuẩn hóa thành knowledge package có phiên bản |
| **Outcome learning** | Chưa có contract và dữ liệu outcome đầy đủ để xác nhận RCA đúng/sai | Bổ sung feedback và register có trạng thái xác minh |
| **AgentCore long-term memory** | Tài liệu ghi nhận đang bị chặn có chủ đích trong IaC | Giữ ràng buộc hiện tại đến khi có quyết định kiến trúc và kiểm thử tương ứng |

Ngày đầu sprint phải lập **Baseline Manifest**, tối thiểu gồm:

- Repository, commit, image và deployment version.
- Environment và tenant/profile dùng để kiểm thử.
- Model/prompt configuration và feature flags.
- Tools đã đăng ký, tools thực sự kết nối và quyền đang có.
- Gateway topology và danh tính thực hiện tool call.
- Workflow/capability đang được bật.
- Nơi lưu case, evidence, session, episode và audit.
- Bằng chứng kiểm thử hiện có.

Trạng thái mỗi khả năng được ghi riêng: **đã có code, đã bật, đã tích hợp, đã kiểm thử, đã qualified**.

Hiện chưa có failure record đã xác minh được cung cấp trong cuộc trao đổi để khẳng định một lỗi agent cụ thể. Các ví dụ lỗi trong tài liệu này là scenario kiểm thử đề xuất.

1. **Phạm vi lõi được giới hạn để hoàn thành một luồng đầy đủ.**

| Hạng mục | Phạm vi P0 của Sprint 2 |
| --- | --- |
| Business scenario | Một scenario family: dịch vụ tăng lỗi 5xx sau thay đổi |
| Agent skill | Một diagnostic skill sử dụng capability hiện có |
| Integration profiles | Hai profile; tối thiểu một profile sử dụng nguồn/tools đang hoạt động |
| Product KB | Một phương pháp RCA cùng các references cần cho skill |
| Tenant/profile KB | Source inventory, entity mapping và tài liệu tối thiểu cho scenario |
| Contracts | Context, evidence, step record, episode, feedback, escalation và learning candidate |
| Memory | Product experience hoạt động; episode và tenant candidates; thử nghiệm đọc memory đã duyệt trong lab |
| Learning loop | Một candidate đi qua phân tích, evaluation và quy trình phát hành |
| Lifecycle | Thử nghiệm một lần cập nhật/thay thế hoặc thu hồi asset |
| XoraOps | Release manifest, qualification scope, activation record và rollback evidence |

Các phần mở rộng được quản lý riêng:

- Tenant long-term memory trực tuyến trong môi trường vận hành.
- AgentCore long-term memory strategy.
- Semantic/vector retrieval nếu chưa có căn cứ cần thiết.
- Connector mới phức tạp hoặc tích hợp vendor thứ hai hoàn chỉnh.
- Fine-tuning model.
- Agent tự thực thi remediation.
- Hợp nhất toàn bộ runtime hoặc di chuyển toàn bộ lịch sử AIOps sang Resolve.

P0 vẫn phải có thử nghiệm tái sử dụng memory trong lab. Việc mở cho tenant vận hành phụ thuộc quyết định kiến trúc, quyền dữ liệu và qualification tương ứng.

1. **Đơn vị thiết kế phải thống nhất để các nhóm đóng gói cùng một sản phẩm.**

| Đơn vị | Định nghĩa sử dụng trong Sprint 2 |
| --- | --- |
| **Business UC** | Bài toán và outcome khách hàng cần đạt |
| **Intent** | Mục tiêu cụ thể của một yêu cầu, cùng đối tượng và phạm vi xử lý |
| **Scenario** | Tình huống có điều kiện, nguồn thông tin, hành vi mong đợi và tiêu chí đánh giá |
| **Capability** | Khả năng thực hiện một chức năng với input/output và giới hạn rõ |
| **Family** | Nhóm capability dùng chung khái niệm, phương pháp và bộ đánh giá |
| **Mental/domain model** | Mô hình khái niệm và quan hệ giúp diễn giải bài toán |
| **Agent** | Thành phần xử lý tác vụ trong runtime, có vai trò, context, quyền và contract |
| **Agent skill** | Phương pháp thực hiện một tác vụ, gồm procedure/instructions, dependencies, constraints và evaluation |
| **Rule** | Logic điều kiện xác định, có input, output và phiên bản |
| **Tool** | Một thao tác có thể được gọi qua contract |
| **Connector** | Kết nối tới nguồn/hệ thống |
| **Adapter** | Ánh xạ interface và semantics giữa nguồn thực tế với contract chung |
| **Integration profile** | Cấu hình đại diện cho một nhóm môi trường cần phát triển và kiểm thử |
| **Tenant binding** | Ánh xạ package/profile tới identity, nguồn, tools, quyền và tài sản cụ thể của tenant |
| **UC package** | Gói capability, skill, workflow, rule, KB và policy để phục vụ một UC |
| **Journey** | Luồng đầu cuối kết hợp package và bước của con người |
| **Qualification** | Kết luận về một phiên bản trong một phạm vi profile dựa trên bộ bằng chứng xác định |

Sprint 2 hiện thực UC A1 — Incident Investigation. Cấu trúc đóng gói có thể dùng lại cho các UC dữ liệu, tri thức hoặc MLOps, nhưng các UC đó có scenario và evaluation riêng.

1. **Kiến trúc gồm các component logic với trách nhiệm rõ ràng.**

Một component logic có thể được hiện thực trong service hoặc worker đang có; tài liệu không yêu cầu tạo một microservice cho mỗi dòng.

| Component | Trách nhiệm | Output chính |
| --- | --- | --- |
| **Intent & Context Resolver** | Chuẩn hóa intent; lấy trusted tenant context; xác định case, thực thể và profile | ExecutionContext, normalized intent |
| **Authority/Admission** | Kiểm tra quyền sử dụng capability/tool và giới hạn thực thi | Allow/deny/approval decision |
| **Workflow Engine** | Quản lý bước xử lý, retries, timers và HITL | Workflow state, activity events |
| **Agent Runtime** | Invoke capability với context và asset versions được cấp | Structured agent result |
| **Skill Resolver** | Chọn skill version tương thích intent/profile | Skill execution configuration |
| **Tool Resolver/Gateway** | Ánh xạ logical capability sang tool binding; kiểm tra và thực hiện lời gọi | Tool result, execution status |
| **Evidence Builder** | Chuẩn hóa nguồn, thời gian, chất lượng và references | EvidenceBundle |
| **Rule/Evidence Evaluator** | Áp dụng rules; đánh giá điều kiện và thiếu hụt evidence | Rule results, evidence assessment |
| **Context Service** | Lấy KB và memory đúng quyền, hiệu lực và mục đích | Context package và lineage |
| **Memory Controller** | Phân loại item; quyết định ghi/candidate/eligibility theo policy | Memory decisions và records |
| **Episode/Feedback Store** | Lưu lịch sử điều tra và kết quả xác minh | InvestigationEpisode, OutcomeFeedback |
| **Learning Intake — XBrain** | Tiếp nhận vấn đề, phân loại, tạo candidate và work package | LearningCandidate |
| **Evaluation Runner** | Chạy bộ ca với các configuration được ghim phiên bản | Evaluation report |
| **Asset/Release Registry — XoraOps** | Quản trị versions, approval, qualification và activation | Release manifest, audit và rollback records |

Admission/policy và tool dispatch cần được phân biệt về trách nhiệm. Nếu môi trường sử dụng gateway riêng cùng AgentCore Gateway, Baseline Manifest phải ghi rõ đường đi và identity của từng bước.

1. **Luồng thực thi giữ quyền quyết định ở engine và policy.**

Luồng đề xuất:

1. Nhận alert hoặc yêu cầu điều tra.
2. Xác thực principal, tenant, environment và quyền.
3. Chuẩn hóa intent và chọn package/profile/binding.
4. Resolve skill, rule, KB và memory policy versions.
5. Tạo context từ evidence hiện tại và các tài sản đủ điều kiện.
6. Agent đề xuất giả thuyết, evidence cần có hoặc tool intent.
7. Engine/gateway kiểm tra quyền, binding, arguments và budget trước khi gọi tool.
8. Chuẩn hóa kết quả thành EvidenceBundle.
9. Rules và capability kiểm chứng đánh giá evidence.
10. Agent tạo assessment/recommendation có references.
11. Workflow thực hiện HITL bằng logic hiện có.
12. Ghi episode, review decision và outcome đã biết.
13. Tạo escalation hoặc learning candidate khi đáp ứng policy.
14. Feedback bổ sung có thể cập nhật sau khi workflow kết thúc.

Thông tin trong log, ticket, file, memory hoặc tool result là dữ liệu đầu vào. Nó không có quyền thay đổi authorization hoặc cấp thêm tool access.

Agent đề xuất tool intent; engine thực hiện quyền gọi tool. Agent không tự ghi trạng thái “verified”, tự kích hoạt memory hoặc tự phê duyệt release.

1. **Agent skill được đưa vào sáu capability hiện có theo trách nhiệm.**

| Capability | Trách nhiệm trong Sprint 2 |
| --- | --- |
| `INVESTIGATION_SUPERVISOR` | Chuyển intent thành kế hoạch có giới hạn; xác định evidence requirements |
| `EVIDENCE_INVESTIGATOR` | Thực hiện procedure của skill; đề xuất phép kiểm tra và tool intents |
| `EVIDENCE_VERIFIER` | Đánh giá nguồn, độ đầy đủ, thời gian và mâu thuẫn của evidence |
| `RCA_ANALYST` | Tạo RCA candidate hoặc từ chối kết luận; liên kết claim với evidence |
| `RUNBOOK_ADVISOR` | Tìm và đề xuất runbook phù hợp; mô tả prerequisites và giới hạn |
| `INCIDENT_COPILOT` | Trình bày cho reviewer; thu reason code và thông tin cần bổ sung |

Một skill có thể được nhiều capability sử dụng. Việc tạo skill không mặc định tăng số capability hoặc tạo thêm runtime.

Vai trò “supervisor” trong workflow này cũng không tự động có nghĩa nền tảng đã chuyển sang một chế độ thực thi tự chủ khác. Mode thực thi và authority phải được khai báo riêng.

Mẫu skill đề xuất:

```
skill_id: SKILL.IR.CHANGE_RELATED_FAILURE
version: 0.1.0
status: candidate

intent: investigate_incident
scenario_family: service_errors_after_change

applicability:
  required:
    - resolved_service_identity
    - incident_time_window
  optional:
    - recent_change_reference
    - dependency_context

logical_tool_capabilities:
  - observability.logs.query
  - observability.metrics.query
  - changes.events.read
  - topology.dependencies.read

procedure_ref: procedures/change-related-failure-v1
rule_set_ref: RULESET.IR.EVIDENCE.V1
knowledge_ref: KB.RCA.METHOD.V1
memory_policy_ref: MEM.IR.V1

outputs:
  - evidence_requirements
  - proposed_tool_intents
  - hypothesis_assessments
  - missing_information
  - recommendation
  - stop_or_escalation_reason

authority:
  tool_execution: engine_only
  final_decision: engine_policy
  remediation_execution: disabled

budget_policy_ref: POLICY.IR.LAB.BUDGET
evaluation_suite_ref: EVAL.IR.CHANGE_FAILURE.V1
```

Các logical capability trên là tên đề xuất. Profile phải công bố capability nào có, không có hoặc không được phép sử dụng. Skill phải xử lý được trường hợp nguồn hoặc quyền bị thiếu.

1. **Thiết kế model cần tách mô hình khái niệm, model AI và vai trò thực thi.**

| Loại model | Vai trò | Phạm vi Sprint 2 |
| --- | --- | --- |
| **Domain/semantic model** | Định nghĩa Service, Dependency, Signal, Change, Evidence, Hypothesis và các quan hệ | Bắt buộc |
| **Data model** | Định nghĩa objects, schema, trạng thái, lineage và scope | Bắt buộc |
| **Policy/rule model** | Quyết định quyền, điều kiện evidence và memory eligibility | Bắt buộc; deterministic |
| **LLM** | Diễn giải intent, đề xuất bước kiểm tra, tổng hợp assessment có references | Dùng model đã được cấu hình và ghim version |
| **Classifier** | Hỗ trợ phân loại nội dung hoặc failure category | Có thể bắt đầu bằng taxonomy/rules; LLM chỉ đề xuất nhãn khi cần |
| **Embedding/reranker** | Hỗ trợ tìm memory/knowledge liên quan | Chỉ bổ sung khi có yêu cầu và evaluation rõ |
| **Anomaly/statistical model** | Tạo tín hiệu phát hiện bất thường | Sử dụng tín hiệu hiện có; không phải trọng tâm xây mới |
| **LLM judge** | Hỗ trợ rà soát output | Tín hiệu bổ trợ; không thay ground truth hoặc các kiểm tra bắt buộc |

Planner, worker và judge là vai trò; có thể sử dụng cùng hoặc khác model nền.

Model Manifest tối thiểu cần ghi:

- Model/deployment reference và phiên bản thực tế.
- Vai trò và task được phép.
- Prompt/instructions version.
- Structured output schema.
- Runtime parameters.
- Processing/data policy.
- Budget, timeout và hành vi khi lỗi.
- Evaluation references.
- Phạm vi profile đã được kiểm thử.

Sprint 2 ưu tiên đánh giá và cải thiện prompt, skill, rules, context và retrieval. RAG, memory update và thay đổi skill không tự thay trọng số model. Fine-tuning chỉ được đề xuất khi có dataset, mục tiêu và phép so sánh phù hợp.

1. **Dữ liệu phát triển được tổ chức theo mục đích, thay vì đưa tất cả vào memory.**

| Dataset/tập thông tin | Đơn vị record | Nguồn phù hợp |
| --- | --- | --- |
| **Scenario catalog** | Một tình huống và expected behavior | Product, SME, yêu cầu thị trường/khách hàng |
| **Evidence fixtures** | Một snapshot quan sát tại thời điểm quyết định | Lab, replay được phép, dữ liệu nội bộ |
| **Decision trajectories** | Một quyết định/bước xử lý | Agent, engine, resolver, gateway |
| **Investigation episodes** | Một lần điều tra | Workflow và evidence references |
| **Outcome/feedback records** | Một lần xác nhận hoặc phản hồi | SME, operator, kết quả lab, nguồn outcome được công nhận |
| **Experience/memory candidates** | Một bài học với applicability | Lab, feedback và episode đủ điều kiện |
| **Retrieval evaluation** | Một context need và tập item phù hợp/không phù hợp | DS và SME |
| **Acceptance benchmark** | Một ca theo profile/configuration | QA/evaluator giữ riêng |
| **Release evaluation history** | Một candidate/release cùng kết quả | Evaluation Runner và XoraOps |

Metadata dùng chung cần phân biệt:

- `origin`: lab, synthetic, replay, customer-observed.
- `verification_status`.
- `dataset_role`: development, regression, holdout.
- Source và thời điểm.
- Scenario/fixture family để kiểm soát trùng lặp.
- Tenant/scope và quyền sử dụng.
- Asset versions.
- Phạm vi chia sẻ.

Nguồn hiện có như tri thức SRE, câu hỏi kiểm chứng, RCA method, runbook và incident lịch sử có thể tạo đầu vào ban đầu. Ground truth cho lab cần dựa trên điều kiện thử đã biết và được kiểm tra; output do LLM sinh không tự trở thành nhãn đúng.

Dữ liệu thô khách hàng vẫn nằm trong processing boundary được duyệt, bao gồm inference, embeddings, logs và traces. Kết quả đánh giá tại tenant có thể được chia sẻ dưới dạng được phê duyệt mà không cần tập trung raw data.

**10. Scenario specification phải phân biệt input cho agent với đáp án của evaluator.**

Mẫu:

```
scenario_id: SCN.IR.ERRORS_AFTER_CHANGE
version: 1.0.0

business_goal:
  - assess_change_related_incident
  - identify_missing_evidence
  - recommend_next_diagnostic_step

intent:
  type: investigate_incident
  action_mode: read_and_recommend

hypothesis_families:
  - application_startup_failure
  - configuration_incompatibility
  - dependency_failure
  - observability_source_failure

profile_refs:
  - PROFILE.LAB.A
  - PROFILE.LAB.B

runtime_fixture_ref: fixtures/runtime/SCN-IR-001
evaluation_contract_ref: evaluation-only/SCN-IR-001

required_behaviors:
  - resolve_target_service
  - respect_available_sources_and_permissions
  - distinguish_observation_from_hypothesis
  - identify_missing_evidence
  - stop_when_budget_or_authority_is_exhausted

prohibited_conclusions:
  - infer_causation_from_timing_alone
  - treat_missing_data_as_normal_operation
  - equate_technical_recovery_with_business_completion
```

`evaluation_contract_ref` và các đáp án tương ứng chỉ phục vụ evaluator. Runtime context không được nhận expected RCA hoặc đáp án của holdout.

Đối với mỗi ca, evaluator có thể định nghĩa nhiều bước/tool hợp lệ, các evidence bắt buộc và các kết luận chưa được phép đưa ra.

**11. Bộ contract chung phải nối được hai hệ mà giữ nguyên quyền và lineage.**

| Contract | Trường tối thiểu |
| --- | --- |
| **ExecutionContext** | Trusted tenant/principal, environment, case/run, intent, profile/binding, policy và asset versions |
| **SourceDescriptor** | Source ID/type, capability, semantic mapping, freshness, quyền, trạng thái sẵn sàng |
| **ToolIntent** | Evidence requirement, logical capability, proposed arguments, reason code và references |
| **ToolExecutionRecord** | Intent, resolved tool/binding, actual arguments, policy decision, result/error và timestamps |
| **EvidenceBundle** | Evidence IDs, source/query/tool, observed time, collection time, chất lượng và references |
| **AgentStepResult** | Hypotheses, claims, missing evidence, next-step proposals, stop/escalation reason |
| **DecisionRecord** | Step, decision owner, input/output refs, lựa chọn, version và căn cứ kiểm tra |
| **InvestigationEpisode** | Intent, context refs, hypotheses, steps, evidence, assessments và trạng thái kết thúc |
| **OutcomeFeedback** | Decision scope, review decision, RCA verification, execution/recovery/business outcome và nguồn xác nhận |
| **EscalationEnvelope** | Failure stage/category, attempted steps, missing evidence, trạng thái xác minh và nội dung được phép chia sẻ |
| **MemoryItem** | Loại, scope, provenance, verification, applicability, allowed uses, lifecycle và version |
| **LearningCandidate** | Vấn đề, bài học, proposed change, applicability, counterexamples, evaluation và approval |
| **ReleaseManifest** | Asset versions, code/config refs, evaluation, qualification, approval và deployment bindings |

Quy tắc:

- Tenant/principal do hệ thống xác thực cung cấp.
- Evidence thiếu, không truy cập được hoặc lỗi phải có trạng thái riêng; không chuyển thành giá trị bình thường.
- Thời gian và đơn vị đo phải được chuẩn hóa.
- Source observation, hypothesis và verified conclusion là các loại nội dung khác nhau.
- Schema cần có version và quy tắc compatibility.
- Existing EvidenceBundle của Resolve được đối chiếu và mở rộng có kiểm soát.
- AIOps sử dụng adapter sang contract chung; không mặc định di chuyển toàn bộ dữ liệu hoặc dùng chung database với Resolve.

**12. Memory được phân loại theo nhiều chiều, với hai nhóm kinh nghiệm ban đầu.**

Hai nhóm v1:

| Nhóm | Nội dung | Mục đích |
| --- | --- | --- |
| **Diagnostic experience** | Điều kiện sự cố, evidence, trình tự kiểm tra, outcome và trạng thái xác minh | Gợi ý hướng kiểm tra hoặc tham khảo precedent phù hợp |
| **Failure/correction experience** | Bước thất bại, nguyên nhân đã xác minh, thay đổi đã thử và kết quả | Tránh lặp lỗi; tạo cải tiến trong skill/rule/integration |

Mỗi item còn mang các chiều riêng:

| Chiều | Ví dụ |
| --- | --- |
| Ownership/access scope | Product, domain pack, tenant, case/run |
| Domain | Incident, payment, database, deployment |
| Loại thông tin | Observation, hypothesis, outcome, feedback, lesson |
| Applicability | System/service, environment, scenario, version |
| Verification | Unverified, observed, verified, disputed, refuted |
| Allowed uses | Audit, plan hint, source hint, verified precedent, development/evaluation |
| Lifecycle | Candidate, evaluated, approved, active, superseded, revoked |
| Validity | Valid-from/to, review date và compatibility conditions |

“Reviewer reject” là một feedback event. Lý do reject quyết định nó ảnh hưởng tới nhãn hoặc phương pháp nào.

“Long-term” mô tả khả năng duy trì và sử dụng qua nhiều lần chạy. Nó không chứng minh thông tin đúng hoặc được phép dùng cho mọi mục đích.

Mẫu record minh họa:

```
memory_item_id: MEM.LAB.EXPERIENCE.001
schema_version: 1.0.0
item_version: 1

memory_type: diagnostic_experience
owner_scope: product_lab
tenant_id: null

domain: incident_investigation
scenario_family: service_errors_after_change

content_ref: experiences/lab/episode-001/lesson
provenance:
  episode_ref: LAB.EPISODE.001
  evidence_refs:
    - LAB.EVIDENCE.001
  verification_record_ref: LAB.VERIFICATION.001

applicability:
  required_conditions:
    - startup_failure_is_a_live_hypothesis
    - application_logs_are_available
  compatible_profile_refs:
    - PROFILE.LAB.A

allowed_uses:
  - lab_next_step_hint

verification_status: verified
lifecycle_status: candidate
lifecycle_policy_ref: MEM.LIFECYCLE.V1

supersedes: null
```

Đây là record do controller tạo sau khi có verification record, không phải quyền để agent tự khai báo “verified”.

`verified` và `active` là hai trạng thái khác nhau: thông tin đã được xác minh vẫn cần kiểm tra quyền, applicability và activation.

**13. Memory Controller thực hiện policy ghi; Context Service thực hiện policy đọc.**

Memory Controller xử lý từng đơn vị nội dung:

1. Kiểm tra quyền lưu và phạm vi.
2. Phân loại observation, hypothesis, recommendation, outcome hoặc lesson.
3. Xác định verification từ nguồn có thẩm quyền.
4. Kiểm tra khả năng sử dụng lại và applicability.
5. Quyết định ghi episode, tạo candidate, gộp hoặc yêu cầu review.
6. Ghi policy rule và lý do quyết định.

Một output có thể tạo nhiều hành động lưu. Ví dụ một observation được ghi vào episode, đồng thời một bài học ứng viên được đưa vào candidate store.

| Input | Routing v1 |
| --- | --- |
| Observation có nguồn | Evidence/episode; working context theo case |
| Hypothesis chưa xác minh | Working context và episode, giữ nhãn hypothesis |
| RCA có verification phù hợp | Episode và candidate cho episodic reuse |
| Tool timeout/permission denied | Failure observation, gắn profile và thời điểm |
| Reject không có reason | Feedback cần làm rõ |
| RCA bị bác bỏ có căn cứ | Failure-learning candidate và regression case |
| Bài học từ lab | Product experience candidate |
| Bài học tenant đề xuất dùng chung | Kiểm tra quyền tái sử dụng và export trước khi promotion |
| Kết quả trùng do retry | Deduplicate theo event/item identity |

Context Service kiểm tra trước mỗi lần sử dụng:

- Quyền hiện tại của principal và tenant.
- Allowed use của item.
- Verification và lifecycle status.
- Hiệu lực và phiên bản tương thích.
- Mức phù hợp với intent, evidence requirement và profile.
- Revocation và dependency changes.
- Source references có còn dùng được hay không.

Context trả về phải giữ riêng:

- **Current evidence:** quan sát của case hiện tại.
- **Authoritative references:** tài liệu/rule có thẩm quyền cho loại quyết định.
- **Historical precedents:** kinh nghiệm quá khứ để tham khảo.

Các nhóm này không được hợp thành một mức “confidence” chung khiến lịch sử thay thế evidence hiện tại hoặc ghi đè policy.

Scope/domain/service là metadata và điều kiện truy xuất. Những quan hệ phụ thuộc liên domain vẫn có thể được sử dụng trong phạm vi quyền.

**14. KB và memory có vòng đời cập nhật, thay thế và thu hồi riêng.**

Các sự kiện ảnh hưởng đến hiệu lực gồm:

- Tài liệu hoặc runbook có phiên bản mới.
- Tool contract, schema hoặc adapter thay đổi.
- Service/configuration/platform được nâng cấp.
- Feedback bác bỏ một kết luận.
- Nguồn bị thu hồi quyền sử dụng.
- Item hết hạn hoặc đến thời điểm review.

| Sự kiện | Hành vi |
| --- | --- |
| Có nguồn/bài học mới | Tạo version/candidate; giữ provenance |
| Candidate đạt evaluation và approval | Cho phép activation trong scope đã duyệt |
| Phiên bản mới thay thế bản cũ | Ghi `supersedes`; cập nhật binding/index theo release |
| Đến `review_after` | Tạo yêu cầu review; không tự kết luận thông tin sai |
| Hết `valid_to` hoặc xác nhận không tương thích | Ngừng sử dụng cho các mục đích không còn hợp lệ |
| Phát hiện sai hoặc thu hồi quyền | Revoke và loại khỏi retrieval phù hợp |
| Đến thời hạn retention | Archive/xóa theo policy, bao gồm index và bản sao liên quan |

Incident cũ vẫn có thể là dữ kiện lịch sử đúng. Phương pháp xử lý của incident đó có thể không còn áp dụng cho phiên bản hệ thống mới.

Run đang xử lý phải lưu asset/context versions đã sử dụng. Khi một nguồn hoặc item bị revoke, các bước sử dụng tiếp cần kiểm tra lại; kết quả đã phát sinh phải truy được để đánh giá ảnh hưởng.

Rollback chỉ được thực hiện về phiên bản còn hợp lệ. Một phiên bản đã bị thu hồi vì sai hoặc không còn quyền sử dụng không tự được kích hoạt lại khi rollback.

**15. Luồng ingest tài liệu phải giữ scope, source và version.**

Quy trình đề xuất:

1. Tiếp nhận file hoặc source reference trong vùng xử lý được phép.
2. Ghi owner, tenant, quyền sử dụng, loại nguồn và effective date.
3. Kiểm tra trùng lặp và phiên bản.
4. Trích xuất nội dung với reference tới phần tương ứng của nguồn.
5. Phân loại thành tài liệu chuẩn, mapping, incident record hoặc lesson candidate.
6. Chạy validation và review cần thiết.
7. Kích hoạt snapshot/index theo scope được duyệt.
8. Tìm các references phụ thuộc vào phiên bản cũ khi cập nhật.

| Nguồn | Đích chính |
| --- | --- |
| Runbook | KB/procedural reference |
| Lịch sử chạy runbook | Episode và experience |
| Kiến trúc, entity mapping | Tenant KB/integration profile |
| Incident/postmortem | Episode và memory candidate |
| Feedback | OutcomeFeedback/learning candidate |
| Phương pháp SRE chung | Product/domain KB |

Dữ liệu được nhập không tự động trở thành active memory.

Một snapshot tái chạy cần chứa hoặc tham chiếu tới dữ liệu còn truy cập được trong phạm vi được phép. Hash/reference hỗ trợ kiểm toán nhưng không tự bảo đảm replay nếu nguồn đã thay đổi hoặc không còn tồn tại.

**16. HITL, RCA verification và recovery phải có trạng thái độc lập.**

OutcomeFeedback cần hỗ trợ tối thiểu:

| Trục | Ý nghĩa |
| --- | --- |
| Workflow status | Case/run đang ở bước nào |
| Review decision | Accept, reject, request information, escalate hoặc timeout |
| Decision scope | Review áp dụng cho RCA candidate, recommendation hay đóng bước/case |
| Diagnosis status | Unknown, hypothesis, verified, disputed hoặc refuted |
| Action approval | Đề xuất hành động có được phép thực hiện không |
| Execution status | Chưa thực hiện, bị chặn, thất bại hoặc thành công |
| Technical recovery | Unknown, partial hoặc verified |
| Business outcome | Unknown, partial hoặc verified theo định nghĩa nghiệp vụ |

Trong Sprint 2, agent vẫn ở chế độ chỉ đọc và đề xuất. Outcome về hành động thủ công của người vận hành có thể được ghi nhận qua feedback có căn cứ.

Mẫu:

```
feedback_id: FB.LAB.001
episode_ref: LAB.EPISODE.001
schema_version: 1.0.0

review:
  decision_scope: recommendation
  decision: accepted
  reason_code: ACCEPTED_FOR_FURTHER_REVIEW

diagnosis:
  status: hypothesis
  verification_record_ref: null

action:
  approval_status: not_requested
  execution_status: not_attempted

recovery:
  technical_status: unknown
  business_status: unknown

provenance:
  actor_ref: authenticated-reviewer-reference
  evidence_refs: []
```

Mẫu trên cho thấy một recommendation được accept vẫn có thể chưa có RCA được xác minh và chưa có recovery.

Review đến muộn hoặc trùng lặp phải được kiểm tra theo review task/version hiện hành. Timer và thao tác người dùng cạnh tranh phải tạo một chuyển trạng thái hợp lệ, có audit.

**17. Muốn tìm bước sai, hệ thống phải có decision records ngoài các chỉ số tracing.**

Mỗi bước cần ghi:

- `case_id`, `run_id`, `step_id`, `parent_step_id`.
- `decision_owner`: agent, engine, resolver, gateway, reviewer hoặc evaluator.
- Input/output references.
- Intent và evidence requirement.
- Proposed capability/tool và tool/binding thực tế.
- Policy decision, arguments, result/error.
- Evidence được dùng và bị loại.
- Model, prompt, skill, rule, KB, memory và profile versions.
- Trạng thái kiểm chứng, latency và mức sử dụng tài nguyên.

Không cần lưu mọi payload vào hệ thống tập trung. Payload, prompt/output và trace content phải theo data policy của môi trường; central telemetry có thể chỉ nhận metadata được phép.

Báo cáo lỗi cần phân biệt:

| Trường | Nội dung |
| --- | --- |
| `detected_stage` | Điểm quan sát được sự lệch |
| `suspected_cause` | Nguyên nhân đang được giả định |
| `verified_cause` | Nguyên nhân đã kiểm chứng, nếu có |
| `verification_status` | Mức độ xác nhận |
| `affected_asset_refs` | Tool, binding, skill, rule, KB hoặc model liên quan |
| `recommended_change_type` | Nơi cần sửa |

Taxonomy đề xuất cho v1:

- Intent/context.
- Knowledge/memory retrieval.
- Evidence planning.
- Tool resolution/binding.
- Policy/access/execution.
- Evidence quality/availability.
- Claim/RCA decision.
- Feedback/outcome verification.

Taxonomy này cần được ánh xạ với mã lỗi hiện có ở ngày đầu sprint.

**RCA của incident và nguyên nhân thất bại của quá trình điều tra là hai đối tượng riêng.** Tool thiếu quyền không chứng minh nguyên nhân của incident. Kết luận không được reviewer chấp nhận cũng chưa đủ để gán nhãn RCA sai.

Điểm lệch đầu tiên là căn cứ định hướng kiểm tra. Việc xác nhận nguyên nhân lỗi xử lý có thể cần replay hoặc thay đổi có kiểm soát một thành phần.

**18. Các operation và event cần bảo đảm quyền, idempotency và lịch sử cập nhật.**

Đây là operation logic đề xuất, không khẳng định các endpoint tương ứng đã tồn tại.

| Operation | Actor chính | Kiểm soát bắt buộc |
| --- | --- | --- |
| `resolve_task_context` | Runtime worker | Trusted tenant scope, policy và version compatibility |
| `record_step_result` | Engine/worker | Schema, identity, idempotency và provenance |
| `record_episode` | Workflow worker | Ghi bền vững trước cleanup session |
| `submit_outcome_feedback` | Reviewer/authorized integration | Decision scope, actor, evidence và expected version |
| `submit_escalation` | Engine/intake | Export policy và nội dung allowlisted |
| `propose_learning_candidate` | XBrain/authorized processor | Source, scope, change hypothesis và verification |
| `evaluate_candidate` | Evaluation Runner | Dataset/version isolation và result lineage |
| `approve_asset` | Release authority | Evidence và separation of responsibilities |
| `activate_binding` | Deployment/XoraOps service | Compatibility, scope và approved release |
| `revoke_asset` | Authorized owner | Audit, dependency impact và retrieval invalidation |

Event envelope tối thiểu:

- Event ID/type/schema version.
- Producer, time và correlation references.
- Scope và object references.
- Payload classification.
- Idempotency information.

Ghi episode, feedback và event phải chịu được retry. Nếu dùng cơ chế phát event sau khi ghi database, cần có khả năng phục hồi khi một bước thành công còn bước kia thất bại.

Duplicate event không được tạo thêm một “bằng chứng độc lập” hoặc tăng số lần xác nhận một bài học.

**19. Evaluation phải tách hiệu quả của engineering, skill và memory.**

Ba configuration:

| Configuration | Nội dung | Phép so sánh |
| --- | --- | --- |
| **A** | Baseline đã freeze | Chất lượng ban đầu |
| **B** | Contract/skill/method đã cải tiến; memory chưa tham gia quyết định | A–B: hiệu quả engineering/phương pháp |
| **C** | Cùng versions và cấu hình với B; thêm approved lab memory | B–C: ảnh hưởng riêng của memory |

B và C phải giữ nguyên các thành phần khác có thể ảnh hưởng kết quả. Context manifest cần ghi chính xác memory items đã được cấp.

Bộ dữ liệu khởi điểm đề xuất:

- 12 ca phát triển.
- 12 ca holdout do QA/evaluator giữ riêng.
- Hai integration profile.
- Mandatory contract/policy/lifecycle tests ngoài các ca chẩn đoán.

Số ca này phục vụ acceptance ban đầu, chưa đủ để khẳng định độ chính xác toàn sản phẩm.

Các nhóm ca phải có:

- Evidence đủ.
- Tương quan thời gian nhưng chưa đủ căn cứ nhân quả.
- Evidence thiếu, cũ hoặc mâu thuẫn.
- Tool timeout, lỗi hoặc thiếu quyền.
- Profile có tên/schema/labels khác.
- Memory phù hợp với case mới cùng pattern.
- Case gần giống nhưng memory không áp dụng.
- Memory sai scope, hết hiệu lực hoặc bị revoke.
- Input/tool content chứa chỉ thị không được tin cậy.
- Feedback muộn, trùng hoặc cạnh tranh với timeout.

Tránh leakage bằng cách:

- Chỉ cấp evidence có tại thời điểm quyết định.
- Giữ expected answers ngoài runtime.
- Không đưa holdout vào memory/context.
- Kiểm soát trùng theo incident/fixture family.
- Freeze dataset, KB và memory snapshots.
- Ghi rõ nguồn lab, replay và customer-observed.

Chỉ số:

| Nhóm | Chỉ số |
| --- | --- |
| Intent/plan | Đúng scope; evidence requirements phù hợp |
| Tool use | Lựa chọn capability phù hợp; sai arguments/binding; tool calls thừa |
| Evidence | Đầy đủ, freshness, provenance và xử lý missing data |
| Decision | Claim có căn cứ; kết luận đúng; dừng/abstain đúng |
| Memory | Retrieval phù hợp; sử dụng nhầm; ảnh hưởng tới kết quả |
| Runtime | Time-to-evidence, latency, lỗi và mức sử dụng |
| Governance | Traceability, scope isolation, approval và rollback |

Model judge là tín hiệu phụ trợ. Hard gates được kiểm tra bằng contract/policy tests; ground truth nghiệp vụ dựa trên điều kiện lab và xác nhận của SME/evaluator.

Nếu C không cải thiện hoặc làm giảm chất lượng, memory chưa đủ căn cứ để được kích hoạt ngoài phạm vi thử nghiệm. Kết quả âm vẫn là đầu ra nghiên cứu hợp lệ và phải được ghi nhận.

**20. Đóng gói phải tách tài sản dùng chung, cấu hình tenant và bằng chứng qualification.**

| Gói | Nội dung |
| --- | --- |
| **Core capability package** | Contracts, workflow references, skill, rules, instructions, product KB references |
| **Integration profile package** | Adapter mappings, source requirements, tool capability bindings và config schema |
| **Tenant overlay** | Entity mapping, quyền, source references, tenant KB references và environment configuration |
| **Evaluation package** | Fixtures, evaluator-only expectations, regression suites và reports |
| **Release/qualification package** | Manifest, approved versions, profile scope, evidence, deployment và rollback guide |

Secrets và credentials được resolve qua cơ chế quản trị secrets của môi trường. Evaluation answers và raw customer payload không được đóng vào runtime package dùng chung.

Cấu trúc asset đề xuất:

| Path logic | Nội dung |
| --- | --- |
| `contracts/` | Schemas và compatibility tests |
| `skills/` | Skill definitions và procedure references |
| `rules/` | Rule definitions/code references và tests |
| `knowledge/` | Knowledge manifests, nguồn và snapshots |
| `prompts/` | Versioned instructions/templates |
| `profiles/` | Profile definitions và adapter requirements |
| `workflows/` | Workflow configuration/references |
| `evaluation/` | Bộ đánh giá tách quyền truy cập |
| `deployment/` | Config, migration và IaC references |
| `release/` | Manifest, approval, qualification và known limitations |

Mẫu Release Manifest:

```
package_id: PKG.IR.SPRINT2
package_version: 0.1.0
release_state: candidate

assets:
  scenario_ref: SCN.IR.ERRORS_AFTER_CHANGE@1.0.0
  skill_ref: SKILL.IR.CHANGE_RELATED_FAILURE@0.1.0
  rule_set_ref: RULESET.IR.EVIDENCE.V1
  knowledge_ref: KB.RCA.METHOD.V1
  memory_policy_ref: MEM.IR.V1
  contract_bundle_ref: CONTRACT.IR.V1
  model_manifest_ref: MODEL.IR.LAB.V1

supported_profiles:
  - profile_ref: PROFILE.LAB.A
    qualification_type: lab_live_connector
  - profile_ref: PROFILE.LAB.B
    qualification_type: contract_replay

runtime_policy:
  action_mode: read_and_recommend
  memory_read_mode: approved_lab_only
  operational_tenant_ltm: disabled

evaluation:
  suite_ref: EVAL.IR.CHANGE_FAILURE.V1
  report_ref: populated_by_evaluation_pipeline

approval_record_ref: null
activation_records: []
```

Trường `release_state` chỉ chuyển theo approval workflow. Một release được duyệt vẫn cần activation record riêng cho từng binding/environment.

**21. Infrastructure Sprint 2 dựa trên baseline đang có và bổ sung phần cần cho luồng này.**

| Khu vực | Công việc |
| --- | --- |
| **Runtime deployment** | Ghim versions của AgentCore configuration, gateway, Temporal workflow và ECS worker |
| **Data stores** | Chọn kho được phê duyệt cho episode, feedback, product experience và candidate; triển khai migration |
| **Identity/policy** | Xác nhận agent/engine/gateway identities; quyền đọc, ghi và phê duyệt tách biệt |
| **Tenant isolation** | Kiểm soát ở database, object store, index, cache, queue, worker context và logs |
| **Network/egress** | Ghi nhận nguồn/endpoint được phép; phân biệt data plane và control/learning plane |
| **Observability** | Correlation IDs, decision records, errors, latency, token/call usage và cost attribution |
| **CI/CD** | Schema validation, contract tests, evaluation, manifest generation và deployment gates |
| **Reliability** | Retry/idempotency, worker recovery, backup/restore trong phạm vi triển khai |
| **Asset lifecycle** | Index refresh, expiry/review jobs, revoke propagation và rollback |
| **Secrets** | Credentials qua cơ chế secrets hiện hành; không nằm trong prompt hoặc package |

AgentCore long-term memory strategy không được mở bằng cách bỏ qua test đang chặn nó. Cần ADR xác định mục đích, quyền, lifecycle, cách đánh giá và thay đổi IaC tương ứng trước khi triển khai.

P0 sử dụng product experience và approved lab memory theo kiến trúc được duyệt. Tenant vận hành chỉ được mở những khả năng nằm trong qualification và deployment policy của tenant đó.

**22. Đội Product và đội phát triển cùng chịu trách nhiệm làm rõ bài toán, với đầu vào/đầu ra cụ thể.**

| Vai trò | Trách nhiệm chính | Đầu ra |
| --- | --- | --- |
| **Product Manager** | Chốt giá trị, scope và ưu tiên | Sprint goal, acceptance priorities |
| **Product Architect** | Đơn vị thiết kế, ranh giới, contracts và packaging | Architecture decisions, package/profile design |
| **SME** | Ý nghĩa nghiệp vụ, phép kiểm chứng và điều kiện áp dụng | Scenario, expected behavior, verification criteria |
| **Data Architect** | Semantic/data model, scope, lifecycle và lineage | Canonical schemas, ontology, memory/data policies |
| **Data Scientist** | Phương pháp điều tra, instructions, memory hypothesis và evaluation | Skill method, experiment design, evaluation analysis |
| **Data Analyst** | Định nghĩa chỉ số và cách tính | Metric definitions, chất lượng dữ liệu và báo cáo |
| **Data Engineer** | Ingestion, normalization, adapters, record stores và datasets | Data pipelines, fixtures, source inventory |
| **Agentic Development** | Skill integration, structured outputs và context handling | Agent implementation và unit/contract evidence |
| **Technical Lead** | Tích hợp engine/runtime, compatibility và engineering quality | Integrated flow, technical decisions |
| **QA/Evaluation** | Bộ nghiệm thu độc lập, regression và qualification | Test results, failure records, acceptance evidence |
| **DevOps/MLOps** | Deployment, IaC, observability, resource limits và release operations | Environment, manifests, operational evidence |
| **XBrain** | Intake, triage, experience curation và learning candidate lifecycle | Work packages, candidates, promotion proposals |
| **XoraOps/Release Authority** | Approval, version governance, qualification scope và activation | Release/activation/revocation records |

Product cung cấp business/customer context. Engineering phải phản hồi bằng quyết định cần chốt, evidence còn thiếu hoặc giới hạn thực thi cụ thể.

Mẫu một yêu cầu làm rõ:

- Quyết định cần đưa ra.
- Scenario/profile bị ảnh hưởng.
- Thông tin đã có.
- Thông tin hoặc quyền còn thiếu.
- Các phương án và tác động.
- Owner trả lời.
- Thời hạn.
- Hành vi khi chưa có câu trả lời: abstain, request information hoặc giới hạn scope.

**23. Backlog được chia thành các work package có dependency và tiêu chí nghiệm thu.**

| ID | Work package | Chủ trì | Dependency | Acceptance chính |
| --- | --- | --- | --- | --- |
| **S2-01** | Baseline, scenario và ADRs | Product Architect | — | Có baseline theo environment, scope và decision log |
| **S2-02** | Canonical contracts và adapters | Technical Lead | S2-01 | Hai hệ ánh xạ được dữ liệu mẫu; schema/error compatibility được kiểm tra |
| **S2-03** | Product KB và agent skill | DS | S2-01, S2-02 | Skill đề xuất bước khác nhau theo evidence và xử lý thiếu nguồn/quyền |
| **S2-04** | Source inventory và hai profiles | DE/Integration | S2-01, S2-02 | Cùng skill chạy được qua hai profile trong phạm vi công bố |
| **S2-05** | Episode, feedback và decision records | DE + Technical Lead | S2-02 | Ghi bền vững, liên kết đúng, chịu được retry và feedback muộn |
| **S2-06** | Memory Controller, lifecycle và learning intake | XBrain + Data Architect | S2-02, S2-05 | Candidate, read/write decisions và một vòng update/revoke hoạt động |
| **S2-07** | Evaluation A/B/C và qualification | QA/Evaluation | Thiết kế từ S2-01; chạy trên các package tích hợp | Có report độc lập theo profile/version; không lẫn nguồn dữ liệu |
| **S2-08** | Infrastructure, packaging và release | DevOps/MLOps | Phối hợp từ S2-01; release sau S2-07 | Có manifest, deployment evidence, activation và rollback hợp lệ |

Các work package được triển khai song song sau khi contract tối thiểu được chốt. Một thay đổi contract phải có changelog và kiểm tra ảnh hưởng đến producer/consumer.

**24. Kế hoạch 10 ngày có các điểm kiểm tra rõ.**

| Thời gian | Công việc | Điều kiện hoàn thành |
| --- | --- | --- |
| **Ngày 1** | Baseline; inventory; scope; chọn scenario và profiles | Biết môi trường nào chạy gì, quyền và nguồn nào sẵn sàng |
| **Ngày 2** | Chốt contracts, skill skeleton, memory policy và evaluation design | Có fixtures và expected behavior đủ để các nhóm cùng triển khai |
| **Ngày 3–4** | Profile A; skill/KB; episode, feedback và tracing | Chạy tới HITL và lưu đầy đủ episode |
| **Ngày 5–6** | Profile B; lỗi tool/quyền; retries; HITL edge cases | Chứng minh tính portable trong phạm vi đã công bố |
| **Ngày 7–8** | Product experience; candidate; memory lifecycle; A/B/C thử nghiệm | Có candidate truy được nguồn và kết quả so sánh |
| **Ngày 9** | Holdout/regression, qualification và đóng gói | Có report và danh sách giới hạn |
| **Ngày 10** | Demo, release review và bàn giao | Có evidence pack, activation scope và quyết định mở/giữ tắt tính năng |

Các quyết định cần chốt trước hết:

- **ADR-01:** Ranh giới contract và adapter giữa AIOps–Resolve.
- **ADR-02:** Kho episode/product experience/candidate và memory mode của lab.
- **ADR-03:** Gateway topology và authority cho tool calls.
- **ADR-04:** Source/export scope và cách chạy evaluation tại tenant.
- **ADR-05:** Package versioning, qualification và activation.

Capacity chưa đủ thì giảm số biến thể hoặc connector bổ sung. Các phần contract, evidence, feedback, evaluation và release lineage vẫn thuộc phạm vi lõi.

**25. Nghiệm thu tách hard gates khỏi kết luận về hiệu quả memory.**

Hard gates:

- Tenant/principal được lấy từ trusted context.
- Engine/gateway kiểm soát tool execution.
- Không xuất hiện vi phạm scope trong bộ kiểm thử bắt buộc.
- Missing/failed evidence được biểu diễn đúng.
- Claim truy được về evidence và asset versions.
- Episode/feedback không bị nhân đôi do retry.
- HITL xử lý được timeout và phản hồi cạnh tranh theo policy.
- Candidate chưa được duyệt không được dùng ngoài phạm vi cho phép.
- Revoked/expired assets bị loại khỏi retrieval tương ứng.
- Profile qualification được ghi đúng loại bằng chứng.
- Package và deployment có version manifest.
- Rollback không kích hoạt lại asset đã bị revoke.

Quality gates:

- Các ca quan trọng đáp ứng expected behavior đã chốt.
- Candidate không gây regression vượt mức chấp nhận được xác định trước đánh giá.
- Báo cáo phân biệt cải thiện engineering với ảnh hưởng của memory.
- Chi phí, latency và resource usage nằm trong budget của profile.
- Những kết quả chưa đủ bằng chứng được ghi rõ.

Sprint có thể hoàn thành phần thí nghiệm và đưa ra kết luận **memory chưa đủ điều kiện mở rộng**. Khi đó memory vẫn giới hạn trong lab, cùng một work package cải tiến dựa trên kết quả đo.

**26. Bài demo cuối sprint phải thể hiện trọn vòng đời sản phẩm.**

Trình tự demo:

1. Chọn intent và profile A; trình bày versions đang được sử dụng.
2. Chạy điều tra; hiển thị evidence requirement, proposed tool intent và tool thực tế.
3. Hiển thị một decision record và cách xác định bước chưa phù hợp.
4. Đi qua HITL và ghi OutcomeFeedback độc lập với approval.
5. Chạy cùng skill trên profile B; chỉ thay profile/binding.
6. Truy xuất một approved lab memory và chỉ ra purpose/applicability.
7. Chạy một ca gần giống nhưng memory không áp dụng; chứng minh item bị loại hoặc hạn chế sử dụng.
8. Tạo learning candidate từ ca phát triển.
9. Trình bày evaluation và quyết định promotion.
10. Phát hành một asset version qua manifest.
11. Cập nhật hoặc revoke một nguồn/item; chứng minh context mới phản ánh thay đổi.
12. Trình bày kết quả A/B/C và phạm vi qualification thực tế.

Bộ bàn giao cuối sprint gồm tám nhóm:

| Gói bàn giao | Nội dung |
| --- | --- |
| **Capability Card** | Mục tiêu, intent, applicability, quyền và giới hạn |
| **Architecture/Flow** | Component relationships, runtime mapping và data boundaries |
| **Input/Output Contracts** | Schemas, examples, validation và compatibility |
| **Tool Manifest** | Logical capabilities, bindings, quyền và error semantics |
| **Configuration Schema** | Profile, tenant overlay, asset references và budgets |
| **Deployment Guide** | Environments, migrations, activation, monitoring và rollback |
| **Test & Acceptance Pack** | Dataset provenance, evaluation A/B/C, hard gates và qualification scope |
| **Known Limitations & Escalation** | Nguồn chưa có, trạng thái chưa xác minh, các tính năng giới hạn và hướng xử lý |

**Điều kiện bàn giao là toàn bộ team có thể truy từ business intent tới scenario, capability, tool/evidence, quyết định, feedback, candidate và phiên bản phát hành; đồng thời xác định được thành phần nào đã chạy, đã được kiểm thử và được phép áp dụng trong phạm vi nào.**
