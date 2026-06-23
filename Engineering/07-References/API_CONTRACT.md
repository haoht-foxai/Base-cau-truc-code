# HOP DONG API - DAU VAO VA DAU RA

> **Phiên bản:** 1.0.0
> **Cập nhật lần cuối:** 2026-06-23
> **Ngôn ngữ:** Vietnamese (Tiếng Việt)
> **Trạng thái:** Tài liệu chính thức - Tham chiếu bắt buộc cho toàn bộ nhóm phát triển

---

Tài liệu này là **tài liệu chuẩn tắc dứt khoát** cho mọi định dạng request/response của API trong hệ thống. Mọi API mới hoặc cũ đều phải tuân theo các quy tắc được định nghĩa ở đây. Không có ngoại lệ trừ khi có phê duyệt kỹ thuật bằng văn bản.

---

## MUC LUC

1. [Quy Tắc Chung](#1-quy-tac-chung)
2. [Format Request Chuẩn](#2-format-request-chuan)
3. [Format Response Chuẩn](#3-format-response-chuan)
4. [HTTP Method → Action Mapping](#4-http-method--action-mapping)
5. [HTTP Status Code → Ý Nghĩa](#5-http-status-code--y-nghia)
6. [Các Pattern API Phổ Biến](#6-cac-pattern-api-pho-bien)
7. [Định Dạng Dữ Liệu Chuẩn](#7-dinh-dang-du-lieu-chuan)
8. [Versioning API](#8-versioning-api)
9. [Ví Dụ Thực Tế Đầy Đủ](#9-vi-du-thuc-te-day-du)

---

## 1. QUY TAC CHUNG

### Nguyên tắc thiết kế API

Toàn bộ API trong hệ thống tuân theo các nguyên tắc sau:

- **RESTful**: Sử dụng HTTP method đúng ngữ nghĩa, URL biểu diễn tài nguyên
- **JSON-only**: Mọi request body và response body đều là JSON (trừ file upload/download)
- **UTF-8**: Mọi chuỗi ký tự đều được mã hóa UTF-8
- **Stateless**: Server không lưu trạng thái phiên; mọi request phải tự đủ thông tin
- **Idempotent nơi có thể**: GET, PUT, DELETE phải idempotent
- **Versioned**: URL luôn chứa phiên bản `/api/v1/`, `/api/v2/`

---

### 1.1 Headers Bắt Buộc (Request)

Đây là các header **bắt buộc** phải có trong mọi request gửi lên server.

| Header Name | Loại | Bắt buộc? | Mô tả | Ví dụ |
|---|---|---|---|---|
| `Content-Type` | string | Có (khi có body) | Kiểu nội dung của request body | `application/json; charset=utf-8` |
| `Accept` | string | Nên có | Kiểu nội dung client muốn nhận | `application/json` |
| `Authorization` | string | Có (API xác thực) | Token xác thực theo chuẩn Bearer JWT | `Bearer eyJhbGciOiJIUzI1NiJ9...` |
| `X-Request-ID` | string (UUID) | Nên có | ID duy nhất cho mỗi request, dùng để trace log | `550e8400-e29b-41d4-a716-446655440000` |
| `X-Client-Version` | string | Nên có | Phiên bản ứng dụng phía client | `mobile-ios/2.1.3` |
| `X-Tenant-ID` | string | Có (multi-tenant) | Mã định danh tổ chức/tenant trong hệ thống multi-tenant | `org_abc123` |
| `Accept-Language` | string | Không | Ngôn ngữ mong muốn cho message lỗi | `vi-VN` |
| `Idempotency-Key` | string (UUID) | Có (POST tạo tài nguyên) | Đảm bảo không tạo trùng khi retry | `idempotency-550e8400-abc123` |

**Lưu ý về `Idempotency-Key`:**
- Bắt buộc với các POST request tạo tài nguyên quan trọng (đơn hàng, thanh toán)
- Client phải sinh UUID mới cho mỗi lần thực sự muốn tạo mới
- Khi retry cùng request thất bại, giữ nguyên `Idempotency-Key`
- Server cache kết quả trong 24 giờ theo key này

---

### 1.2 Headers Chuẩn (Response)

Đây là các header server **phải trả về** trong mọi response.

| Header Name | Loại | Luôn có? | Mô tả | Ví dụ |
|---|---|---|---|---|
| `Content-Type` | string | Có | Kiểu nội dung của response | `application/json; charset=utf-8` |
| `X-Request-ID` | string (UUID) | Có | Echo lại request ID từ client (hoặc server tự sinh) | `550e8400-e29b-41d4-a716-446655440000` |
| `X-Trace-ID` | string | Có | ID để trace xuyên suốt hệ thống microservices | `trace-7f3d2b1a9e4c` |
| `X-Response-Time` | string (ms) | Có | Thời gian server xử lý tính bằng millisecond | `127ms` |
| `X-Rate-Limit-Limit` | integer | Có | Số request tối đa trong cửa sổ thời gian hiện tại | `1000` |
| `X-Rate-Limit-Remaining` | integer | Có | Số request còn lại trong cửa sổ thời gian | `743` |
| `X-Rate-Limit-Reset` | integer (Unix timestamp) | Có | Thời điểm reset rate limit (Unix epoch seconds) | `1750672800` |
| `X-API-Version` | string | Có | Phiên bản API đang phục vụ request này | `1.2.3` |
| `Cache-Control` | string | Tùy API | Chỉ thị cache cho response | `no-store` hoặc `max-age=300` |
| `ETag` | string | GET có thể cache | Entity tag cho conditional request | `"abc123def456"` |
| `Last-Modified` | string (RFC7231) | GET có thể cache | Thời điểm tài nguyên thay đổi lần cuối | `Mon, 23 Jun 2026 10:00:00 GMT` |
| `Location` | string (URL) | 201, 301, 302 | URL của tài nguyên vừa tạo hoặc địa chỉ redirect | `/api/v1/orders/uuid-123` |
| `Retry-After` | integer (giây) | 429, 503 | Thời gian client phải chờ trước khi retry | `60` |
| `Deprecation` | string (date) | API cũ | Thông báo API sắp bị xóa, kèm ngày | `2027-01-01` |
| `Sunset` | string (date) | API cũ | Ngày chính xác API ngừng hoạt động | `Mon, 01 Jan 2027 00:00:00 GMT` |

---

### 1.3 Quy Tắc Đặt Tên

- **URL path**: `kebab-case`, số ít cho tài nguyên đơn, số nhiều cho collection
  - Đúng: `/api/v1/orders`, `/api/v1/user-profiles`
  - Sai: `/api/v1/Order`, `/api/v1/getOrders`, `/api/v1/userprofile`
- **JSON fields**: `camelCase` cho mọi field trong request và response
  - Đúng: `createdAt`, `totalAmount`, `userId`
  - Sai: `created_at`, `TotalAmount`, `user_id`
- **Query params**: `camelCase`
  - Đúng: `?pageSize=20&sortBy=createdAt`
  - Sai: `?page_size=20&sort_by=createdAt`

---

## 2. FORMAT REQUEST CHUAN

### 2.1 Request Không Có Body (GET, DELETE)

```http
GET /api/v1/orders/550e8400-e29b-41d4-a716-446655440000 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
X-Request-ID: 7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d
X-Client-Version: web/3.2.1
```

### 2.2 Request Tạo Tài Nguyên (POST)

```http
POST /api/v1/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-ID: 7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d
Idempotency-Key: idem-550e8400-e29b-41d4-a716-446655440001
X-Client-Version: web/3.2.1

{
  "customerId": "cust-abc123",
  "shippingAddress": {
    "street": "123 Nguyen Hue",
    "ward": "Ben Nghe",
    "district": "Quan 1",
    "city": "Ho Chi Minh",
    "zipCode": "700000"
  },
  "items": [
    {
      "productId": "prod-xyz789",
      "quantity": 2,
      "unitPrice": 150000
    }
  ],
  "couponCode": "SUMMER2026",
  "note": "Giao hàng buổi sáng, gọi trước 30 phút"
}
```

**Giải thích các field trong request body POST:**

| Field | Loại | Bắt buộc? | Mô tả | Ràng buộc |
|---|---|---|---|---|
| `customerId` | string | Có | ID khách hàng đặt đơn | Phải tồn tại trong hệ thống |
| `shippingAddress` | object | Có | Địa chỉ giao hàng | Xem schema địa chỉ |
| `shippingAddress.street` | string | Có | Số nhà và tên đường | Tối đa 200 ký tự |
| `shippingAddress.ward` | string | Có | Phường/Xã | Tối đa 100 ký tự |
| `shippingAddress.district` | string | Có | Quận/Huyện | Tối đa 100 ký tự |
| `shippingAddress.city` | string | Có | Tỉnh/Thành phố | Tối đa 100 ký tự |
| `shippingAddress.zipCode` | string | Không | Mã bưu chính | 6 chữ số |
| `items` | array | Có | Danh sách sản phẩm đặt mua | Ít nhất 1 phần tử |
| `items[].productId` | string | Có | ID sản phẩm | Phải tồn tại và còn hàng |
| `items[].quantity` | integer | Có | Số lượng đặt mua | Lớn hơn 0, nhỏ hơn tồn kho |
| `items[].unitPrice` | integer | Có | Giá tại thời điểm đặt (VNĐ) | Lớn hơn 0 |
| `couponCode` | string | Không | Mã giảm giá | Tối đa 50 ký tự |
| `note` | string | Không | Ghi chú của khách hàng | Tối đa 500 ký tự |

### 2.3 Request Cập Nhật Toàn Phần (PUT)

PUT thay thế toàn bộ tài nguyên. Mọi field không gửi lên sẽ bị xóa/reset về giá trị mặc định.

```http
PUT /api/v1/products/prod-xyz789 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-ID: 8a4b2c1d-3e5f-6a7b-8c9d-0e1f2a3b4c5d

{
  "name": "Áo thun nam cổ tròn - Phiên bản 2026",
  "description": "Áo thun chất liệu cotton 100%, thoáng mát, phù hợp mặc hàng ngày",
  "price": 189000,
  "category": "ao-thun",
  "stock": 150,
  "images": ["https://cdn.example.com/products/p1.jpg"],
  "isActive": true
}
```

### 2.4 Request Cập Nhật Một Phần (PATCH)

PATCH chỉ cập nhật các field được gửi lên. Field không gửi giữ nguyên giá trị cũ.

```http
PATCH /api/v1/products/prod-xyz789 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-ID: 9b5c3d2e-4f6a-7b8c-9d0e-1f2a3b4c5d6e

{
  "price": 199000,
  "stock": 200
}
```

---

## 3. FORMAT RESPONSE CHUAN

### 3.1 Response Thành Công - Object Đơn (200 OK)

Dùng khi trả về một tài nguyên duy nhất.

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "CONFIRMED",
    "customerId": "cust-abc123",
    "customerName": "Nguyễn Văn An",
    "totalAmount": 300000,
    "shippingFee": 30000,
    "discountAmount": 0,
    "finalAmount": 330000,
    "shippingAddress": {
      "street": "123 Nguyen Hue",
      "ward": "Ben Nghe",
      "district": "Quan 1",
      "city": "Ho Chi Minh",
      "zipCode": "700000"
    },
    "items": [
      {
        "productId": "prod-xyz789",
        "productName": "Áo thun nam cổ tròn",
        "quantity": 2,
        "unitPrice": 150000,
        "subtotal": 300000
      }
    ],
    "createdAt": "2026-06-23T10:00:00Z",
    "updatedAt": "2026-06-23T10:05:00Z"
  },
  "meta": {
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:05:00Z",
    "version": "1.0"
  }
}
```

**Giải thích cấu trúc response object đơn:**

| Field | Loại | Mô tả |
|---|---|---|
| `data` | object | Chứa toàn bộ dữ liệu tài nguyên được trả về |
| `data.id` | string (UUID) | ID duy nhất của tài nguyên |
| `data.*` | any | Các field thuộc tính của tài nguyên, tùy theo loại |
| `meta` | object | Thông tin siêu dữ liệu về response |
| `meta.requestId` | string (UUID) | ID của request, dùng để tra cứu log |
| `meta.timestamp` | string (ISO8601) | Thời điểm server tạo response (UTC) |
| `meta.version` | string | Phiên bản schema của response |

---

### 3.2 Response Thành Công - Danh Sách Có Phân Trang (200 OK)

Dùng khi trả về danh sách tài nguyên với phân trang dựa trên số trang (page-based).

```json
{
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "status": "CONFIRMED",
        "customerName": "Nguyễn Văn An",
        "finalAmount": 330000,
        "createdAt": "2026-06-23T10:00:00Z"
      },
      {
        "id": "660f9511-f30c-52e5-b827-557766551111",
        "status": "DELIVERED",
        "customerName": "Trần Thị Bình",
        "finalAmount": 450000,
        "createdAt": "2026-06-22T15:30:00Z"
      }
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
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:05:00Z",
    "version": "1.0"
  }
}
```

**Giải thích cấu trúc phân trang:**

| Field | Loại | Mô tả |
|---|---|---|
| `data.items` | array | Danh sách các phần tử trong trang hiện tại |
| `data.pagination` | object | Thông tin phân trang |
| `data.pagination.page` | integer | Số trang hiện tại (bắt đầu từ 1) |
| `data.pagination.pageSize` | integer | Số phần tử tối đa mỗi trang |
| `data.pagination.total` | integer | Tổng số phần tử thỏa điều kiện lọc |
| `data.pagination.totalPages` | integer | Tổng số trang = ceil(total / pageSize) |
| `data.pagination.hasNextPage` | boolean | `true` nếu còn trang tiếp theo |
| `data.pagination.hasPrevPage` | boolean | `true` nếu có trang trước đó |

---

### 3.3 Response Lỗi - Lỗi Đơn

Dùng khi xảy ra một lỗi cụ thể (không phải lỗi validation nhiều field).

```json
{
  "error": {
    "code": "USER_001",
    "message": "Không tìm thấy người dùng",
    "traceId": "trace-7f3d2b1a9e4c4b8a"
  }
}
```

**Ví dụ thêm các loại lỗi đơn:**

Lỗi chưa xác thực (401):
```json
{
  "error": {
    "code": "AUTH_001",
    "message": "Token xác thực không hợp lệ hoặc đã hết hạn. Vui lòng đăng nhập lại.",
    "traceId": "trace-8a4b2c1d3e5f6a7b"
  }
}
```

Lỗi không có quyền truy cập (403):
```json
{
  "error": {
    "code": "AUTHZ_002",
    "message": "Bạn không có quyền thực hiện thao tác này. Liên hệ quản trị viên nếu cần thêm quyền.",
    "traceId": "trace-9b5c3d2e4f6a7b8c"
  }
}
```

Lỗi xung đột nghiệp vụ (409):
```json
{
  "error": {
    "code": "ORDER_003",
    "message": "Đơn hàng không thể hủy vì đã được giao cho đơn vị vận chuyển",
    "traceId": "trace-0c6d4e3f5a7b8c9d",
    "context": {
      "orderId": "550e8400-e29b-41d4-a716-446655440000",
      "currentStatus": "SHIPPING",
      "allowedCancelStatuses": ["PENDING", "CONFIRMED"]
    }
  }
}
```

**Giải thích cấu trúc lỗi:**

| Field | Loại | Bắt buộc? | Mô tả |
|---|---|---|---|
| `error` | object | Có | Container chứa thông tin lỗi |
| `error.code` | string | Có | Mã lỗi nghiệp vụ (xem bảng mã lỗi) |
| `error.message` | string | Có | Thông báo lỗi có thể hiển thị cho người dùng |
| `error.traceId` | string | Có | ID để tra cứu log kỹ thuật chi tiết |
| `error.context` | object | Không | Thông tin bổ sung về ngữ cảnh lỗi |

---

### 3.4 Response Lỗi - Lỗi Validation (Nhiều Lỗi)

Dùng khi dữ liệu đầu vào vi phạm nhiều quy tắc validation cùng lúc. HTTP status: `422 Unprocessable Entity`.

```json
{
  "error": {
    "code": "VAL_001",
    "message": "Dữ liệu đầu vào không hợp lệ. Vui lòng kiểm tra lại các trường được đánh dấu.",
    "details": [
      {
        "field": "email",
        "code": "VAL_003",
        "message": "Email không đúng định dạng. Ví dụ hợp lệ: ten@domain.com"
      },
      {
        "field": "phone",
        "code": "VAL_002",
        "message": "Số điện thoại là bắt buộc"
      },
      {
        "field": "items[0].quantity",
        "code": "VAL_005",
        "message": "Số lượng phải lớn hơn 0 và không vượt quá tồn kho hiện tại (15)"
      },
      {
        "field": "shippingAddress.city",
        "code": "VAL_006",
        "message": "Thành phố không hợp lệ. Chỉ hỗ trợ giao hàng trong 63 tỉnh thành Việt Nam"
      }
    ],
    "traceId": "trace-1d7e5f4a3b2c1d0e"
  }
}
```

**Giải thích cấu trúc lỗi validation:**

| Field | Loại | Mô tả |
|---|---|---|
| `error.code` | string | Luôn là `VAL_001` cho lỗi validation tổng quát |
| `error.message` | string | Thông báo tổng quát cho toàn bộ lỗi |
| `error.details` | array | Danh sách từng lỗi chi tiết theo field |
| `error.details[].field` | string | Đường dẫn đến field lỗi (dùng dot notation, có hỗ trợ array index) |
| `error.details[].code` | string | Mã lỗi cụ thể cho từng field |
| `error.details[].message` | string | Thông báo lỗi cho field đó, hiển thị được cho người dùng |

**Ví dụ đường dẫn field (field path):**

| Path | Ý nghĩa |
|---|---|
| `email` | Field `email` ở root |
| `shippingAddress.city` | Field `city` trong object `shippingAddress` |
| `items[0].quantity` | Field `quantity` của phần tử đầu tiên trong array `items` |
| `items[2].variants[1].price` | Nested array path |

---

### 3.5 Response Tác Vụ Không Đồng Bộ (202 Accepted)

Dùng khi server nhận request nhưng xử lý bất đồng bộ (background job). Client cần polling để theo dõi tiến trình.

```json
{
  "data": {
    "jobId": "job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "status": "PENDING",
    "estimatedDuration": 30,
    "statusUrl": "/api/v1/jobs/job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "createdAt": "2026-06-23T10:00:00Z"
  },
  "meta": {
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:00:00Z",
    "version": "1.0"
  }
}
```

**Giải thích:**

| Field | Loại | Mô tả |
|---|---|---|
| `data.jobId` | string | ID của background job để tra cứu |
| `data.status` | enum | Trạng thái hiện tại: `PENDING`, `PROCESSING`, `COMPLETED`, `FAILED` |
| `data.estimatedDuration` | integer | Ước tính thời gian hoàn thành (giây) |
| `data.statusUrl` | string | URL để poll trạng thái job |
| `data.createdAt` | string (ISO8601) | Thời điểm job được tạo |

---

### 3.6 Response Phân Trang Cursor-Based

Dùng cho danh sách lớn cần hiệu suất cao, hoặc dữ liệu có thể thay đổi liên tục (real-time feed, activity log). Không hỗ trợ nhảy trang, chỉ đi tiến/lùi.

```json
{
  "data": {
    "items": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "action": "ORDER_CREATED",
        "description": "Đơn hàng #DH001234 đã được tạo",
        "createdAt": "2026-06-23T10:00:00Z"
      },
      {
        "id": "660f9511-f30c-52e5-b827-557766551111",
        "action": "PAYMENT_CONFIRMED",
        "description": "Thanh toán cho đơn hàng #DH001234 đã xác nhận",
        "createdAt": "2026-06-23T09:55:00Z"
      }
    ],
    "cursor": {
      "current": "eyJpZCI6IjY2MGY5NTExIiwiY3JlYXRlZEF0IjoiMjAyNi0wNi0yM1QwOTo1NTowMFoifQ==",
      "next": "eyJpZCI6Ijc3MGcwNjIyIiwiY3JlYXRlZEF0IjoiMjAyNi0wNi0yM1QwOTo1MDowMFoifQ==",
      "prev": null,
      "hasMore": true
    }
  },
  "meta": {
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:05:00Z",
    "version": "1.0"
  }
}
```

**Cách sử dụng cursor pagination:**

- Lần đầu: `GET /api/v1/activity-logs?limit=20`
- Trang tiếp: `GET /api/v1/activity-logs?limit=20&cursor=eyJpZCI6Ijc3MGcwN...`
- Trang trước: `GET /api/v1/activity-logs?limit=20&before=eyJpZCI6IjU1MGU4N...`

| Field | Loại | Mô tả |
|---|---|---|
| `data.cursor.current` | string (base64) | Cursor vị trí hiện tại (để bookmark) |
| `data.cursor.next` | string (base64) | Cursor để lấy trang tiếp theo, `null` nếu hết |
| `data.cursor.prev` | string (base64) | Cursor để lấy trang trước đó, `null` nếu ở đầu |
| `data.cursor.hasMore` | boolean | `true` nếu còn dữ liệu phía sau |

---

### 3.7 Response Tạo Thành Công (201 Created)

```json
{
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "status": "PENDING",
    "orderCode": "DH001234",
    "finalAmount": 330000,
    "createdAt": "2026-06-23T10:00:00Z"
  },
  "meta": {
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:00:00Z",
    "version": "1.0"
  }
}
```

Kèm theo header: `Location: /api/v1/orders/550e8400-e29b-41d4-a716-446655440000`

### 3.8 Response Xóa Thành Công (204 No Content)

Khi xóa thành công, server trả về `204 No Content` với **body rỗng**.

```http
HTTP/1.1 204 No Content
X-Request-ID: 7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d
X-Trace-ID: trace-7f3d2b1a9e4c4b8a
X-Response-Time: 45ms
```

---

## 4. HTTP METHOD -> ACTION MAPPING

| HTTP Method | Hành động | Request Body | Response thành công | Idempotent? | Mô tả |
|---|---|---|---|---|---|
| `GET` | Đọc dữ liệu | Không | 200 + data | Có | Lấy một hoặc nhiều tài nguyên. Không được thay đổi state. |
| `POST` | Tạo mới | Có | 201 + data | Không | Tạo tài nguyên mới. Dùng `Idempotency-Key` nếu cần retry an toàn. |
| `PUT` | Thay thế toàn bộ | Có | 200 + data | Có | Thay thế toàn bộ tài nguyên. Field nào không gửi sẽ bị xóa. |
| `PATCH` | Cập nhật một phần | Có | 200 + data | Không | Chỉ cập nhật các field được gửi lên. Field không gửi giữ nguyên. |
| `DELETE` | Xóa | Không | 204 no body | Có | Xóa tài nguyên. Xóa cùng tài nguyên nhiều lần vẫn trả 204. |
| `HEAD` | Kiểm tra tồn tại | Không | 200 no body | Có | Kiểm tra resource có tồn tại không, lấy metadata từ headers |
| `OPTIONS` | Xem options | Không | 200 + Allow header | Có | Xem các HTTP method được phép trên endpoint này |

**Ví dụ mapping URL cho tài nguyên Đơn Hàng:**

| Method | URL | Hành động |
|---|---|---|
| `GET` | `/api/v1/orders` | Lấy danh sách đơn hàng |
| `POST` | `/api/v1/orders` | Tạo đơn hàng mới |
| `GET` | `/api/v1/orders/{id}` | Xem chi tiết đơn hàng |
| `PUT` | `/api/v1/orders/{id}` | Cập nhật toàn bộ đơn hàng |
| `PATCH` | `/api/v1/orders/{id}` | Cập nhật một số trường đơn hàng |
| `DELETE` | `/api/v1/orders/{id}` | Xóa đơn hàng |
| `POST` | `/api/v1/orders/{id}/cancel` | Hủy đơn hàng (action) |
| `POST` | `/api/v1/orders/{id}/confirm` | Xác nhận đơn hàng (action) |
| `GET` | `/api/v1/orders/{id}/items` | Lấy danh sách sản phẩm trong đơn |

---

## 5. HTTP STATUS CODE -> Y NGHIA

| Status | Tên | Ý nghĩa | Khi nào dùng |
|---|---|---|---|
| `200` | OK | Thành công | GET, PUT, PATCH thành công |
| `201` | Created | Tạo thành công | POST tạo tài nguyên mới thành công |
| `202` | Accepted | Đã nhận, đang xử lý | Tác vụ được đưa vào hàng đợi xử lý bất đồng bộ |
| `204` | No Content | Thành công, không có body | DELETE thành công; PATCH không có gì cần trả về |
| `400` | Bad Request | Request sai cú pháp | JSON không parse được; query param sai kiểu |
| `401` | Unauthorized | Chưa xác thực | Thiếu token; token hết hạn; token không hợp lệ |
| `403` | Forbidden | Không có quyền | Token hợp lệ nhưng không có quyền với tài nguyên này |
| `404` | Not Found | Không tìm thấy | Tài nguyên không tồn tại hoặc đã bị xóa |
| `405` | Method Not Allowed | HTTP method không được phép | Gọi DELETE trên endpoint chỉ hỗ trợ GET |
| `409` | Conflict | Xung đột | Email đã tồn tại; đơn đã ở trạng thái không thể hủy |
| `410` | Gone | Đã xóa vĩnh viễn | Tài nguyên từng tồn tại nhưng đã bị xóa cứng |
| `422` | Unprocessable Entity | Lỗi validation | Dữ liệu sai ràng buộc nghiệp vụ dù cú pháp đúng |
| `429` | Too Many Requests | Vượt rate limit | Client gọi quá nhiều, phải chờ theo `Retry-After` |
| `500` | Internal Server Error | Lỗi server | Lỗi không mong đợi phía server |
| `502` | Bad Gateway | Gateway lỗi | Server trung gian nhận response lỗi từ upstream |
| `503` | Service Unavailable | Service không khả dụng | Server bảo trì hoặc quá tải |
| `504` | Gateway Timeout | Gateway timeout | Upstream không phản hồi trong thời gian cho phép |

**Chi tiết từng mã lỗi server:**

**400 Bad Request** - Ví dụ:
```json
{
  "error": {
    "code": "REQ_001",
    "message": "Request body không phải JSON hợp lệ",
    "traceId": "trace-abc123"
  }
}
```

**401 Unauthorized** - Ví dụ:
```json
{
  "error": {
    "code": "AUTH_001",
    "message": "Token xác thực không hợp lệ hoặc đã hết hạn",
    "traceId": "trace-abc123"
  }
}
```

**429 Too Many Requests** - Ví dụ:
```json
{
  "error": {
    "code": "RATE_001",
    "message": "Bạn đã vượt quá giới hạn 1000 request/giờ. Vui lòng thử lại sau 60 giây.",
    "retryAfter": 60,
    "traceId": "trace-abc123"
  }
}
```
Kèm header: `Retry-After: 60`

**500 Internal Server Error** - Ví dụ (KHÔNG lộ stack trace ra ngoài):
```json
{
  "error": {
    "code": "SYS_001",
    "message": "Hệ thống gặp lỗi không mong đợi. Nhóm kỹ thuật đã được thông báo. Vui lòng thử lại sau.",
    "traceId": "trace-abc123"
  }
}
```

---

## 6. CAC PATTERN API PHO BIEN

### 6.1 CRUD Cơ Bản

#### Tạo mới (Create)

```
POST /api/v1/products
```

Request:
```json
{
  "name": "Áo khoác bomber nam",
  "description": "Áo khoác thời trang, chất liệu polyester cao cấp",
  "price": 450000,
  "category": "ao-khoac",
  "stock": 100
}
```

Response 201:
```json
{
  "data": {
    "id": "prod-new-uuid-here",
    "name": "Áo khoác bomber nam",
    "description": "Áo khoác thời trang, chất liệu polyester cao cấp",
    "price": 450000,
    "category": "ao-khoac",
    "stock": 100,
    "isActive": true,
    "createdAt": "2026-06-23T10:00:00Z",
    "updatedAt": "2026-06-23T10:00:00Z"
  },
  "meta": {
    "requestId": "req-uuid",
    "timestamp": "2026-06-23T10:00:00Z",
    "version": "1.0"
  }
}
```

#### Đọc một tài nguyên (Read Single)

```
GET /api/v1/products/prod-xyz789
```

Response 200: Trả về object như trên.

Nếu không tìm thấy, trả về 404:
```json
{
  "error": {
    "code": "PROD_001",
    "message": "Không tìm thấy sản phẩm với ID: prod-xyz789",
    "traceId": "trace-abc123"
  }
}
```

#### Đọc danh sách (List)

```
GET /api/v1/products
```

Response 200: Trả về danh sách có phân trang như mục 3.2.

#### Cập nhật toàn phần (Full Update - PUT)

```
PUT /api/v1/products/prod-xyz789
```

Request: Gửi toàn bộ object (kể cả field không thay đổi)
Response 200: Trả về object sau khi cập nhật

#### Cập nhật một phần (Partial Update - PATCH)

```
PATCH /api/v1/products/prod-xyz789
```

Request:
```json
{
  "price": 399000,
  "stock": 85
}
```

Response 200:
```json
{
  "data": {
    "id": "prod-xyz789",
    "name": "Áo khoác bomber nam",
    "price": 399000,
    "stock": 85,
    "updatedAt": "2026-06-23T11:00:00Z"
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

#### Xóa (Delete)

```
DELETE /api/v1/products/prod-xyz789
```

Response 204: Body rỗng

---

### 6.2 Phân Trang (Pagination)

Tham số query cho phân trang:

| Query Param | Mặc định | Mô tả |
|---|---|---|
| `page` | `1` | Số trang (bắt đầu từ 1) |
| `pageSize` | `20` | Số phần tử mỗi trang (tối đa 100) |

Ví dụ danh sách đơn hàng trang 3:

```
GET /api/v1/orders?page=3&pageSize=20
```

Response:
```json
{
  "data": {
    "items": [
      {
        "id": "ord-uuid-41",
        "orderCode": "DH000041",
        "customerName": "Lê Văn Cường",
        "status": "DELIVERED",
        "finalAmount": 520000,
        "createdAt": "2026-06-20T08:30:00Z"
      },
      {
        "id": "ord-uuid-42",
        "orderCode": "DH000042",
        "customerName": "Phạm Thị Dung",
        "status": "CONFIRMED",
        "finalAmount": 280000,
        "createdAt": "2026-06-20T09:15:00Z"
      }
    ],
    "pagination": {
      "page": 3,
      "pageSize": 20,
      "total": 543,
      "totalPages": 28,
      "hasNextPage": true,
      "hasPrevPage": true
    }
  },
  "meta": {
    "requestId": "req-uuid",
    "timestamp": "2026-06-23T10:05:00Z",
    "version": "1.0"
  }
}
```

---

### 6.3 Sắp Xếp (Sorting)

Tham số query cho sắp xếp:

| Query Param | Mô tả | Ví dụ |
|---|---|---|
| `sortBy` | Tên field cần sắp xếp | `sortBy=createdAt` |
| `sortOrder` | Chiều sắp xếp: `asc` hoặc `desc` | `sortOrder=desc` |

**Sắp xếp đơn:**
```
GET /api/v1/orders?sortBy=createdAt&sortOrder=desc
```

**Sắp xếp nhiều cấp (multi-sort):**

Sắp xếp theo `status` tăng dần, sau đó theo `createdAt` giảm dần:
```
GET /api/v1/orders?sortBy=status&sortBy=createdAt&sortOrder=asc&sortOrder=desc
```

Thứ tự của `sortBy` và `sortOrder` phải tương ứng nhau theo vị trí.

**Các field cho phép sắp xếp:**
Server phải định nghĩa danh sách field được phép sắp xếp và trả về lỗi `400` nếu client dùng field không được phép:

```json
{
  "error": {
    "code": "SORT_001",
    "message": "Không thể sắp xếp theo field 'password'. Các field được phép: createdAt, updatedAt, status, finalAmount",
    "traceId": "trace-abc123"
  }
}
```

---

### 6.4 Lọc Dữ Liệu (Filtering)

**Lọc cơ bản theo giá trị cụ thể:**
```
GET /api/v1/orders?status=CONFIRMED
GET /api/v1/orders?status=CONFIRMED&status=PENDING
```

**Lọc theo khoảng (range filter):**
```
GET /api/v1/orders?createdAtFrom=2026-01-01&createdAtTo=2026-06-30
GET /api/v1/orders?finalAmountMin=100000&finalAmountMax=500000
```

**Lọc theo danh sách giá trị (list filter):**
```
GET /api/v1/orders?status=CONFIRMED,PENDING,SHIPPING
GET /api/v1/products?category=ao-thun,ao-khoac,quan-jean
```

**Lọc text (partial match):**
```
GET /api/v1/products?nameLike=áo thun
GET /api/v1/customers?emailLike=@gmail.com
```

**Ví dụ kết hợp lọc và phân trang và sắp xếp:**
```
GET /api/v1/orders?status=CONFIRMED,PENDING&createdAtFrom=2026-06-01&createdAtTo=2026-06-30&page=1&pageSize=50&sortBy=finalAmount&sortOrder=desc
```

**Quy ước tên filter:**

| Hậu tố | Ý nghĩa | Ví dụ |
|---|---|---|
| (không hậu tố) | Khớp chính xác | `?status=ACTIVE` |
| `Like` | Chứa chuỗi (case-insensitive) | `?nameLike=điện thoại` |
| `From` | Lớn hơn hoặc bằng | `?createdAtFrom=2026-01-01` |
| `To` | Nhỏ hơn hoặc bằng | `?createdAtTo=2026-06-30` |
| `Min` | Lớn hơn hoặc bằng (số) | `?priceMin=100000` |
| `Max` | Nhỏ hơn hoặc bằng (số) | `?priceMax=500000` |
| `IsNull` | Field là null | `?deletedAtIsNull=true` |

---

### 6.5 Tìm Kiếm (Search)

Tìm kiếm toàn văn (full-text search) dùng parameter `q`:

```
GET /api/v1/products?q=áo thun cotton nam
```

Server thực hiện tìm kiếm across nhiều fields (name, description, tags, ...) theo logic nội bộ.

Response không thay đổi cấu trúc. Có thể thêm `score` trong từng item để cho biết độ phù hợp (nếu cần):

```json
{
  "data": {
    "items": [
      {
        "id": "prod-abc",
        "name": "Áo thun nam cotton 100%",
        "score": 0.98
      },
      {
        "id": "prod-def",
        "name": "Áo thun cotton unisex",
        "score": 0.85
      }
    ],
    "pagination": { "page": 1, "pageSize": 20, "total": 12, "totalPages": 1, "hasNextPage": false, "hasPrevPage": false }
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

---

### 6.6 Tải File Lên (Upload)

Dùng `multipart/form-data`. **Không dùng** `application/json` cho upload.

Request:
```http
POST /api/v1/products/prod-xyz789/images HTTP/1.1
Host: api.example.com
Authorization: Bearer ...
Content-Type: multipart/form-data; boundary=----FormBoundary7MA4YWxkTrZu0gW
X-Request-ID: req-uuid

------FormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="product-front.jpg"
Content-Type: image/jpeg

[binary file data]
------FormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="type"

PRODUCT_IMAGE
------FormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="altText"

Áo thun nam màu trắng, nhìn từ phía trước
------FormBoundary7MA4YWxkTrZu0gW--
```

Response 201:
```json
{
  "data": {
    "id": "img-uuid-here",
    "url": "https://cdn.example.com/products/prod-xyz789/img-uuid-here.jpg",
    "thumbnailUrl": "https://cdn.example.com/products/prod-xyz789/img-uuid-here-thumb.jpg",
    "type": "PRODUCT_IMAGE",
    "altText": "Áo thun nam màu trắng, nhìn từ phía trước",
    "size": 245760,
    "mimeType": "image/jpeg",
    "width": 800,
    "height": 800,
    "createdAt": "2026-06-23T10:00:00Z"
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Giới hạn upload:**
- Kích thước tối đa: 10MB mỗi file
- Định dạng hỗ trợ: JPEG, PNG, WebP, PDF
- Số file mỗi request: tối đa 5 file

---

### 6.7 Tải File Xuống (Download)

Response trả về binary data kèm header phù hợp:

```http
GET /api/v1/reports/rpt-uuid-123/download HTTP/1.1
Authorization: Bearer ...
```

Response:
```http
HTTP/1.1 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="bao-cao-doanh-thu-thang-6-2026.pdf"
Content-Length: 1048576
Cache-Control: private, no-store
X-Request-ID: req-uuid
X-Trace-ID: trace-abc123

[binary PDF data]
```

Nếu file chưa sẵn sàng (đang generate):
```json
{
  "error": {
    "code": "FILE_002",
    "message": "File đang được tạo, vui lòng thử lại sau 30 giây",
    "retryAfter": 30,
    "traceId": "trace-abc123"
  }
}
```
HTTP status: 202 Accepted, kèm header `Retry-After: 30`

---

### 6.8 Thao Tác Hàng Loạt (Batch Operations)

Dùng để thực hiện nhiều thao tác trong một request nhằm giảm round-trip.

**Batch cùng loại thao tác:**

```
POST /api/v1/products/batch
```

Request:
```json
{
  "operation": "UPDATE_STATUS",
  "ids": [
    "prod-uuid-001",
    "prod-uuid-002",
    "prod-uuid-003"
  ],
  "data": {
    "isActive": false
  }
}
```

Response 200:
```json
{
  "data": {
    "succeeded": [
      "prod-uuid-001",
      "prod-uuid-002"
    ],
    "failed": [
      {
        "id": "prod-uuid-003",
        "code": "PROD_002",
        "message": "Sản phẩm đang có đơn hàng đang xử lý, không thể tắt kích hoạt"
      }
    ],
    "total": 3,
    "successCount": 2,
    "failCount": 1
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Batch nhiều loại thao tác khác nhau:**

```
POST /api/v1/batch
```

Request:
```json
{
  "requests": [
    {
      "requestId": "local-req-1",
      "method": "GET",
      "url": "/api/v1/orders/ord-uuid-001"
    },
    {
      "requestId": "local-req-2",
      "method": "PATCH",
      "url": "/api/v1/orders/ord-uuid-002",
      "body": {
        "note": "Khách yêu cầu giao trước 10h sáng"
      }
    },
    {
      "requestId": "local-req-3",
      "method": "DELETE",
      "url": "/api/v1/cart-items/item-uuid-005"
    }
  ]
}
```

Response 200:
```json
{
  "data": {
    "responses": [
      {
        "requestId": "local-req-1",
        "status": 200,
        "body": { "data": { "id": "ord-uuid-001", "status": "CONFIRMED" }, "meta": {} }
      },
      {
        "requestId": "local-req-2",
        "status": 200,
        "body": { "data": { "id": "ord-uuid-002", "note": "Khách yêu cầu giao trước 10h sáng" }, "meta": {} }
      },
      {
        "requestId": "local-req-3",
        "status": 204,
        "body": null
      }
    ]
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Lưu ý batch:**
- Tối đa 20 thao tác trong một batch request
- Mỗi thao tác vẫn tính vào rate limit riêng
- Nếu một thao tác trong batch thất bại, các thao tác khác vẫn tiếp tục (không có transaction)

---

### 6.9 Webhook Callback

Khi sự kiện xảy ra trong hệ thống, server gửi HTTP POST đến URL đã đăng ký của client (webhook endpoint).

**Payload webhook:**
```json
{
  "eventId": "evt-550e8400-e29b-41d4-a716-446655440000",
  "eventType": "ORDER_STATUS_CHANGED",
  "occurredAt": "2026-06-23T10:30:00Z",
  "version": "1.0",
  "data": {
    "orderId": "ord-abc123",
    "orderCode": "DH001234",
    "previousStatus": "CONFIRMED",
    "currentStatus": "SHIPPING",
    "changedBy": "system",
    "changedAt": "2026-06-23T10:30:00Z"
  },
  "signature": "sha256=abc123def456..."
}
```

**Header của webhook request:**
```http
POST /webhook/receiver HTTP/1.1
Host: client-app.example.com
Content-Type: application/json
X-Webhook-ID: evt-550e8400-e29b-41d4-a716-446655440000
X-Webhook-Signature: sha256=abc123def456...
X-Webhook-Timestamp: 1750675800
X-Webhook-Version: 1.0
```

**Các loại sự kiện webhook:**

| Event Type | Mô tả |
|---|---|
| `ORDER_CREATED` | Đơn hàng mới được tạo |
| `ORDER_STATUS_CHANGED` | Trạng thái đơn hàng thay đổi |
| `ORDER_CANCELLED` | Đơn hàng bị hủy |
| `PAYMENT_CONFIRMED` | Thanh toán được xác nhận |
| `PAYMENT_FAILED` | Thanh toán thất bại |
| `PRODUCT_STOCK_LOW` | Tồn kho sản phẩm thấp hơn ngưỡng |
| `USER_REGISTERED` | Người dùng mới đăng ký |

**Yêu cầu với client nhận webhook:**
- Phải response `200 OK` trong vòng **5 giây**
- Nếu không response hoặc response khác 2xx, server sẽ retry theo lịch: 1m, 5m, 30m, 2h, 12h, 24h
- Phải xác thực chữ ký `X-Webhook-Signature` để đảm bảo webhook hợp lệ
- Phải xử lý idempotent (cùng `eventId` có thể nhận nhiều lần)

**Xác thực chữ ký webhook:**
```
signature = HMAC-SHA256(webhookSecret, timestamp + "." + rawRequestBody)
```

---

### 6.10 Job Status Polling

Dùng khi tác vụ xử lý bất đồng bộ (async job). Client poll endpoint trạng thái job.

**Lấy trạng thái job:**
```
GET /api/v1/jobs/job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4
```

**Response khi job đang chờ (PENDING):**
```json
{
  "data": {
    "jobId": "job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "type": "GENERATE_REPORT",
    "status": "PENDING",
    "progress": 0,
    "estimatedDuration": 30,
    "createdAt": "2026-06-23T10:00:00Z",
    "startedAt": null,
    "completedAt": null,
    "result": null,
    "error": null
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Response khi job đang xử lý (PROCESSING):**
```json
{
  "data": {
    "jobId": "job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "type": "GENERATE_REPORT",
    "status": "PROCESSING",
    "progress": 65,
    "estimatedDuration": 12,
    "createdAt": "2026-06-23T10:00:00Z",
    "startedAt": "2026-06-23T10:00:05Z",
    "completedAt": null,
    "result": null,
    "error": null
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Response khi job hoàn thành (COMPLETED):**
```json
{
  "data": {
    "jobId": "job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "type": "GENERATE_REPORT",
    "status": "COMPLETED",
    "progress": 100,
    "createdAt": "2026-06-23T10:00:00Z",
    "startedAt": "2026-06-23T10:00:05Z",
    "completedAt": "2026-06-23T10:00:28Z",
    "result": {
      "downloadUrl": "/api/v1/reports/rpt-uuid-123/download",
      "expiresAt": "2026-06-24T10:00:28Z",
      "fileSize": 1048576,
      "mimeType": "application/pdf"
    },
    "error": null
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Response khi job thất bại (FAILED):**
```json
{
  "data": {
    "jobId": "job-9f8e7d6c-5b4a-3c2d-1e0f-a9b8c7d6e5f4",
    "type": "GENERATE_REPORT",
    "status": "FAILED",
    "progress": 43,
    "createdAt": "2026-06-23T10:00:00Z",
    "startedAt": "2026-06-23T10:00:05Z",
    "completedAt": "2026-06-23T10:00:18Z",
    "result": null,
    "error": {
      "code": "RPT_003",
      "message": "Không đủ dữ liệu để tạo báo cáo cho khoảng thời gian đã chọn"
    }
  },
  "meta": { "requestId": "...", "timestamp": "...", "version": "1.0" }
}
```

**Các trạng thái job và vòng đời:**

```
[TẠO] ---> PENDING ---> PROCESSING ---> COMPLETED
                    |                   
                    +----> FAILED
                    |
                    +----> CANCELLED
```

| Trạng thái | Mô tả |
|---|---|
| `PENDING` | Đã nhận, đang chờ trong hàng đợi |
| `PROCESSING` | Đang xử lý |
| `COMPLETED` | Hoàn thành thành công |
| `FAILED` | Thất bại, không retry |
| `CANCELLED` | Đã bị hủy bởi người dùng |

**Khuyến nghị polling interval:**
- 2 giây cho 30 giây đầu
- 5 giây cho phút tiếp theo
- 15 giây sau đó
- Dừng poll sau 10 phút nếu vẫn không xong, hiển thị thông báo timeout cho người dùng

---

## 7. DINH DANG DU LIEU CHUAN

| Loại dữ liệu | Format chuẩn | Ví dụ | Ghi chú |
|---|---|---|---|
| **Date** | `YYYY-MM-DD` (ISO 8601) | `"2026-06-23"` | Không bao giờ dùng `23/06/2026` |
| **DateTime** | `YYYY-MM-DDTHH:mm:ssZ` (ISO 8601 UTC) | `"2026-06-23T10:30:00Z"` | Luôn UTC, chữ Z ở cuối |
| **DateTime có múi giờ** | `YYYY-MM-DDTHH:mm:ss+07:00` | `"2026-06-23T17:30:00+07:00"` | Dùng khi cần giữ múi giờ gốc |
| **Time** | `HH:mm:ss` | `"10:30:00"` | 24 giờ |
| **Duration** | ISO 8601 Duration hoặc giây | `"PT30M"` hoặc `1800` | Thống nhất trong một project |
| **Tiền tệ / Số tiền** | integer (đơn vị nhỏ nhất) | `150000` (VNĐ) | Không dùng float cho tiền! |
| **Tiền tệ USD** | integer (cents) | `1500` = $15.00 | |
| **Mã tiền tệ** | ISO 4217 (3 chữ hoa) | `"VND"`, `"USD"` | |
| **Số điện thoại** | E.164 format | `"+84901234567"` | Luôn có mã quốc gia |
| **Email** | Chuỗi lowercase | `"user@example.com"` | Validate theo RFC 5322 |
| **UUID** | lowercase với dấu gạch ngang | `"550e8400-e29b-41d4-a716-446655440000"` | v4 UUID |
| **Boolean** | `true` / `false` JSON native | `true` | Không dùng `"true"` (string) hay `1` |
| **Enum** | UPPER_SNAKE_CASE string | `"ORDER_CONFIRMED"` | Không dùng số nguyên cho enum |
| **Array rỗng** | `[]` | `"images": []` | Không trả về `null` cho array |
| **Object rỗng** | `{}` | `"metadata": {}` | Ưu tiên hơn `null` |
| **Giá trị không có** | `null` | `"deletedAt": null` | Chỉ dùng `null` khi field thực sự không có giá trị |
| **Field không tồn tại vs null** | Omit field | (không có key) | Nếu field không áp dụng, bỏ hẳn key đó ra khỏi JSON |
| **Số thập phân (tỷ lệ)** | Chuỗi hoặc float với precision rõ ràng | `"0.15"` hoặc `0.15` | Thống nhất trong API |
| **Mã màu** | Hex code | `"#FF5733"` | |
| **URL** | Absolute URL đầy đủ | `"https://cdn.example.com/img.jpg"` | Không dùng path tương đối |
| **Tọa độ địa lý** | GeoJSON hoặc `{lat, lng}` | `{"lat": 10.7769, "lng": 106.7009}` | |

**Ví dụ object với đầy đủ các kiểu dữ liệu:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "orderCode": "DH001234",
  "status": "CONFIRMED",
  "finalAmount": 330000,
  "currency": "VND",
  "isPaid": true,
  "customerPhone": "+84901234567",
  "customerEmail": "nguyen.van.an@gmail.com",
  "deliveryDate": "2026-06-25",
  "confirmedAt": "2026-06-23T10:05:00Z",
  "cancelledAt": null,
  "tags": ["uu-tien", "khach-vip"],
  "metadata": {},
  "shippingCoordinate": {
    "lat": 10.7769,
    "lng": 106.7009
  }
}
```

---

## 8. VERSIONING API

### 8.1 Quy Tắc Đặt Phiên Bản

API phiên bản được nhúng trực tiếp vào URL path:

```
https://api.example.com/api/v1/orders
https://api.example.com/api/v2/orders
```

- Phiên bản chính (`v1`, `v2`, `v3`) tăng khi có **thay đổi không tương thích ngược**
- Trong cùng phiên bản, chỉ được thêm field mới, **không được xóa hay đổi tên** field cũ
- Thay đổi không tương thích ngược bao gồm:
  - Xóa field trong response
  - Đổi tên field
  - Thay đổi kiểu dữ liệu của field
  - Thay đổi ngữ nghĩa của field
  - Xóa endpoint

### 8.2 Thay Đổi Tương Thích Ngược (Không Cần Tăng Phiên Bản)

Các thay đổi này **an toàn** và không cần tạo phiên bản mới:
- Thêm field mới vào response (client phải bỏ qua field không biết)
- Thêm endpoint mới
- Thêm giá trị mới vào enum (client phải xử lý unknown enum gracefully)
- Thêm optional field vào request
- Thay đổi giới hạn validation ít nghiêm ngặt hơn

### 8.3 Deprecated API

Khi API cũ sắp bị xóa, server phải trả về các header cảnh báo:

```http
HTTP/1.1 200 OK
Deprecation: 2027-01-01
Sunset: Mon, 01 Jan 2027 00:00:00 GMT
Link: <https://api.example.com/api/v2/orders>; rel="successor-version"
```

Timeline tiêu chuẩn:
- Thông báo deprecated tối thiểu **6 tháng** trước ngày ngừng
- Email thông báo gửi cho mọi đối tác đang dùng API đó

### 8.4 Gọi API Nhiều Phiên Bản Song Song

```
# Phiên bản cũ (đang deprecated)
GET /api/v1/orders

# Phiên bản mới
GET /api/v2/orders

# So sánh response v1 vs v2 (ví dụ field đổi tên)
# v1 trả về: "customerName"
# v2 trả về: "customer": { "id": "...", "fullName": "...", "email": "..." }
```

---

## 9. VI DU THUC TE DAY DU

### Ví Dụ: Order API - Toàn Bộ Vòng Đời Đơn Hàng

---

#### 9.1 POST /api/v1/orders - Tạo Đơn Hàng

**Request:**
```http
POST /api/v1/orders HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyLWFiYzEyMyIsImlhdCI6MTc1MDY3MjgwMCwiZXhwIjoxNzUwNjc2NDAwfQ.signature
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-ID: 7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d
Idempotency-Key: idem-23f8a4b1-cc4d-4e1a-9a23-44e6cc09f2b1
X-Client-Version: mobile-android/4.1.2

{
  "customerId": "cust-nguyen-van-an-001",
  "shippingAddress": {
    "recipientName": "Nguyễn Văn An",
    "recipientPhone": "+84901234567",
    "street": "123 Nguyễn Huệ, Phòng 501",
    "ward": "Bến Nghé",
    "district": "Quận 1",
    "city": "Hồ Chí Minh",
    "zipCode": "700000"
  },
  "items": [
    {
      "productId": "prod-ao-thun-cotton-001",
      "variantId": "var-size-l-color-white",
      "quantity": 2,
      "unitPrice": 150000
    },
    {
      "productId": "prod-quan-jean-slim-002",
      "variantId": "var-size-32-color-blue",
      "quantity": 1,
      "unitPrice": 350000
    }
  ],
  "couponCode": "SUMMER2026",
  "shippingMethod": "STANDARD",
  "paymentMethod": "COD",
  "note": "Giao hàng buổi sáng, gọi trước 30 phút. Không giao chiều tối."
}
```

**Response 201 - Tạo thành công:**
```json
{
  "data": {
    "id": "ord-550e8400-e29b-41d4-a716-446655440000",
    "orderCode": "DH20260623001234",
    "status": "PENDING",
    "customerId": "cust-nguyen-van-an-001",
    "customerName": "Nguyễn Văn An",
    "shippingAddress": {
      "recipientName": "Nguyễn Văn An",
      "recipientPhone": "+84901234567",
      "street": "123 Nguyễn Huệ, Phòng 501",
      "ward": "Bến Nghé",
      "district": "Quận 1",
      "city": "Hồ Chí Minh",
      "zipCode": "700000",
      "fullAddress": "123 Nguyễn Huệ, Phòng 501, Phường Bến Nghé, Quận 1, TP. Hồ Chí Minh"
    },
    "items": [
      {
        "id": "item-001",
        "productId": "prod-ao-thun-cotton-001",
        "productName": "Áo thun nam cotton 100%",
        "variantId": "var-size-l-color-white",
        "variantName": "Size L - Màu Trắng",
        "quantity": 2,
        "unitPrice": 150000,
        "subtotal": 300000,
        "imageUrl": "https://cdn.example.com/products/ao-thun-cotton-white.jpg"
      },
      {
        "id": "item-002",
        "productId": "prod-quan-jean-slim-002",
        "productName": "Quần jean slim fit nam",
        "variantId": "var-size-32-color-blue",
        "variantName": "Size 32 - Màu Xanh",
        "quantity": 1,
        "unitPrice": 350000,
        "subtotal": 350000,
        "imageUrl": "https://cdn.example.com/products/quan-jean-slim-blue.jpg"
      }
    ],
    "pricing": {
      "subtotal": 650000,
      "shippingFee": 30000,
      "discountAmount": 65000,
      "couponCode": "SUMMER2026",
      "couponDescription": "Giảm 10% tổng đơn hàng",
      "finalAmount": 615000
    },
    "shippingMethod": "STANDARD",
    "estimatedDelivery": {
      "from": "2026-06-25",
      "to": "2026-06-27"
    },
    "paymentMethod": "COD",
    "paymentStatus": "PENDING",
    "note": "Giao hàng buổi sáng, gọi trước 30 phút. Không giao chiều tối.",
    "createdAt": "2026-06-23T10:00:00Z",
    "updatedAt": "2026-06-23T10:00:00Z",
    "confirmedAt": null,
    "shippedAt": null,
    "deliveredAt": null,
    "cancelledAt": null
  },
  "meta": {
    "requestId": "7f3d2b1a-9e4c-4b8a-a3f2-1d5e6c7b8a9d",
    "timestamp": "2026-06-23T10:00:00Z",
    "version": "1.0"
  }
}
```

Kèm header: `Location: /api/v1/orders/ord-550e8400-e29b-41d4-a716-446655440000`

**Response 422 - Lỗi validation khi tạo đơn:**
```json
{
  "error": {
    "code": "VAL_001",
    "message": "Dữ liệu đầu vào không hợp lệ",
    "details": [
      {
        "field": "items[0].quantity",
        "code": "STOCK_001",
        "message": "Số lượng yêu cầu (2) vượt quá tồn kho hiện tại (1) của sản phẩm 'Áo thun nam cotton 100% - Size L - Màu Trắng'"
      },
      {
        "field": "couponCode",
        "code": "COUPON_002",
        "message": "Mã giảm giá 'SUMMER2026' đã hết số lần sử dụng"
      }
    ],
    "traceId": "trace-7f3d2b1a9e4c4b8a"
  }
}
```

---

#### 9.2 GET /api/v1/orders/{id} - Xem Chi Tiết Đơn Hàng

**Request:**
```http
GET /api/v1/orders/ord-550e8400-e29b-41d4-a716-446655440000 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
X-Request-ID: 8a4b2c1d-3e5f-6a7b-8c9d-0e1f2a3b4c5d
```

**Response 200 - Thành công:**
```json
{
  "data": {
    "id": "ord-550e8400-e29b-41d4-a716-446655440000",
    "orderCode": "DH20260623001234",
    "status": "CONFIRMED",
    "customerId": "cust-nguyen-van-an-001",
    "customerName": "Nguyễn Văn An",
    "shippingAddress": {
      "recipientName": "Nguyễn Văn An",
      "recipientPhone": "+84901234567",
      "street": "123 Nguyễn Huệ, Phòng 501",
      "ward": "Bến Nghé",
      "district": "Quận 1",
      "city": "Hồ Chí Minh",
      "zipCode": "700000",
      "fullAddress": "123 Nguyễn Huệ, Phòng 501, Phường Bến Nghé, Quận 1, TP. Hồ Chí Minh"
    },
    "items": [
      {
        "id": "item-001",
        "productId": "prod-ao-thun-cotton-001",
        "productName": "Áo thun nam cotton 100%",
        "variantId": "var-size-l-color-white",
        "variantName": "Size L - Màu Trắng",
        "quantity": 2,
        "unitPrice": 150000,
        "subtotal": 300000,
        "imageUrl": "https://cdn.example.com/products/ao-thun-cotton-white.jpg"
      },
      {
        "id": "item-002",
        "productId": "prod-quan-jean-slim-002",
        "productName": "Quần jean slim fit nam",
        "variantId": "var-size-32-color-blue",
        "variantName": "Size 32 - Màu Xanh",
        "quantity": 1,
        "unitPrice": 350000,
        "subtotal": 350000,
        "imageUrl": "https://cdn.example.com/products/quan-jean-slim-blue.jpg"
      }
    ],
    "pricing": {
      "subtotal": 650000,
      "shippingFee": 30000,
      "discountAmount": 65000,
      "couponCode": "SUMMER2026",
      "couponDescription": "Giảm 10% tổng đơn hàng",
      "finalAmount": 615000
    },
    "shippingMethod": "STANDARD",
    "trackingCode": "GHN-VN20260623456789",
    "trackingUrl": "https://tracking.ghn.dev?code=GHN-VN20260623456789",
    "estimatedDelivery": {
      "from": "2026-06-25",
      "to": "2026-06-27"
    },
    "paymentMethod": "COD",
    "paymentStatus": "PENDING",
    "note": "Giao hàng buổi sáng, gọi trước 30 phút. Không giao chiều tối.",
    "statusHistory": [
      {
        "status": "PENDING",
        "note": "Đơn hàng được tạo",
        "changedBy": "customer",
        "changedAt": "2026-06-23T10:00:00Z"
      },
      {
        "status": "CONFIRMED",
        "note": "Đơn hàng được xác nhận bởi nhân viên kho",
        "changedBy": "staff-warehouse-001",
        "changedAt": "2026-06-23T10:30:00Z"
      }
    ],
    "createdAt": "2026-06-23T10:00:00Z",
    "updatedAt": "2026-06-23T10:30:00Z",
    "confirmedAt": "2026-06-23T10:30:00Z",
    "shippedAt": null,
    "deliveredAt": null,
    "cancelledAt": null
  },
  "meta": {
    "requestId": "8a4b2c1d-3e5f-6a7b-8c9d-0e1f2a3b4c5d",
    "timestamp": "2026-06-23T11:00:00Z",
    "version": "1.0"
  }
}
```

**Response 404 - Không tìm thấy:**
```json
{
  "error": {
    "code": "ORDER_001",
    "message": "Không tìm thấy đơn hàng với ID: ord-550e8400-e29b-41d4-a716-446655440000",
    "traceId": "trace-8a4b2c1d3e5f6a7b"
  }
}
```

**Response 403 - Không có quyền (đơn hàng của người khác):**
```json
{
  "error": {
    "code": "ORDER_004",
    "message": "Bạn không có quyền xem đơn hàng này",
    "traceId": "trace-8a4b2c1d3e5f6a7b"
  }
}
```

---

#### 9.3 GET /api/v1/orders - Danh Sách Đơn Hàng

**Request:**
```http
GET /api/v1/orders?status=CONFIRMED,SHIPPING&createdAtFrom=2026-06-01&createdAtTo=2026-06-23&page=1&pageSize=20&sortBy=createdAt&sortOrder=desc HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Accept: application/json
X-Request-ID: 9b5c3d2e-4f6a-7b8c-9d0e-1f2a3b4c5d6e
```

**Response 200 - Danh sách:**
```json
{
  "data": {
    "items": [
      {
        "id": "ord-550e8400-e29b-41d4-a716-446655440000",
        "orderCode": "DH20260623001234",
        "status": "CONFIRMED",
        "customerName": "Nguyễn Văn An",
        "customerPhone": "+84901234567",
        "itemCount": 2,
        "finalAmount": 615000,
        "paymentMethod": "COD",
        "paymentStatus": "PENDING",
        "shippingMethod": "STANDARD",
        "createdAt": "2026-06-23T10:00:00Z",
        "updatedAt": "2026-06-23T10:30:00Z"
      },
      {
        "id": "ord-660f9511-f30c-52e5-b827-557766551111",
        "orderCode": "DH20260622005678",
        "status": "SHIPPING",
        "customerName": "Trần Thị Bình",
        "customerPhone": "+84912345678",
        "itemCount": 3,
        "finalAmount": 780000,
        "paymentMethod": "BANK_TRANSFER",
        "paymentStatus": "PAID",
        "shippingMethod": "EXPRESS",
        "createdAt": "2026-06-22T14:00:00Z",
        "updatedAt": "2026-06-23T08:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "total": 47,
      "totalPages": 3,
      "hasNextPage": true,
      "hasPrevPage": false
    }
  },
  "meta": {
    "requestId": "9b5c3d2e-4f6a-7b8c-9d0e-1f2a3b4c5d6e",
    "timestamp": "2026-06-23T11:00:00Z",
    "version": "1.0"
  }
}
```

---

#### 9.4 PATCH /api/v1/orders/{id}/cancel - Hủy Đơn Hàng

**Request:**
```http
PATCH /api/v1/orders/ord-550e8400-e29b-41d4-a716-446655440000/cancel HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-ID: 0c6d4e3f-5a7b-8c9d-0e1f-2a3b4c5d6e7f
Idempotency-Key: idem-cancel-550e8400-abc123

{
  "reason": "CUSTOMER_REQUEST",
  "reasonDetail": "Khách hàng đổi ý, muốn đặt đơn khác"
}
```

**Response 200 - Hủy thành công:**
```json
{
  "data": {
    "id": "ord-550e8400-e29b-41d4-a716-446655440000",
    "orderCode": "DH20260623001234",
    "status": "CANCELLED",
    "cancellationReason": "CUSTOMER_REQUEST",
    "cancellationDetail": "Khách hàng đổi ý, muốn đặt đơn khác",
    "cancelledAt": "2026-06-23T11:15:00Z",
    "cancelledBy": "cust-nguyen-van-an-001",
    "refund": {
      "status": "NOT_APPLICABLE",
      "reason": "Đơn hàng thanh toán COD chưa thu tiền"
    },
    "updatedAt": "2026-06-23T11:15:00Z"
  },
  "meta": {
    "requestId": "0c6d4e3f-5a7b-8c9d-0e1f-2a3b4c5d6e7f",
    "timestamp": "2026-06-23T11:15:00Z",
    "version": "1.0"
  }
}
```

**Response 409 - Không thể hủy (đã giao đơn vị vận chuyển):**
```json
{
  "error": {
    "code": "ORDER_003",
    "message": "Không thể hủy đơn hàng DH20260623001234 vì đã được giao cho đơn vị vận chuyển (GHN). Vui lòng liên hệ hotline 1900-xxxx để được hỗ trợ.",
    "context": {
      "orderId": "ord-550e8400-e29b-41d4-a716-446655440000",
      "currentStatus": "SHIPPING",
      "trackingCode": "GHN-VN20260623456789",
      "allowedCancelStatuses": ["PENDING", "CONFIRMED"],
      "supportPhone": "1900-xxxx"
    },
    "traceId": "trace-0c6d4e3f5a7b8c9d"
  }
}
```

**Giải thích các lý do hủy đơn (enum `reason`):**

| Giá trị | Mô tả |
|---|---|
| `CUSTOMER_REQUEST` | Khách hàng tự yêu cầu hủy |
| `OUT_OF_STOCK` | Sản phẩm hết hàng |
| `PAYMENT_FAILED` | Thanh toán thất bại |
| `ADDRESS_INVALID` | Địa chỉ giao hàng không hợp lệ |
| `DUPLICATE_ORDER` | Đơn hàng bị đặt trùng |
| `FRAUD_SUSPECTED` | Nghi ngờ gian lận |
| `SYSTEM_ERROR` | Lỗi hệ thống |
| `OTHER` | Lý do khác (bắt buộc điền `reasonDetail`) |

---

### Tóm Tắt Mã Lỗi Nghiệp Vụ

| Mã lỗi | HTTP Status | Ý nghĩa |
|---|---|---|
| `AUTH_001` | 401 | Token không hợp lệ hoặc hết hạn |
| `AUTH_002` | 401 | Thiếu Authorization header |
| `AUTHZ_001` | 403 | Không có quyền truy cập resource |
| `AUTHZ_002` | 403 | Không có quyền thực hiện thao tác |
| `VAL_001` | 422 | Lỗi validation tổng quát (xem `details`) |
| `VAL_002` | 422 | Field bắt buộc bị thiếu |
| `VAL_003` | 422 | Sai định dạng giá trị |
| `VAL_004` | 422 | Giá trị vượt quá giới hạn cho phép |
| `VAL_005` | 422 | Giá trị phải lớn hơn 0 |
| `VAL_006` | 422 | Giá trị không thuộc danh sách hợp lệ |
| `ORDER_001` | 404 | Không tìm thấy đơn hàng |
| `ORDER_002` | 409 | Trạng thái đơn hàng không cho phép thao tác này |
| `ORDER_003` | 409 | Không thể hủy đơn đã giao vận chuyển |
| `ORDER_004` | 403 | Không có quyền xem đơn hàng này |
| `PROD_001` | 404 | Không tìm thấy sản phẩm |
| `PROD_002` | 409 | Sản phẩm đang có đơn hàng active |
| `STOCK_001` | 422 | Số lượng vượt tồn kho |
| `COUPON_001` | 422 | Mã giảm giá không tồn tại |
| `COUPON_002` | 422 | Mã giảm giá đã hết lượt dùng |
| `COUPON_003` | 422 | Mã giảm giá chưa đến thời gian hiệu lực |
| `COUPON_004` | 422 | Mã giảm giá đã hết hạn |
| `RATE_001` | 429 | Vượt rate limit |
| `REQ_001` | 400 | Request body không phải JSON hợp lệ |
| `REQ_002` | 400 | Content-Type không được hỗ trợ |
| `FILE_001` | 400 | File quá lớn |
| `FILE_002` | 202 | File đang được tạo |
| `FILE_003` | 415 | Định dạng file không được hỗ trợ |
| `SYS_001` | 500 | Lỗi hệ thống không mong đợi |
| `SYS_002` | 503 | Hệ thống đang bảo trì |

---

*Tài liệu này được duy trì bởi nhóm Platform Engineering.*
*Mọi thay đổi phải được review và phê duyệt trước khi áp dụng.*
*Phiên bản hiện tại: 1.0.0 - Cập nhật: 2026-06-23*
