# Nhật ký Tương tác AI (AI Prompt Log) - Dự án HealthSync

Trong quá trình phân tích và tái cấu trúc cơ sở dữ liệu cho phòng khám HealthSync, tôi đã sử dụng các câu lệnh (prompt) trao đổi với trợ lý AI như sau:

1. **Thảo luận về vòng đời trạng thái và kiểu dữ liệu:**
   - *Prompt:* "Trong thiết kế cơ sở dữ liệu quan hệ, tại sao việc dùng một cột `is_active` (kiểu TINYINT/BOOLEAN) để theo dõi vòng đời của một Đơn hàng/Lịch hẹn lại là một thiết kế tồi (Anti-pattern)? Tôi nên thay thế bằng cấu trúc nào?"
   - *Kết quả trao đổi:* AI tư vấn chuyển sang dùng kiểu dữ liệu `ENUM` hoặc bảng danh mục trạng thái riêng để đảm bảo tính mở rộng và tường minh cho các trạng thái phức tạp.

2. **Lựa chọn kiểu dữ liệu tài chính:**
   - *Prompt:* "Khi thiết kế cột `deposit_amount` và `penalty_fee` trong MySQL phục vụ tính toán tài chính, tôi nên dùng kiểu dữ liệu FLOAT, DOUBLE hay DECIMAL? Tại sao?"
   - *Kết quả trao đổi:* AI giải thích rõ nhược điểm sai số dấu phẩy động của `FLOAT`/`DOUBLE` và đề xuất bắt buộc phải dùng `DECIMAL(10,2)` để đảm bảo độ chính xác tuyệt đối trong các bài toán tiền tệ.

3. **Tối ưu hóa câu lệnh truy vấn liên kết (JOIN):**
   - *Prompt:* "Viết cho tôi một câu lệnh SQL SELECT kết hợp JOIN giữa bảng Appointments và Prescriptions để lấy ra danh sách tất cả các bệnh nhân đã hoàn tất khám bệnh và chi tiết đơn thuốc của họ."
   - *Kết quả trao đổi:* AI hỗ trợ cấu trúc câu lệnh sử dụng `LEFT JOIN` kết hợp điều kiện lọc `WHERE status = 'COMPLETED'` giúp truy xuất dữ liệu chính xác và sạch sẽ.
