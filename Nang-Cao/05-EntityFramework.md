# 05 — Entity Framework Core: Cơ chế bên dưới & bẫy hiệu năng

---

## 1. DbContext là gì — không chỉ là "cái để query"

### 🧠 Ví dụ trước

```csharp
using var context = new AppDbContext();
var product = context.Products.First(p => p.Id == 1);
product.Price = 199;
context.SaveChanges(); // Chỉ gọi hàm này, KHÔNG viết câu UPDATE nào — vậy EF biết sửa gì mà UPDATE?
```

### 📌 Bản chất: Change Tracker

`DbContext` giữ một "sổ theo dõi" nội bộ gọi là **Change Tracker**. Khi bạn query một entity ra (`context.Products.First(...)`), EF **chụp lại một bản snapshot** giá trị ban đầu của entity đó và lưu vào bộ nhớ của context. Khi bạn gọi `SaveChanges()`, EF **so sánh** giá trị hiện tại của entity với snapshot ban đầu, phát hiện field nào đổi (`Price`: 150 → 199), rồi tự sinh ra câu `UPDATE Products SET Price = 199 WHERE Id = 1`.

Mỗi entity do context quản lý có 1 trong các trạng thái:

| Trạng thái | Ý nghĩa | Khi SaveChanges() sẽ |
|---|---|---|
| `Added` | Entity mới, chưa có trong DB | Sinh `INSERT` |
| `Unchanged` | Lấy từ DB ra, chưa sửa gì | Không làm gì |
| `Modified` | Lấy từ DB ra, đã sửa field nào đó | Sinh `UPDATE` |
| `Deleted` | Đánh dấu xóa | Sinh `DELETE` |
| `Detached` | Entity KHÔNG được context theo dõi (không liên quan DB hiện tại) | Không ảnh hưởng gì khi SaveChanges |

### ⚠️ Bẫy: `DbContext` **không thread-safe** và **không nên sống lâu**

```csharp
// ⚠️ SAI — dùng 1 DbContext DUY NHẤT cho cả app (như Singleton), nhiều request cùng đụng vào 1 lúc
builder.Services.AddSingleton<AppDbContext>();
```

Vì Change Tracker là state nội bộ có thể bị nhiều thread cùng sửa đồng thời (gợi nhớ race condition ở file 03), dùng chung 1 `DbContext` cho nhiều request sẽ gây lỗi khó lường hoặc crash. Đây chính là lý do EF luôn đăng ký theo kiểu **Scoped** (file 04) — mỗi HTTP request có 1 `DbContext` riêng, dùng xong (request kết thúc) thì Dispose.

✅ Quy tắc: `DbContext` nên có vòng đời **ngắn** — tạo ra, dùng cho 1 "đơn vị công việc" (unit of work, thường là 1 request), rồi Dispose ngay. Không giữ context làm field static/singleton.

---

## 2. N+1 Query Problem — bẫy hiệu năng kinh điển nhất của ORM

### 🧠 Ví dụ trước

```csharp
var orders = context.Orders.ToList(); // 1 câu SELECT lấy N đơn hàng

foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name); // ⚠️ mỗi lần truy cập Customer lại chạy 1 SELECT RIÊNG!
}
```

### 📌 Vì sao xảy ra: Lazy Loading

Nếu bật **Lazy Loading**, `order.Customer` chỉ thật sự được load từ database **đúng lúc bạn truy cập nó** (không load sẵn cùng với `Orders`). Với N đơn hàng, vòng lặp trên sinh ra: **1 câu query lấy Orders + N câu query riêng lẻ lấy từng Customer** = tổng cộng N+1 câu SQL, thay vì lẽ ra chỉ cần 1-2 câu là đủ. Với 1000 đơn hàng → 1001 round-trip tới database, chậm khủng khiếp.

### ✅ Giải pháp 1: Eager Loading (`Include`) — load sẵn luôn trong 1 câu query

```csharp
var orders = context.Orders
    .Include(o => o.Customer)        // JOIN sẵn, lấy Customer CÙNG LÚC
    .Include(o => o.Items)           // có thể Include nhiều quan hệ
        .ThenInclude(i => i.Product) // Include "lồng" — quan hệ của quan hệ
    .ToList();
```

