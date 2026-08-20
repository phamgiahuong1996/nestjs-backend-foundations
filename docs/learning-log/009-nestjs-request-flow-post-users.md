# 009 - Luồng request NestJS với POST /users

## Câu hỏi của bạn

Bạn muốn học luồng request trong NestJS và chưa triển khai code:

```text
HTTP Request
→ UsersController
→ UsersService
→ Prisma
→ PostgreSQL
→ Response
```

Bạn cũng hỏi vì sao không nên đặt toàn bộ business logic trong controller.

## Câu trả lời

Ví dụ client gửi `POST /users`, luồng đi và về đầy đủ là:

```text
Client gửi POST /users
→ UsersController nhận HTTP request
→ UsersService xử lý business rules
→ Prisma thực hiện thao tác dữ liệu
→ PostgreSQL lưu và trả kết quả
→ Prisma chuyển kết quả thành object TypeScript
→ UsersService xử lý kết quả
→ UsersController trả kết quả
→ NestJS tạo HTTP response
```

### Vai trò của từng tầng

#### UsersController

- Biết route và HTTP method, ví dụ `POST /users`.
- Nhận input từ request.
- Gọi phương thức phù hợp trong `UsersService`.
- Chuyển kết quả cho NestJS để tạo HTTP response.

Controller nên tập trung vào HTTP, không nên chứa toàn bộ business logic.

#### UsersService

- Thực hiện use case tạo user.
- Áp dụng business rules, ví dụ email không được trùng.
- Quyết định dữ liệu nào an toàn để trả về.
- Gọi Prisma để thao tác dữ liệu.
- Chuyển lỗi kỹ thuật thành lỗi có ý nghĩa với API, ví dụ `409 Conflict`.

#### Prisma

- Không phải database.
- Là lớp truy cập dữ liệu giữa ứng dụng TypeScript và PostgreSQL.
- Chuyển lời gọi Prisma Client thành thao tác gửi tới PostgreSQL.
- Chuyển kết quả database thành object TypeScript có type.

#### PostgreSQL

- Là database thật sự.
- Lưu bản ghi lâu dài.
- Bảo vệ tính toàn vẹn dữ liệu bằng constraint, ví dụ `UNIQUE` cho email.

### Vì sao business logic không nên nằm hết trong controller?

Nếu business logic nằm trong controller:

- Logic bị gắn chặt với HTTP.
- Khó tái sử dụng từ background job hoặc điểm vào khác.
- Controller dễ phình to và trộn nhiều trách nhiệm.
- Khó kiểm thử business logic riêng.
- Dễ lặp lại logic ở nhiều controller hoặc route.

Giữ business logic trong service giúp controller nhỏ và cho phép nhiều điểm vào dùng lại cùng một quy tắc.

## Câu hỏi kiểm tra

1. `UsersController` và `UsersService` khác nhau chủ yếu ở đâu?
2. Prisma có phải database không?
3. Vì sao đặt logic tạo user trong service giúp ích cho background job?

## Câu trả lời của bạn

1. Controller nhận request từ client và trả response từ service; service xử lý business rules.
2. Prisma không phải database, mà là tool chuyển đổi giữa ứng dụng NestJS và PostgreSQL.
3. Logic trong service có thể được tái sử dụng.

## Feedback / Đáp án đúng

Cả ba ý chính đều đúng.

Điểm diễn đạt chính xác hơn:

- Controller chuyển kết quả cho NestJS; NestJS thường serialize JSON và tạo HTTP response.
- Prisma là lớp truy cập dữ liệu giữa ứng dụng và PostgreSQL, không phải chỉ riêng framework NestJS.
- Service cho phép controller, background job hoặc điểm vào khác dùng chung business rules.

## Câu hỏi follow-up - Luồng trả về và email trùng

1. Nếu email đã tồn tại, việc quyết định báo lỗi nên nằm ở đâu?
2. Kết quả từ PostgreSQL đi qua các tầng nào trước khi thành HTTP response?
3. Controller có nên gọi trực tiếp `prisma.user.create()` không?

## Câu trả lời follow-up của bạn

1. Báo lỗi nên nằm trong service.
2. Ban đầu bạn trả lời: `PostgreSQL → Prisma → Controller → HTTP response`.
3. Không nên vì controller chỉ nhận request và không nên thao tác trực tiếp với database.

