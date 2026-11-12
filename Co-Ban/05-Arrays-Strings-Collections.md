# 05 — Mảng, Chuỗi, và Collections cơ bản

---

## 1. Mảng (Array) — nhiều giá trị cùng kiểu, số lượng cố định

### 🧠 Ví dụ trước

```csharp
int[] diem = { 8, 7, 9, 6, 10 }; // khai báo kèm giá trị luôn

Console.WriteLine(diem[0]);      // 8  — phần tử ĐẦU TIÊN, chỉ số (index) bắt đầu từ 0
Console.WriteLine(diem[4]);      // 10 — phần tử CUỐI, index = Length - 1
Console.WriteLine(diem.Length);  // 5  — tổng số phần tử
```

### 📌 Giải thích

Mảng lưu **nhiều giá trị cùng 1 kiểu dữ liệu**, truy cập qua **chỉ số (index)** — luôn **bắt đầu từ 0**, không phải 1. Mảng có **kích thước cố định** ngay lúc tạo, không thể thêm/bớt phần tử sau đó (muốn linh hoạt hơn, xem `List<T>` ở mục 3).

```csharp
int[] mangRong = new int[5]; // tạo mảng 5 phần tử, TẤT CẢ mặc định = 0 (vì int mặc định là 0)
mangRong[0] = 100;
mangRong[1] = 200;
```

⚠️ **Lỗi hay gặp nhất: `IndexOutOfRangeException`**

```csharp
int[] a = { 1, 2, 3 };
Console.WriteLine(a[3]); // ⚠️ CRASH — mảng có 3 phần tử, index hợp lệ chỉ từ 0 tới 2, KHÔNG có a[3]!
```

✅ Luôn nhớ: index hợp lệ chạy từ `0` tới `Length - 1`.

### Duyệt mảng

```csharp
for (int i = 0; i < diem.Length; i++)
    Console.WriteLine(diem[i]); // dùng for khi CẦN biết vị trí (index)

foreach (int d in diem)
    Console.WriteLine(d); // dùng foreach khi CHỈ cần lấy từng giá trị, không cần biết vị trí
```

---

## 2. Chuỗi (String) — những thao tác dùng hằng ngày

```csharp
string ten = "  Nguyễn Văn An  ";

Console.WriteLine(ten.Length);          // 17 — độ dài CHUỖI GỐC (tính cả khoảng trắng)
Console.WriteLine(ten.Trim());          // "Nguyễn Văn An" — cắt khoảng trắng 2 đầu
Console.WriteLine(ten.ToUpper());       // "  NGUYỄN VĂN AN  " — viết hoa toàn bộ
Console.WriteLine(ten.ToLower());       // "  nguyễn văn an  " — viết thường toàn bộ
Console.WriteLine(ten.Contains("Văn")); // true — có chứa chuỗi con không
Console.WriteLine(ten.Replace("An", "Bình")); // thay thế 1 đoạn chuỗi

string[] parts = "a,b,c".Split(','); // TÁCH chuỗi theo dấu phẩy → ["a", "b", "c"]
string joined = string.Join(" - ", parts); // GHÉP mảng chuỗi lại → "a - b - c"

string hoTen = "An" + " " + "Nguyễn"; // nối chuỗi bằng "+"
string loiChao = $"Xin chào, {hoTen}!"; // string interpolation — nhúng biến (đã học ở file 01)

string doan = "Xin chào các bạn";
string con = doan.Substring(5);     // "chào các bạn" — lấy từ vị trí 5 tới hết
string con2 = doan.Substring(5, 4); // "chào"         — lấy từ vị trí 5, lấy đúng 4 ký tự
```

⚠️ **Quan trọng:** mọi method ở trên (`Trim`, `ToUpper`, `Replace`...) **không sửa chuỗi gốc** — chúng **trả về 1 chuỗi MỚI**. Đây là vì `string` trong C# là **bất biến (immutable)**:

```csharp
string s = "hello";
s.ToUpper();               // ⚠️ KHÔNG LÀM GÌ CẢ với "s" — kết quả bị "ném đi", không ai nhận lại
Console.WriteLine(s);      // "hello" — VẪN NGUYÊN, không đổi!

s = s.ToUpper();           // ✅ ĐÚNG — phải GÁN LẠI kết quả trả về
Console.WriteLine(s);      // "HELLO"
```

---

## 3. `List<T>` — mảng nhưng linh hoạt về kích thước

### 🧠 Ví dụ trước

```csharp
List<string> gioHang = new List<string>(); // danh sách RỖNG ban đầu

gioHang.Add("Áo thun");
gioHang.Add("Quần jean");
gioHang.Add("Giày");

Console.WriteLine(gioHang.Count);   // 3 — số lượng phần tử HIỆN TẠI (tương tự Length của mảng)
gioHang.Remove("Quần jean");        // xóa theo GIÁ TRỊ
Console.WriteLine(gioHang[0]);      // "Áo thun" — vẫn truy cập qua index như mảng

foreach (string sp in gioHang)
    Console.WriteLine(sp);
```

