# 08 — Nhập môn Async/Await và LINQ

---

## 1. Vì sao cần `async`/`await`

### 🧠 Ví dụ trước

```csharp
using var httpClient = new HttpClient();

string ketQua = httpClient.GetStringAsync("https://api.github.com").Result; // ⚠️ KHÔNG NÊN — có thể gây treo app
Console.WriteLine(ketQua);
```

```csharp
// ✅ Cách đúng
using var httpClient = new HttpClient();

string ketQua = await httpClient.GetStringAsync("https://api.github.com"); // "await" — chờ ĐÚNG CÁCH
Console.WriteLine(ketQua);
```

### 📌 Giải thích ở mức cần biết để dùng đúng

Khi chương trình gọi ra bên ngoài (gọi API, đọc file, truy vấn database...), nó phải **chờ** kết quả trả về — việc chờ này **không tốn CPU xử lý gì cả**, chỉ đơn giản là ngồi đợi. `await` là cách báo cho chương trình: "trong lúc chờ cái này, hãy để máy làm việc khác, đừng ngồi khóa cứng (block) không làm gì".

✅ **Quy tắc thực dụng cho người mới — chỉ cần nhớ đúng 3 điều này:**

1. Hàm nào bên trong có `await`, hàm đó **phải** đánh dấu `async`, và kiểu trả về phải là `Task` (nếu không trả gì) hoặc `Task<T>` (nếu trả về giá trị kiểu `T`):

```csharp
async Task<string> LayDuLieuAsync() // "Task<string>" vì bên trong return về string
{
    using var client = new HttpClient();
    string data = await client.GetStringAsync("https://api.github.com");
    return data;
}
```

2. Khi **gọi** 1 hàm `async`, luôn dùng `await` phía trước (trừ vài tình huống nâng cao sẽ học sau):

```csharp
string data = await LayDuLieuAsync(); // luôn "await" khi gọi hàm async
```

3. **Không bao giờ** dùng `.Result` hoặc `.Wait()` thay cho `await` — 2 cách này **chờ theo kiểu chặn cứng (blocking)**, mất hết lợi ích của async, và trong 1 số trường hợp còn gây **treo ứng dụng hẳn (deadlock)**.

> 📌 Vì sao đúng là "không tốn thread để chờ" và cơ chế bên dưới `async`/`await` hoạt động thế nào (state machine, ThreadPool...) được giải thích kỹ ở track [Nâng Cao](../Nang-Cao/03-Exception-And-Async.md) — ở đây chỉ cần nhớ **3 quy tắc thực dụng** ở trên để viết code đúng.

### `Main` cũng có thể là `async`

```csharp
async Task Main() // cho phép dùng "await" ngay trong Main
{
    string data = await LayDuLieuAsync();
    Console.WriteLine(data);
}
```

Với cú pháp top-level statements (file 01), chỉ cần viết thẳng:

```csharp
using var client = new HttpClient();
string data = await client.GetStringAsync("https://api.github.com"); // "await" dùng được NGAY, không cần khai báo Main
Console.WriteLine(data);
```

---

## 2. LINQ — truy vấn dữ liệu trên danh sách, gọn hơn viết vòng lặp tay

### 🧠 Ví dụ trước

```csharp
List<int> tuoi = new List<int> { 15, 22, 17, 30, 8, 45 };

// ⚠️ Cách CŨ — viết vòng lặp tay để lọc người lớn (>= 18)
List<int> nguoiLon = new List<int>();
foreach (int t in tuoi)
{
    if (t >= 18)
        nguoiLon.Add(t);
}
```

```csharp
// ✅ Cách DÙNG LINQ — 1 dòng, dễ đọc hơn hẳn
using System.Linq; // nhớ "using" thư viện này

List<int> nguoiLon = tuoi.Where(t => t >= 18).ToList();
```

### 📌 Giải thích các method LINQ hay dùng nhất

