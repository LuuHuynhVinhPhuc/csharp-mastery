# 04 — .NET Platform: CLR, Dependency Injection, ASP.NET Core

---

## 1. .NET là gì, thật ra? — Phân biệt các khái niệm hay bị lẫn lộn

### 📌 Bản chất

| Thuật ngữ | Là gì |
|---|---|
| **.NET** (trước đây gọi .NET Core) | Nền tảng (platform) mã nguồn mở, đa hệ điều hành (Windows/Linux/macOS) — hiện là chuẩn dùng cho dự án mới |
| **.NET Framework** | Phiên bản cũ, chỉ chạy trên Windows, không còn phát triển thêm tính năng mới (chỉ vá lỗi bảo trì) |
| **CLR (Common Language Runtime)** | "Cỗ máy ảo" thực thi code — nạp assembly, JIT compile, quản lý GC, bảo mật, xử lý exception (đã nói ở file 01) |
| **BCL (Base Class Library)** | Tập hợp class dựng sẵn dùng chung: `String`, `List<T>`, `File`, `HttpClient`... |
| **C#** | Ngôn ngữ lập trình — chỉ là **một trong nhiều** ngôn ngữ chạy trên CLR (còn có F#, VB.NET...) |
| **ASP.NET Core** | Framework xây dựng trên .NET, chuyên để làm web app / Web API |
| **SDK vs Runtime** | SDK = bộ công cụ để **build** (compiler, CLI...). Runtime = chỉ để **chạy** ứng dụng đã build sẵn (không build được) |

➡️ Tức là: C# là ngôn ngữ, .NET là nền tảng chạy nó, ASP.NET Core là framework làm web app **được viết bằng C# chạy trên .NET**. Ba tầng khác nhau, hay bị gộp chung khi nói chuyện.

### 1.1. CTS & CLS — vì sao nhiều ngôn ngữ dùng chung được 1 thư viện `.dll`

- **CTS (Common Type System)**: định nghĩa mọi type (class, struct, interface...) trong .NET phải tuân theo 1 bộ quy tắc chung, bất kể viết bằng ngôn ngữ nào. Nhờ vậy, một `class` viết bằng C# và một `class` viết bằng F# đều biên dịch ra IL theo cùng 1 "khuôn", gọi lẫn nhau được.
- **CLS (Common Language Specification)**: tập con quy tắc **nhỏ hơn** CTS, đảm bảo một thư viện viết ra sẽ dùng được bởi **mọi** ngôn ngữ .NET (kể cả ngôn ngữ không hỗ trợ `unsigned int` như VB.NET cũ) — khi publish thư viện công khai (NuGet), nên tuân thủ CLS để không giới hạn người dùng.

---

## 2. Dependency Injection (DI) — không phải "tính năng của framework", mà là một nguyên tắc thiết kế

### 🧠 Ví dụ trước

```csharp
// ⚠️ KHÔNG dùng DI — class tự new ra dependency của mình
class OrderService
{
    private readonly SqlOrderRepository _repo = new SqlOrderRepository(); // tự tạo, tự buộc chết cứng vào SQL

    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

Vấn đề: muốn viết **unit test** cho `OrderService` mà không chạm vào SQL Server thật? Không thể — vì `OrderService` tự `new` ra `SqlOrderRepository` bên trong, không có cách nào "thay thế" nó bằng bản giả (fake/mock).

```csharp
// ✅ DÙNG DI — dependency được "tiêm" từ bên ngoài vào qua constructor
interface IOrderRepository { void Save(Order order); }

class OrderService
{
    private readonly IOrderRepository _repo;
    public OrderService(IOrderRepository repo) => _repo = repo; // nhận từ bên ngoài, KHÔNG tự new

