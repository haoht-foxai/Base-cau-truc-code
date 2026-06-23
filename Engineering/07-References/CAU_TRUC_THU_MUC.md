# CẤU TRÚC THƯ MỤC PROJECT

> Tài liệu tham khảo thực hành - Copy trực tiếp vào project của bạn

---

## Triết Lý Cốt Lõi

**Feature-based, không phải Type-based.**

Tổ chức code theo **tính năng nghiệp vụ** (business feature), không phải theo **loại kỹ thuật** (technical type).

**Lý do:**
- Khi thêm tính năng "Thanh toán", bạn chỉ cần vào thư mục `payments/` - tất cả code liên quan đều ở đó.
- Không phải chạy khắp nơi: `models/payment.py`, `controllers/payment.py`, `services/payment.py`, `repositories/payment.py`.
- Dễ xóa tính năng: chỉ xóa một thư mục thay vì tìm kiếm và xóa nhiều file rải rác.
- Dễ làm việc nhóm: mỗi team sở hữu một feature folder riêng.
- Dễ chuyển sang microservice: mỗi feature folder có thể tách ra độc lập.

---

## 1. Cấu Trúc Chung (Áp dụng mọi ngôn ngữ)

Đây là cấu trúc "xương sống" áp dụng được cho hầu hết mọi project, bất kể ngôn ngữ hay framework.

```
project-root/
├── .github/                    # Cấu hình GitHub (CI/CD, PR templates, issue templates)
│   ├── workflows/              # GitHub Actions workflow files
│   │   ├── ci.yml              # Chạy test và lint khi có push/PR
│   │   ├── cd.yml              # Deploy khi merge vào main
│   │   └── release.yml         # Tạo release version tự động
│   ├── ISSUE_TEMPLATE/         # Mẫu tạo issue chuẩn
│   │   ├── bug_report.md
│   │   └── feature_request.md
│   └── pull_request_template.md
│
├── docs/                       # Tài liệu kỹ thuật của project
│   ├── architecture/           # Sơ đồ kiến trúc, ADR (Architecture Decision Records)
│   │   ├── overview.md         # Tổng quan kiến trúc
│   │   ├── adr-001-database-choice.md
│   │   └── adr-002-auth-strategy.md
│   ├── api/                    # Tài liệu API (nếu không dùng Swagger tự động)
│   ├── deployment/             # Hướng dẫn triển khai
│   └── onboarding.md           # Hướng dẫn cho developer mới
│
├── src/                        # Toàn bộ source code của ứng dụng
│   ├── features/               # Các tính năng nghiệp vụ (xem Section 2)
│   ├── shared/                 # Code dùng chung giữa các features
│   │   ├── exceptions/         # Custom exceptions/errors
│   │   ├── middleware/         # Middleware tái sử dụng
│   │   ├── utils/              # Helper functions, utility functions
│   │   ├── constants/          # Hằng số dùng toàn ứng dụng
│   │   └── types/              # Type definitions, interfaces dùng chung
│   ├── infrastructure/         # Kết nối hạ tầng bên ngoài
│   │   ├── database/           # Database connection, migrations, seeders
│   │   ├── cache/              # Redis, Memcached configuration
│   │   ├── messaging/          # Message queue (RabbitMQ, Kafka)
│   │   ├── storage/            # File storage (S3, local)
│   │   └── email/              # Email service provider
│   └── config/                 # Cấu hình ứng dụng đọc từ environment variables
│
├── tests/                      # Tất cả tests (unit, integration, e2e)
│   ├── unit/                   # Unit tests - test từng hàm độc lập
│   ├── integration/            # Integration tests - test nhiều layer cùng nhau
│   ├── e2e/                    # End-to-end tests - test toàn bộ luồng
│   └── fixtures/               # Dữ liệu mẫu dùng trong tests
│
├── scripts/                    # Shell scripts tiện ích cho developer
│   ├── setup.sh                # Cài đặt môi trường development lần đầu
│   ├── seed.sh                 # Chạy seeder để có dữ liệu test
│   ├── migrate.sh              # Chạy database migrations
│   └── generate.sh             # Code generation scripts
│
├── deployments/                # Tất cả files liên quan đến deployment
│   ├── docker/                 # Dockerfiles cho các môi trường khác nhau
│   │   ├── Dockerfile.dev      # Development image
│   │   ├── Dockerfile.prod     # Production image (multi-stage build)
│   │   └── Dockerfile.test     # Test image
│   ├── kubernetes/             # Kubernetes manifests
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── ingress.yaml
│   │   └── configmap.yaml
│   └── terraform/              # Infrastructure as Code
│
├── .env.example                # Mẫu biến môi trường - COMMIT vào git
├── .env                        # Biến môi trường thực - KHÔNG commit vào git
├── .gitignore                  # Danh sách file không commit
├── .editorconfig               # Cấu hình editor thống nhất cho team
├── .dockerignore               # File không copy vào Docker image
├── docker-compose.yml          # Local development với Docker
├── docker-compose.test.yml     # Test environment với Docker
├── Makefile                    # Các lệnh thường dùng (make test, make run, v.v.)
├── README.md                   # Giới thiệu project, hướng dẫn cài đặt nhanh
└── CHANGELOG.md                # Lịch sử thay đổi theo version
```

---

## 2. Vertical Slice / Feature-Based (Khuyến nghị)

Đây là cách tổ chức code **được khuyến nghị nhất**. Mỗi feature là một "lát cắt dọc" xuyên suốt từ API đến database.

Ví dụ: Hệ thống e-commerce với 5 modules chính.