### ✅ Giải pháp 2: Explicit Loading — chủ động load khi cần, kiểm soát rõ ràng

```csharp
context.Entry(order).Reference(o => o.Customer).Load(); // chỉ load khi thật sự cần
```

✅ **Khi nào dùng cái nào:** Eager Loading (`Include`) khi bạn **biết chắc** sẽ cần dữ liệu liên quan ngay (hiển thị danh sách đơn hàng kèm tên khách). Tắt hẳn Lazy Loading trong dự án thực tế (không cấu hình proxy) để buộc dev phải viết `Include` tường minh — tránh N+1 xảy ra "ngầm" mà không ai để ý.

---

## 3. Tracking vs No-Tracking — khi nào cần, khi nào không

### 🧠 Ví dụ trước

```csharp
// Query CHỈ ĐỂ HIỂN THỊ (read-only) — không có ý định sửa gì
var products = context.Products.ToList(); // mặc định: EF VẪN track từng entity, dù bạn không sửa gì cả!
```

### 📌 Bản chất

Mặc định, **mọi** entity query ra đều bị Change Tracker theo dõi (tốn bộ nhớ lưu snapshot + tốn CPU so sánh mỗi lần `SaveChanges`), kể cả khi bạn chỉ đọc để hiển thị, không hề có ý định sửa. Với danh sách lớn, chi phí tracking này cộng dồn đáng kể.

```csharp
// ✅ Query chỉ để đọc — tắt tracking, nhanh hơn, tốn ít bộ nhớ hơn
var products = context.Products.AsNoTracking().ToList();
```

✅ **Quy tắc:** mọi API/màn hình **chỉ hiển thị dữ liệu** (GET, report, danh sách) nên dùng `.AsNoTracking()`. Chỉ giữ tracking (mặc định) khi bạn **sẽ** `Update`/`SaveChanges` lên chính entity đó trong cùng request.

---

## 4. `IQueryable` — nhắc lại và mở rộng (liên kết với file 02)

### ⚠️ Bẫy "kéo cả bảng về rồi mới lọc"

```csharp
// ⚠️ SAI — .ToList() ép thực thi SQL NGAY (SELECT * FROM Products, KHÔNG WHERE),
// rồi .Where() sau đó chạy TRONG BỘ NHỚ ỨNG DỤNG (LINQ to Objects), không phải trên DB
var expensive = context.Products.ToList().Where(p => p.Price > 1000);

// ✅ ĐÚNG — .Where() đứng TRƯỚC .ToList(), được dịch thành SQL "WHERE Price > 1000"
// → database chỉ trả về đúng những dòng cần, nhanh hơn rất nhiều với bảng lớn
var expensive = context.Products.Where(p => p.Price > 1000).ToList();
```

📌 Quy tắc vàng: **luôn giữ `IQueryable` càng lâu càng tốt** (lọc, sort, phân trang...), chỉ gọi hàm ép thực thi (`.ToList()`, `.ToArray()`, `.FirstOrDefault()`...) **ở bước CUỐI CÙNG**, ngay trước khi thật sự cần dữ liệu trong tay.

### 4.1. Projection (`Select`) — chỉ lấy cột cần, không lấy cả entity

```csharp
// Lấy nguyên cả Product (mọi cột) dù chỉ cần hiển thị tên và giá
var products = context.Products.Where(p => p.IsActive).ToList();

// ✅ Chỉ SELECT đúng 2 cột cần — ít dữ liệu truyền qua mạng, nhanh hơn, cũng không bị tracking
var products = context.Products
    .Where(p => p.IsActive)
    .Select(p => new ProductDto { Name = p.Name, Price = p.Price })
    .ToList();
```

---

## 5. Migration — quản lý thay đổi schema theo thời gian

### 🧠 Ví dụ trước

```bash
dotnet ef migrations add AddDiscountColumnToProduct
dotnet ef database update
```

### 📌 Bản chất

