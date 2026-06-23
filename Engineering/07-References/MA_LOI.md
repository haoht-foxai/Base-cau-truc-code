# DANH SÁCH MÃ LỖI HỆ THỐNG

> **Phiên bản:** 1.0.0
> **Cập nhật lần cuối:** 2026-06-23
> **Chủ sở hữu:** Team Backend / Architecture
> **Trạng thái:** CHÍNH THỨC - Bắt buộc tuân thủ

---

## Mục Lục

1. [Quy ước mã lỗi](#quy-ươc-mã-lỗi)
2. [Cách sử dụng](#cách-sử-dụng)
3. [Bảng mã lỗi đầy đủ](#bảng-mã-lỗi-đầy-đủ)
   - [AUTH - Xác Thực](#-auth---xác-thực-auth_001--auth_099)
   - [ACCESS - Phân Quyền](#-access---phân-quyền-access_001--access_099)
   - [USER - Người Dùng](#-user---người-dùng-user_001--user_099)
   - [VAL - Validation](#-val---validation-val_001--val_099)
   - [RES - Tài Nguyên](#-res---tài-nguyên-res_001--res_099)
   - [ORDER - Đơn Hàng](#-order---đơn-hàng-order_001--order_099)
   - [PAY - Thanh Toán](#-pay---thanh-toán-pay_001--pay_099)
   - [FILE - File/Upload](#-file---fileupload-file_001--file_099)
   - [RATE - Giới Hạn Tốc Độ](#-rate---giới-hạn-tốc-độ-rate_001--rate_009)
   - [SYS - Hệ Thống](#-sys---hệ-thống-sys_001--sys_099)
   - [INT - Tích Hợp Bên Ngoài](#-int---tích-hợp-bên-ngoài-int_001--int_099)
4. [Quy trình thêm mã lỗi mới](#quy-trình-thêm-mã-lỗi-mới)
5. [Mapping HTTP Status → Domain](#mapping-http-status--domain)
6. [Bảng tra cứu nhanh](#bảng-tra-cứu-nhanh)

---

## Quy ước mã lỗi

### Format chuẩn

```
[DOMAIN]_[NNN]
```

| Thành phần | Mô tả | Ví dụ |
|------------|-------|-------|
| `DOMAIN` | Tên miền nghiệp vụ, viết HOA, tối đa 6 ký tự | `AUTH`, `PAY`, `USER` |
| `_` | Ký tự phân cách bắt buộc | `_` |
| `NNN` | Số thứ tự 3 chữ số, bắt đầu từ `001` | `001`, `042`, `099` |

### Nguyên tắc đặt tên

- **Duy nhất:** Mỗi mã lỗi chỉ được xuất hiện MỘT LẦN trong toàn bộ hệ thống.
- **Bất biến:** Sau khi đã phát hành, mã lỗi KHÔNG được đổi tên hoặc tái sử dụng.
- **Tuần tự:** Số thứ tự tăng dần, không được bỏ khoảng trống khi đặt mới.
- **Phân vùng:** Mỗi domain chiếm 1 phân vùng số riêng biệt (`001-099`).
- **Mỗi domain tối đa 99 mã lỗi** trong phân vùng hiện tại.

### Danh sách Domain hiện có

| Domain | Phạm vi | Mô tả |
|--------|---------|-------|
| `AUTH` | AUTH_001 - AUTH_099 | Xác thực người dùng, token, session |
| `ACCESS` | ACCESS_001 - ACCESS_099 | Phân quyền, kiểm soát truy cập |
| `USER` | USER_001 - USER_099 | Quản lý tài khoản, hồ sơ người dùng |
| `VAL` | VAL_001 - VAL_099 | Kiểm tra dữ liệu đầu vào (Validation) |
| `RES` | RES_001 - RES_099 | Tài nguyên chung (Resource) |
| `ORDER` | ORDER_001 - ORDER_099 | Đơn hàng, đặt hàng |
| `PAY` | PAY_001 - PAY_099 | Thanh toán, giao dịch tài chính |
| `FILE` | FILE_001 - FILE_099 | Tải lên, quản lý tập tin |
| `RATE` | RATE_001 - RATE_009 | Giới hạn tốc độ (Rate Limiting) |
| `SYS` | SYS_001 - SYS_099 | Lỗi hệ thống nội bộ |
| `INT` | INT_001 - INT_099 | Tích hợp dịch vụ bên ngoài (Integration) |

---

## Cách sử dụng

### 1. Cấu trúc phản hồi lỗi chuẩn (API Response)

Mọi API endpoint khi xảy ra lỗi đều phải trả về cấu trúc JSON thống nhất sau:

```json
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
```

| Trường | Kiểu | Bắt buộc | Mô tả |
|--------|------|----------|-------|
| `success` | boolean | Có | Luôn là `false` khi có lỗi |
| `error.code` | string | Có | Mã lỗi theo quy ước `[DOMAIN]_[NNN]` |
| `error.message` | string | Có | Thông báo lỗi ngắn gọn, thân thiện với người dùng |
| `error.details` | object/array/null | Không | Chi tiết bổ sung, thường dùng cho lỗi validation |
| `error.timestamp` | string (ISO 8601) | Có | Thời điểm xảy ra lỗi |
| `error.traceId` | string | Có | ID để tra cứu log trên hệ thống giám sát |

### 2. Ví dụ lỗi Validation với nhiều trường

```json
{
  "success": false,
  "error": {
    "code": "VAL_001",
    "message": "Dữ liệu đầu vào không hợp lệ. Vui lòng kiểm tra lại các trường bị lỗi.",
    "details": [
      {
        "field": "email",
        "code": "VAL_005",
        "message": "Email không đúng định dạng"
      },
      {
        "field": "phone",
        "code": "VAL_006",
        "message": "Số điện thoại không hợp lệ"
      }
    ],
    "timestamp": "2026-06-23T10:30:00Z",
    "traceId": "xyz789"
  }
}
```

### 3. Cách xử lý mã lỗi ở Client (Frontend / Mobile)

Khi nhận được phản hồi lỗi từ API, phía client cần:

**Bước 1 - Đọc HTTP Status Code:**
- `4xx` - Lỗi do phía client (dữ liệu sai, thiếu quyền, v.v.)
- `5xx` - Lỗi phía server (thông báo chung chung, không lộ chi tiết kỹ thuật)

**Bước 2 - Đọc `error.code` để xử lý nghiệp vụ:**
- `AUTH_002` hoặc `AUTH_008` → Tự động refresh token, nếu thất bại → redirect đến trang đăng nhập.
- `ACCESS_001` → Hiển thị thông báo "Bạn không có quyền thực hiện thao tác này".
- `VAL_*` → Hiển thị lỗi inline ngay trên form, bên cạnh từng trường bị lỗi.
- `RATE_001` → Đọc header `Retry-After`, hiển thị countdown và vô hiệu hóa nút.
- `SYS_001` → Hiển thị trang lỗi chung "Hệ thống đang gặp sự cố, vui lòng thử lại sau".

**Bước 3 - Xử lý Retry:**
- Chỉ retry khi cột "Có Retry?" là "Có" và lỗi là tạm thời.
- Dùng thuật toán Exponential Backoff: chờ `2^n * 100ms` giữa các lần retry (n = số lần thất bại).
- Tối đa 3 lần retry, sau đó hiển thị thông báo lỗi cho người dùng.

### 4. Cách sử dụng mã lỗi ở Backend (Server)

```typescript
// Ví dụ TypeScript - Ném ngoại lệ với mã lỗi chuẩn
import { AppException } from '@common/exceptions';
import { ErrorCode } from '@common/error-codes';

// Khi token hết hạn
throw new AppException(ErrorCode.AUTH_002, 401);

// Khi dữ liệu không hợp lệ
throw new AppException(ErrorCode.VAL_001, 400, [
  { field: 'email', code: ErrorCode.VAL_005, message: 'Email không đúng định dạng' }
]);

// Khi không tìm thấy tài nguyên
throw new AppException(ErrorCode.RES_001, 404);
```

### 5. Logging và Monitoring

- Mọi lỗi `5xx` phải được log ở mức `ERROR` với đầy đủ stack trace và `traceId`.
- Các lỗi `4xx` log ở mức `WARN` (trừ `VAL_*` log ở mức `DEBUG`).
- Dashboard giám sát phải theo dõi tần suất xuất hiện của từng mã lỗi.
- Alert tự động khi tỷ lệ lỗi `SYS_001` vượt ngưỡng 1% trong vòng 5 phút.

---

## Bảng mã lỗi đầy đủ

---

### 🔐 AUTH - Xác Thực (AUTH_001 → AUTH_099)

**Phạm vi:** Mọi vấn đề liên quan đến xác thực danh tính người dùng, bao gồm đăng nhập, token, session, API key và xác thực hai bước.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `AUTH_001` | 401 | Thông tin đăng nhập không hợp lệ | Sai tên đăng nhập hoặc mật khẩu | Yêu cầu người dùng kiểm tra lại thông tin đăng nhập | Không |
| `AUTH_002` | 401 | Token truy cập đã hết hạn | Access token vượt quá thời gian sống (TTL) | Tự động gọi API refresh token, nếu thất bại → đăng nhập lại | Có (1 lần) |
| `AUTH_003` | 401 | Token truy cập không hợp lệ | Token bị giả mạo, sai chữ ký, hoặc sai định dạng | Xóa token cũ, yêu cầu đăng nhập lại | Không |
| `AUTH_004` | 401 | Thiếu token xác thực | Request không gửi kèm Authorization header | Thêm token vào header `Authorization: Bearer <token>` | Không |
| `AUTH_005` | 403 | Tài khoản bị khóa tạm thời | Quản trị viên khóa tài khoản do vi phạm chính sách | Thông báo tài khoản bị khóa, hướng dẫn liên hệ hỗ trợ | Không |
| `AUTH_006` | 403 | Tài khoản chưa xác minh email | Người dùng chưa nhấp vào link xác minh email sau khi đăng ký | Hiển thị màn hình xác minh email, cung cấp nút gửi lại email | Không |
| `AUTH_007` | 403 | Yêu cầu xác thực hai bước (2FA) | Tài khoản bật 2FA nhưng chưa cung cấp mã OTP | Chuyển hướng đến màn hình nhập mã 2FA | Không |
| `AUTH_008` | 401 | Phiên đăng nhập đã hết hạn | Session trên server đã bị xóa do không hoạt động quá lâu | Yêu cầu đăng nhập lại, có thể lưu lại trang hiện tại để redirect sau | Không |
| `AUTH_009` | 401 | Refresh token không hợp lệ | Refresh token sai định dạng hoặc đã bị thu hồi | Xóa toàn bộ token lưu cục bộ, yêu cầu đăng nhập lại | Không |
| `AUTH_010` | 401 | Refresh token đã hết hạn | Refresh token vượt quá thời gian sống (thường 30-90 ngày) | Xóa token, yêu cầu đăng nhập lại, thông báo phiên đã hết hạn | Không |
| `AUTH_011` | 403 | Đăng nhập từ thiết bị không được phép | Thiết bị chưa được đưa vào danh sách tin cậy của tài khoản | Hướng dẫn xác nhận thiết bị qua email hoặc SMS | Không |
| `AUTH_012` | 429 | Quá nhiều lần đăng nhập thất bại | Nhập sai mật khẩu nhiều lần, vượt ngưỡng cho phép (thường 5 lần) | Hiển thị countdown, vô hiệu hóa nút đăng nhập, gợi ý quên mật khẩu | Không |
| `AUTH_013` | 401 | API key không hợp lệ | API key sai, không tồn tại trong hệ thống | Kiểm tra lại API key, đảm bảo không có ký tự thừa | Không |
| `AUTH_014` | 401 | API key đã hết hạn | API key vượt quá ngày hết hạn được đặt khi tạo | Tạo API key mới trong trang quản lý tài khoản | Không |
| `AUTH_015` | 403 | Tài khoản đã bị vô hiệu hóa vĩnh viễn | Tài khoản vi phạm nghiêm trọng điều khoản sử dụng | Thông báo tài khoản đã bị vô hiệu hóa, hướng dẫn kháng cáo | Không |
| `AUTH_016` | 403 | Mã OTP không đúng | Người dùng nhập sai mã xác thực một lần | Yêu cầu nhập lại mã OTP, hiển thị số lần còn lại | Không |
| `AUTH_017` | 403 | Mã OTP đã hết hạn | Mã OTP chỉ có hiệu lực trong vòng 5 phút | Yêu cầu người dùng gửi lại mã OTP mới | Không |
| `AUTH_018` | 429 | Gửi OTP quá nhiều lần | Yêu cầu gửi OTP vượt giới hạn trong khoảng thời gian ngắn | Hiển thị thời gian chờ trước khi gửi lại, thường 60 giây | Không |
| `AUTH_019` | 401 | Token đăng nhập một lần (Magic Link) đã được sử dụng | Magic link chỉ dùng được một lần | Yêu cầu gửi magic link mới | Không |
| `AUTH_020` | 401 | Token đặt lại mật khẩu không hợp lệ hoặc đã hết hạn | Link đặt lại mật khẩu đã cũ hoặc đã được sử dụng | Yêu cầu gửi lại email đặt lại mật khẩu | Không |

---

### 🚫 ACCESS - Phân Quyền (ACCESS_001 → ACCESS_099)

**Phạm vi:** Mọi vấn đề liên quan đến kiểm soát truy cập, phân quyền theo vai trò (RBAC), giới hạn theo gói dịch vụ và chính sách bảo mật.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `ACCESS_001` | 403 | Không có quyền truy cập tài nguyên | Người dùng không có role hoặc permission cần thiết | Hiển thị thông báo "Bạn không có quyền thực hiện thao tác này" | Không |
| `ACCESS_002` | 403 | Không phải chủ sở hữu tài nguyên | Cố gắng sửa/xóa tài nguyên của người dùng khác | Hiển thị lỗi, không cho phép thao tác | Không |
| `ACCESS_003` | 403 | Địa chỉ IP không được phép | IP của client nằm trong danh sách đen hoặc không có trong whitelist | Thông báo IP bị từ chối, hướng dẫn liên hệ quản trị viên | Không |
| `ACCESS_004` | 403 | Hành động bị từ chối theo chính sách bảo mật | Hành động vi phạm chính sách (policy) được cấu hình trong hệ thống | Hiển thị lý do từ chối nếu có thể, hướng dẫn liên hệ hỗ trợ | Không |
| `ACCESS_005` | 403 | Vượt giới hạn tài nguyên của gói dịch vụ | Người dùng gói miễn phí cố gắng sử dụng tính năng của gói trả phí | Hiển thị upsell popup, hướng dẫn nâng cấp gói | Không |
| `ACCESS_006` | 403 | Tài nguyên chỉ dành cho quản trị viên | Endpoint chỉ dành cho admin nhưng user thường đang gọi | Redirect về trang chủ hoặc thông báo lỗi 403 | Không |
| `ACCESS_007` | 403 | Truy cập bị giới hạn theo vùng địa lý | Dịch vụ chưa hoạt động tại quốc gia/khu vực của người dùng | Thông báo dịch vụ chưa có tại khu vực, hướng dẫn sử dụng VPN (nếu phù hợp) | Không |
| `ACCESS_008` | 403 | Tenant không có quyền thực hiện hành động | Trong hệ thống multi-tenant, tenant không được cấp quyền cụ thể | Liên hệ quản trị viên của tổ chức để được cấp quyền | Không |
| `ACCESS_009` | 403 | Tính năng đang trong giai đoạn thử nghiệm (Beta) | Tính năng chỉ mở cho nhóm người dùng được chọn (beta tester) | Thông báo tính năng chưa khả dụng, cung cấp form đăng ký tham gia beta | Không |
| `ACCESS_010` | 403 | Vượt quá số lượng thiết bị đăng nhập đồng thời | Gói dịch vụ chỉ cho phép đăng nhập trên N thiết bị cùng lúc | Hiển thị danh sách thiết bị đang đăng nhập, yêu cầu đăng xuất thiết bị khác | Không |
| `ACCESS_011` | 403 | Tài nguyên bị ẩn hoặc không công khai | Tài nguyên tồn tại nhưng chủ sở hữu đã đặt ở chế độ riêng tư | Thông báo "Tài nguyên không tồn tại hoặc không được phép truy cập" | Không |
| `ACCESS_012` | 403 | Hết hạn gói dịch vụ | Gói subscription của người dùng đã hết hạn | Hiển thị thông báo gia hạn, redirect đến trang thanh toán | Không |

---

### 👤 USER - Người Dùng (USER_001 → USER_099)

**Phạm vi:** Mọi vấn đề liên quan đến quản lý tài khoản người dùng, đăng ký, hồ sơ cá nhân và cài đặt tài khoản.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `USER_001` | 404 | Người dùng không tồn tại | Tìm kiếm user theo ID hoặc email không có kết quả | Thông báo không tìm thấy người dùng | Không |
| `USER_002` | 409 | Email đã được sử dụng | Đăng ký với email đã có trong hệ thống | Gợi ý đăng nhập hoặc quên mật khẩu với email đó | Không |
| `USER_003` | 409 | Tên đăng nhập (username) đã tồn tại | Username đã có người đăng ký | Gợi ý các username thay thế | Không |
| `USER_004` | 409 | Số điện thoại đã được sử dụng | Số điện thoại đã liên kết với tài khoản khác | Thông báo và hướng dẫn liên hệ hỗ trợ nếu không phải chủ sở hữu | Không |
| `USER_005` | 400 | Mật khẩu quá yếu | Mật khẩu không đáp ứng yêu cầu độ mạnh tối thiểu | Hiển thị yêu cầu mật khẩu và indicator độ mạnh | Không |
| `USER_006` | 400 | Mật khẩu xác nhận không khớp | Trường `password` và `confirm_password` khác nhau | Yêu cầu nhập lại mật khẩu xác nhận | Không |
| `USER_007` | 400 | Mật khẩu cũ không đúng | Người dùng nhập sai mật khẩu hiện tại khi đổi mật khẩu | Yêu cầu nhập lại mật khẩu cũ, cung cấp link quên mật khẩu | Không |
| `USER_008` | 400 | Mật khẩu mới trùng với mật khẩu cũ | Người dùng đặt lại mật khẩu giống mật khẩu hiện tại | Yêu cầu chọn mật khẩu khác với mật khẩu hiện tại | Không |
| `USER_009` | 400 | Ảnh đại diện không hợp lệ | File upload không phải ảnh hoặc kích thước quá lớn | Thông báo yêu cầu định dạng và kích thước ảnh hợp lệ | Không |
| `USER_010` | 403 | Không thể xóa tài khoản do ràng buộc nghiệp vụ | Tài khoản có đơn hàng đang xử lý hoặc số dư ví chưa rút | Thông báo lý do cụ thể và hướng dẫn giải quyết trước khi xóa | Không |
| `USER_011` | 400 | Định dạng ngày sinh không hợp lệ | Ngày sinh không đúng định dạng hoặc không hợp lý | Yêu cầu nhập lại ngày sinh theo định dạng quy định | Không |
| `USER_012` | 400 | Tuổi không đủ điều kiện sử dụng dịch vụ | Người dùng dưới độ tuổi tối thiểu (thường 18 tuổi) | Thông báo yêu cầu độ tuổi, không cho phép tiếp tục | Không |
| `USER_013` | 404 | Hồ sơ người dùng chưa được tạo | Tài khoản tồn tại nhưng chưa hoàn thiện hồ sơ | Redirect đến trang hoàn thiện hồ sơ | Không |
| `USER_014` | 409 | Tài khoản mạng xã hội đã liên kết với người dùng khác | OAuth account (Google/Facebook) đã bind với tài khoản khác | Thông báo và hướng dẫn hủy liên kết từ tài khoản cũ | Không |
| `USER_015` | 400 | Không thể hủy liên kết phương thức đăng nhập duy nhất | Người dùng cố hủy link OAuth khi đó là phương thức đăng nhập duy nhất | Yêu cầu đặt mật khẩu trước khi hủy liên kết mạng xã hội | Không |

---

### ✅ VAL - Validation (VAL_001 → VAL_099)

**Phạm vi:** Mọi lỗi kiểm tra tính hợp lệ của dữ liệu đầu vào từ client. Đây là nhóm lỗi phổ biến nhất và thường đi kèm với `details` array.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `VAL_001` | 400 | Dữ liệu đầu vào không hợp lệ (lỗi tổng hợp) | Nhiều trường cùng bị lỗi validation | Xử lý mảng `details` để hiển thị lỗi từng trường | Không |
| `VAL_002` | 400 | Trường bắt buộc bị thiếu | Field `required` không được gửi trong request body | Hiển thị thông báo "Trường này là bắt buộc" bên cạnh field | Không |
| `VAL_003` | 400 | Giá trị vượt độ dài tối đa | Chuỗi ký tự vượt quá `maxLength` cho phép | Hiển thị giới hạn ký tự và số ký tự hiện tại | Không |
| `VAL_004` | 400 | Giá trị dưới độ dài tối thiểu | Chuỗi ký tự không đủ `minLength` yêu cầu | Hiển thị yêu cầu độ dài tối thiểu | Không |
| `VAL_005` | 400 | Email không đúng định dạng | Email thiếu `@`, thiếu domain, hoặc sai cú pháp | Hiển thị gợi ý định dạng email hợp lệ | Không |
| `VAL_006` | 400 | Số điện thoại không hợp lệ | Số điện thoại sai định dạng hoặc không đúng quốc gia | Hiển thị ví dụ định dạng số điện thoại hợp lệ | Không |
| `VAL_007` | 400 | Định dạng ngày giờ không hợp lệ | Ngày tháng không theo chuẩn ISO 8601 hoặc không tồn tại | Hiển thị định dạng ngày giờ yêu cầu, ví dụ `YYYY-MM-DD` | Không |
| `VAL_008` | 400 | Giá trị nằm ngoài khoảng cho phép | Số vượt `min` hoặc `max` được quy định | Hiển thị khoảng giá trị hợp lệ | Không |
| `VAL_009` | 400 | Giá trị không thuộc tập hợp cho phép (Enum) | Gửi giá trị không nằm trong danh sách enum hợp lệ | Hiển thị danh sách các giá trị được chấp nhận | Không |
| `VAL_010` | 400 | Giá trị bị trùng lặp | Cố gắng thêm phần tử đã tồn tại trong danh sách unique | Thông báo giá trị đã tồn tại, không cho phép thêm | Không |
| `VAL_011` | 400 | Định dạng URL không hợp lệ | URL thiếu scheme (`http://`), sai cú pháp | Hiển thị ví dụ URL hợp lệ | Không |
| `VAL_012` | 400 | Giá trị không phải số nguyên hợp lệ | Gửi chuỗi ký tự khi field yêu cầu số nguyên | Thông báo trường phải là số nguyên | Không |
| `VAL_013` | 400 | Giá trị không phải boolean hợp lệ | Gửi chuỗi "yes"/"no" thay vì `true`/`false` | Thông báo trường phải là `true` hoặc `false` | Không |
| `VAL_014` | 400 | Mảng rỗng không được phép | Gửi mảng `[]` khi field yêu cầu ít nhất 1 phần tử | Thông báo phải chọn ít nhất một giá trị | Không |
| `VAL_015` | 400 | Vượt số lượng phần tử tối đa trong mảng | Mảng có nhiều phần tử hơn giới hạn cho phép | Thông báo số lượng phần tử tối đa được phép | Không |
| `VAL_016` | 400 | Mã bưu chính (ZIP/Postal Code) không hợp lệ | Mã bưu chính sai định dạng hoặc không tồn tại | Yêu cầu nhập lại mã bưu chính đúng định dạng | Không |
| `VAL_017` | 400 | Số thẻ ngân hàng/thẻ tín dụng không hợp lệ | Số thẻ không qua kiểm tra Luhn algorithm | Yêu cầu kiểm tra lại số thẻ | Không |
| `VAL_018` | 400 | Mã màu không hợp lệ | Mã màu không theo định dạng HEX (`#RRGGBB`) | Thông báo định dạng màu hợp lệ, ví dụ `#FF5733` | Không |
| `VAL_019` | 400 | Tọa độ địa lý (latitude/longitude) không hợp lệ | Latitude ngoài khoảng [-90, 90] hoặc longitude ngoài [-180, 180] | Thông báo khoảng giá trị hợp lệ cho tọa độ | Không |
| `VAL_020` | 400 | Nội dung chứa ký tự không được phép | Input chứa ký tự đặc biệt, mã script, hoặc HTML nguy hiểm | Thông báo ký tự không được phép, gợi ý nhập lại | Không |

---

### 📦 RES - Tài Nguyên (RES_001 → RES_099)

**Phạm vi:** Mọi vấn đề liên quan đến tài nguyên chung trong hệ thống: không tìm thấy, xung đột, bị khóa, đã xóa và kiểm soát phiên bản.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `RES_001` | 404 | Tài nguyên không tồn tại | ID hoặc slug được cung cấp không có trong cơ sở dữ liệu | Hiển thị trang 404, cung cấp link quay lại hoặc tìm kiếm | Không |
| `RES_002` | 409 | Tài nguyên đã tồn tại | Cố gắng tạo tài nguyên với định danh unique đã có | Thông báo đã tồn tại, gợi ý xem/chỉnh sửa tài nguyên đó | Không |
| `RES_003` | 409 | Xung đột dữ liệu (Conflict) | Hai thao tác cùng lúc cố sửa đổi cùng một tài nguyên | Tải lại dữ liệu mới nhất, yêu cầu người dùng thực hiện lại thao tác | Có |
| `RES_004` | 423 | Tài nguyên đang bị khóa | Tài nguyên đang được xử lý bởi tiến trình khác | Thông báo "Đang xử lý, vui lòng thử lại sau", retry sau 5 giây | Có |
| `RES_005` | 410 | Tài nguyên đã bị xóa vĩnh viễn | Tài nguyên đã bị xóa và không thể khôi phục | Thông báo tài nguyên không còn tồn tại, cập nhật UI | Không |
| `RES_006` | 409 | Xung đột phiên bản (Version Mismatch) | Client gửi `version` cũ, tài nguyên đã được cập nhật bởi người khác | Tải lại phiên bản mới nhất, hiển thị diff để người dùng quyết định | Không |
| `RES_007` | 400 | Không thể xóa tài nguyên do phụ thuộc | Tài nguyên còn được tham chiếu bởi các tài nguyên con | Hiển thị danh sách tài nguyên phụ thuộc, yêu cầu xóa chúng trước | Không |
| `RES_008` | 400 | Thao tác không được phép ở trạng thái hiện tại | Tài nguyên không hỗ trợ hành động này ở trạng thái (state) hiện tại | Thông báo trạng thái hiện tại và các hành động khả dụng | Không |
| `RES_009` | 400 | ID tài nguyên không đúng định dạng | ID được truyền vào không phải UUID hoặc sai format | Thông báo định dạng ID không hợp lệ | Không |
| `RES_010` | 403 | Tài nguyên bị lưu trữ (Archived) | Tài nguyên đã được lưu trữ, không cho phép chỉnh sửa | Thông báo tài nguyên đã lưu trữ, cung cấp tùy chọn khôi phục | Không |
| `RES_011` | 404 | Tài nguyên bị xóa mềm (Soft Deleted) | Tài nguyên đã bị xóa mềm (is_deleted = true) | Thông báo tài nguyên đã bị xóa, cung cấp tùy chọn khôi phục nếu có quyền | Không |
| `RES_012` | 400 | Danh sách tài nguyên rỗng | Batch operation được gọi với danh sách ID rỗng | Yêu cầu chọn ít nhất một tài nguyên trước khi thực hiện | Không |

---

### 🛒 ORDER - Đơn Hàng (ORDER_001 → ORDER_099)

**Phạm vi:** Mọi vấn đề liên quan đến vòng đời đơn hàng, từ tạo mới, xác nhận, vận chuyển đến hủy và hoàn trả.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `ORDER_001` | 404 | Đơn hàng không tồn tại | Mã đơn hàng không hợp lệ hoặc không thuộc tài khoản này | Kiểm tra lại mã đơn hàng, hiển thị lỗi | Không |
| `ORDER_002` | 400 | Giỏ hàng trống | Cố gắng đặt hàng khi giỏ hàng không có sản phẩm | Thông báo giỏ hàng trống, redirect đến trang sản phẩm | Không |
| `ORDER_003` | 400 | Sản phẩm không còn trong kho | Sản phẩm hết hàng tại thời điểm đặt hàng | Thông báo sản phẩm hết hàng, gợi ý sản phẩm tương tự | Không |
| `ORDER_004` | 400 | Số lượng yêu cầu vượt quá tồn kho | Số lượng đặt hàng lớn hơn số lượng còn trong kho | Thông báo số lượng còn lại, điều chỉnh số lượng tự động | Không |
| `ORDER_005` | 400 | Không thể hủy đơn hàng ở trạng thái này | Đơn hàng đã được giao hoặc đang vận chuyển, không thể hủy | Thông báo trạng thái đơn hàng, hướng dẫn yêu cầu hoàn trả thay thế | Không |
| `ORDER_006` | 400 | Mã giảm giá không hợp lệ | Mã coupon/voucher sai hoặc không tồn tại | Yêu cầu kiểm tra lại mã giảm giá | Không |
| `ORDER_007` | 400 | Mã giảm giá đã hết hạn | Coupon đã qua ngày hết hạn | Thông báo mã đã hết hạn, hiển thị các mã khả dụng khác | Không |
| `ORDER_008` | 400 | Mã giảm giá đã được sử dụng | Coupon dùng một lần đã được dùng bởi tài khoản này | Thông báo mã đã dùng, không cho phép áp dụng lại | Không |
| `ORDER_009` | 400 | Đơn hàng không đủ điều kiện áp dụng mã giảm giá | Giá trị đơn hàng dưới ngưỡng tối thiểu của coupon | Thông báo điều kiện áp dụng và số tiền còn thiếu | Không |
| `ORDER_010` | 400 | Địa chỉ giao hàng không hợp lệ | Địa chỉ không đầy đủ hoặc không thuộc vùng giao hàng | Yêu cầu nhập lại địa chỉ đầy đủ hoặc chọn vùng hỗ trợ | Không |
| `ORDER_011` | 400 | Phương thức vận chuyển không khả dụng | Phương thức giao hàng không hỗ trợ địa chỉ được chọn | Hiển thị các phương thức vận chuyển khả dụng | Không |
| `ORDER_012` | 409 | Đơn hàng đang được xử lý | Đơn hàng trùng lặp trong khoảng thời gian ngắn (idempotency) | Thông báo đơn hàng đang được xử lý, không cho phép đặt lại | Có (sau 5s) |
| `ORDER_013` | 400 | Sản phẩm đã bị gỡ khỏi hệ thống | Sản phẩm trong giỏ hàng đã bị xóa hoặc tạm ngưng bán | Thông báo sản phẩm không còn khả dụng, xóa khỏi giỏ hàng | Không |
| `ORDER_014` | 400 | Yêu cầu hoàn trả vượt quá thời hạn | Yêu cầu refund/return sau thời gian cho phép (thường 7-30 ngày) | Thông báo đã quá thời hạn hoàn trả, hướng dẫn liên hệ CSKH | Không |
| `ORDER_015` | 400 | Số lượng hoàn trả vượt quá số lượng đã mua | Yêu cầu hoàn trả số lượng nhiều hơn số lượng trong đơn hàng gốc | Thông báo số lượng tối đa có thể hoàn trả | Không |

---

### 💳 PAY - Thanh Toán (PAY_001 → PAY_099)

**Phạm vi:** Mọi vấn đề liên quan đến giao dịch thanh toán, thẻ tín dụng, ví điện tử và hoàn tiền.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `PAY_001` | 402 | Số dư không đủ | Số dư ví hoặc tài khoản ngân hàng không đủ | Thông báo số dư không đủ, hướng dẫn nạp thêm tiền | Không |
| `PAY_002` | 402 | Thẻ bị từ chối | Ngân hàng phát hành từ chối giao dịch (decline) | Thông báo thẻ bị từ chối, đề xuất thử thẻ khác hoặc liên hệ ngân hàng | Không |
| `PAY_003` | 400 | Số thẻ tín dụng/ghi nợ không hợp lệ | Số thẻ không qua kiểm tra Luhn hoặc sai định dạng | Yêu cầu kiểm tra lại số thẻ | Không |
| `PAY_004` | 400 | Thẻ đã hết hạn | Ngày hết hạn của thẻ đã qua | Thông báo thẻ hết hạn, yêu cầu cập nhật thẻ mới | Không |
| `PAY_005` | 400 | Mã CVV/CVC không đúng | Mã bảo mật 3-4 số trên thẻ không khớp | Yêu cầu kiểm tra lại mã CVV, thường ở mặt sau thẻ | Không |
| `PAY_006` | 408 | Giao dịch thanh toán hết thời gian chờ | Cổng thanh toán không phản hồi trong thời gian quy định | Thông báo timeout, kiểm tra lịch sử giao dịch trước khi thử lại | Có (1 lần, sau 30s) |
| `PAY_007` | 402 | Thẻ bị khóa hoặc bị mất cắp | Ngân hàng đã đánh dấu thẻ là bị khóa/mất | Thông báo thẻ không thể sử dụng, hướng dẫn liên hệ ngân hàng | Không |
| `PAY_008` | 400 | Phương thức thanh toán không được hỗ trợ | Loại thẻ hoặc ví điện tử chưa được tích hợp | Hiển thị danh sách phương thức thanh toán được chấp nhận | Không |
| `PAY_009` | 409 | Giao dịch trùng lặp | Cùng một giao dịch được gửi hai lần trong thời gian ngắn | Thông báo giao dịch đang xử lý, không cho phép gửi lại ngay | Có (sau 60s) |
| `PAY_010` | 402 | Vượt hạn mức giao dịch | Số tiền giao dịch vượt quá hạn mức thẻ hoặc ngày | Thông báo vượt hạn mức, gợi ý chia nhỏ giao dịch | Không |
| `PAY_011` | 400 | Tiền tệ không được hỗ trợ | Đồng tiền thanh toán không được cổng thanh toán chấp nhận | Thông báo và hiển thị danh sách đồng tiền được hỗ trợ | Không |
| `PAY_012` | 400 | Số tiền thanh toán không hợp lệ | Số tiền âm, bằng 0, hoặc không đúng định dạng tiền tệ | Kiểm tra lại số tiền trước khi gửi | Không |
| `PAY_013` | 404 | Giao dịch không tồn tại | Mã giao dịch không tìm thấy trong hệ thống | Kiểm tra lại mã giao dịch | Không |
| `PAY_014` | 400 | Không thể hoàn tiền giao dịch này | Giao dịch đã quá hạn hoàn tiền hoặc loại giao dịch không hỗ trợ refund | Thông báo lý do không thể hoàn tiền | Không |
| `PAY_015` | 400 | Số tiền hoàn tiền vượt quá số tiền gốc | Yêu cầu refund nhiều hơn số tiền trong giao dịch gốc | Thông báo số tiền tối đa có thể hoàn | Không |

---

### 📁 FILE - File/Upload (FILE_001 → FILE_099)

**Phạm vi:** Mọi vấn đề liên quan đến tải lên, tải xuống, lưu trữ và quản lý tập tin trong hệ thống.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `FILE_001` | 413 | Kích thước file vượt quá giới hạn | File upload lớn hơn kích thước tối đa cho phép | Thông báo giới hạn kích thước, gợi ý nén file | Không |
| `FILE_002` | 415 | Định dạng file không được hỗ trợ | Phần mở rộng file hoặc MIME type không trong danh sách cho phép | Thông báo danh sách định dạng được chấp nhận | Không |
| `FILE_003` | 500 | Tải lên file thất bại | Lỗi mạng, lỗi storage service, hoặc gián đoạn kết nối | Thông báo thất bại, cung cấp nút thử lại | Có |
| `FILE_004` | 404 | File không tìm thấy | File đã bị xóa hoặc URL không chính xác | Thông báo file không còn tồn tại | Không |
| `FILE_005` | 422 | File bị phát hiện chứa mã độc (Virus) | Kết quả quét virus dương tính | Thông báo file nguy hiểm, hướng dẫn kiểm tra thiết bị | Không |
| `FILE_006` | 400 | File bị hỏng hoặc không đọc được | File upload bị corrupt, không thể parse | Yêu cầu tải lên lại file hợp lệ | Không |
| `FILE_007` | 507 | Hết dung lượng lưu trữ | Tài khoản hoặc hệ thống đã hết quota lưu trữ | Thông báo hết dung lượng, hướng dẫn xóa file cũ hoặc nâng cấp gói | Không |
| `FILE_008` | 400 | File rỗng (0 bytes) | File được tải lên không có nội dung | Thông báo không thể tải file rỗng, yêu cầu chọn lại | Không |
| `FILE_009` | 400 | Số lượng file vượt quá giới hạn | Tải lên nhiều file hơn số lượng cho phép trong một lần | Thông báo giới hạn số file, yêu cầu chọn ít file hơn | Không |
| `FILE_010` | 403 | Không có quyền truy cập file | File thuộc về người dùng khác hoặc ở chế độ riêng tư | Thông báo không có quyền truy cập | Không |
| `FILE_011` | 400 | Kích thước ảnh không đúng yêu cầu | Ảnh tải lên có độ phân giải quá thấp hoặc quá cao | Thông báo yêu cầu kích thước ảnh (ví dụ: tối thiểu 200x200px) | Không |
| `FILE_012` | 408 | Tải lên bị timeout | Kết nối mạng chậm, file không tải lên xong trong thời gian quy định | Thông báo timeout, gợi ý kiểm tra kết nối và thử lại | Có |

---

### ⏱️ RATE - Giới Hạn Tốc Độ (RATE_001 → RATE_009)

**Phạm vi:** Mọi vấn đề liên quan đến giới hạn tốc độ request (Rate Limiting) và kiểm soát lưu lượng.

> **Lưu ý quan trọng:** Khi trả về lỗi RATE, server PHẢI gửi kèm các HTTP header sau:
> - `X-RateLimit-Limit`: Giới hạn tối đa
> - `X-RateLimit-Remaining`: Số request còn lại
> - `X-RateLimit-Reset`: Thời điểm (Unix timestamp) giới hạn được reset
> - `Retry-After`: Số giây cần chờ trước khi thử lại

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `RATE_001` | 429 | Vượt quá giới hạn request toàn cục | Quá nhiều request trong một khoảng thời gian (ví dụ: 100 req/phút) | Đọc header `Retry-After`, hiển thị countdown và dừng gọi API | Có (sau Retry-After) |
| `RATE_002` | 429 | Vượt quá giới hạn request cho endpoint cụ thể | Một endpoint nhạy cảm (login, OTP) bị gọi quá nhiều lần | Thông báo cụ thể và thời gian cần chờ cho endpoint đó | Có (sau Retry-After) |
| `RATE_003` | 429 | Vượt quá giới hạn tải lên file | Quá nhiều file được upload trong thời gian ngắn | Thông báo giới hạn upload, hướng dẫn chờ đợi | Có (sau Retry-After) |
| `RATE_004` | 429 | Vượt quá giới hạn gửi email | Quá nhiều email được gửi từ một tài khoản | Thông báo giới hạn email, thử lại sau thời gian quy định | Có (sau Retry-After) |
| `RATE_005` | 429 | Vượt quá giới hạn gửi SMS | Quá nhiều SMS trong thời gian ngắn | Thông báo giới hạn SMS, thử lại sau | Có (sau Retry-After) |
| `RATE_006` | 429 | Vượt quá giới hạn tìm kiếm | Endpoint search bị gọi quá nhiều lần | Debounce request tìm kiếm ở phía client (tối thiểu 300ms) | Có (sau Retry-After) |
| `RATE_007` | 429 | Vượt quá giới hạn export dữ liệu | Quá nhiều yêu cầu export trong một ngày | Thông báo giới hạn export hàng ngày | Không |

---

### ⚙️ SYS - Hệ Thống (SYS_001 → SYS_099)

**Phạm vi:** Mọi lỗi phát sinh từ nội bộ hệ thống server, bao gồm cơ sở dữ liệu, cache, hàng đợi và các dịch vụ nội bộ.

> **Nguyên tắc bảo mật:** Khi trả về lỗi SYS cho client, KHÔNG BAO GIỜ tiết lộ chi tiết kỹ thuật (tên database, cấu trúc bảng, stack trace). Chỉ trả về thông báo chung chung. Chi tiết lỗi phải được ghi vào hệ thống log nội bộ.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `SYS_001` | 500 | Lỗi máy chủ nội bộ không xác định | Ngoại lệ không được xử lý, lỗi logic không lường trước | Hiển thị trang lỗi chung, cung cấp mã `traceId` để hỗ trợ | Có (sau 5s) |
| `SYS_002` | 503 | Cơ sở dữ liệu không khả dụng | Database down, connection pool cạn kiệt | Hiển thị trang bảo trì, tự động retry sau 30 giây | Có (sau 30s) |
| `SYS_003` | 503 | Cache không khả dụng | Redis/Memcached không kết nối được | Tự động retry, hệ thống nên fallback sang DB | Có (sau 10s) |
| `SYS_004` | 503 | Dịch vụ đang bảo trì | Hệ thống đang trong cửa sổ bảo trì theo lịch | Hiển thị trang bảo trì có thời gian dự kiến hoàn thành | Không |
| `SYS_005` | 503 | Dịch vụ tạm thời không khả dụng | Microservice nội bộ gặp sự cố, circuit breaker mở | Hiển thị thông báo dịch vụ gián đoạn, retry sau 60 giây | Có (sau 60s) |
| `SYS_006` | 504 | Timeout khi gọi dịch vụ nội bộ | Dịch vụ nội bộ phản hồi quá chậm | Thông báo timeout, cung cấp traceId và gợi ý thử lại | Có (sau 10s) |
| `SYS_007` | 500 | Lỗi ghi dữ liệu vào cơ sở dữ liệu | Constraint violation, deadlock, disk full | Thông báo lỗi chung, cung cấp traceId | Có (1 lần) |
| `SYS_008` | 500 | Lỗi hàng đợi tin nhắn (Message Queue) | RabbitMQ/Kafka không nhận được message | Thông báo thao tác có thể bị trì hoãn, sẽ được xử lý sau | Có (sau 30s) |
| `SYS_009` | 500 | Lỗi xử lý bất đồng bộ (Async Job) | Background job thất bại nhiều lần | Thông báo tác vụ thất bại, cung cấp tùy chọn thử lại thủ công | Không |
| `SYS_010` | 503 | Dịch vụ lưu trữ (Storage) không khả dụng | S3/GCS/Azure Blob Storage không phản hồi | Thông báo không thể xử lý file lúc này, thử lại sau | Có (sau 30s) |
| `SYS_011` | 500 | Lỗi tạo báo cáo hoặc export | Quá trình generate PDF/Excel thất bại | Thông báo lỗi export, cung cấp tùy chọn thử lại | Có (1 lần) |
| `SYS_012` | 501 | Tính năng chưa được triển khai | Endpoint tồn tại nhưng chưa có logic xử lý | Thông báo tính năng đang phát triển, dự kiến ngày ra mắt | Không |
| `SYS_013` | 500 | Lỗi mã hóa/giải mã dữ liệu | Lỗi trong quá trình encrypt/decrypt dữ liệu nhạy cảm | Thông báo lỗi chung, cung cấp traceId cho kỹ thuật | Không |
| `SYS_014` | 500 | Lỗi gửi thông báo nội bộ | Hệ thống notification nội bộ gặp lỗi | Thông báo sẽ được gửi sau, không cần hành động từ người dùng | Có (hệ thống tự retry) |

---

### 🔗 INT - Tích Hợp Bên Ngoài (INT_001 → INT_099)

**Phạm vi:** Mọi lỗi phát sinh khi giao tiếp với các dịch vụ, API bên thứ ba ngoài hệ thống.

> **Nguyên tắc:** Lỗi từ bên ngoài phải được wrap thành mã lỗi `INT_*` trước khi trả về client. KHÔNG trả về thông báo lỗi raw từ third-party vì có thể lộ thông tin nhạy cảm.

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `INT_001` | 502 | Cổng thanh toán (Payment Gateway) gặp lỗi | VNPAY, Momo, ZaloPay, Stripe trả về lỗi nội bộ | Thông báo cổng thanh toán gặp sự cố, thử phương thức khác | Có (1 lần, sau 10s) |
| `INT_002` | 502 | Nhà cung cấp SMS gặp lỗi | Viettel SMS, ESMS, Twilio không gửi được tin nhắn | Thông báo không gửi được SMS, thử lại sau hoặc dùng email | Có (sau 30s) |
| `INT_003` | 502 | Dịch vụ gửi email gặp lỗi | SendGrid, Mailgun, SES không gửi được email | Thông báo email có thể bị trì hoãn, kiểm tra hộp thư sau vài phút | Có (hệ thống tự retry) |
| `INT_004` | 502 | Dịch vụ bản đồ (Maps API) gặp lỗi | Google Maps/HERE Maps API trả về lỗi | Hiển thị địa chỉ dạng text thay vì bản đồ, retry tự động | Có |
| `INT_005` | 502 | Dịch vụ xác minh danh tính (eKYC) gặp lỗi | FPT eKYC, VNPT eKYC không phản hồi | Thông báo không thể xác minh lúc này, thử lại sau | Có (sau 60s) |
| `INT_006` | 502 | Dịch vụ đẩy thông báo (Push Notification) gặp lỗi | Firebase FCM, Apple APNs gặp lỗi | Không cần thông báo client (lỗi background), hệ thống retry | Có (hệ thống tự retry) |
| `INT_007` | 504 | Timeout khi gọi dịch vụ bên ngoài | Third-party API không phản hồi trong thời gian quy định | Thông báo timeout, gợi ý thử lại sau | Có (sau 30s) |
| `INT_008` | 502 | Dịch vụ đăng nhập mạng xã hội (OAuth) gặp lỗi | Google/Facebook/Zalo OAuth callback lỗi | Thông báo đăng nhập mạng xã hội gặp sự cố, thử đăng nhập bằng email | Có (1 lần) |
| `INT_009` | 502 | Dịch vụ vận chuyển (Shipping API) gặp lỗi | Giao Hàng Nhanh, GHTK, ViettelPost API lỗi | Thông báo không thể tính phí ship lúc này, thử lại hoặc liên hệ hỗ trợ | Có (sau 30s) |
| `INT_010` | 502 | Dịch vụ chữ ký điện tử gặp lỗi | API ký số không phản hồi | Thông báo không thể ký lúc này, thử lại sau | Có (sau 60s) |

---

## Quy trình thêm mã lỗi mới

Để đảm bảo tính nhất quán và tính duy nhất của mã lỗi, mọi mã lỗi mới đều phải trải qua quy trình sau:

### Bước 1: Xác định domain phù hợp

Trước khi tạo mã lỗi mới, hãy tự hỏi:
- Lỗi này thuộc nghiệp vụ nào? (Xác thực? Thanh toán? Người dùng?)
- Domain nào trong bảng hiện tại phù hợp nhất?
- Nếu không có domain nào phù hợp, liệu có cần tạo domain mới không?

**Lưu ý:** Chỉ tạo domain mới khi nghiệp vụ đủ lớn và khác biệt rõ ràng. Tránh tạo domain chỉ có 1-2 mã lỗi.

### Bước 2: Kiểm tra mã lỗi tương tự đã tồn tại

Tìm kiếm trong file này và trong codebase:
```bash
# Tìm mã lỗi tương tự trong file này
grep -i "từ_khóa" Engineering/07-References/MA_LOI.md

# Tìm trong codebase
grep -r "ErrorCode\." src/ | grep -i "từ_khóa"
```

Nếu mã lỗi tương tự đã tồn tại, hãy **tái sử dụng** thay vì tạo mới.

### Bước 3: Chọn số thứ tự tiếp theo

Tìm số thứ tự cao nhất trong domain tương ứng:
```bash
grep "DOMAIN_" Engineering/07-References/MA_LOI.md | sort -t_ -k2 -n | tail -5
```

Lấy số tiếp theo. Ví dụ: nếu `AUTH_020` là cao nhất, mã mới sẽ là `AUTH_021`.

### Bước 4: Điền đầy đủ thông tin

Khi thêm vào bảng, bắt buộc phải điền đầy đủ tất cả 6 cột:
- **Mã Lỗi:** Theo định dạng `[DOMAIN]_[NNN]`
- **HTTP Status:** Phải chọn đúng HTTP status code (xem bảng Mapping bên dưới)
- **Mô Tả:** Ngắn gọn, rõ ràng, bằng tiếng Việt
- **Nguyên Nhân Thường Gặp:** Giải thích tại sao lỗi này xảy ra
- **Hành Động Client:** Hướng dẫn cụ thể phía client phải làm gì
- **Có Retry:** "Có" hoặc "Không", nếu "Có" nên ghi thêm điều kiện

### Bước 5: Định nghĩa constant trong codebase

Sau khi thêm vào file này, phải thêm constant vào file enum mã lỗi trong code:

```typescript
// File: src/common/constants/error-codes.ts
export enum ErrorCode {
  // ... mã lỗi hiện có ...

  // AUTH - Xác Thực
  AUTH_021 = 'AUTH_021', // Thêm mô tả ngắn ở đây
}
```

### Bước 6: Viết unit test

Đảm bảo mã lỗi mới được kiểm tra trong test:
```typescript
it('should return AUTH_021 when <điều kiện>', async () => {
  // Arrange
  // Act
  const response = await request(app).post('/api/...').send({...});
  // Assert
  expect(response.status).toBe(401);
  expect(response.body.error.code).toBe('AUTH_021');
});
```

### Bước 7: Tạo Pull Request

- **Tiêu đề PR:** `feat(error-codes): add AUTH_021 - [mô tả ngắn]`
- **Mô tả PR** phải bao gồm:
  - Lý do cần mã lỗi mới
  - Endpoint hoặc nghiệp vụ sẽ sử dụng
  - Link đến task/ticket liên quan
- **Reviewer bắt buộc:** Tech Lead hoặc Architecture Team
- **Không được merge** nếu chưa cập nhật file `MA_LOI.md` này

### Bước 8: Thông báo cho team

Sau khi merge, thông báo qua kênh Slack/Teams của team về mã lỗi mới:
```
[ERROR CODE] Đã thêm AUTH_021 - <mô tả>
Sử dụng khi: <giải thích>
PR: <link>
```

### Các quy định nghiêm cấm

- **NGHIÊM CẤM** thay đổi ý nghĩa của mã lỗi đã tồn tại.
- **NGHIÊM CẤM** tái sử dụng mã lỗi đã bị deprecated (đánh dấu lỗi thời thay vì xóa).
- **NGHIÊM CẤM** tạo mã lỗi không có trong file này (mọi mã lỗi phải được đăng ký).
- **NGHIÊM CẤM** sử dụng mã lỗi hard-coded dưới dạng string (luôn dùng enum/constant).

---

## Mapping HTTP Status → Domain

Bảng dưới đây giúp xác định domain phù hợp dựa trên HTTP Status Code dự kiến:

| HTTP Status | Tên | Domain Thường Dùng | Mô Tả |
|-------------|-----|---------------------|-------|
| `200` | OK | *(Không có lỗi)* | Thành công |
| `400` | Bad Request | `VAL`, `USER`, `ORDER`, `PAY`, `FILE` | Dữ liệu đầu vào sai, nghiệp vụ không hợp lệ |
| `401` | Unauthorized | `AUTH` | Chưa xác thực hoặc token không hợp lệ |
| `402` | Payment Required | `PAY` | Vấn đề liên quan đến thanh toán |
| `403` | Forbidden | `ACCESS`, `AUTH`, `USER` | Đã xác thực nhưng không có quyền |
| `404` | Not Found | `RES`, `USER`, `ORDER`, `PAY`, `FILE` | Tài nguyên không tồn tại |
| `408` | Request Timeout | `PAY`, `FILE` | Client-side timeout |
| `409` | Conflict | `RES`, `USER`, `ORDER`, `PAY` | Xung đột dữ liệu, trùng lặp |
| `410` | Gone | `RES` | Tài nguyên đã bị xóa vĩnh viễn |
| `413` | Payload Too Large | `FILE` | File hoặc request body quá lớn |
| `415` | Unsupported Media Type | `FILE` | Định dạng file không được hỗ trợ |
| `422` | Unprocessable Entity | `FILE`, `VAL` | Cú pháp đúng nhưng không xử lý được |
| `423` | Locked | `RES` | Tài nguyên đang bị khóa |
| `429` | Too Many Requests | `RATE`, `AUTH` | Vượt giới hạn tốc độ |
| `500` | Internal Server Error | `SYS` | Lỗi hệ thống không lường trước |
| `501` | Not Implemented | `SYS` | Tính năng chưa triển khai |
| `502` | Bad Gateway | `INT` | Dịch vụ bên ngoài gặp lỗi |
| `503` | Service Unavailable | `SYS`, `INT` | Dịch vụ tạm thời không khả dụng |
| `504` | Gateway Timeout | `SYS`, `INT` | Timeout khi gọi dịch vụ khác |
| `507` | Insufficient Storage | `FILE` | Hết dung lượng lưu trữ |

### Hướng dẫn chọn HTTP Status Code

```
Request có hợp lệ về cú pháp không?
├── Không → 400 Bad Request (VAL_*)
└── Có
    └── Người dùng đã xác thực chưa?
        ├── Chưa → 401 Unauthorized (AUTH_*)
        └── Đã xác thực
            └── Người dùng có quyền không?
                ├── Không → 403 Forbidden (ACCESS_*)
                └── Có quyền
                    └── Tài nguyên có tồn tại không?
                        ├── Không → 404 Not Found (RES_001)
                        └── Có
                            └── Lỗi xảy ra ở đâu?
                                ├── Server nội bộ → 500/503 (SYS_*)
                                ├── Dịch vụ ngoài → 502/504 (INT_*)
                                ├── Xung đột dữ liệu → 409 (RES_*)
                                └── Rate limit → 429 (RATE_*)
```

---

## Bảng tra cứu nhanh

Dùng bảng này khi cần tìm mã lỗi theo tình huống cụ thể:

### Tình huống đăng nhập / xác thực

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Sai username/password | `AUTH_001` | 401 |
| Access token hết hạn | `AUTH_002` | 401 |
| Token không hợp lệ | `AUTH_003` | 401 |
| Thiếu Authorization header | `AUTH_004` | 401 |
| Tài khoản bị khóa | `AUTH_005` | 403 |
| Chưa xác minh email | `AUTH_006` | 403 |
| Cần nhập OTP/2FA | `AUTH_007` | 403 |
| Session hết hạn | `AUTH_008` | 401 |
| Refresh token hết hạn | `AUTH_010` | 401 |
| Quá nhiều lần đăng nhập sai | `AUTH_012` | 429 |
| API key hết hạn | `AUTH_014` | 401 |

### Tình huống phân quyền

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Không có quyền | `ACCESS_001` | 403 |
| Không phải chủ sở hữu | `ACCESS_002` | 403 |
| IP bị chặn | `ACCESS_003` | 403 |
| Tính năng của gói cao hơn | `ACCESS_005` | 403 |
| Chỉ dành cho admin | `ACCESS_006` | 403 |
| Gói dịch vụ hết hạn | `ACCESS_012` | 403 |

### Tình huống validation dữ liệu

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Lỗi validation tổng hợp | `VAL_001` | 400 |
| Thiếu trường bắt buộc | `VAL_002` | 400 |
| Chuỗi quá dài | `VAL_003` | 400 |
| Chuỗi quá ngắn | `VAL_004` | 400 |
| Email sai định dạng | `VAL_005` | 400 |
| SĐT không hợp lệ | `VAL_006` | 400 |
| Ngày sai định dạng | `VAL_007` | 400 |
| Giá trị ngoài khoảng | `VAL_008` | 400 |
| Giá trị không hợp lệ (enum) | `VAL_009` | 400 |

### Tình huống tài nguyên

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Không tìm thấy | `RES_001` | 404 |
| Đã tồn tại | `RES_002` | 409 |
| Đang bị khóa | `RES_004` | 423 |
| Đã xóa vĩnh viễn | `RES_005` | 410 |
| Xung đột phiên bản | `RES_006` | 409 |
| Không thể xóa do phụ thuộc | `RES_007` | 400 |
| Sai trạng thái để thao tác | `RES_008` | 400 |

### Tình huống thanh toán

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Số dư không đủ | `PAY_001` | 402 |
| Thẻ bị từ chối | `PAY_002` | 402 |
| Số thẻ sai | `PAY_003` | 400 |
| Thẻ hết hạn | `PAY_004` | 400 |
| CVV sai | `PAY_005` | 400 |
| Giao dịch timeout | `PAY_006` | 408 |
| Vượt hạn mức | `PAY_010` | 402 |

### Tình huống lỗi hệ thống

| Tình huống | Mã Lỗi | HTTP |
|-----------|--------|------|
| Lỗi không xác định | `SYS_001` | 500 |
| Database không khả dụng | `SYS_002` | 503 |
| Đang bảo trì | `SYS_004` | 503 |
| Dịch vụ không khả dụng | `SYS_005` | 503 |
| Timeout nội bộ | `SYS_006` | 504 |
| Tính năng chưa làm xong | `SYS_012` | 501 |

---

## Mã lỗi bổ sung theo domain

---

### 🔐 AUTH - Mã lỗi bổ sung (AUTH_021 → AUTH_040)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `AUTH_021` | 400 | Mật khẩu đặt lại không đáp ứng yêu cầu | Mật khẩu mới quá yếu hoặc giống mật khẩu cũ | Hiển thị yêu cầu độ mạnh mật khẩu, gợi ý dùng password manager | Không |
| `AUTH_022` | 403 | Tài khoản chờ xóa, không thể thực hiện hành động | Tài khoản đang trong giai đoạn chờ xóa (ví dụ: 30 ngày) | Thông báo tài khoản sắp bị xóa, cung cấp tùy chọn hủy xóa | Không |
| `AUTH_023` | 401 | Token loại không đúng | Gửi Refresh Token vào chỗ cần Access Token hoặc ngược lại | Kiểm tra lại logic xử lý token ở phía client | Không |
| `AUTH_024` | 403 | Xác thực hai bước đã thất bại quá nhiều lần | Nhập sai OTP/2FA vượt ngưỡng cho phép | Thông báo bị khóa tạm thời, hiển thị thời gian mở khóa | Không |
| `AUTH_025` | 403 | Yêu cầu xác thực lại danh tính cho hành động nhạy cảm | Thao tác như đổi email, xóa tài khoản yêu cầu xác nhận lại mật khẩu | Hiển thị dialog xác nhận mật khẩu hoặc OTP | Không |

---

### 👤 USER - Mã lỗi bổ sung (USER_016 → USER_030)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `USER_016` | 400 | Họ tên chứa ký tự không hợp lệ | Họ tên chứa số, ký tự đặc biệt hoặc quá ngắn | Thông báo yêu cầu định dạng họ tên hợp lệ | Không |
| `USER_017` | 400 | Địa chỉ email mới giống email hiện tại | Người dùng cố đổi email sang email đang dùng | Thông báo email mới phải khác email hiện tại | Không |
| `USER_018` | 400 | Mã xác minh email không hợp lệ hoặc hết hạn | Code trong link xác minh đã cũ hoặc đã dùng | Cung cấp nút gửi lại email xác minh | Không |
| `USER_019` | 404 | Địa chỉ giao hàng không tồn tại | ID địa chỉ không thuộc về người dùng này | Tải lại danh sách địa chỉ | Không |
| `USER_020` | 400 | Vượt quá số địa chỉ tối đa được lưu | Người dùng đã có tối đa N địa chỉ (thường là 10) | Thông báo giới hạn và hướng dẫn xóa địa chỉ cũ | Không |
| `USER_021` | 400 | Không thể xóa địa chỉ giao hàng mặc định | Địa chỉ đang được đặt làm mặc định và là địa chỉ duy nhất | Yêu cầu đặt địa chỉ khác làm mặc định trước | Không |
| `USER_022` | 409 | Số CCCD/CMND đã được sử dụng | Số giấy tờ tùy thân đã liên kết với tài khoản khác | Thông báo và hướng dẫn liên hệ hỗ trợ | Không |
| `USER_023` | 400 | Thông tin CCCD/CMND không khớp với dữ liệu eKYC | Họ tên hoặc ngày sinh trên CCCD không khớp hồ sơ | Yêu cầu kiểm tra lại thông tin, upload ảnh CCCD rõ nét hơn | Không |

---

### ✅ VAL - Mã lỗi bổ sung (VAL_021 → VAL_040)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `VAL_021` | 400 | Khoảng thời gian không hợp lệ (ngày bắt đầu sau ngày kết thúc) | `start_date` lớn hơn `end_date` | Thông báo khoảng thời gian không hợp lệ | Không |
| `VAL_022` | 400 | Khoảng thời gian vượt quá giới hạn cho phép | Khoảng thời gian quá dài (ví dụ: hơn 1 năm) | Thông báo khoảng thời gian tối đa cho phép | Không |
| `VAL_023` | 400 | Tham số phân trang không hợp lệ | `page` hoặc `limit` là số âm hoặc bằng 0 | Sử dụng giá trị mặc định: `page=1`, `limit=20` | Không |
| `VAL_024` | 400 | Kích thước trang (page size) vượt quá giới hạn | `limit` vượt quá giá trị tối đa (thường là 100) | Thông báo giới hạn tối đa và tự động điều chỉnh | Không |
| `VAL_025` | 400 | Trường sắp xếp (sort field) không hợp lệ | Tên field để sort không tồn tại hoặc không được phép | Hiển thị danh sách field được phép sort | Không |
| `VAL_026` | 400 | Giá tiền không hợp lệ | Giá tiền là số âm hoặc không đúng định dạng tiền tệ | Thông báo giá tiền phải là số dương, tối đa 2 chữ số thập phân | Không |
| `VAL_027` | 400 | Tỷ lệ phần trăm không hợp lệ | Phần trăm nằm ngoài khoảng [0, 100] | Thông báo phần trăm phải từ 0 đến 100 | Không |
| `VAL_028` | 400 | Nội dung JSON không hợp lệ | Body request không thể parse thành JSON hợp lệ | Kiểm tra lại Content-Type header và định dạng JSON | Không |
| `VAL_029` | 400 | Tham số query string không hợp lệ | Giá trị query parameter sai kiểu hoặc sai định dạng | Kiểm tra lại các tham số trong URL | Không |
| `VAL_030` | 400 | Mảng chứa giá trị trùng lặp | Mảng yêu cầu các phần tử unique nhưng có giá trị trùng | Thông báo và highlight phần tử trùng lặp | Không |

---

### 📦 RES - Mã lỗi bổ sung (RES_013 → RES_020)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `RES_013` | 400 | Không thể khôi phục tài nguyên đã xóa | Tài nguyên đã xóa vĩnh viễn, không có tính năng restore | Thông báo không thể khôi phục | Không |
| `RES_014` | 409 | Tên tài nguyên đã tồn tại trong phạm vi này | Slug/name unique bị trùng trong cùng parent | Gợi ý tên thay thế hoặc yêu cầu nhập tên khác | Không |
| `RES_015` | 400 | Không thể di chuyển tài nguyên vào chính nó | Circular reference: di chuyển folder vào subfolder của nó | Thông báo không thể thực hiện thao tác | Không |
| `RES_016` | 400 | Vượt quá giới hạn số lượng tài nguyên trong gói | Người dùng đã đạt giới hạn tài nguyên của gói hiện tại | Hiển thị số lượng đã dùng/giới hạn, gợi ý nâng cấp gói | Không |

---

### 🛒 ORDER - Mã lỗi bổ sung (ORDER_016 → ORDER_025)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `ORDER_016` | 400 | Giỏ hàng đã hết hạn | Giỏ hàng không hoạt động quá lâu (thường 7 ngày) | Thông báo giỏ hàng hết hạn, yêu cầu thêm lại sản phẩm | Không |
| `ORDER_017` | 400 | Giá sản phẩm đã thay đổi kể từ khi thêm vào giỏ | Merchant đã cập nhật giá | Thông báo giá đã thay đổi, hiển thị giá mới để người dùng xác nhận | Không |
| `ORDER_018` | 400 | Sản phẩm không thể kết hợp với phương thức vận chuyển | Ví dụ: hàng cồng kềnh không thể giao hỏa tốc | Thông báo giới hạn vận chuyển, gợi ý phương thức khác | Không |
| `ORDER_019` | 400 | Không thể áp dụng đồng thời nhiều mã giảm giá | Hệ thống chỉ cho phép 1 coupon mỗi đơn hàng | Thông báo giới hạn số coupon | Không |
| `ORDER_020` | 403 | Không thể thay đổi đơn hàng do đã được xác nhận | Đơn hàng đã chuyển sang trạng thái "Đã xác nhận" | Thông báo trạng thái, hướng dẫn liên hệ CSKH nếu cần thay đổi | Không |

---

### 💳 PAY - Mã lỗi bổ sung (PAY_016 → PAY_025)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `PAY_016` | 402 | Thẻ không hỗ trợ thanh toán quốc tế | Thẻ nội địa cố thanh toán trên merchant nước ngoài | Thông báo giới hạn thẻ, gợi ý dùng thẻ quốc tế hoặc ví điện tử | Không |
| `PAY_017` | 400 | Tên chủ thẻ không hợp lệ | Tên không khớp với tên trên thẻ | Yêu cầu nhập đúng tên in trên thẻ | Không |
| `PAY_018` | 409 | Giao dịch đang chờ xác nhận | Payment đang ở trạng thái pending, cần chờ webhook | Thông báo đang chờ xác nhận thanh toán, không cho phép thao tác thêm | Có (poll sau 10s) |
| `PAY_019` | 400 | Phương thức thanh toán đã bị xóa | Thẻ/ví đã bị người dùng xóa khỏi hệ thống | Yêu cầu chọn phương thức thanh toán khác | Không |
| `PAY_020` | 400 | Không thể hoàn tiền vì giao dịch chưa được thanh toán | Giao dịch ở trạng thái pending hoặc failed | Thông báo trạng thái giao dịch hiện tại | Không |

---

### ⚙️ SYS - Mã lỗi bổ sung (SYS_015 → SYS_025)

| Mã Lỗi | HTTP Status | Mô Tả | Nguyên Nhân Thường Gặp | Hành Động Client | Có Retry? |
|--------|-------------|-------|------------------------|------------------|-----------|
| `SYS_015` | 500 | Lỗi cấu hình hệ thống | Environment variable bị thiếu hoặc sai giá trị | Thông báo lỗi hệ thống, liên hệ hỗ trợ kỹ thuật | Không |
| `SYS_016` | 503 | Hệ thống đang nâng cấp | Deployment đang diễn ra, traffic bị giảm tải | Hiển thị màn hình "Đang nâng cấp hệ thống" với thời gian dự kiến | Có (sau 60s) |
| `SYS_017` | 500 | Lỗi kết nối internal service | Giao tiếp giữa các microservice thất bại | Thông báo lỗi và cung cấp traceId | Có (sau 5s) |
| `SYS_018` | 500 | Lỗi xử lý template (Email/Notification) | Template bị thiếu hoặc có lỗi cú pháp | Không cần thông báo client, ghi log để fix | Có (hệ thống tự retry) |
| `SYS_019` | 429 | Quá tải hệ thống (Overload) | CPU/Memory vượt ngưỡng, hệ thống tự bảo vệ | Thông báo hệ thống đang tải cao, thử lại sau | Có (sau 30s) |

---

## Danh sách mã lỗi theo thứ tự ABC (Index)

Dùng để tra cứu nhanh khi biết mã lỗi cụ thể:

| Mã Lỗi | HTTP | Tóm tắt |
|--------|------|---------|
| `ACCESS_001` | 403 | Không có quyền truy cập tài nguyên |
| `ACCESS_002` | 403 | Không phải chủ sở hữu tài nguyên |
| `ACCESS_003` | 403 | Địa chỉ IP không được phép |
| `ACCESS_004` | 403 | Hành động bị từ chối theo chính sách |
| `ACCESS_005` | 403 | Vượt giới hạn tài nguyên gói dịch vụ |
| `ACCESS_006` | 403 | Tài nguyên chỉ dành cho admin |
| `ACCESS_007` | 403 | Truy cập bị giới hạn theo vùng địa lý |
| `ACCESS_008` | 403 | Tenant không có quyền thực hiện hành động |
| `ACCESS_009` | 403 | Tính năng đang trong giai đoạn thử nghiệm |
| `ACCESS_010` | 403 | Vượt quá số thiết bị đăng nhập đồng thời |
| `ACCESS_011` | 403 | Tài nguyên bị ẩn hoặc không công khai |
| `ACCESS_012` | 403 | Hết hạn gói dịch vụ |
| `AUTH_001` | 401 | Thông tin đăng nhập không hợp lệ |
| `AUTH_002` | 401 | Token truy cập đã hết hạn |
| `AUTH_003` | 401 | Token truy cập không hợp lệ |
| `AUTH_004` | 401 | Thiếu token xác thực |
| `AUTH_005` | 403 | Tài khoản bị khóa tạm thời |
| `AUTH_006` | 403 | Tài khoản chưa xác minh email |
| `AUTH_007` | 403 | Yêu cầu xác thực hai bước |
| `AUTH_008` | 401 | Phiên đăng nhập đã hết hạn |
| `AUTH_009` | 401 | Refresh token không hợp lệ |
| `AUTH_010` | 401 | Refresh token đã hết hạn |
| `AUTH_011` | 403 | Đăng nhập từ thiết bị không được phép |
| `AUTH_012` | 429 | Quá nhiều lần đăng nhập thất bại |
| `AUTH_013` | 401 | API key không hợp lệ |
| `AUTH_014` | 401 | API key đã hết hạn |
| `AUTH_015` | 403 | Tài khoản đã bị vô hiệu hóa vĩnh viễn |
| `AUTH_016` | 403 | Mã OTP không đúng |
| `AUTH_017` | 403 | Mã OTP đã hết hạn |
| `AUTH_018` | 429 | Gửi OTP quá nhiều lần |
| `AUTH_019` | 401 | Magic Link đã được sử dụng |
| `AUTH_020` | 401 | Token đặt lại mật khẩu không hợp lệ |
| `AUTH_021` | 400 | Mật khẩu đặt lại không đáp ứng yêu cầu |
| `AUTH_022` | 403 | Tài khoản chờ xóa |
| `AUTH_023` | 401 | Token loại không đúng |
| `AUTH_024` | 403 | Xác thực hai bước thất bại quá nhiều lần |
| `AUTH_025` | 403 | Yêu cầu xác thực lại danh tính |
| `FILE_001` | 413 | Kích thước file vượt quá giới hạn |
| `FILE_002` | 415 | Định dạng file không được hỗ trợ |
| `FILE_003` | 500 | Tải lên file thất bại |
| `FILE_004` | 404 | File không tìm thấy |
| `FILE_005` | 422 | File bị phát hiện chứa mã độc |
| `FILE_006` | 400 | File bị hỏng hoặc không đọc được |
| `FILE_007` | 507 | Hết dung lượng lưu trữ |
| `FILE_008` | 400 | File rỗng (0 bytes) |
| `FILE_009` | 400 | Số lượng file vượt quá giới hạn |
| `FILE_010` | 403 | Không có quyền truy cập file |
| `FILE_011` | 400 | Kích thước ảnh không đúng yêu cầu |
| `FILE_012` | 408 | Tải lên bị timeout |
| `INT_001` | 502 | Cổng thanh toán gặp lỗi |
| `INT_002` | 502 | Nhà cung cấp SMS gặp lỗi |
| `INT_003` | 502 | Dịch vụ gửi email gặp lỗi |
| `INT_004` | 502 | Dịch vụ bản đồ gặp lỗi |
| `INT_005` | 502 | Dịch vụ eKYC gặp lỗi |
| `INT_006` | 502 | Dịch vụ Push Notification gặp lỗi |
| `INT_007` | 504 | Timeout khi gọi dịch vụ bên ngoài |
| `INT_008` | 502 | Dịch vụ OAuth mạng xã hội gặp lỗi |
| `INT_009` | 502 | Dịch vụ vận chuyển gặp lỗi |
| `INT_010` | 502 | Dịch vụ chữ ký điện tử gặp lỗi |
| `ORDER_001` | 404 | Đơn hàng không tồn tại |
| `ORDER_002` | 400 | Giỏ hàng trống |
| `ORDER_003` | 400 | Sản phẩm không còn trong kho |
| `ORDER_004` | 400 | Số lượng yêu cầu vượt quá tồn kho |
| `ORDER_005` | 400 | Không thể hủy đơn hàng ở trạng thái này |
| `ORDER_006` | 400 | Mã giảm giá không hợp lệ |
| `ORDER_007` | 400 | Mã giảm giá đã hết hạn |
| `ORDER_008` | 400 | Mã giảm giá đã được sử dụng |
| `ORDER_009` | 400 | Đơn hàng không đủ điều kiện áp dụng mã giảm giá |
| `ORDER_010` | 400 | Địa chỉ giao hàng không hợp lệ |
| `ORDER_011` | 400 | Phương thức vận chuyển không khả dụng |
| `ORDER_012` | 409 | Đơn hàng đang được xử lý (trùng lặp) |
| `ORDER_013` | 400 | Sản phẩm đã bị gỡ khỏi hệ thống |
| `ORDER_014` | 400 | Yêu cầu hoàn trả vượt quá thời hạn |
| `ORDER_015` | 400 | Số lượng hoàn trả vượt quá số lượng đã mua |
| `ORDER_016` | 400 | Giỏ hàng đã hết hạn |
| `ORDER_017` | 400 | Giá sản phẩm đã thay đổi |
| `ORDER_018` | 400 | Sản phẩm không thể kết hợp với phương thức vận chuyển |
| `ORDER_019` | 400 | Không thể áp dụng đồng thời nhiều mã giảm giá |
| `ORDER_020` | 403 | Không thể thay đổi đơn hàng đã xác nhận |
| `PAY_001` | 402 | Số dư không đủ |
| `PAY_002` | 402 | Thẻ bị từ chối |
| `PAY_003` | 400 | Số thẻ không hợp lệ |
| `PAY_004` | 400 | Thẻ đã hết hạn |
| `PAY_005` | 400 | Mã CVV/CVC không đúng |
| `PAY_006` | 408 | Giao dịch thanh toán timeout |
| `PAY_007` | 402 | Thẻ bị khóa hoặc bị mất cắp |
| `PAY_008` | 400 | Phương thức thanh toán không hỗ trợ |
| `PAY_009` | 409 | Giao dịch trùng lặp |
| `PAY_010` | 402 | Vượt hạn mức giao dịch |
| `PAY_011` | 400 | Tiền tệ không được hỗ trợ |
| `PAY_012` | 400 | Số tiền thanh toán không hợp lệ |
| `PAY_013` | 404 | Giao dịch không tồn tại |
| `PAY_014` | 400 | Không thể hoàn tiền giao dịch này |
| `PAY_015` | 400 | Số tiền hoàn tiền vượt quá số tiền gốc |
| `PAY_016` | 402 | Thẻ không hỗ trợ thanh toán quốc tế |
| `PAY_017` | 400 | Tên chủ thẻ không hợp lệ |
| `PAY_018` | 409 | Giao dịch đang chờ xác nhận |
| `PAY_019` | 400 | Phương thức thanh toán đã bị xóa |
| `PAY_020` | 400 | Không thể hoàn tiền vì chưa thanh toán |
| `RATE_001` | 429 | Vượt quá giới hạn request toàn cục |
| `RATE_002` | 429 | Vượt quá giới hạn endpoint cụ thể |
| `RATE_003` | 429 | Vượt quá giới hạn tải lên file |
| `RATE_004` | 429 | Vượt quá giới hạn gửi email |
| `RATE_005` | 429 | Vượt quá giới hạn gửi SMS |
| `RATE_006` | 429 | Vượt quá giới hạn tìm kiếm |
| `RATE_007` | 429 | Vượt quá giới hạn export dữ liệu |
| `RES_001` | 404 | Tài nguyên không tồn tại |
| `RES_002` | 409 | Tài nguyên đã tồn tại |
| `RES_003` | 409 | Xung đột dữ liệu |
| `RES_004` | 423 | Tài nguyên đang bị khóa |
| `RES_005` | 410 | Tài nguyên đã bị xóa vĩnh viễn |
| `RES_006` | 409 | Xung đột phiên bản |
| `RES_007` | 400 | Không thể xóa do phụ thuộc |
| `RES_008` | 400 | Thao tác không được phép ở trạng thái này |
| `RES_009` | 400 | ID tài nguyên không đúng định dạng |
| `RES_010` | 403 | Tài nguyên bị lưu trữ |
| `RES_011` | 404 | Tài nguyên bị xóa mềm |
| `RES_012` | 400 | Danh sách tài nguyên rỗng |
| `RES_013` | 400 | Không thể khôi phục tài nguyên đã xóa |
| `RES_014` | 409 | Tên tài nguyên trùng trong phạm vi này |
| `RES_015` | 400 | Không thể di chuyển tài nguyên vào chính nó |
| `RES_016` | 400 | Vượt quá giới hạn tài nguyên trong gói |
| `SYS_001` | 500 | Lỗi máy chủ nội bộ không xác định |
| `SYS_002` | 503 | Cơ sở dữ liệu không khả dụng |
| `SYS_003` | 503 | Cache không khả dụng |
| `SYS_004` | 503 | Dịch vụ đang bảo trì |
| `SYS_005` | 503 | Dịch vụ tạm thời không khả dụng |
| `SYS_006` | 504 | Timeout khi gọi dịch vụ nội bộ |
| `SYS_007` | 500 | Lỗi ghi dữ liệu vào cơ sở dữ liệu |
| `SYS_008` | 500 | Lỗi hàng đợi tin nhắn |
| `SYS_009` | 500 | Lỗi xử lý bất đồng bộ |
| `SYS_010` | 503 | Dịch vụ lưu trữ không khả dụng |
| `SYS_011` | 500 | Lỗi tạo báo cáo hoặc export |
| `SYS_012` | 501 | Tính năng chưa được triển khai |
| `SYS_013` | 500 | Lỗi mã hóa/giải mã dữ liệu |
| `SYS_014` | 500 | Lỗi gửi thông báo nội bộ |
| `SYS_015` | 500 | Lỗi cấu hình hệ thống |
| `SYS_016` | 503 | Hệ thống đang nâng cấp |
| `SYS_017` | 500 | Lỗi kết nối internal service |
| `SYS_018` | 500 | Lỗi xử lý template |
| `SYS_019` | 429 | Quá tải hệ thống |
| `USER_001` | 404 | Người dùng không tồn tại |
| `USER_002` | 409 | Email đã được sử dụng |
| `USER_003` | 409 | Username đã tồn tại |
| `USER_004` | 409 | Số điện thoại đã được sử dụng |
| `USER_005` | 400 | Mật khẩu quá yếu |
| `USER_006` | 400 | Mật khẩu xác nhận không khớp |
| `USER_007` | 400 | Mật khẩu cũ không đúng |
| `USER_008` | 400 | Mật khẩu mới trùng mật khẩu cũ |
| `USER_009` | 400 | Ảnh đại diện không hợp lệ |
| `USER_010` | 403 | Không thể xóa tài khoản do ràng buộc |
| `USER_011` | 400 | Ngày sinh không hợp lệ |
| `USER_012` | 400 | Tuổi không đủ điều kiện |
| `USER_013` | 404 | Hồ sơ người dùng chưa được tạo |
| `USER_014` | 409 | Tài khoản mạng xã hội đã liên kết với người khác |
| `USER_015` | 400 | Không thể hủy liên kết phương thức đăng nhập duy nhất |
| `USER_016` | 400 | Họ tên chứa ký tự không hợp lệ |
| `USER_017` | 400 | Email mới giống email hiện tại |
| `USER_018` | 400 | Mã xác minh email không hợp lệ |
| `USER_019` | 404 | Địa chỉ giao hàng không tồn tại |
| `USER_020` | 400 | Vượt quá số địa chỉ tối đa |
| `USER_021` | 400 | Không thể xóa địa chỉ mặc định duy nhất |
| `USER_022` | 409 | Số CCCD/CMND đã được sử dụng |
| `USER_023` | 400 | Thông tin CCCD/CMND không khớp eKYC |
| `VAL_001` | 400 | Dữ liệu đầu vào không hợp lệ (tổng hợp) |
| `VAL_002` | 400 | Trường bắt buộc bị thiếu |
| `VAL_003` | 400 | Giá trị vượt độ dài tối đa |
| `VAL_004` | 400 | Giá trị dưới độ dài tối thiểu |
| `VAL_005` | 400 | Email không đúng định dạng |
| `VAL_006` | 400 | Số điện thoại không hợp lệ |
| `VAL_007` | 400 | Định dạng ngày giờ không hợp lệ |
| `VAL_008` | 400 | Giá trị nằm ngoài khoảng cho phép |
| `VAL_009` | 400 | Giá trị không thuộc tập hợp cho phép (Enum) |
| `VAL_010` | 400 | Giá trị bị trùng lặp |
| `VAL_011` | 400 | Định dạng URL không hợp lệ |
| `VAL_012` | 400 | Giá trị không phải số nguyên hợp lệ |
| `VAL_013` | 400 | Giá trị không phải boolean hợp lệ |
| `VAL_014` | 400 | Mảng rỗng không được phép |
| `VAL_015` | 400 | Vượt số lượng phần tử tối đa trong mảng |
| `VAL_016` | 400 | Mã bưu chính không hợp lệ |
| `VAL_017` | 400 | Số thẻ ngân hàng không hợp lệ |
| `VAL_018` | 400 | Mã màu không hợp lệ |
| `VAL_019` | 400 | Tọa độ địa lý không hợp lệ |
| `VAL_020` | 400 | Nội dung chứa ký tự không được phép |
| `VAL_021` | 400 | Khoảng thời gian không hợp lệ |
| `VAL_022` | 400 | Khoảng thời gian vượt quá giới hạn |
| `VAL_023` | 400 | Tham số phân trang không hợp lệ |
| `VAL_024` | 400 | Kích thước trang vượt quá giới hạn |
| `VAL_025` | 400 | Trường sắp xếp không hợp lệ |
| `VAL_026` | 400 | Giá tiền không hợp lệ |
| `VAL_027` | 400 | Tỷ lệ phần trăm không hợp lệ |
| `VAL_028` | 400 | Nội dung JSON không hợp lệ |
| `VAL_029` | 400 | Tham số query string không hợp lệ |
| `VAL_030` | 400 | Mảng chứa giá trị trùng lặp |

---

## Thống kê tổng hợp

| Domain | Số mã lỗi | HTTP Status phổ biến | Ghi chú |
|--------|-----------|----------------------|---------|
| AUTH | 25 | 401, 403 | Quan trọng nhất với bảo mật |
| ACCESS | 12 | 403 | Liên quan RBAC và subscription |
| USER | 23 | 400, 404, 409 | Đăng ký và quản lý tài khoản |
| VAL | 30 | 400 | Nhóm lớn nhất, dùng nhiều nhất |
| RES | 16 | 404, 409, 400 | Tài nguyên chung |
| ORDER | 20 | 400, 404, 409 | Nghiệp vụ đặt hàng |
| PAY | 20 | 400, 402, 408 | Thanh toán - nhạy cảm nhất |
| FILE | 12 | 400, 404, 413 | Upload và quản lý file |
| RATE | 7 | 429 | Kiểm soát lưu lượng |
| SYS | 19 | 500, 503 | Lỗi hệ thống nội bộ |
| INT | 10 | 502, 504 | Tích hợp bên ngoài |
| **Tổng** | **194** | | |

---

## Lịch sử thay đổi

| Phiên bản | Ngày | Người thay đổi | Nội dung |
|-----------|------|----------------|---------|
| `1.0.0` | 2026-06-23 | Architecture Team | Khởi tạo file, định nghĩa 11 domain với 194 mã lỗi, bảng tra cứu và quy trình đầy đủ |

---

> **Nhắc nhở quan trọng:** File này là nguồn sự thật duy nhất (Single Source of Truth) cho tất cả mã lỗi trong hệ thống. Mọi thay đổi phải được thực hiện tại đây trước, sau đó mới cập nhật vào codebase. Nếu bạn thấy mã lỗi trong code không có trong file này, hãy tạo Pull Request để đăng ký ngay.
