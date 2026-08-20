# 004 - Giải thích cú pháp Prisma cho User model

## Câu hỏi của bạn

Explain this User model line by line.

Especially explain:

- `@id`
- `@default`
- `@unique`
- `@updatedAt`

## Câu trả lời

Model này là bản mô tả bảng `User` cho Prisma hiểu. Nó chưa tự tạo bảng trong PostgreSQL cho tới khi chạy migration ở bước sau.

```prisma
model User {
```

Khai báo một model tên `User`.

Trong PostgreSQL, model này sẽ tương ứng với một table dùng để lưu user.

```prisma
id String @id @default(uuid())
```

`id` là field định danh của user.

`String` nghĩa là Prisma lưu giá trị này dưới dạng chuỗi.

`@id` đánh dấu field này là primary key. Mỗi record `User` phải có một `id` duy nhất để phân biệt với record khác.

`@default(uuid())` nghĩa là nếu khi tạo user mà không truyền `id`, Prisma sẽ tự sinh một UUID cho field này.

```prisma
email String @unique
```

`email` là field lưu email của user.

`String` nghĩa là email là chuỗi.

`@unique` không cho phép hai user có cùng email. Nếu đã có user với email đó, tạo thêm user khác cùng email sẽ bị lỗi.

```prisma
name String
```

`name` là tên user.

`String` nghĩa là tên là chuỗi.

Không có `@unique`, nên nhiều user có thể trùng tên.

```prisma
createdAt DateTime @default(now())
```

`createdAt` lưu thời điểm record được tạo.

`DateTime` là kiểu dữ liệu ngày giờ.

`@default(now())` nghĩa là nếu không truyền `createdAt`, Prisma/PostgreSQL sẽ tự lấy thời điểm hiện tại khi record được tạo.

```prisma
updatedAt DateTime @updatedAt
```

`updatedAt` lưu thời điểm record được cập nhật lần cuối.

`DateTime` là kiểu dữ liệu ngày giờ.

`@updatedAt` làm Prisma tự đổi giá trị field này mỗi khi record được update. Với CRUD cơ bản, mình không cần tự set thủ công field này.

```prisma
}
```

Kết thúc model `User`.

Tóm tắt:

```text
@id        → field này là khóa chính
@default   → tự tạo giá trị nếu mình không truyền vào
@unique    → giá trị không được trùng giữa các records
@updatedAt → tự cập nhật thời gian khi record thay đổi
```

## Câu hỏi kiểm tra

1. Nếu không có `@id`, Prisma có biết field nào là khóa chính của `User` không?
2. Nếu bỏ `@unique` ở `email`, chuyện gì có thể xảy ra?
3. Vì sao mình không muốn client tự gửi `createdAt` và `updatedAt`?

## Câu trả lời của bạn

1. Không.
2. Email không còn unique.
3. Sẽ không nhất quán thời điểm tạo record `createdAt` và `updatedAt`, nên tự sinh để chính xác thời điểm record update.

## Feedback / Đáp án đúng

### 1. Nếu không có `@id`, Prisma có biết field nào là khóa chính không?

Đúng.

Nếu không có `@id`, Prisma không biết field nào là primary key của model `User`.

### 2. Nếu bỏ `@unique` ở `email`, chuyện gì có thể xảy ra?

Đúng.

Database sẽ cho phép nhiều user dùng cùng một email. Với hệ thống user thật, điều này dễ gây lỗi khi tìm user theo email hoặc sau này làm authentication.

### 3. Vì sao không muốn client tự gửi `createdAt` và `updatedAt`?

Đúng.

`createdAt` và `updatedAt` nên do backend/database flow tự quản lý để thời gian nhất quán và đáng tin hơn. Client có thể sai giờ, cố tình gửi sai, hoặc mỗi client ở timezone khác nhau.

## Câu hỏi kiểm tra tiếp theo

1. `@default(uuid())` chạy khi nào: khi đọc user, khi tạo user, hay khi update user?
2. `@updatedAt` có đổi giá trị khi chỉ đọc user từ database không?
3. Theo bạn, bước tiếp theo nên là tạo migration hay viết NestJS service trước?

## Câu trả lời của bạn

1. Khi tạo user.
2. Không đổi.
3. Tạo migration.

## Feedback / Đáp án đúng tiếp theo

Đúng.

`@default(uuid())` chạy khi tạo record mới, nếu mình không truyền `id`.

`@updatedAt` không đổi khi chỉ đọc dữ liệu. Nó chỉ đổi khi update record.

Bước tiếp theo nên là tạo migration, vì hiện tại `schema.prisma` mới chỉ là bản thiết kế. Migration mới biến model `User` thành table thật trong PostgreSQL.

## Câu hỏi chuẩn bị cho migration

1. Migration khác gì với Prisma schema?
2. Vì sao không nên viết CRUD API trước khi database table thật sự tồn tại?
3. Nếu đổi model `User` sau này, mình nên sửa trực tiếp database hay tạo migration mới?

## Câu trả lời của bạn

1. Prisma schema là bản thiết kế. Migration giúp bản thiết kế đó thành table thật sự trong PostgreSQL.
2. Vì chưa có table, chưa query hoặc tạo data được.
3. Tạo migration mới.

## Feedback / Đáp án đúng cho migration

Đúng.

`schema.prisma` là bản thiết kế dữ liệu trong code.

Migration là lịch sử thay đổi để áp dụng bản thiết kế đó vào PostgreSQL thật.

PostgreSQL mới là nơi lưu data thật.

Nếu sau này đổi `User` model, nên tạo migration mới thay vì sửa tay trực tiếp database. Cách này giúp team biết database đã thay đổi thế nào, deploy nhất quán hơn, và debug dễ hơn.

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là kết nối PostgreSQL và chạy migration đầu tiên cho `User`.
