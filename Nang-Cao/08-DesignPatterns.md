# 08 — Design Patterns: Áp dụng thực tế trong C#

> Design Pattern không phải "công thức phải thuộc lòng" — mỗi pattern là **giải pháp đã được kiểm chứng** cho 1 vấn đề lặp đi lặp lại. Phần này tập trung vào: **vấn đề gì** khiến pattern đó ra đời, chứ không chỉ liệt kê UML.

---

## 1. Strategy — thay đổi thuật toán lúc runtime, không dùng `if/else` chằng chịt

### 🧠 Ví dụ trước

```csharp
// ⚠️ SAI — mỗi lần thêm phương thức tính phí mới lại phải sửa class này (vi phạm Open/Closed Principle)
decimal CalculateShippingFee(Order order, string method)
{
    if (method == "Standard") return order.Weight * 1000;
    else if (method == "Express") return order.Weight * 2500 + 20000;
    else if (method == "SameDay") return order.Weight * 5000 + 50000;
    throw new NotSupportedException();
}
```

### ✅ Strategy Pattern

```csharp
interface IShippingStrategy { decimal Calculate(Order order); }

class StandardShipping : IShippingStrategy { public decimal Calculate(Order o) => o.Weight * 1000; }
class ExpressShipping  : IShippingStrategy { public decimal Calculate(Order o) => o.Weight * 2500 + 20000; }

class ShippingCalculator
{
    private readonly IShippingStrategy _strategy; // nhận "thuật toán" từ bên ngoài — gợi nhớ DI ở file 04
    public ShippingCalculator(IShippingStrategy strategy) => _strategy = strategy;
    public decimal Calculate(Order order) => _strategy.Calculate(order);
}
```

📌 **Bản chất**: đóng gói **mỗi thuật toán** thành 1 class riêng cùng chung 1 interface, rồi "tiêm" thuật toán cụ thể vào nơi cần dùng. Thêm phương thức giao hàng mới (`SameDayShipping`) chỉ cần **thêm class mới**, không sửa code cũ — đúng nguyên tắc **Open/Closed** (mở để mở rộng, đóng để sửa đổi) trong SOLID.

✅ **Khi nào dùng:** khi có **nhiều cách làm cùng 1 việc**, và danh sách cách làm đó có xu hướng tăng theo thời gian (thêm phương thức thanh toán, thêm công thức tính giá, thêm loại validation...).

---

## 2. Factory Method / Abstract Factory — tách việc "tạo object" ra khỏi nơi "dùng object"

### 🧠 Ví dụ trước

```csharp
// ⚠️ Code dùng trực tiếp "new" nên bị khóa cứng vào 1 loại cụ thể, khó mở rộng/test
void ProcessPayment(string provider)
{
    IPaymentGateway gateway = provider == "VNPay" ? new VnPayGateway() : new MomoGateway();
    gateway.Charge(100000);
}
```

### ✅ Factory Pattern

```csharp
interface IPaymentGatewayFactory { IPaymentGateway Create(string provider); }

class PaymentGatewayFactory : IPaymentGatewayFactory
{
    public IPaymentGateway Create(string provider) => provider switch
    {
        "VNPay" => new VnPayGateway(),
        "Momo"  => new MomoGateway(),
        _ => throw new NotSupportedException($"Không hỗ trợ cổng thanh toán {provider}")
    };
}

class PaymentProcessor
{
    private readonly IPaymentGatewayFactory _factory;
    public PaymentProcessor(IPaymentGatewayFactory factory) => _factory = factory; // chỉ phụ thuộc INTERFACE của factory

    public void ProcessPayment(string provider) => _factory.Create(provider).Charge(100000);
}
```

📌 **Bản chất**: tập trung logic "tạo ra object nào, tùy điều kiện gì" vào **đúng 1 nơi** (Factory), thay vì rải rác khắp codebase. `PaymentProcessor` giờ không biết `VnPayGateway` hay `MomoGateway` tồn tại — chỉ biết interface. Thêm cổng thanh toán mới chỉ sửa Factory, không đụng tới nơi dùng.

✅ Khác biệt với Strategy: Strategy tập trung vào **"nhiều cách làm 1 việc"**, Factory tập trung vào **"nhiều cách tạo ra 1 loại object"** — thường 2 pattern này đi kèm nhau (Factory tạo ra đúng Strategy cần dùng).

---

## 3. Decorator — thêm hành vi mà không sửa code gốc

### 🧠 Ví dụ trước

Muốn thêm cache, rồi thêm logging, rồi thêm retry cho `IProductRepository`, nhưng **không muốn sửa** class gốc đang hoạt động ổn định (rủi ro phá vỡ cái đang chạy tốt).

```csharp
interface IProductRepository { Product? GetById(int id); }

class SqlProductRepository : IProductRepository
{
    public Product? GetById(int id) => /* query database */ null;
}

// Decorator: BỌC bên ngoài 1 implementation có sẵn, cùng implement chung interface
class CachedProductRepository : IProductRepository
{
    private readonly IProductRepository _inner; // "bọc" lấy implementation gốc
    private readonly IMemoryCache _cache;

    public CachedProductRepository(IProductRepository inner, IMemoryCache cache)
    {
        _inner = inner;
        _cache = cache;
    }

    public Product? GetById(int id) =>
        _cache.GetOrCreate($"product_{id}", entry =>
        {
            entry.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5);
            return _inner.GetById(id); // chỉ gọi xuống DB thật khi cache miss
        });
}
```

```csharp
// Đăng ký trong DI container: SqlProductRepository bị "bọc" bởi CachedProductRepository
builder.Services.AddScoped<SqlProductRepository>();
builder.Services.AddScoped<IProductRepository>(sp =>
    new CachedProductRepository(sp.GetRequiredService<SqlProductRepository>(), sp.GetRequiredService<IMemoryCache>()));
```

