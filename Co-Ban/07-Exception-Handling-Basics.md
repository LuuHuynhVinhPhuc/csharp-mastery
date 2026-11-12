# 07 — Xử lý lỗi cơ bản (Exception Handling)

---

## 1. Exception là gì

### 🧠 Ví dụ trước

```csharp
int a = 10;
int b = 0;
int ketQua = a / b; // ⚠️ CRASH NGAY LẬP TỨC — chương trình dừng lại, in ra lỗi đỏ

Console.WriteLine("Dòng này KHÔNG BAO GIỜ chạy tới");
```

Chạy thử sẽ thấy chương trình dừng đột ngột và hiện ra:
```
Unhandled exception. System.DivideByZeroException: Attempted to divide by zero.
```

### 📌 Giải thích

**Exception (ngoại lệ)** là 1 lỗi xảy ra **lúc chương trình đang chạy** (khác với lỗi cú pháp bị compiler bắt lúc viết code) — chia cho 0, mở file không tồn tại, truy cập phần tử mảng không có, ép kiểu sai... Nếu không ai "bắt" lỗi này lại, chương trình sẽ **dừng hẳn ngay tại dòng gây lỗi**.

---

## 2. `try / catch` — "bắt" lỗi lại, không cho crash

### 🧠 Ví dụ trước

```csharp
int a = 10;
int b = 0;

try
{
    int ketQua = a / b; // nếu dòng NÀY lỗi...
    Console.WriteLine(ketQua);
}
catch (DivideByZeroException ex) // ...nhảy thẳng vào ĐÂY, không crash chương trình
{
    Console.WriteLine("Lỗi: không thể chia cho 0!");
}

Console.WriteLine("Chương trình vẫn tiếp tục chạy bình thường"); // dòng này VẪN chạy được
```

### 📌 Giải thích

```
try
{
    // code CÓ THỂ gây lỗi
}
catch (LoaiLoi ex)
{
    // code xử lý KHI lỗi xảy ra — thay vì để chương trình crash
}
```

Nếu lỗi xảy ra bên trong khối `try`, chương trình **ngay lập tức nhảy** sang khối `catch` tương ứng, bỏ qua phần còn lại của `try` — rồi **tiếp tục chạy bình thường** sau khối `catch`, không hề crash.

### Một vài loại Exception thường gặp

| Exception | Xảy ra khi nào |
|---|---|
| `DivideByZeroException` | Chia 1 số nguyên cho 0 |
| `IndexOutOfRangeException` | Truy cập mảng với index không hợp lệ (file 05) |
| `NullReferenceException` | Gọi method/truy cập property trên 1 biến đang là `null` |
| `FormatException` | `int.Parse("abc")` — chuỗi không đúng định dạng số |
| `FileNotFoundException` | Mở 1 file không tồn tại trên đĩa |

### Bắt nhiều loại lỗi khác nhau

```csharp
try
{
    Console.Write("Nhập 1 số: ");
    int so = int.Parse(Console.ReadLine());
    Console.WriteLine(100 / so);
}
catch (FormatException)
{
    Console.WriteLine("Bạn phải nhập 1 SỐ, không phải chữ!");
}
catch (DivideByZeroException)
{
    Console.WriteLine("Không thể nhập số 0!");
}
catch (Exception ex) // bắt MỌI lỗi còn lại chưa được xử lý ở trên — LUÔN để cuối cùng
{
    Console.WriteLine($"Có lỗi khác xảy ra: {ex.Message}");
}
```

⚠️ **Quy tắc thứ tự:** `catch` càng **cụ thể** phải đặt **trước**, `catch (Exception ex)` (bắt mọi lỗi) phải đặt **cuối cùng** — vì compiler kiểm tra từ trên xuống, đặt `Exception` lên đầu sẽ "nuốt" hết mọi lỗi, không bao giờ chạy tới các `catch` cụ thể bên dưới.

---

## 3. `finally` — luôn chạy, dù có lỗi hay không

```csharp
try
{
    Console.WriteLine("Đang mở file...");
    throw new Exception("Giả lập lỗi"); // giả lập có lỗi xảy ra
}
catch (Exception ex)
{
    Console.WriteLine($"Lỗi: {ex.Message}");
}
finally
{
    Console.WriteLine("Đóng file lại"); // LUÔN CHẠY — dù try thành công, thất bại, hay crash giữa chừng
}
```

📌 `finally` dùng cho những việc **bắt buộc phải làm** bất kể kết quả ra sao — ví dụ đóng file đã mở, đóng kết nối database — để tránh "rò rỉ" tài nguyên nếu quên dọn dẹp.

---

## 4. `throw` — tự tạo ra lỗi khi phát hiện tình huống bất thường

### 🧠 Ví dụ trước

```csharp
void RutTien(decimal soDu, decimal soTienRut)
{
    if (soTienRut > soDu)
    {
        throw new Exception("Số dư không đủ để rút!"); // CHỦ ĐỘNG báo lỗi
    }
    Console.WriteLine($"Rút thành công {soTienRut}");
}

try
{
    RutTien(100000, 500000);
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message); // "Số dư không đủ để rút!"
}
```

📌 Dùng `throw` khi hàm của bạn phát hiện 1 tình huống **không hợp lệ về mặt nghiệp vụ** — thay vì để chương trình âm thầm chạy tiếp với dữ liệu sai, chủ động báo lỗi ngay để người gọi hàm biết và xử lý.

---

## 5. ⚠️ Lỗi phổ biến nhất của người mới: "nuốt" lỗi mà không xử lý gì

```csharp
try
{
    LamViecGiDo();
}
catch (Exception ex)
{
    // ⚠️ ĐỂ TRỐNG — tệ hơn cả không có try/catch!
    // Lỗi xảy ra nhưng KHÔNG AI BIẾT, chương trình cứ chạy tiếp như không có chuyện gì
}
```

✅ **Tối thiểu**, luôn nên **in ra** hoặc **ghi log** lỗi để biết chuyện gì đã xảy ra:

```csharp
catch (Exception ex)
{
    Console.WriteLine($"Đã xảy ra lỗi: {ex.Message}");
}
```

---

## 6. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| Exception | Lỗi xảy ra LÚC CHẠY, nếu không bắt sẽ làm crash chương trình |
| try/catch | Bắt lỗi lại, chương trình tiếp tục chạy thay vì crash |
| Thứ tự catch | Cụ thể trước, `Exception` (chung nhất) luôn để CUỐI |
| finally | Luôn chạy dù try thành công hay thất bại — dùng để dọn dẹp |
| throw | Chủ động báo lỗi khi phát hiện tình huống nghiệp vụ không hợp lệ |
| catch rỗng | KHÔNG BAO GIỜ để trống — luôn ít nhất in ra lỗi để biết chuyện gì xảy ra |

➡️ Tiếp theo: [08-Intro-Async-And-LINQ.md](08-Intro-Async-And-LINQ.md) — làm quen với bất đồng bộ và truy vấn dữ liệu.
