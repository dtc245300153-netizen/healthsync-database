# Báo cáo Chẩn đoán Lỗ hổng Dữ liệu (Gap Analysis) - Hệ thống HealthSync

Sau khi đối chiếu giữa **UML Activity Diagram** (quy trình nghiệp vụ thực tế) và **Legacy SQL Script** (thiết kế ban đầu của Lập trình viên), hệ thống hiện tồn tại 3 điểm vênh nghiêm trọng sau đây:

1. **Sử dụng kiểu Boolean cho quy trình đa trạng thái (Anti-pattern):**
   - *Vấn đề cũ:* Thiết kế dùng cột `is_active` kiểu `BOOLEAN` chỉ biểu diễn được 2 trạng thái (TRUE/FALSE).
   - *Thực tế nghiệp vụ:* Vòng đời lịch hẹn phải trải qua ít nhất 5 trạng thái rõ rệt (`PENDING`, `CONFIRMED`, `CHECKED_IN`, `COMPLETED`, `CANCELLED`). Việc dùng Boolean làm mất hoàn toàn khả năng theo dõi tiến trình của lịch hẹn.

2. **Thiếu hoàn toàn các thuộc tính tài chính và lý do hủy lịch:**
   - *Vấn đề cũ:* Bảng `Appointments` hoàn toàn trống trơn các thông tin về tiền cọc (`deposit_amount`), phí phạt khi hủy (`penalty_fee`) và lý do hủy (`cancel_reason`).
   - *Thực tế nghiệp vụ:* Khi khách hàng hủy lịch hoặc đặt cọc, hệ thống không có cột dữ liệu nào để lưu trữ, dẫn đến việc tính năng "Phạt tiền cọc" bị lỗi hệ thống.

3. **Sự vắng mặt hoàn toàn của bảng Quản lý Đơn thuốc (`Prescriptions`):**
   - *Vấn đề cũ:* Cấu trúc cơ sở dữ liệu cũ không hề có bảng hoặc trường dữ liệu nào dành cho bác sĩ kê đơn khi khám xong.
   - *Thực tế nghiệp vụ:* Bước "Kê đơn thuốc" là bắt buộc khi lịch hẹn đạt trạng thái `COMPLETED`. Thiếu cấu trúc này khiến tính năng kê đơn liên tục báo lỗi không thể lưu dữ liệu vào hệ thống.
