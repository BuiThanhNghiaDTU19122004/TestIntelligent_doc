**Người feedback: Nguyễn Thành Đạt**

Em có một góc nhìn nhỏ muốn góp ý để team tham khảo thêm.

Theo cảm nhận của em, phần hiện tại đang đầu tư khá nhiều vào việc xây một architecture đầy đủ và chi tiết. Em nghĩ điều này cũng hoàn toàn dễ hiểu vì task team nhận được vốn là **đề xuất architecture**, sau đó đưa sang phía SA/DevOps để review. Vì vậy việc team tập trung mạnh vào kiến trúc là hợp lý trong bối cảnh được giao.

Tuy nhiên, em cảm giác vì tập trung nhiều vào architecture nên một phần giá trị rất quan trọng từ góc nhìn QA/QC chưa được thể hiện rõ bằng phần solution.

Ví dụ trước khi đi tới câu hỏi:

**“Nên dùng Fargate, Playwright, k6, S3 hay các component nào?”**

Có thể team dành thêm một bước để làm rõ:

- Business flow end-to-end hiện tại là gì?
- Trong flow đó đang có pain point hoặc quality risk nào?
- Với mỗi risk, câu hỏi quan trọng nhất mà QA/QC muốn trả lời là gì?
- Cần evidence nào để trả lời câu hỏi đó?
- Test nào sẽ tạo ra evidence có giá trị nhất?
- Kết quả test đó sẽ hỗ trợ decision nào, ví dụ PASS / HOLD / DO_NOT_PASS?

Em nghĩ đây có thể là phần mà QA/QC mang lại nhiều giá trị nhất, vì QA/QC hiểu rõ hơn **cần chứng minh điều gì về chất lượng**, còn SA/DevOps có thể hỗ trợ chuyển những requirement đó thành architecture và implementation phù hợp.

Flow em đang hình dung là:

```text
Business flow
      ↓
Pain point / Quality risk
      ↓
Quality question
      ↓
Evidence cần thu
      ↓
Test strategy
      ↓
Platform requirement
      ↓
Architecture
```

Như vậy architecture hiện tại không cần bỏ đi. Ngược lại, nó sẽ dễ giải thích và defend hơn vì từng component đều có thể trace ngược về một requirement hoặc một câu hỏi chất lượng cụ thể.

Một điểm nữa em thấy hiện tại technical flow đã khá rõ, nhưng có thể vẫn thiếu một **business end-to-end flow** đơn giản để các team cùng nhìn chung một bức tranh: từ lúc developer có một thay đổi, TI được sử dụng ở đâu, QA/QC cần biết điều gì, hệ thống tạo evidence gì, ai sử dụng kết quả đó và cuối cùng nó ảnh hưởng như thế nào tới quyết định release.

Đây chỉ là góc nhìn của em khi đọc tài liệu hiện tại, có thể em chưa nắm hết context của task ban đầu. Em nghĩ nếu bổ sung thêm lớp business flow + quality questions trước architecture thì phần proposal của team sẽ rõ hơn về cả **“tại sao cần”** chứ không chỉ **“sẽ xây như thế nào”**.