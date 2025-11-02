# SE360 Cloud - Đồ án môn học

## 👥 Thông tin thành viên

| MSSV     | Họ và tên           |
| -------- | ------------------- |
| 23520384 | Phạm Trần Khánh Duy |
| 23520025 | Phạm Hữu An         |

---

## 🏗️ Kiến trúc hệ thống

Tham khảo file [ARCHITECTURE.md](./ARCHITECTURE.md) để xem chi tiết về kiến trúc hệ thống, các quyết định thiết kế và sơ đồ tổng quan.

**Cách xem file ARCHITECTURE.md trong VS Code:**
- Mở file `ARCHITECTURE.md`
- Nhấn `Ctrl+Shift+V` để xem Preview
- Hoặc nhấn `Ctrl+K` rồi `V` để mở Preview side-by-side

---

## 🚀 Microservices

Dự án **UIT-GO** (Ride Sharing Platform) bao gồm 3 microservices chính:

1. **User Service** (Port: 8080) - _Quản lý người dùng và xác thực_
   - Đăng ký, đăng nhập và quản lý người dùng
   - JWT Authentication và phân quyền
   - Database: `userdb` (MySQL)
   - Repository: [SE360-User-Service](https://github.com/PhDuy2005/SE360-User-Service)
   
2. **Driver Service** (Port: 8081) - _Quản lý tài xế và phương tiện_
   - Quản lý thông tin tài xế, giấy phép lái xe
   - Cập nhật vị trí và trạng thái (ONLINE/OFFLINE/ON_TRIP)
   - Quản lý thông tin phương tiện và đánh giá
   - Database: `driverdb` (MySQL)
   - Repository: [Microservice-for-Driver](https://github.com/PhDuy2005/Microservice-for-Driver)
   
3. **Trip Service** (Port: 8082) - _Quản lý chuyến đi_
   - Tạo và quản lý chuyến đi
   - Ghép nối hành khách với tài xế
   - Theo dõi trạng thái chuyến đi (REQUESTED/ACCEPTED/IN_PROGRESS/COMPLETED/CANCELLED)
   - Database: `tripdb` (MySQL)
   - Repository: [Microservice-for-Trip](https://github.com/PhDuy2005/Microservice-for-Trip)

---

## 📝 Báo cáo thực hành

### Phạm Trần Khánh Duy (23520384)
- [Báo cáo thực hành](./SE360_LabReport_PhamTranKhanhDuy_23520384.pdf)

### Phạm Hữu An (23520025)
- [Báo cáo thực hành - Link sẽ cập nhật]

---

## 📚 Tài liệu tham khảo

- [ARCHITECTURE.md](./ARCHITECTURE.md) - Tài liệu kiến trúc hệ thống

---

## 🛠️ Công nghệ sử dụng

### Backend & Framework
- **Spring Boot** - Java framework cho phát triển microservices

### Cloud Infrastructure
- **AWS EC2** - Máy chủ ảo để deploy các microservices
- **AWS RDS** - Dịch vụ cơ sở dữ liệu quan hệ trên cloud

### Database
- **MySQL** - Hệ quản trị cơ sở dữ liệu quan hệ

### DevOps & Containerization
- **Docker** - Containerization platform để đóng gói và triển khai ứng dụng

---

## 📞 Liên hệ

- **Phạm Trần Khánh Duy**: 23520384@gm.uit.edu.vn
- **Phạm Hữu An**: 23520025@gm.uit.edu.vn

---

_Cập nhật lần cuối: November 2, 2025_
