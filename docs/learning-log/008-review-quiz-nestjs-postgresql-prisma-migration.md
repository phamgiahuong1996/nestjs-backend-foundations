# 008 - Quiz ôn tập NestJS, PostgreSQL, Prisma và migration

## Câu hỏi của bạn

Quiz me with 5 short question about today work:

- NestJS
- PostgreSQL
- Prisma
- Prisma model
- migration

Ask one question at a time.

Do not give me the answer until I answer.

## Câu trả lời

Mình đã hỏi từng câu một, chờ bạn trả lời rồi mới feedback.

## Câu hỏi kiểm tra 1 - NestJS

Trong NestJS, controller nên chịu trách nhiệm chính việc gì: xử lý business logic hay nhận request/trả response?

## Câu trả lời của bạn

Nhận request, trả response.

## Feedback / Đáp án đúng 1

Đúng.

Controller chủ yếu nhận request, đọc params/body/query khi cần, rồi trả response.

Business logic nên để trong service để controller nhỏ và dễ maintain.

## Câu hỏi kiểm tra 2 - PostgreSQL

PostgreSQL đóng vai trò gì trong kiến trúc `NestJS → Prisma → PostgreSQL`?

## Câu trả lời của bạn

Database thật sự.

## Feedback / Đáp án đúng 2

Đúng.

PostgreSQL là database thật sự, nơi table và data thật được lưu lâu dài.

## Câu hỏi kiểm tra 3 - Prisma

Prisma có phải là database không? Nếu không, nó đóng vai trò gì?

## Câu trả lời của bạn

Prisma không phải database. Nó đóng vai trò là tool cung cấp API thao tác với database thật.

## Feedback / Đáp án đúng 3

Đúng.

Prisma không phải database.

Prisma là tool/ORM giúp TypeScript code giao tiếp với database thật bằng API có type, thay vì viết SQL tay cho mọi thao tác.

## Câu hỏi kiểm tra 4 - Prisma model

Trong model `User`, vì sao `email` nên có `@unique` còn `name` thì không?

## Câu trả lời của bạn

Email nên là duy nhất cho một user, còn name thì có thể trùng trong một real system.

## Feedback / Đáp án đúng 4

Đúng.

`email` thường dùng để định danh user nên cần unique.

`name` chỉ là thông tin hiển thị. Ngoài đời nhiều người có thể trùng tên.

## Câu hỏi kiểm tra 5 - Migration

Migration khác gì với việc chỉ sửa `schema.prisma`?

## Câu trả lời của bạn

Migration biến bản thiết kế trong `schema.prisma` thành dữ liệu thật sự trong PostgreSQL.

## Feedback / Đáp án đúng 5

Gần đúng, nhưng cần chỉnh một từ quan trọng.

Migration biến bản thiết kế trong `schema.prisma` thành cấu trúc database thật trong PostgreSQL, ví dụ:

```text
- table
- columns
- indexes
- constraints
```

Migration không biến thành dữ liệu thật sự.

Dữ liệu là records bên trong table, ví dụ user A, user B.

Câu chuẩn:

```text
schema.prisma → bản thiết kế trong code
migration     → áp dụng thiết kế đó thành table/columns/constraints thật trong PostgreSQL
data          → records thật nằm trong table
```

## Tóm tắt cuối

Bạn đã nắm tốt các ý chính hôm nay:

```text
NestJS controller → nhận request và trả response
PostgreSQL        → database thật sự
Prisma            → tool/ORM để code giao tiếp với database
Prisma model      → bản mô tả shape dữ liệu trong code
Migration         → áp dụng schema thành cấu trúc database thật
```

Điểm cần nhớ nhất là phân biệt cấu trúc database với dữ liệu database.

```text
Cấu trúc database → table, columns, indexes, constraints
Dữ liệu database  → records thật nằm trong table
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là tiếp tục sau migration: chạy `prisma generate`, kiểm tra Prisma Client, rồi tạo `PrismaService` trong NestJS.
