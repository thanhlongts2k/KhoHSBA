# Kịch Bản & Payload Test Dự Án KhoHSBA API (.NET 8.0)

Mọi API bảo mật trong hệ thống KhoHSBA đều yêu cầu xác thực bằng Token kiểu Bearer (Ngoại trừ nhóm Auth).
1. Thử nghiệm trên Postman: Chọn Tab `Authorization` -> `Bearer Token`.
2. Thử nghiệm trên VS Code REST Client (`.http`): Chèn Header `Authorization: Bearer {{token}}`.

---

## 1. Authentication (`/Auth`)
Nhóm API chuyên xử lý vấn đề định danh và quản lý phiên.

### 1.1 Đăng nhập lấy Token
- **POST** `/Auth/login`
- **Body JSON**: 
```json
{
  "UserName": "Admin",
  "Password": "123"
}
```

### 1.2 Đăng xuất (Huỷ Token)
- **POST** `/Auth/logout`
- **Cấu hình**: Yêu cầu nhúng Bearer Token mới lấy được vào Header.

### 1.3 Cấp mới Token (Refresh)
- **POST** `/Auth/refresh`
- **Body JSON**:
```json
{
  "Token": "<chuỗi_token_cũ_đã_hết_hạn>",
  "RefreshToken": "<chuỗi_refresh_token_cũ_trong_db>"
}
```

---

## 2. API Khuôn Mẫu Chung (Dành cho 8 Feature Core)
Các phân hệ dưới đây đều có cấu trúc API hoàn toàn giống hệt nhau bao gồm 5 Endpoint:
- `/MedicalRecords`
- `/BorrowRequests`
- `/BorrowRequestDetails`
- `/Departments`
- `/Users`
- `/AssessmentBatches`
- `/RequestApprovals`
- `/AuditLogs`

### 2.1 Get All (GET)
- **GET** `/{FeatureName}/getAll`
- Không cần Payload Body. Lấy ra toàn bộ danh sách trong CSDL.

### 2.2 Get Theo Khóa Chính (POST)
- **POST** `/{FeatureName}/get`
- **Body JSON** (Model CommonRequest):
```json
{
  "ID": "1"
}
```

### 2.3 Thêm mới / Insert (POST)
- **POST** `/{FeatureName}/insert`
- **Body JSON Mẫu (Dùng `/Departments` làm ví dụ)**:
```json
{
  "Id": "PB001",
  "DeptCode": "HCNN",
  "DeptName": "Hành Chính Nhân Sự",
  "IsActive": 1
}
```

### 2.4 Cập nhật / Update (POST)
- **POST** `/{FeatureName}/update`
- **Body JSON Mẫu**:
```json
{
  "Id": "PB001",
  "DeptCode": "HCNN",
  "DeptName": "Hành Chính Tổng Hợp (Vừa Sửa Lại)",
  "IsActive": 1
}
```

### 2.5 Xóa theo ID (POST)
- **POST** `/{FeatureName}/delete`
- **Body JSON Mẫu (`CommonRequest`)**:
```json
{
  "ID": "PB001"
}
```

---

## 3. Cấu Trúc Ngoại Lệ: AssessmentDetails (Khoá chính kép)
Vì `AssessmentDetails` có cấu hình 2 mã định danh chạy song song là `BatchId` và `MedicalRecordId`, API đã được tinh chỉnh lại hoàn toàn thành POST Body để thuận tiện truyền Model.

### 3.1 Get & Delete By Multi-ID (POST)
- **POST** `/AssessmentDetails/get`
  *(Hoặc POST `/AssessmentDetails/delete`)*
- **Body JSON**:
```json
{
  "BatchId": "Batch001",
  "MedicalRecordId": "HS001"
}
```

### 3.2 Update & Insert (POST)
- **POST** `/AssessmentDetails/update` 
  *(Hoặc `/insert`)*
- **Body JSON**:
```json
{
  "BatchId": "Batch001",
  "MedicalRecordId": "HS001",
  "AssessorNote": "Hồ sơ lưu trữ đầy đủ, chữ ký rõ ràng",
  "AssessmentResult": 1
}
```

---

## 4. Kiểm tra Chuẩn Định Dạng Trả Về (Response Checking)
Mọi API (kể cả thành công hay báo lỗi Validate) đều phải xuất ra định dạng chung duy nhất:
```json
// Ví dụ Thành công
{
  "status": true,
  "message": "",
  "result": { ...dữ_liệu_bên_trong... }
}

// Ví dụ Báo lỗi nhập sai Field (Model State Invalid do BaseController quăng ra)
{
  "status": false,
  "message": "Dữ liệu không hợp lệ",
  "result": null
}
```