## Feedback follow-up / Đáp án đúng

### Luồng trả về

Câu trả lời ban đầu còn thiếu service. Luồng đúng là:

```text
PostgreSQL
→ Prisma
→ UsersService
→ UsersController
→ NestJS tạo HTTP response
```

Service có thể xử lý, chuyển đổi hoặc giới hạn kết quả trước khi trả về controller.

### Email trùng và UNIQUE constraint

Service xử lý quy tắc nghiệp vụ và tạo lỗi thân thiện. PostgreSQL vẫn cần `UNIQUE constraint` để bảo vệ dữ liệu ở tầng database.

Chỉ kiểm tra email trong service là chưa đủ vì có thể xảy ra race condition:

```text
Request A kiểm tra: email chưa tồn tại
Request B kiểm tra: email chưa tồn tại
Request A tạo user
Request B cũng cố tạo user
```

`UNIQUE constraint` bảo đảm chỉ một bản ghi được tạo thành công. Service nên chuyển lỗi kỹ thuật do database hoặc Prisma trả về thành lỗi API có ý nghĩa, ví dụ `409 Conflict`.

## Câu hỏi follow-up 2 - Dữ liệu nhạy cảm với Prisma

1. Lớp nào nên bảo đảm `passwordHash` không xuất hiện trong HTTP response?
2. Prisma có thể loại bỏ trường nhạy cảm như thế nào?
3. `data` và `select` trong `prisma.user.create()` khác nhau thế nào?

## Câu trả lời follow-up 2 của bạn

- Service nên bảo đảm trường nhạy cảm không xuất hiện trong response.
- `select` an toàn hơn `omit`.
- Ban đầu bạn hiểu nhầm `select` kiểm soát dữ liệu ghi vào PostgreSQL; sau khi được giải thích, bạn xác nhận `passwordHash` vẫn được lưu nếu nằm trong `data`, nhưng không xuất hiện trong kết quả nếu không nằm trong `select`.

## Feedback follow-up 2 / Đáp án đúng

Prisma có hai cách giới hạn trường trả về:

### `select` - Chỉ lấy các trường được phép

```ts
const user = await prisma.user.create({
  data: userData,
  select: {
    id: true,
    name: true,
    email: true,
    createdAt: true,
  },
});
```

`select` thường an toàn hơn cho API response vì nó là danh sách cho phép. Trường mới được thêm vào model sẽ không tự xuất hiện trong kết quả.

### `omit` - Loại bỏ các trường cụ thể

```ts
const user = await prisma.user.create({
  data: userData,
  omit: {
    passwordHash: true,
  },
});
```

`omit` trả về các trường còn lại. Nếu sau này thêm trường nhạy cảm mới mà quên cập nhật `omit`, trường đó có thể bị lộ. Không dùng `select` và `omit` cùng lúc trong một query.

Công thức cần nhớ:

```text
create() = thao tác tạo bản ghi
data     = dữ liệu được ghi vào PostgreSQL
select   = các trường Prisma trả về cho ứng dụng
omit     = các trường Prisma loại khỏi kết quả trả về
```

Ví dụ, `passwordHash` có thể nằm trong `data` để được lưu vào PostgreSQL nhưng không nằm trong `select`, nên object Prisma trả về không chứa trường đó.

## Tóm tắt cuối

```text
Controller  → xử lý HTTP request/response
Service     → xử lý use case và business rules
Prisma      → truy cập và định hình kết quả dữ liệu
PostgreSQL  → lưu trữ và bảo vệ tính toàn vẹn dữ liệu
```

Luồng đầy đủ:

```text
HTTP → Controller → Service → Prisma → PostgreSQL
HTTP ← Controller ← Service ← Prisma ← PostgreSQL
```

Các điểm quan trọng đã nắm được:

- Business logic trong service có thể tái sử dụng.
- Kết quả từ Prisma quay lại service trước khi tới controller.
- Service cung cấp lỗi thân thiện; database constraint bảo vệ dữ liệu thật sự.
- `select` kiểm soát dữ liệu trả về, không kiểm soát dữ liệu được ghi.
- `data` và `select` có hai trách nhiệm khác nhau.

## Ghi chú bước tiếp theo

Bước học nhỏ tiếp theo phù hợp là tìm hiểu validation của request body trước khi `UsersController` chuyển dữ liệu cho `UsersService`.