    public void PlaceOrder(Order order) => _repo.Save(order);
}
```

Giờ muốn test, chỉ cần truyền vào 1 `FakeOrderRepository` giả lập (hoặc dùng thư viện mock như Moq) thay vì đụng tới SQL thật.

### 📌 Bản chất: Inversion of Control (IoC)

DI là cách hiện thực nguyên tắc **Inversion of Control**: thay vì class **tự chủ động** tạo ra thứ nó cần (control nằm trong class), ta **đảo ngược quyền kiểm soát** ra bên ngoài — một "ai đó" (DI Container) chịu trách nhiệm tạo và "tiêm" dependency vào. Class giờ chỉ khai báo **nó cần gì** (qua interface trong constructor), không quan tâm **ai cung cấp** thứ đó.

### 2.1. 3 vòng đời (Lifetime) trong ASP.NET Core DI Container

```csharp
builder.Services.AddSingleton<ICacheService, MemoryCacheService>(); // 1 instance DUY NHẤT, dùng suốt vòng đời app
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();  // 1 instance MỚI cho MỖI HTTP request
builder.Services.AddTransient<IEmailSender, SmtpEmailSender>();      // 1 instance MỚI MỖI LẦN được yêu cầu (dù cùng 1 request)
```

| Lifetime | Tạo mới khi nào | Ví dụ phù hợp |
|---|---|---|
| **Singleton** | 1 lần duy nhất cho cả ứng dụng | Cache trong RAM, Configuration, Logger |
| **Scoped** | 1 lần cho mỗi HTTP request (dùng lại trong cùng request) | `DbContext` của Entity Framework — đây là mặc định chuẩn |
| **Transient** | Mỗi lần có ai yêu cầu (inject) | Service nhẹ, không giữ state, không tốn kém để tạo |

⚠️ **Bẫy nguy hiểm nhất: Captive Dependency** — Singleton **không được** phụ thuộc vào Scoped:

```csharp
class CacheService // đăng ký Singleton
{
    private readonly AppDbContext _db; // ⚠️ DbContext là Scoped!
    public CacheService(AppDbContext db) => _db = db;
}
```

Vì `CacheService` chỉ tạo 1 lần duy nhất lúc app khởi động, nó sẽ "giữ chết" `AppDbContext` của **request đầu tiên** mãi mãi — các request sau vẫn dùng chung connection/context đó, dẫn tới lỗi dữ liệu chéo giữa các user, hoặc exception `DbContext đã bị Dispose` (vì ASP.NET tự Dispose Scoped service khi request đó kết thúc, nhưng Singleton vẫn giữ tham chiếu tới object đã chết). ASP.NET Core sẽ **cảnh báo hoặc throw lỗi ngay lúc khởi động** nếu phát hiện kiểu lồng sai này (gọi là *scope validation*) — đừng tắt validation này đi để "cho qua".

✅ Quy tắc nhớ nhanh: **vòng đời ngắn có thể phụ thuộc vòng đời dài hơn** (Transient dùng Singleton: OK), **không bao giờ ngược lại** (Singleton dùng Scoped/Transient: NGUY HIỂM).

---

## 3. Middleware Pipeline — ASP.NET Core xử lý 1 request thế nào

### 🧠 Ví dụ trước

```csharp
var app = builder.Build();

app.Use(async (context, next) =>
{
    Console.WriteLine("1. Trước");
    await next(context);
    Console.WriteLine("4. Sau");
});
app.Use(async (context, next) =>
{
    Console.WriteLine("2. Trước");
    await next(context);
    Console.WriteLine("3. Sau");
});
app.Run(async context => await context.Response.WriteAsync("Xin chào"));
```

In ra thứ tự: `1 → 2 → (xử lý response) → 3 → 4`.

### 📌 Bản chất: mô hình "Chuỗi trách nhiệm" (giống lớp hành — onion/nested)

```mermaid
sequenceDiagram
    participant C as Client
    participant M1 as Middleware 1
    participant M2 as Middleware 2
    participant E as Endpoint

    C->>M1: 1. Request vào
    M1->>M2: next()
    M2->>E: 2. next()
    E-->>M2: Xử lý response
    M2-->>M1: 3. trả về
    M1-->>C: 4. Response ra
```

Mỗi middleware được quyền: xử lý trước khi gọi `next()` (đi vào trong), và xử lý sau khi `next()` trả về (đi ra ngoài) — giống như từng lớp vỏ hành bao quanh phần lõi (nơi xử lý request thật sự). Thứ tự `app.Use(...)` được đăng ký quyết định **thứ tự lồng** — middleware đăng ký trước nằm **ngoài cùng**.

✅ Đây là lý do thứ tự đăng ký middleware trong `Program.cs` **cực kỳ quan trọng**:

```csharp
app.UseExceptionHandler("/error");  // nên ở NGOÀI CÙNG — bắt lỗi của TẤT CẢ middleware bên trong
app.UseHttpsRedirection();
app.UseAuthentication();            // PHẢI trước Authorization — phải biết "ai" trước khi biết "được phép làm gì"
app.UseAuthorization();
app.MapControllers();               // endpoint — lõi trong cùng
```

Nếu lỡ đặt `UseAuthorization()` trước `UseAuthentication()`, hệ thống sẽ luôn thấy user "chưa xác định danh tính" khi kiểm tra quyền → luôn từ chối truy cập, dù user đã đăng nhập đúng.

---

## 4. Configuration & Options Pattern

### 🧠 Ví dụ trước

```json
// appsettings.json
{
  "EmailSettings": {
    "SmtpHost": "smtp.example.com",
    "Port": 587
  }
}
```

```csharp
builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));

