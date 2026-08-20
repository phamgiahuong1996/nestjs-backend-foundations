# 014 - Chọn Prisma query cho GET /users/:id

## Câu hỏi của bạn

Trước khi triển khai `GET /users/:id`, bạn muốn biết Prisma query nào phù hợp và vì sao.

Bạn cũng nhận ra cùng một `select` đang được lặp trong nhiều query và hỏi cách tái sử dụng để tránh sửa sót khi API response thay đổi.

Đây chỉ là bước giải thích. Chưa triển khai endpoint theo id và chưa refactor source code.

## Câu trả lời

### Đáp án: `prisma.user.findUnique()` với `where: { id }`

Query dự kiến:

```ts
const user = await this.prisma.user.findUnique({
  where: {
    id,
  },
  select: {
    id: true,
    email: true,
    name: true,
    createdAt: true,
    updatedAt: true,
  },
});
```

Trong Prisma schema:

```prisma
id String @id @default(uuid())
```

`id` là primary key nên mỗi giá trị chỉ thuộc về tối đa một user. `findUnique()` thể hiện đúng yêu cầu của route:

```text
Input  → một id unique
Output → một User object hoặc null
```

Nếu tìm thấy, Prisma trả một object:

```ts
{
  id: 'một-uuid',
  email: 'test@example.com',
  name: 'Huong',
  createdAt: Date,
  updatedAt: Date,
}
```

Nếu không tìm thấy:

```ts
null
```

### Chuyển null thành HTTP 404

Prisma chỉ biết record có tồn tại hay không. Service chịu trách nhiệm chuyển kết quả database thành lỗi có ý nghĩa với API:

```ts
const user = await this.prisma.user.findUnique(...);

if (!user) {
  throw new NotFoundException(`User with id ${id} not found`);
}

return user;
```

Luồng khi không tìm thấy:

```text
PostgreSQL không có record
→ Prisma trả null
→ UsersService phát hiện null
→ UsersService ném NotFoundException
→ NestJS trả HTTP 404
```

### Vì sao chưa ưu tiên `findUniqueOrThrow()`?

`findUniqueOrThrow()` ném lỗi kỹ thuật của Prisma khi không tìm thấy record. Ứng dụng vẫn phải bắt và chuyển lỗi đó thành HTTP `404`.

Trong bước học hiện tại, `findUnique()` làm luồng rõ hơn:

```text
Prisma trả null
→ Service kiểm tra null
→ Service chủ động tạo NotFoundException
```

`findUniqueOrThrow()` phù hợp khi project đã có cơ chế chuyển đổi Prisma error thống nhất.

### Vì sao không dùng `findFirst()`?

`findFirst()` phù hợp với điều kiện không nhất thiết unique. Vì `id` đã là primary key, `findUnique()` diễn đạt chính xác hơn.

### Vì sao không dùng `findMany()`?

`findMany()` trả array, còn endpoint theo id cần một resource:

```text
findMany()  → User[]
findUnique() → User | null
```

Response đúng cho endpoint theo id là một object, không phải array một phần tử.

### Vì sao không cần `orderBy`?

Query trả tối đa một record nên không có danh sách để sắp xếp.

### Luồng dự kiến

```text
GET /users/:id
→ UsersController.findOne(id)
→ UsersService.findOne(id)
→ prisma.user.findUnique({ where: { id } })
→ PostgreSQL
→ User hoặc null
→ Service trả User hoặc ném NotFoundException
→ NestJS trả 200 hoặc 404
```

## Câu hỏi kiểm tra

1. `findUnique()` trả gì khi tìm thấy và khi không tìm thấy user?
2. Thành phần nào nên chuyển `null` thành HTTP `404`?
3. Vì sao `GET /users/:id` không cần `orderBy`?

## Câu trả lời của bạn

1. Tìm thấy thì trả một User object; không tìm thấy thì trả `null`.
2. Service xử lý việc chuyển kết quả `null` thành HTTP `404`.
3. Chỉ có tối đa một record được trả về nên không cần sắp xếp.

## Feedback / Đáp án đúng

### Đáp án 1: User object hoặc null - đúng

`findUnique()` không trả array. Nó trả đúng một object hoặc `null`.

### Đáp án 2: Service - đúng

