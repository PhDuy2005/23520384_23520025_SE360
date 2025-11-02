# 🚀 UIT-GO Platform – System Architecture Documentation

## 🧭 Tổng quan

**UIT-GO** là nền tảng chia sẻ chuyến đi (Ride Sharing Platform) được thiết kế theo mô hình **Microservice Architecture**, cho phép các thành phần chính hoạt động độc lập, có khả năng mở rộng và triển khai linh hoạt trên môi trường Cloud (AWS).

---

## 🧩 Thành phần chính

| Service | Port | Chức năng chính | Database riêng | Ngôn ngữ / Framework |
|----------|-------|----------------|----------------|----------------------|
| **User Service** | `8080` | Quản lý người dùng, đăng ký, đăng nhập, xác thực JWT | ✅ `userdb` | Java 17 / Spring Boot / Gradle Kotlin |
| **Driver Service** | `8081` | Quản lý tài xế, trạng thái hoạt động, vị trí | ✅ `driverdb` | Java 17 / Spring Boot / Gradle Kotlin |
| **Trip Service** | `8082` | Quản lý chuyến đi, kết nối user-driver | ✅ `tripdb` | Java 17 / Spring Boot / Gradle Kotlin |

---

## 🧱 Kiến trúc tổng quan hệ thống

```
                       ┌──────────────────────────────┐
                       │         User Service         │
                       │  - Đăng nhập / Đăng ký       │
                       │  - JWT Authentication         │
                       │  - Quản lý người dùng         │
                       └──────────────┬───────────────┘
                                      │
                                      ▼
┌──────────────────────┐      ┌──────────────────────┐      ┌──────────────────────┐
│    Driver Service     │◀────▶│     Trip Service     │◀────▶│    User Service       │
│  - Quản lý tài xế     │      │ - Quản lý chuyến đi  │      │ - Xác thực & JWT     │
│  - Cập nhật vị trí    │      │ - Ghép nối tài xế    │      │                      │
│  - REST API           │      │ - REST API           │      │ - REST API           │
└────────────┬──────────┘      └────────────┬─────────┘      └────────────┬─────────┘
             │                             │                           │
             ▼                             ▼                           ▼
      🗄️ driverdb (MySQL)          🗄️ tripdb (MySQL)            🗄️ userdb (MySQL)
```

---

## 🔐 Authentication & Authorization Flow

1. Người dùng đăng nhập qua **User Service → /login**
2. User Service phát hành **JWT token**
3. Client (mobile/web) gửi token này trong header:
   ```
   Authorization: Bearer <jwt>
   ```
4. Các service khác (**Driver**, **Trip**) xác thực token bằng `spring-boot-starter-oauth2-resource-server` sử dụng **shared secret key**.

**Key chia sẻ giữa các service:**
```properties
uitgo.jwt.base64-secret=noVGO4KXfRQijWLkkHTdwMZzJcsvohOLNTzXHkWOEOwwj50/QWunAGce8b6XKqUwss6ozCb5A/e++2SPZN/d2Q==
```

---

## 🧠 Mô hình cơ sở dữ liệu tổng quan

### 1️⃣ UserDB
| Trường | Kiểu | Ghi chú |
|--------|------|----------|
| id | BIGINT | PK |
| username | VARCHAR(50) | duy nhất |
| email | VARCHAR(100) | duy nhất |
| password_hash | VARCHAR(255) | đã mã hóa |
| role | ENUM('USER','DRIVER','ADMIN') | Phân quyền |
| created_at | TIMESTAMP |  |

---

### 2️⃣ DriverDB
| Trường | Kiểu | Ghi chú |
|--------|------|----------|
| id | BIGINT | PK |
| user_id | BIGINT | FK logic từ User Service |
| full_name | VARCHAR(100) |  |
| license_number | VARCHAR(50) |  |
| vehicle_type | VARCHAR(50) |  |
| vehicle_plate | VARCHAR(20) |  |
| status | ENUM('ONLINE','OFFLINE','ON_TRIP') |  |
| rating | DOUBLE | trung bình |
| latitude | DOUBLE | vị trí hiện tại |
| longitude | DOUBLE | vị trí hiện tại |

---

