# 04 — Hàm / Phương thức (Functions / Methods)

---

## 1. Vì sao cần hàm

### 🧠 Ví dụ trước

```csharp
// ⚠️ KHÔNG dùng hàm — lặp lại y hệt logic 3 lần
int a1 = 5, b1 = 3;
Console.WriteLine(a1 > b1 ? a1 : b1);

int a2 = 10, b2 = 7;
Console.WriteLine(a2 > b2 ? a2 : b2);

int a3 = 1, b3 = 9;
Console.WriteLine(a3 > b3 ? a3 : b3);
```

```csharp
// ✅ DÙNG hàm — viết logic 1 lần, gọi lại nhiều lần
int TimSoLonHon(int x, int y)
{
    return x > y ? x : y;
}

Console.WriteLine(TimSoLonHon(5, 3));
Console.WriteLine(TimSoLonHon(10, 7));
Console.WriteLine(TimSoLonHon(1, 9));
```

### 📌 Giải thích

**Hàm (function/method)** là 1 khối code được **đặt tên**, nhận đầu vào (tham số), thực hiện logic, và (có thể) trả về kết quả. Lợi ích: viết 1 lần, dùng lại nhiều lần — sửa logic chỉ cần sửa **1 chỗ**, không phải tìm sửa khắp nơi.

### Cú pháp khai báo

```
<kiểu trả về>  <tên hàm>  (<kiểu tham số> <tên tham số>, ...)
{
    // thân hàm
    return <giá trị>; // nếu kiểu trả về KHÔNG PHẢI void
}
```

```csharp
int Cong(int a, int b)      // trả về int — PHẢI có return trả về 1 số int
{
    return a + b;
}

void InChao(string ten)      // "void" = KHÔNG trả về gì cả
{
    Console.WriteLine($"Chào {ten}!");
    // không có return, hoặc "return;" không kèm giá trị
}
```

---

## 2. Tham số (Parameters) và Đối số (Arguments)

```csharp
void DatHang(string tenSanPham, int soLuong)   // "tenSanPham", "soLuong": THAM SỐ (parameter) — khai báo
{
    Console.WriteLine($"Đặt {soLuong} {tenSanPham}");
}

DatHang("Áo thun", 2); // "Áo thun", 2: ĐỐI SỐ (argument) — giá trị thật truyền vào lúc gọi
```

📌 Phân biệt nhỏ: **Parameter** là tên biến khai báo trong hàm (chỗ "định nghĩa"), **Argument** là giá trị thực tế bạn truyền vào lúc gọi hàm — 2 từ hay dùng lẫn nhau trong giao tiếp hàng ngày, nhưng kỹ thuật đúng là như vậy.

### Tham số mặc định (Optional Parameters)

```csharp
void ThongBao(string noiDung, string loai = "Thông tin") // "loai" có giá trị mặc định
{
    Console.WriteLine($"[{loai}] {noiDung}");
}

ThongBao("Đã lưu thành công");          // dùng mặc định: [Thông tin] Đã lưu thành công
ThongBao("Có lỗi xảy ra", "Lỗi");       // ghi đè mặc định: [Lỗi] Có lỗi xảy ra
```

✅ Tham số có giá trị mặc định phải đặt **sau cùng** trong danh sách tham số.

### Named Arguments — gọi hàm rõ ràng hơn

```csharp
void TaoTaiKhoan(string ten, int tuoi, bool kichHoat)
{
    // ...
}

TaoTaiKhoan(ten: "An", tuoi: 20, kichHoat: true); // gọi theo TÊN, không cần đúng thứ tự, dễ đọc hơn
TaoTaiKhoan(kichHoat: true, ten: "An", tuoi: 20); // vẫn đúng, thứ tự không quan trọng khi đã ghi rõ tên
```

---

## 3. Nạp chồng hàm (Method Overloading)

### 🧠 Ví dụ trước

```csharp
int Cong(int a, int b) => a + b;
double Cong(double a, double b) => a + b;
string Cong(string a, string b) => a + b; // nối chuỗi
```

### 📌 Giải thích

**Overload** cho phép **nhiều hàm cùng tên** tồn tại, miễn **danh sách tham số khác nhau** (khác kiểu, hoặc khác số lượng). Compiler tự chọn đúng phiên bản phù hợp dựa vào kiểu/số lượng đối số bạn truyền vào lúc gọi:

```csharp
Cong(1, 2);        // gọi bản int
Cong(1.5, 2.5);    // gọi bản double
Cong("Xin ", "chào"); // gọi bản string → "Xin chào"
```

