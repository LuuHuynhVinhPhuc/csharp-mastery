# 12 — SOLID Principles: 5 nguyên tắc nền tảng của thiết kế OOP

> SOLID không phải là luật phải tuân theo 100% tuyệt đối, mà là **bộ công cụ chẩn đoán** — khi code bắt đầu "khó sửa, dễ vỡ", SOLID giúp bạn gọi tên chính xác **vấn đề gì** đang xảy ra để sửa đúng chỗ. Đây là nền tảng lý thuyết đứng sau hầu hết Design Patterns (file 08) và Clean Architecture (file 06) đã học.

---

## S — Single Responsibility Principle (Nguyên tắc đơn trách nhiệm)

> **Một class chỉ nên có 1 lý do để thay đổi.**

### 🧠 Ví dụ trước

```csharp
// ⚠️ VI PHẠM SRP — class này có TỚI 3 lý do để phải sửa đổi
class OrderService
{
    public void PlaceOrder(Order order)
    {
        // Lý do thay đổi #1: LOGIC NGHIỆP VỤ thay đổi (ví dụ thêm điều kiện giảm giá)
        if (order.Items.Count == 0) throw new Exception("Đơn hàng rỗng");
        decimal total = order.Items.Sum(i => i.Price * i.Quantity);

        // Lý do thay đổi #2: CÁCH LƯU TRỮ thay đổi (đổi từ SQL sang NoSQL)
        using var connection = new SqlConnection("...");
        connection.Execute("INSERT INTO Orders ...");

        // Lý do thay đổi #3: CÁCH GỬI THÔNG BÁO thay đổi (đổi từ Email sang SMS)
        var smtpClient = new SmtpClient("smtp.example.com");
        smtpClient.Send(new MailMessage(...));
    }
}
```

### 📌 Vấn đề thực tế