```
src/
├── features/
│   │
│   ├── users/                              # Module quản lý người dùng
│   │   ├── domain/                         # Logic nghiệp vụ thuần túy
│   │   │   ├── user.entity.ts              # Entity User với các business rules
│   │   │   ├── user-role.enum.ts           # Enum: ADMIN, CUSTOMER, VENDOR
│   │   │   ├── user.repository.interface.ts # Interface repository (không phụ thuộc DB)
│   │   │   └── user.domain-service.ts      # Business logic phức tạp liên quan User
│   │   │
│   │   ├── application/                    # Use cases / Application services
│   │   │   ├── commands/                   # Thay đổi dữ liệu (CQRS Command side)
│   │   │   │   ├── register-user/
│   │   │   │   │   ├── register-user.command.ts
│   │   │   │   │   ├── register-user.handler.ts
│   │   │   │   │   └── register-user.dto.ts
│   │   │   │   ├── update-user-profile/
│   │   │   │   │   ├── update-user-profile.command.ts
│   │   │   │   │   ├── update-user-profile.handler.ts
│   │   │   │   │   └── update-user-profile.dto.ts
│   │   │   │   ├── change-password/
│   │   │   │   │   ├── change-password.command.ts
│   │   │   │   │   └── change-password.handler.ts
│   │   │   │   └── deactivate-user/
│   │   │   │       ├── deactivate-user.command.ts
│   │   │   │       └── deactivate-user.handler.ts
│   │   │   ├── queries/                    # Truy vấn dữ liệu (CQRS Query side)
│   │   │   │   ├── get-user-by-id/
│   │   │   │   │   ├── get-user-by-id.query.ts
│   │   │   │   │   ├── get-user-by-id.handler.ts
│   │   │   │   │   └── get-user-by-id.result.ts
│   │   │   │   └── list-users/
│   │   │   │       ├── list-users.query.ts
│   │   │   │       └── list-users.handler.ts
│   │   │   └── events/                     # Domain event handlers
│   │   │       └── user-registered.handler.ts
│   │   │
│   │   ├── infrastructure/                 # Triển khai cụ thể (DB, Email, v.v.)
│   │   │   ├── persistence/
│   │   │   │   ├── user.repository.ts      # Implement UserRepository interface
│   │   │   │   ├── user.schema.ts          # Database schema / ORM model
│   │   │   │   └── user.mapper.ts          # Chuyển đổi giữa DB model và Domain entity
│   │   │   └── services/
│   │   │       └── bcrypt-password.service.ts # Implement IPasswordService
│   │   │
│   │   ├── api/                            # HTTP interface layer
│   │   │   ├── users.controller.ts         # Route handlers
│   │   │   ├── users.routes.ts             # Route definitions
│   │   │   ├── dtos/                       # Request/Response DTOs
│   │   │   │   ├── create-user.request.dto.ts
│   │   │   │   ├── update-user.request.dto.ts
│   │   │   │   └── user.response.dto.ts
│   │   │   └── validators/                 # Input validation rules
│   │   │       └── user.validator.ts
│   │   │
│   │   └── users.module.ts                 # Module registration (DI container)
│   │
│   ├── orders/                             # Module quản lý đơn hàng
│   │   ├── domain/
│   │   │   ├── order.entity.ts             # Order entity với trạng thái: PENDING, CONFIRMED, SHIPPED, DELIVERED
│   │   │   ├── order-item.value-object.ts  # Value object cho từng sản phẩm trong đơn
│   │   │   ├── order-status.enum.ts
│   │   │   ├── order.repository.interface.ts
│   │   │   └── order.domain-service.ts     # Tính tổng tiền, áp dụng discount, v.v.
│   │   │
│   │   ├── application/
│   │   │   ├── commands/
│   │   │   │   ├── create-order/
│   │   │   │   │   ├── create-order.command.ts
│   │   │   │   │   ├── create-order.handler.ts
│   │   │   │   │   └── create-order.dto.ts
│   │   │   │   ├── cancel-order/
│   │   │   │   │   ├── cancel-order.command.ts
│   │   │   │   │   └── cancel-order.handler.ts
│   │   │   │   └── confirm-order/
│   │   │   │       ├── confirm-order.command.ts
│   │   │   │       └── confirm-order.handler.ts
│   │   │   ├── queries/
│   │   │   │   ├── get-order-by-id/
│   │   │   │   │   ├── get-order-by-id.query.ts
│   │   │   │   │   └── get-order-by-id.handler.ts
│   │   │   │   └── get-user-orders/
│   │   │   │       ├── get-user-orders.query.ts
│   │   │   │       └── get-user-orders.handler.ts
│   │   │   └── events/
│   │   │       ├── order-created.handler.ts    # Gửi email xác nhận khi tạo đơn
│   │   │       └── order-shipped.handler.ts    # Gửi thông báo khi đơn được giao
│   │   │
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── order.repository.ts
│   │   │   │   ├── order.schema.ts
│   │   │   │   └── order.mapper.ts
│   │   │   └── external/
│   │   │       └── shipping.service.ts     # Tích hợp API đơn vị vận chuyển
│   │   │
│   │   ├── api/
│   │   │   ├── orders.controller.ts
│   │   │   ├── orders.routes.ts
│   │   │   └── dtos/
│   │   │       ├── create-order.request.dto.ts
│   │   │       └── order.response.dto.ts
│   │   │
│   │   └── orders.module.ts
│   │
│   ├── products/                           # Module quản lý sản phẩm
│   │   ├── domain/
│   │   │   ├── product.entity.ts           # Product với giá, tồn kho, SKU
│   │   │   ├── category.entity.ts
│   │   │   ├── price.value-object.ts       # Bao gồm amount và currency
│   │   │   ├── product.repository.interface.ts
│   │   │   └── product.domain-service.ts   # Kiểm tra tồn kho, tính giá sau khuyến mãi
│   │   │
│   │   ├── application/
│   │   │   ├── commands/
│   │   │   │   ├── create-product/
│   │   │   │   │   ├── create-product.command.ts
│   │   │   │   │   ├── create-product.handler.ts
│   │   │   │   │   └── create-product.dto.ts
│   │   │   │   ├── update-product/
│   │   │   │   │   ├── update-product.command.ts
│   │   │   │   │   └── update-product.handler.ts
│   │   │   │   ├── update-stock/
│   │   │   │   │   ├── update-stock.command.ts
│   │   │   │   │   └── update-stock.handler.ts
│   │   │   │   └── delete-product/
│   │   │   │       ├── delete-product.command.ts
│   │   │   │       └── delete-product.handler.ts
│   │   │   └── queries/
│   │   │       ├── get-product-by-id/
│   │   │       │   ├── get-product-by-id.query.ts
│   │   │       │   └── get-product-by-id.handler.ts
│   │   │       ├── search-products/
│   │   │       │   ├── search-products.query.ts
│   │   │       │   └── search-products.handler.ts
│   │   │       └── get-products-by-category/
│   │   │           ├── get-products-by-category.query.ts
│   │   │           └── get-products-by-category.handler.ts
│   │   │
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── product.repository.ts
│   │   │   │   ├── product.schema.ts
│   │   │   │   └── product.mapper.ts
│   │   │   └── search/
│   │   │       └── elasticsearch-product.search.ts
│   │   │
│   │   ├── api/
│   │   │   ├── products.controller.ts
│   │   │   ├── products.routes.ts
│   │   │   └── dtos/
│   │   │       ├── create-product.request.dto.ts
│   │   │       └── product.response.dto.ts
│   │   │
│   │   └── products.module.ts
│   │
│   ├── payments/                           # Module xử lý thanh toán
│   │   ├── domain/
│   │   │   ├── payment.entity.ts           # Payment với trạng thái: PENDING, SUCCESS, FAILED, REFUNDED
│   │   │   ├── payment-method.enum.ts      # CREDIT_CARD, BANK_TRANSFER, E_WALLET, COD
│   │   │   ├── payment.repository.interface.ts
│   │   │   └── payment-gateway.interface.ts # Interface cho payment gateway
│   │   │
│   │   ├── application/
│   │   │   ├── commands/
│   │   │   │   ├── process-payment/
│   │   │   │   │   ├── process-payment.command.ts
│   │   │   │   │   └── process-payment.handler.ts
│   │   │   │   └── refund-payment/
│   │   │   │       ├── refund-payment.command.ts
│   │   │   │       └── refund-payment.handler.ts
│   │   │   ├── queries/
│   │   │   │   └── get-payment-status/
│   │   │   │       ├── get-payment-status.query.ts
│   │   │   │       └── get-payment-status.handler.ts
│   │   │   └── events/
│   │   │       ├── payment-success.handler.ts  # Confirm order sau khi thanh toán thành công
│   │   │       └── payment-failed.handler.ts   # Thông báo thất bại
│   │   │
│   │   ├── infrastructure/
│   │   │   ├── persistence/
│   │   │   │   ├── payment.repository.ts
│   │   │   │   └── payment.schema.ts
│   │   │   └── gateways/
│   │   │       ├── stripe.payment-gateway.ts   # Implement IPaymentGateway với Stripe
│   │   │       ├── vnpay.payment-gateway.ts    # Implement IPaymentGateway với VNPay
│   │   │       └── momo.payment-gateway.ts     # Implement IPaymentGateway với MoMo
│   │   │
│   │   ├── api/
│   │   │   ├── payments.controller.ts
│   │   │   ├── payments.routes.ts
│   │   │   └── webhooks/
│   │   │       ├── stripe-webhook.handler.ts   # Xử lý webhook từ Stripe
│   │   │       └── vnpay-webhook.handler.ts
│   │   │
│   │   └── payments.module.ts
│   │
│   └── notifications/                      # Module gửi thông báo
│       ├── domain/
│       │   ├── notification.entity.ts      # Notification với: type, channel, status
│       │   ├── notification-channel.enum.ts # EMAIL, SMS, PUSH, IN_APP
│       │   ├── notification-type.enum.ts   # ORDER_CONFIRMED, PAYMENT_RECEIVED, v.v.
│       │   └── notification-provider.interface.ts
│       │
│       ├── application/
│       │   ├── commands/
│       │   │   ├── send-email/
│       │   │   │   ├── send-email.command.ts
│       │   │   │   └── send-email.handler.ts
│       │   │   ├── send-sms/
│       │   │   │   ├── send-sms.command.ts
│       │   │   │   └── send-sms.handler.ts
│       │   │   └── send-push-notification/
│       │   │       ├── send-push-notification.command.ts
│       │   │       └── send-push-notification.handler.ts
│       │   └── events/
│       │       └── notification-sent.handler.ts
│       │
│       ├── infrastructure/
│       │   └── providers/
│       │       ├── sendgrid.email-provider.ts
│       │       ├── twilio.sms-provider.ts
│       │       └── firebase.push-provider.ts
│       │
│       ├── api/
│       │   ├── notifications.controller.ts
│       │   └── dtos/
│       │       └── notification.response.dto.ts
│       │
│       └── notifications.module.ts
│
├── shared/                                 # Code dùng chung (không phụ thuộc feature)
│   ├── domain/
│   │   ├── base.entity.ts                  # Base class cho tất cả entities (id, createdAt, updatedAt)
│   │   ├── aggregate-root.ts               # Base class cho Aggregate Roots
│   │   └── domain-event.ts                 # Base class cho Domain Events
│   ├── exceptions/
│   │   ├── not-found.exception.ts
│   │   ├── unauthorized.exception.ts
│   │   ├── validation.exception.ts
│   │   └── conflict.exception.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts              # Xác thực JWT token
│   │   ├── rate-limit.middleware.ts        # Rate limiting
│   │   └── request-logger.middleware.ts
│   ├── guards/
│   │   ├── roles.guard.ts                  # Kiểm tra quyền truy cập theo role
│   │   └── jwt-auth.guard.ts
│   ├── decorators/
│   │   ├── current-user.decorator.ts       # Lấy user từ request context
│   │   └── roles.decorator.ts
│   ├── pipes/
│   │   ├── parse-uuid.pipe.ts
│   │   └── validation.pipe.ts
│   ├── interceptors/
│   │   ├── response-transform.interceptor.ts  # Chuẩn hóa response format
│   │   └── cache.interceptor.ts
│   ├── utils/
│   │   ├── date.utils.ts                   # Xử lý ngày tháng
│   │   ├── string.utils.ts                 # Xử lý chuỗi
│   │   ├── pagination.utils.ts             # Phân trang
│   │   └── crypto.utils.ts                 # Mã hóa, hash
│   └── constants/
│       ├── app.constants.ts
│       └── http-status.constants.ts
│
└── infrastructure/                         # Kết nối hạ tầng (setup 1 lần)
    ├── database/
    │   ├── database.module.ts
    │   ├── database.config.ts
    │   └── migrations/
    │       ├── 1700000001-create-users-table.ts
    │       ├── 1700000002-create-products-table.ts
    │       ├── 1700000003-create-orders-table.ts
    │       └── 1700000004-create-payments-table.ts
    ├── cache/
    │   ├── cache.module.ts
    │   └── redis.config.ts
    └── messaging/
        ├── message-bus.module.ts
        └── rabbitmq.config.ts
```

