# ADR-0002: Thứ tự ưu tiên nguồn sự thật (Ground Truth Priority)

- **Trạng thái:** ACCEPTED
- **Ngày chốt:** 25/09/2026
- **Người quyết định:** Nghĩa (Tester #4)
- **Bị ảnh hưởng:** Checklist E, mọi mục có cột "Nguồn đối chiếu"

## Bối cảnh

Có ít nhất 5 loại nguồn mô tả kiến trúc TI và chúng **mâu thuẫn nhau theo thời gian**: portal live đổi nội dung theo từng lượt đo (21/09 → 22/09 → 23/09), receipt 14/09 là mốc lịch sử, bản in PDF trong repo cũ hơn, Research docs là thiết kế (CANDIDATE) chứ không phải hiện trạng. Không chốt thứ tự ưu tiên thì mỗi reviewer sẽ "chọn nguồn hợp lý hóa ý mình".

## Quyết định

Thứ tự tin cậy **giảm dần**, xung đột thì **nguồn đứng trước thắng**:

1. **Portal live** (`/diagrams`, `/evidence`, `/api`) — trạng thái đang phục vụ tại ngày review.
2. **Receipt triển khai** (14/09: `fbdd8dfc…`/`efabf57dcd8c`/Runtime 18; 23/09: `0409cb5a`, ECR `sha256:b717be38…`) — cho claim "đã triển khai".
3. **Biên bản họp TI 23/09/2026** — cho phạm vi, phân công, vấn đề mở.
4. **Research docs Task 1–4 + Báo cáo Đối soát** — cho chuẩn thiết kế & tiêu chí.
5. **Bản in portal 21/09** (`PDF/Diagram_1..4.pdf`) — lịch sử, **không** làm chuẩn hiện tại.

## Hậu quả

- (+) Mọi Fail đều chỉ được lập trên cơ sở trích nguồn đứng trước.
- (+) Ép reviewer ghi ngày đo — một ngày đo không phân biệt được hai lượt rollout (`b55fed16` vs `7dca7c0a` cùng 22/09).
- (−) Research docs (Task 2 v0.2…) có thể "thua" portal live — chấp nhận: đó là đề xuất, chưa phải hiện trạng; khác portal phải giải thích được *vì sao* khác.

## Tham chiếu

Portal — Receipt / Cách đọc · Task 2 Phụ lục A (nguồn OBSERVED) · Đối soát §3.9 (kỷ luật nhãn sự thật).