Class này sẽ phải sửa khi: (1) nghiệp vụ tính tổng tiền đổi khác, (2) đổi công nghệ lưu trữ, (3) đổi nhà cung cấp email. Ba team khác nhau (Business, DBA, DevOps) có thể cùng cần sửa **cùng 1 file** vì 3 lý do hoàn toàn không liên quan — dễ xung đột merge, và test logic nghiệp vụ (#1) buộc phải "dính" luôn cả database thật (#2) và SMTP thật (#3), dù bạn chỉ muốn kiểm tra đúng 1 công thức tính tiền.

### ✅ Áp dụng đúng

```csharp
class OrderCalculator           // lý do thay đổi: LOGIC tính toán nghiệp vụ
{
    public decimal CalculateTotal(Order order) => order.Items.Sum(i => i.Price * i.Quantity);
}

class OrderRepository           // lý do thay đổi: CÁCH LƯU TRỮ
{
    public void Save(Order order) { /* ... */ }
}

class NotificationService       // lý do thay đổi: CÁCH THÔNG BÁO
{
    public void NotifyCustomer(Order order) { /* ... */ }
}

class OrderService               // chỉ còn 1 lý do thay đổi: ĐIỀU PHỐI các bước lại với nhau
{
    private readonly OrderCalculator _calculator;
    private readonly OrderRepository _repository;
    private readonly NotificationService _notification;

    public void PlaceOrder(Order order)
    {
        var total = _calculator.CalculateTotal(order);
        _repository.Save(order);
        _notification.NotifyCustomer(order);
    }
}
```

⚠️ **Hiểu lầm phổ biến:** SRP **không** có nghĩa là "mỗi class chỉ được có đúng 1 method". Nó nghĩa là mọi method/field trong 1 class nên **cùng phục vụ 1 mục đích, cùng chịu tác động bởi cùng 1 nhóm lý do thay đổi**. Một class `OrderCalculator` có 5 method cùng để tính toán giá khác nhau vẫn hoàn toàn tuân thủ SRP — vì cả 5 method đó chỉ thay đổi khi **logic tính giá** thay đổi.

✅ **Dấu hiệu nhận biết vi phạm SRP:** class có tên chung chung kiểu "Manager", "Helper", "Processor" nhưng bên trong làm đủ thứ không liên quan; hoặc khi mô tả class bằng lời, bạn phải dùng chữ "và" ("class này lưu đơn hàng **và** gửi email **và** tính giá").

---

## O — Open/Closed Principle (Nguyên tắc đóng/mở)

> **Mở để mở rộng (extension), đóng để sửa đổi (modification).** Thêm hành vi mới bằng cách **thêm code mới**, không phải **sửa code cũ** đang chạy ổn định.

### 🧠 Ví dụ trước (đã thấy ở file 08, nhắc lại vì đây là ví dụ kinh điển nhất của OCP)

```csharp
// ⚠️ VI PHẠM OCP — mỗi lần công ty có thêm 1 phương thức giao hàng mới,
// BẮT BUỘC phải MỞ FILE NÀY RA SỬA, có nguy cơ ảnh hưởng tới các case cũ đang chạy tốt
decimal CalculateShippingFee(Order order, string method)
{
    if (method == "Standard") return order.Weight * 1000;
    else if (method == "Express") return order.Weight * 2500 + 20000;
    // thêm "SameDay" → phải sửa NGAY TẠI ĐÂY, động vào code đã test kỹ trước đó
    throw new NotSupportedException();
}
```

### ✅ Áp dụng đúng (Strategy Pattern, file 08 mục 1)

```csharp
interface IShippingStrategy { decimal Calculate(Order order); }
class StandardShipping : IShippingStrategy { public decimal Calculate(Order o) => o.Weight * 1000; }
class ExpressShipping  : IShippingStrategy { public decimal Calculate(Order o) => o.Weight * 2500 + 20000; }

// Thêm "SameDayShipping" sau này = THÊM 1 CLASS MỚI, không đụng tới StandardShipping/ExpressShipping đã có
class SameDayShipping : IShippingStrategy { public decimal Calculate(Order o) => o.Weight * 5000 + 50000; }
```

### 📌 Bản chất

OCP không có nghĩa là "không bao giờ được sửa code" — mọi hệ thống đều phải sửa khi có bug hoặc nghiệp vụ lõi thay đổi thật sự. OCP nhắm vào tình huống cụ thể: **khi thêm 1 biến thể mới** (thêm 1 loại giao hàng, 1 loại thanh toán, 1 loại thông báo...) thuộc **cùng 1 họ hành vi đã biết trước**, bạn nên thiết kế sao cho việc đó chỉ cần **viết thêm** (implement thêm 1 interface có sẵn), không cần **sửa lại** logic điều phối trung tâm (nơi code cũ, đã test kỹ, đang chạy ổn định trong production).

✅ **Công cụ hiện thực OCP phổ biến:** `interface`/`abstract class` + Strategy Pattern (file 08), `virtual`/`override` (file 02), Dependency Injection (file 04) để "cắm" hành vi mới vào mà không sửa nơi sử dụng.

⚠️ **Đừng áp dụng cực đoan ngay từ đầu:** thiết kế mọi thứ đều "mở rộng được qua interface" ngay từ dòng code đầu tiên, khi chưa chắc có thật sự cần nhiều biến thể hay không, là **over-engineering** — tạo ra độ phức tạp không cần thiết (nhớ lại nguyên tắc "không thiết kế cho yêu cầu giả định" ở đầu quá trình làm việc). Áp dụng OCP khi bạn **đã thấy** dấu hiệu rõ ràng: đoạn `if/else` hoặc `switch` theo 1 loại điều kiện đang **phình to dần theo thời gian** mỗi khi có nghiệp vụ mới.

---

## L — Liskov Substitution Principle (Nguyên tắc thay thế Liskov)

> **Nếu B là lớp con của A, mọi nơi đang dùng A phải dùng được B thay vào mà không làm chương trình sai lệch hay crash.**

### 🧠 Ví dụ trước — bài toán kinh điển "Hình chữ nhật — Hình vuông"

```csharp
class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    public int CalculateArea() => Width * Height;
}

class Square : Rectangle // "về mặt toán học", hình vuông LÀ 1 trường hợp đặc biệt của hình chữ nhật
{
    public override int Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; } // ép Width = Height để giữ tính chất "vuông"
    }
    public override int Height
    {
        get => base.Height;
        set { base.Width = value; base.Height = value; }
    }
}
```

```csharp
void TestRectangleBehavior(Rectangle r)
{
    r.Width = 5;
    r.Height = 10;
    Console.WriteLine(r.CalculateArea()); // Code gọi hàm này LUÔN KỲ VỌNG kết quả 50 (5 × 10)
}

TestRectangleBehavior(new Rectangle()); // In ra 50 ✅ đúng như kỳ vọng
TestRectangleBehavior(new Square());    // In ra 100! (vì set Height=10 làm Width TỰ ĐỘNG đổi theo thành 10)
```

### 📌 Vấn đề

Về mặt lý thuyết toán học, "hình vuông là hình chữ nhật đặc biệt" nghe rất hợp lý để kế thừa (`Square : Rectangle`). Nhưng về mặt **hành vi lập trình**, `Square` đã **phá vỡ kỳ vọng hợp lý** mà bất kỳ ai dùng `Rectangle` đều có quyền tin tưởng ("set Width không làm Height tự đổi theo") — code `TestRectangleBehavior` hoàn toàn đúng khi viết cho `Rectangle`, nhưng **âm thầm sai** khi nhận vào 1 `Square`, dù về mặt compiler, `Square` là `Rectangle` hợp lệ 100% (kế thừa đúng cú pháp).

✅ **Bài học:** LSP không kiểm tra việc kế thừa có đúng về mặt "is-a" theo ngôn ngữ tự nhiên/toán học hay không — nó kiểm tra việc kế thừa có giữ đúng **hợp đồng hành vi (behavioral contract)** của lớp cha hay không. "Hình vuông là hình chữ nhật" đúng về hình học, nhưng **sai** về hợp đồng hành vi lập trình ("Width và Height độc lập nhau").

### 📌 Dấu hiệu vi phạm LSP thường gặp khác

```csharp
class Bird { public virtual void Fly() { /* bay */ } }
class Penguin : Bird
{
    public override void Fly() => throw new NotSupportedException("Chim cánh cụt không biết bay!"); // ⚠️ VI PHẠM LSP
}
```

Bất kỳ code nào gọi `bird.Fly()` với kỳ vọng "mọi Bird đều bay được" sẽ **crash bất ngờ** khi gặp `Penguin` — dù về compiler, hoàn toàn hợp lệ để truyền `Penguin` vào chỗ cần `Bird`. ✅ **Cách sửa đúng:** đừng ép tất cả `Bird` đều có `Fly()` — tách riêng khả năng "biết bay" ra 1 interface riêng (`IFlyable`), chỉ những loài chim bay được mới implement nó; `Bird` (lớp cha chung) chỉ giữ những hành vi **chắc chắn đúng với mọi loài chim** (ăn, đẻ trứng...). Đây chính là cầu nối dẫn thẳng tới nguyên tắc tiếp theo.

---

## I — Interface Segregation Principle (Nguyên tắc phân tách interface)

> **Không nên ép 1 class phải implement những method nó không bao giờ dùng tới — nhiều interface nhỏ, chuyên biệt, tốt hơn 1 interface to ôm đồm.**

### 🧠 Ví dụ trước

```csharp
// ⚠️ VI PHẠM ISP — interface "to", ép mọi loại Worker phải có cả 2 khả năng
interface IWorker
{
    void Work();
    void Eat(); // không phải Worker nào cũng... ăn được
}

class HumanWorker : IWorker
{
    public void Work() => Console.WriteLine("Đang làm việc");
    public void Eat() => Console.WriteLine("Đang ăn trưa");
}

class RobotWorker : IWorker
{
    public void Work() => Console.WriteLine("Đang làm việc");
    public void Eat() => throw new NotSupportedException(); // ⚠️ bị ép implement thứ KHÔNG BAO GIỜ dùng tới
}
```

Đây chính là biểu hiện cụ thể của vi phạm LSP (mục L) đã thấy ở trên, nhưng **nguyên nhân gốc** nằm ở thiết kế interface quá "to", gộp chung những khả năng không phải lúc nào cũng đi cùng nhau.

### ✅ Áp dụng đúng — tách thành nhiều interface nhỏ, chuyên biệt

```csharp
interface IWorkable { void Work(); }
interface IFeedable { void Eat(); }

class HumanWorker : IWorkable, IFeedable // implement CẢ 2, vì con người cần cả 2 khả năng
{
    public void Work() => Console.WriteLine("Đang làm việc");
    public void Eat() => Console.WriteLine("Đang ăn trưa");
}

class RobotWorker : IWorkable // CHỈ implement cái nó THỰC SỰ có — không còn method thừa, không còn throw exception giả
{
    public void Work() => Console.WriteLine("Đang làm việc");
}
```

### 📌 Bản chất

Interface không chỉ là "hợp đồng" (contract) giữa code — nó còn là **tài liệu giao tiếp** cho người đọc code sau này. Interface `IWorker` to, ôm đồm khiến `RobotWorker` **buộc phải "nói dối"** (implement `Eat()` nhưng throw exception) — vi phạm trực tiếp LSP vừa học. Chia nhỏ interface theo **đúng nhóm khả năng thường đi cùng nhau trong thực tế**, class nào cần gì thì implement đúng cái đó, không thừa không thiếu.

✅ **Quy tắc thực dụng:** nếu bạn thấy mình phải viết `throw new NotImplementedException()` hoặc để trống thân method khi implement 1 interface, đó là **tín hiệu rõ ràng nhất** của vi phạm ISP — hãy tách interface đó ra nhỏ hơn.

---

## D — Dependency Inversion Principle (Nguyên tắc đảo ngược phụ thuộc)

> **Module cấp cao không nên phụ thuộc vào module cấp thấp — cả hai nên phụ thuộc vào abstraction (interface). Abstraction không nên phụ thuộc vào chi tiết — chi tiết nên phụ thuộc vào abstraction.**

> 📌 Nguyên tắc này đã được trình bày chi tiết với ví dụ đầy đủ ở [06-CleanArchitecture.md](06-CleanArchitecture.md) mục 1 (vì nó chính là **nền tảng lý thuyết** của toàn bộ Clean Architecture — Dependency Rule chỉ là DIP áp dụng ở quy mô kiến trúc). Nhắc nhanh cốt lõi tại đây:

### 🧠 Ví dụ trước (rút gọn)

```csharp
// ⚠️ VI PHẠM DIP — module cấp cao (CreateOrderHandler) phụ thuộc TRỰC TIẾP vào module cấp thấp (EfOrderRepository cụ thể)
class CreateOrderHandler
{
    private readonly EfOrderRepository _repo = new EfOrderRepository(); // new TRỰC TIẾP class cụ thể
}

// ✅ ĐÚNG DIP — cả 2 bên đều phụ thuộc vào INTERFACE (abstraction) ở giữa
interface IOrderRepository { void Save(Order order); }

class CreateOrderHandler // module CẤP CAO — chỉ biết interface
{
    private readonly IOrderRepository _repo;
    public CreateOrderHandler(IOrderRepository repo) => _repo = repo;
}

class EfOrderRepository : IOrderRepository // module CẤP THẤP — implement đúng interface, PHỤ THUỘC NGƯỢC VÀO abstraction
{
    public void Save(Order order) { /* dùng EF Core */ }
}
```

### 📌 Chữ "Inversion" (đảo ngược) nằm ở đâu

Không có DIP, hướng phụ thuộc tự nhiên sẽ là: code nghiệp vụ cấp cao **gọi thẳng** vào chi tiết kỹ thuật cấp thấp (`CreateOrderHandler` → `EfOrderRepository` → EF Core). Có DIP, interface (`IOrderRepository`) được **module cấp cao định nghĩa ra** (theo đúng ngôn ngữ nghiệp vụ nó cần), còn module cấp thấp phải **tự implement đúng theo interface đó** — hướng phụ thuộc **compile-time** bị "đảo ngược" lại: giờ `EfOrderRepository` phụ thuộc vào `IOrderRepository` (nằm ở tầng cấp cao), thay vì ngược lại.

✅ Đây là nguyên tắc **quan trọng nhất** trong 5 nguyên tắc SOLID đối với kiến trúc hệ thống lớn — nó chính là cơ chế giúp Dependency Injection (file 04), Repository Pattern, và toàn bộ Clean Architecture (file 06) có thể tồn tại. Xem file 06 để có ví dụ đầy đủ về cách DIP áp dụng xuyên suốt 1 ứng dụng thật.

---

## Tóm tắt chương

| Nguyên tắc | Câu hỏi để tự kiểm tra | Dấu hiệu vi phạm thường gặp |
|---|---|---|
| **S**ingle Responsibility | "Class này có bao nhiêu LÝ DO để phải sửa?" | Tên class chung chung (Manager/Helper), mô tả phải dùng chữ "và" |
| **O**pen/Closed | "Thêm 1 biến thể mới có cần SỬA code cũ không?" | `if/else`/`switch` theo 1 loại điều kiện phình to dần theo thời gian |
| **L**iskov Substitution | "Lớp con có giữ đúng HÀNH VI mà lớp cha hứa hẹn không?" | Override rồi `throw NotSupportedException`, hoặc đổi hành vi bất ngờ so với cha |
| **I**nterface Segregation | "Class có bị ép implement method nó KHÔNG BAO GIỜ dùng không?" | Thân method rỗng hoặc `throw NotImplementedException` khi implement interface |
| **D**ependency Inversion | "Module cấp cao có đang `new` trực tiếp chi tiết kỹ thuật cấp thấp không?" | Business logic `using` thẳng EF Core/HttpClient/SmtpClient thay vì qua interface |

➡️ Quay lại [00-README.md](00-README.md) để xem toàn bộ lộ trình, hoặc xem [06-CleanArchitecture.md](06-CleanArchitecture.md) và [08-DesignPatterns.md](08-DesignPatterns.md) để thấy SOLID được áp dụng vào kiến trúc và pattern cụ thể.
