# 010 - UsersModule, UsersController, UsersService và dependency injection

## Câu hỏi của bạn

Bạn yêu cầu chỉ tạo ba file tối thiểu, chưa triển khai CRUD:

- `users.module.ts`
- `users.controller.ts`
- `users.service.ts`

Sau đó bạn muốn giải thích:

1. Vì sao `UsersController` dùng `@Controller('users')`?
2. Vì sao `UsersService` dùng `@Injectable()`?
3. `UsersModule` kết nối controller và service như thế nào?
4. Dependency injection cho phép controller sử dụng service như thế nào?

## Câu trả lời

### Trạng thái code đã tạo

`src/users/users.module.ts`:

```ts
import { Module } from '@nestjs/common';
import { UsersController } from './users.controller';
import { UsersService } from './users.service';

@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

`src/users/users.controller.ts`:

```ts
import { Controller } from '@nestjs/common';
import { UsersService } from './users.service';

@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
}
```

`src/users/users.service.ts`:

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {}
```

Chưa có CRUD, Prisma hoặc route handler. `UsersModule` cũng chưa được import vào `AppModule`, nên module này chưa được NestJS khởi tạo khi ứng dụng chạy. Kiểm tra lint riêng ba file đã thành công.

### 1. `@Controller('users')`

`@Controller()` đánh dấu class là controller xử lý HTTP request. Chuỗi `'users'` là route prefix:

```text
@Controller('users') → /users
```

Nếu sau này có `@Post()` thì route hoàn chỉnh là `POST /users`. Hiện tại chưa có `@Get()`, `@Post()` hoặc handler khác nên gọi `/users` sẽ nhận `404 Not Found`.

### 2. `@Injectable()`

`@Injectable()` đánh dấu `UsersService` là provider có thể được NestJS DI container quản lý.

NestJS có thể:

- Tạo instance của `UsersService`.
- Inject instance đó vào controller hoặc provider khác.
- Cung cấp dependency cho chính `UsersService` trong tương lai.
- Quản lý vòng đời của instance.

### 3. `UsersModule` kết nối controller và service

```ts
@Module({
  controllers: [UsersController],
  providers: [UsersService],
})
```

Ý nghĩa:

```text
controllers → các controller thuộc module
providers   → các dependency DI container có thể cung cấp trong module
```

Khi NestJS khởi tạo `UsersModule`, framework biết `UsersController` thuộc module và `UsersService` là provider có thể được inject trong module đó.

### 4. Constructor injection

```ts
constructor(private readonly usersService: UsersService) {}
```

Controller khai báo rằng nó cần một `UsersService`. NestJS tìm provider tương ứng, tạo hoặc lấy instance rồi truyền vào constructor.

Mental model:

```text
NestJS đọc UsersController
→ thấy constructor cần UsersService
→ tìm UsersService trong providers của UsersModule
→ tạo hoặc lấy instance UsersService
→ truyền instance vào UsersController
→ controller dùng this.usersService
```

Về ý tưởng, NestJS tự động làm công việc tương tự:

```ts
const usersService = new UsersService();
const usersController = new UsersController(usersService);
```

Controller không tự gọi `new UsersService()` vì như vậy sẽ bỏ qua DI container, khó thay thế dependency khi test và khó quản lý các dependency bên trong service.

## Câu hỏi kiểm tra

1. Vì sao `UsersService` nằm trong `providers`, còn `UsersController` nằm trong `controllers`?
2. Vì chưa import `UsersModule` vào `AppModule`, module này đã hoạt động chưa?
3. Hiện tại gọi `POST /users` có thành công không?

## Câu trả lời của bạn

Qua phần giải thích và các câu hỏi tiếp theo, bạn đã xác nhận:

- Xóa `UsersService` khỏi `providers` thì NestJS không thể inject service vào controller.
- Chưa có route handler thì route sẽ trả về `404 Not Found`.
- Controller không cần tự tạo service vì NestJS DI đã tạo và inject instance.

## Feedback / Đáp án đúng

Các ý trên đều đúng.

Nếu dependency không được đăng ký hoặc không khả dụng trong DI context, NestJS không thể tạo controller và thường báo lỗi không resolve được dependency.

`@Controller('users')` chỉ khai báo route prefix; nó chưa tạo endpoint nếu không có method được gắn decorator HTTP như `@Get()` hoặc `@Post()`.