### 📌 Khác biệt với `Array`

| | `Array` (`int[]`) | `List<T>` |
|---|---|---|
| Kích thước | Cố định ngay lúc tạo | Linh hoạt — `Add`/`Remove` thoải mái |
| Khi nào dùng | Biết trước chính xác số lượng, không đổi | **Mặc định nên dùng** khi không chắc số lượng sẽ cố định |

✅ Trong thực tế, `List<T>` được dùng **nhiều hơn hẳn** `Array` thuần vì tính linh hoạt — chỉ dùng `Array` khi thật sự chắc chắn kích thước cố định.

### Các thao tác thường dùng với `List<T>`

```csharp
List<int> so = new List<int> { 5, 2, 8, 1 };

so.Sort();                          // sắp xếp TẠI CHỖ → {1, 2, 5, 8}
bool co8 = so.Contains(8);          // true — có tồn tại giá trị 8 không
int viTri = so.IndexOf(5);          // vị trí (index) của giá trị 5, -1 nếu không có
so.Insert(0, 100);                  // chèn 100 vào vị trí 0 (đầu danh sách)
so.Clear();                         // xóa SẠCH toàn bộ phần tử
```

---

## 4. `Dictionary<TKey, TValue>` — tra cứu theo "khóa" thay vì theo vị trí

### 🧠 Ví dụ trước

```csharp
Dictionary<string, int> tonKho = new Dictionary<string, int>();

tonKho["Áo thun"] = 50;   // thêm/ghi đè theo KEY (khóa)
tonKho["Quần jean"] = 30;
tonKho.Add("Giày", 20);    // cách khác để thêm — nhưng Add() SẼ LỖI nếu key đã tồn tại rồi

Console.WriteLine(tonKho["Áo thun"]); // 50 — tra cứu theo khóa, KHÔNG phải theo vị trí 0,1,2...
```

### 📌 Giải thích

`Dictionary` lưu dữ liệu theo từng **cặp (khóa — giá trị)**, giống như 1 cuốn từ điển thật: bạn tra theo **từ khóa** (key) để lấy ra **nghĩa** (value), không quan tâm nó nằm ở "vị trí" nào. Đây là lựa chọn lý tưởng khi dữ liệu có 1 "tên định danh" tự nhiên (tên sản phẩm, mã khách hàng, email...).

⚠️ **Lỗi hay gặp: tra khóa không tồn tại**

```csharp
Console.WriteLine(tonKho["Mũ"]); // ⚠️ CRASH — KeyNotFoundException, vì "Mũ" chưa được thêm vào
```

✅ **Cách tra cứu an toàn:**

```csharp
if (tonKho.ContainsKey("Mũ"))
    Console.WriteLine(tonKho["Mũ"]);
else
    Console.WriteLine("Không có sản phẩm này");

// ✅ Gọn hơn — 1 bước thay vì 2 (tránh phải tra 2 lần)
if (tonKho.TryGetValue("Mũ", out int soLuong))
    Console.WriteLine(soLuong);
else
    Console.WriteLine("Không có sản phẩm này");
```

### Duyệt qua `Dictionary`

```csharp
foreach (KeyValuePair<string, int> item in tonKho)
    Console.WriteLine($"{item.Key}: {item.Value}");

// hoặc gọn hơn với cú pháp giải cấu trúc (deconstruction)
foreach (var (ten, soLuong) in tonKho)
    Console.WriteLine($"{ten}: {soLuong}");
```

---

## 5. Khi nào dùng cái nào — tóm tắt quyết định nhanh

| Tình huống | Dùng |
|---|---|
| Biết chắc số lượng cố định, không đổi | `Array` |
| Danh sách có thể thêm/bớt phần tử | `List<T>` |
| Cần tra cứu nhanh theo "tên"/"mã" thay vì vị trí | `Dictionary<TKey, TValue>` |

> 📌 Còn nhiều cấu trúc dữ liệu khác (`Queue`, `Stack`, `HashSet`...) và lý do kỹ thuật đằng sau việc chọn lựa (độ phức tạp thuật toán) — tìm hiểu kỹ hơn ở track [Nâng Cao](../Nang-Cao/02-OOP-And-Language-Features.md) khi đã vững phần cơ bản này.

---

## 6. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| Array | Kích thước cố định, index bắt đầu từ 0, cẩn thận IndexOutOfRangeException |
| string là immutable | Mọi method (Trim, ToUpper...) trả về chuỗi MỚI, phải gán lại mới có hiệu lực |
| List\<T\> | Giống mảng nhưng linh hoạt, `Add`/`Remove`/`Count` — nên dùng mặc định |
| Dictionary\<TKey,TValue\> | Tra cứu theo khóa, dùng `TryGetValue` để tránh crash khi khóa không tồn tại |

➡️ Tiếp theo: [06-OOP-Basics.md](06-OOP-Basics.md) — tổ chức code theo hướng đối tượng.