### 3️⃣ TripDB
| Trường | Kiểu | Ghi chú |
|--------|------|----------|
| id | BIGINT | PK |
| passenger_id | BIGINT | userId từ User Service |
| driver_id | BIGINT | driverId từ Driver Service |
| pickup_location | VARCHAR(255) | điểm đón |
| dropoff_location | VARCHAR(255) | điểm đến |
| fare | DOUBLE | giá cước |
| status | ENUM('REQUESTED','ACCEPTED','IN_PROGRESS','COMPLETED','CANCELLED') |  |
| requested_at | TIMESTAMP |  |
| started_at | TIMESTAMP |  |
| completed_at | TIMESTAMP |  |

---

## 📦 Communication (Inter-service)

| Từ | Đến | Loại giao tiếp | Mục đích |
|----|-----|----------------|-----------|
| Trip Service | User Service | REST | Xác thực hành khách (JWT) |
| Trip Service | Driver Service | REST | Lấy thông tin tài xế, trạng thái |
| Driver Service | User Service | REST | Lấy thông tin user của tài xế |

**Mẫu giao tiếp:**
```java
WebClient.create("http://user-service:8080")
    .get()
    .uri("/api/v1/users/{id}", id)
    .header(HttpHeaders.AUTHORIZATION, "Bearer " + jwt)
    .retrieve()
    .bodyToMono(UserDTO.class);
```

---

## 🐳 Triển khai bằng Docker Compose

```yaml
version: '3.8'
services:
  user-service:
    build: ./user-service
    container_name: user-service
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-user:3306/userdb
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 123456
    depends_on:
      - mysql-user
    networks:
      - uitgo-network

  driver-service:
    build: ./driver-service
    container_name: driver-service
    ports:
      - "8081:8081"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-driver:3306/driverdb
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 123456
    depends_on:
      - mysql-driver
    networks:
      - uitgo-network

  trip-service:
    build: ./trip-service
    container_name: trip-service
    ports:
      - "8082:8082"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql-trip:3306/tripdb
      SPRING_DATASOURCE_USERNAME: root
      SPRING_DATASOURCE_PASSWORD: 123456
    depends_on:
      - mysql-trip
    networks:
      - uitgo-network

  mysql-user:
    image: mysql:8.0
    container_name: mysql-user
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: userdb
    ports:
      - "3306:3306"

  mysql-driver:
    image: mysql:8.0
    container_name: mysql-driver
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: driverdb
    ports:
      - "3307:3306"

  mysql-trip:
    image: mysql:8.0
    container_name: mysql-trip
    environment:
      MYSQL_ROOT_PASSWORD: 123456
      MYSQL_DATABASE: tripdb
    ports:
      - "3308:3306"

networks:
  uitgo-network:
    driver: bridge
```

---

## ☁️ Triển khai trên AWS

- **ECR:** lưu trữ image Docker của từng microservice  
- **ECS Fargate:** chạy container độc lập  
- **RDS:** lưu trữ MySQL cho từng service  
- **CloudWatch Logs:** giám sát logs và metrics  
- **Secrets Manager:** quản lý thông tin nhạy cảm (DB, JWT secret)

---

## 🧱 Quy tắc thiết kế

1. **Database per Service:** mỗi service sở hữu dữ liệu riêng.  
2. **API Gateway (tùy chọn):** có thể thêm để điều phối request.  
3. **Stateless:** tất cả service không lưu session trong bộ nhớ cục bộ.  
4. **Security:** mọi API đều yêu cầu JWT xác thực.  
5. **IaC:** Dockerfile + Compose + Terraform (AWS provisioning).  
6. **Monitoring:** dùng Spring Actuator + CloudWatch.

---

## 🔄 Phiên bản

| Version | Ngày | Mô tả |
|----------|------|-------|
| 1.0 | 2025-11-02 | Hoàn thiện kiến trúc tổng thể UIT-GO Microservices |
| 1.1 | (future) | Tích hợp API Gateway và Payment Service |

---

## ✅ Kết luận

Kiến trúc này đảm bảo:
- Dễ mở rộng và bảo trì từng service độc lập.  
- Tối ưu cho CI/CD và Cloud deployment.  
- Dễ dàng mở rộng thêm các service như **Payment**, **Notification**, hoặc **Analytics** trong tương lai.
