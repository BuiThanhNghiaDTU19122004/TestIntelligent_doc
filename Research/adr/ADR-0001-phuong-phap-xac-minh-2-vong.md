# ADR-0001: Phương pháp xác minh shift-left 3 sơ đồ = Review tĩnh tại draft + Đối chiếu evidence (2 vòng)

- **Trạng thái:** ACCEPTED
- **Ngày chốt:** 25/09/2026
- **Người quyết định:** Nghĩa (Tester #4) — qua buổi grilling với AI
- **Bị ảnh hưởng:** Task_5_Shift_Left_Verification_Plan_and_Checklist.md

## Bối cảnh

Biên bản họp TI 23/09/2026 (Mục 5) giao cho Nghĩa "phụ trách mảng Shift-left testing cho 3 nội dung trên" — tức 3 sơ đồ Hoàng/Hùng/Trang sẽ vẽ sau họp. Chưa từng có quy trình xác minh sơ đồ; hiện tại sơ đồ chỉ được *tin* khi người có thẩm quyền nói nó đúng. Có 3 phương án khả dĩ:

1. Chỉ review tĩnh checklist khi draft nộp.
2. Chỉ đối chiếu sơ đồ với evidence thật (receipt, API, code).
3. Kết hợp cả hai, evidence làm vòng 2.
4. Chỉ sinh test case / tiêu chí nghiệm thu, việc review để người khác làm.

## Quyết định

Chọn **phương án 3**: **Vòng 1 = review tĩnh checklist ngay khi draft nộp** (trace về biên bản + Research docs) → **Vòng 2 = đối chiếu từng claim với evidence thật** (portal live `/diagrams` `/evidence`, receipt, API) → **Re-review ngắn** để chốt.

## Hậu quả

- (+) Phát hiện lỗi ở khâu vẽ — đúng tinh thần shift-left (Task 4 §2.1: sửa sớm rẻ hơn 10–100×).
- (+) Không cho phép claim vượt evidence sống sót trên sơ đồ.
- (−) Tốn 2 vòng thay vì 1 — chấp nhận vì stakeholder dùng sơ đồ làm căn cứ.
- (−) Vòng 2 phụ thuộc portal live có truy cập được; nếu sập → mục tương ứng = Fail/`UNVERIFIED`, không tự động Pass.

## Tham chiếu

Task 4 §2.1 (nguyên tắc Shift Left) · Portal — "Bản vẽ giải thích hệ thống, không thay bằng chứng của một lượt TI đã chạy".
