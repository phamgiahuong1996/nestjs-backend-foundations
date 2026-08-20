# 006 - PostgreSQL local bằng Docker Compose và biến môi trường

## Câu hỏi của bạn

Dùng gì để run PostgreSQL chuẩn production?

Sau đó bạn chọn học trước phần local development.

Bạn cũng hỏi thêm:

- Vì sao cần chắc PostgreSQL đang chạy trước khi migration?
- Ghi password trực tiếp vào `docker-compose.yml` rồi commit lên git có sao không?
- `DATABASE_URL` dùng làm gì?
- Vì sao cần `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` rồi vẫn cần `DATABASE_URL`?

## Câu trả lời

Với local development, cách gọn và đúng cho project học này là dùng Docker Compose để chạy PostgreSQL.

Flow local là:

```text
NestJS app
→ Prisma
→ PostgreSQL container
```

Trong setup này:

- NestJS chạy trên máy local.
- Prisma là tool để code TypeScript giao tiếp với PostgreSQL.
- PostgreSQL chạy trong Docker container.
- Docker Compose giúp start/stop database dễ hơn và giữ cấu hình database trong project.

## Docker Compose PostgreSQL local

File `docker-compose.yml` hiện tại tạo một service PostgreSQL:

```yml
services:
  postgres:
    image: postgres:16-alpine
    container_name: nestjs-backend-foundations-postgres
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: nestjs_backend_foundations
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Ý nghĩa chính:

`services` khai báo các container mà Docker Compose quản lý.

`postgres` là tên service PostgreSQL trong project.

`image: postgres:16-alpine` dùng PostgreSQL version 16, bản Alpine nhẹ hơn, phù hợp local development.

`container_name` đặt tên container rõ ràng để dễ nhìn khi chạy `docker ps`.

`restart: unless-stopped` giúp container tự restart nếu bị tắt bất ngờ, trừ khi mình chủ động stop.

`ports: "5432:5432"` map port giữa máy local và container:

```text
localhost:5432 trên máy mình
→ port 5432 trong PostgreSQL container
```

Nhờ vậy NestJS/Prisma trên máy local có thể kết nối tới PostgreSQL qua `localhost:5432`.

`environment` truyền biến môi trường cho PostgreSQL container khi khởi tạo lần đầu.

`POSTGRES_DB` là tên database được tạo.

`POSTGRES_USER` là username để login vào database.

`POSTGRES_PASSWORD` là password của user đó.

`volumes` giúp lưu data PostgreSQL ra Docker volume.

`postgres_data:/var/lib/postgresql/data` nghĩa là data thật của PostgreSQL được lưu trong named volume `postgres_data`, không chỉ nằm tạm bên trong container.

Nếu không có volume, data local có thể mất khi container bị xóa hoặc recreate.

## Password trong docker-compose.yml

Với local development, `postgres/postgres` là chấp nhận được nếu database chỉ chạy ở máy local và không chứa dữ liệu thật/nhạy cảm.

Nhưng nếu commit lên git, cách tốt hơn là không hardcode password trong `docker-compose.yml`.

Thay vào đó, compose nên đọc từ `.env`:

```yml
environment:
  POSTGRES_DB: ${POSTGRES_DB}
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
```

File `.env` local có thể chứa:

```env
POSTGRES_DB=nestjs_backend_foundations
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/nestjs_backend_foundations?schema=public"
```

`.env` thật không nên commit nếu chứa secret.

Project thật thường commit `.env.example` làm template, còn `.env` nằm trong `.gitignore`.

```text
.env          → local secret, không commit
.env.example  → template, có thể commit
```

## DATABASE_URL dùng làm gì?

`DATABASE_URL` là connection string để Prisma/NestJS kết nối vào PostgreSQL.

Ví dụ:

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/nestjs_backend_foundations?schema=public"
```

Tách ra:

```text
postgresql://                → protocol
postgres                     → username
postgres                     → password
localhost                    → host
5432                         → port
nestjs_backend_foundations   → database name
schema=public                → PostgreSQL schema
```

`POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` dùng cho Docker container PostgreSQL khi nó khởi tạo database lần đầu.

```text
Docker Compose
→ tạo PostgreSQL container
→ dùng POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD
→ tạo database + user ban đầu
```

