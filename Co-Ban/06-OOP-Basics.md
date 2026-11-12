# 06 — Lập trình Hướng đối tượng cơ bản (OOP)

---

## 1. Class và Object — khuôn mẫu và thực thể

### 🧠 Ví dụ trước

```csharp
class ConMeo // "class" = KHUÔN MẪU, bản thiết kế
{
    public string Ten;
    public int Tuoi;

    public void Keu()
    {
        Console.WriteLine($"{Ten}: Meo meo!");
    }
}

ConMeo meo1 = new ConMeo(); // "new" = TẠO RA 1 OBJECT THẬT từ khuôn mẫu ConMeo
meo1.Ten = "Mun";
meo1.Tuoi = 2;
meo1.Keu(); // "Mun: Meo meo!"

ConMeo meo2 = new ConMeo(); // 1 OBJECT KHÁC, độc lập hoàn toàn với meo1
meo2.Ten = "Lu";
meo2.Keu(); // "Lu: Meo meo!"
```

### 📌 Giải thích

- **Class** giống như 1 **bản thiết kế/khuôn bánh** — định nghĩa "con mèo thì có những gì" (Tên, Tuổi) và "con mèo làm được gì" (Kêu).
- **Object** (hay *instance*) là **1 con mèo thật sự**, được tạo ra từ khuôn mẫu đó bằng từ khóa `new`. Mỗi object có dữ liệu **riêng của nó** — `meo1.Ten` và `meo2.Ten` hoàn toàn độc lập.

| Thành phần trong class | Gọi là | Ví dụ |
|---|---|---|
| Biến bên trong class (`Ten`, `Tuoi`) | **Field** (trường dữ liệu) | lưu trạng thái của object |
| Hàm bên trong class (`Keu()`) | **Method** (phương thức) | hành vi của object |

---

## 2. Constructor — "khởi tạo" object ngay lúc tạo ra

### 🧠 Ví dụ trước

```csharp
class ConMeo
{
    public string Ten;
    public int Tuoi;

    public ConMeo(string ten, int tuoi) // CONSTRUCTOR — tên TRÙNG tên class, KHÔNG có kiểu trả về
    {
        Ten = ten;
        Tuoi = tuoi;
    }
}

ConMeo meo = new ConMeo("Mun", 2); // BẮT BUỘC truyền đủ tham số ngay lúc "new"
```

### 📌 Giải thích

**Constructor** là 1 method đặc biệt tự động chạy **ngay khi object được tạo** (`new`) — dùng để đảm bảo object luôn có đủ dữ liệu cần thiết ngay từ đầu, thay vì phải gán từng field 1 sau khi tạo (dễ quên gán, dễ tạo ra object "thiếu dữ liệu").

⚠️ Nếu bạn **không viết constructor nào**, C# tự cho bạn 1 constructor **rỗng, không tham số** (gọi là *default constructor*) — y hệt ví dụ `new ConMeo()` ở mục 1. Nhưng nếu bạn **đã tự viết** 1 constructor có tham số, C# sẽ **không còn tự tạo** bản rỗng đó nữa (trừ khi bạn tự viết thêm).

---

## 3. Property — cách "đúng chuẩn" để lộ dữ liệu ra ngoài

### 🧠 Ví dụ trước

```csharp
class TaiKhoanNganHang
{
    private decimal _soDu; // "private" — giấu kín, KHÔNG ai bên ngoài sửa trực tiếp được

    public decimal SoDu // PROPERTY — cổng ra vào có kiểm soát
    {
        get { return _soDu; }       // cho phép ĐỌC
        set
        {
            if (value < 0) throw new ArgumentException("Số dư không được âm");
            _soDu = value;          // cho phép GHI, nhưng CÓ KIỂM TRA
        }
    }
}

var tk = new TaiKhoanNganHang();
tk.SoDu = 1000;      // gọi "set" phía trên — qua được vì 1000 >= 0
tk.SoDu = -500;       // ⚠️ CRASH — "set" chặn lại vì có kiểm tra value < 0
```

### 📌 Giải thích

**Property** trông giống field (gọi `tk.SoDu` như biến bình thường) nhưng thực chất là **2 method ẩn** (`get` và `set`) — cho phép bạn **kiểm soát** việc đọc/ghi dữ liệu thay vì để ai cũng sửa trực tiếp field tùy ý. Đây là nguyên tắc **Encapsulation** (đóng gói) — giấu dữ liệu nội bộ (`_soDu`), chỉ cho tương tác qua "cổng" đã kiểm soát (`SoDu`).

### Cách viết gọn — Auto-property

