# 01 — Làm quen với C# và .NET

---

## 1. C# là gì, .NET là gì

Nói ngắn gọn để bắt đầu code được ngay, không cần hiểu sâu lúc này:

- **C#** là **ngôn ngữ lập trình** — thứ bạn gõ ra để viết logic chương trình.
- **.NET** là **nền tảng chạy chương trình đó** — gồm công cụ build, thư viện dựng sẵn (đọc file, kết nối database, làm web...), và "cỗ máy" thực thi code của bạn.

> 📌 Ví von dễ hiểu: C# giống như **tiếng Việt** (ngôn ngữ để diễn đạt), còn .NET giống như **cuốn từ điển + bộ máy hiểu tiếng Việt đó** để biến nó thành hành động thật.

.NET hiện tại (từ bản .NET 5 trở đi, không còn gọi ".NET Core" nữa) chạy được trên **Windows, macOS, Linux** — và miễn phí hoàn toàn.

---

## 2. Cài đặt môi trường

1. Tải **.NET SDK** (bộ công cụ để viết và chạy code) tại [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download) — chọn bản **LTS** (Long Term Support, ổn định lâu dài) nếu không chắc chọn bản nào.
2. Cài 1 trong các công cụ soạn code sau (chọn 1 cái, không cần cả 3):
   - **Visual Studio Code** (nhẹ, miễn phí, phổ biến nhất để học) + extension **C# Dev Kit**.
   - **Visual Studio Community** (đầy đủ tính năng hơn, nặng hơn, miễn phí cho cá nhân/học tập).
   - **JetBrains Rider** (trả phí, rất mạnh, nhiều dev chuyên nghiệp dùng).
3. Kiểm tra cài đặt thành công bằng cách mở terminal/cmd, gõ:

```bash
dotnet --version
```

Nếu hiện ra số phiên bản (ví dụ `9.0.100`) — xong, sẵn sàng code.

---

## 3. Chương trình đầu tiên

### 🧠 Tạo project

```bash
dotnet new console -n HelloWorldApp
cd HelloWorldApp
dotnet run
```

Lệnh `dotnet new console` tạo ra 1 project console (chạy trong terminal, không có giao diện), `dotnet run` build và chạy ngay. Bạn sẽ thấy dòng chữ `Hello, World!` hiện ra.

### 📌 Nhìn vào file `Program.cs` vừa được tạo

```csharp
Console.WriteLine("Hello, World!");
```

Chỉ đúng **1 dòng**! Đây là cú pháp hiện đại gọi là **top-level statements** (từ C# 9 trở đi) — cho phép viết thẳng code mà không cần khai báo `class`/`Main` lòng vòng, rất phù hợp để học và viết script ngắn.

### 📌 Nhưng thực chất "bên dưới" vẫn là cấu trúc đầy đủ này

```csharp
using System; // "mượn" các công cụ có sẵn từ thư viện System (ở đây là Console)

namespace HelloWorldApp // "ngăn tủ" chứa code, tránh trùng tên với code người khác
{
    class Program // MỌI code C# đều phải nằm trong 1 class
    {
        static void Main(string[] args) // "Main" là nơi chương trình BẮT ĐẦU CHẠY — bắt buộc phải có, đúng 1 cái
        {
            Console.WriteLine("Hello, World!"); // in ra màn hình, tự động xuống dòng
        }
    }
}
```

| Thành phần | Vai trò |
|---|---|
| `using System;` | Cho phép dùng các công cụ nằm trong "ngăn tủ" tên `System` (ví dụ `Console`) mà không cần gõ tên đầy đủ `System.Console` |
| `namespace` | Gom nhóm code lại theo dự án/thư mục — tránh 2 class trùng tên ở 2 chỗ khác nhau bị xung đột |
| `class Program` | C# là ngôn ngữ **hướng đối tượng** (OOP, học kỹ ở file 06) — code bắt buộc phải nằm trong 1 class |
| `static void Main(string[] args)` | **Điểm bắt đầu** của chương trình — CLR (bộ máy chạy .NET) tìm đúng hàm này để khởi chạy đầu tiên |
| `Console.WriteLine(...)` | In ra 1 dòng chữ, rồi tự xuống dòng. Có `Console.Write(...)` (không tự xuống dòng) nếu cần |

✅ **Khi học và viết code ngắn:** cứ dùng cú pháp 1-dòng (top-level statements) cho gọn. Khi làm dự án thật sự (nhiều file, nhiều class), cấu trúc `namespace`/`class`/`Main` đầy đủ vẫn luôn tồn tại phía sau — chỉ là bạn không thấy nó khi mới bắt đầu.

---

## 4. Comment — ghi chú trong code

```csharp
// Đây là comment 1 dòng — compiler bỏ qua hoàn toàn, chỉ để NGƯỜI đọc hiểu

/* Đây là comment
   nhiều dòng */

/// <summary>
/// Comment dạng này (3 dấu gạch chéo) dùng để viết TÀI LIỆU cho hàm/class,
/// IDE sẽ hiện nội dung này khi ai đó gọi tới hàm/class này
/// </summary>
void DoSomething() { }
```

✅ Comment dùng để giải thích **vì sao** code viết như vậy (lý do), không phải để lặp lại **cái gì** code đang làm (code đã tự nói lên điều đó nếu đặt tên rõ ràng).

---

## 5. Nhập dữ liệu từ bàn phím

```csharp
Console.Write("Nhập tên của bạn: ");
string? ten = Console.ReadLine(); // đọc 1 dòng người dùng gõ vào, rồi Enter

Console.WriteLine($"Xin chào, {ten}!"); // "$" cho phép nhúng biến thẳng vào chuỗi — gọi là string interpolation
```

📌 `Console.ReadLine()` luôn trả về kiểu `string` (chuỗi chữ) — nếu muốn nhập số, phải **chuyển đổi kiểu** (học ở file 02):

```csharp
Console.Write("Nhập tuổi: ");
int tuoi = int.Parse(Console.ReadLine()!); // chuyển chuỗi "25" thành số 25
Console.WriteLine($"Năm sau bạn {tuoi + 1} tuổi");
```

> 📌 Dấu `!` cuối `Console.ReadLine()!` gọi là **null-forgiving operator** — khác hoàn toàn với `!` ở phép toán luận lý (NOT, học ở file 02). Nó chỉ là cách bảo trình biên dịch: "tôi biết rõ giá trị này không phải `null`, đừng cảnh báo nữa" — không làm thay đổi gì lúc chạy. Xem giải thích đầy đủ ở file 02, mục 5.

⚠️ Nếu người dùng gõ chữ không phải số (ví dụ "abc") vào chỗ cần `int.Parse`, chương trình sẽ **crash** (ném lỗi) — cách xử lý an toàn hơn sẽ học ở file 07 (Xử lý lỗi).

---

## 6. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| C# vs .NET | C# là ngôn ngữ để viết code, .NET là nền tảng chạy code đó |
| `dotnet new console` | Tạo project console mới |
| `dotnet run` | Build và chạy project |
| `Main` | Điểm bắt đầu chương trình, bắt buộc phải có |
| `Console.WriteLine` / `Write` | In ra màn hình, có/không tự xuống dòng |
| `Console.ReadLine()` | Đọc 1 dòng nhập từ bàn phím, luôn trả về `string` |
| `$"..."` | String interpolation — nhúng biến thẳng vào chuỗi |

➡️ Tiếp theo: [02-Variables-And-DataTypes.md](02-Variables-And-DataTypes.md) — học cách lưu trữ và thao tác dữ liệu.
