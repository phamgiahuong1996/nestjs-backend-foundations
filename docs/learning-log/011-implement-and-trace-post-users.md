# 011 - Triển khai và trace POST /users

## Câu hỏi của bạn

Bạn yêu cầu triển khai duy nhất endpoint:

```text
POST /users
```

Input:

```json
{
  "email": "string",
  "name": "string hoặc không gửi"
}
```

Luồng bắt buộc:

```text
Controller
→ Service
→ Prisma
→ Database
```

Không triển khai `GET`, `PATCH`, `DELETE` và chưa thêm DTO validation vì đó là nội dung Day 4.

Sau khi triển khai, bạn yêu cầu trace request cụ thể:

```http
POST /users
Content-Type: application/json

{
  "email": "test@example.com",
  "name": "Huong"
}
```

## Câu trả lời

### 1. Controller nhận POST /users

```ts
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Post()
  create(@Body() input: CreateUserInput) {
    return this.usersService.create(input);
  }
}
```

`@Controller('users')` tạo route prefix `/users`. `@Post()` trên method `create()` tạo endpoint hoàn chỉnh:

```text
POST /users
```

`@Body()` lấy JSON request body và gán cho `input`:

```ts
{
  email: 'test@example.com',
  name: 'Huong',
}
```

Controller không gọi Prisma trực tiếp. Nó gọi:

```ts
this.usersService.create(input)
```

### 2. Service tạo Prisma query

Input TypeScript hiện tại:

```ts
export interface CreateUserInput {
  email: string;
  name?: string;
}
```

Service:

```ts
create(input: CreateUserInput) {
  return this.prisma.user.create({
    data: {
      email: input.email,
      name: input.name,
    },
    select: {
      id: true,
      email: true,
      name: true,
      createdAt: true,
      updatedAt: true,
    },
  });
}
```

Với request đã cho, service nhận:

```ts
{
  email: 'test@example.com',
  name: 'Huong',
}
```

Sau đó service truyền cho Prisma:

```ts
data: {
  email: 'test@example.com',
  name: 'Huong',
}
```

Ý nghĩa:

```text
data   → dữ liệu cần ghi vào PostgreSQL
select → các trường Prisma trả về cho ứng dụng
```

### 3. Prisma gọi PostgreSQL

`this.prisma.user.create()` chuyển thao tác create thành truy vấn database và gửi qua PostgreSQL driver adapter.

PostgreSQL tạo bản ghi. Các field `id`, `createdAt` và `updatedAt` được xử lý theo Prisma schema:

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### 4. Kết quả quay về

Luồng kết quả:

```text
PostgreSQL
→ Prisma trả object theo select
→ UsersService.create()
→ UsersController.create()
→ NestJS chờ Promise hoàn thành
→ NestJS serialize object thành JSON
→ HTTP 201 Created
```

Response có shape:

```json
{
  "id": "một-uuid",
  "email": "test@example.com",
  "name": "Huong",
  "createdAt": "thời-gian-ISO",
  "updatedAt": "thời-gian-ISO"
}
```

Prisma tạo và resolve Promise chứa kết quả database. Prisma không tạo HTTP response. NestJS là thành phần chờ Promise, serialize object thành JSON và gửi response.

## Câu hỏi kiểm tra

1. `data` được tạo trong controller hay service?
2. Thành phần nào chờ Promise hoàn thành rồi chuyển object thành JSON response?
3. `PrismaService.onModuleInit()` chạy khi ứng dụng khởi động hay mỗi lần có `POST /users`?

## Câu trả lời của bạn

1. `data` được tạo trong service.
2. Ban đầu bạn trả lời `prisma.user.create()`.
3. Ban đầu bạn chưa chắc và nghĩ `onModuleInit()` có thể chạy mỗi lần gọi `POST /users`.

## Feedback / Đáp án đúng

### Đáp án 1: Service - đúng

`UsersService.create()` tạo object `data` rồi truyền nó cho Prisma.

### Đáp án 2: NestJS

`prisma.user.create()` thực hiện database operation và trả Promise. Promise đi ngược qua service và controller. NestJS chờ Promise hoàn thành, serialize object thành JSON rồi gửi HTTP response.

```text
Prisma tạo kết quả dữ liệu
NestJS tạo HTTP response
```

Nếu Prisma chịu trách nhiệm tạo HTTP response thì tầng truy cập database sẽ bị gắn chặt với HTTP, làm sai ranh giới trách nhiệm.

### Đáp án 3: `onModuleInit()` chạy lúc module được khởi tạo

Khi ứng dụng khởi động:

```text
NestJS tạo PrismaService
→ gọi constructor
→ gọi onModuleInit()
→ kết nối PostgreSQL
```

