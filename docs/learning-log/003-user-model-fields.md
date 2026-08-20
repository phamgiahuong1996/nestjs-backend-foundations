# 003 - Thiết kế field cho User model

## Câu hỏi của bạn

I need a simple User model for learning CRUD.

Requirements:

- id
- email
- name
- createdAt
- updatedAt

Do not write the Prisma schema yet.

Explain:

1. What type each field should use
2. Which field should be unique
3. Which fields need default values
4. Why

## Câu trả lời

Với User model đơn giản cho CRUD, ta có thể nghĩ nó như shape của một row trong bảng `users`.

### 1. `id`

Nên dùng kiểu `String` nếu theo hướng phổ biến với Prisma và PostgreSQL là dùng UUID.

`id` là định danh duy nhất của mỗi user. UUID tiện cho backend vì không phụ thuộc vào số tăng dần như `1`, `2`, `3`, và ít lộ thông tin về số lượng records trong database.

### 2. `email`

Nên dùng kiểu `String`.

Field này nên là `unique`.

Email thường dùng để nhận diện user. Trong hệ thống bình thường, không nên có hai user dùng cùng một email, vì sau này khi update, search, hoặc authentication, email trùng sẽ gây nhập nhằng.

### 3. `name`

Nên dùng kiểu `String`.

Field này không cần unique.

`name` chỉ là thông tin hiển thị. Nhiều người có thể trùng tên, nên không nên dùng `name` làm định danh duy nhất.

### 4. `createdAt`

Nên dùng kiểu `DateTime`.

Field này cần default value là thời điểm record được tạo.

Khi tạo user mới, backend/database nên tự ghi lại thời điểm tạo, thay vì bắt client gửi lên. Client không đáng tin cho dữ liệu hệ thống kiểu này.

### 5. `updatedAt`

Nên dùng kiểu `DateTime`.

Field này cần tự động cập nhật mỗi khi record thay đổi.

Ta cần biết lần cuối user được sửa là khi nào. Giống `createdAt`, client không nên tự quyết định giá trị này.

Tóm tắt:

```text
id        → String, unique by nature, default UUID
email     → String, unique
name      → String
createdAt → DateTime, default now
updatedAt → DateTime, auto update when changed
```

## Câu hỏi kiểm tra

1. Vì sao `email` nên unique nhưng `name` không nên unique?
2. Vì sao `createdAt` không nên lấy từ request body của client?
3. `updatedAt` khác `createdAt` ở điểm nào?

## Câu trả lời của bạn

1. Email nên unique để không bị trùng. Trong một hệ thống, một user chỉ nên có một email để xác thực.
2. Vì client gửi lên không đáng tin. Nó nên tự sinh ra từ backend thì sẽ chính xác hơn.
3. `createdAt` là thời điểm lúc đầu tạo ra một lần duy nhất. `updatedAt` lưu thời gian record bị change, dễ dàng cho xử lý sau này.

## Feedback / Đáp án đúng

### 1. Vì sao `email` nên unique nhưng `name` không nên unique?

Đúng.

Một hệ thống thường nên xem email là định danh duy nhất của user. Đặc biệt sau này khi có authentication, email trùng sẽ làm hệ thống không biết login vào account nào.

`name` không nên unique vì nhiều người có thể trùng tên. `name` là thông tin hiển thị, không phải định danh.

### 2. Vì sao `createdAt` không nên lấy từ client?

Đúng.

Client không đáng tin cho dữ liệu hệ thống. Backend hoặc database nên tự sinh `createdAt` để thời gian thống nhất và khó bị giả mạo.

### 3. `updatedAt` khác `createdAt` ở điểm nào?

Đúng.

`createdAt` là thời điểm record được tạo lần đầu và thường không đổi.

`updatedAt` là thời điểm record được sửa gần nhất, nên sẽ thay đổi mỗi lần update.

Tóm tắt chuẩn:

```text
id        → định danh duy nhất, nên tự sinh
email     → unique, dùng để phân biệt user
name      → thông tin hiển thị, không unique
createdAt → tự sinh khi tạo record
updatedAt → tự cập nhật khi record thay đổi
```

## Ghi chú bước tiếp theo

Bước nhỏ tiếp theo hợp lý là học Prisma schema syntax cho model `User`, nhưng chưa implement cho tới khi có yêu cầu rõ ràng.