---

## 3. Clean Architecture (Onion Architecture)

Kiến trúc này chia code thành các tầng (layers) đồng tâm. Phụ thuộc chỉ hướng vào trong - tầng trong không biết tầng ngoài tồn tại.

```
src/
├── domain/                         # Tầng trong cùng - Không phụ thuộc bất kỳ thứ gì
│   │                               # Chứa: Entities, Value Objects, Domain Events, Repository Interfaces
│   ├── entities/
│   │   ├── user.entity.ts          # User với logic: isActive(), hasPermission()
│   │   ├── order.entity.ts         # Order với logic: canBeCancelled(), calculateTotal()
│   │   ├── product.entity.ts
│   │   └── payment.entity.ts
│   │
│   ├── value-objects/              # Immutable objects, không có identity
│   │   ├── email.value-object.ts   # Validation email trong constructor
│   │   ├── money.value-object.ts   # amount + currency, tránh floating point
│   │   ├── address.value-object.ts
│   │   └── phone-number.value-object.ts
│   │
│   ├── events/                     # Domain Events - điều gì đó đã xảy ra
│   │   ├── user-registered.event.ts
│   │   ├── order-placed.event.ts
│   │   ├── order-cancelled.event.ts
│   │   └── payment-processed.event.ts
│   │
│   ├── exceptions/                 # Domain-specific exceptions
│   │   ├── user-not-found.exception.ts
│   │   ├── insufficient-stock.exception.ts
│   │   └── payment-failed.exception.ts
│   │
│   ├── repositories/               # Chỉ là INTERFACE - không có implementation
│   │   ├── user.repository.interface.ts
│   │   ├── order.repository.interface.ts
│   │   ├── product.repository.interface.ts
│   │   └── payment.repository.interface.ts
│   │
│   └── services/                   # Domain Services - logic không thuộc về Entity cụ thể nào
│       ├── order-pricing.service.ts    # Tính giá, áp dụng discount cho order
│       └── inventory.service.ts        # Quản lý tồn kho phức tạp
│
├── application/                    # Tầng thứ 2 - Biết về Domain, không biết về Infrastructure/UI
│   │                               # Chứa: Use Cases, Application Services, DTOs, Port Interfaces
│   ├── use-cases/
│   │   ├── user/
│   │   │   ├── register-user.use-case.ts       # Orchestrate: validate -> create -> save -> send email
│   │   │   ├── login-user.use-case.ts
│   │   │   └── get-user-profile.use-case.ts
│   │   ├── order/
│   │   │   ├── create-order.use-case.ts
│   │   │   ├── cancel-order.use-case.ts
│   │   │   └── get-order-history.use-case.ts
│   │   ├── product/
│   │   │   ├── create-product.use-case.ts
│   │   │   └── search-products.use-case.ts
│   │   └── payment/
│   │       ├── process-payment.use-case.ts
│   │       └── refund-payment.use-case.ts
│   │
│   ├── dtos/                       # Data Transfer Objects - format dữ liệu giữa layers
│   │   ├── user/
│   │   │   ├── create-user.dto.ts
│   │   │   ├── update-user.dto.ts
│   │   │   └── user-response.dto.ts
│   │   └── order/
│   │       ├── create-order.dto.ts
│   │       └── order-response.dto.ts
│   │
│   └── ports/                      # Interfaces cho External Services (Email, SMS, v.v.)
│       ├── email-sender.port.ts    # Interface gửi email - không biết SendGrid hay SMTP
│       ├── sms-sender.port.ts
│       ├── file-storage.port.ts
│       └── payment-gateway.port.ts
│
├── infrastructure/                 # Tầng thứ 3 - Implement các Interfaces từ Domain/Application
│   │                               # Chứa: Repository implementations, External service adapters
│   ├── persistence/
│   │   ├── typeorm/                # Hoặc prisma/, sequelize/, mongoose/
│   │   │   ├── entities/           # ORM entity schemas
│   │   │   │   ├── user.orm-entity.ts
│   │   │   │   └── order.orm-entity.ts
│   │   │   ├── repositories/       # Implement domain repository interfaces
│   │   │   │   ├── typeorm-user.repository.ts
│   │   │   │   └── typeorm-order.repository.ts
│   │   │   ├── mappers/            # Convert ORM entity <-> Domain entity
│   │   │   │   ├── user.mapper.ts
│   │   │   │   └── order.mapper.ts
│   │   │   └── migrations/
│   │   │       ├── 1700000001-CreateUsersTable.ts
│   │   │       └── 1700000002-CreateOrdersTable.ts
│   │   └── in-memory/              # Dùng cho testing
│   │       ├── in-memory-user.repository.ts
│   │       └── in-memory-order.repository.ts
│   │
│   ├── adapters/                   # Implement Port interfaces từ Application layer
│   │   ├── sendgrid.email-adapter.ts       # Implement EmailSenderPort
│   │   ├── twilio.sms-adapter.ts           # Implement SmsSenderPort
│   │   ├── s3.file-storage-adapter.ts      # Implement FileStoragePort
│   │   ├── stripe.payment-gateway-adapter.ts
│   │   └── redis.cache-adapter.ts
│   │
│   └── config/
│       ├── database.config.ts
│       ├── redis.config.ts
│       └── app.config.ts
│
└── presentation/                   # Tầng ngoài cùng - Giao diện với thế giới bên ngoài
    │                               # Chứa: Controllers, GraphQL Resolvers, gRPC handlers, CLI commands
    ├── http/
    │   ├── controllers/
    │   │   ├── users.controller.ts
    │   │   ├── orders.controller.ts
    │   │   └── products.controller.ts
    │   ├── middleware/
    │   │   ├── auth.middleware.ts
    │   │   └── error-handler.middleware.ts
    │   └── routes/
    │       ├── user.routes.ts
    │       └── order.routes.ts
    │
    ├── graphql/                    # Nếu dùng GraphQL
    │   ├── resolvers/
    │   │   ├── user.resolver.ts
    │   │   └── order.resolver.ts
    │   └── types/
    │       ├── user.type.ts
    │       └── order.type.ts
    │
    └── cli/                        # CLI commands (cron jobs, admin scripts)
        ├── seed-database.command.ts
        └── cleanup-old-orders.command.ts
```

**Quy tắc phụ thuộc (Dependency Rule):**
- `domain` -> không phụ thuộc gì
- `application` -> phụ thuộc `domain`
- `infrastructure` -> phụ thuộc `domain` + `application`
- `presentation` -> phụ thuộc `application`

---

## 4. Modular Monolith

Ứng dụng một khối nhưng được chia thành các modules độc lập với ranh giới rõ ràng. Mỗi module là một "mini application" tự chứa.

