# 012 - Chọn Prisma query cho GET /users

## Câu hỏi của bạn

Trước khi triển khai `GET /users`, bạn muốn biết Prisma query nào phù hợp và vì sao.

Đây chỉ là bước giải thích kiến trúc. Chưa sửa controller, service hoặc thêm endpoint GET.

## Câu trả lời

### Đáp án: `prisma.user.findMany()`

Query dự kiến:

```ts
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
```

### Vì sao dùng `findMany()`?

`GET /users` có mục đích lấy danh sách gồm zero, one hoặc nhiều user. `findMany()` luôn trả về một array:

```ts
[
  {
    id: 'uuid-1',
    email: 'user1@example.com',
    name: 'User 1',
    createdAt: Date,
    updatedAt: Date,
  },
  {
    id: 'uuid-2',
    email: 'user2@example.com',
    name: null,
    createdAt: Date,
    updatedAt: Date,
  },
]
```

Nếu database không có user, kết quả là:

```ts
[]
```

Array rỗng là kết quả thành công bình thường. API không cần trả `404 Not Found` chỉ vì danh sách không có phần tử.

### Vì sao không dùng query khác?

```text
findMany()  → lấy danh sách nhiều record
findUnique() → lấy một record bằng field unique như id hoặc email
findFirst()  → lấy record đầu tiên phù hợp
count()      → chỉ lấy số lượng record
```

`findUnique()` không phù hợp vì nó bắt buộc cần điều kiện unique trong `where` và chỉ trả một record hoặc `null`.

### Vì sao dùng `select`?

Nếu gọi:

```ts
this.prisma.user.findMany();
```

Prisma trả về tất cả scalar field hiện có trong model `User`.

`select` tạo danh sách field được phép xuất hiện trong response. Nó mang lại hai lợi ích:

1. Giữ API response shape ổn định.
2. Tránh vô tình lộ field mới nếu sau này model thêm dữ liệu nhạy cảm như `passwordHash`.

Ví dụ, dù model có thêm `passwordHash`, query sử dụng `select` ở trên vẫn chỉ trả:

```text
id, email, name, createdAt, updatedAt
```

### Vì sao dùng `orderBy`?

Database không bảo đảm thứ tự record nếu query không có `orderBy`.

```ts
orderBy: {
  createdAt: 'desc',
}
```

Yêu cầu PostgreSQL trả user mới nhất trước và giúp response có thứ tự dễ dự đoán.

### Có cần `where` không?

Hiện tại không cần vì `GET /users` dự kiến lấy tất cả user. Sau này, khi endpoint có filter theo email hoặc tên, query mới cần `where`.

### Có cần pagination ngay không?

Chưa cần trong bước học tối thiểu này. Trong API thực tế có nhiều dữ liệu, không nên trả toàn bộ bảng. Có thể bổ sung `take`, `skip` hoặc cursor pagination trong một bước học sau.

### Luồng dự kiến

```text
GET /users
→ UsersController gọi UsersService.findAll()
→ UsersService gọi prisma.user.findMany()
→ PostgreSQL đọc danh sách user
→ Prisma trả array user
→ NestJS trả HTTP 200 JSON
```

## Câu hỏi kiểm tra

1. Nếu database không có user, `findMany()` trả gì?
2. Vì sao `findUnique()` không phù hợp với `GET /users`?
3. Nếu bỏ `orderBy`, database có bảo đảm user mới nhất đứng trước không?

## Câu trả lời của bạn

1. `findMany()` trả về array rỗng.
2. `GET /users` lấy tất cả user, còn `findUnique()` tìm một user bằng field unique như `id` hoặc `email`.
3. Database không bảo đảm trả user mới nhất trước nếu không có `orderBy`.

## Feedback / Đáp án đúng

### Đáp án 1: Array rỗng - đúng

```ts
[]
```

Đây là response thành công, không phải lỗi `404`.

### Đáp án 2: `findUnique()` chỉ tìm một record - đúng

`findUnique()` cần một điều kiện unique và trả một object hoặc `null`. Nó không thể đại diện cho yêu cầu lấy danh sách tất cả user.

### Đáp án 3: Không bảo đảm thứ tự - đúng

Không có `orderBy`, PostgreSQL có thể trả record theo thứ tự khác tùy query plan và trạng thái dữ liệu.

## Câu hỏi follow-up

1. Vì sao vẫn nên dùng `select` dù model hiện chưa có field nhạy cảm?
2. Nếu database có ba user, kết quả tổng quát của `findMany()` là một object hay array các object?

## Câu trả lời follow-up của bạn

1. `select` bảo đảm response đúng shape mong đợi. Nếu sau này thêm field như password mà không có `select`, query có thể trả tất cả column của bảng `User`.
2. Kết quả là array các object.

## Feedback follow-up / Đáp án đúng

### Đáp án 1: Kiểm soát response shape và bảo vệ dữ liệu - đúng

`select` vừa giữ API contract ổn định, vừa ngăn field mới tự xuất hiện trong response.

### Đáp án 2: Array các object - đúng

Kiểu kết quả tổng quát là một array chứa các object có shape được quyết định bởi `select`.

## Trạng thái hiện tại

- `POST /users` đã được triển khai.
- `GET /users` chưa được triển khai.
- `UsersController` hiện chỉ có `@Post()`.
- `UsersService` hiện chỉ có `create()`.
- Prisma model `User` có các field `id`, `email`, `name`, `createdAt`, `updatedAt`.
- Chưa thêm filter hoặc pagination.
- Chưa thêm DTO validation.

## Tóm tắt cuối

```text
Query phù hợp → prisma.user.findMany()
Kết quả       → array user hoặc []
select        → giới hạn field trả về
orderBy       → bảo đảm thứ tự mong muốn
where         → chưa cần khi lấy tất cả user
pagination    → để bước sau khi cần xử lý nhiều dữ liệu
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo phù hợp là triển khai duy nhất `GET /users` bằng `UsersController.findAll()` và `UsersService.findAll()` với `findMany`, `select` và `orderBy`. Chưa triển khai GET theo id, PATCH, DELETE, filter, pagination hoặc DTO validation.