Mỗi request `POST /users` gọi:

```text
UsersController.create()
→ UsersService.create()
→ prisma.user.create()
```

Nó không gọi lại constructor hoặc `onModuleInit()`. Mặc định các request sử dụng lại cùng một instance `PrismaService` trong DI context.

Nếu tạo kết nối mới cho mỗi request, ứng dụng sẽ tốn thời gian, tăng tải PostgreSQL và có nguy cơ làm cạn connection pool.

`onModuleDestroy()` được dùng để disconnect khi ứng dụng hoặc module đóng.

## Câu hỏi follow-up - name optional

1. Mỗi request gọi `onModuleInit()` hay `user.create()`?
2. Nếu client không gửi `name`, service truyền gì cho Prisma và PostgreSQL lưu gì?
3. `undefined` và chuỗi rỗng có cùng ý nghĩa không?

## Câu trả lời follow-up của bạn

1. Mỗi request gọi `user.create()`.
2. Ban đầu bạn cho rằng service không truyền `null` và PostgreSQL sẽ lưu chuỗi rỗng.
3. Sau khi được giải thích, bạn xác nhận chuỗi rỗng và `undefined` là hai giá trị có ý nghĩa khác nhau.

## Feedback follow-up / Đáp án đúng

### Đáp án 1: `user.create()` - đúng

`onModuleInit()` thuộc vòng đời khởi động module. `user.create()` thuộc luồng xử lý từng request tạo user.

### Đáp án 2: PostgreSQL lưu NULL

Khi client không gửi `name`:

```ts
input.name === undefined
```

Service hiện tạo property `name` với giá trị `undefined`. Prisma hiểu `undefined` là không cung cấp field đó. Vì schema khai báo `name String?` và không có default, PostgreSQL lưu `NULL`.

Migration đã áp dụng:

```sql
ALTER TABLE "User" ALTER COLUMN "name" DROP NOT NULL;
```

### Đáp án 3: `undefined` và chuỗi rỗng khác nhau - đúng

```text
Không gửi name      → input.name là undefined → Prisma bỏ qua → PostgreSQL lưu NULL
Gửi "name": ""     → giá trị là empty string → PostgreSQL lưu ""
Gửi "name": "Huong" → PostgreSQL lưu "Huong"
```

JSON không có kiểu `undefined`. `input.name` trở thành `undefined` trong JavaScript vì property `name` không tồn tại trong request body.

Do chưa có DTO validation, endpoint hiện chưa ngăn client gửi chuỗi rỗng hoặc dữ liệu sai kiểu. Quy tắc đó được giữ lại cho Day 4.

## Trạng thái triển khai đã xác minh

Các thành phần đã được nối:

```text
AppModule imports UsersModule
→ UsersModule imports PrismaModule
→ PrismaModule exports PrismaService
→ UsersService injects PrismaService
→ UsersController injects UsersService
```

Hạ tầng Prisma 7 tối thiểu đã có:

- `PrismaService` mở kết nối trong `onModuleInit()`.
- `PrismaService` đóng kết nối trong `onModuleDestroy()`.
- PostgreSQL driver adapter đọc `DATABASE_URL` từ biến môi trường.
- Prisma Client được generate với `moduleFormat = "cjs"` phù hợp project hiện tại.

Các kiểm tra đã có bằng chứng thành công:

- Prisma schema hợp lệ.
- Migration `make_user_name_optional` đã được tạo và áp dụng.
- Prisma Client đã được generate.
- Lint các file liên quan không có lỗi.
- TypeScript type-check thành công.
- Ứng dụng khởi động và map route `POST /users` thành công.
- Không tạo user test trong database trong quá trình kiểm tra.

## Tóm tắt cuối

Thứ tự method với request cụ thể:

```text
1. UsersController.create(input)
2. UsersService.create(input)
3. Prisma user.create({ data, select })
4. PostgreSQL tạo bản ghi
5. Prisma resolve kết quả
6. NestJS tạo HTTP 201 JSON response
```

Điểm cần nhớ:

```text
Controller      → nhận HTTP body và gọi service
Service         → tạo data và gọi Prisma
Prisma          → thao tác database và trả object
PostgreSQL      → lưu bản ghi thật
NestJS          → chờ Promise và tạo HTTP response
onModuleInit()  → chạy lúc khởi động, không chạy cho từng request
name omitted    → PostgreSQL lưu NULL
name: ""        → PostgreSQL lưu empty string
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo phù hợp là tự gửi một request `POST /users` và quan sát response cùng bản ghi trong PostgreSQL. Chưa triển khai endpoint khác và chưa thêm DTO validation.
