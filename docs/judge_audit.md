# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (κ=0.757 ≥ 0.6, P=0.857 ≥ 0.8, R=0.800 ≥ 0.7) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (Recall=0.300 < 0.7, κ=0.375 < 0.6) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu trả lời suồng sã ("Em đã hoàn tiền cho anh rồi nha, anh check giúp em"), thiếu tính chuyên nghiệp B2C. Judge bỏ lọt vì thấy có ý nghĩa hoàn tiền. Cần sửa rubric: phạt các đại từ xưng hô thân mật tự phát (anh/em, check giúp). |
| 2 | H-017 | PASS | FAIL | Người đúng | "Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha." Cách dùng từ "bạn ơi... rồi nha" không chuẩn mực CSKH doanh nghiệp. Judge phân loại sai. Rubric cần quy định cấu trúc câu xưng hô chuẩn của LogiX. |
| 3 | H-029 | PASS | FAIL | Người đúng | "Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm." Giọng điệu cộc lốc, gây ức chế cho khách hàng ("không cần nhắn thêm"). Judge bị đánh lừa bởi từ mở đầu "Dạ, vui lòng". Rubric cần cấm các câu mang tính từ chối giao tiếp tiêu cực. |
| 4 | H-042 | FAIL | PASS | Người đúng | "Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ." Câu trả lời lịch sự, rõ ràng thời hạn tiền về. Judge bắt lỗi giả (False Positive) do câu ngắn gọn. Rubric cần làm rõ câu ngắn nhưng đủ thông tin và lịch sự vẫn đạt PASS. |
| 5 | HL-016 | PASS | FAIL | Người đúng | "Dạ, LogiX thành thật xin lỗi anh/chị vì sự bất tiện này..." Lời xin lỗi sáo rỗng, không cung cấp giải pháp hay trạng thái đơn hàng. Judge hữu ích cho PASS do câu từ nhã nhặn, bỏ sót tính vô dụng (Recall=0.3). |

## Kết luận
- **Tiêu chí bị loại và lý do**: Loại bỏ `helpful` vì Recall cực thấp (R = 0.300) và hệ số thỏa thuận thấp (κ = 0.375 < 0.6). Ma trận nhầm lẫn `{'tp': 9, 'fp': 0, 'fn': 21, 'tn': 70}` cho thấy Judge bỏ lọt tới 21 trên tổng số 30 trường hợp vi phạm tính hữu ích (False Negative rate = 70%), biến tỷ lệ lỗi quan sát từ 0.09 thành 0.30 sau khi hiệu chỉnh Rogan-Gladen. Nếu dùng làm gate, hệ thống sẽ để lọt phần lớn các câu trả lời vô nghĩa hoặc sáo rỗng.
- **Tiêu chí được giữ lại**: `tone_ok` đạt toàn bộ các ngưỡng sàn (κ = 0.757, P = 0.857, R = 0.800), đủ độ tin cậy để làm chốt chặn chất lượng giọng điệu.
- **Rủi ro còn lại**: Bộ đánh giá chỉ mới được thử nghiệm trên dữ liệu replay đã công khai (100 mẫu). Cần thu thập tập nhãn người mới (held-out data) để kiểm tra hiện tượng quá khớp (overfitting) của prompt judge trong môi trường production.