⚠️ Chỉ đổi **tên kiểu trả về** mà giữ nguyên tham số **không phải** overload hợp lệ — compiler không phân biệt được 2 hàm chỉ dựa vào kiểu trả về.

---

## 4. Expression-bodied Method — viết gọn cho hàm 1 dòng

```csharp
// Cách đầy đủ
int BinhPhuong(int x)
{
    return x * x;
}

// ✅ Cách gọn — dùng "=>" khi thân hàm chỉ có 1 biểu thức
int BinhPhuong(int x) => x * x;
```

✅ Dùng cho hàm ngắn gọn, logic đơn giản (1 dòng) — hàm phức tạp nhiều bước vẫn nên viết đầy đủ với `{}` cho dễ đọc.

---

## 5. `ref` và `out` — truyền tham số "ngược lại" ra ngoài

### 🧠 Ví dụ trước

```csharp
void TangGapDoi(int so) // truyền BÌNH THƯỜNG (by value — theo giá trị, COPY)
{
    so = so * 2;
}

int a = 5;
TangGapDoi(a);
Console.WriteLine(a); // VẪN LÀ 5! — vì hàm chỉ sửa trên 1 BẢN SAO của a, không ảnh hưởng a thật
```

### 📌 `ref` — truyền theo tham chiếu, sửa được giá trị gốc

```csharp
void TangGapDoi(ref int so) // "ref" = truyền THEO THAM CHIẾU, không phải bản sao
{
    so = so * 2;
}

int a = 5;
TangGapDoi(ref a); // PHẢI viết "ref" cả lúc gọi, để biết rõ giá trị a SẼ bị thay đổi
Console.WriteLine(a); // 10 — ĐÃ đổi thật!
```

### 📌 `out` — dùng khi hàm cần "trả về" nhiều hơn 1 giá trị

```csharp
bool ChiaHaiSo(int tuSo, int mauSo, out double ketQua)
{
    if (mauSo == 0)
    {
        ketQua = 0;
        return false; // chia cho 0 — thất bại
    }
    ketQua = (double)tuSo / mauSo;
    return true;
}

if (ChiaHaiSo(10, 2, out double result))
    Console.WriteLine($"Kết quả: {result}"); // 5
else
    Console.WriteLine("Không chia được");
```

| | Truyền thường (mặc định) | `ref` | `out` |
|---|---|---|---|
| Truyền vào hàm | Copy giá trị | Tham chiếu tới biến gốc | Tham chiếu tới biến gốc |
| Cần gán giá trị TRƯỚC khi gọi? | Có | **Có** (biến phải có giá trị sẵn) | **Không cần** (hàm sẽ gán giá trị bên trong) |
| Mục đích | Chỉ đưa dữ liệu VÀO | Sửa đổi giá trị đã có | "Trả thêm" 1 kết quả phụ ra ngoài |

✅ `out` đặc biệt hữu ích khi hàm cần trả về **cả kết quả lẫn trạng thái thành/bại** cùng lúc (như `int.TryParse(s, out int n)` đã thấy ở file 02) — thay vì phải trả về 1 object gộp phức tạp.

---

## 6. Phạm vi biến (Scope)

```csharp
void HamA()
{
    int x = 10; // x chỉ TỒN TẠI bên trong HamA()
    Console.WriteLine(x);
}

void HamB()
{
    Console.WriteLine(x); // ⚠️ LỖI COMPILE — "x" không tồn tại ở đây, nó thuộc về HamA()
}
```

📌 Biến khai báo bên trong 1 hàm chỉ **"sống"** trong phạm vi hàm đó (gọi là **local scope**) — hàm khác không nhìn thấy, không dùng được. Đây là điều **tốt**: tránh các hàm vô tình ảnh hưởng lẫn nhau qua biến dùng chung.

---

## 7. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| Hàm (function/method) | Khối code đặt tên, viết 1 lần dùng lại nhiều lần |
| Parameter vs Argument | Tham số lúc khai báo vs giá trị thật lúc gọi |
| Optional parameter | Giá trị mặc định, phải đặt cuối danh sách tham số |
| Overload | Nhiều hàm cùng tên, khác tham số — compiler tự chọn đúng bản |
| `=>` (expression-bodied) | Viết gọn cho hàm 1 dòng |
| `ref` | Truyền theo tham chiếu, sửa được giá trị gốc, biến phải có giá trị trước |
| `out` | "Trả thêm" 1 kết quả ra ngoài, không cần gán giá trị trước khi gọi |
| Scope | Biến local chỉ tồn tại trong phạm vi hàm khai báo nó |

➡️ Tiếp theo: [05-Arrays-Strings-Collections.md](05-Arrays-Strings-Collections.md) — lưu trữ nhiều dữ liệu cùng lúc.