class EmailSettings { public string SmtpHost { get; set; } public int Port { get; set; } }

class EmailSender
{
    private readonly EmailSettings _settings;
    public EmailSender(IOptions<EmailSettings> options) => _settings = options.Value; // được inject, không hardcode
}
```

### 📌 Vì sao dùng Options Pattern thay vì đọc `IConfiguration["EmailSettings:SmtpHost"]` trực tiếp khắp nơi

- **Type-safe**: đọc trực tiếp bằng string key dễ gõ sai (`"EmailSetting"` thiếu chữ `s`) mà chỉ phát hiện lúc runtime. Options Pattern dùng class strongly-typed, IDE gợi ý, bắt lỗi compile-time.
- **Tập trung một nơi**: thay đổi cấu trúc config chỉ sửa 1 class, không phải tìm từng chỗ dùng string key rải rác khắp codebase.
- **IOptionsSnapshot/IOptionsMonitor**: cho phép đọc lại config **mới nhất** khi file thay đổi lúc đang chạy (reload tự động), mà không cần restart app — `IOptions<T>` thường (Singleton) chỉ đọc 1 lần lúc khởi động.

---

## 5. Static vs Instance — nhắc lại cho rõ vì liên quan DI

```csharp
class MathHelper
{
    public static int Square(int x) => x * x; // KHÔNG cần tạo object để gọi
}

