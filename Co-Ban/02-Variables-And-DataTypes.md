# 02 — Biến, Kiểu dữ liệu, Toán tử

---

## 1. Biến là gì

### 🧠 Ví dụ trước

```csharp
int tuoi = 25;
string hoTen = "Nguyễn Văn A";
double chieuCao = 1.75;
bool daKetHon = false;

Console.WriteLine($"{hoTen}, {tuoi} tuổi, cao {chieuCao}m");
```

### 📌 Giải thích

**Biến (variable)** là 1 "cái hộp có tên" dùng để lưu trữ dữ liệu, dữ liệu trong hộp có thể thay đổi sau này. Cú pháp khai báo:

```
<kiểu dữ liệu>  <tên biến>  =  <giá trị>;
```

C# là ngôn ngữ **kiểu tĩnh (statically-typed)** — 1 khi biến đã khai báo kiểu gì, nó **chỉ chứa được đúng kiểu đó** suốt vòng đời:

```csharp
int soLuong = 10;
soLuong = "mười"; // ⚠️ LỖI COMPILE — không thể gán chuỗi vào biến kiểu int
```

⚠️ Đây là tin tốt, không phải hạn chế: compiler **bắt lỗi ngay khi viết code**, thay vì để chương trình chạy sai lúc người dùng đang dùng thật.

### Quy tắc đặt tên biến

- Bắt đầu bằng chữ cái hoặc `_`, không bắt đầu bằng số.
- Theo quy ước, dùng **camelCase** cho biến (`hoTen`, `soLuong`), **PascalCase** cho tên class/method (`Program`, `TinhTong`).
- Đặt tên **có ý nghĩa**: `soLuongSanPham` tốt hơn `x` hay `sl`.

---

## 2. Các kiểu dữ liệu cơ bản

| Kiểu | Chứa gì | Ví dụ |
|---|---|---|
| `int` | Số nguyên (phổ biến nhất cho số đếm) | `int tuoi = 25;` |
| `double` | Số thực (thập phân), độ chính xác tương đối | `double pi = 3.14;` |
| `decimal` | Số thực, **chính xác tuyệt đối** — dùng cho **tiền tệ** | `decimal gia = 19999.99m;` (chú ý hậu tố `m`) |
| `bool` | Đúng/Sai (`true`/`false`) | `bool daXong = true;` |
| `char` | 1 ký tự duy nhất (trong dấu nháy đơn) | `char kyTu = 'A';` |
| `string` | Chuỗi ký tự (trong dấu nháy kép) | `string ten = "An";` |

### ⚠️ Vì sao tiền tệ phải dùng `decimal`, không dùng `double`

```csharp
double a = 0.1 + 0.2;
Console.WriteLine(a); // 0.30000000000000004 — KHÔNG đúng 0.3!

decimal b = 0.1m + 0.2m;
Console.WriteLine(b); // 0.3 — chính xác tuyệt đối
```

📌 `double` lưu số theo hệ nhị phân, một số thập phân "tròn trịa" trong hệ 10 (như 0.1) **không biểu diễn chính xác tuyệt đối** được trong hệ nhị phân — gây sai số rất nhỏ nhưng tích lũy lại thành vấn đề lớn nếu tính toán tiền bạc nhiều lần. `decimal` được thiết kế riêng để tránh đúng vấn đề này.

✅ **Quy tắc:** tiền tệ, điểm số cần chính xác tuyệt đối → `decimal`. Tính toán khoa học, đồ họa, không cần chính xác tuyệt đối → `double`.

### `var` — để compiler tự đoán kiểu

```csharp
var tuoi = 25;        // compiler tự hiểu tuoi là int
var ten = "An";        // compiler tự hiểu ten là string
```

📌 `var` không phải "kiểu động" — compiler vẫn xác định kiểu chính xác **ngay lúc biên dịch**, chỉ là bạn không cần gõ lại tên kiểu. Dùng `var` khi kiểu dữ liệu đã rõ ràng từ vế phải, giúp code gọn hơn.

### `const` — giá trị không bao giờ đổi

```csharp
const double PI = 3.14159;
PI = 3.14; // ⚠️ LỖI COMPILE — const không được gán lại
```