## Câu hỏi follow-up - providers, exports và imports

1. Khi nào cần thêm `UsersService` vào `exports`?
2. Nếu `AdminModule` muốn inject `UsersService`, cần cấu hình gì?
3. Điều gì xảy ra nếu import trực tiếp service thay vì module?

## Câu trả lời follow-up của bạn

- Chỉ cần export `UsersService` khi module khác muốn sử dụng.
- `UsersModule` phải export `UsersService`; `AdminModule` phải import `UsersModule`.
- Chỉ TypeScript import service không tự đăng ký service với NestJS DI container.
- Đăng ký trực tiếp `UsersService` trong nhiều module có thể tạo nhiều instance.

## Feedback follow-up / Đáp án đúng

Mental model:

```text
providers → provider module hiện tại sở hữu và có thể sử dụng
exports   → provider module khác được phép sử dụng
imports   → module cung cấp các provider đã export
```

Ví dụ chia sẻ `UsersService`:

```ts
@Module({
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

Module sử dụng phải import `UsersModule`, không đặt `UsersService` vào `imports`:

```ts
@Module({
  imports: [UsersModule],
})
export class AdminModule {}
```

Luồng chia sẻ provider:

```text
UsersModule sở hữu UsersService trong providers
→ UsersModule công khai UsersService qua exports
→ AdminModule import UsersModule
→ AdminModule có thể inject UsersService
```

### Phân biệt ba cách “import service”

#### Chỉ dùng TypeScript import

```ts
import { UsersService } from '../users/users.service';
```

Dòng này chỉ đưa tên class vào phạm vi của file. Nó không đăng ký service với DI container.

#### Đặt service vào `imports`

```ts
@Module({
  imports: [UsersService],
})
```

Cách này sai vì `imports` dành cho module. NestJS sẽ cố xử lý `UsersService` như module và có thể báo lỗi khi khởi động.

#### Đặt lại service vào `providers`

```ts
@Module({
  providers: [UsersService],
})
```

Cách này có thể tạo một provider thuộc module mới, nhưng gây lặp cấu hình và có thể tạo thêm instance:

```text
UsersModule → UsersService instance A
AdminModule → UsersService instance B
```

Nó cũng buộc module mới tự đáp ứng toàn bộ dependency của `UsersService`, làm mất ranh giới sở hữu giữa các module.

## Câu hỏi follow-up 2 - Dependency bên trong service

1. Nếu `UsersService` cần `PrismaService`, module nào nên import `PrismaModule`?
2. `AdminModule` có cần import thêm `PrismaModule` chỉ để sử dụng `UsersService` không?

## Câu trả lời follow-up 2 của bạn

1. `PrismaModule` nên được import vào `UsersModule`.
2. `AdminModule` không cần import thêm `PrismaModule` vì `UsersModule` đã quản lý dependency nội bộ đó.

## Feedback follow-up 2 / Đáp án đúng

Cả hai câu đều đúng.

Giả sử `PrismaModule` cung cấp và export `PrismaService`:

```text
PrismaModule
├── providers: [PrismaService]
└── exports: [PrismaService]
          ↓
UsersModule imports PrismaModule
          ↓
UsersService injects PrismaService
```

Nếu `AdminModule` chỉ cần `UsersService`, nó chỉ cần import `UsersModule`:

```text
AdminModule
→ import UsersModule
   → import PrismaModule
      → cung cấp PrismaService cho UsersService
```

Đây là module encapsulation: module sử dụng chỉ cần biết provider được export, không cần tự cấu hình các dependency nội bộ của provider đó.

## Tóm tắt cuối

```text
@Controller('users') → khai báo controller và route prefix /users
@Injectable()        → cho phép NestJS quản lý UsersService như provider
providers            → đăng ký provider trong module
exports              → công khai provider cho module khác
imports              → nhập module cung cấp provider đã export
constructor          → khai báo dependency cần được NestJS inject
```

Nguyên tắc tổ chức:

```text
Một module sở hữu provider
→ export provider khi module khác cần dùng
→ module khác import module sở hữu
→ không đăng ký lặp lại provider ở nhiều module
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo phù hợp là import `UsersModule` vào `AppModule` để NestJS thực sự khởi tạo module, nhưng chưa cần thêm CRUD.