```
src/
├── modules/
│   ├── user-management/            # Module quản lý người dùng
│   │   ├── domain/
│   │   │   ├── user.entity.ts
│   │   │   └── user.repository.interface.ts
│   │   ├── application/
│   │   │   ├── register-user.use-case.ts
│   │   │   └── user.service.ts
│   │   ├── infrastructure/
│   │   │   ├── user.repository.ts
│   │   │   └── user.schema.ts
│   │   ├── api/
│   │   │   ├── users.controller.ts
│   │   │   └── users.routes.ts
│   │   └── index.ts                # PUBLIC API của module - chỉ export những gì cho phép dùng
│   │                               # export { UserService, RegisterUserDto } - KHÔNG export internals
│   │
│   ├── catalog/                    # Module quản lý danh mục sản phẩm
│   │   ├── domain/
│   │   │   ├── product.entity.ts
│   │   │   ├── category.entity.ts
│   │   │   └── product.repository.interface.ts
│   │   ├── application/
│   │   │   ├── create-product.use-case.ts
│   │   │   └── search-products.use-case.ts
│   │   ├── infrastructure/
│   │   │   ├── product.repository.ts
│   │   │   └── product.schema.ts
│   │   ├── api/
│   │   │   ├── products.controller.ts
│   │   │   └── products.routes.ts
│   │   └── index.ts                # export { ProductService, ProductDto }
│   │
│   ├── ordering/                   # Module xử lý đơn hàng
│   │   ├── domain/
│   │   │   ├── order.entity.ts
│   │   │   ├── order-item.value-object.ts
│   │   │   └── order.repository.interface.ts
│   │   ├── application/
│   │   │   ├── create-order.use-case.ts    # Dùng ProductService từ catalog module qua public API
│   │   │   └── order.service.ts
│   │   ├── infrastructure/
│   │   │   ├── order.repository.ts
│   │   │   └── order.schema.ts
│   │   ├── api/
│   │   │   ├── orders.controller.ts
│   │   │   └── orders.routes.ts
│   │   └── index.ts                # export { OrderService, CreateOrderDto }
│   │
│   ├── billing/                    # Module thanh toán
│   │   ├── domain/
│   │   │   ├── invoice.entity.ts
│   │   │   └── payment.entity.ts
│   │   ├── application/
│   │   │   └── process-payment.use-case.ts
│   │   ├── infrastructure/
│   │   │   ├── payment.repository.ts
│   │   │   └── stripe.client.ts
│   │   ├── api/
│   │   │   └── billing.controller.ts
│   │   └── index.ts
│   │
│   └── notifications/              # Module thông báo
│       ├── domain/
│       │   └── notification.entity.ts
│       ├── application/
│       │   └── send-notification.use-case.ts
│       ├── infrastructure/
│       │   ├── sendgrid.client.ts
│       │   └── twilio.client.ts
│       ├── api/
│       │   └── notifications.controller.ts
│       └── index.ts
│
├── shared-kernel/                  # Code dùng chung - tối thiểu hóa, chỉ để "ngôn ngữ chung"
│   ├── domain/
│   │   ├── base.entity.ts
│   │   └── domain-event.ts
│   ├── value-objects/
│   │   ├── money.value-object.ts   # Dùng chung vì nhiều module cần
│   │   └── email.value-object.ts
│   └── events/
│       └── event-bus.interface.ts  # Interface pub/sub giữa các modules
│
└── app.ts                          # Khởi động ứng dụng, import tất cả modules
```

**Nguyên tắc ranh giới module:**
- Module A KHÔNG được import trực tiếp từ bên trong module B
- Module A chỉ được dùng những gì module B export qua `index.ts`
- Giao tiếp giữa modules qua Events (pub/sub) để giảm coupling

---

## 5. Microservice (Single Service Structure)

Cấu trúc cho MỘT microservice. Mỗi service là một ứng dụng độc lập, deploy riêng.

```
order-service/                      # Tên service rõ ràng theo nghiệp vụ
├── src/
│   ├── domain/
│   │   ├── order.entity.ts
│   │   ├── order-item.value-object.ts
│   │   ├── order-status.enum.ts
│   │   ├── order.repository.interface.ts
│   │   └── events/
│   │       ├── order-created.event.ts
│   │       ├── order-cancelled.event.ts
│   │       └── order-shipped.event.ts
│   │
│   ├── application/
│   │   ├── use-cases/
│   │   │   ├── create-order.use-case.ts
│   │   │   ├── cancel-order.use-case.ts
│   │   │   └── get-order.use-case.ts
│   │   ├── event-handlers/             # Xử lý events từ services khác
│   │   │   ├── payment-confirmed.handler.ts    # Từ payment-service
│   │   │   └── inventory-reserved.handler.ts  # Từ inventory-service
│   │   └── ports/
│   │       ├── product-catalog.port.ts # Interface để gọi product-service
│   │       └── event-publisher.port.ts
│   │
│   ├── infrastructure/
│   │   ├── persistence/
│   │   │   ├── order.repository.ts
│   │   │   ├── order.schema.ts
│   │   │   └── migrations/
│   │   ├── http-clients/
│   │   │   └── product-catalog.http-client.ts  # Gọi product-service qua HTTP
│   │   ├── messaging/
│   │   │   ├── kafka-event-publisher.ts        # Publish events lên Kafka
│   │   │   └── kafka-consumer.ts               # Subscribe events từ Kafka
│   │   └── cache/
│   │       └── redis-cache.ts
│   │
│   ├── api/
│   │   ├── http/                       # REST API
│   │   │   ├── orders.controller.ts
│   │   │   ├── orders.routes.ts
│   │   │   └── dtos/
│   │   │       ├── create-order.request.dto.ts
│   │   │       └── order.response.dto.ts
│   │   ├── grpc/                       # gRPC (cho internal service communication)
│   │   │   ├── order.proto
│   │   │   └── order.grpc-handler.ts
│   │   └── health/
│   │       └── health.controller.ts    # /health, /ready endpoints bắt buộc
│   │
│   ├── config/
│   │   ├── app.config.ts
│   │   ├── database.config.ts
│   │   └── kafka.config.ts
│   │
│   └── main.ts                         # Entry point
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── deployments/
│   ├── Dockerfile
│   └── kubernetes/
│       ├── deployment.yaml
│       ├── service.yaml
│       └── configmap.yaml
│
├── .env.example
├── package.json                        # Hoặc pom.xml, go.mod, requirements.txt
└── README.md                           # Mô tả service này làm gì, API docs link
```

---

## 6. Cấu Trúc Theo Ngôn Ngữ

### C# / .NET

```
EcommerceApp/                           # Solution folder
├── EcommerceApp.sln                    # Solution file - mở bằng Visual Studio
│
├── src/
│   ├── EcommerceApp.Domain/            # Domain layer project
│   │   ├── Entities/
│   │   │   ├── User.cs
│   │   │   ├── Order.cs
│   │   │   └── Product.cs
│   │   ├── ValueObjects/
│   │   │   ├── Money.cs
│   │   │   └── Email.cs
│   │   ├── Events/
│   │   │   └── OrderCreatedEvent.cs
│   │   ├── Repositories/
│   │   │   └── IOrderRepository.cs     # Interface only
│   │   ├── Exceptions/
│   │   │   └── OrderNotFoundException.cs
│   │   └── EcommerceApp.Domain.csproj  # Không reference project nào khác
│   │
│   ├── EcommerceApp.Application/       # Application layer project
│   │   ├── Features/
│   │   │   ├── Orders/
│   │   │   │   ├── Commands/
│   │   │   │   │   ├── CreateOrder/
│   │   │   │   │   │   ├── CreateOrderCommand.cs
│   │   │   │   │   │   ├── CreateOrderCommandHandler.cs
│   │   │   │   │   │   └── CreateOrderCommandValidator.cs  # FluentValidation
│   │   │   │   │   └── CancelOrder/
│   │   │   │   │       ├── CancelOrderCommand.cs
│   │   │   │   │       └── CancelOrderCommandHandler.cs
│   │   │   │   └── Queries/
│   │   │   │       └── GetOrderById/
│   │   │   │           ├── GetOrderByIdQuery.cs
│   │   │   │           ├── GetOrderByIdQueryHandler.cs
│   │   │   │           └── OrderDto.cs
│   │   │   └── Users/
│   │   │       └── Commands/
│   │   │           └── RegisterUser/
│   │   │               ├── RegisterUserCommand.cs
│   │   │               └── RegisterUserCommandHandler.cs
│   │   ├── Behaviours/                 # MediatR pipeline behaviours
│   │   │   ├── ValidationBehaviour.cs
│   │   │   ├── LoggingBehaviour.cs
│   │   │   └── CachingBehaviour.cs
│   │   ├── Interfaces/
│   │   │   └── IEmailService.cs
│   │   └── EcommerceApp.Application.csproj  # Reference: Domain
│   │
│   ├── EcommerceApp.Infrastructure/    # Infrastructure layer project
│   │   ├── Persistence/
│   │   │   ├── ApplicationDbContext.cs # EF Core DbContext
│   │   │   ├── Configurations/
│   │   │   │   ├── OrderConfiguration.cs   # Fluent API config
│   │   │   │   └── UserConfiguration.cs
│   │   │   ├── Repositories/
│   │   │   │   └── OrderRepository.cs  # Implement IOrderRepository
│   │   │   └── Migrations/
│   │   │       └── 20240101000000_InitialCreate.cs
│   │   ├── Services/
│   │   │   └── EmailService.cs         # Implement IEmailService
│   │   ├── DependencyInjection.cs      # Extension method đăng ký services
│   │   └── EcommerceApp.Infrastructure.csproj  # Reference: Domain, Application
│   │
│   └── EcommerceApp.WebApi/            # Presentation layer project
│       ├── Controllers/
│       │   ├── OrdersController.cs
│       │   └── UsersController.cs
│       ├── Middleware/
│       │   └── ExceptionHandlingMiddleware.cs
│       ├── Extensions/
│       │   └── ServiceCollectionExtensions.cs
│       ├── Program.cs                  # .NET 6+ entry point
│       ├── appsettings.json
│       ├── appsettings.Development.json
│       └── EcommerceApp.WebApi.csproj  # Reference: Application, Infrastructure
│
├── tests/
│   ├── EcommerceApp.Domain.Tests/
│   │   ├── Entities/
│   │   │   └── OrderTests.cs
│   │   └── EcommerceApp.Domain.Tests.csproj
│   ├── EcommerceApp.Application.Tests/
│   │   ├── Features/
│   │   │   └── Orders/
│   │   │       └── CreateOrderCommandHandlerTests.cs
│   │   └── EcommerceApp.Application.Tests.csproj
│   └── EcommerceApp.WebApi.Tests/
│       ├── Controllers/
│       │   └── OrdersControllerTests.cs
│       └── EcommerceApp.WebApi.Tests.csproj
│
├── .editorconfig
├── .gitignore
├── docker-compose.yml
└── README.md
```

