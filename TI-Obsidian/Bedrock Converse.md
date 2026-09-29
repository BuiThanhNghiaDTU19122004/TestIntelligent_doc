1. Converse là gì ?
- Bản chất thì nó là 1 "API chuẩn hóa" để gửi message tới model trên Amazon Bedrock và nhận response
- API chuẩn hóa: *hiểu căn bản là nó sẽ đưa tin nhắn ta vào 1 trường dữ liệu có thể là 'question; và kết hợp các trường khác ta không thấy được như 'model', 'time' vào thành có thể là 1 JSON hoàn chỉnh* 
Luồng hoạt động:
```text
							Application
							    ↓
							Converse API
							    ↓
							Bedrock Model (Claude / Nova / Llama...)
							    ↓
							Response
```