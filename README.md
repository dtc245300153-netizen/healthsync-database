# healthsync-database
-- =========================================================================
-- HỆ THỐNG HEALTHSYNC - OPTIMIZED DATABASE SCRIPT
-- =========================================================================

CREATE DATABASE IF NOT EXISTS healthsync_db;
USE healthsync_db;

-- Xóa bảng cũ theo thứ tự ràng buộc khóa ngoại nếu chạy lại từ đầu
DROP TABLE IF EXISTS Prescriptions;
DROP TABLE IF EXISTS Appointments;
DROP TABLE IF EXISTS Doctors;
DROP TABLE IF EXISTS Patients;

-- 1. Bảng Bệnh nhân (Patients)
CREATE TABLE Patients (
    patient_id INT AUTO_INCREMENT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    phone VARCHAR(15) NOT NULL
);

-- 2. Bảng Bác sĩ (Doctors)
CREATE TABLE Doctors (
    doctor_id INT AUTO_INCREMENT PRIMARY KEY,
    full_name VARCHAR(100) NOT NULL,
    specialty VARCHAR(50)
);

-- 3. Bảng Lịch hẹn (Appointments) - Đã sửa lỗi dùng Boolean và bổ sung tài chính/trạng thái
CREATE TABLE Appointments (
    appointment_id INT AUTO_INCREMENT PRIMARY KEY,
    patient_id INT NOT NULL,
    doctor_id INT NOT NULL,
    appointment_date DATETIME NOT NULL,
    
    -- Sử dụng ENUM để quản lý 5 trạng thái vòng đời lịch hẹn theo Activity Diagram
    status ENUM('PENDING', 'CONFIRMED', 'CHECKED_IN', 'COMPLETED', 'CANCELLED') DEFAULT 'PENDING',
    
    -- Bổ sung các trường tài chính và lý do hủy lịch
    deposit_amount DECIMAL(10, 2) DEFAULT 0.00,
    penalty_fee DECIMAL(10, 2) DEFAULT 0.00,
    cancel_reason VARCHAR(255) DEFAULT NULL,
    
    FOREIGN KEY (patient_id) REFERENCES Patients(patient_id),
    FOREIGN KEY (doctor_id) REFERENCES Doctors(doctor_id)
);

-- 4. Bảng Đơn thuốc (Prescriptions) - Khắc phục lỗi thiếu hoàn toàn trong bản thiết kế cũ
CREATE TABLE Prescriptions (
    prescription_id INT AUTO_INCREMENT PRIMARY KEY,
    appointment_id INT NOT NULL UNIQUE, -- Quan hệ 1-1 với lịch hẹn đã hoàn tất
    medication_details TEXT NOT NULL,
    issued_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    
    FOREIGN KEY (appointment_id) REFERENCES Appointments(appointment_id)
);

-- =========================================================================
-- MÔ PHỎNG KỊCH BẢN DỮ LIỆU THỰC TẾ (DML)
-- =========================================================================

-- Thêm dữ liệu mẫu ban đầu cho Bệnh nhân và Bác sĩ
INSERT INTO Patients (full_name, phone) VALUES ('Nguyễn Văn An', '0901234567'), ('Trần Thị Bình', '0909876543');
INSERT INTO Doctors (full_name, specialty) VALUES ('Dr. John Smith', 'Tim mạch'), ('Dr. Mary Jane', 'Nhi khoa');

-- KỊCH BẢN 1: Thành công (PENDING -> CHECKED_IN -> COMPLETED + Kê đơn thuốc)
-- Bước 1: Tạo lịch hẹn mới (PENDING) với tiền cọc 500.000đ
INSERT INTO Appointments (patient_id, doctor_id, appointment_date, status, deposit_amount) 
VALUES (1, 1, '2026-04-10 08:30:00', 'PENDING', 500000.00);

-- Bước 2: Bệnh nhân đến check-in
UPDATE Appointments SET status = 'CHECKED_IN' WHERE appointment_id = 1;

-- Bước 3: Khám xong, chuyển trạng thái COMPLETED
UPDATE Appointments SET status = 'COMPLETED' WHERE appointment_id = 1;

-- Bước 4: Bác sĩ kê đơn thuốc cho lịch hẹn hoàn tất này
INSERT INTO Prescriptions (appointment_id, medication_details) 
VALUES (1, 'Paracetamol 500mg: 2 viên/ngày, Vitamin C: 1 viên/ngày');


-- KỊCH BẢN 2: Hủy lịch và Phạt cọc (CONFIRMED -> CANCELLED + Phí phạt)
-- Bước 1: Tạo lịch hẹn (CONFIRMED) với tiền cọc 300.000đ
INSERT INTO Appointments (patient_id, doctor_id, appointment_date, status, deposit_amount) 
VALUES (2, 2, '2026-04-11 09:00:00', 'CONFIRMED', 300000.00);

-- Bước 2: Hủy lịch, ghi nhận lý do và trừ phí phạt 150.000đ
UPDATE Appointments 
SET status = 'CANCELLED', 
    cancel_reason = 'Bận việc đột xuất', 
    penalty_fee = 150000.00 
WHERE appointment_id = 2;


-- =========================================================================
-- TRUY VẤN KIỂM TRA DỮ LIỆU (SELECT)
-- =========================================================================

-- Truy vấn danh sách lịch hẹn thành công kèm chi tiết đơn thuốc
SELECT 
    a.appointment_id,
    p.full_name AS patient_name,
    d.full_name AS doctor_name,
    a.status,
    a.deposit_amount,
    pr.medication_details
FROM Appointments a
JOIN Patients p ON a.patient_id = p.patient_id
JOIN Doctors d ON a.doctor_id = d.doctor_id
LEFT JOIN Prescriptions pr ON a.appointment_id = pr.appointment_id
WHERE a.status = 'COMPLETED';

-- Truy vấn các lịch hẹn bị hủy và thông tin phí phạt
SELECT 
    appointment_id,
    patient_id,
    status,
    deposit_amount,
    penalty_fee,
    cancel_reason
FROM Appointments
WHERE status = 'CANCELLED';
