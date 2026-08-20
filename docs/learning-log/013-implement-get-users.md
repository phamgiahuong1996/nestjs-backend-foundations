# 013 - Triển khai GET /users

## Câu hỏi của bạn

Bạn yêu cầu triển khai duy nhất:

```text
GET /users
```

Không triển khai GET theo id, filter, pagination, PATCH, DELETE hoặc DTO validation.

## Câu trả lời

### 1. Controller thêm route GET

`UsersController` hiện có:

```ts
@Get()
findAll() {
  return this.usersService.findAll();
}
```

Kết hợp với route prefix:

```ts
@Controller('users')
```

endpoint hoàn chỉnh là:

```text
GET /users
```

Controller không gọi Prisma trực tiếp. Nó chuyển request cho `UsersService.findAll()`.

### 2. Service dùng `findMany()`

`UsersService` hiện có:

```ts
findAll() {
  return this.prisma.user.findMany({
    select: {
      id: true,
      email: true,
      name: true,
      createdAt: true,
      updatedAt: true,
    },
    orderBy: {
      createdAt: 'desc',
    },
  });
}
```

Ý nghĩa:

```text
findMany() → lấy danh sách user
select     → chỉ trả các field API cho phép
orderBy    → sắp xếp user mới nhất trước
```

### 3. Luồng thực thi

```text
GET /users
→ UsersController.findAll()
→ UsersService.findAll()
→ prisma.user.findMany()
→ PostgreSQL đọc các record User
→ Prisma trả array
→ NestJS trả HTTP 200 JSON
```

Nếu có user, response có dạng:

```json
[
  {
    "id": "một-uuid",
    "email": "user@example.com",
    "name": "User",
    "createdAt": "thời-gian-ISO",
    "updatedAt": "thời-gian-ISO"
  }
]
```

Nếu không có user:

```json
[]
```

Cả hai trường hợp đều là request thành công và trả `200 OK`.

### 4. Vì sao không cần migration?

Bước này chỉ thêm code đọc dữ liệu. Prisma model và cấu trúc bảng PostgreSQL không thay đổi.

Migration chỉ cần khi thay đổi database schema, ví dụ:

- Thêm hoặc xóa column.
- Đổi kiểu dữ liệu.
- Thay nullable hoặc required.
- Thêm index hoặc constraint.

`findMany()` chỉ sử dụng cấu trúc `User` đã tồn tại nên không cần migration hoặc generate lại Prisma Client.

## Câu hỏi kiểm tra

1. Sau `UsersController.findAll()`, method nào chạy tiếp theo?
2. Vì sao `GET /users` trả mặc định `200`, còn `POST /users` trả `201`?
3. Vì sao bước này không cần Prisma migration?

## Câu trả lời của bạn

1. `UsersService.findAll()` chạy tiếp theo.
2. Theo chuẩn REST API, GET thành công trả `200`, còn POST tạo resource thành công trả `201`.
3. Không có bản thiết kế mới cho model nên không cần migrate thay đổi vào PostgreSQL.

## Feedback / Đáp án đúng

### Đáp án 1: `UsersService.findAll()` - đúng

Luồng đầy đủ:

```text
UsersController.findAll()
→ UsersService.findAll()
→ prisma.user.findMany()
→ PostgreSQL
```

### Đáp án 2: GET trả 200, POST tạo mới trả 201 - đúng

```text
GET 200 OK       → đọc dữ liệu thành công
POST 201 Created → tạo resource mới thành công
```

NestJS đặt status mặc định dựa trên decorator HTTP:

- `@Get()` trả `200 OK`.
- `@Post()` trả `201 Created`.

### Đáp án 3: Database schema không thay đổi - đúng

Thêm controller method, service method và Prisma query không làm thay đổi table, column hoặc constraint trong PostgreSQL. Vì vậy không cần migration.

## Trạng thái đã xác minh

- `UsersController` có `@Get()` và `@Post()`.
- `UsersService` có `findAll()` và `create()`.
- `GET /users` dùng `prisma.user.findMany()`.
- Response được giới hạn bằng `select`.
- Danh sách được sắp xếp bằng `createdAt: 'desc'`.
- Lint riêng các file thay đổi đã thành công.
- TypeScript type-check đã thành công.
- Request thực tế `GET /users` đã trả `HTTP 200` và array user.
- Không tạo hoặc thay đổi dữ liệu trong quá trình kiểm tra GET.
- Không có migration mới vì Prisma schema không đổi.

## Tóm tắt cuối

```text
Controller method → UsersController.findAll()
Service method    → UsersService.findAll()
Prisma query      → prisma.user.findMany()
Database action   → đọc danh sách User
Success status    → 200 OK
Empty result      → []
Migration         → không cần
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo phù hợp là trace một request `GET /users` cụ thể qua controller, service, Prisma, PostgreSQL và response. Chưa triển khai endpoint mới.