---

### Java / Spring Boot

```
ecommerce-app/
├── pom.xml                             # Maven build file (hoặc build.gradle cho Gradle)
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── company/
│   │   │           └── ecommerce/      # Base package
│   │   │               ├── EcommerceApplication.java  # @SpringBootApplication entry
│   │   │               │
│   │   │               ├── domain/     # Tầng domain - không Spring dependency
│   │   │               │   ├── model/
│   │   │               │   │   ├── Order.java          # Java record hoặc class thuần
│   │   │               │   │   ├── OrderItem.java
│   │   │               │   │   └── OrderStatus.java    # Enum
│   │   │               │   ├── repository/
│   │   │               │   │   └── OrderRepository.java # Interface
│   │   │               │   ├── service/
│   │   │               │   │   └── OrderDomainService.java
│   │   │               │   └── exception/
│   │   │               │       └── OrderNotFoundException.java
│   │   │               │
│   │   │               ├── application/ # Use cases
│   │   │               │   ├── order/
│   │   │               │   │   ├── CreateOrderUseCase.java
│   │   │               │   │   ├── CancelOrderUseCase.java
│   │   │               │   │   └── GetOrderUseCase.java
│   │   │               │   ├── dto/
│   │   │               │   │   ├── CreateOrderRequest.java  # Record (Java 16+)
│   │   │               │   │   └── OrderResponse.java
│   │   │               │   └── port/
│   │   │               │       └── EmailPort.java       # Interface cho external service
│   │   │               │
│   │   │               ├── infrastructure/
│   │   │               │   ├── persistence/
│   │   │               │   │   ├── jpa/
│   │   │               │   │   │   ├── OrderJpaEntity.java  # @Entity
│   │   │               │   │   │   ├── OrderJpaRepository.java # extends JpaRepository
│   │   │               │   │   │   └── OrderJpaRepositoryAdapter.java # Implement domain OrderRepository
│   │   │               │   │   └── mapper/
│   │   │               │   │       └── OrderMapper.java  # MapStruct mapper
│   │   │               │   ├── email/
│   │   │               │   │   └── SendGridEmailAdapter.java # Implement EmailPort
│   │   │               │   └── config/
│   │   │               │       ├── DatabaseConfig.java   # @Configuration
│   │   │               │       └── SecurityConfig.java   # Spring Security config
│   │   │               │
│   │   │               └── presentation/
│   │   │                   ├── api/
│   │   │                   │   ├── OrderController.java  # @RestController
│   │   │                   │   └── UserController.java
│   │   │                   ├── advice/
│   │   │                   │   └── GlobalExceptionHandler.java # @RestControllerAdvice
│   │   │                   └── filter/
│   │   │                       └── JwtAuthenticationFilter.java
│   │   │
│   │   └── resources/
│   │       ├── application.yml             # Cấu hình chính
│   │       ├── application-dev.yml         # Override cho development
│   │       ├── application-prod.yml        # Override cho production
│   │       ├── application-test.yml        # Override cho testing
│   │       └── db/
│   │           └── migration/              # Flyway migrations
│   │               ├── V1__Create_users_table.sql
│   │               ├── V2__Create_products_table.sql
│   │               └── V3__Create_orders_table.sql
│   │
│   └── test/
│       └── java/
│           └── com/
│               └── company/
│                   └── ecommerce/
│                       ├── domain/
│                       │   └── model/
│                       │       └── OrderTest.java
│                       ├── application/
│                       │   └── order/
│                       │       └── CreateOrderUseCaseTest.java
│                       └── presentation/
│                           └── api/
│                               └── OrderControllerTest.java    # @SpringBootTest
│
├── .mvn/                               # Maven wrapper
├── mvnw                                # Maven wrapper script (Linux/Mac)
├── mvnw.cmd                            # Maven wrapper script (Windows)
├── .gitignore
├── docker-compose.yml
└── README.md
```

---

### Python / FastAPI hoặc Django

```
ecommerce_app/                          # Project root
├── pyproject.toml                      # Hoặc requirements.txt + setup.py
├── poetry.lock                         # Lock file của Poetry
├── alembic.ini                         # Cấu hình database migrations (Alembic)
│
├── alembic/                            # Database migrations
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│       ├── 001_create_users_table.py
│       └── 002_create_orders_table.py
│
├── src/
│   └── ecommerce/                      # Main package
│       ├── __init__.py
│       ├── main.py                     # FastAPI app entry point
│       ├── settings.py                 # Pydantic BaseSettings cho config
│       │
│       ├── features/                   # Feature-based modules
│       │   ├── users/
│       │   │   ├── __init__.py
│       │   │   ├── router.py           # APIRouter với các endpoints
│       │   │   ├── service.py          # Business logic
│       │   │   ├── repository.py       # Database operations
│       │   │   ├── models.py           # SQLAlchemy models
│       │   │   ├── schemas.py          # Pydantic schemas (request/response)
│       │   │   ├── exceptions.py       # User-specific exceptions
│       │   │   └── dependencies.py     # FastAPI dependencies (get_current_user)
│       │   │
│       │   ├── orders/
│       │   │   ├── __init__.py
│       │   │   ├── router.py
│       │   │   ├── service.py
│       │   │   ├── repository.py
│       │   │   ├── models.py
│       │   │   ├── schemas.py
│       │   │   │   # CreateOrderRequest, OrderResponse, OrderListResponse
│       │   │   └── exceptions.py
│       │   │
│       │   ├── products/
│       │   │   ├── __init__.py
│       │   │   ├── router.py
│       │   │   ├── service.py
│       │   │   ├── repository.py
│       │   │   ├── models.py
│       │   │   └── schemas.py
│       │   │
│       │   └── payments/
│       │       ├── __init__.py
│       │       ├── router.py
│       │       ├── service.py
│       │       ├── repository.py
│       │       ├── models.py
│       │       ├── schemas.py
│       │       └── providers/          # Payment provider integrations
│       │           ├── __init__.py
│       │           ├── stripe_provider.py
│       │           └── vnpay_provider.py
│       │
│       ├── shared/                     # Code dùng chung
│       │   ├── __init__.py
│       │   ├── database.py             # SQLAlchemy engine, session factory
│       │   ├── base_model.py           # Base SQLAlchemy model với id, created_at
│       │   ├── exceptions.py           # HTTPException helpers
│       │   ├── pagination.py           # Pydantic schema cho phân trang
│       │   ├── security.py             # JWT creation/verification
│       │   └── middleware/
│       │       ├── __init__.py
│       │       ├── auth.py
│       │       └── logging.py
│       │
│       └── infrastructure/
│           ├── __init__.py
│           ├── cache.py                # Redis client setup
│           └── email.py                # Email service
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                     # Pytest fixtures dùng chung
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_order_service.py
│   │   └── test_user_service.py
│   ├── integration/
│   │   ├── __init__.py
│   │   └── test_order_repository.py
│   └── e2e/
│       ├── __init__.py
│       └── test_create_order_flow.py
│
├── scripts/
│   ├── seed_db.py
│   └── create_admin.py
│
├── .env.example
├── .env
├── .gitignore
├── docker-compose.yml
├── Dockerfile
└── README.md
```

---

### Go