✅ Dùng `const` cho những giá trị **chắc chắn cố định** (số Pi, số ngày trong tuần = 7...).

---

## 3. Ép kiểu (Type Casting)

### 🧠 Ví dụ trước

```csharp
int soNguyen = 10;
double soThuc = soNguyen; // ✅ TỰ ĐỘNG — int → double không bao giờ mất dữ liệu

double a = 9.7;
int b = (int)a; // phải CHỦ ĐỘNG viết (int) — vì CÓ THỂ mất phần thập phân
Console.WriteLine(b); // 9 — phần .7 bị CẮT BỎ, không làm tròn!
```

### 📌 Giải thích

- **Ép kiểu ngầm định (implicit)**: compiler tự làm, khi chuyển đổi **không bao giờ mất dữ liệu** (int → double, vì double chứa được mọi giá trị int).
- **Ép kiểu tường minh (explicit)**: bạn phải tự viết `(kiểu)`, dùng khi chuyển đổi **có thể mất dữ liệu** (double → int, mất phần thập phân).

### Chuyển đổi giữa `string` và số

```csharp
// string → số
string s = "123";
int n = int.Parse(s);           // ⚠️ CRASH nếu chuỗi không phải số hợp lệ
int n2 = int.Parse("abc");      // ném lỗi FormatException ngay lập tức

bool thanhCong = int.TryParse("abc", out int ketQua); // ✅ AN TOÀN — không crash
if (thanhCong)
    Console.WriteLine($"Đã chuyển thành công: {ketQua}");
else
    Console.WriteLine("Không phải số hợp lệ");

// số → string
int tuoi = 25;
string tuoiStr = tuoi.ToString();
string tuoiStr2 = Convert.ToString(tuoi); // cách khác, tương đương
```

✅ **Quy tắc:** khi dữ liệu nhập từ người dùng (không chắc chắn đúng định dạng), **luôn dùng `TryParse`** thay vì `Parse` để tránh crash chương trình — học sâu hơn về xử lý lỗi ở file 07.

---

## 4. Toán tử (Operators)

### Toán tử số học

```csharp
int a = 10, b = 3;
Console.WriteLine(a + b);  // 13 — cộng
Console.WriteLine(a - b);  // 7  — trừ
Console.WriteLine(a * b);  // 30 — nhân
Console.WriteLine(a / b);  // 3  — chia LẤY NGUYÊN vì cả 2 đều là int!
Console.WriteLine(a % b);  // 1  — chia lấy DƯ (modulo)

double c = 10.0 / 3; // 3.333... — MUỐN chia có thập phân, ít nhất 1 bên phải là double
```

⚠️ **Lỗi rất hay gặp:** `10 / 3` với cả 2 là `int` cho kết quả `3` (bỏ phần dư), không phải `3.33`. Muốn phép chia có phần thập phân, ít nhất 1 trong 2 số phải là `double`/`decimal`.

### Toán tử so sánh (trả về `bool`)

```csharp
Console.WriteLine(5 == 5);  // true  — bằng nhau
Console.WriteLine(5 != 3);  // true  — khác nhau
Console.WriteLine(5 > 3);   // true  — lớn hơn
Console.WriteLine(5 <= 5);  // true  — nhỏ hơn hoặc bằng
```

⚠️ Phân biệt `=` (gán giá trị) và `==` (so sánh bằng nhau) — nhầm 2 cái này là lỗi cú pháp phổ biến nhất của người mới.

### Toán tử luận lý (kết hợp điều kiện bool)

```csharp
bool coTien = true;
bool coThe = false;

Console.WriteLine(coTien && coThe);  // false — AND: CẢ HAI phải true
Console.WriteLine(coTien || coThe);  // true  — OR: CHỈ CẦN 1 cái true
Console.WriteLine(!coTien);          // false — NOT: đảo ngược giá trị
```

### Toán tử gán rút gọn và tăng/giảm

```csharp
int x = 10;
x += 5;  // tương đương x = x + 5;   → x = 15
x -= 3;  // tương đương x = x - 3;   → x = 12
x++;     // tương đương x = x + 1;   → x = 13
x--;     // tương đương x = x - 1;   → x = 12
```

