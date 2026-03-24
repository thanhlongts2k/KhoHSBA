# Database Schema Design - Nexus HSBA (Hồ sơ bệnh án)

Tài liệu: mô tả cấu trúc cơ sở dữ liệu chi tiết cho hệ thống quản lý kho hồ sơ bệnh án số hóa, quy trình mượn trả các cấp và phân hệ giám định BHXH.

---

## 1. Phân hệ Quản trị & Phân quyền (RBAC)
Lưu trữ thông tin cơ cấu tổ chức và kiểm soát truy cập.

### Bảng `Departments` (Danh mục khoa phòng)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | INT (PK) | Định danh tự tăng |
| `DeptCode` | VARCHAR(20) | Mã khoa (VD: KHTH, NOITIEU, DUOC) |
| `DeptName` | NVARCHAR(255) | Tên khoa phòng đầy đủ |
| `IsActive` | BIT | Trạng thái hoạt động (1: Active, 0: Inactive) |

### Bảng `Users` (Người dùng hệ thống)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | UNIQUEIDENTIFIER (PK) | Định danh GUID |
| `Username` | VARCHAR(50) | Tên đăng nhập |
| `FullName` | NVARCHAR(100) | Họ và tên nhân viên |
| `PasswordHash` | VARCHAR(MAX) | Mật khẩu đã mã hóa Argon2/BCrypt |
| `DeptId` | INT (FK) | Liên kết tới bảng Departments |
| `UserType` | VARCHAR(20) | Vai trò (DOCTOR, NURSE, ADMIN, KHTH, BHXH) |
| `IsActive` | BIT | Trạng thái tài khoản |

---

## 2. Phân hệ Kho Hồ sơ bệnh án (Medical Records)
Quản lý định danh hồ sơ và liên kết dữ liệu số hóa (PDF/XML).

### Bảng `MedicalRecords` (Hồ sơ bệnh án)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | UNIQUEIDENTIFIER (PK) | Định danh duy nhất của hồ sơ |
| `PatientCode` | VARCHAR(20) | Mã quản lý bệnh nhân (Mã y tế) |
| `RecordNumber` | VARCHAR(30) | Số lưu trữ HSBA (Ví dụ: BA_2026_0001) |
| `PatientName` | NVARCHAR(200) | Tên bệnh nhân |
| `AdmissionDate` | DATETIME | Ngày giờ nhập viện |
| `DischargeDate` | DATETIME | Ngày giờ xuất viện |
| `XmlPath` | VARCHAR(500) | Đường dẫn vật lý đến file Metadata XML |
| `PdfPath` | VARCHAR(500) | URL/Path file PDF trên FPT Cloud Storage |
| `IsSigned` | BIT | Đã hoàn tất ký số nội bộ chưa |
| `Status` | INT | 0: Lưu kho, 1: Đang mượn, 2: Đang giám định |
| `mavaovien` | VARCHAR(500) | Mã vào viện|
| `loaibenhan` | VARCHAR(500) | loại bệnh án |
| `ketquadieutri` | VARCHAR(500) | kết quả điều trị |
| `tinhtrangravien` | VARCHAR(500) | tình trạng ra viện |
| `khoa` | VARCHAR(500) | Khoa phòng |
---

## 3. Phân hệ Quy trình Mượn trả (Borrowing Workflow)
Quản lý các loại đơn mượn (1, 2, 3) và phê duyệt ký số.

### Bảng `BorrowRequests` (Phiếu mượn hồ sơ)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | UNIQUEIDENTIFIER (PK) | Định danh phiếu mượn |
| `RequestCode` | VARCHAR(20) | Mã số phiếu (Tự động sinh) |
| `RequesterId` | UNIQUEIDENTIFIER (FK) | Người lập đơn (BS/Điều dưỡng) |
| `RequestType` | INT | 1: Xem, 2: Nghiên cứu, 3: Số lượng lớn |
| `Purpose` | NVARCHAR(MAX) | Lý do mượn hồ sơ chi tiết |
| `FromDate` | DATETIME | Thời điểm bắt đầu được phép xem |
| `ToDate` | DATETIME | Thời điểm hết hạn truy cập |
| `CurrentStatus` | INT | 0: Chờ duyệt, 1: Đã duyệt, 2: Từ chối, 3: Đã trả |

### Bảng `BorrowRequestDetails` (Chi tiết hồ sơ trong đơn)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | BIGINT (PK) | Định danh dòng |
| `RequestId` | UNIQUEIDENTIFIER (FK) | Liên kết tới BorrowRequests |
| `MedicalRecordId` | UNIQUEIDENTIFIER (FK) | Liên kết tới MedicalRecords |

### Bảng `RequestApprovals` (Nhật ký phê duyệt & Ký số)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | BIGINT (PK) | Định danh |
| `RequestId` | UNIQUEIDENTIFIER (FK) | Thuộc đơn mượn nào |
| `ApproverId` | UNIQUEIDENTIFIER (FK) | Người ký duyệt (Trưởng khoa/KHTH/GĐ) |
| `StepOrder` | INT | Thứ tự bước duyệt (1, 2, 3) |
| `Action` | INT | 1: Duyệt, 2: Từ chối |
| `Comment` | NVARCHAR(500) | Ý kiến phản hồi |
| `DigitalSignature` | VARBINARY(MAX) | Lưu vết Hash hoặc chuỗi ký số CA |
| `CreatedAt` | DATETIME | Thời gian thực hiện ký/duyệt |

---

## 4. Phân hệ Giám định BHXH (Assessment)
Gom nhóm hồ sơ phục vụ công tác thanh quyết toán bảo hiểm.

### Bảng `AssessmentBatches` (Đợt giám định)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | INT (PK) | Định danh đợt |
| `BatchName` | NVARCHAR(255) | Tên đợt (VD: Giám định nội trú Q1-2026) |
| `IsLocked` | BIT | Trạng thái khóa (1: Đã khóa, không được sửa) |
| `CreatedBy` | UNIQUEIDENTIFIER | Người tạo đợt |
| `CreatedAt` | DATETIME | Ngày khởi tạo |

### Bảng `AssessmentDetails` (Chi tiết giám định hồ sơ)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `BatchId` | INT (FK) | Liên kết tới AssessmentBatches |
| `MedicalRecordId` | UNIQUEIDENTIFIER (FK) | Hồ sơ nằm trong đợt này |
| `AssessorNote` | NVARCHAR(MAX) | Ghi chú của giám định viên BHXH |
| `AssessmentResult` | INT | 0: Đạt, 1: Xuất toán, 2: Bổ sung hồ sơ |

---

## 5. Phân hệ Nhật ký (Security & Audit Logs)
| Trường dữ liệu | Kiểu dữ liệu | Mô tả |
| :--- | :--- | :--- |
| `Id` | BIGINT (PK) | Định danh Log |
| `UserId` | UNIQUEIDENTIFIER | ID người thực hiện thao tác |
| `Action` | VARCHAR(100) | Hành động (VIEW_PDF, DOWNLOAD_XML, LOGIN) |
| `TargetId` | VARCHAR(50) | ID của đối tượng bị tác động (RecordId/RequestId) |
| `ClientIP` | VARCHAR(50) | Địa chỉ IP máy thực hiện |
| `LogTime` | DATETIME | Thời điểm ghi nhận (Default: GETDATE()) |
