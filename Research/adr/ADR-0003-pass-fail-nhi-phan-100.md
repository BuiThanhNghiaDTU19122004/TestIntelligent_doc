# ADR-0003: Chế độ Pass/Fail nhị phân, nghiệm thu 100%

- **Trạng thái:** ACCEPTED
- **Ngày chốt:** 25/09/2026
- **Người quyết định:** Nghĩa (Tester #4) — chủ động chọn, không đổ ngược lại AI
- **Bị ảnh hưởng:** Toàn bộ §5, §9 của Task_5

## Bối cảnh

3 phương án thang điểm được cân nhắc:

1. 4 mức Blocker/Major/Minor/Note, gate = 0 Blocker + 0 Major.
2. **Pass/Fail nhị phân từng mục, tổng thể 100% mới đạt.**
3. Chấm điểm % có trọng số (≥90% pass).

## Quyết định

Chọn **phương án 2**: mỗi mục checklist = Pass hoặc Fail; một sơ đồ đạt nghiệm thu **chỉ khi 100% mục Pass**.

Để giữ tính khả thi, **vẫn phân loại Fail** thành `Blocking` (sai hiện trạng/biên bản — sửa ≤24h) và `Minor` (trình bày — sửa ≤3 ngày). Hai loại đều là Fail, đều phải sửa, nhưng thứ tự ưu tiên khác nhau.

## Hậu quả & rủi ro đã chấp nhận

- (−) **Rủi ro chính:** ngưỡng 100% rất chặt — dễ kẹt vô hạn ở mục lặt vặt hoặc mục phụ thuộc người khác (vd: "nơi lưu script K6 chưa do người vẽ quyết"). *Giảm thiểu:* mục phụ thuộc quyết định bên thứ ba được ghi rõ "gap + owner + kế hoạch" và chỉ Pass khi việc đó được ghi trên sơ đồ chứ không đòi hoàn thành thật.
- (−) Không có % nên không so sánh được mức độ tiến bộ giữa các vòng — chấp nhận: mục tiêu là "đủ điều kiện làm căn cứ", không phải đo improvements.
- (+) Không có vùng xám → không có tranh cãi "75% là đạt chưa".

## Tham chiếu

Task_5 §5, §9 · Law 18 (`completed ≠ PASS`) — tư duy không mặc định đạt.