```
ecommerce-service/
├── go.mod                              # Module definition: module github.com/company/ecommerce
├── go.sum                              # Checksums cho dependencies
├── Makefile                            # make build, make test, make run
│
├── cmd/                                # Entry points (có thể có nhiều binaries)
│   ├── api/
│   │   └── main.go                     # HTTP API server entry point
│   ├── worker/
│   │   └── main.go                     # Background worker entry point
│   └── migrate/
│       └── main.go                     # Database migration tool
│
├── internal/                           # Code PRIVATE - không thể import từ ngoài module
│   ├── domain/                         # Domain layer - pure Go, no framework
│   │   ├── order/
│   │   │   ├── order.go                # Order struct, methods, business rules
│   │   │   ├── order_item.go
│   │   │   ├── order_status.go         # type OrderStatus string + constants
│   │   │   ├── order_repository.go     # type Repository interface{}
│   │   │   └── order_service.go        # Domain service
│   │   └── product/
│   │       ├── product.go
│   │       └── product_repository.go
│   │
│   ├── application/                    # Use cases
│   │   ├── order/
│   │   │   ├── create_order.go         # type CreateOrderUseCase struct{}
│   │   │   ├── create_order_dto.go     # type CreateOrderInput, CreateOrderOutput
│   │   │   ├── cancel_order.go
│   │   │   └── get_order.go
│   │   └── product/
│   │       └── create_product.go
│   │
│   ├── infrastructure/                 # Concrete implementations
│   │   ├── postgres/
│   │   │   ├── order_repository.go     # Implement domain.order.Repository
│   │   │   ├── product_repository.go
│   │   │   └── db.go                   # Connection pool setup
│   │   ├── redis/
│   │   │   └── cache.go
│   │   └── email/
│   │       └── sendgrid_client.go
│   │
│   └── api/                            # HTTP handlers
│       ├── handler/
│       │   ├── order_handler.go        # HTTP handlers cho orders
│       │   └── product_handler.go
│       ├── middleware/
│       │   ├── auth.go
│       │   └── logging.go
│       ├── dto/
│       │   ├── order_request.go
│       │   └── order_response.go
│       └── router.go                   # Route setup
│
├── pkg/                                # Code PUBLIC - có thể import từ ngoài module
│   ├── validator/                      # Generic validation utilities
│   │   └── validator.go
│   ├── logger/                         # Logger wrapper
│   │   └── logger.go
│   └── errors/                         # Custom error types
│       └── errors.go
│
├── migrations/                         # SQL migration files
│   ├── 000001_create_users_table.up.sql
│   ├── 000001_create_users_table.down.sql
│   ├── 000002_create_products_table.up.sql
│   └── 000002_create_products_table.down.sql
│
├── configs/
│   ├── config.go                       # Config struct với env tags
│   └── config.yaml                     # Default config values
│
├── tests/
│   ├── integration/
│   │   └── order_repository_test.go
│   └── e2e/
│       └── create_order_test.go
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
└── README.md
```

---

### NodeJS / TypeScript

```
ecommerce-api/
├── tsconfig.json                       # TypeScript compiler options
├── tsconfig.build.json                 # Build-specific TS config (exclude tests)
├── package.json
├── package-lock.json                   # Hoặc yarn.lock / pnpm-lock.yaml
├── .eslintrc.json                      # ESLint config
├── .prettierrc                         # Prettier formatting rules
├── jest.config.ts                      # Jest test configuration
│
├── src/
│   ├── main.ts                         # Application bootstrap
│   ├── app.ts                          # Express/Fastify app setup
│   │
│   ├── features/
│   │   ├── users/
│   │   │   ├── domain/
│   │   │   │   ├── user.entity.ts
│   │   │   │   ├── user-role.enum.ts
│   │   │   │   └── user.repository.interface.ts
│   │   │   ├── application/
│   │   │   │   ├── register-user.use-case.ts
│   │   │   │   ├── login-user.use-case.ts
│   │   │   │   └── dtos/
│   │   │   │       ├── register-user.dto.ts
│   │   │   │       └── login-user.dto.ts
│   │   │   ├── infrastructure/
│   │   │   │   ├── prisma-user.repository.ts   # Implement interface
│   │   │   │   └── bcrypt.password-service.ts
│   │   │   ├── api/
│   │   │   │   ├── users.controller.ts
│   │   │   │   ├── users.router.ts
│   │   │   │   └── validators/
│   │   │   │       └── user.validator.ts       # zod schema
│   │   │   └── users.module.ts
│   │   │
│   │   └── orders/
│   │       ├── domain/
│   │       │   ├── order.entity.ts
│   │       │   └── order.repository.interface.ts
│   │       ├── application/
│   │       │   └── create-order.use-case.ts
│   │       ├── infrastructure/
│   │       │   └── prisma-order.repository.ts
│   │       ├── api/
│   │       │   ├── orders.controller.ts
│   │       │   └── orders.router.ts
│   │       └── orders.module.ts
│   │
│   ├── shared/
│   │   ├── domain/
│   │   │   └── base.entity.ts
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts
│   │   │   ├── error-handler.middleware.ts
│   │   │   └── request-logger.middleware.ts
│   │   ├── guards/
│   │   │   └── auth.guard.ts
│   │   ├── types/
│   │   │   ├── express.d.ts            # Augment Express Request type
│   │   │   └── common.types.ts
│   │   └── utils/
│   │       ├── date.utils.ts
│   │       └── pagination.utils.ts
│   │
│   └── infrastructure/
│       ├── database/
│       │   ├── prisma.client.ts        # Singleton PrismaClient
│       │   └── prisma/
│       │       └── schema.prisma       # Prisma schema
│       ├── cache/
│       │   └── redis.client.ts
│       └── config/
│           └── app.config.ts           # env variables với Joi/zod validation
│
├── tests/
│   ├── unit/
│   │   ├── features/
│   │   │   └── users/
│   │   │       └── register-user.use-case.spec.ts
│   │   └── shared/
│   ├── integration/
│   │   └── features/
│   │       └── orders/
│   │           └── orders.controller.spec.ts
│   └── e2e/
│       └── create-order-flow.e2e-spec.ts
│
├── dist/                               # Compiled JavaScript output (gitignored)
├── node_modules/                       # Dependencies (gitignored)
├── .env.example
├── .gitignore
├── docker-compose.yml
├── Dockerfile
└── README.md
```

---

### PHP / Laravel

Laravel có cấu trúc riêng nhưng có thể tổ chức theo feature bằng cách dùng Service Layer.

```
ecommerce-laravel/
├── app/
│   ├── Console/
│   │   └── Commands/
│   │       └── ProcessPendingOrdersCommand.php # Artisan command
│   │
│   ├── Features/                       # Feature modules (tổ chức custom)
│   │   ├── Users/
│   │   │   ├── Actions/               # Single-action classes (như Use Cases)
│   │   │   │   ├── RegisterUserAction.php
│   │   │   │   └── UpdateUserProfileAction.php
│   │   │   ├── DTOs/
│   │   │   │   ├── RegisterUserData.php    # Spatie data hoặc class thuần
│   │   │   │   └── UserProfileData.php
│   │   │   ├── Repositories/
│   │   │   │   ├── UserRepositoryInterface.php
│   │   │   │   └── EloquentUserRepository.php
│   │   │   └── Services/
│   │   │       └── UserService.php
│   │   │
│   │   ├── Orders/
│   │   │   ├── Actions/
│   │   │   │   ├── CreateOrderAction.php
│   │   │   │   └── CancelOrderAction.php
│   │   │   ├── DTOs/
│   │   │   │   └── CreateOrderData.php
│   │   │   ├── Repositories/
│   │   │   │   ├── OrderRepositoryInterface.php
│   │   │   │   └── EloquentOrderRepository.php
│   │   │   ├── Services/
│   │   │   │   └── OrderService.php
│   │   │   └── Events/
│   │   │       ├── OrderCreated.php
│   │   │       └── OrderCancelled.php
│   │   │
│   │   ├── Products/
│   │   │   ├── Actions/
│   │   │   │   ├── CreateProductAction.php
│   │   │   │   └── UpdateProductAction.php
│   │   │   ├── DTOs/
│   │   │   │   └── CreateProductData.php
│   │   │   └── Repositories/
│   │   │       └── EloquentProductRepository.php
│   │   │
│   │   └── Payments/
│   │       ├── Actions/
│   │       │   └── ProcessPaymentAction.php
│   │       ├── Gateways/
│   │       │   ├── PaymentGatewayInterface.php
│   │       │   ├── StripeGateway.php
│   │       │   └── VnPayGateway.php
│   │       └── Webhooks/
│   │           ├── StripeWebhookHandler.php
│   │           └── VnPayWebhookHandler.php
│   │
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Api/
│   │   │   │   ├── UserController.php  # Gọi Action/Service, không có logic
│   │   │   │   ├── OrderController.php
│   │   │   │   └── ProductController.php
│   │   │   └── Webhooks/
│   │   │       └── PaymentWebhookController.php
│   │   ├── Middleware/
│   │   │   └── CheckSubscription.php
│   │   └── Requests/
│   │       ├── Users/
│   │       │   └── RegisterUserRequest.php   # Form Request validation
│   │       └── Orders/
│   │           └── CreateOrderRequest.php
│   │
│   ├── Models/                         # Eloquent models (Laravel convention)
│   │   ├── User.php
│   │   ├── Order.php
│   │   ├── OrderItem.php
│   │   └── Product.php
│   │
│   └── Providers/
│       ├── AppServiceProvider.php      # Bind interfaces to implementations
│       └── EventServiceProvider.php
│
├── database/
│   ├── migrations/
│   │   ├── 2024_01_01_000000_create_users_table.php
│   │   ├── 2024_01_01_000001_create_products_table.php
│   │   └── 2024_01_01_000002_create_orders_table.php
│   ├── factories/
│   │   ├── UserFactory.php
│   │   └── OrderFactory.php
│   └── seeders/
│       ├── DatabaseSeeder.php
│       └── ProductSeeder.php
│
├── routes/
│   ├── api.php                         # API routes
│   ├── web.php                         # Web routes
│   └── console.php                     # Artisan command routes
│
├── config/
│   ├── app.php
│   ├── database.php
│   └── payment.php                     # Custom config cho payment
│
├── tests/
│   ├── Feature/
│   │   ├── Users/
│   │   │   └── RegisterUserTest.php
│   │   └── Orders/
│   │       └── CreateOrderTest.php
│   └── Unit/
│       ├── Actions/
│       │   └── CreateOrderActionTest.php
│       └── Services/
│           └── OrderServiceTest.php
│
├── .env.example
├── artisan
├── composer.json
├── composer.lock
├── docker-compose.yml
└── README.md
```