`DATABASE_URL` dùng cho Prisma/NestJS để kết nối vào database đã có.

```text
NestJS / Prisma
→ đọc DATABASE_URL
→ connect tới PostgreSQL
```

Tóm tắt:

```text
POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD
→ cho container PostgreSQL biết phải tạo database/user gì

DATABASE_URL
→ cho app biết phải kết nối tới database nào
```

`DATABASE_URL` bị lặp lại thông tin từ các biến PostgreSQL, nhưng đây là convention phổ biến vì Prisma chỉ cần một biến đầy đủ để kết nối database.

## Câu hỏi kiểm tra

1. Vì sao cần `ports: "5432:5432"`?
2. Nếu không có `postgres_data` volume, data local có thể gặp rủi ro gì?
3. `POSTGRES_DB` khác gì với `POSTGRES_USER`?

## Câu trả lời của bạn

1. Mapping port máy local với Docker.
2. Không biết.
3. `POSTGRES_DB` là tên, còn user là username để login.

## Feedback / Đáp án đúng

### 1. Vì sao cần `ports: "5432:5432"`?

Đúng.

Nó map port từ máy local vào container. Nhờ vậy app NestJS/Prisma trên máy có thể kết nối qua `localhost:5432`.

### 2. Nếu không có `postgres_data` volume, data local có thể gặp rủi ro gì?

Nếu không có volume, data có thể mất khi container bị xóa hoặc recreate.

Container nên được xem là thứ có thể tạo lại được, còn data nên nằm ở volume riêng.

### 3. `POSTGRES_DB` khác gì với `POSTGRES_USER`?

Đúng.

`POSTGRES_DB` là tên database được tạo.

`POSTGRES_USER` là username dùng để login vào database đó.

## Câu hỏi kiểm tra tiếp theo

1. Vì sao password production không nên hardcode trong `docker-compose.yml`?
2. `.env` nên commit lên git không nếu chứa secret thật?
3. Docker Compose dùng `${POSTGRES_PASSWORD}` để lấy giá trị từ đâu?

## Câu trả lời của bạn

1. Lộ password.
2. Lộ password, người khác thấy.
3. Lấy từ `.env`.

## Feedback / Đáp án đúng tiếp theo

Đúng cả ba câu.

Password production không nên hardcode vì dễ bị lộ khi commit lên git hoặc chia sẻ source code.

`.env` chứa secret thật thì không nên commit.

Docker Compose có thể đọc `${POSTGRES_PASSWORD}` từ biến môi trường hoặc từ file `.env` cùng thư mục với compose file.

## Câu hỏi về DATABASE_URL

Bạn hỏi:

`DATABASE_URL` dùng làm gì, sao cần ba biến kia rồi còn cần database URL?

## Câu trả lời về DATABASE_URL

`DATABASE_URL` dùng để Prisma/NestJS kết nối vào database.

Nó chứa đủ thông tin: username, password, host, port, database name và schema.

Ba biến `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` là input cho PostgreSQL container lúc khởi tạo.

`DATABASE_URL` là input cho Prisma/NestJS lúc app cần kết nối vào database.

## Câu hỏi kiểm tra về DATABASE_URL

1. `POSTGRES_PASSWORD` dùng khi container PostgreSQL khởi tạo hay khi Prisma generate TypeScript client?
2. `DATABASE_URL` được Prisma dùng để làm gì?
3. Vì sao `DATABASE_URL` có thể chứa lại username/password/database name?

## Câu trả lời của bạn

1. Dùng khi container khởi tạo.
2. Connect đến database URL từ code.
3. Vì nó cần credential để access database.

## Feedback / Đáp án đúng về DATABASE_URL

Đúng.

`POSTGRES_PASSWORD` dùng khi PostgreSQL container khởi tạo user/database ban đầu.

`DATABASE_URL` được Prisma dùng để biết phải connect tới database nào.

`DATABASE_URL` chứa credential vì Prisma cần username/password để access database.

Tóm tắt chuẩn:

```text
Docker Compose env:
POSTGRES_DB / POSTGRES_USER / POSTGRES_PASSWORD
→ cấu hình PostgreSQL container

Prisma env:
DATABASE_URL
→ connection string để Prisma connect vào PostgreSQL
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là chỉnh `docker-compose.yml` để dùng biến từ `.env`, thay vì hardcode password trong file compose.
