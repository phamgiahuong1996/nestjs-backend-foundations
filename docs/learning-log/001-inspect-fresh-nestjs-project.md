# 001 - Inspect Fresh NestJS Project

## Câu hỏi của bạn

Inspect this fresh NestJS project.

Do not modify anything.

Explain:

1. What `main.ts` does
2. What `app.module.ts` does
3. What `app.controller.ts` does
4. What `app.service.ts` does
5. What happens when opening `http://localhost:3000`

Explain it for a senior frontend developer who is learning backend.

## Câu trả lời

### 1. `main.ts` làm gì?

`main.ts` là entry point của backend app.

Nếu frontend có `main.tsx` hoặc `index.tsx` để mount React app vào DOM, thì NestJS có `main.ts` để tạo và start HTTP server.

Trong project này, `main.ts` tạo Nest application từ `AppModule`, sau đó lắng nghe request ở port từ `process.env.PORT`, hoặc mặc định là `3000`.

```ts
const app = await NestFactory.create(AppModule);
await app.listen(process.env.PORT ?? 3000);
```

Nói ngắn gọn: `main.ts` là nơi app backend bắt đầu chạy.

### 2. `app.module.ts` làm gì?

`app.module.ts` là root module của app.

Hãy nghĩ nó như nơi Nest biết app này gồm những phần nào. Trong starter project:

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

`controllers: [AppController]` cho Nest biết controller nào có thể nhận HTTP request.

`providers: [AppService]` cho Nest biết service nào cần được tạo và quản lý để có thể inject vào class khác.

Điểm quan trọng: `AppModule` không tự động biết toàn bộ class trong project. Nó chỉ biết những gì được đăng ký trong module metadata.

### 3. `app.controller.ts` làm gì?

`app.controller.ts` là nơi định nghĩa HTTP route.

Trong frontend, bạn quen với route như `/users` hoặc `/settings`. Trong backend, controller map HTTP request vào method xử lý.

```ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

`@Controller()` nghĩa là controller này dùng root path.

`@Get()` nghĩa là method `getHello()` xử lý request `GET /`.

Controller không tự tạo message. Nó gọi `AppService` để lấy kết quả.

Mental model: controller là lớp nhận request và trả response, còn logic nên được đẩy sang service.

### 4. `app.service.ts` làm gì?

`app.service.ts` chứa logic của app.

Trong starter project, logic rất nhỏ:

```ts
@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

`@Injectable()` cho Nest biết class này có thể được quản lý bởi dependency injection system.

Backend mental model: controller nhận request, service xử lý logic.

### 5. Điều gì xảy ra khi mở `http://localhost:3000`?

Khi mở `http://localhost:3000`, browser gửi request:

```text
GET /
```

Flow:

```text
Browser
  -> GET http://localhost:3000/
  -> Nest server được start từ main.ts
  -> AppModule cho Nest biết AppController tồn tại
  -> AppController.getHello()
  -> AppService.getHello()
  -> "Hello World!"
  -> Browser hiển thị Hello World!
```

Điểm quan trọng: Nest không tạo route dựa trên file path như Next.js. Nest dùng decorators như `@Controller()` và `@Get()` để build routing table khi app start.

## Câu hỏi kiểm tra

1. Theo bạn, vì sao controller nên gọi service thay vì chứa toàn bộ logic trong controller?
2. `AppModule` đang biết những class nào trong app hiện tại?
3. Nếu đổi `@Controller()` thành `@Controller('api')`, URL nào sẽ trả về `Hello World!`?

## Câu trả lời của bạn

1. Giữ controller nhỏ, tách business logic với request nhận API.
2. Biết toàn bộ class.
3. Syntax sai.

## Feedback / Đáp án đúng

### 1. Vì sao controller nên gọi service?

Đúng.

Controller nên nhỏ vì nó là HTTP boundary: nhận request, đọc params/body/query, gọi service, và trả response.

Business logic nên nằm trong service để dễ test, dễ tái sử dụng, và không bị dính chặt vào HTTP.

### 2. `AppModule` biết những class nào?

Gần đúng, nhưng cần nói chính xác hơn.

`AppModule` không tự động biết toàn bộ class trong project. Nó chỉ biết những gì được đăng ký trong metadata của module.

Hiện tại `AppModule` biết:

```ts
controllers: [AppController],
providers: [AppService],
```

### 3. `@Controller('api')` có sai syntax không?

Không sai. `@Controller('api')` là syntax đúng trong NestJS.

Nếu đổi:

```ts
@Controller()
```

thành:

```ts
@Controller('api')
```

thì route `@Get()` bên trong sẽ thành:

```text
GET /api
```

Vì controller prefix là `api`, còn method `@Get()` không có path con.

## Câu hỏi follow-up

1. Nếu có `@Controller('users')` và `@Get(':id')`, URL sẽ trông như thế nào?
2. Theo bạn, `providers: [AppService]` giúp Nest làm được việc gì khi controller cần service?

## Câu trả lời follow-up của bạn

1. `/user/:id`
2. Giúp Nest hiểu được controller đó có service nào, ở đây là `appService`.

## Feedback follow-up / Đáp án đúng

### 1. Route với `@Controller('users')` và `@Get(':id')`

Nếu code là:

```ts
@Controller('users')
@Get(':id')
```

thì URL là:

```text
/users/:id
```

Có `s` vì prefix là `'users'`, không phải `'user'`.

### 2. `providers: [AppService]` giúp gì?

Ý đúng, nhưng nói chính xác hơn:

`providers: [AppService]` cho Nest biết cách tạo và quản lý instance của `AppService`.

Sau đó khi `AppController` khai báo:

```ts
constructor(private readonly appService: AppService) {}
```

Nest sẽ tự inject instance đó vào controller.