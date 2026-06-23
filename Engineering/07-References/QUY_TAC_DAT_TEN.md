# QUY TAC DAT TEN

> **Pham vi ap dung**: Moi du an backend bat ke ngon ngu lap trinh  
> **Phien ban**: 1.0  
> **Cap nhat lan cuoi**: 2026-06-23  
> **Muc dich**: Tai lieu tham chieu chu for moi identifier trong he thong

---

## Muc Luc

1. [Quy Uoc Chung (Ap Dung Moi Ngon Ngu)](#1-quy-uoc-chung)
2. [Dat Ten Theo Ngon Ngu Lap Trinh](#2-dat-ten-theo-ngon-ngu-lap-trinh)
3. [Dat Ten Theo Loai Thanh Phan](#3-dat-ten-theo-loai-thanh-phan)
4. [Dat Ten Database](#4-dat-ten-database)
5. [Dat Ten API](#5-dat-ten-api)
6. [Dat Ten Thu Muc va File](#6-dat-ten-thu-muc-va-file)
7. [Dat Ten Bien](#7-dat-ten-bien)
8. [Dat Ten Event va Queue](#8-dat-ten-event-va-queue)
9. [Dat Ten Trong DevOps / Infrastructure](#9-dat-ten-trong-devops--infrastructure)
10. [Cache va Redis Keys](#10-cache-va-redis-keys)
11. [Logging Fields](#11-logging-fields)
12. [Metrics](#12-metrics)
13. [Branch va Commit](#13-branch-va-commit)
14. [Bang Anti-Pattern Dat Ten](#14-bang-anti-pattern-dat-ten)

---

## 1. Quy Uoc Chung (Ap Dung Moi Ngon Ngu)

> Truoc khi doc tung phan chi tiet, hay nam vung bang tong hop nay. Day la "ban do" tham chieu nhanh cho moi loai identifier.

### Bang Tong Hop Convention

| Loai Identifier | Convention | Vi du Dung | Vi du Sai |
|---|---|---|---|
| Class / Struct | PascalCase | `OrderService`, `UserProfile` | `orderService`, `user_profile` |
| Interface (C#) | `I` + PascalCase | `IOrderRepository`, `IUserService` | `OrderRepository`, `userRepository` |
| Interface (Java/TS/Go) | PascalCase | `OrderRepository`, `Reader` | `IOrderRepository`, `order_repository` |
| Method (C#/.NET) | PascalCase | `GetOrderById`, `CreateUser` | `getOrderById`, `get_order_by_id` |
| Method (Java/JS/TS/PHP) | camelCase | `getOrderById`, `createUser` | `GetOrderById`, `get_order_by_id` |
| Method (Python) | snake_case | `get_order_by_id`, `create_user` | `getOrderById`, `GetOrderById` |
| Method (Go - public) | PascalCase | `GetOrderById`, `CreateUser` | `getOrderById`, `get_order_by_id` |
| Method (Go - private) | camelCase | `validateOrder`, `buildQuery` | `ValidateOrder`, `validate_order` |
| Property (C#) | PascalCase | `OrderId`, `CreatedAt` | `orderId`, `order_id` |
| Property (Java/TS) | camelCase | `orderId`, `createdAt` | `OrderId`, `order_id` |
| Variable (Java/C#/TS) | camelCase | `currentUser`, `orderList` | `CurrentUser`, `current_user` |
| Variable (Python/Go private) | snake_case | `current_user`, `order_list` | `currentUser`, `CurrentUser` |
| Field private (C#) | `_` + camelCase | `_orderRepository`, `_logger` | `orderRepository`, `m_orderRepository` |
| Constant (C#) | PascalCase hoac UPPER_SNAKE | `MaxRetryCount`, `MAX_RETRY_COUNT` | `maxretrycount`, `max-retry-count` |
| Constant (Java/Python/Go) | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_PAGE` | `maxRetryCount`, `MaxRetryCount` |
| Constant (TypeScript module) | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `BASE_URL` | `maxRetryCount`, `baseUrl` |
| Enum (name) | PascalCase | `OrderStatus`, `PaymentMethod` | `order_status`, `ORDERSTATUS` |
| Enum (values C#/TS) | PascalCase | `OrderStatus.Pending`, `PaymentMethod.CreditCard` | `ORDER_STATUS.PENDING` |
| Enum (values Java/Python) | UPPER_SNAKE_CASE | `OrderStatus.PENDING`, `PaymentMethod.CREDIT_CARD` | `OrderStatus.Pending` |
| Generic type param | T, TResult, TKey, TValue | `Task<TResult>`, `Dictionary<TKey, TValue>` | `Type1`, `type`, `genericType` |
| Namespace / Package (C#) | PascalCase dot-separated | `Company.Product.Module` | `company.product.module` |
| Package (Java) | all lowercase dot-separated | `com.company.product.module` | `com.Company.Product` |
| Package (Go) | lowercase no underscore | `orderservice`, `user` | `order_service`, `OrderService` |
| Module / File (Python) | snake_case | `order_service.py`, `user_model.py` | `OrderService.py`, `orderService.py` |
| File (C#/Java) | PascalCase | `OrderService.cs`, `UserController.java` | `order_service.cs` |
| File (Go) | snake_case | `order_service.go`, `user_handler.go` | `orderService.go`, `OrderService.go` |
| File (TypeScript) | kebab-case | `order-service.ts`, `user-controller.ts` | `OrderService.ts`, `orderService.ts` |
| Folder / Directory | kebab-case | `user-management`, `order-processing` | `UserManagement`, `user_management` |
| Database Table | snake_case PLURAL | `users`, `order_items`, `product_categories` | `User`, `OrderItems`, `tbl_users` |
| Database Column | snake_case | `user_id`, `created_at`, `is_active` | `userId`, `CreatedAt`, `IsActive` |
| Primary Key | `id` | `id` | `user_id` (tren chinh bang users), `userId` |
| Foreign Key | `[table_singular]_id` | `user_id`, `order_id`, `category_id` | `userId`, `userID`, `fk_user` |
| Index | `idx_[table]_[cols]` | `idx_users_email`, `idx_orders_user_id` | `users_email_idx`, `index_users_email` |
| Constraint PK | `pk_[table]` | `pk_users`, `pk_orders` | `users_pk`, `primary_users` |
| Constraint FK | `fk_[table]_[ref]` | `fk_orders_users`, `fk_items_products` | `orders_users_fk` |
| Constraint Unique | `uq_[table]_[cols]` | `uq_users_email`, `uq_products_sku` | `users_email_unique` |
| API Endpoint URL | kebab-case PLURAL noun | `/users`, `/order-items`, `/product-categories` | `/User`, `/getUsers`, `/orderItems` |
| Path Parameter | camelCase | `/users/{userId}`, `/orders/{orderId}` | `/users/{user_id}`, `/users/{UserID}` |
| Query Parameter | camelCase | `?pageSize=20&sortBy=name` | `?page_size=20&sort_by=name` |
| HTTP Custom Header | `X-[Purpose]` Train-Case | `X-Request-Id`, `X-Correlation-Id` | `x-request-id`, `XRequestId` |
| Environment Variable | UPPER_SNAKE_CASE | `DB_HOST`, `JWT_SECRET_KEY` | `dbHost`, `Db-Host` |
| Docker Image | lowercase kebab-case | `company/user-service:1.2.3` | `company/UserService:1.2.3` |
| Kubernetes Resource | kebab-case + resource suffix | `user-service-deployment` | `userServiceDeployment`, `user_service_dep` |
| Cache Key | lowercase colon-separated | `cache:v1:user:12345` | `Cache:V1:User:12345`, `cache_v1_user_12345` |
| Domain Event | PascalCase + `Event` | `OrderPlacedEvent`, `PaymentFailedEvent` | `OrderPlaced`, `order_placed_event` |
| Queue / Topic | lowercase kebab dot-separated | `order-service.high.payment-processing` | `OrderService.High.PaymentProcessing` |
| Branch Git | `[type]/[ticket]-[desc]` | `feature/PROJ-123-user-profile` | `feature_user_profile`, `UserProfileFeature` |
| Metric | `[ns]_[component]_[measure]_[unit]` | `order_service_http_requests_total` | `orderServiceHttpRequests`, `http_requests` |
| Log Field | camelCase | `traceId`, `correlationId`, `durationMs` | `trace_id`, `TraceId`, `traceid` |
| Async method suffix (C#) | `Async` | `GetUserAsync`, `CreateOrderAsync` | `GetUserAsynchronous`, `AsyncGetUser` |
| Controller | PascalCase + `Controller` | `OrderController`, `UserController` | `OrderCtrl`, `order_controller` |
| Service | PascalCase + `Service` | `OrderService`, `PaymentService` | `OrderSvc`, `order_service_class` |
| Repository | PascalCase + `Repository` | `OrderRepository`, `UserRepository` | `OrderRepo`, `OrderDAO` (tru Java co the dung DAO) |
| Async method suffix (C#) | `Async` | `GetUserAsync`, `CreateOrderAsync` | `GetUser`, `AsyncGetUser` |

---

### Cac Kieu Casing

#### PascalCase (UpperCamelCase)

Moi tu bat dau bang chu hoa. Khong co ki tu phan cach.

```
# Quy tac:
# - Moi tu: chu cai dau la HOA
# - Khong co dau gach ngang, dau gach duoi, khoang trang

# Vi du dung:
OrderService
UserProfileController
ProductCategoryRepository
CreateOrderCommandHandler
OrderPlacedEvent
HttpRequestException
GetUserByEmailAsync

# Vi du sai:
orderService          # chu cai dau viet thuong -> camelCase
Order_Service         # co dau gach duoi
ORDER_SERVICE         # tat ca chu hoa -> UPPER_SNAKE_CASE
orderservice          # tat ca chu thuong -> lowercase
```

**Su dung**: Class, Interface (Java/TS/Go), Method (C#/.NET), Property (C#), Enum name, Enum values (C#/TS), Namespace (C#), Generic type param.

---

#### camelCase (lowerCamelCase)

Tu dau viet thuong toan bo. Cac tu tiep theo bat dau bang chu hoa. Khong co ki tu phan cach.

```
# Quy tac:
# - Tu dau tien: TAT CA chu thuong
# - Tu thu 2 tro di: chu cai dau la HOA

# Vi du dung:
getUserById
createOrder
orderService
currentUserId
pageSize
isActive
hasPermission
totalOrderCount

# Vi du sai:
GetUserById          # tu dau viet hoa -> PascalCase
get_user_by_id       # co dau gach duoi -> snake_case
getuserbyid          # tat ca thuong, khong phan cach -> lowercase
```

**Su dung**: Method (Java/JS/TS/PHP), Variable (Java/C#/TS/PHP), Property (Java/TS), Field private (C# sau prefix `_`), Query param, Log field, Path param template.

---

#### snake_case

Tat ca chu thuong. Cac tu cach nhau boi dau gach duoi `_`.

```
# Quy tac:
# - TAT CA chu thuong
# - Dau gach duoi _ ngan cach cac tu

# Vi du dung:
get_order_by_id
create_user
order_service
current_user_id
is_active
has_permission
total_order_count
user_profile_picture_url

# Vi du sai:
Get_Order_By_Id      # co chu hoa -> Train_Case (khong pho bien)
getOrderById         # khong co dau gach duoi -> camelCase
get-order-by-id      # dau gach ngang -> kebab-case
GETORDERBYID         # tat ca hoa -> UPPER_SNAKE
```

**Su dung**: Function/Method (Python/Go private), Variable (Python), File (Python/Go), Module (Python), Database table, Database column, Migration file description.

---

#### UPPER_SNAKE_CASE (SCREAMING_SNAKE_CASE)

Tat ca chu hoa. Cac tu cach nhau boi dau gach duoi `_`.

```
# Quy tac:
# - TAT CA chu HOA
# - Dau gach duoi _ ngan cach cac tu

# Vi du dung:
MAX_RETRY_COUNT
DEFAULT_PAGE_SIZE
JWT_EXPIRY_SECONDS
HTTP_TIMEOUT_MS
BASE_URL
API_VERSION
DATABASE_CONNECTION_POOL_SIZE
FEATURE_FLAG_NEW_CHECKOUT

# Vi du sai:
maxRetryCount        # camelCase
max_retry_count      # snake_case (chu thuong)
MaxRetryCount        # PascalCase
Max-Retry-Count      # Train-Case
```

**Su dung**: Constant (Java/Python/Go/JS-module), Enum values (Java/Python), Environment variable, Configuration key (env).

---

#### kebab-case (param-case)

Tat ca chu thuong. Cac tu cach nhau boi dau gach ngang `-`.

```
# Quy tac:
# - TAT CA chu thuong
# - Dau gach ngang - ngan cach cac tu

# Vi du dung:
user-management
order-processing
background-jobs
api-gateway
order-service
product-category
order-items

# URL resources:
/users
/order-items
/product-categories
/user-addresses

# Vi du sai:
UserManagement       # PascalCase
user_management      # snake_case
userManagement       # camelCase
User-Management      # Train-Case
```

**Su dung**: Folder/Directory (moi ngon ngu), File (TypeScript), URL resource path, Docker image name, Kubernetes resource name, CI/CD pipeline job name, Queue/topic name.

---

#### Train-Case (HTTP-Header-Case)

Moi tu bat dau bang chu hoa. Cac tu cach nhau boi dau gach ngang `-`.

```
# Quy tac:
# - Moi tu: chu cai dau la HOA
# - Dau gach ngang - ngan cach cac tu

# Vi du dung:
X-Request-Id
X-Correlation-Id
X-Idempotency-Key
X-Api-Version
X-User-Id
Content-Type
Authorization
Accept-Language

# Vi du sai:
x-request-id         # tat ca thuong -> kebab-case
X_REQUEST_ID         # dau gach duoi -> UPPER_SNAKE
xRequestId           # camelCase
XRequestId           # PascalCase (khong co dau phan cach)
```

**Su dung**: HTTP Header (ca standard va custom).

---

## 2. Dat Ten Theo Ngon Ngu Lap Trinh

### C# / .NET

C# co nhung quy tac dat ten rieng bien biet so voi Java hay JavaScript. Dac biet: method dung PascalCase (khac Java dung camelCase).

#### Namespace

```csharp
// Quy tac: CompanyName.ProductName.Feature.SubFeature
// Tat ca PascalCase, phan cach bang dau cham

// Dung:
namespace AcmeCorp.ECommerce.Orders.Domain
namespace AcmeCorp.ECommerce.Users.Application.Commands
namespace AcmeCorp.ECommerce.Shared.Infrastructure.Persistence

// Sai:
namespace acmecorp.ecommerce.orders           // tat ca thuong
namespace AcmeCorp.ECommerce.orders           // khong nhat quan
namespace Acme_Corp.E_Commerce.Orders         // co dau gach duoi
```

#### Class

```csharp
// Quy tac: PascalCase, danh tu so it, mo ta ro rang
public class OrderService { }
public class UserProfileController { }
public class ProductCategoryRepository { }
public class CreateOrderCommandHandler { }
public class OrderNotFoundException : Exception { }
public class ApplicationDbContext : DbContext { }
public class JwtTokenValidator { }

// Sai:
public class orderService { }           // camelCase
public class Order_Service { }          // co dau gach duoi
public class ordersrv { }              // viet tat khong ro nghia
```

#### Interface

```csharp
// Quy tac: "I" + PascalCase (bat buoc trong C#)
public interface IOrderRepository { }
public interface IUserService { }
public interface IEmailSender { }
public interface IQueryHandler<TQuery, TResult> { }
public interface ICommandHandler<TCommand> { }
public interface ICacheProvider { }
public interface ICurrentUserService { }

// Sai:
public interface OrderRepository { }    // thieu prefix I
public interface iOrderRepository { }   // I chu thuong
public interface I_OrderRepository { }  // co dau gach duoi
```

#### Method

```csharp
// Quy tac: PascalCase (KHAC Java, Python, Go private)
// Async method: them suffix "Async"

// Dung:
public Task<Order> GetOrderByIdAsync(Guid orderId)
public Task<IEnumerable<Order>> GetOrdersByUserIdAsync(Guid userId)
public Task CreateOrderAsync(CreateOrderCommand command)
public Task<bool> ValidatePaymentAsync(PaymentRequest request)
public void CancelOrder(Guid orderId)
private void ValidateOrder(Order order)
private static string BuildConnectionString(DbConfig config)

// Async suffix:
public Task<User> GetUserAsync(Guid userId)        // Dung
public Task<User> GetUser(Guid userId)             // Sai - thieu Async
public Task<User> GetUserAsynchronous(Guid userId) // Sai - qua dai

// Sai:
public Task<Order> getOrderByIdAsync(Guid id)    // camelCase - Java style
public Task<Order> get_order_by_id(Guid id)      // snake_case
public Task<Order> GetOrder(Guid id)             // "id" qua ngan, thieu context
```

#### Property

```csharp
// Quy tac: PascalCase
public class Order
{
    public Guid Id { get; private set; }
    public Guid UserId { get; private set; }
    public string OrderNumber { get; private set; }
    public decimal TotalAmount { get; private set; }
    public OrderStatus Status { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public DateTime? UpdatedAt { get; private set; }
    public bool IsDeleted { get; private set; }
    public IReadOnlyList<OrderItem> Items { get; private set; }
}

// Sai:
public Guid orderId { get; set; }         // camelCase
public Guid order_id { get; set; }        // snake_case
public string orderNumber { get; set; }   // camelCase
```

#### Field Private

```csharp
// Quy tac: prefix "_" + camelCase
// Khong dung "m_", "this.", hay khong co prefix

public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ILogger<OrderService> _logger;
    private readonly ICurrentUserService _currentUserService;
    private readonly ICacheProvider _cacheProvider;
    private static readonly TimeSpan _defaultCacheDuration = TimeSpan.FromMinutes(5);

    // Sai:
    private readonly IOrderRepository orderRepository;   // khong co prefix _
    private readonly IOrderRepository m_orderRepository; // m_ prefix (C++ style)
    private readonly IOrderRepository _OrderRepository;  // _ + PascalCase
}
```

#### Constant

```csharp
// Quy tac: PascalCase (Microsoft guidelines) hoac UPPER_SNAKE_CASE (nhieu team chon)
// Quan trong: nhat quan trong toan project, chon mot va giu nguyen

// Option 1: PascalCase (Microsoft official):
public const int MaxRetryCount = 3;
public const string DefaultConnectionString = "...";
public const int DefaultPageSize = 20;

// Option 2: UPPER_SNAKE_CASE (pho bien trong nhieu team):
public const int MAX_RETRY_COUNT = 3;
public const string DEFAULT_CONNECTION_STRING = "...";
public const int DEFAULT_PAGE_SIZE = 20;

// readonly static (PascalCase theo Microsoft):
public static readonly TimeSpan DefaultTimeout = TimeSpan.FromSeconds(30);
public static readonly string[] AllowedOrigins = { "https://app.company.com" };

// Sai:
public const int maxRetryCount = 3;         // camelCase
public const int max_retry_count = 3;       // snake_case
public const int MAXRETRYCOUNT = 3;         // khong co dau phan cach
```

#### Generic Type Parameter

```csharp
// Quy tac: T, TResult, TKey, TValue, TEntity, TCommand, TQuery
// Mot tham so: T
// Nhieu tham so: T + mo ta nghia

// Dung:
public interface IRepository<TEntity> { }
public interface IQueryHandler<TQuery, TResult> { }
public class Result<T> { }
public class PagedList<T> { }
public interface IMapper<TSource, TDestination> { }
public class Cache<TKey, TValue> { }
public interface ICommandHandler<TCommand, TResult> { }

// Sai:
public interface IRepository<Entity> { }      // khong co T prefix
public interface IRepository<Type1> { }       // khong mo ta nghia
public interface IRepository<GenericType> { } // qua dai, khong theo convention
```

#### Enum

```csharp
// Quy tac: PascalCase cho ca ten enum va cac gia tri
public enum OrderStatus
{
    Pending,
    Confirmed,
    Processing,
    Shipped,
    Delivered,
    Cancelled,
    Refunded
}

public enum PaymentMethod
{
    CreditCard,
    DebitCard,
    BankTransfer,
    DigitalWallet,
    CashOnDelivery
}

// [Flags] enum: ten nen la plural
[Flags]
public enum UserPermissions
{
    None = 0,
    ReadProducts = 1,
    WriteProducts = 2,
    DeleteProducts = 4,
    ManageUsers = 8,
    All = ReadProducts | WriteProducts | DeleteProducts | ManageUsers
}

// Sai:
public enum order_status { pending, confirmed }          // snake_case
public enum OrderStatus { PENDING, CONFIRMED }           // UPPER_SNAKE cho values
public enum EOrderStatus { Pending, Confirmed }          // prefix E (C++ style)
```

---

### Java / Spring Boot

#### Package

```java
// Quy tac: tat ca chu thuong, dot-separated, reverse domain + product + module
// Khong viet hoa, khong dau gach duoi, khong dau gach ngang

// Dung:
package com.acmecorp.ecommerce.order.domain;
package com.acmecorp.ecommerce.order.application.command;
package com.acmecorp.ecommerce.order.application.query;
package com.acmecorp.ecommerce.order.infrastructure.persistence;
package com.acmecorp.ecommerce.shared.event;

// Sai:
package com.AcmeCorp.ECommerce.Order.Domain;   // PascalCase
package com.acmecorp.ecommerce.Order.domain;   // khong nhat quan
package com.acme_corp.ecommerce.order;          // co dau gach duoi
```

#### Class

```java
// Quy tac: PascalCase, danh tu so it
@Service
public class OrderService { }

@RestController
@RequestMapping("/orders")
public class OrderController { }

@Repository
public class JpaOrderRepository implements OrderRepository { }

@Component
public class OrderEventPublisher { }

public class OrderNotFoundException extends RuntimeException { }

// Sai:
public class orderService { }           // camelCase
public class Order_Service { }          // co dau gach duoi
```

#### Interface

```java
// Quy tac: PascalCase, KHONG co prefix "I" (khac C#)
// Ten interface nen la danh tu hoac danh tu cum + tac vu

public interface OrderRepository { }
public interface UserService { }
public interface EmailSender { }
public interface PaymentGateway { }
public interface CacheProvider { }
public interface TokenValidator { }
public interface Auditable { }          // interface "kha nang"

// Sai:
public interface IOrderRepository { }  // prefix I (C# style, khong dung trong Java)
public interface OrderRepositoryInterface { } // suffix Interface - verbose
```

#### Method

```java
// Quy tac: camelCase, dong tu + bo sung
// Spring Data JPA naming convention rat quan trong

public Order getOrderById(UUID orderId)
public List<Order> getOrdersByUserId(UUID userId)
public Order createOrder(CreateOrderRequest request)
public void cancelOrder(UUID orderId)
public boolean isOrderEligibleForRefund(UUID orderId)
public int countActiveOrdersByUser(UUID userId)

// Spring Data JPA repository methods:
Optional<User> findByEmail(String email);
List<Order> findByUserIdAndStatus(UUID userId, OrderStatus status);
List<Order> findAllByCreatedAtBetween(LocalDateTime from, LocalDateTime to);
boolean existsByEmail(String email);
long countByStatus(OrderStatus status);

// Sai:
public Order GetOrderById(UUID id)       // PascalCase (C# style)
public Order get_order_by_id(UUID id)    // snake_case (Python style)
public Order FetchOrder(UUID id)         // PascalCase
```

#### Variable va Field

```java
// Quy tac: camelCase
// Instance field: camelCase (khong co prefix)

public class OrderService {
    private final OrderRepository orderRepository;
    private final UserService userService;
    private final ApplicationEventPublisher eventPublisher;
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);

    // Local variable:
    public Order processOrder(UUID orderId) {
        Order currentOrder = orderRepository.findById(orderId).orElseThrow();
        User orderOwner = userService.findById(currentOrder.getUserId());
        List<OrderItem> validItems = currentOrder.getItems().stream()
            .filter(item -> !item.isOutOfStock())
            .collect(Collectors.toList());
        int totalItemCount = validItems.size();
        return currentOrder;
    }
}

// Sai:
private final OrderRepository OrderRepository;   // PascalCase
private final OrderRepository _orderRepository;  // _ prefix (C# style)
private final OrderRepository m_orderRepository; // m_ prefix
```

#### Constant

```java
// Quy tac: UPPER_SNAKE_CASE, static final
public class OrderConstants {
    public static final int MAX_ITEMS_PER_ORDER = 100;
    public static final int DEFAULT_PAGE_SIZE = 20;
    public static final long JWT_EXPIRY_SECONDS = 900L;
    public static final String DEFAULT_CURRENCY = "USD";
    public static final BigDecimal FREE_SHIPPING_THRESHOLD = new BigDecimal("50.00");
}

// Trong class:
public class OrderService {
    private static final int MAX_RETRY_COUNT = 3;
    private static final Duration LOCK_TIMEOUT = Duration.ofSeconds(10);
}

// Sai:
public static final int maxRetryCount = 3;         // camelCase
public static final int Max_Retry_Count = 3;       // khong nhat quan
```

#### Enum

```java
// Quy tac: PascalCase cho ten, UPPER_SNAKE_CASE cho gia tri
public enum OrderStatus {
    PENDING,
    CONFIRMED,
    PROCESSING,
    SHIPPED,
    DELIVERED,
    CANCELLED,
    REFUNDED;

    public boolean isFinal() {
        return this == DELIVERED || this == CANCELLED || this == REFUNDED;
    }
}

public enum PaymentMethod {
    CREDIT_CARD,
    DEBIT_CARD,
    BANK_TRANSFER,
    DIGITAL_WALLET,
    CASH_ON_DELIVERY
}

// Sai:
public enum OrderStatus { Pending, Confirmed }     // PascalCase values (C# style)
public enum order_status { pending, confirmed }    // snake_case
```

---

### Python

#### Module va File

```python
# Quy tac: snake_case, .py extension
# Ten file = ten module

# Dung:
order_service.py
user_controller.py
product_repository.py
create_order_handler.py
order_domain_event.py
database_config.py
jwt_token_validator.py

# Sai:
OrderService.py          # PascalCase
orderService.py          # camelCase
order-service.py         # kebab-case (loi import)
```

#### Class

```python
# Quy tac: PascalCase (giong moi ngon ngu khac)
class OrderService:
    pass

class UserProfileController:
    pass

class ProductCategoryRepository:
    pass

class OrderNotFoundException(Exception):
    pass

class CreateOrderCommandHandler:
    pass

# Sai:
class orderService:         # camelCase
class order_service:        # snake_case
```

#### Function va Method

```python
# Quy tac: snake_case
def get_order_by_id(order_id: str) -> Order:
    pass

def create_order(command: CreateOrderCommand) -> Order:
    pass

def is_order_eligible_for_refund(order: Order) -> bool:
    pass

def count_active_orders_by_user(user_id: str) -> int:
    pass

# Instance method:
class OrderService:
    def get_order_by_id(self, order_id: str) -> Order:
        pass

    def _validate_order(self, order: Order) -> None:
        """Private method - single underscore"""
        pass

    def __hash_payment_token(self, token: str) -> str:
        """Very private - name mangling, double underscore"""
        pass

# Sai:
def GetOrderById(order_id):    # PascalCase (C# style)
def getOrderById(order_id):    # camelCase (Java style)
```

#### Variable

```python
# Quy tac: snake_case
current_user_id = "..."
order_list = []
total_item_count = 0
is_active = True
has_permission = False
page_size = 20

# Loop variables:
for order in orders:
    for item in order.items:
        pass

# Sai:
currentUserId = "..."      # camelCase
CurrentUserId = "..."      # PascalCase
```

#### Constant

```python
# Quy tac: UPPER_SNAKE_CASE, module-level
MAX_RETRY_COUNT = 3
DEFAULT_PAGE_SIZE = 20
JWT_EXPIRY_SECONDS = 900
BASE_API_URL = "https://api.company.com"
ALLOWED_EXTENSIONS = {".jpg", ".png", ".pdf"}
HTTP_TIMEOUT_SECONDS = 30

# Sai:
maxRetryCount = 3          # camelCase - trong nhu variable
max_retry_count = 3        # snake_case - trong nhu variable
MaxRetryCount = 3          # PascalCase - trong nhu class
```

#### Private va Name Mangling

```python
class OrderService:
    # Public:
    order_count: int = 0

    # Protected (convention - khong enforced):
    _internal_cache: dict = {}

    # Private (name mangling - enforced by Python):
    __secret_key: str = ""

    def process_order(self, order: Order) -> None:
        """Public method"""
        self._validate_order(order)
        self.__encrypt_payment_data(order.payment)

    def _validate_order(self, order: Order) -> None:
        """Protected - dung duoc tu subclass, khong nen goi tu ngoai"""
        pass

    def __encrypt_payment_data(self, payment) -> None:
        """Private - bi name mangling thanh _OrderService__encrypt_payment_data"""
        pass
```

---

### Go

#### Package

```go
// Quy tac: tat ca chu thuong, ngan, khong dau gach duoi, khong dau gach ngang
// Ten package nen la danh tu so it

package order
package user
package payment
package notification
package config
package middleware
package handler
package repository

// Sai:
package orderService    // camelCase
package order_service   // co dau gach duoi
package OrderService    // PascalCase
package orders          // plural (nen tranh, Go prefer singular)
```

#### Exported (Public) Identifier

```go
// Quy tac: PascalCase - bat ki identifier nao bat dau bang chu HOA deu la exported

// Struct:
type Order struct {
    ID        uuid.UUID
    UserID    uuid.UUID
    Status    OrderStatus
    TotalAmount decimal.Decimal
    CreatedAt time.Time
}

// Function / Method:
func GetOrderByID(ctx context.Context, id uuid.UUID) (*Order, error) {}
func CreateOrder(ctx context.Context, cmd CreateOrderCommand) (*Order, error) {}

// Interface:
type OrderRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*Order, error)
    Save(ctx context.Context, order *Order) error
    Delete(ctx context.Context, id uuid.UUID) error
}

// Constant:
const MaxRetryCount = 3
const DefaultPageSize = 20
```

#### Unexported (Private) Identifier

```go
// Quy tac: camelCase - bat dau bang chu thuong

type orderValidator struct {
    minAmount decimal.Decimal
    maxItems  int
}

func validateOrder(order *Order) error {}
func buildQueryFilter(params FilterParams) string {}

// Method private:
func (s *OrderService) validateOrder(order *Order) error {}
func (s *OrderService) buildCacheKey(id uuid.UUID) string {}
```

#### Interface

```go
// Quy tac: PascalCase, thong thung co suffix "-er" neu mo ta hanh dong
// Interface don phuong thuc: ten = ten method + "er"

type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}

type ReadWriter interface {
    Reader
    Writer
}

// Application interface - mo ta chuc nang:
type OrderRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*Order, error)
    Save(ctx context.Context, order *Order) error
}

type OrderService interface {
    CreateOrder(ctx context.Context, cmd CreateOrderCommand) (*Order, error)
    GetOrder(ctx context.Context, id uuid.UUID) (*Order, error)
}

type EmailSender interface {
    Send(ctx context.Context, email Email) error
}
```

#### Error Types

```go
// Quy tac: PascalCase + "Error" suffix
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation error on field %s: %s", e.Field, e.Message)
}

type NotFoundError struct {
    Resource string
    ID       string
}

type UnauthorizedError struct {
    UserID  string
    Action  string
    Resource string
}

// Sentinel errors: Err + PascalCase
var (
    ErrOrderNotFound    = errors.New("order not found")
    ErrInvalidOrderStatus = errors.New("invalid order status")
    ErrPaymentFailed    = errors.New("payment processing failed")
    ErrInsufficientStock = errors.New("insufficient stock")
)
```

#### Constant va Variable trong Go

```go
// Constant: UPPER_SNAKE_CASE
const (
    MaxRetryCount      = 3
    DefaultPageSize    = 20
    JWTExpirySeconds   = 900
    MaxItemsPerOrder   = 100
)

// Hoac nhom thanh iota:
type OrderStatus int
const (
    OrderStatusPending OrderStatus = iota
    OrderStatusConfirmed
    OrderStatusProcessing
    OrderStatusShipped
    OrderStatusDelivered
    OrderStatusCancelled
)

// Variable ngan trong short declaration:
func processOrder(id uuid.UUID) {
    order, err := repository.FindByID(ctx, id)   // "err" la convention Go
    if err != nil {
        return err
    }
    total := calculateTotal(order.Items)
    ok := validatePayment(order.Payment)
}
```

---

### TypeScript / NodeJS

#### File

```typescript
// Quy tac: kebab-case.ts (recommended) hoac camelCase.ts
// Chon MOT cho toan project, khong pha tron

// Recommended (kebab-case):
order-service.ts
user-controller.ts
product-repository.ts
create-order.handler.ts
order-placed.event.ts
database.config.ts
auth.middleware.ts

// Hoac camelCase (mot so project chon style nay):
orderService.ts
userController.ts
productRepository.ts

// Test files:
order-service.test.ts
order-service.spec.ts
```

#### Class

```typescript
// Quy tac: PascalCase
class OrderService {
    constructor(
        private readonly orderRepository: OrderRepository,
        private readonly logger: Logger
    ) {}
}

class UserProfileController {}
class ProductCategoryRepository {}
class CreateOrderCommandHandler {}
class OrderNotFoundException extends Error {}
class JwtTokenValidator {}
```

#### Interface va Type

```typescript
// Quy tac: PascalCase, KHONG co prefix "I" (TS hien dai)
// Interface cho object shape:
interface Order {
    id: string;
    userId: string;
    status: OrderStatus;
    totalAmount: number;
    createdAt: Date;
}

interface OrderRepository {
    findById(id: string): Promise<Order | null>;
    save(order: Order): Promise<void>;
}

// Type alias:
type OrderId = string;
type UserId = string;
type CreateOrderRequest = {
    userId: string;
    items: OrderItemRequest[];
};

// Utility types:
type PartialOrder = Partial<Order>;
type ReadonlyOrder = Readonly<Order>;
type OrderWithUser = Order & { user: User };

// Sai:
interface IOrder {}              // prefix I (C# style, deprecated trong TS)
interface IOrderRepository {}    // prefix I
type orderRequest = {}           // camelCase
```

#### Enum

```typescript
// Quy tac: PascalCase cho ca ten enum va cac gia tri
enum OrderStatus {
    Pending = 'PENDING',
    Confirmed = 'CONFIRMED',
    Processing = 'PROCESSING',
    Shipped = 'SHIPPED',
    Delivered = 'DELIVERED',
    Cancelled = 'CANCELLED'
}

enum PaymentMethod {
    CreditCard = 'CREDIT_CARD',
    DebitCard = 'DEBIT_CARD',
    BankTransfer = 'BANK_TRANSFER',
    DigitalWallet = 'DIGITAL_WALLET'
}

// Hoac string union (pho bien hon trong TS hien dai):
type OrderStatus = 'PENDING' | 'CONFIRMED' | 'PROCESSING' | 'SHIPPED' | 'DELIVERED' | 'CANCELLED';
```

#### Function va Variable

```typescript
// Function: camelCase
function getOrderById(orderId: string): Promise<Order> {}
function createOrder(request: CreateOrderRequest): Promise<Order> {}
const validateOrder = (order: Order): boolean => {};
const buildCacheKey = (id: string): string => `cache:v1:order:${id}`;

// Variable: camelCase
const currentUser: User = await userService.findById(userId);
const orderList: Order[] = await orderRepository.findAll();
let totalItemCount = 0;
let isOrderValid = false;

// Constant module-level: UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const DEFAULT_PAGE_SIZE = 20;
const JWT_EXPIRY_SECONDS = 900;
const BASE_URL = process.env.BASE_URL ?? 'http://localhost:3000';

// Constant local: camelCase hoac UPPER_SNAKE_CASE (nhat quan trong team)
const maxRetries = 3;           // local, camelCase ok
const MAX_RETRIES = 3;          // local, UPPER_SNAKE_CASE cung ok
```

---

### PHP

#### Class

```php
<?php
// Quy tac: PascalCase
class OrderService {}
class UserProfileController {}
class ProductCategoryRepository {}
class CreateOrderCommandHandler {}
class OrderNotFoundException extends \RuntimeException {}
```

#### Interface

```php
<?php
// Quy tac: PascalCase + "Interface" suffix (PSR convention)
// Hoac: "I" + PascalCase (mot so team chon)
interface OrderRepositoryInterface {}
interface UserServiceInterface {}
interface EmailSenderInterface {}

// Hoac (mot so team):
interface IOrderRepository {}
interface IUserService {}
```

#### Method va Variable

```php
<?php
class OrderService
{
    private OrderRepository $orderRepository;
    private LoggerInterface $logger;

    // Method: camelCase
    public function getOrderById(string $orderId): Order {}
    public function createOrder(CreateOrderRequest $request): Order {}
    private function validateOrder(Order $order): void {}

    // Variable: camelCase
    public function processOrders(): void
    {
        $currentUser = $this->getCurrentUser();
        $orderList = $this->orderRepository->findByUserId($currentUser->getId());
        $totalOrderCount = count($orderList);
        $isProcessingEnabled = $this->featureFlag->isEnabled('order_processing');
    }
}
```

#### Constant

```php
<?php
// Quy tac: UPPER_SNAKE_CASE
class OrderConstants
{
    public const MAX_ITEMS_PER_ORDER = 100;
    public const DEFAULT_PAGE_SIZE = 20;
    public const FREE_SHIPPING_THRESHOLD = 50.00;
    public const PAYMENT_TIMEOUT_SECONDS = 300;
}

// Interface constant:
interface PaymentGatewayInterface
{
    public const STATUS_SUCCESS = 'SUCCESS';
    public const STATUS_FAILED = 'FAILED';
    public const STATUS_PENDING = 'PENDING';
}
```

---

## 3. Dat Ten Theo Loai Thanh Phan

### 3.1 Layer Application

| Loai Component | Quy Tac | Vi du C# | Vi du Java | Vi du Python | Vi du Go | Vi du TS |
|---|---|---|---|---|---|---|
| **Controller** | PascalCase + `Controller` | `OrderController` | `OrderController` | `OrderController` | `OrderHandler` | `OrderController` |
| **Service** | PascalCase + `Service` | `OrderService` | `OrderService` | `OrderService` | `OrderService` | `OrderService` |
| **Repository** | PascalCase + `Repository` | `OrderRepository` | `OrderRepository` | `OrderRepository` | `OrderRepository` | `OrderRepository` |
| **UseCase** | PascalCase + `UseCase` | `CreateOrderUseCase` | `CreateOrderUseCase` | `CreateOrderUseCase` | `CreateOrderUseCase` | `CreateOrderUseCase` |
| **Handler** | PascalCase + `Handler` | `CreateOrderCommandHandler` | `CreateOrderHandler` | `CreateOrderHandler` | `CreateOrderHandler` | `CreateOrderHandler` |
| **Command** | PascalCase + `Command` | `CreateOrderCommand` | `CreateOrderCommand` | `CreateOrderCommand` | `CreateOrderCommand` | `CreateOrderCommand` |
| **Query** | PascalCase + `Query` | `GetOrderByIdQuery` | `GetOrderByIdQuery` | `GetOrderByIdQuery` | `GetOrderByIdQuery` | `GetOrderByIdQuery` |
| **Event** | PascalCase + `Event` | `OrderPlacedEvent` | `OrderPlacedEvent` | `OrderPlacedEvent` | `OrderPlacedEvent` | `OrderPlacedEvent` |
| **Factory** | PascalCase + `Factory` | `OrderFactory` | `OrderFactory` | `OrderFactory` | `OrderFactory` | `OrderFactory` |
| **Validator** | PascalCase + `Validator` | `CreateOrderValidator` | `CreateOrderValidator` | `CreateOrderValidator` | `CreateOrderValidator` | `CreateOrderValidator` |
| **Mapper** | PascalCase + `Mapper` | `OrderMapper` | `OrderMapper` | `OrderMapper` | `OrderMapper` | `OrderMapper` |
| **Middleware** | PascalCase + `Middleware` | `AuthenticationMiddleware` | `AuthenticationFilter` | `AuthenticationMiddleware` | `AuthMiddleware` | `AuthMiddleware` |
| **Decorator** | PascalCase + `Decorator` | `LoggingOrderServiceDecorator` | `LoggingOrderServiceDecorator` | `LoggingOrderServiceDecorator` | N/A | `LoggingOrderServiceDecorator` |
| **Interceptor** | PascalCase + `Interceptor` | `LoggingInterceptor` | `LoggingInterceptor` | N/A | N/A | `LoggingInterceptor` |
| **Pipeline** | PascalCase + `PipelineBehavior` | `ValidationPipelineBehavior` | N/A | N/A | N/A | N/A |
| **Specification** | PascalCase + `Specification` | `ActiveUserSpecification` | `ActiveUserSpecification` | `ActiveUserSpecification` | N/A | `ActiveUserSpecification` |
| **Builder** | PascalCase + `Builder` | `OrderBuilder` | `OrderBuilder` | `OrderBuilder` | `OrderBuilder` | `OrderBuilder` |
| **Provider** | PascalCase + `Provider` | `JwtTokenProvider` | `JwtTokenProvider` | `JwtTokenProvider` | `JwtTokenProvider` | `JwtTokenProvider` |
| **Resolver** | PascalCase + `Resolver` | `UserResolver` | `UserResolver` | `UserResolver` | N/A | `UserResolver` |
| **Converter** | PascalCase + `Converter` | `OrderStatusConverter` | `OrderStatusConverter` | `OrderStatusConverter` | N/A | `OrderStatusConverter` |
| **Publisher** | PascalCase + `Publisher` | `OrderEventPublisher` | `OrderEventPublisher` | `OrderEventPublisher` | `OrderEventPublisher` | `OrderEventPublisher` |
| **Consumer** | PascalCase + `Consumer` | `OrderCreatedEventConsumer` | `OrderCreatedEventConsumer` | `OrderCreatedEventConsumer` | `OrderCreatedEventConsumer` | `OrderCreatedEventConsumer` |
| **Scheduler** | PascalCase + `Scheduler` hoac `Job` | `OrderCleanupJob` | `OrderCleanupJob` | `OrderCleanupJob` | `OrderCleanupJob` | `OrderCleanupJob` |
| **Gateway** | PascalCase + `Gateway` | `PaymentGateway` | `PaymentGateway` | `PaymentGateway` | `PaymentGateway` | `PaymentGateway` |
| **Client** | PascalCase + `Client` | `StripePaymentClient` | `StripePaymentClient` | `StripePaymentClient` | `StripePaymentClient` | `StripePaymentClient` |

---

### 3.2 Domain Layer (DDD)

#### Entity

```
# Quy tac: PascalCase, danh tu so it, mo ta ro doi tuong domain
# Khong them suffix "Entity"

# Dung:
Order
User
Product
PaymentTransaction
ShippingAddress
ProductCategory
OrderItem
Invoice
Subscription

# Sai:
OrderEntity           # suffix "Entity" - khong can thiet
order                 # chu thuong
Orders                # plural
OrderModel            # suffix "Model" - dat cho data model, khong phai entity
```

#### Value Object

```
# Quy tac: PascalCase, mo ta RO RET gia tri/khai niem

# Dung:
Money
Address
Email
PhoneNumber
OrderNumber
Coordinates
DateRange
ProductSku
CreditCardNumber
FullName

# Sai:
MoneyValue           # "Value" thua vi Value Object da la khai niem
AddressVO            # suffix "VO" - khong can thiet
emailAddress         # camelCase
```

#### Aggregate Root

```
# Quy tac: Giong Entity - PascalCase, danh tu so it
# Aggregate Root la entity, khong can them suffix

# Dung:
Order                # Aggregate root cua Order aggregate
Cart                 # Aggregate root cua Cart aggregate
User                 # Aggregate root cua User aggregate
Product              # Aggregate root cua Product aggregate

# Sai:
OrderAggregate       # suffix "Aggregate" - khong can thiet
OrderRoot            # suffix "Root" - khong can thiet
```

#### Repository Interface (Domain)

```
# Quy tac (C#): I + [Entity] + Repository
# Quy tac (Java/TS): [Entity] + Repository (khong co I prefix)

# C#:
IOrderRepository
IUserRepository
IProductRepository

# Java:
OrderRepository
UserRepository
ProductRepository

# TypeScript:
OrderRepository
UserRepository

# Sai:
IOrderRepo           # viet tat "Repo" - khong nhat quan
IOrderStore          # "Store" - co the nham voi state store
OrderRepositoryInterface  # qua dai
```

#### Domain Service

```
# Quy tac: PascalCase + [Noun] + DomainService hoac [Noun] + Service
# Domain Service xu ly logic khong thuoc ve mot entity cu the

# Dung:
OrderPricingService
PaymentProcessingService
ShippingCostCalculationService
InventoryAllocationService
DiscountApplicationService

# Sai:
OrderDomainService    # prefix "Domain" thua
pricingService        # camelCase
```

#### Domain Event

```
# Quy tac: [Entity] + [PastTenseVerb] + Event
# Phan anh dieu gi da xay ra trong qua khu

# Dung:
OrderPlacedEvent
OrderConfirmedEvent
OrderCancelledEvent
OrderShippedEvent
PaymentCompletedEvent
PaymentFailedEvent
UserRegisteredEvent
UserEmailVerifiedEvent
PasswordChangedEvent
ProductOutOfStockEvent
InventoryReservedEvent

# Sai:
OrderPlaceEvent      # "Place" khong phai qua khu
PlaceOrderEvent      # dong tu truoc - khong phai event, giong command
OrderEvent           # qua chung, khong ro dieu gi xay ra
order_placed_event   # snake_case
```

---

### 3.3 DTO va Data Models

#### Request DTO

| Hanh Dong | Quy Tac | Vi du C# / TS | Vi du Java / Python |
|---|---|---|---|
| Tao moi | `Create[Entity]Request` | `CreateOrderRequest` | `CreateOrderRequest` |
| Cap nhat toan phan | `Update[Entity]Request` | `UpdateOrderRequest` | `UpdateOrderRequest` |
| Cap nhat mot phan | `Patch[Entity]Request` | `PatchOrderRequest` | `PatchOrderRequest` |
| Tim kiem / Loc | `[Entity]FilterRequest` | `OrderFilterRequest` | `OrderFilterRequest` |
| Phan trang | `[Entity]PageRequest` | `OrderPageRequest` | `OrderPageRequest` |
| Dang nhap | `[Action]Request` | `LoginRequest` | `LoginRequest` |
| Doi mat khau | `[Action]Request` | `ChangePasswordRequest` | `ChangePasswordRequest` |
| Xac thuc email | `[Action]Request` | `VerifyEmailRequest` | `VerifyEmailRequest` |

#### Response DTO

| Loai | Quy Tac | Vi du |
|---|---|---|
| Chi tiet day du | `[Entity]Response` | `OrderResponse` |
| Xem ngan / danh sach | `[Entity]SummaryResponse` | `OrderSummaryResponse` |
| Phan trang | `PagedResponse<T>` hoac `[Entity]PageResponse` | `PagedResponse<OrderSummaryResponse>` |
| Xac thuc | `[Action]Response` | `LoginResponse`, `RefreshTokenResponse` |
| Tao moi | `Create[Entity]Response` | `CreateOrderResponse` |
| Thong tin co ban | `[Entity]BasicResponse` | `UserBasicResponse` |
| Thong ke / Bao cao | `[Entity]StatisticsResponse` | `OrderStatisticsResponse` |

#### Internal DTO (trung gian giua cac layer)

```
# Request Command:
CreateOrderCommand
UpdateOrderStatusCommand
CancelOrderCommand
ProcessPaymentCommand

# Query:
GetOrderByIdQuery
GetOrdersByUserIdQuery
GetActiveOrdersQuery

# Internal DTO:
OrderDto              # trung gian giua Application va Domain
UserDto
PaymentDto

# Event payload:
OrderPlacedEventData
PaymentCompletedEventData
```

---

## 4. Dat Ten Database

### Ten Bang (Tables)

| Dung | Sai | Ghi Chu |
|---|---|---|
| `users` | `User`, `user`, `tbl_users`, `t_users` | snake_case, plural, khong prefix |
| `orders` | `Order`, `Orders`, `tbl_order` | plural |
| `order_items` | `OrderItems`, `orderItem`, `items_of_order` | compound: snake_case |
| `product_categories` | `ProductCategory`, `products_categories`, `category` | plural cho moi tu |
| `payment_transactions` | `PaymentTransaction`, `transactions` | ro rang, plural |
| `shipping_addresses` | `ShippingAddress`, `ship_addr` | khong viet tat |
| `user_roles` | `UserRole`, `roles_users` | bang trung gian: [table1]_[table2] |
| `role_permissions` | `RolePermission`, `permissions` | bang trung gian |
| `audit_logs` | `AuditLog`, `Logs`, `tbl_audit` | |
| `refresh_tokens` | `RefreshToken`, `tokens` | |
| `email_verifications` | `EmailVerification`, `email_verify` | |
| `product_images` | `ProductImage`, `images` | |
| `discount_codes` | `DiscountCode`, `coupons`, `promo_code` | nhat quan trong project |
| `inventory_snapshots` | `InventorySnapshot`, `inv_snap` | |
| `notification_templates` | `NotificationTemplate`, `templates` | |
| `system_configurations` | `SystemConfiguration`, `config`, `settings` | |

### Ten Cot (Columns)

#### Primary Key

```sql
-- Quy tac: chi dung "id" (khong phai user_id, order_id tren chinh bang do)
-- Tren bang users:
id UUID PRIMARY KEY DEFAULT gen_random_uuid()
-- Tren bang orders:
id UUID PRIMARY KEY DEFAULT gen_random_uuid()

-- Sai:
user_id UUID PRIMARY KEY   -- tren chinh bang users
order_id UUID PRIMARY KEY  -- tren chinh bang orders
ID UUID PRIMARY KEY        -- viet hoa
UserId UUID PRIMARY KEY    -- camelCase
```

#### Foreign Key

```sql
-- Quy tac: [referenced_table_singular]_id

-- Bang order_items:
order_id UUID REFERENCES orders(id)          -- tro toi bang orders
product_id UUID REFERENCES products(id)      -- tro toi bang products

-- Bang orders:
user_id UUID REFERENCES users(id)            -- tro toi bang users
coupon_id UUID REFERENCES discount_codes(id) -- tro toi bang discount_codes

-- Bang user_roles:
user_id UUID REFERENCES users(id)
role_id UUID REFERENCES roles(id)

-- Sai:
orderId UUID REFERENCES orders(id)           -- camelCase
fk_order_id UUID REFERENCES orders(id)      -- prefix fk
orders_id UUID REFERENCES orders(id)         -- plural table name
```

#### Cot Boolean

```sql
-- Quy tac: is_[state], has_[thing], can_[action]
is_active BOOLEAN NOT NULL DEFAULT TRUE
is_deleted BOOLEAN NOT NULL DEFAULT FALSE
is_verified BOOLEAN NOT NULL DEFAULT FALSE
is_featured BOOLEAN NOT NULL DEFAULT FALSE
has_discount BOOLEAN NOT NULL DEFAULT FALSE
has_free_shipping BOOLEAN NOT NULL DEFAULT FALSE
can_edit BOOLEAN NOT NULL DEFAULT FALSE

-- Sai:
active BOOLEAN          -- thieu prefix is/has
deleted BOOLEAN         -- thieu prefix
verified BOOLEAN        -- thieu prefix
status_active BOOLEAN   -- khong ro rang
```

#### Cot Timestamp

```sql
-- Quy tac: [event]_at (gio xay ra su kien)
created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
updated_at TIMESTAMP WITH TIME ZONE
deleted_at TIMESTAMP WITH TIME ZONE       -- soft delete
published_at TIMESTAMP WITH TIME ZONE
confirmed_at TIMESTAMP WITH TIME ZONE
cancelled_at TIMESTAMP WITH TIME ZONE
shipped_at TIMESTAMP WITH TIME ZONE
delivered_at TIMESTAMP WITH TIME ZONE
expired_at TIMESTAMP WITH TIME ZONE
last_login_at TIMESTAMP WITH TIME ZONE
verified_at TIMESTAMP WITH TIME ZONE

-- Sai:
created TIMESTAMP          -- thieu _at
create_time TIMESTAMP      -- _time thay vi _at
createdAt TIMESTAMP        -- camelCase
creation_date TIMESTAMP    -- _date (nen dung _at cho timestamp)
```

#### Cot Enum/Status/Type

```sql
-- Quy tac: [noun]_status, [noun]_type
order_status VARCHAR(50) NOT NULL DEFAULT 'PENDING'
payment_status VARCHAR(50) NOT NULL DEFAULT 'PENDING'
payment_method VARCHAR(50)
user_role VARCHAR(50) NOT NULL DEFAULT 'CUSTOMER'
notification_type VARCHAR(50) NOT NULL
account_type VARCHAR(50) NOT NULL
product_type VARCHAR(50)

-- Sai:
status VARCHAR(50)          -- qua chung, co the nham lan
type VARCHAR(50)            -- qua chung
orderStatus VARCHAR(50)     -- camelCase
```

#### Cot So luong / Gia tri

```sql
-- Ten ro rang, don vi neu can
quantity INTEGER NOT NULL DEFAULT 0
unit_price NUMERIC(10, 2) NOT NULL
total_amount NUMERIC(15, 2) NOT NULL
discount_amount NUMERIC(10, 2) NOT NULL DEFAULT 0
tax_amount NUMERIC(10, 2) NOT NULL DEFAULT 0
weight_grams INTEGER          -- don vi trong ten
height_cm INTEGER             -- don vi trong ten
timeout_seconds INTEGER
retry_count INTEGER NOT NULL DEFAULT 0
view_count INTEGER NOT NULL DEFAULT 0
sort_order INTEGER NOT NULL DEFAULT 0
```

### Ten Index

```sql
-- Quy tac: idx_[table]_[column(s)]
-- Nhieu cot: gach ngang ngan cach moi cot

-- Single column indexes:
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(is_active);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(order_status);
CREATE INDEX idx_products_category_id ON products(category_id);

-- Composite indexes:
CREATE INDEX idx_orders_user_id_status ON orders(user_id, order_status);
CREATE INDEX idx_products_category_id_created_at ON products(category_id, created_at DESC);
CREATE INDEX idx_order_items_order_id_product_id ON order_items(order_id, product_id);
CREATE INDEX idx_audit_logs_user_id_created_at ON audit_logs(user_id, created_at DESC);

-- Partial indexes:
CREATE INDEX idx_users_email_verified ON users(email) WHERE is_verified = TRUE;
CREATE INDEX idx_orders_pending ON orders(created_at) WHERE order_status = 'PENDING';

-- Unique index:
CREATE UNIQUE INDEX idx_users_email_unique ON users(email);
CREATE UNIQUE INDEX idx_products_sku_unique ON products(sku);

-- Sai:
CREATE INDEX users_email ON users(email);           -- thieu prefix idx
CREATE INDEX index_users_email ON users(email);     -- "index" qua dai
CREATE INDEX email_idx ON users(email);             -- suffix thay vi prefix
CREATE INDEX i_users_email ON users(email);         -- prefix i, khong ro
```

### Ten Constraint

```sql
-- Primary Key:
CONSTRAINT pk_users PRIMARY KEY (id)
CONSTRAINT pk_orders PRIMARY KEY (id)
CONSTRAINT pk_order_items PRIMARY KEY (id)

-- Foreign Key:
CONSTRAINT fk_orders_users FOREIGN KEY (user_id) REFERENCES users(id)
CONSTRAINT fk_order_items_orders FOREIGN KEY (order_id) REFERENCES orders(id)
CONSTRAINT fk_order_items_products FOREIGN KEY (product_id) REFERENCES products(id)
CONSTRAINT fk_user_roles_users FOREIGN KEY (user_id) REFERENCES users(id)
CONSTRAINT fk_user_roles_roles FOREIGN KEY (role_id) REFERENCES roles(id)

-- Unique:
CONSTRAINT uq_users_email UNIQUE (email)
CONSTRAINT uq_products_sku UNIQUE (sku)
CONSTRAINT uq_user_roles_user_role UNIQUE (user_id, role_id)

-- Check:
CONSTRAINT chk_orders_total_positive CHECK (total_amount >= 0)
CONSTRAINT chk_order_items_quantity_positive CHECK (quantity > 0)
CONSTRAINT chk_products_price_positive CHECK (unit_price > 0)
CONSTRAINT chk_users_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')

-- Sai:
PRIMARY KEY (id)                           -- khong ten constraint
FOREIGN KEY (user_id) REFERENCES users(id) -- khong ten constraint
user_id_fk                                 -- suffix thay vi prefix fk
orders_users_fk                            -- sai thu tu
```

### Ten Migration

```sql
-- Quy tac Option 1: YYYYMMDD_HHMMSS_[description].sql (Flyway V timelstamp)
-- Quy tac Option 2: V[NNN]__[description].sql (Flyway versioned)
-- Quy tac Option 3: [timestamp]_[description].sql (nhiều ORM)

-- Option 1 - Timestamp:
20240101_120000_create_users_table.sql
20240102_090000_create_orders_table.sql
20240103_150000_add_payment_method_to_orders.sql
20240110_120000_create_order_items_table.sql
20240115_080000_add_index_orders_user_id.sql
20240120_143000_alter_users_add_phone_number.sql

-- Option 2 - Flyway versioned:
V001__create_users_table.sql
V002__create_products_table.sql
V003__create_orders_table.sql
V004__create_order_items_table.sql
V005__add_payment_status_to_orders.sql
V006__create_index_users_email.sql
V007__alter_orders_add_shipping_address.sql

-- Option 3 - EF Core / Rails style:
20240101120000_CreateUsersTable.cs        -- EF Core
20240101120001_AddEmailIndexToUsers.cs
20240102090000_CreateOrdersTable.cs

-- Sai:
migration_1.sql                           -- khong mo ta
new_migration.sql                         -- khong co thong tin
fix.sql                                   -- qua chung
001_users.sql                             -- thieu chu description ro rang
```

---

## 5. Dat Ten API

### URL Resources

| Dung | Sai | Ghi Chu |
|---|---|---|
| `GET /users` | `GET /User`, `GET /getUsers` | Plural, khong co dong tu |
| `GET /users/{userId}` | `GET /users/{user_id}`, `GET /getUserById` | camelCase path param |
| `POST /users` | `POST /createUser`, `POST /user` | Khong dong tu trong URL |
| `PUT /users/{userId}` | `PUT /updateUser/{userId}` | Khong dong tu |
| `DELETE /users/{userId}` | `DELETE /deleteUser/{userId}` | Khong dong tu |
| `GET /order-items` | `GET /orderItems`, `GET /order_items` | kebab-case, plural |
| `GET /product-categories` | `GET /productCategories`, `GET /ProductCategories` | kebab-case |
| `GET /orders/{orderId}/items` | `GET /orders/{orderId}/getItems` | Nested resource, khong dong tu |
| `POST /orders/{orderId}/cancel` | `POST /cancelOrder/{orderId}` | Hanh dong dac biet: nested verb ok |
| `POST /orders/{orderId}/items` | `POST /orders/{orderId}/addItem` | Khong dong tu cho CRUD |
| `GET /users/{userId}/orders` | `GET /getUserOrders/{userId}` | Nested resource |
| `GET /products/search` | `GET /searchProducts`, `GET /products/Search` | "search" la resource exception |
| `POST /auth/login` | `POST /login`, `POST /auth/Login` | Authentication group |
| `POST /auth/logout` | `POST /logout`, `POST /auth/signout` | Nhat quan trong project |
| `POST /auth/refresh` | `POST /refreshToken`, `POST /auth/refresh-token` | Nhat quan |
| `PATCH /users/{userId}/password` | `POST /users/{userId}/changePassword` | PATCH cho partial update |

#### Versioning

```
# Quy tac: prefix /v[N]/ trong URL path (pho bien nhat)

# Dung:
GET /v1/users
GET /v2/users
GET /v1/orders/{orderId}
GET /v2/products

# Hoac header versioning:
GET /users
Header: X-Api-Version: 2

# Hoac subdomain:
GET https://v2.api.company.com/users

# Sai:
GET /users/v1          # version o cuoi
GET /usersV2           # version trong resource name
```

### URL Parameters

#### Path Parameters

```
# Quy tac: camelCase trong template
# Ten param nen dung lại chính xác tên field trong schema

# Dung:
/users/{userId}
/orders/{orderId}
/products/{productId}
/categories/{categoryId}
/orders/{orderId}/items/{itemId}
/users/{userId}/addresses/{addressId}

# Sai:
/users/{user_id}          # snake_case
/users/{user-id}          # kebab-case
/users/{UserID}           # PascalCase
/users/{id}               # qua chung khi co nested route
```

#### Query Parameters

```
# Quy tac: camelCase

# Phan trang:
GET /orders?page=1&pageSize=20&sortBy=createdAt&sortOrder=desc

# Tim kiem / Loc:
GET /products?search=laptop&categoryId=abc123&minPrice=100&maxPrice=500

# Include / Expand:
GET /orders?include=items,user&fields=id,status,totalAmount

# Trang thai:
GET /orders?status=PENDING&userId=user123

# Thoi gian:
GET /orders?startDate=2024-01-01&endDate=2024-12-31

# Sai:
GET /orders?page_size=20         # snake_case
GET /orders?PageSize=20          # PascalCase
GET /orders?page-size=20         # kebab-case
GET /orders?sort=createdAt&dir=desc   # "dir" viet tat qua
```

### HTTP Headers (Custom)

```
# Quy tac: X-[Company]-[Purpose] hoac X-[Purpose] (Train-Case)
# Dung "X-" prefix cho custom headers

# Standard:
Content-Type: application/json
Authorization: Bearer <token>
Accept: application/json
Accept-Language: vi-VN,vi;q=0.9,en;q=0.8
Cache-Control: no-cache
If-None-Match: "abc123"

# Custom - Tracking:
X-Request-Id: 550e8400-e29b-41d4-a716-446655440000
X-Correlation-Id: 7f3d9a2b-1234-5678-abcd-ef0123456789
X-Trace-Id: 4bf92f3577b34da6a3ce929d0e0e4736

# Custom - Auth / Security:
X-Api-Key: <api-key>
X-Idempotency-Key: <uuid>
X-Signature: <hmac-signature>
X-Timestamp: 1704067200

# Custom - Business:
X-Tenant-Id: tenant-acme
X-User-Id: user123
X-Api-Version: 2
X-Client-Version: 3.2.1

# Sai:
x-request-id: ...         # tat ca thuong
X_REQUEST_ID: ...         # UPPER_SNAKE
xRequestId: ...           # camelCase
XRequestId: ...           # PascalCase (khong co dau gach ngang)
```

---

## 6. Dat Ten Thu Muc va File

### Thu Muc (Folders)

| Dung | Sai | Ghi Chu |
|---|---|---|
| `user-management` | `UserManagement`, `user_management`, `usermanagement` | kebab-case |
| `order-processing` | `OrderProcessing`, `order_processing` | kebab-case |
| `background-jobs` | `BackgroundJobs`, `background_jobs`, `jobs` | ro rang, kebab-case |
| `domain-events` | `DomainEvents`, `domain_events` | kebab-case |
| `use-cases` | `UseCases`, `use_cases`, `usecases` | kebab-case |
| `api-gateway` | `ApiGateway`, `api_gateway` | kebab-case |
| `shared-kernel` | `SharedKernel`, `shared_kernel` | kebab-case |
| `integration-tests` | `IntegrationTests`, `integration_tests` | kebab-case |
| `unit-tests` | `UnitTests`, `unit_tests` | kebab-case |
| `docker` | `Docker`, `DOCKER` | tat ca thuong ok cho cong cu |
| `scripts` | `Scripts`, `SCRIPTS` | tat ca thuong ok |
| `.github` | `.Github`, `.GITHUB` | theo quy dinh cua GitHub |

```
# Cau truc thu muc dung:
src/
  user-management/
    domain/
      entities/
      value-objects/
      events/
      repositories/
    application/
      commands/
      queries/
      handlers/
    infrastructure/
      persistence/
      messaging/
    presentation/
      controllers/
      dtos/

# Cau truc thu muc sai:
src/
  UserManagement/        # PascalCase
  user_management/       # snake_case
  usermanagement/        # khong co phan cach
```

### File Source Code

| Ngon Ngu | Convention | Vi du Dung | Vi du Sai |
|---|---|---|---|
| **C#** | PascalCase.cs | `OrderService.cs`, `UserController.cs` | `order-service.cs`, `orderService.cs` |
| **Java** | PascalCase.java | `OrderService.java`, `UserController.java` | `order_service.java`, `orderService.java` |
| **Python** | snake_case.py | `order_service.py`, `user_controller.py` | `OrderService.py`, `orderService.py` |
| **Go** | snake_case.go | `order_service.go`, `user_handler.go` | `OrderService.go`, `orderService.go` |
| **TypeScript** | kebab-case.ts | `order-service.ts`, `user-controller.ts` | `OrderService.ts`, `orderService.ts` |
| **PHP** | PascalCase.php | `OrderService.php`, `UserController.php` | `order_service.php`, `orderService.php` |

### File Test

| Ngon Ngu | Convention | Vi du |
|---|---|---|
| **C#** | `[FileName]Tests.cs` | `OrderServiceTests.cs`, `UserControllerTests.cs` |
| **C# Integration** | `[FileName]IntegrationTests.cs` | `OrderServiceIntegrationTests.cs` |
| **Java** | `[FileName]Test.java` | `OrderServiceTest.java` |
| **Java Integration** | `[FileName]IT.java` hoac `[FileName]IntegrationTest.java` | `OrderServiceIT.java` |
| **Python** | `test_[filename].py` | `test_order_service.py`, `test_user_controller.py` |
| **Go** | `[filename]_test.go` | `order_service_test.go`, `user_handler_test.go` |
| **TypeScript Jest** | `[filename].test.ts` | `order-service.test.ts` |
| **TypeScript Mocha** | `[filename].spec.ts` | `order-service.spec.ts` |
| **PHP** | `[FileName]Test.php` | `OrderServiceTest.php` |

### File Config / DevOps

```
# Docker:
Dockerfile                    # production default
Dockerfile.dev                # development
Dockerfile.prod               # production (neu co nhieu variant)
Dockerfile.test               # testing
.dockerignore

# Docker Compose:
docker-compose.yml            # production / default
docker-compose.dev.yml        # development override
docker-compose.test.yml       # testing
docker-compose.prod.yml       # production specific

# Kubernetes:
user-service-deployment.yaml
user-service-service.yaml     # Service resource (svc)
user-service-hpa.yaml         # HorizontalPodAutoscaler
user-service-configmap.yaml
user-service-secret.yaml
user-service-ingress.yaml
user-service-pdb.yaml         # PodDisruptionBudget

# Helm:
Chart.yaml
values.yaml
values-dev.yaml
values-prod.yaml
templates/
  deployment.yaml
  service.yaml
  ingress.yaml

# CI/CD (.github/workflows/):
ci.yml                        # Continuous Integration
cd.yml                        # Continuous Deployment
security-scan.yml             # Quet bao mat
code-quality.yml              # SonarQube, Snyk
release.yml                   # Tao release
dependency-update.yml         # Dependabot / Renovate

# Config files:
appsettings.json              # C#
appsettings.Development.json
appsettings.Production.json
application.yml               # Java Spring
application-dev.yml
application-prod.yml
.env                          # dotenv (KHONG commit)
.env.example                  # commit template
config.yaml                   # Go
config.dev.yaml
```

---

## 7. Dat Ten Bien

### Bien Thuong Gap

| Loai | Quy Tac | Vi du Dung | Vi du Sai |
|---|---|---|---|
| **Boolean** | `is/has/can/should` + adjective/noun | `isActive`, `hasPermission`, `canEdit`, `shouldRetry` | `active`, `permission`, `edit`, `retry` |
| **Collection** | plural noun | `users`, `orderItems`, `productIds`, `categories` | `userList`, `orderData`, `items_arr` |
| **Single item** | singular noun | `user`, `order`, `currentProduct` | `userData`, `orderObject`, `the_user` |
| **Count** | noun + `Count` hoac `total` + Noun | `userCount`, `totalItems`, `pageCount` | `numUsers`, `cntOrders`, `num` |
| **Index trong loop** | `i`, `j`, `k` (CHI trong loop don gian) | `for (int i = 0; i < n; i++)` | `for (int x = 0; ...)`, `for (int index = ...)` |
| **Index ngoai loop** | `[noun]Index` | `currentIndex`, `activeTabIndex` | `i` (ngoai loop), `idx` |
| **Temp / item trong loop** | `item`, `element`, `entry`, noun singular | `for item in items:`, `orders.forEach(order => ...)` | `x`, `y`, `z`, `obj`, `elem` |
| **Result / return** | descriptive: `foundUser`, `createdOrder` | `foundUser`, `createdOrder`, `matchedProducts` | `result`, `res`, `ret`, `data` |
| **Error** | `err` (Go), `error` (Python), `e`/`exception` (Java/C#) | `err`, `error`, `validationError` | `e` trong Java/C# (qua ngan), `ex` |
| **Optional/Nullable** | `optional` prefix hoac `OrNull` suffix (C#) | `optionalUser`, `userOrNull`, `maybeUser` | `nullableUser`, `userNullable` |
| **Pointer / Reference** | descriptive (Go: no hungarian) | `orderPtr *Order` ok ngoai Go | `p`, `ptr`, `ref` |
| **Context** | `ctx` (Go convention) | `ctx context.Context` | `context`, `c` |
| **Request / Response** | `req`, `resp` hoac `request`, `response` | `req`, `resp`, `request`, `response` | `r`, `rq`, `rs` |
| **Configuration** | `config`, `cfg`, `settings` | `config`, `cfg` | `c`, `conf`, `cnf` |
| **Database connection** | `db`, `conn`, `connection` | `db`, `conn` | `d`, `database` (qua dai cho local) |

### Boolean Naming

```
# Quy tac: luon bat dau bang is/has/can/should/will/did
# Doc nhu mot cau hoi co/khong

# Dung:
isActive
isDeleted
isVerified
isLoggedIn
isFeatured
hasPermission
hasDiscount
hasChildren
canEdit
canDelete
canPublish
shouldRetry
shouldSkipValidation
willExpire
didSendEmail

# Sai:
active            # adjective thuan, khong clear la boolean
loggedIn          # thieu prefix
userVerified      # noun + adjective, khong ro
featureEnabled    # noun + adjective, nen la "isFeatureEnabled"
editAllowed       # cau truc nguoc
deleted           # past tense, khong ro la boolean
```

### Collection va Plural Naming

```
# Quy tac: collection luon la plural noun
# Khong them suffix List, Array, Collection vao ten

# Dung:
users              # List<User>, IEnumerable<User>
orders             # []Order, List<Order>
productIds         # List<Guid>, []uuid.UUID
categoryNames      # List<string>
activeOrderItems   # filtered collection

# Sai:
userList           # suffix "List" - kieu du lieu la implementation detail
orderArray         # suffix "Array"
usersCollection    # suffix "Collection"
listOfOrders       # prefix "listOf"
ordersData         # suffix "Data" qua chung

# Ngoai le - khi can phan biet nhieu collection cung entity:
confirmedOrders    # descriptive adjective ok
pendingOrders
cancelledOrders
recentOrders
```

### Constants

```
# Quy tac: UPPER_SNAKE_CASE
# Ten phai day du, khong viet tat

# Time:
JWT_EXPIRY_SECONDS = 900
CACHE_TTL_MINUTES = 30
SESSION_TIMEOUT_MINUTES = 60
PAYMENT_LOCK_TIMEOUT_SECONDS = 300
HTTP_REQUEST_TIMEOUT_MS = 5000

# Pagination:
DEFAULT_PAGE_SIZE = 20
MAX_PAGE_SIZE = 100
MIN_PAGE_SIZE = 1

# Retry:
MAX_RETRY_COUNT = 3
RETRY_DELAY_MS = 1000
RETRY_MULTIPLIER = 2

# Limits:
MAX_ITEMS_PER_ORDER = 100
MAX_FILE_SIZE_MB = 10
MAX_LOGIN_ATTEMPTS = 5
MAX_PASSWORD_LENGTH = 128
MIN_PASSWORD_LENGTH = 8

# URLs / Paths:
BASE_API_URL = "https://api.company.com"
HEALTH_CHECK_PATH = "/health"

# Sai:
maxRetryCount = 3          # camelCase - trong nhu variable
max_retry = 3              # snake_case, "count" bi thieu
MAXRETRYCOUNT = 3          # khong co dau phan cach
MRC = 3                    # viet tat khong ro nghia
```

---

## 8. Dat Ten Event va Queue

### Domain Events

```
# Quy tac: [Entity][PastTenseVerb]Event

# Order domain:
OrderPlacedEvent
OrderConfirmedEvent
OrderProcessingStartedEvent
OrderShippedEvent
OrderDeliveredEvent
OrderCancelledEvent
OrderReturnRequestedEvent

# Payment domain:
PaymentInitiatedEvent
PaymentCompletedEvent
PaymentFailedEvent
PaymentRefundedEvent
PaymentMethodChangedEvent

# User domain:
UserRegisteredEvent
UserEmailVerifiedEvent
UserPasswordChangedEvent
UserProfileUpdatedEvent
UserSuspendedEvent
UserDeactivatedEvent

# Inventory domain:
InventoryReservedEvent
InventoryReleasedEvent
ProductOutOfStockEvent
ProductRestockedEvent

# Sai:
OrderPlaceEvent           # "Place" khong phai qua khu
PlaceOrderEvent           # dong tu truoc (giong command)
OrderEvent                # qua chung
OrderStatusChangedEvent   # ok nhung qua chung - nen specifc hon
order_placed_event        # snake_case
```

### Integration Events (Cross-service)

```
# Quy tac: [Entity][PastTenseVerb]IntegrationEvent
# Dung cho su kien truyen qua cac service khac nhau (message bus)

OrderPlacedIntegrationEvent
PaymentCompletedIntegrationEvent
UserRegisteredIntegrationEvent
InventoryReservedIntegrationEvent
ShipmentCreatedIntegrationEvent

# Payload structure (JSON):
{
  "eventId": "uuid",
  "eventType": "order.order.placed",
  "occurredAt": "2024-01-15T10:30:00Z",
  "version": "1",
  "data": {
    "orderId": "uuid",
    "userId": "uuid",
    "totalAmount": 150.00
  }
}
```

### Queue / Topic Names

```
# Quy tac: [service].[priority].[action-type]
# Hoac: [service].[entity].[action]
# Tat ca lowercase, dau gach ngang hoac dot phan cach

# RabbitMQ / ActiveMQ style (dot separator):
order-service.high.payment-processing
order-service.medium.order-confirmation
order-service.low.order-cleanup
email-service.high.transactional-email
email-service.low.marketing-email
notification-service.high.push-notification
notification-service.medium.in-app-notification
inventory-service.high.stock-reservation
inventory-service.medium.stock-update

# Kafka style (dot separator):
order.order.placed
order.payment.completed
order.inventory.reserved
user.user.registered
user.email.verified
payment.payment.failed
notification.email.queued

# AWS SQS style:
order-service-payment-queue
order-service-payment-dlq              # dead letter queue suffix
email-service-notification-queue
email-service-notification-dlq

# Sai:
OrderPaymentQueue             # PascalCase
order_payment_queue           # snake_case (co the dung nhung khong nhat quan)
orderServiceHighPriority      # camelCase, qua dai
```

### Event Types (trong message payload)

```
# Quy tac: [domain].[entity].[past-tense-action] (tat ca chu thuong, dau cham phan cach)
# Rat cu the, doc duoc nhu "domain entity action"

# Order events:
"order.order.placed"
"order.order.confirmed"
"order.order.shipped"
"order.order.delivered"
"order.order.cancelled"
"order.payment.completed"
"order.payment.failed"
"order.item.added"
"order.item.removed"

# User events:
"user.user.registered"
"user.user.suspended"
"user.email.verified"
"user.password.changed"
"user.profile.updated"

# Inventory events:
"inventory.stock.reserved"
"inventory.stock.released"
"inventory.product.out-of-stock"

# Sai:
"OrderPlaced"             # PascalCase, khong co phan cach domain
"order_placed"            # snake_case, thieu domain
"ORDER.PLACED"            # UPPER_SNAKE
"placeOrder"              # command form, khong phai event
```

---

## 9. Dat Ten Trong DevOps / Infrastructure

### Docker Image

```
# Quy tac: [org/company]/[service-name]:[semver-version]
# Tat ca lowercase, dau gach ngang, semantic versioning

# Production images:
company/user-service:1.2.3
company/order-service:2.0.0
company/payment-service:1.0.5
company/notification-service:3.1.0
company/api-gateway:1.5.2

# Release candidates:
company/user-service:1.2.3-rc.1
company/user-service:1.2.3-rc.2
company/order-service:2.0.0-beta.1

# Development / CI:
company/user-service:latest
company/user-service:develop
company/user-service:sha-a1b2c3d

# Multi-arch:
company/user-service:1.2.3-linux-amd64
company/user-service:1.2.3-linux-arm64

# Sai:
Company/UserService:1.2.3      # PascalCase
company/user_service:v1.2.3    # snake_case, v prefix
company/userservice:latest     # khong phan cach
```

### Kubernetes Resources

```yaml
# Quy tac: [service-name]-[resource-type] (kebab-case)

# Deployment:
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service-deployment
  namespace: production

# Service (Kubernetes Service):
apiVersion: v1
kind: Service
metadata:
  name: user-service-svc     # hoac user-service (ngan hon)

# HorizontalPodAutoscaler:
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa

# ConfigMap:
apiVersion: v1
kind: ConfigMap
metadata:
  name: user-service-config

# Secret:
apiVersion: v1
kind: Secret
metadata:
  name: user-service-secret

# Ingress:
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: user-service-ingress

# PodDisruptionBudget:
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: user-service-pdb

# ServiceAccount:
apiVersion: v1
kind: ServiceAccount
metadata:
  name: user-service-sa

# Sai:
name: userServiceDeployment    # camelCase
name: user_service_deployment  # snake_case
name: UserService-Deployment   # PascalCase + kebab
```

#### Kubernetes Labels va Annotations

```yaml
# Labels (app.kubernetes.io/* standard labels):
metadata:
  labels:
    app.kubernetes.io/name: user-service
    app.kubernetes.io/version: "1.2.3"
    app.kubernetes.io/component: api
    app.kubernetes.io/part-of: ecommerce-platform
    app.kubernetes.io/managed-by: helm
    environment: production
    team: platform

# Annotations:
metadata:
  annotations:
    description: "User management service"
    contact: "platform-team@company.com"
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/metrics"
```

### Environment Variables

```bash
# Quy tac: UPPER_SNAKE_CASE, nhom theo prefix

# Application:
APP_ENV=production           # development, staging, production
APP_PORT=8080
APP_NAME=user-service
APP_VERSION=1.2.3
APP_LOG_LEVEL=info

# Database:
DB_HOST=postgres.internal
DB_PORT=5432
DB_NAME=ecommerce_db
DB_USER=app_user
DB_PASSWORD=<secret>
DB_MAX_CONNECTIONS=20
DB_CONNECTION_TIMEOUT_MS=5000
DB_SSL_MODE=require

# Redis:
REDIS_URL=redis://redis.internal:6379
REDIS_PASSWORD=<secret>
REDIS_DB=0
REDIS_TTL_SECONDS=3600
REDIS_MAX_CONNECTIONS=10

# JWT / Auth:
JWT_SECRET_KEY=<secret>
JWT_EXPIRY_SECONDS=900
JWT_REFRESH_EXPIRY_SECONDS=604800
JWT_ISSUER=https://auth.company.com
JWT_AUDIENCE=https://api.company.com

# AWS:
AWS_REGION=ap-southeast-1
AWS_ACCESS_KEY_ID=<secret>
AWS_SECRET_ACCESS_KEY=<secret>
AWS_S3_BUCKET=company-user-uploads
AWS_SQS_QUEUE_URL=https://sqs.ap-southeast-1.amazonaws.com/...
AWS_SNS_TOPIC_ARN=arn:aws:sns:...

# External Services:
STRIPE_API_KEY=<secret>
STRIPE_WEBHOOK_SECRET=<secret>
SENDGRID_API_KEY=<secret>
TWILIO_ACCOUNT_SID=<secret>
TWILIO_AUTH_TOKEN=<secret>

# Feature Flags:
FEATURE_NEW_CHECKOUT=true
FEATURE_AI_RECOMMENDATIONS=false
FEATURE_DARK_MODE=true
FEATURE_LOYALTY_PROGRAM=false

# Observability:
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
SENTRY_DSN=https://...
DATADOG_API_KEY=<secret>

# Sai:
dbHost=postgres.internal          # camelCase
Db_Host=postgres.internal         # khong nhat quan
database-host=postgres.internal   # kebab-case
DB-HOST=postgres.internal         # kebab-case voi hoa
```

### CI/CD Pipeline

```yaml
# GitHub Actions workflow naming:
# File: .github/workflows/ci.yml

name: Continuous Integration

jobs:
  # Quy tac job names: kebab-case, mo ta ro hanh dong
  lint-code:
    name: Lint Code
    runs-on: ubuntu-latest

  run-unit-tests:
    name: Run Unit Tests
    needs: lint-code

  run-integration-tests:
    name: Run Integration Tests
    needs: run-unit-tests

  build-docker-image:
    name: Build Docker Image
    needs: run-integration-tests

  scan-security-vulnerabilities:
    name: Scan Security Vulnerabilities
    needs: build-docker-image

  push-docker-image:
    name: Push Docker Image
    needs: scan-security-vulnerabilities

  deploy-to-staging:
    name: Deploy to Staging
    needs: push-docker-image
    environment: staging

  run-smoke-tests:
    name: Run Smoke Tests
    needs: deploy-to-staging

  deploy-to-production:
    name: Deploy to Production
    needs: run-smoke-tests
    environment: production

# Artifact naming:
# [service]-[version]-[git-sha].[ext]
# user-service-1.2.3-a1b2c3d.tar.gz
# order-service-2.0.0-b4c5d6e.docker.tar
```

---

## 10. Cache va Redis Keys

### Pattern va Quy Tac

```
# Quy tac: [namespace]:[version]:[entity]:[id][:[qualifier]]
# Tat ca LOWERCASE
# Dau hai cham : lam separator
# Include version de cache bust de dang

# Cau truc:
[namespace]:[version]:[entity]:[identifier]:[optional-qualifier]

# User cache:
cache:v1:user:12345
cache:v1:user:12345:profile
cache:v1:user:12345:permissions
cache:v1:user:12345:orders
cache:v1:user:12345:cart

# Product cache:
cache:v1:product:abc123
cache:v1:product:abc123:details
cache:v1:product:abc123:images
cache:v1:product:catalog:category:electronics
cache:v1:product:catalog:featured
cache:v1:product:catalog:new-arrivals

# Order cache:
cache:v1:order:order-uuid
cache:v1:orders:user:12345                # list orders cua user
cache:v1:orders:user:12345:page:1:size:20

# Session:
session:user:12345
session:token:abc-refresh-token

# Distributed Lock:
lock:order:order-uuid
lock:payment:payment-uuid
lock:inventory:product-uuid
lock:user:12345:profile-update

# Rate Limiting:
rate-limit:user:12345:create-order
rate-limit:ip:192.168.1.1:login
rate-limit:api-key:key123:global

# OTP / Verification:
otp:email:user@example.com:verification
otp:phone:+84901234567:reset-password

# Sai:
Cache:V1:User:12345               # PascalCase
cache_v1_user_12345               # snake_case (kho read, khong hierarchy)
user:12345                        # thieu namespace va version
CACHE:V1:USER:12345               # UPPER_SNAKE
```

### TTL Naming Convention

```
# Khi define TTL constants, dung UPPER_SNAKE_CASE voi don vi ro rang

USER_PROFILE_CACHE_TTL_SECONDS = 3600       # 1 hour
PRODUCT_CATALOG_CACHE_TTL_SECONDS = 7200    # 2 hours
ORDER_LIST_CACHE_TTL_SECONDS = 300          # 5 minutes
SESSION_TTL_SECONDS = 86400                 # 24 hours
OTP_TTL_SECONDS = 300                       # 5 minutes
RATE_LIMIT_WINDOW_SECONDS = 60             # 1 minute

# Sai:
USER_CACHE_TTL = 3600              # thieu don vi seconds/minutes
userCacheTtl = 3600                # camelCase
user_cache_ttl = 3600              # snake_case (nen UPPER_SNAKE cho constant)
```

---

## 11. Logging Fields

### Standard Fields (bat buoc)

```json
{
  "timestamp": "2024-01-15T10:30:00.000Z",
  "level": "INFO",
  "message": "Order created successfully",
  "service": "order-service",
  "version": "1.2.3",
  "environment": "production"
}
```

### Tracing / Correlation Fields

```json
{
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "spanId": "00f067aa0ba902b7",
  "correlationId": "7f3d9a2b-1234-5678-abcd-ef0123456789",
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "sessionId": "session-abc123"
}
```

### Context Fields

```json
{
  "userId": "user-uuid",
  "tenantId": "tenant-uuid",
  "organizationId": "org-uuid",
  "orderId": "order-uuid",
  "paymentId": "payment-uuid",
  "resourceType": "order",
  "resourceId": "order-uuid",
  "action": "create",
  "component": "OrderService",
  "method": "CreateOrder"
}
```

### HTTP Fields

```json
{
  "httpMethod": "POST",
  "httpPath": "/v1/orders",
  "httpStatus": 201,
  "httpDurationMs": 145,
  "httpUserAgent": "Mozilla/5.0 ...",
  "httpClientIp": "192.168.1.1",
  "httpRequestSize": 512,
  "httpResponseSize": 1024
}
```

### Duration / Performance Fields

```json
{
  "durationMs": 145,
  "dbQueryMs": 23,
  "externalCallMs": 87,
  "cacheHitMs": 2,
  "serializationMs": 5,
  "totalProcessingMs": 145
}
```

### Status / Flag Fields

```json
{
  "isSuccess": true,
  "isError": false,
  "isCached": false,
  "isRetry": false,
  "retryCount": 0,
  "isAuthenticated": true,
  "isAuthorized": true
}
```

### Error Fields

```json
{
  "errorCode": "ORDER_CREATION_FAILED",
  "errorMessage": "Insufficient stock for product ABC",
  "errorType": "BusinessException",
  "errorStack": "...",
  "validationErrors": [
    {
      "field": "quantity",
      "message": "Quantity must be positive"
    }
  ]
}
```

### Quy Tac Dat Ten Log Field

```
# Tat ca field: camelCase
# Khong viet tat (tru cac standard nhu "ms", "ip")
# Nhat quan trong toan he thong

# Dung:
timestamp, level, message, service, traceId, spanId
correlationId, requestId, userId, tenantId
httpMethod, httpPath, httpStatus, httpDurationMs
durationMs, dbQueryMs, isSuccess, isError
errorCode, errorMessage, errorType

# Sai:
TimeStamp, TIMESTAMP                     # PascalCase, UPPER
trace_id, span_id, correlation_id        # snake_case
traceid, spanid, userid                  # khong co camelCase
http_method, http_status, http_path      # snake_case
dur_ms, dms, reqMs                       # viet tat qua nhieu
```

---

## 12. Metrics

### Pattern va Quy Tac

```
# Quy tac: [namespace]_[component]_[measurement]_[unit]
# Tat ca lowercase, dau gach duoi _ phan cach
# Theo Prometheus convention

# HTTP metrics:
order_service_http_requests_total
order_service_http_request_duration_seconds
order_service_http_request_size_bytes
order_service_http_response_size_bytes

# Business metrics:
order_service_orders_created_total
order_service_orders_cancelled_total
order_service_order_processing_duration_seconds
payment_service_transactions_total
payment_service_transaction_amount_dollars
payment_service_payment_failures_total

# Infrastructure metrics:
user_service_active_sessions_count
user_service_cache_hits_total
user_service_cache_misses_total
database_connections_active_count
database_connections_idle_count
database_query_duration_seconds

# Queue metrics:
message_queue_messages_received_total
message_queue_messages_processed_total
message_queue_messages_failed_total
message_queue_processing_duration_seconds
message_queue_queue_depth_count

# Suffix conventions (Prometheus):
# _total: counter (always increasing)
# _count: gauge (can go up and down) hoac histogram count
# _sum: histogram sum
# _bucket: histogram bucket
# _seconds: duration in seconds
# _bytes: size in bytes
# _ratio: ratio 0..1
# _info: info metric

# Sai:
orderServiceHttpRequests         # camelCase, thieu unit
OrderService_HTTP_Requests       # mixed case
order-service-http-requests      # kebab-case (khong hop le trong Prometheus)
http_requests                    # thieu service namespace
```

### Labels

```
# Prometheus metric labels (camelCase cho custom, snake_case cung chap nhan)

# HTTP metrics labels:
http_request_duration_seconds{
  method="POST",
  path="/v1/orders",
  status="201",
  service="order-service"
}

# Business metrics labels:
order_service_orders_created_total{
  payment_method="credit_card",
  user_tier="premium",
  channel="web"
}

# Error metrics labels:
order_service_errors_total{
  error_type="ValidationError",
  component="OrderController",
  severity="warning"
}
```

---

## 13. Branch va Commit

### Branch Names

```
# Quy tac: [type]/[ticket-id]-[short-description]
# Tat ca lowercase, dau gach ngang phan cach

# Feature branches:
feature/PROJ-123-user-profile-page
feature/PROJ-456-order-tracking-api
feature/PROJ-789-payment-gateway-integration
feature/PROJ-101-product-search-elasticsearch
feature/PROJ-202-email-notification-system

# Bug fix branches:
fix/PROJ-321-auth-token-expiry-bug
fix/PROJ-654-order-total-calculation-error
fix/PROJ-987-cart-items-not-saved

# Hotfix (production urgent):
hotfix/PROJ-111-payment-null-pointer-exception
hotfix/PROJ-222-login-redirect-loop
hotfix/PROJ-333-critical-data-loss-fix

# Chore (maintenance, dependencies):
chore/update-dependencies
chore/upgrade-dotnet-8
chore/configure-eslint-rules
chore/PROJ-444-migrate-to-postgres-15

# Refactor:
refactor/PROJ-555-extract-payment-module
refactor/PROJ-666-clean-up-order-service
refactor/PROJ-777-apply-repository-pattern

# Documentation:
docs/PROJ-888-update-api-documentation
docs/add-readme-setup-guide

# Release branches:
release/1.2.0
release/2.0.0-rc.1

# Sai:
feature_user_profile              # snake_case
FeatureUserProfile                # PascalCase
feature-user-profile-page         # thieu ticket id (neu co ticket system)
PROJ-123-user-profile             # thieu type prefix
feature/UserProfilePage           # PascalCase trong description
```

### Git Tags

```
# Release tags - Semantic Versioning:
v1.0.0
v1.2.3
v2.0.0

# Release Candidates:
v1.2.3-rc.1
v1.2.3-rc.2
v2.0.0-rc.1

# Beta:
v1.2.3-beta.1
v2.0.0-beta.3

# Alpha:
v1.2.3-alpha.1

# Sai:
1.0.0              # thieu v prefix
V1.0.0             # V hoa
release-1.0.0      # prefix "release-" thay vi "v"
v1_0_0             # dau gach duoi
```

### Commit Message

```
# Quy tac: Conventional Commits
# [type]([scope]): [short description]
# [body - optional]
# [footer - optional]

# Types:
feat      - tinh nang moi
fix       - sua bug
docs      - chi thay doi tai lieu
style     - format, missing semicolons, etc (khong thay doi code logic)
refactor  - refactor code (khong phai feature, khong phai bug fix)
perf      - cai thien hieu nang
test      - them hoac sua tests
build     - thay doi build system hoac dependencies
ci        - thay doi CI/CD
chore     - cac thay doi khac (update .gitignore, etc)
revert    - revert commit truoc

# Scope: ten module/component bi anh huong

# Vi du dung:
feat(order): add order cancellation endpoint
fix(auth): resolve JWT token expiry calculation error
docs(api): update order API documentation
refactor(payment): extract payment validation to separate service
perf(product): add database index for category queries
test(order): add unit tests for order creation use case
chore: upgrade to .NET 8.0
build(docker): optimize multi-stage build for user-service

# Vi du sai:
Added order cancellation           # thieu type
FEAT: add order cancellation       # UPPER type
feat - add order cancellation      # dau gach ngang thay vi (:)
feat(order): Added order cancellation endpoint  # "Added" (past tense) thay vi "add"
update order service               # qua chung, khong co type
```

---

## 14. Bang Anti-Pattern Dat Ten

| Anti-Pattern | Vi du Sai | Vi du Dung | Ly Do |
|---|---|---|---|
| **Hungarian Notation** | `strUserName`, `intCount`, `bIsActive`, `lstOrders` | `userName`, `count`, `isActive`, `orders` | Kieu du lieu hien thi qua IDE, khong can trong ten. Khi kieu thay doi, ten sai. |
| **Viet tat qua muc** | `usrSvc`, `ordRepo`, `pmtGtw`, `cfg` | `userService`, `orderRepository`, `paymentGateway`, `config` | Viet tat lam giam kha nang doc. Hai nguoi doc khac nhau se hieu khac nhau. |
| **Ki tu don (ngoai loop)** | `u`, `o`, `p`, `s`, `x`, `y`, `z` | `user`, `order`, `product`, `status` | Khong co nghia ngay ca khi doc cung context. Kho tim kiem. |
| **Ten qua chung** | `data`, `info`, `result`, `temp`, `obj`, `val`, `stuff` | `userData`, `orderInfo`, `createdOrder`, `temporaryOrderId`, `orderObject`, `paymentValue` | Khong mo ta gi. "data" la data gi? result la result cua cai gi? |
| **Ten cong cu chung** | `utils`, `helper`, `manager`, `processor`, `misc` | `OrderFormatter`, `DateUtils`, `EmailValidator`, `PaymentProcessor` | Class "Utils" thu hut moi thu. Hay tach thanh class co nghia cu the. |
| **Kieu du lieu trong ten** | `userList`, `nameString`, `countInt`, `dataMap`, `orderArray` | `users`, `name`, `count`, `data`, `orders` | Kieu du lieu la implementation detail, co the thay doi. Ten nen mo ta nghia. |
| **Tieng Viet trong code** | `nguoiDung`, `donHang`, `sanPham`, `kiemTra`, `xacNhan` | `user`, `order`, `product`, `validate`, `confirm` | Khong nhat quan voi naming convention. Kho hop tac. Cong cu IDE/IDE khong ho tro tot. |
| **Plural cho single** | `users = getUserById(id)` | `user = getUserById(id)` | Ten plural am chi collection. Single item phai la singular. |
| **Bien the acronym khong ro** | `OTP`, `CTR`, `AOV`, `GMV`, `LTV` (trong code) | `oneTimePassword`, `clickThroughRate`, `averageOrderValue` hoac comment ro | Nguoi moi khong biet. Nen dung day du ten trong code, viet tat chi trong doc/comment. |
| **Prefix nhua** | `tbl_users`, `fn_getUser`, `sp_createOrder`, `v_activeUsers` | `users`, `get_user`, `create_order`, `active_users_view` | Prefix nhua la artifact cua SQL Server cu. Hien dai khong dung. |
| **Ten phan anh implementation** | `arrayBasedOrderList`, `linkedListQueue`, `hashMapCache` | `orders`, `orderQueue`, `cache` | Implementation co the thay doi. Giao dien (interface) quan trong hon. |
| **Trong hop tran nghia** | `order.ProcessAndValidateAndSaveAndNotify()` | Tach thanh: `validate()`, `process()`, `save()`, `notify()` | "And" trong ten method nghia la method dang lam qua nhieu viec (vi pham SRP). |
| **Ten phu dinh** | `isNotActive`, `hasNoPermission`, `isNotDeleted`, `cannotEdit` | `isInactive`, `lacksPermission`, `exists`, `isReadOnly` | Ten phu dinh kho doc trong dieu kien if/else. `if (!isNotActive)` - rat kho. |
| **Ten qua dai** | `theCurrentlyLoggedInUserWhoMadeThisRequest` | `currentUser`, `requestingUser` | Qua 30 ki tu la dau hieu qua dai. |
| **Ten xau doi so voi context** | `order1`, `order2`, `user_new`, `user_old` | `originalOrder`, `updatedOrder`, `existingUser`, `newUser` | So va suffix "new"/"old" khong mo ta quan he. |
| **Magic number** | `if (status == 2)`, `if (retryCount > 3)` | `if (status == OrderStatus.CONFIRMED)`, `if (retryCount > MAX_RETRY_COUNT)` | So ma thuat khong co nghia. Phai dung ten hang so. |
| **Boolean Trap** | `createUser(true, false, true)` | `createUser(isAdmin: true, sendEmail: false, isActive: true)` | tham so boolean khong co ten gay nham lan. Dung named parameters hoac config object. |
| **Inconsistent Language Mix** | `getUserDonHang()`, `taoOrder()`, `kiemTraUser()` | `getUserOrders()`, `createOrder()`, `validateUser()` | Pha tron tieng Viet va tieng Anh trong code gay nham lan, kho doc. |
| **"God" class names** | `OrderManager`, `SystemController`, `DataProcessor`, `BaseHelper` | `OrderService`, `OrderController`, `PaymentProcessor`, `ValidationHelper` | "Manager", "Controller" (tru MVC), "Processor", "Helper" co the la dau hieu class dang lam qua nhieu. |
| **Numbering** | `UserService1`, `OrderServiceV2`, `NewUserController` | `UserService`, `OrderServiceV2` (chi khi co versioning that su) | So trong ten class thuong la dau hieu cua refactoring chua hoan chinh. |
| **Copy-paste naming** | `CopyOfOrderService`, `OrderServiceFinal`, `OrderServiceNew` | Dung version control, khong dat ten "Final" hay "Copy" | "Final" bao gio cung co "FinalV2". Dung Git de track changes. |
| **Abbreviation inconsistency** | Mix `Svc` va `Service`, mix `Repo` va `Repository`, mix `Ctrl` va `Controller` | Chon mot va nhat quan: `Service`, `Repository`, `Controller` | Khong nhat quan gay nham lan khi search. |

---

## Tong Ket - Nguyen Tac Vang

> Nhung nguyen tac nay vuot qua moi ngon ngu va framework.

### 1. Ro Rang Tren Ngan Gon
```
# Ten phai tra loi cau hoi "cai nay la gi / lam gi?"
# Ngan gon tot nhung ro rang tren het

# DUNG:
getUserByEmail()          # ro rang: lay user theo email
isEmailVerified           # ro rang: co verified chua?
maxRetryCount             # ro rang: so lan retry toi da

# SAI:
get()                     # qua ngan, khong ro
flag                      # qua chung
n                         # ki tu don ngoai loop
```

### 2. Nhat Quan Trong Toan Project
```
# Chon MOT convention va giu nguyen trong toan project
# Khong pha tron:
# - Repo va Repository
# - Svc va Service
# - Ctrl va Controller
# - camelCase va snake_case cho cung loai identifier
```

### 3. Ten Mo Ta Y Dinh, Khong Mo Ta Implementation
```
# Y dinh (DUNG):
users                 # collection cua User
emailSender           # vat co kha nang gui email
orderQueue            # hang doi cho order

# Implementation (SAI):
arrayOfUsers          # mo ta structure
SmtpEmailService      # mo ta protocol (neu co the thay doi)
linkedListQueue       # mo ta data structure
```

### 4. Tranh Viet Tat Tru Cac Truong Hop Pho Bien
```
# Viet tat chap nhan duoc (qua pho bien de hieu):
id, url, api, http, sql, db, ui, io, os, dto, dto

# Khong nen viet tat:
usr (-> user), msg (-> message), cnt (-> count)
btn (-> button), img (-> image), str (-> string)
```

### 5. Dong Tu Cho Methods / Hanh Dong, Danh Tu Cho Classes / Objects
```
# Method: bat dau bang dong tu
getUser(), createOrder(), validatePayment(), isActive()

# Class/Interface: danh tu
UserService, Order, PaymentGateway, OrderRepository
```

### 6. Follow Language / Framework Convention
```
# Khong dung C# convention trong Python
# Khong dung Java convention trong Go
# Doc va follow cac style guide chinh thuc:
# - C#: Microsoft C# Coding Conventions
# - Java: Google Java Style Guide
# - Python: PEP 8
# - Go: Effective Go + Google Go Style
# - TypeScript: TypeScript Handbook
```

---

*Tai lieu nay duoc duy tri boi ky su backend. Moi de xuat thay doi phai duoc review qua pull request va dong thuan boi team.*

*Phien ban: 1.0 | Cap nhat: 2026-06-23*