MathHelper.Square(5); // gọi thẳng qua tên class
```

📌 Member `static` thuộc về **chính class** (type), không thuộc về một instance (object) cụ thể nào — chỉ có **1 bản duy nhất** tồn tại trong suốt vòng đời ứng dụng, tương tự như Singleton nhưng được CLR quản lý tự động, không qua DI container.

⚠️ **Hệ quả về DI:** method/class `static` **không thể** inject dependency vào (không có constructor instance để tiêm), nên rất khó test (không thể thay thế bằng mock) và dễ vô tình tạo **shared mutable state** nguy hiểm khi nhiều request cùng truy cập field static. Trong code hiện đại dùng DI, nên hạn chế `static` cho những gì thực sự **không có trạng thái** (pure function, hằng số, hàm tiện ích thuần túy như `Math.Max`), tránh dùng cho service có phụ thuộc hay giữ trạng thái nghiệp vụ.

---

## 6. BackgroundService & IHostedService — chạy tác vụ nền trong ASP.NET Core

### 🧠 Ví dụ trước

Bạn cần **mỗi 1 giờ**, tự động quét database xóa các bản ghi tạm (OTP, session hết hạn...) — tác vụ này **không liên quan tới bất kỳ HTTP request nào**, cần tự chạy ngầm suốt vòng đời ứng dụng, song song với việc xử lý các request bình thường.

### 📌 `IHostedService` — interface nền tảng

```csharp
public class CleanupHostedService : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Service bắt đầu chạy — gọi 1 LẦN lúc ứng dụng khởi động");
        return Task.CompletedTask;
    }

    public Task StopAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Service dừng lại — gọi 1 LẦN lúc ứng dụng tắt (graceful shutdown)");
        return Task.CompletedTask;
    }
}
```

```csharp
builder.Services.AddHostedService<CleanupHostedService>(); // đăng ký — ASP.NET Core TỰ gọi StartAsync/StopAsync đúng lúc
```

📌 `IHostedService` chỉ cho bạn 2 "móc nối" (hook) vào đúng 2 thời điểm: **lúc ứng dụng khởi động xong** (`StartAsync`) và **lúc chuẩn bị tắt** (`StopAsync`) — muốn tự lặp lại công việc định kỳ bên trong, bạn phải tự quản lý (`Timer`, vòng lặp...), khá thủ công.

### ✅ `BackgroundService` — lớp trừu tượng dựng sẵn, dễ dùng hơn nhiều cho tác vụ lặp

```csharp
public class CleanupBackgroundService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory; // xem ⚠️ bên dưới — vì sao KHÔNG inject DbContext trực tiếp

    public CleanupBackgroundService(IServiceScopeFactory scopeFactory) => _scopeFactory = scopeFactory;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested) // vòng lặp chạy SUỐT vòng đời app, tới khi bị yêu cầu dừng
        {
            using (var scope = _scopeFactory.CreateScope()) // tự tạo 1 "scope" MỚI cho mỗi lần chạy
            {
                var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
                await db.Sessions.Where(s => s.ExpiresAt < DateTime.UtcNow).ExecuteDeleteAsync(stoppingToken);
            }

            await Task.Delay(TimeSpan.FromHours(1), stoppingToken); // nhớ lại file 03 — await KHÔNG tốn thread trong lúc chờ
        }
    }
}
```

### ⚠️ Bẫy quan trọng nhất: `BackgroundService` là Singleton — không được inject thẳng Scoped service

Nhớ lại mục 2.1 ở trên (Captive Dependency): mọi `IHostedService`/`BackgroundService` được ASP.NET Core đăng ký **như Singleton** (tạo đúng 1 lần, sống suốt vòng đời app). Nếu constructor nhận thẳng `AppDbContext` (vốn là **Scoped**, tạo mới mỗi request) làm tham số, bạn sẽ **tái phạm đúng lỗi Captive Dependency** đã học — `DbContext` đó sẽ bị "giữ chết" mãi mãi, dùng chung giữa các lần chạy tác vụ nền, dễ gây lỗi dữ liệu cũ/exception "context đã bị Dispose".

✅ **Cách giải quyết chuẩn:** inject `IServiceScopeFactory` (bản thân nó an toàn ở Singleton) vào `BackgroundService`, rồi **tự tạo 1 scope mới** (`CreateScope()`) mỗi lần cần dùng service Scoped — y hệt như cách ASP.NET Core tự làm ngầm cho mỗi HTTP request, chỉ khác là bạn phải **tự gọi tay** vì không có request nào kích hoạt sẵn.

### 📌 Khi nào dùng `BackgroundService` thay vì hệ thống job scheduler riêng (Hangfire, Quartz.NET...)

✅ **Phù hợp:** tác vụ nền đơn giản, không cần lịch phức tạp (cron expression), không cần retry/giao diện theo dõi, không cần đảm bảo chạy đúng 1 lần khi scale nhiều instance.

⚠️ **Không phù hợp:** nếu ứng dụng chạy **nhiều instance** (scale-out, file 07) và tác vụ **không được phép chạy trùng lặp** (ví dụ: gửi báo cáo tổng hợp mỗi ngày — không được cả 3 instance cùng gửi 3 lần), `BackgroundService` tự viết tay **không có cơ chế khóa phân tán** để đảm bảo chỉ 1 instance chạy — lúc này nên dùng thư viện job scheduler chuyên dụng (Hangfire, Quartz.NET) có hỗ trợ sẵn distributed lock, hoặc tách hẳn thành 1 service/job riêng biệt ngoài cụm web app.

---

## 7. Tóm tắt chương

| Khái niệm | Ghi nhớ cốt lõi |
|---|---|
| .NET vs .NET Framework vs CLR vs ASP.NET Core | Nền tảng / phiên bản cũ Windows-only / runtime thực thi / framework làm web, 4 tầng khác nhau |
| Dependency Injection | Đảo ngược quyền kiểm soát — class khai báo cần gì, container lo tạo và tiêm vào |
| Singleton/Scoped/Transient | Vòng đời ngắn phụ thuộc vòng đời dài: OK. Ngược lại: nguy hiểm (captive dependency) |
| Middleware pipeline | Lồng nhau như lớp hành — thứ tự đăng ký quyết định thứ tự chạy |
| Options Pattern | Config strongly-typed, an toàn hơn đọc string key trực tiếp |
| static | Thuộc về type, không phải instance — khó test, tránh dùng cho service có state/dependency |
| BackgroundService | Chạy tác vụ nền — là Singleton, PHẢI tự tạo scope (`IServiceScopeFactory`) để dùng service Scoped như DbContext |

➡️ Tiếp theo: [05-EntityFramework.md](05-EntityFramework.md) — EF Core và các bẫy hiệu năng kinh điển.
