# TÓM TẮT CHUẨN CHUNG — BACKEND ENGINEERING

> Tóm lược từ 4 tài liệu: `CAU_TRUC_THU_MUC` · `MA_LOI` · `API_CONTRACT` · `QUY_TAC_DAT_TEN`  
> Mỗi mục chỉ giữ lại điều quan trọng nhất + ví dụ trực tiếp.

---

## MỤC LỤC

1. [Cấu Trúc Thư Mục](#1-cấu-trúc-thư-mục)
2. [Quy Tắc Đặt Tên](#2-quy-tắc-đặt-tên)
3. [Mã Lỗi](#3-mã-lỗi)
4. [API — Đầu Vào & Đầu Ra](#4-api--đầu-vào--đầu-ra)

---

## 1. Cấu Trúc Thư Mục

> **Triết lý:** Feature-based (theo nghiệp vụ), KHÔNG phải Type-based (theo kỹ thuật).

### ❌ Sai — Type-based
```
src/
├── controllers/
│   ├── UserController.ts
│   └── OrderController.ts
├── services/
│   ├── UserService.ts
│   └── OrderService.ts
└── repositories/
    ├── UserRepository.ts
    └── OrderRepository.ts
```

### ✅ Đúng — Feature-based (Vertical Slice)
```
src/
├── features/
│   ├── users/
│   │   ├── api/              # Controller / Handler / Router
│   │   ├── application/      # Use Cases, Commands, Queries
│   │   ├── domain/           # Entity, Value Object, Domain Event
│   │   ├── infrastructure/   # Repository impl, DB queries
│   │   └── tests/            # Test của feature này
│   ├── orders/
│   │   ├── api/
│   │   ├── application/
│   │   ├── domain/
│   │   ├── infrastructure/
│   │   └── tests/
│   └── payments/
│       └── ...
├── shared/                   # Code dùng chung giữa các features
│   ├── exceptions/
│   ├── middleware/
│   ├── utils/
│   └── types/
├── infrastructure/           # Kết nối hạ tầng (DB, Redis, S3, Queue)
│   ├── database/
│   ├── cache/
│   ├── messaging/
│   └── storage/
└── config/                   # Đọc từ env variables
```

### Cấu trúc root project
```
project-root/
├── src/                      # Source code
├── tests/                    # Unit / Integration / E2E
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── docs/                     # Tài liệu kỹ thuật, ADR
├── deployments/              # Dockerfile, K8s manifests
│   ├── docker/
│   └── kubernetes/
├── scripts/                  # Shell scripts tiện ích
├── .env.example              # ✅ COMMIT — mẫu biến môi trường
├── .env                      # ❌ KHÔNG commit — giá trị thật
├── .gitignore
├── .editorconfig
├── docker-compose.yml
├── Makefile
├── README.md
└── CHANGELOG.md
```

### Cấu trúc theo từng ngôn ngữ

| Ngôn ngữ | Tên file | Tên thư mục |
|---|---|---|
| **C# / .NET** | `OrderService.cs` | `order-processing/` |
| **Java** | `OrderService.java` | `order-processing/` |
| **Python** | `order_service.py` | `order-processing/` |
| **Go** | `order_service.go` | `order-processing/` |
| **TypeScript** | `order-service.ts` | `order-processing/` |
| **PHP** | `OrderService.php` | `order-processing/` |

> Chi tiết từng ngôn ngữ xem [`CAU_TRUC_THU_MUC.md`](CAU_TRUC_THU_MUC.md)

---

## 2. Quy Tắc Đặt Tên

### Bảng tổng hợp casing

| Loại | Quy tắc | ✅ Đúng | ❌ Sai |
|---|---|---|---|
| Class / Struct | PascalCase | `OrderService` | `orderService`, `order_service` |
| Interface (C#) | `I` + PascalCase | `IOrderRepository` | `OrderRepository` |
| Interface (Java/TS/Go) | PascalCase | `OrderRepository` | `IOrderRepository` |
| Method (C# / Go public) | PascalCase | `GetOrderById` | `getOrderById` |
| Method (Java / JS / TS / PHP) | camelCase | `getOrderById` | `GetOrderById` |
| Method (Python / Go private) | snake_case | `get_order_by_id` | `getOrderById` |
| Variable (Java / C# / TS) | camelCase | `currentUser` | `CurrentUser`, `current_user` |
| Variable (Python) | snake_case | `current_user` | `currentUser` |
| Field private (C#) | `_` + camelCase | `_orderRepository` | `orderRepository` |
| Hằng số (Java / Python / Go) | UPPER\_SNAKE\_CASE | `MAX_RETRY_COUNT` | `maxRetryCount` |
| Enum tên | PascalCase | `OrderStatus` | `order_status` |
| Enum value (C# / TS) | PascalCase | `OrderStatus.Pending` | `OrderStatus.PENDING` |
| Enum value (Java / Python) | UPPER\_SNAKE\_CASE | `OrderStatus.PENDING` | `OrderStatus.Pending` |
| DB Table | snake\_case PLURAL | `order_items` | `OrderItems`, `tbl_order` |
| DB Column | snake\_case | `created_at`, `user_id` | `createdAt`, `UserId` |
| DB Primary Key | `id` | `id` | `order_id` (trên bảng orders) |
| DB Foreign Key | `[ref_table_singular]_id` | `user_id`, `order_id` | `userId`, `fk_user` |
| DB Index | `idx_[table]_[cols]` | `idx_orders_user_id` | `orders_user_id_idx` |
| API URL | kebab-case PLURAL | `/order-items` | `/OrderItems`, `/getOrders` |
| Path Param | camelCase | `{userId}`, `{orderId}` | `{user_id}`, `{UserID}` |
| Query Param | camelCase | `?pageSize=20&sortBy=name` | `?page_size=20` |
| HTTP Header custom | Train-Case `X-` | `X-Request-Id` | `x-request-id` |
| Env Variable | UPPER\_SNAKE\_CASE | `DB_HOST`, `JWT_SECRET` | `dbHost`, `Db-Host` |
| Folder | kebab-case | `user-management/` | `UserManagement/` |
| Branch Git | `[type]/[ticket]-[desc]` | `feature/PROJ-123-login` | `feature_login` |
| Docker Image | `org/name:version` | `company/user-service:1.2.3` | `company/UserService:latest` |
| Cache Key | lowercase colon | `cache:v1:user:123` | `Cache_V1_User_123` |
| Domain Event | PascalCase + `Event` | `OrderPlacedEvent` | `OrderPlaced`, `order_placed` |
| Log field | camelCase | `traceId`, `durationMs` | `trace_id`, `duration_ms` |
| Metric | `[ns]_[comp]_[measure]_[unit]` | `order_svc_http_requests_total` | `httpRequests` |

### Ví dụ đặt tên component theo layer

```
Feature: Orders

API Layer:
  Controller/Handler → OrderController, OrderHandler
  Request DTO       → CreateOrderRequest, UpdateOrderRequest
  Response DTO      → OrderResponse, OrderSummaryDto

Application Layer:
  Command           → CreateOrderCommand, CancelOrderCommand
  Query             → GetOrderByIdQuery, ListOrdersQuery
  Handler           → CreateOrderCommandHandler, GetOrderByIdQueryHandler

Domain Layer:
  Entity            → Order, OrderLine
  Value Object      → Money, DeliveryAddress
  Repository (IF)   → IOrderRepository  (C#) / OrderRepository (Java/TS)
  Domain Event      → OrderPlacedEvent, OrderCancelledEvent
  Domain Service    → OrderPricingDomainService

Infrastructure:
  Repository impl   → OrderRepository (C#) / OrderRepositoryImpl (Java)
  Migration file    → 20260623_143000_create_orders_table.sql

Database:
  Table             → orders, order_lines
  Column            → id, user_id, total_amount, status, created_at, updated_at, deleted_at
  Index             → idx_orders_user_id, idx_orders_status_created_at
```

### Quy tắc đặt tên biến — nhanh

```
# ✅ Boolean → bắt đầu bằng is / has / can / should
isActive, hasSubscription, canEdit, shouldRetry

# ✅ Collection → số nhiều
users, orderItems, productIds

# ✅ Đếm → suffix Count
userCount, totalItems, retryCount

# ✅ Hằng số
MAX_RETRY_COUNT = 3
DEFAULT_PAGE_SIZE = 20
JWT_EXPIRY_SECONDS = 900

# ❌ Tránh
data, info, result, temp    # quá chung chung
x, y, z                     # không có nghĩa (trừ toán học)
usrSvc, ord, cfg            # viết tắt khó đọc
userList, nameString        # có type trong tên
```

> Chi tiết đầy đủ xem [`QUY_TAC_DAT_TEN.md`](QUY_TAC_DAT_TEN.md)

---

## 3. Mã Lỗi

### Format chuẩn
```
[DOMAIN]_[NNN]     →     AUTH_001, USER_004, PAY_002
```

### Danh sách domain

| Domain | Phạm vi | Mô tả |
|---|---|---|
| `AUTH` | AUTH_001–099 | Xác thực, token, session |
| `ACCESS` | ACCESS_001–099 | Phân quyền, kiểm soát truy cập |
| `USER` | USER_001–099 | Tài khoản, hồ sơ người dùng |
| `VAL` | VAL_001–099 | Validation đầu vào |
| `RES` | RES_001–099 | Tài nguyên chung (not found, conflict…) |
| `ORDER` | ORDER_001–099 | Đơn hàng |
| `PAY` | PAY_001–099 | Thanh toán, giao dịch |
| `FILE` | FILE_001–099 | Upload, quản lý file |
| `RATE` | RATE_001–009 | Rate limiting |
| `SYS` | SYS_001–099 | Lỗi hệ thống nội bộ |
| `INT` | INT_001–099 | Tích hợp bên ngoài |

### Mã lỗi thường dùng nhất

| Mã Lỗi | HTTP | Mô Tả | Retry? |
|---|---|---|---|
| `AUTH_001` | 401 | Thông tin đăng nhập sai | Không |
| `AUTH_002` | 401 | Token hết hạn | Không (làm mới token) |
| `AUTH_003` | 401 | Token không hợp lệ | Không |
| `AUTH_004` | 401 | Thiếu token | Không |
| `AUTH_005` | 403 | Tài khoản bị khóa | Không |
| `AUTH_007` | 403 | Yêu cầu xác thực 2FA | Không |
| `ACCESS_001` | 403 | Không đủ quyền | Không |
| `ACCESS_002` | 403 | Không phải chủ sở hữu | Không |
| `USER_001` | 404 | Không tìm thấy user | Không |
| `USER_002` | 409 | Email đã tồn tại | Không |
| `VAL_001` | 422 | Thiếu trường bắt buộc | Không |
| `VAL_002` | 422 | Giá trị quá dài | Không |
| `VAL_003` | 422 | Sai định dạng | Không |
| `VAL_004` | 422 | Giá trị ngoài khoảng cho phép | Không |
| `RES_001` | 404 | Tài nguyên không tồn tại | Không |
| `RES_002` | 409 | Tài nguyên đã tồn tại | Không |
| `RES_004` | 423 | Tài nguyên đang bị khóa | Có (sau delay) |
| `PAY_001` | 402 | Không đủ số dư | Không |
| `PAY_002` | 402 | Thẻ bị từ chối | Không |
| `PAY_005` | 408 | Thanh toán hết thời gian | Có |
| `FILE_001` | 413 | File vượt kích thước cho phép | Không |
| `FILE_002` | 415 | Định dạng file không hỗ trợ | Không |
| `RATE_001` | 429 | Quá nhiều request | Có (sau Retry-After) |
| `SYS_001` | 500 | Lỗi server nội bộ | Có (backoff) |
| `SYS_002` | 503 | Dịch vụ tạm ngưng | Có (backoff) |
| `SYS_004` | 504 | Timeout | Có (backoff) |

### Response lỗi chuẩn

```json
// Lỗi đơn
{
  "success": false,
  "error": {
    "code": "AUTH_002",
    "message": "Token đã hết hạn. Vui lòng đăng nhập lại.",
    "details": null,
    "timestamp": "2026-06-23T10:30:00Z",
    "traceId": "abc123def456"
  }
}

// Lỗi validation (nhiều trường)
{
  "success": false,
  "error": {
    "code": "VAL_001",
    "message": "Dữ liệu đầu vào không hợp lệ.",
    "details": [
      { "field": "email",    "code": "VAL_003", "message": "Email không đúng định dạng" },
      { "field": "phone",    "code": "VAL_001", "message": "Số điện thoại là bắt buộc" },
      { "field": "age",      "code": "VAL_004", "message": "Tuổi phải từ 18 đến 100" }
    ],
    "timestamp": "2026-06-23T10:30:00Z",
    "traceId": "abc123def456"
  }
}
```

> Danh sách đầy đủ 100+ mã lỗi xem [`MA_LOI.md`](MA_LOI.md)

---

## 4. API — Đầu Vào & Đầu Ra

### Headers request bắt buộc

| Header | Bắt buộc | Ví dụ |
|---|---|---|
| `Content-Type` | Khi có body | `application/json; charset=utf-8` |
| `Authorization` | API cần auth | `Bearer eyJhbGci...` |
| `X-Request-ID` | Nên có | `550e8400-e29b-41d4-a716-446655440000` |
| `Idempotency-Key` | POST tạo tài nguyên | `idempotency-550e8400-abc123` |
| `X-Tenant-ID` | Multi-tenant | `org_abc123` |

### HTTP Method → Hành động

| Method | Hành động | Có body? | Idempotent? | Status thành công |
|---|---|---|---|---|
| `GET` | Lấy dữ liệu | Không | ✅ | 200 |
| `POST` | Tạo mới | Có | ❌ | 201 |
| `PUT` | Thay thế toàn bộ | Có | ✅ | 200 |
| `PATCH` | Cập nhật một phần | Có | ❌ | 200 |
| `DELETE` | Xóa | Không | ✅ | 204 |

### HTTP Status Code quan trọng

| Status | Ý nghĩa | Khi nào dùng |
|---|---|---|
| `200 OK` | Thành công | GET, PUT, PATCH thành công |
| `201 Created` | Tạo mới thành công | POST tạo tài nguyên |
| `202 Accepted` | Đã nhận, xử lý bất đồng bộ | Job dài, gửi email, export |
| `204 No Content` | Thành công, không có nội dung | DELETE thành công |
| `400 Bad Request` | Request sai cú pháp | JSON parse error, missing field |
| `401 Unauthorized` | Chưa xác thực | Thiếu/sai token |
| `403 Forbidden` | Đã xác thực, không đủ quyền | Đúng token, sai quyền |
| `404 Not Found` | Không tìm thấy | Resource không tồn tại |
| `409 Conflict` | Xung đột dữ liệu | Email đã tồn tại, duplicate |
| `422 Unprocessable` | Validation thất bại | Dữ liệu sai logic nghiệp vụ |
| `429 Too Many Requests` | Rate limit | Quá nhiều request |
| `500 Internal Error` | Lỗi server | Bug, exception không xử lý |
| `503 Unavailable` | Dịch vụ tạm ngưng | Maintenance, overload |

### Format Response chuẩn

```json
// ✅ Thành công — Object đơn
{
  "data": {
    "id": "01J2X5K8M3N7P9Q1R4S6T0",
    "orderNumber": "ORD-2026-001234",
    "status": "PENDING",
    "totalAmount": 250000,
    "createdAt": "2026-06-23T10:30:00Z"
  },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2026-06-23T10:30:00Z",
    "version": "1.0"
  }
}

// ✅ Thành công — Danh sách có phân trang
{
  "data": {
    "items": [
      { "id": "01J2X...", "orderNumber": "ORD-001", "status": "PENDING" },
      { "id": "01J2Y...", "orderNumber": "ORD-002", "status": "SHIPPED" }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "total": 543,
      "totalPages": 28,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  },
  "meta": {
    "requestId": "550e8400-e29b-41d4-a716-446655440000",
    "timestamp": "2026-06-23T10:30:00Z",
    "version": "1.0"
  }
}

// ✅ Tác vụ bất đồng bộ (202 Accepted)
{
  "data": {
    "jobId": "job_01J2X5K8M3N7P9Q1R4S6T0",
    "status": "PENDING",
    "estimatedDurationSeconds": 30,
    "statusUrl": "/api/v1/jobs/job_01J2X5K8M3N7P9Q1R4S6T0"
  },
  "meta": { "requestId": "...", "timestamp": "..." }
}
```

### Phân trang — Query params chuẩn

```
# Offset-based (dùng cho admin, dataset nhỏ < 100k)
GET /api/v1/orders?page=2&pageSize=20

# Cursor-based (dùng cho feed, dataset lớn, real-time)
GET /api/v1/orders?cursor=eyJpZCI6IjAxSj...&pageSize=20

# Sắp xếp
GET /api/v1/orders?sortBy=createdAt&sortOrder=desc

# Lọc
GET /api/v1/orders?status=PENDING&createdAtFrom=2026-01-01&createdAtTo=2026-06-30

# Tìm kiếm
GET /api/v1/orders?q=john+doe
```

### Ví dụ thực tế — Order API đầy đủ

```
# Tạo đơn hàng
POST /api/v1/orders
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json
  Idempotency-Key: {uuid}

Body:
{
  "customerId": "cust_01J2X...",
  "items": [
    { "productId": "prod_01J...", "quantity": 2, "unitPrice": 125000 }
  ],
  "deliveryAddress": {
    "street": "123 Lê Lợi",
    "city": "Hồ Chí Minh",
    "postalCode": "70000"
  },
  "note": "Giao buổi sáng"
}

Response 201:
{
  "data": {
    "id": "ord_01J2X5K8M3N7P9Q1R4S6T0",
    "orderNumber": "ORD-2026-001234",
    "status": "PENDING",
    "totalAmount": 250000,
    "currency": "VND",
    "createdAt": "2026-06-23T10:30:00Z"
  },
  "meta": { "requestId": "...", "timestamp": "..." }
}

---

# Lấy đơn hàng theo ID
GET /api/v1/orders/ord_01J2X5K8M3N7P9Q1R4S6T0
Response 200: { "data": { ...full order... }, "meta": {...} }

# Danh sách đơn hàng
GET /api/v1/orders?status=PENDING&page=1&pageSize=20&sortBy=createdAt&sortOrder=desc
Response 200: { "data": { "items": [...], "pagination": {...} }, "meta": {...} }

# Hủy đơn hàng
PATCH /api/v1/orders/ord_01J2X5K8M3N7P9Q1R4S6T0/cancel
Body: { "reason": "Khách hàng đổi ý" }
Response 200: { "data": { "id": "...", "status": "CANCELLED" }, "meta": {...} }

# Xóa đơn nháp
DELETE /api/v1/orders/ord_01J2X5K8M3N7P9Q1R4S6T0
Response 204: (no body)
```

### Định dạng dữ liệu chuẩn trong JSON

| Loại | Format | Ví dụ |
|---|---|---|
| Date (chỉ ngày) | `YYYY-MM-DD` | `"2026-06-23"` |
| DateTime | ISO 8601 UTC | `"2026-06-23T10:30:00Z"` |
| Tiền tệ | Integer (đơn vị nhỏ nhất) | `250000` (VND) hoặc `1999` (cent USD) |
| UUID | lowercase + hyphen | `"550e8400-e29b-41d4-a716-446655440000"` |
| Boolean | `true` / `false` | `true` |
| Enum | UPPER\_SNAKE\_CASE string | `"PENDING"`, `"CREDIT_CARD"` |
| Null vs vắng mặt | Không trả về field null | Bỏ field thay vì `"field": null` |
| Số điện thoại | Chuỗi, E.164 | `"+84901234567"` |

> Chi tiết đầy đủ xem [`API_CONTRACT.md`](API_CONTRACT.md)

---

## Nguyên Tắc Không Được Vi Phạm

```
DATABASE
  ✅ Tên bảng: snake_case, số nhiều (orders, order_items)
  ✅ Audit fields bắt buộc: created_at, updated_at, deleted_at
  ✅ Timestamp: luôn UTC
  ✅ Tiền: integer (không dùng float/double)
  ❌ Không SELECT * trong code production
  ❌ Không nối chuỗi SQL (dùng parameterized query)

API
  ✅ URL: danh từ số nhiều, kebab-case, không có động từ
  ✅ JSON fields: camelCase
  ✅ Response: luôn có envelope { data, meta }
  ✅ Error: luôn dùng mã từ MA_LOI.md, không tự đặt
  ❌ Không trả 200 cho mọi thứ kể cả lỗi
  ❌ Không để stack trace trong response production

CODE
  ✅ Business logic chỉ nằm trong domain / application layer
  ✅ Inject dependency qua interface, không new trực tiếp
  ✅ Hàm làm đúng 1 việc, không vừa query vừa modify state
  ❌ Không hardcode config value
  ❌ Không log password, token, thông tin nhạy cảm

GIT
  ✅ Commit: feat/fix/docs/refactor/test/chore: mô tả ngắn
  ✅ Branch: feature/PROJ-123-ten-chuc-nang
  ❌ Không push thẳng lên main
  ❌ Không commit .env có giá trị thật
```
