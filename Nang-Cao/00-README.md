# C# & .NET Mastery — Track Nâng Cao

Dành cho người **đã biết code C#** nhưng muốn hiểu **sâu bản chất**: vùng nhớ hoạt động thế nào, tại sao ngôn ngữ thiết kế như vậy, và cách dựng hệ thống thực chiến với .NET, Entity Framework, Clean Architecture, Microservices.

Nguyên tắc trình bày xuyên suốt: mỗi chủ đề khó sẽ có **ví dụ trước, giải thích sau**. Ít lý thuyết suông, nhiều hình dung trực quan (bộ nhớ như kệ sách, GC như người dọn nhà...).

> Nếu bạn mới tiếp cận C#, nên học qua track [Co-Ban](../Co-Ban/00-README.md) trước khi vào đây.

## Cấu trúc

| File | Nội dung |
|---|---|
| [01-Memory-And-Runtime.md](01-Memory-And-Runtime.md) | Từ source code đến lúc chạy, Stack vs Heap, Value Type vs Reference Type, Boxing/Unboxing, Garbage Collector, String Immutability, Span\<T\>/Memory\<T\>/stackalloc |
| [02-OOP-And-Language-Features.md](02-OOP-And-Language-Features.md) | OOP sâu (Encapsulation/Abstraction/Access Modifiers/virtual/override/sealed), Generics, Extension Methods, Delegate/Event, Collections & Big-O, Pattern Matching, Record, var/dynamic/Reflection, Nullable Reference Types, LINQ |
| [03-Exception-And-Async.md](03-Exception-And-Async.md) | Exception Handling đúng cách, Threading chi tiết (Monitor/Mutex/Semaphore), Task, async/await, deadlock, cancellation |
| [04-DotNet-Platform.md](04-DotNet-Platform.md) | CLR/JIT chi tiết, Dependency Injection, Configuration, Middleware pipeline, ASP.NET Core, BackgroundService/IHostedService |
| [05-EntityFramework.md](05-EntityFramework.md) | EF Core: DbContext, Change Tracker, Migration, N+1, Transaction, Performance |
| [06-CleanArchitecture.md](06-CleanArchitecture.md) | Clean Architecture: tại sao, layer nào làm gì, CQRS, Mediator, Domain-Driven Design cơ bản |
| [07-Microservices.md](07-Microservices.md) | Microservices: khi nào nên dùng, giao tiếp (sync/async), Saga, Outbox, API Gateway, Observability |
| [08-DesignPatterns.md](08-DesignPatterns.md) | Design Patterns thực chiến: Strategy, Factory, Decorator, Unit of Work, Observer, Singleton |
| [09-UnitTesting-TDD.md](09-UnitTesting-TDD.md) | Unit Testing: AAA pattern, Mock/Stub/Fake, test gì/không test gì, chu trình TDD |
| [10-SignalR-Realtime.md](10-SignalR-Realtime.md) | SignalR & Real-time: WebSocket, Hub, Group, scale-out với Backplane |
| [11-Security-AuthN-AuthZ.md](11-Security-AuthN-AuthZ.md) | Security: Claims, JWT, Refresh Token, OAuth2/OIDC, Policy-based Authorization, Password Hashing |
| [12-SOLID-Principles.md](12-SOLID-Principles.md) | SOLID: Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion |
| [13-Tooling-And-Project-Structure.md](13-Tooling-And-Project-Structure.md) | Tooling: .csproj/.sln, cấu trúc nhiều project theo Clean Architecture, NuGet, dotnet CLI, appsettings theo môi trường |

## Lộ trình đọc đề xuất

1. Đọc file 01 → 03 trước: đây là **nền móng ngôn ngữ**, không hiểu phần này thì học framework chỉ là học thuộc lòng.
2. File 04 là cầu nối giữa ngôn ngữ C# và thế giới ứng dụng thực (web app, API).
3. File 05 → 07 là kiến trúc hệ thống, nên đọc sau khi đã chắc 01-04, vì kiến trúc chỉ là cách tổ chức lại những khái niệm nền tảng đó cho gọn và dễ bảo trì.
4. File 08 → 11 là các chủ đề bổ sung thực chiến, đọc độc lập tùy nhu cầu (không bắt buộc theo thứ tự): 08 đi cùng mạch với 06 (Clean Architecture), 09 nên đọc sau 06 (test Application layer), 10 và 11 là 2 chủ đề ứng dụng độc lập, đọc khi cần.
5. File 12 (SOLID) nên đọc **trước hoặc song song** với file 06 và 08 — SOLID là lý thuyết nền tảng đứng sau cả Clean Architecture lẫn Design Patterns, giúp hiểu "vì sao" chứ không chỉ "làm sao".
6. File 13 (Tooling) nên đọc **ngay sau file 06** — phần "nhiều project theo Clean Architecture" chỉ thật sự "ngấm" khi bạn đã hiểu Dependency Rule, và đây là lúc thấy nó được ép buộc bằng compiler (ProjectReference) chứ không chỉ nằm trên giấy.

## Quy ước trong tài liệu

- 🧠 **Ví dụ trước** — tình huống thực tế để hình dung vấn đề.
- 📌 **Bản chất** — giải thích cơ chế bên dưới (compiler làm gì, runtime làm gì).
- ⚠️ **Lỗi thường gặp** — sai lầm phổ biến khi áp dụng.
- ✅ **Khi nào dùng** — quy tắc ra quyết định trong thực tế.

➡️ Xem lại [mục lục chung](../README.md) của toàn bộ tài liệu.
