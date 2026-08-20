# 005 - Database migration trong Prisma

## Câu hỏi của bạn

Before running a migration, explain to me:

What is a database migration?

What is the difference between:

- changing `schema.prisma`
- running `prisma migrate`
- running `prisma generate`

Do not run anything yet.

## Câu trả lời

Migration là lịch sử thay đổi database.

Khi mình thay đổi model trong `schema.prisma`, mình mới chỉ thay đổi bản thiết kế trong code. PostgreSQL chưa tự biết chuyện đó.

Migration là bước tạo ra và áp dụng các câu lệnh SQL để biến database thật khớp với bản thiết kế đó.

Ví dụ nếu đã thêm model `User` trong `schema.prisma` nhưng chưa chạy migration, PostgreSQL vẫn chưa có table `User`.

### 1. Thay đổi `schema.prisma`

`schema.prisma` là file thiết kế dữ liệu ở phía code.

Nó nói với Prisma rằng hệ thống có model nào, field nào, kiểu dữ liệu gì, field nào unique, field nào có default value.

Sửa `schema.prisma` giống như sửa bản vẽ thiết kế. Bản vẽ đã thay đổi, nhưng database thật chưa tự thay đổi.

### 2. Chạy `prisma migrate`

`prisma migrate` là bước biến thiết kế trong `schema.prisma` thành thay đổi thật trong PostgreSQL.

Nó thường làm hai việc:

1. Tạo migration file để ghi lại thay đổi.
2. Áp dụng SQL vào database thật.

Sau khi migrate, PostgreSQL mới thật sự có table `User`, các cột tương ứng, và constraint như unique email.

### 3. Chạy `prisma generate`

`prisma generate` tạo Prisma Client cho TypeScript code dùng.

Nó không tạo table trong database.

Nó đọc `schema.prisma` rồi tạo code client để sau này NestJS service có thể gọi database bằng Prisma, ví dụ:

```ts
prisma.user.findMany()
prisma.user.create()
```

Tóm tắt:

```text
schema.prisma   → bản thiết kế
prisma migrate  → áp dụng thiết kế vào PostgreSQL thật
prisma generate → tạo TypeScript client để code gọi database
```

Flow thường gặp:

```text
Sửa schema.prisma
→ Chạy prisma migrate
→ Prisma tạo/cập nhật table trong PostgreSQL
→ Prisma generate client
→ NestJS service dùng Prisma Client để query database
```

## Câu hỏi kiểm tra

1. Nếu chỉ sửa `schema.prisma` mà chưa migrate, PostgreSQL đã có table mới chưa?
2. `prisma generate` có tạo table trong database không?
3. Migration giúp ích gì khi làm việc team hoặc deploy lên server?

## Câu trả lời của bạn

1. Chưa.
2. Không. Nó tạo TypeScript model client.
3. Đồng bộ dữ liệu mọi project và môi trường, không phải tự tạo database bằng tay.

## Feedback / Đáp án đúng

### 1. Nếu chỉ sửa `schema.prisma` mà chưa migrate, PostgreSQL đã có table mới chưa?

Đúng.

Chỉ sửa `schema.prisma` thì PostgreSQL chưa có table mới.

### 2. `prisma generate` có tạo table trong database không?

Đúng.

`prisma generate` không tạo table. Nó tạo Prisma Client/TypeScript API để code gọi database dễ hơn và có type.

### 3. Migration giúp ích gì khi làm việc team hoặc deploy lên server?

Đúng ý, nhưng cần chỉnh thuật ngữ một chút.

Migration đồng bộ cấu trúc database, còn gọi là database schema. Nó không đồng bộ dữ liệu thật bên trong table.

Migration quản lý:

```text
- tạo table User
- thêm column email
- thêm unique constraint cho email
```

Migration không tự đồng bộ:

```text
- user A
- user B
- data production
```

## Câu hỏi kiểm tra tiếp theo

1. Nếu migration tạo table `User`, dữ liệu user thật nằm ở đâu?
2. Nếu đổi `schema.prisma` rồi quên chạy migration, lỗi có thể xảy ra ở bước nào?
3. Theo bạn, vì sao migration file nên được commit vào source code?

## Câu trả lời của bạn

1. Nằm trong chính table đó.
2. Bước truy xuất cơ sở dữ liệu.
3. Để người khác có thể sử dụng.

## Feedback / Đáp án đúng tiếp theo

Đúng hướng.

Dữ liệu user thật nằm trong table `User` trong PostgreSQL.

Nếu đổi `schema.prisma` rồi quên chạy migration, lỗi thường xảy ra khi code truy xuất database. Ví dụ Prisma nghĩ có model/table đó, nhưng PostgreSQL thật chưa có table hoặc column tương ứng.

Migration file nên được commit để người khác, CI/CD, staging, production có thể áp dụng cùng một lịch sử thay đổi database.

Nói chuẩn hơn: migration không chỉ để người khác sử dụng, mà để toàn bộ team và mọi môi trường có cùng cấu trúc database.

## Tóm tắt cuối

```text
schema.prisma  → thiết kế trong code
migration file → lịch sử thay đổi database, nên commit
PostgreSQL     → nơi table và data thật tồn tại
generate       → tạo Prisma Client cho TypeScript
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là chuẩn bị PostgreSQL rồi chạy migration đầu tiên cho `User`.