Mỗi lệnh `migrations add` tạo ra 1 file C# mô tả **sự khác biệt** giữa model hiện tại trong code và lần migration trước đó (ví dụ: "thêm cột `Discount` kiểu `decimal` vào bảng `Products`"), với 2 method `Up()` (áp dụng thay đổi) và `Down()` (hoàn tác). `database update` sẽ chạy các migration **chưa áp dụng** theo đúng thứ tự, lên database thật.

✅ Lợi ích: schema database được **version hóa như code** (commit cùng Git), team nhiều người cùng thay đổi schema mà không giẫm chân nhau, có thể rollback (`Down()`) nếu cần, và mỗi môi trường (dev/staging/production) luôn đồng bộ được schema bằng cách chạy đúng tập migration.

⚠️ **Lưu ý khi làm production:** không nên để ứng dụng tự `context.Database.Migrate()` lúc khởi động trong môi trường có **nhiều instance chạy song song** (scale-out) — nhiều instance cùng cố chạy migration 1 lúc sẽ xung đột. Nên tách bước migrate thành 1 job riêng, chạy 1 lần, trước khi deploy ứng dụng.

---

## 6. Optimistic Concurrency — xử lý khi 2 người sửa cùng 1 dòng dữ liệu

### 🧠 Ví dụ trước

User A và User B cùng mở 1 đơn hàng để sửa. A sửa trước, lưu thành công. B sửa dựa trên dữ liệu **cũ** (chưa biết A đã đổi), lưu sau — nếu không kiểm soát, **B sẽ vô tình ghi đè mất thay đổi của A** mà không ai biết.

### 📌 Giải pháp: Concurrency Token (RowVersion)

```csharp
class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }

    [Timestamp] // EF tự so sánh giá trị này mỗi lần UPDATE
    public byte[] RowVersion { get; set; }
}
```

EF sinh câu UPDATE kèm điều kiện: `UPDATE Orders SET Total = @p WHERE Id = @id AND RowVersion = @oldRowVersion`. Nếu A đã sửa trước (RowVersion trong DB đã đổi khác), câu UPDATE của B sẽ **ảnh hưởng 0 dòng** → EF ném `DbUpdateConcurrencyException`, báo cho B biết "dữ liệu đã bị người khác thay đổi, hãy tải lại trước khi lưu" — thay vì âm thầm ghi đè.

⚠️ **Lưu ý: `[Timestamp]`/kiểu `rowversion` là tính năng riêng của SQL Server** (cột tự động đổi giá trị mỗi lần UPDATE, do chính SQL Server quản lý) — không có sẵn y hệt trên PostgreSQL/MySQL. Với các provider khác, dùng `[ConcurrencyCheck]` trên 1 cột thường (ví dụ `UpdatedAt`, tự tay cập nhật mỗi lần sửa), hoặc cấu hình qua Fluent API (`IsConcurrencyToken()`) để chỉ định cột nào đóng vai trò token — cơ chế kiểm tra lúc `SaveChanges()` thì vẫn giống nhau, chỉ khác cách cột đó được sinh giá trị.

---

## 7. Tóm tắt chương

| Khái niệm | Ghi nhớ cốt lõi |
|---|---|
| DbContext | Giữ Change Tracker để tự sinh SQL khi SaveChanges — vòng đời ngắn, Scoped, không thread-safe |
| N+1 Problem | Lazy Loading gây ra N query lẻ trong vòng lặp — giải quyết bằng `Include`/`ThenInclude` |
| AsNoTracking | Dùng cho mọi query chỉ-đọc — tiết kiệm bộ nhớ và CPU |
| IQueryable | Giữ càng lâu càng tốt, chỉ `.ToList()` ở bước cuối cùng để lọc được dịch ra SQL |
| Select (Projection) | Chỉ lấy cột cần, giảm dữ liệu truyền tải, tự động không bị tracking |
| Migration | Schema version hóa như code, Up/Down, tách job migrate riêng khi scale-out |
| Concurrency Token | Phát hiện xung đột ghi đè giữa 2 người dùng cùng sửa 1 dòng dữ liệu |

➡️ Tiếp theo: [06-CleanArchitecture.md](06-CleanArchitecture.md) — tổ chức code ở quy mô ứng dụng lớn.
