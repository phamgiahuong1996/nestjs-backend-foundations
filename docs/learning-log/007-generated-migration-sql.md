# 007 - Generated migration SQL cho User model

## Câu hỏi của bạn

Show me where the generated migration is.

Explain the SQL that Prisma generated.

Do not modify it.

## Câu trả lời

Migration đầu tiên được generate ở đây:

```text
prisma/migrations/20260820033609_init_user/migration.sql
```

Folder migration là:

```text
prisma/migrations/20260820033609_init_user
```

Prisma cũng tạo file lock ở:

```text
prisma/migrations/migration_lock.toml
```

File `migration_lock.toml` ghi provider là PostgreSQL:

```toml
provider = "postgresql"
```

File này giúp Prisma biết migration history này thuộc loại database nào. Không nên sửa tay.

## SQL Prisma generated

```sql
-- CreateTable
CREATE TABLE "User" (
    "id" TEXT NOT NULL,
    "email" TEXT NOT NULL,
    "name" TEXT NOT NULL,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,

    CONSTRAINT "User_pkey" PRIMARY KEY ("id")
);

-- CreateIndex
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

## Giải thích SQL

### `CREATE TABLE "User"`

Tạo table thật trong PostgreSQL tên là `User`.

Prisma giữ đúng tên model nên table được quote bằng dấu `"User"`.

### `"id" TEXT NOT NULL`

Tạo column `id`.

Kiểu PostgreSQL là `TEXT` vì trong Prisma schema mình dùng `String`.

`NOT NULL` nghĩa là field này bắt buộc có giá trị.

### `CONSTRAINT "User_pkey" PRIMARY KEY ("id")`

Biến `id` thành primary key.

Đây là kết quả từ `@id` trong Prisma schema.

Primary key dùng để định danh duy nhất mỗi record trong table.

Một điểm đáng chú ý: SQL không có `DEFAULT uuid()` cho `id`.

Với `@default(uuid())`, Prisma thường tự sinh UUID ở Prisma Client layer khi create record, thay vì để PostgreSQL tự sinh bằng SQL default.

### `"email" TEXT NOT NULL`

Tạo column `email`, kiểu `TEXT`, bắt buộc có giá trị.

### `CREATE UNIQUE INDEX "User_email_key" ON "User"("email")`

Tạo unique index cho `email`.

Đây là kết quả từ `@unique` trong Prisma schema.

Database sẽ chặn việc insert hai user cùng email.

### `"name" TEXT NOT NULL`

Tạo column `name`, kiểu `TEXT`, bắt buộc có giá trị.

Không có unique index cho `name`, nên nhiều user có thể trùng tên.

### `"createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP`

Tạo column `createdAt`.

`TIMESTAMP(3)` lưu thời gian với độ chính xác milliseconds.

`DEFAULT CURRENT_TIMESTAMP` nghĩa là PostgreSQL tự set thời điểm hiện tại khi record được tạo nếu không truyền `createdAt`.

Đây là kết quả từ `@default(now())`.

### `"updatedAt" TIMESTAMP(3) NOT NULL`

Tạo column `updatedAt`, bắt buộc có giá trị.

SQL không có `DEFAULT` hoặc trigger cho `updatedAt`.

Lý do: `@updatedAt` là hành vi do Prisma Client xử lý, không phải PostgreSQL tự xử lý trong migration này.

Khi tạo hoặc update user qua Prisma Client, Prisma sẽ tự gửi giá trị `updatedAt` vào query.

Nếu update trực tiếp bằng SQL tay trong PostgreSQL mà không set `updatedAt`, field này sẽ không tự đổi.

## Tóm tắt annotation sang SQL

```text
@id             → PRIMARY KEY trong SQL
@unique         → UNIQUE INDEX trong SQL
@default(now()) → DEFAULT CURRENT_TIMESTAMP trong SQL
@updatedAt      → Prisma Client xử lý, không tạo trigger SQL
```

## Câu hỏi kiểm tra

1. `@unique` trong Prisma schema biến thành SQL gì trong migration này?
2. `createdAt` được PostgreSQL tự set hay Prisma tự set?
3. Vì sao `updatedAt` không thấy `DEFAULT` hoặc trigger trong SQL?

## Câu trả lời của bạn

1. `CREATE UNIQUE INDEX "User_email_key" ON "User"("email");`
2. PostgreSQL tự set.
3. Không biết.

## Feedback / Đáp án đúng

### 1. `@unique` biến thành SQL gì?

Đúng chính xác.

`@unique` được Prisma chuyển thành unique index:

```sql
CREATE UNIQUE INDEX "User_email_key" ON "User"("email");
```

Database dùng unique index này để đảm bảo email không bị trùng.

### 2. `createdAt` được PostgreSQL tự set hay Prisma tự set?

Đúng.

`createdAt` được PostgreSQL tự set vì migration SQL có:

```sql
DEFAULT CURRENT_TIMESTAMP
```

### 3. Vì sao `updatedAt` không thấy `DEFAULT` hoặc trigger trong SQL?

`updatedAt` không thấy `DEFAULT` hoặc trigger trong SQL vì `@updatedAt` là behavior do Prisma Client xử lý.

Nó không phải trigger tự động trong PostgreSQL.

Tóm tắt:

```text
createdAt @default(now())
→ PostgreSQL có DEFAULT CURRENT_TIMESTAMP
→ database tự set lúc insert

updatedAt @updatedAt
→ Prisma Client tự set khi create/update record
→ SQL migration không tạo trigger
```

## Câu hỏi kiểm tra tiếp theo

1. Nếu insert user trực tiếp bằng SQL mà không truyền `createdAt`, PostgreSQL có tự set không?
2. Nếu update user trực tiếp bằng SQL mà không set `updatedAt`, `updatedAt` có tự đổi không?
3. Nếu update user qua Prisma Client, `updatedAt` có tự đổi không?

## Câu trả lời của bạn

1. Có.
2. Không.
3. Có.

## Feedback / Đáp án đúng tiếp theo

Đúng hết.

`createdAt` được database tự set vì có `DEFAULT CURRENT_TIMESTAMP`.

`updatedAt` được Prisma tự set khi thao tác qua Prisma Client.

Database không tự set `updatedAt` nếu mình update bằng SQL tay.

## Câu hỏi cuối

1. Theo bạn, migration file nên sửa tay không?
2. Nếu muốn database tự update `updatedAt` kể cả khi update bằng SQL tay, mình cần thêm cơ chế gì ở PostgreSQL?
3. Trong project học CRUD này, mình có cần trigger cho `updatedAt` ngay bây giờ không?

## Câu trả lời của bạn

1. Không.
2. Không biết.
3. Không.

## Feedback / Đáp án đúng cuối

Đúng.

Migration file không nên sửa tay sau khi đã generated/applied, vì nó là lịch sử thay đổi database.

Nếu muốn PostgreSQL tự update `updatedAt` kể cả khi update bằng SQL tay, cần dùng database trigger.

Trigger là logic chạy tự động trong database khi có event như `INSERT`, `UPDATE`, `DELETE`.

Tuy nhiên, Week 1 chưa cần đi sâu vào trigger.

Trong project học CRUD này, mình chưa cần trigger cho `updatedAt` vì app sẽ thao tác database qua Prisma Client. `@updatedAt` là đủ cho bước hiện tại.

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là kiểm tra database thật sau migration hoặc bắt đầu kết nối Prisma Client vào NestJS theo từng bước nhỏ.