```csharp
class SanPham
{
    public string Ten { get; set; }     // KHÔNG cần field riêng, compiler tự tạo ngầm
    public decimal Gia { get; set; }
    public string MaSP { get; init; }   // "init" — chỉ gán được LÚC TẠO object, sau đó KHÔNG sửa được nữa
}

var sp = new SanPham { Ten = "Áo thun", Gia = 150000, MaSP = "SP001" };
sp.Gia = 160000;  // ✅ OK — có "set"
sp.MaSP = "SP002"; // ⚠️ LỖI COMPILE — "init" chỉ cho gán lúc khởi tạo
```

✅ Dùng **auto-property** (`{ get; set; }`) khi không cần logic kiểm tra gì đặc biệt — ngắn gọn, đủ dùng cho phần lớn trường hợp. Chỉ viết property đầy đủ (có field `_soDu` riêng) khi cần **thêm logic kiểm tra** như ví dụ `TaiKhoanNganHang` ở trên.

---

## 4. Kế thừa (Inheritance) — tái sử dụng code giữa các class liên quan

### 🧠 Ví dụ trước

```csharp
class DongVat // class CHA
{
    public string Ten { get; set; }

    public void An()
    {
        Console.WriteLine($"{Ten} đang ăn");
    }
}

class ConCho : DongVat // ": DongVat" nghĩa là ConCho KẾ THỪA từ DongVat
{
    public void Sua()
    {
        Console.WriteLine($"{Ten} đang sủa: Gâu gâu!"); // dùng LUÔN "Ten" của class cha, không cần khai báo lại
    }
}

var cho = new ConCho();
cho.Ten = "Lu";
cho.An();   // kế thừa từ DongVat — "Lu đang ăn"
cho.Sua();  // của riêng ConCho  — "Lu đang sủa: Gâu gâu!"
```

### 📌 Giải thích

**Kế thừa** cho phép 1 class (`ConCho`, gọi là **lớp con**) **tự động có sẵn** mọi field/method của 1 class khác (`DongVat`, gọi là **lớp cha**), rồi thêm vào những gì **riêng** của nó (`Sua()`). Dùng khi nhiều class có **quan hệ "là một loại của"** (ConCho **là một loại** DongVat) và **chia sẻ chung** 1 phần hành vi/dữ liệu.

⚠️ **Đừng lạm dụng kế thừa** chỉ để "dùng chung code" khi 2 class không thật sự có quan hệ "là một loại của" nhau về mặt nghĩa — dễ tạo ra cấu trúc rối rắm, khó bảo trì. Nếu phân vân, cứ tách riêng từng class trước, kế thừa chỉ nên dùng khi quan hệ **rõ ràng và tự nhiên**.

> 📌 Phần sâu hơn của OOP (interface, abstract class, đa hình `virtual/override`, `sealed`...) được trình bày kỹ ở track [Nâng Cao](../Nang-Cao/02-OOP-And-Language-Features.md) — ở đây chỉ cần nắm vững 3 khái niệm nền tảng: Class/Object, Property, Kế thừa cơ bản.

---

## 5. `static` — thuộc về cả class, không phải riêng từng object

```csharp
class MayTinh
{
    public static int BinhPhuong(int x) => x * x; // "static" — gọi qua TÊN CLASS, không cần tạo object
}

int ketQua = MayTinh.BinhPhuong(5); // gọi thẳng "MayTinh.BinhPhuong", KHÔNG cần "new MayTinh()"
```

📌 Method/field `static` thuộc về **chính class**, dùng chung cho mọi nơi gọi tới, không cần tạo object trước. Phù hợp cho các hàm tiện ích thuần túy, không liên quan tới dữ liệu riêng của từng object (`Math.Max`, `Console.WriteLine` chính là các method `static` có sẵn của .NET).

---

## 6. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| Class vs Object | Class = khuôn mẫu, Object = thực thể thật tạo ra bằng `new` |
| Field vs Method | Field = dữ liệu lưu trữ, Method = hành vi/hành động |
| Constructor | Chạy tự động lúc `new`, đảm bảo object có đủ dữ liệu ngay từ đầu |
| Property | "Cổng" get/set có kiểm soát, ưu tiên auto-property (`{ get; set; }`) khi không cần logic đặc biệt |
| Inheritance | Lớp con kế thừa field/method của lớp cha — chỉ dùng khi quan hệ "là một loại của" rõ ràng |
| static | Thuộc về class, gọi qua tên class, không cần tạo object |

➡️ Tiếp theo: [07-Exception-Handling-Basics.md](07-Exception-Handling-Basics.md) — xử lý khi chương trình gặp lỗi.