---

## 7. Cấu Trúc File Test

Tests phải **phản chiếu** cấu trúc source code. Dễ tìm test khi biết file source.

```
# Ví dụ với TypeScript/NestJS

src/
├── features/
│   ├── orders/
│   │   ├── application/
│   │   │   ├── commands/
│   │   │   │   └── create-order/
│   │   │   │       └── create-order.handler.ts       # Source file
│   │   │   └── queries/
│   │   │       └── get-order-by-id/
│   │   │           └── get-order-by-id.handler.ts    # Source file
│   │   ├── domain/
│   │   │   └── order.entity.ts                       # Source file
│   │   └── api/
│   │       └── orders.controller.ts                  # Source file

tests/
├── unit/                               # Test nhanh, không cần database/external
│   └── features/
│       └── orders/
│           ├── application/
│           │   ├── commands/
│           │   │   └── create-order/
│           │   │       └── create-order.handler.spec.ts  # Mirror source path
│           │   └── queries/
│           │       └── get-order-by-id/
│           │           └── get-order-by-id.handler.spec.ts
│           └── domain/
│               └── order.entity.spec.ts              # Test business rules
│
├── integration/                        # Test với database thật (dùng test DB)
│   └── features/
│       └── orders/
│           ├── infrastructure/
│           │   └── order.repository.spec.ts          # Test DB queries
│           └── api/
│               └── orders.controller.spec.ts         # Test HTTP layer
│
├── e2e/                                # Test toàn bộ luồng qua HTTP
│   ├── create-order-flow.e2e-spec.ts   # Từ tạo user -> tạo sản phẩm -> đặt hàng -> thanh toán
│   ├── cancel-order-flow.e2e-spec.ts
│   └── user-registration-flow.e2e-spec.ts
│
└── fixtures/                           # Dữ liệu mẫu dùng trong tests
    ├── users/
    │   ├── valid-user.fixture.ts
    │   └── admin-user.fixture.ts
    ├── orders/
    │   ├── pending-order.fixture.ts
    │   └── completed-order.fixture.ts
    └── products/
        └── available-product.fixture.ts
```

**Quy tắc đặt tên file test:**
- Unit test: `<tên-file>.spec.ts` hoặc `<tên-file>.test.ts`
- Integration test: `<tên-file>.integration.spec.ts`
- E2E test: `<tên-feature>.e2e-spec.ts`
- Python: `test_<tên-file>.py`
- Go: `<tên-file>_test.go` (cùng thư mục với source)
- Java: `<TênClass>Test.java` hoặc `<TênClass>IT.java` (Integration Test)

---

## 8. File Cấu Hình Ở Root

Tất cả file cấu hình quan trọng phải ở root của project.

```
project-root/
├── .gitignore                  # LUÔN có - liệt kê file không commit vào git
├── .gitattributes              # Cấu hình git cho line endings, diff, merge
│
├── .env.example                # LUÔN COMMIT - mẫu biến môi trường, không có giá trị thật
├── .env                        # KHÔNG BAO GIỜ COMMIT - giá trị thật cho local
├── .env.test                   # Biến môi trường cho test
│
├── .editorconfig               # Quy tắc formatting chung: indent, charset, newline
│                               # Hỗ trợ VS Code, JetBrains, Vim, Emacs
│
├── README.md                   # Giới thiệu project, quick start, links tới docs
├── CHANGELOG.md                # Lịch sử thay đổi theo SemVer (vd: ## [1.2.0] - 2024-01-15)
├── CONTRIBUTING.md             # Hướng dẫn đóng góp cho open source projects
├── LICENSE                     # Giấy phép phần mềm
│
├── Makefile                    # Các lệnh thường dùng:
│                               #   make install  - cài đặt dependencies
│                               #   make dev      - chạy development server
│                               #   make test     - chạy tất cả tests
│                               #   make build    - build production
│                               #   make lint     - chạy linter
│                               #   make migrate  - chạy database migrations
│                               #   make seed     - seed database
│                               #   make clean    - xóa build artifacts
│
├── docker-compose.yml          # Local development environment
│                               # Services: app, database, redis, mailhog
├── docker-compose.test.yml     # Test environment (extend docker-compose.yml)
├── docker-compose.prod.yml     # Production-like environment cho testing
├── Dockerfile                  # Production Docker image (multi-stage build)
│
├── .dockerignore               # Files không copy vào Docker image
│                               # node_modules/, .git/, *.log, .env
│
├── sonar-project.properties    # SonarQube code quality configuration
├── .codecov.yml                # Codecov coverage thresholds
│
├── # --- Linting & Formatting ---
├── .eslintrc.json              # ESLint rules (JavaScript/TypeScript)
├── .eslintignore               # Files ESLint bỏ qua
├── .prettierrc                 # Prettier formatting rules
├── .prettierignore
├── pyproject.toml              # Black formatter config (Python)
├── .flake8                     # Flake8 linter config (Python)
├── .golangci.yml               # golangci-lint config (Go)
├── checkstyle.xml              # Checkstyle config (Java)
├── phpcs.xml                   # PHP CodeSniffer config
│
├── # --- Testing ---
├── jest.config.ts              # Jest configuration (Node.js)
├── pytest.ini                  # Pytest configuration (Python)
├── phpunit.xml                 # PHPUnit configuration (PHP)
│
├── # --- CI/CD ---
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── cd.yml
├── .gitlab-ci.yml              # GitLab CI (nếu dùng GitLab)
├── Jenkinsfile                 # Jenkins pipeline (nếu dùng Jenkins)
├── azure-pipelines.yml         # Azure DevOps pipeline
│
└── # --- Language specific ---
    ├── package.json            # Node.js
    ├── tsconfig.json           # TypeScript
    ├── go.mod                  # Go
    ├── pom.xml                 # Java Maven
    ├── build.gradle            # Java Gradle
    ├── requirements.txt        # Python (pip)
    ├── pyproject.toml          # Python (Poetry/modern)
    ├── composer.json           # PHP
    ├── Gemfile                 # Ruby
    └── *.csproj / *.sln       # .NET
```

**Nội dung .env.example mẫu:**

```dotenv
# Application
APP_NAME=EcommerceApp
APP_ENV=development
APP_PORT=3000
APP_SECRET=your-secret-key-here

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=ecommerce_db
DATABASE_USER=postgres
DATABASE_PASSWORD=your-password-here
DATABASE_POOL_SIZE=10

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=

# JWT
JWT_SECRET=your-jwt-secret-here
JWT_EXPIRES_IN=7d
JWT_REFRESH_EXPIRES_IN=30d

# Email (SendGrid)
SENDGRID_API_KEY=SG.xxxxxxxxxxxx
EMAIL_FROM=noreply@yourapp.com

# Payment
STRIPE_SECRET_KEY=sk_test_xxxxxxxxxxxx
STRIPE_WEBHOOK_SECRET=whsec_xxxxxxxxxxxx
VNPAY_TERMINAL_ID=your-terminal-id
VNPAY_HASH_SECRET=your-hash-secret

# Storage (AWS S3)
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key
AWS_REGION=ap-southeast-1
S3_BUCKET_NAME=your-bucket-name

# Logging
LOG_LEVEL=info
LOG_FORMAT=json
```

