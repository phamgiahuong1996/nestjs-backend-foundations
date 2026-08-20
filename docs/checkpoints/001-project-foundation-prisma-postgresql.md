# 001 - Project foundation, Prisma và PostgreSQL

## 1. Learning notes included

Checkpoint này bao gồm các note chưa từng được checkpoint trước đó:

1. [001 - Inspect Fresh NestJS Project](../learning-log/001-inspect-fresh-nestjs-project.md)
2. [002 - Kiến trúc NestJS, Prisma và PostgreSQL](../learning-log/002-nestjs-prisma-postgresql-architecture.md)
3. [003 - Thiết kế field cho User model](../learning-log/003-user-model-fields.md)
4. [004 - Giải thích cú pháp Prisma cho User model](../learning-log/004-user-model-prisma-syntax.md)
5. [005 - Database migration trong Prisma](../learning-log/005-database-migration-prisma.md)
6. [006 - PostgreSQL local bằng Docker Compose và biến môi trường](../learning-log/006-local-postgresql-docker-compose-env.md)
7. [007 - Generated migration SQL cho User model](../learning-log/007-generated-migration-sql.md)
8. [008 - Quiz ôn tập NestJS, PostgreSQL, Prisma và migration](../learning-log/008-review-quiz-nestjs-postgresql-prisma-migration.md)

Không có checkpoint cũ trong `docs/checkpoints/` tại thời điểm tạo file này, nên toàn bộ note `001` đến `008` được xem là phạm vi đầu tiên.

## 2. What has been completed

Đã scaffold project NestJS TypeScript cơ bản và inspect các file nền tảng:

- `src/main.ts`: entry point, bootstrap Nest app từ `AppModule` và start HTTP server.
- `src/app.module.ts`: root module, đăng ký controller và provider.
- `src/app.controller.ts`: nhận HTTP request, map route bằng decorator.
- `src/app.service.ts`: chứa logic đơn giản được controller gọi.

Đã hiểu flow backend cơ bản:

```text
Client / Browser
→ HTTP request
→ NestJS Controller
→ NestJS Service
→ Response
```

Đã học kiến trúc kết nối database:

```text
NestJS
→ Prisma
→ PostgreSQL
```

Trong đó:

- NestJS là tầng API/application.
- Prisma là tool/ORM giúp TypeScript code giao tiếp với database.
- PostgreSQL là database thật, nơi lưu table và data lâu dài.

Đã thiết kế và thêm Prisma `User` model với các field:

```prisma
model User {
  id        String   @id @default(uuid())
  email     String   @unique
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

Đã học ý nghĩa của `@id`, `@default`, `@unique`, `@updatedAt`, migration, Prisma Client, `DATABASE_URL`, Docker Compose local PostgreSQL và SQL migration do Prisma tạo ra.

## 3. Current verified project state

Project hiện tại có NestJS starter files trong `src/`.

Prisma đã được cấu hình với PostgreSQL trong `prisma/schema.prisma`:

```prisma
datasource db {
  provider = "postgresql"
}
```

`User` model đã tồn tại trong `prisma/schema.prisma`.

Migration đầu tiên đã tồn tại tại:

```text
prisma/migrations/20260820033609_init_user/migration.sql
```

Migration SQL hiện tại tạo:

- table `"User"`
- primary key trên `"id"`
- unique index trên `"email"`
- default timestamp cho `"createdAt"`

Project cũng có `docker-compose.yml` cho PostgreSQL local development.

Thư mục `generated/prisma/` đang tồn tại trong project, nghĩa là Prisma Client files đã có trong workspace. Tuy nhiên checkpoint này không chạy lại `prisma generate` và không xác minh database live connection.

## 4. What is ready

Đã sẵn sàng để tiếp tục học bước kết nối Prisma vào NestJS theo hướng nhỏ:

- hiểu NestJS controller/service/module ở mức nền tảng
- hiểu PostgreSQL là database thật
- hiểu Prisma không phải database
- có `User` model trong Prisma schema
- có migration SQL đầu tiên cho table `User`
- có nền tảng để tạo `PrismaService` sau này

## 5. What is not ready yet

Chưa có User CRUD REST API.

Chưa có:

- `UserModule`
- `UserController`
- `UserService`
- DTO validation
- error handling cho User API
- pagination
- search
- sorting
- authentication

Authentication không thuộc Week 1.

Checkpoint này cũng chưa xác minh PostgreSQL container hiện đang chạy hay database thật đã apply migration thành công, vì yêu cầu hiện tại chỉ là documentation changes.

## 6. Important concepts learned

Controller nên nhỏ, chủ yếu nhận request và trả response. Business logic nên nằm trong service.

Module không tự biết toàn bộ class trong project. Module chỉ biết các class được đăng ký trong metadata như `controllers` và `providers`.

Prisma schema là bản thiết kế dữ liệu ở phía code.

Migration là lịch sử thay đổi giúp áp dụng schema thành cấu trúc database thật trong PostgreSQL, ví dụ table, columns, indexes và constraints.

Prisma Client là TypeScript API để code gọi database. `prisma generate` không tạo table.

PostgreSQL mới là nơi lưu dữ liệu thật. Prisma không lưu dữ liệu thay PostgreSQL.

`email` nên unique vì thường dùng để định danh user. `name` không unique vì nhiều người có thể trùng tên.

`createdAt` nên được tự set khi tạo record. `updatedAt` nên tự thay đổi khi update record qua Prisma Client.

`DATABASE_URL` là connection string để Prisma/NestJS kết nối tới PostgreSQL. Các biến `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` là input để PostgreSQL container khởi tạo database/user ban đầu.

Secret như password, token, connection string thật không nên hardcode trong source code hoặc committed config. Nên dùng environment variables và `.env.example` cho template.

## 7. Commands and files involved

Các command đã được học hoặc dùng trong giai đoạn này:

```bash
npx prisma validate
npx prisma migrate dev --name init_user
npx prisma generate
docker compose up -d
```

Các file quan trọng:

```text
src/main.ts
src/app.module.ts
src/app.controller.ts
src/app.service.ts
prisma/schema.prisma
prisma.config.ts
prisma/migrations/20260820033609_init_user/migration.sql
prisma/migrations/migration_lock.toml
docker-compose.yml
docs/learning-log.md
docs/learning-log/
```

Lưu ý: checkpoint này chỉ xác nhận các file đang tồn tại trong project. Nó không tự chạy command để xác minh runtime/database state.

## 8. Next smallest learning step

Bước nhỏ tiếp theo nên là: giải thích và tạo `PrismaService` trong NestJS.

Mục tiêu của bước đó là để NestJS có một service quản lý Prisma Client, sau này `UserService` có thể inject và gọi database một cách rõ ràng.

Chưa nên tạo User CRUD đầy đủ ngay. Sau `PrismaService`, bước tiếp theo mới nên là tạo `UserModule` rỗng hoặc tạo endpoint nhỏ đầu tiên theo từng phần.
