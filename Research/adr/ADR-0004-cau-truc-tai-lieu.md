# ADR-0004: Cấu trúc tài liệu & nơi lưu deliverable

- **Trạng thái:** ACCEPTED (một phần là ASSUMPTION — chưa được user confirm do câu hỏi timeout)
- **Ngày chốt:** 25/09/2026
- **Người quyết định:** Nghĩa (Tester #4) + AI (đề xuất mặc định)
- **Bị ảnh hưởng:** Kho `D:\Doc\Research\`

## Bối cảnh

Kế hoạch + checklist phải lưu dạng nào, ở đâu? Repo có sẵn convention `Task_1..Task_4_*.md` trong `D:\Doc\Research\`. Skill `grill-with-docs` bắt buộc phải tạo thêm ADR và Glossary — chưa rõ để riêng hay gộp.

## Quyết định

1. **Kế hoạch + Checklist gộp trong 1 file Markdown** duy nhất: `D:\Doc\Research\Task_5_Shift_Left_Verification_Plan_and_Checklist.md` (theo convention `Task_*`; đánh số 5 để không đè `Task_4_Test_Plan_Strategy_Evaluation.md`).
2. **ADR để file riêng**, đặt tại `D:\Doc\Research\adr\ADR-00xx-*.md`.
3. **Glossary để file riêng**: `D:\Doc\Research\GLOSSARY_TI.md`.

## Hậu quả

- (+) 1 file checklist in ra / mở song song khi review được, không phải nhảy nhiều file.
- (+) ADR độc lập → sau này thay quy trình không phải sửa lại checklist.
- (−) *Assumption chưa confirm:* nếu user muốn gộp ADR/Glossary vào Task_5 thì chỉ cần move nội dung, không ảnh hưởng logic.

## Tham chiếu

Task_5 §1.4 (giảm định A1, A2) · convention đặt tên `Task_*` trong `D:\Doc\Research\`.