### Toán tử 3 ngôi (Ternary) — if/else viết gọn trong 1 dòng

```csharp
int tuoi = 20;
string loai = tuoi >= 18 ? "Người lớn" : "Trẻ em"; // điều_kiện ? giá_trị_nếu_đúng : giá_trị_nếu_sai
```

✅ Dùng khi logic đơn giản, chỉ 2 nhánh — nếu logic phức tạp hơn, nên dùng `if/else` đầy đủ (học ở file 03) cho dễ đọc.

---

## 5. Giá trị `null` — biến "chưa có gì"

```csharp
string? ten = null; // dấu "?" cho phép biến string được phép null
int? tuoi = null;    // dấu "?" cho phép biến int (vốn KHÔNG BAO GIỜ null) được phép null

if (ten == null)
    Console.WriteLine("Chưa có tên");
```

📌 `null` nghĩa là "biến này hiện không trỏ tới giá trị nào cả". Với kiểu số như `int`, mặc định **không được phép** null (luôn phải có giá trị, mặc định là `0`) — muốn cho phép null, phải thêm dấu `?` (`int?`). Đây là khái niệm sẽ gặp lại sâu hơn khi làm việc với database (1 cột có thể "chưa nhập").

### Dấu `!` — null-forgiving operator (khác với `!` phép toán NOT)

```csharp
string? dong = Console.ReadLine(); // ReadLine() CÓ THỂ trả về null (ví dụ Ctrl+Z ở console)
int doDai = dong.Length;           // ⚠️ CẢNH BÁO: "dong" có thể null, truy cập .Length không an toàn

int doDai2 = dong!.Length;         // ✅ dấu "!" báo compiler: "tôi CHẮC CHẮN dong không null lúc này, đừng cảnh báo nữa"
```

⚠️ Dễ nhầm với `!` ở mục Toán tử luận lý (phần 4 — phủ định `true`/`false`) nhưng đây là **2 ký hiệu khác nhau hoàn toàn**, chỉ trùng mặt chữ:

| | `!` đứng TRƯỚC 1 biến `bool` | `!` đứng SAU 1 biến (null-forgiving) |
|---|---|---|
| Ý nghĩa | Đảo ngược `true` ↔ `false` | "Tôi chắc chắn giá trị này không null" |
| Có ảnh hưởng lúc chạy không | **Có** — thật sự đảo giá trị | **Không** — chỉ tắt cảnh báo lúc biên dịch, giá trị giữ nguyên |

⚠️ **Quan trọng:** `!` null-forgiving **không hề kiểm tra** gì cả lúc chạy — nếu bạn đoán sai (giá trị thật sự là `null`), chương trình vẫn **crash** với `NullReferenceException` y như không viết `!`. Nó chỉ là lời hứa với compiler, không phải phép màu. Chỉ dùng khi bạn **thật sự chắc chắn** (ví dụ: vừa kiểm tra `if (ten != null)` ngay phía trên).

---

## 6. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| int/double/decimal/bool/char/string | 6 kiểu dữ liệu cơ bản nhất, nhớ decimal cho tiền tệ |
| var | Compiler tự đoán kiểu lúc biên dịch, không phải kiểu "động" |
| Ép kiểu ngầm vs tường minh | Ngầm = an toàn, tự động. Tường minh = có thể mất dữ liệu, phải tự viết `(kiểu)` |
| Parse vs TryParse | Parse crash nếu sai định dạng, TryParse an toàn — luôn ưu tiên TryParse với dữ liệu không chắc chắn |
| int / int | Chia 2 số nguyên cho kết quả nguyên, mất phần thập phân |
| == vs = | == so sánh, = gán giá trị — đừng nhầm |
| null / int? | `null` = chưa có giá trị; thêm `?` để kiểu giá trị (int, bool...) được phép null |
| `!` null-forgiving | Khác với `!` phủ định bool — chỉ tắt cảnh báo compile, KHÔNG kiểm tra gì lúc chạy, vẫn crash nếu đoán sai |

➡️ Tiếp theo: [03-Control-Flow.md](03-Control-Flow.md) — điều khiển luồng chạy của chương trình.