---

## 9. Quy Tắc KHÔNG Được Làm (Anti-patterns)

### Cấu trúc SAI - Type-based (Không nên dùng)

```
src/
├── controllers/            # Mỗi khi thêm feature, phải sửa nhiều thư mục
│   ├── UserController.ts
│   ├── OrderController.ts
│   ├── ProductController.ts
│   └── PaymentController.ts
│
├── services/               # File liên quan đến một feature bị rải khắp nơi
│   ├── UserService.ts
│   ├── OrderService.ts
│   ├── ProductService.ts
│   └── PaymentService.ts
│
├── repositories/
│   ├── UserRepository.ts
│   ├── OrderRepository.ts
│   ├── ProductRepository.ts
│   └── PaymentRepository.ts
│
├── models/
│   ├── User.ts
│   ├── Order.ts
│   ├── Product.ts
│   └── Payment.ts
│
├── dtos/
│   ├── CreateUserDto.ts
│   ├── CreateOrderDto.ts
│   ├── CreateProductDto.ts
│   └── CreatePaymentDto.ts
│
└── utils/
    └── helpers.ts          # Dump tất cả vào đây, file ngày càng to
```

**Vấn đề:** Khi làm việc với tính năng "Orders", bạn phải mở 5 thư mục khác nhau.

---

### Cấu trúc ĐÚNG - Feature-based (Nên dùng)

```
src/
├── features/
│   ├── orders/             # Tất cả code liên quan Orders đều ở đây
│   │   ├── domain/
│   │   │   ├── order.entity.ts
│   │   │   └── order.repository.interface.ts
│   │   ├── application/
│   │   │   └── create-order.use-case.ts
│   │   ├── infrastructure/
│   │   │   └── order.repository.ts
│   │   └── api/
│   │       └── orders.controller.ts
│   │
│   ├── users/              # Tất cả code liên quan Users đều ở đây
│   ├── products/           # Tất cả code liên quan Products đều ở đây
│   └── payments/           # Tất cả code liên quan Payments đều ở đây
│
└── shared/                 # Chỉ những gì THỰC SỰ dùng chung
    └── utils/
        └── date.utils.ts   # Tiện ích chung thực sự, không phải nơi dump code
```

**Lợi ích:** Mở thư mục `orders/`, mọi thứ liên quan đều ở đó.

---

### Bảng So Sánh

| Tiêu chí | Type-based (Sai) | Feature-based (Đúng) |
|---|---|---|
| Tìm code theo tính năng | Phải nhảy giữa nhiều thư mục | Mở 1 thư mục là đủ |
| Thêm tính năng mới | Thêm file vào 5-6 thư mục | Tạo 1 thư mục mới |
| Xóa tính năng | Phải tìm và xóa rải rác | Xóa 1 thư mục là xong |
| Làm việc nhóm | Conflict khi nhiều người cùng sửa | Ít conflict, ai sở hữu feature nấy |
| Tách thành microservice | Rất khó, code đan xen | Dễ, mỗi feature là 1 module |
| Onboarding | Khó hiểu cấu trúc | Dễ đọc, tự giải thích |
| Coupling | Cao - service A gọi service B thoải mái | Thấp hơn - rõ ràng ranh giới |

---

### Các Anti-pattern Phổ Biến Khác

```
# SAI: File helpers quá to
utils/
└── helpers.ts  # 1000 dòng, đủ loại hàm không liên quan

# ĐÚNG: Chia nhỏ theo mục đích
utils/
├── date.utils.ts
├── string.utils.ts
├── number.utils.ts
└── array.utils.ts

---

# SAI: Nested quá sâu (quá 4 cấp là vấn đề)
features/users/application/commands/register/handlers/v2/RegisterUserCommandHandlerV2.ts

# ĐÚNG: Giữ cấu trúc phẳng hợp lý
features/users/application/commands/register-user.handler.ts

---

# SAI: import cross-feature trực tiếp
# Trong orders/service.ts:
import { UserEntity } from '../users/domain/user.entity';  # SAI

# ĐÚNG: import qua public interface hoặc shared
import { IUserInfo } from '../../shared/types/user.types';  # ĐÚNG

---

# SAI: Business logic trong Controller
class OrdersController {
  createOrder(req, res) {
    const user = await User.findById(req.userId);  # SAI - query DB trong controller
    if (user.balance < req.total) throw error;     # SAI - business logic trong controller
    const order = new Order({ ...req.body, userId: user.id });
    await order.save();
    res.json(order);
  }
}

# ĐÚNG: Controller chỉ chuyển tiếp
class OrdersController {
  createOrder(req, res) {
    const result = await this.createOrderUseCase.execute(req.body);  # ĐÚNG
    res.json(result);
  }
}
```

---

## 10. Checklist Khi Tạo Project Mới

Checklist này áp dụng cho mọi project mới, bất kể ngôn ngữ hay framework.

### Giai đoạn 1: Khởi tạo cơ bản

1. Tạo repository trên GitHub/GitLab với tên rõ ràng theo định dạng `kebab-case`
2. Clone repository về máy local
3. Tạo file `.gitignore` phù hợp với ngôn ngữ (dùng gitignore.io)
4. Tạo file `README.md` với mô tả project và hướng dẫn cài đặt cơ bản
5. Tạo file `.editorconfig` để thống nhất formatting cho cả team
6. Khởi tạo project theo ngôn ngữ (`npm init`, `go mod init`, `poetry init`, v.v.)
7. Commit initial commit với message chuẩn: `chore: initial project setup`

### Giai đoạn 2: Cấu hình môi trường

8. Tạo file `.env.example` với TẤT CẢ biến môi trường cần thiết (có comment giải thích)
9. Tạo file `.env` từ `.env.example` (không commit file này)
10. Thêm `.env` vào `.gitignore`
11. Thiết lập `docker-compose.yml` với các services cần thiết (database, cache, v.v.)
12. Viết script `scripts/setup.sh` để developer mới có thể setup trong 1 lệnh
13. Tạo `Makefile` với các lệnh thường dùng: `make install`, `make dev`, `make test`

### Giai đoạn 3: Cấu trúc thư mục

14. Tạo cấu trúc thư mục theo feature-based (tham khảo Section 2)
15. Tạo thư mục `src/` hoặc tương đương là root của source code
16. Tạo thư mục `tests/` với các sub-folders: `unit/`, `integration/`, `e2e/`
17. Tạo thư mục `docs/` cho tài liệu kỹ thuật
18. Tạo thư mục `scripts/` cho các script tiện ích
19. Tạo thư mục `deployments/` cho Dockerfile và configs deployment

### Giai đoạn 4: Cấu hình CI/CD

20. Tạo `.github/workflows/ci.yml` chạy test và lint tự động khi push
21. Thiết lập lint check trong CI (fail nếu code không đúng style)
22. Thiết lập test coverage report trong CI
23. Cấu hình branch protection: require PR review và CI pass trước khi merge
24. Tạo PR template tại `.github/pull_request_template.md`
25. Tạo Issue templates cho bug report và feature request

### Giai đoạn 5: Chất lượng code

26. Cài đặt và cấu hình linter (ESLint, golangci-lint, pylint, v.v.)
27. Cài đặt và cấu hình formatter (Prettier, gofmt, black, v.v.)
28. Thiết lập pre-commit hooks (Husky cho Node.js, pre-commit cho Python)
29. Viết test đầu tiên (ít nhất 1 unit test) để xác nhận test setup hoạt động
30. Cấu hình test coverage threshold tối thiểu (thường 70-80%)

### Giai đoạn 6: Tài liệu

31. Viết `CHANGELOG.md` với template SemVer chuẩn
32. Tạo `docs/architecture/overview.md` mô tả kiến trúc tổng quan
33. Viết hướng dẫn onboarding tại `docs/onboarding.md`
34. Thiết lập Swagger/OpenAPI documentation cho API (nếu có)
35. Ghi chép các Architecture Decision Records (ADR) cho quyết định quan trọng

### Giai đoạn 7: Kiểm tra cuối

36. Chạy `make install` trên máy mới để xác nhận setup script hoạt động
37. Chạy `make test` để xác nhận tất cả tests pass
38. Chạy `make build` để xác nhận build production hoạt động
39. Đọc lại `README.md` từ góc nhìn developer mới - có đủ thông tin không?
40. Mời ít nhất 1 người khác trong team thử setup project từ đầu theo README

---

*Tài liệu này được cập nhật theo kinh nghiệm thực tế. Không có cấu trúc nào là hoàn hảo cho mọi trường hợp - hãy điều chỉnh cho phù hợp với quy mô và đặc thù của project.*