```csharp
List<int> so = new List<int> { 5, 2, 8, 1, 9, 3 };

var locSoLon = so.Where(x => x > 3);         // LỌC — chỉ giữ phần tử thỏa điều kiện
var nhanDoi = so.Select(x => x * 2);          // BIẾN ĐỔI — áp dụng công thức lên từng phần tử
var sapXep = so.OrderBy(x => x);              // SẮP XẾP tăng dần
var sapXepGiam = so.OrderByDescending(x => x); // SẮP XẾP giảm dần
int tong = so.Sum();                          // TỔNG tất cả phần tử
int lonNhat = so.Max();                       // giá trị LỚN NHẤT
double trungBinh = so.Average();              // TRUNG BÌNH CỘNG
int demSoLon = so.Count(x => x > 3);          // ĐẾM số phần tử thỏa điều kiện
bool coSoAm = so.Any(x => x < 0);             // CÓ phần tử nào thỏa điều kiện không? (true/false)
int dauTien = so.First(x => x > 3);           // phần tử ĐẦU TIÊN thỏa điều kiện (crash nếu không có)
int dauTienAnToan = so.FirstOrDefault(x => x > 100); // 0 (giá trị mặc định) nếu KHÔNG tìm thấy, không crash
```

| Method | Vai trò | Ghi nhớ |
|---|---|---|
| `Where` | Lọc theo điều kiện | Trả về **nhiều** phần tử thỏa điều kiện |
| `Select` | Biến đổi từng phần tử | "Map" mỗi phần tử sang 1 giá trị/kiểu khác |
| `OrderBy` / `OrderByDescending` | Sắp xếp | Tăng dần / giảm dần |
| `First` / `FirstOrDefault` | Lấy 1 phần tử đầu tiên | `First` crash nếu không có, `FirstOrDefault` trả về mặc định (an toàn hơn) |
| `Any` | Kiểm tra tồn tại | Trả về `bool`, không lấy ra giá trị |
| `Sum` / `Max` / `Min` / `Average` / `Count` | Tính toán tổng hợp | Dùng trên dãy số |

### Kết hợp nhiều bước LINQ lại với nhau

```csharp
List<string> ten = new List<string> { "Bình", "An", "Châu", "Đạt", "Em" };

var ketQua = ten
    .Where(t => t.Length > 2)     // chỉ giữ tên dài hơn 2 ký tự
    .OrderBy(t => t)               // sắp xếp theo alphabet
    .Select(t => t.ToUpper())      // viết hoa toàn bộ
    .ToList();                      // chuyển kết quả thành List để dùng

// Kết quả: ["BÌNH", "CHÂU", "ĐẠT"]
```

✅ Mỗi bước LINQ có thể **nối tiếp nhau** (gọi là *method chaining*) — đọc từ trên xuống giống như 1 "dây chuyền xử lý": lọc → sắp xếp → biến đổi → lấy kết quả. Đây là lý do LINQ thường **dễ đọc hơn** nhiều so với viết vòng lặp `for`/`foreach` lồng nhau nhiều bước.

⚠️ Luôn nhớ gọi `.ToList()` (hoặc `.ToArray()`) ở **cuối cùng** nếu muốn lưu kết quả lại dùng nhiều lần — nếu không, mỗi lần duyệt lại `ketQua` ở trên, toàn bộ chuỗi `Where/OrderBy/Select` sẽ **chạy lại từ đầu** (lý do kỹ thuật được giải thích sâu hơn ở track Nâng Cao).

---

## 3. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| async/await | Hàm gọi I/O (API, file, DB) nên là `async Task`/`Task<T>`, luôn `await` khi gọi |
| .Result / .Wait() | KHÔNG dùng — nguy cơ treo ứng dụng, luôn dùng `await` |
| Where | Lọc phần tử theo điều kiện |
| Select | Biến đổi từng phần tử sang giá trị khác |
| OrderBy | Sắp xếp danh sách |
| First vs FirstOrDefault | First crash nếu không có, FirstOrDefault an toàn hơn |
| .ToList() cuối chuỗi LINQ | Lưu lại kết quả, tránh chạy lại từ đầu mỗi lần dùng |

---

➡️ Đã hoàn thành track Cơ Bản! Bước tiếp theo: quay lại [00-README.md](00-README.md) của track này để ôn lại tổng quan, hoặc bắt đầu track [Nâng Cao](../Nang-Cao/00-README.md) để hiểu sâu bản chất (vùng nhớ, OOP nâng cao, kiến trúc hệ thống...).