📌 **Bản chất**: `CachedProductRepository` và `SqlProductRepository` cùng implement 1 interface, cái này "bọc" lấy cái kia — người dùng cuối (`OrderService` chẳng hạn) chỉ thấy `IProductRepository`, hoàn toàn không biết có cache ở giữa. Có thể **xếp chồng nhiều lớp decorator** (cache → logging → retry → thật) mà mỗi lớp không biết gì về các lớp khác, chỉ biết "gọi xuống lớp trong cùng interface".

✅ **Khi nào dùng:** cần thêm hành vi xuyên suốt (cross-cutting: cache, log, retry, validate) cho 1 interface có sẵn, mà không muốn sửa trực tiếp implementation gốc hoặc dùng kế thừa (kế thừa chỉ thêm được hành vi tĩnh lúc compile, decorator linh hoạt hơn vì ghép lúc runtime qua DI).

---

## 4. Unit of Work — đi kèm Repository Pattern (file 06), quản lý transaction xuyên nhiều repository

### 🧠 Ví dụ trước

```csharp
// ⚠️ Nếu Save() của OrderRepo và InventoryRepo là 2 lần SaveChanges() RIÊNG BIỆT,
// lỡ InventoryRepo.Save() thất bại SAU KHI OrderRepo.Save() đã thành công → dữ liệu KHÔNG NHẤT QUÁN
orderRepo.Save(order);
inventoryRepo.Save(inventoryUpdate);
```

### ✅ Unit of Work

```csharp
interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }
    Task<int> SaveChangesAsync(); // CHỈ 1 LẦN DUY NHẤT — cả 2 thay đổi cùng 1 transaction
}

class EfUnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context; // 1 DbContext DÙNG CHUNG cho mọi repository bên trong
    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }

    public EfUnitOfWork(AppDbContext context)
    {
        _context = context;
        Orders = new EfOrderRepository(context);
        Inventory = new EfInventoryRepository(context);
    }

    public Task<int> SaveChangesAsync() => _context.SaveChangesAsync(); // 1 transaction DUY NHẤT
}
```

📌 **Bản chất**: vì `DbContext` của EF Core (file 05) **bản thân nó đã là 1 Unit of Work** (Change Tracker gộp mọi thay đổi, `SaveChanges()` sinh ra 1 transaction bao trọn mọi thay đổi đã track), nhiều dự án dùng EF Core **không cần tự viết thêm lớp Unit of Work riêng** — chỉ cần đảm bảo mọi repository trong cùng 1 use case dùng **chung 1 instance `DbContext`** (đã tự nhiên đúng nhờ Scoped lifetime ở file 04), rồi gọi `SaveChanges()` đúng 1 lần ở Application layer, sau khi đã gọi xong mọi repository cần thiết.

---

## 5. Observer — đã học rồi, chỉ cần nhận ra

📌 Pattern **Observer** (1 đối tượng "subject" thông báo cho nhiều "observer" khi có sự kiện xảy ra) đã được tích hợp sẵn vào ngôn ngữ C# qua từ khóa `event`/`delegate` (file 02) — không cần tự dựng lại từ đầu như Java/C++ truyền thống. Ở quy mô lớn hơn (giữa các service), chính là **Message/Event Bus** đã nói ở file 07.

---

## 6. Singleton — pattern kinh điển, nhưng cẩn thận khi dùng trong .NET hiện đại

### 🧠 Ví dụ trước (cách viết tay truyền thống)

```csharp
class ConfigManager
{
    private static readonly Lazy<ConfigManager> _instance = new(() => new ConfigManager());
    public static ConfigManager Instance => _instance.Value;
    private ConfigManager() { } // constructor private — không ai new() được từ bên ngoài
}
```

⚠️ **Trong ASP.NET Core, hầu như không cần viết Singleton pattern tay như trên nữa** — dùng `builder.Services.AddSingleton<ConfigManager>()` (file 04) đã cho cùng hiệu quả (1 instance duy nhất, dùng chung toàn app), nhưng **còn test được** (vì có thể inject fake qua constructor), trong khi Singleton viết tay (static `Instance`) **không test được** — không thể thay thế bằng mock vì mọi nơi gọi thẳng `ConfigManager.Instance` (hard-coded dependency, vi phạm chính nguyên tắc DI đã nói ở file 04).

✅ **Quy tắc:** trong ứng dụng dùng DI container (ASP.NET Core, gần như mọi app hiện đại), hãy dùng `AddSingleton` qua container thay vì tự viết static Singleton tay — trừ khi đó là 1 utility cực kỳ đơn giản, thuần túy, không có dependency gì (ví dụ class hằng số).

---

## 7. Tóm tắt chương

| Pattern | Vấn đề giải quyết |
|---|---|
| Strategy | Nhiều cách làm cùng 1 việc, muốn thêm cách mới mà không sửa code cũ |
| Factory | Tập trung logic "tạo object nào" vào 1 nơi, nơi dùng không biết implementation cụ thể |
| Decorator | Thêm hành vi (cache/log/retry) bọc quanh implementation có sẵn, không sửa code gốc |
| Unit of Work | Đảm bảo nhiều repository cùng tham gia 1 transaction duy nhất — EF Core's `DbContext` đã tự làm việc này |
| Observer | Thông báo sự kiện cho nhiều bên quan tâm — đã có sẵn qua `event`/`delegate` trong C# |
| Singleton | 1 instance duy nhất toàn app — nên dùng qua DI container, không viết static tay |

➡️ Tiếp theo: [09-UnitTesting-TDD.md](09-UnitTesting-TDD.md) — viết test có giá trị thật sự, không chỉ để "có coverage".
