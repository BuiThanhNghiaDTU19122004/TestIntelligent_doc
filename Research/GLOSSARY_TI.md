# GLOSSARY_TI — Bảng thuật ngữ dùng cho xác minh 3 sơ đồ TI

**Mục đích:** chuẩn hóa từ ngữ để 3 sơ đồ (Hoàng/Hùng/Trang) và Checklist `Task_5` nói **cùng một ý** — mọi tranh luận "sơ đồ sai đúng nghĩa nào" sẽ tra tại đây.
**Ngày tạo:** 25/09/2026 · **Duy trì bởi:** Nghĩa (Tester #4) · Áp dụng: Checklist A05

## A. Trạng thái sự thật (nhãn bắt buộc — Đối soát §3.9)

| Thuật ngữ | Định nghĩa | Không được nhầm lẫn |
| :--- | :--- | :--- |
| `OBSERVED` | Kết quả **đo được trên live** tại một ngày cụ thể, kèm nguồn | ≠ "mặc định đúng"; hết hạn khi có lượt đo mới |
| `INFERRED` | Suy diễn từ mã/cấu hình, chưa đo live | ≠ OBSERVED |
| `CANDIDATE` | Đề xuất chờ phê duyệt / chờ spike chứng minh | ≠ ĐÃ CHẠY |
| `UNVERIFIED` | Có ghi nhận nhưng không tra được evidence | Không được diễn đạt thành khẳng định |
| `FROZEN` | Đường dẫn còn sống nhưng bị đóng băng (vd `/v1/testing/changes`) | ≠ bị xóa; ≠ LIVE bình thường |
| Nét liền / Nét đứt | Nét liền = đã đo được; nét đứt = khai báo/chưa có lượt đo | Legend phải ghi ngay trên sơ đồ |

## B. Ba ranh giới không được xóa (Portal — "Ba ranh giới")

| Thuật ngữ | Định nghĩa |
| :--- | :--- |
| `CANDIDATE ≠ ĐÃ CHẠY` | Case được sinh ra chưa phải đã thực thi |
| `KHUYẾN NGHỊ ≠ PHÊ DUYỆT` | S09 trả về khuyến nghị; quyền phê duyệt ở người duyệt |
| `completed ≠ PASS` (Law 18) | Job xong xử lý ≠ mọi check đạt |

## C. Pipeline & thành phần

| Thuật ngữ | Định nghĩa |
| :--- | :--- |
| S01–S10 | 10 năng lực TI: S01 Target Registry, S02 Change Detector, S03 Impact Engine, S04 Risk Engine, S05 Test Planning, S06 Candidate Generation, S07 Orchestration, S08 Evidence Store, S09 Gate Recommendation, S10 Production Learning |
| `ImpactSet` | Tập ảnh hưởng chuẩn do S03 sinh — input cho điều phối |
| `TargetBinding` / Tenant Binding | Ánh xạ tenant → phạm vi được phép chạy (URL thật, secrets) — **quyết định chạy phần nào** |
| Dispatch có lease | Giao việc kèm quyền sở hữu có hạn (lease) + heartbeat để recover khi worker chết |
| ToolIntent JSON | Yêu cầu công cụ do Harness gửi về — **không chứa credential/URL/SQL tùy ý** |
| Job Controller | Thành phần giữ "workflow authority" trên EC2 (auth, persist, governance) |
| TIJobRunner | Job runner chạy trên AgentCore Runtime, tính pipeline S02–S09 |
| Evaluation Pack | Gói mở rộng năng lực: image + oracle + evidence schema, gắn qua provider port |
| IsolatedRunner | Interface sandbox hóa runner (input image digest/command/limits → output exit code/logs/artifacts/trace) |

## D. Hạ tầng & bằng chứng

| Thuật ngữ | Định nghĩa |
| :--- | :--- |
| Receipt | Biên nhận triển khai: source commit, image digest, runtime — **mốc có ngày, ≠ bản đang chạy** |
| Lượt đo (lần đo) | Snapshot kiến trúc đo live trong một ngày trên một commit — một ngày có thể có 2 lượt |
| Evidence | Bằng chứng kết quả: raw result normalize + hash SHA-256 (Law 16) → lưu ở S08/S3 |
| Direct-to-S3 | Runner đẩy thẳng bằng chứng lên S3 + Object Lock, **không** chồng lên ổ EC2 |
| Outbox + ACK | Cơ chế ghi bền rồi chờ receiver xác nhận (HTTP 2xx); publisher phải idempotent |
| `/evidence` | Route đọc revision của tiến trình **đang phục vụ lúc gọi** |
| K1 / K2 | Hai tenant — mỗi tenant có bảng chứng riêng, không suy diễn chung |
| `runtime_binding` | Khẳng định binding runtime — hiện `UNVERIFIED` trên mọi check |

## E. Kiểm thử & đánh giá

| Thuật ngữ | Định nghĩa |
| :--- | :--- |
| Entry / Exit Criteria | Điều kiện vào / ra của từng level kiểm thử (Task 4 §6) |
| `DO_NOT_PASS` | Trạng thái cứng, không override được (Critical > 0, secret lộ, hash mismatch, rollback fail) |
| `HOLD` | Cần human review + Waiver Document để chuyển PASS |
| 6 chỉ số GenAI | Bộ chỉ số đánh giá output GenAI — gồm **Faithfulness** (bổ sung theo biên bản §4) |
| Tolerance band ±0.03 | Dải dung sai điểm khi 2 lần chạy liên tiếp ra kết quả khác (kèm temperature 0.0) |
| Faithfulness | Mức độ câu trả lời bám đúng nguồn — thiếu thì **HOLD** (Task 4 §6, Đối soát §3.9) |
| GATE_RECOMMENDATION | Kết quả PASS từ TI chỉ là khuyến nghị cổng, không phải xác nhận tuyệt đối |
| 24 Architecture Laws | 24 luật kiến trúc TI — trọng điểm: Law 5, 7, 12–16, 18, 23 |
| Law 23 / Exit note | Mỗi quyết định kèm lối thoát kỹ thuật bằng văn bản (chống lock-in) |

## F. Thuật ngữ quy trình nghiệm thu (file này & Task_5)

| Thuật ngữ | Định nghĩa |
| :--- | :--- |
| Shift-left testing (bản đồ) | Xác minh sơ đồ **tại khâu vẽ**, trước khi nó dẫn tới test case/hạ tầng |
| Vòng 1 / Vòng 2 | Review tĩnh checklist / Đối chiếu evidence |
| Evidence Pack | `URL + timestamp + kết quả/screenshot` tối thiểu cho mỗi lần kiểm |
| Blocking / Minor | Mức defect: sai hiện trạng-biên bản (sửa ≤24h) / trình bày (≤3 ngày) |
| Traceability | Khả năng truy ngược mỗi thành phần sơ đồ về ≥1 nguồn |

## Từ cấm dùng trên sơ đồ (tức là Fail)

- "đã nghiệm thu" khi **#97** còn mở · "đã chạy" khi chỉ có `CANDIDATE` · "đã thử nghiệm" khi `UNVERIFIED`.
- Tên dịch vụ **CodeGuru Security** (EOL 20/11/2025).
- "lưu bằng chứng trên EC2" như kiểu đương nhiên (phải là direct-to-S3 hoặc ghi rõ chưa chốt).