Service hiểu use case và quyết định rằng user không tồn tại phải trở thành `NotFoundException`. Prisma không chịu trách nhiệm về HTTP.

### Đáp án 3: Không có danh sách để sắp xếp - đúng

`orderBy` không cần thiết khi query có tối đa một kết quả.

## Câu hỏi follow-up - Tái sử dụng select

Bạn hỏi: nếu cùng một `select` được copy trong POST, GET danh sách và GET theo id, có cách nào tránh sửa một nơi nhưng quên nơi khác không?

## Câu trả lời follow-up

Có thể tách các field chung thành một type-safe query fragment:

```ts
import type { Prisma } from '../../generated/prisma/client';

export const publicUserSelect = {
  id: true,
  email: true,
  name: true,
  createdAt: true,
  updatedAt: true,
} satisfies Prisma.UserSelect;
```

Sau đó tái sử dụng:

```ts
this.prisma.user.create({
  data,
  select: publicUserSelect,
});
```

```ts
this.prisma.user.findMany({
  select: publicUserSelect,
  orderBy: {
    createdAt: 'desc',
  },
});
```

```ts
this.prisma.user.findUnique({
  where: { id },
  select: publicUserSelect,
});
```

### Vai trò của `satisfies Prisma.UserSelect`

Nó giúp TypeScript:

- Phát hiện field không tồn tại trong Prisma model.
- Phát hiện cấu trúc hoặc giá trị select không hợp lệ.
- Giữ type inference cụ thể cho kết quả Prisma.

Ví dụ sai:

```ts
const userSelect = {
  invalidField: true,
} satisfies Prisma.UserSelect;
```

TypeScript sẽ báo lỗi vì `invalidField` không thuộc `UserSelect`.

### Khi các endpoint cần response khác nhau

Không nên ép mọi endpoint dùng một select nếu API contract khác nhau. Có thể dùng base select và select riêng theo use case:

```ts
export const publicUserSelect = {
  id: true,
  email: true,
  name: true,
} satisfies Prisma.UserSelect;

export const userDetailSelect = {
  ...publicUserSelect,
  createdAt: true,
  updatedAt: true,
} satisfies Prisma.UserSelect;
```

Nguyên tắc:

```text
Các endpoint có cùng response contract
→ dùng chung select

Các endpoint có response contract khác nhau
→ dùng base select + select riêng theo use case
```

## Câu hỏi follow-up kiểm tra

1. Nếu một field chỉ được trả ở `GET /users/:id`, nên thêm vào `publicUserSelect` hay `userDetailSelect`?
2. `satisfies Prisma.UserSelect` giúp phát hiện lỗi gì?

## Câu trả lời follow-up của bạn

1. Nên thêm field đó vào `userDetailSelect`.
2. Nó giúp phát hiện lỗi khi khai báo field không tồn tại trong `Prisma.UserSelect`.

## Feedback follow-up / Đáp án đúng

### Đáp án 1: `userDetailSelect` - đúng

Thêm vào select chung sẽ vô tình thay đổi response của những endpoint không cần field đó.

### Đáp án 2: Phát hiện select không khớp Prisma model - đúng

Ngoài tên field sai, `satisfies` còn kiểm tra cấu trúc select và giữ type inference chính xác.

## Trạng thái hiện tại

- `POST /users` đã được triển khai.
- `GET /users` đã được triển khai.
- `GET /users/:id` chưa được triển khai.
- `UsersService` hiện đang lặp cùng một `select` trong `create()` và `findAll()`.
- `publicUserSelect` và `userDetailSelect` mới là thiết kế đề xuất, chưa có trong source code.
- Chưa thêm xử lý `NotFoundException` cho user theo id.
- Prisma schema không thay đổi.

## Tóm tắt cuối

```text
Query theo id       → prisma.user.findUnique()
Điều kiện           → where: { id }
Kết quả             → User object hoặc null
Không tìm thấy      → Service ném NotFoundException
HTTP response       → 200 hoặc 404
Sắp xếp             → không cần orderBy
Select dùng chung   → publicUserSelect
Select riêng detail → userDetailSelect
Type safety         → satisfies Prisma.UserSelect
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo phù hợp là quyết định có tách `publicUserSelect` trước hay triển khai trực tiếp `GET /users/:id`. Chưa thực hiện cả hai thay đổi trong note này.
