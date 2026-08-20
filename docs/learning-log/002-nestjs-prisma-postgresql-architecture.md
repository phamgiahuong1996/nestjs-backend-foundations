# 002 - Kiến trúc NestJS, Prisma và PostgreSQL

## Câu hỏi của bạn

I want to connect this NestJS project to PostgreSQL using Prisma.

Do NOT change code yet.

Explain the architecture first:

```text
NestJS
→ Prisma
→ PostgreSQL
```

What role does each one play?

## Câu trả lời

Kiến trúc tổng thể:

```text
HTTP request
  → NestJS
  → Prisma
  → PostgreSQL
```

### 1. NestJS làm gì?

NestJS là tầng application/API.

Nó nhận HTTP request, định tuyến request tới controller phù hợp, gọi service để xử lý logic, rồi trả HTTP response.

Trong project này, NestJS sẽ là nơi ta xây User CRUD REST API.

Controller không nên chứa nhiều business logic. Controller nên nhỏ, tập trung vào request/response. Service là nơi xử lý logic chính.

### 2. Prisma làm gì?

Prisma là công cụ giúp NestJS giao tiếp với PostgreSQL.

Prisma không phải database và không tự lưu dữ liệu thay PostgreSQL.

Vai trò chính của Prisma:

- Cho phép code TypeScript gọi database theo cách có type rõ ràng.
- Chuyển lời gọi trong code thành câu query gửi tới PostgreSQL.
- Quản lý schema và migration khi cần thay đổi cấu trúc database.

Ví dụ sau này service có thể gọi Prisma để lấy user:

```ts
this.prisma.user.findMany()
```

Lời gọi này không lấy dữ liệu từ Prisma. Prisma sẽ gửi query xuống PostgreSQL, rồi nhận kết quả từ PostgreSQL trả lại cho NestJS.

Mental model:

```text
Prisma = cầu nối / công cụ truy cập dữ liệu
PostgreSQL = nơi lưu dữ liệu thật
```

### 3. PostgreSQL làm gì?

PostgreSQL là database thật.

Nó chịu trách nhiệm lưu dữ liệu lâu dài trên ổ đĩa, ví dụ bảng `users`, các record user, email, tên, ngày tạo, ngày cập nhật.

Nếu server NestJS restart, dữ liệu không mất vì dữ liệu được lưu trong PostgreSQL.

Nếu Prisma không chạy hoặc NestJS không kết nối được database, dữ liệu vẫn nằm trong PostgreSQL. Nhưng app sẽ không đọc/ghi được dữ liệu cho tới khi kết nối hoạt động lại.

### 4. Flow khi có request tạo user

Ví dụ sau này client gửi:

```text
POST /users
```

Flow sẽ là:

```text
Client
  → NestJS Controller nhận request
  → NestJS Service xử lý logic tạo user
  → Prisma gửi query insert xuống PostgreSQL
  → PostgreSQL lưu user thật vào database
  → Prisma trả kết quả về service
  → Controller trả response cho client
```

Điểm quan trọng:

- NestJS xử lý API và business logic.
- Prisma giúp code TypeScript giao tiếp với database.
- PostgreSQL là nơi lưu dữ liệu thật.

## Câu hỏi kiểm tra

1. Prisma có phải database không? Vì sao?
2. Nếu controller gọi thẳng Prisma thay vì đi qua service, điều gì có thể bất tiện về sau?
3. Nếu NestJS server restart, dữ liệu user có bị mất không? Vì sao?

## Câu trả lời của bạn

1. Không. Prisma chỉ là tool giao tiếp với PostgreSQL, nó không phải là database.
2. Service là nơi xử lý business logic. Nếu gọi Prisma trực tiếp từ controller thì tiện hơn lúc đầu do bỏ được lớp service.
3. Không bị mất vì đã được lưu vào database PostgreSQL.

## Feedback / Đáp án đúng

### 1. Prisma có phải database không?

Đúng.

Prisma không phải database. Prisma chỉ là công cụ giao tiếp với PostgreSQL.

PostgreSQL mới là nơi lưu dữ liệu thật.

### 2. Controller gọi thẳng Prisma có bất tiện gì?

Ý của bạn đúng ở phần service là nơi xử lý business logic.

Gọi Prisma trực tiếp trong controller có thể nhìn tiện lúc ban đầu vì ít file hơn, nhưng về sau sẽ bất tiện:

- Controller bị dính chặt với database logic.
- Khó test business logic riêng.
- Nếu đổi cách lưu dữ liệu, controller cũng dễ bị ảnh hưởng.
- Controller phình to vì vừa nhận request vừa xử lý logic vừa gọi database.

Với dự án học backend này, ta giữ flow rõ:

```text
Controller → Service → Prisma → PostgreSQL
```

### 3. NestJS restart thì dữ liệu có mất không?

Đúng.

Dữ liệu không mất vì dữ liệu được lưu trong PostgreSQL, không nằm trong memory của NestJS.

NestJS là app server. PostgreSQL là nơi lưu trữ lâu dài.

## Câu hỏi follow-up

1. HTTP request đầu tiên đi vào NestJS hay Prisma?
2. Vì sao service giúp controller ít phụ thuộc hơn vào database?
3. PostgreSQL lưu dữ liệu ở đâu?

## Câu trả lời follow-up của bạn

1. HTTP.
2. Khiến controller không bị ảnh hưởng bởi service. Chỉ cần thay database trong service thành tool khác là dùng được.
3. Service lưu dữ liệu lên ổ đĩa.

## Feedback follow-up / Đáp án đúng

### 1. HTTP request đầu tiên đi vào đâu?

Ý đúng nhưng cần nói chính xác hơn:

HTTP request đầu tiên đi vào NestJS, cụ thể là route/controller tương ứng.

Prisma không nhận HTTP request. Prisma chỉ được gọi sau đó, thường từ service.

### 2. Vì sao service giúp controller ít phụ thuộc hơn vào database?

Ý chính đúng.

Nói chính xác hơn: controller không nên biết chi tiết database được gọi bằng Prisma, raw SQL, hoặc tool khác.

Controller chỉ gọi service:

```text
usersService.findAll()
```

Bên trong service, ta có thể dùng Prisma để nói chuyện với PostgreSQL. Nếu sau này đổi cách truy cập database, ta chủ yếu sửa trong service, controller ít bị ảnh hưởng hơn.

### 3. PostgreSQL lưu dữ liệu ở đâu?

Chỗ này cần sửa lại.

Service không phải nơi lưu dữ liệu lên ổ đĩa. Service chỉ chứa logic và gọi công cụ truy cập dữ liệu.

PostgreSQL mới là nơi ghi dữ liệu thật xuống ổ đĩa.

## Câu hỏi follow-up 2

1. Trong bộ ba NestJS, Prisma, PostgreSQL, cái nào thật sự ghi dữ liệu?
2. Nếu PostgreSQL không chạy, Prisma có tự lưu dữ liệu thay được không?

## Câu trả lời follow-up 2 của bạn

1. PostgreSQL.
2. Prisma không lưu thay thế được.

## Feedback follow-up 2 / Đáp án đúng

Đúng cả hai ý.

PostgreSQL là thành phần thật sự ghi và lưu dữ liệu.

Prisma không lưu dữ liệu thay PostgreSQL. Nếu PostgreSQL không chạy hoặc không kết nối được, Prisma không thể đọc/ghi dữ liệu.

Tóm tắt cuối:

```text
NestJS → xử lý API và business logic
Prisma → giao tiếp với database
PostgreSQL → lưu trữ dữ liệu
```
