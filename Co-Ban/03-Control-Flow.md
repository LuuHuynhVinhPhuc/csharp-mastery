# 03 — Cấu trúc điều khiển (Control Flow)

---

## 1. `if / else` — rẽ nhánh theo điều kiện

### 🧠 Ví dụ trước

```csharp
int diem = 7;

if (diem >= 8)
{
    Console.WriteLine("Giỏi");
}
else if (diem >= 6.5)
{
    Console.WriteLine("Khá");
}
else
{
    Console.WriteLine("Trung bình");
}
```

### 📌 Giải thích

Chương trình kiểm tra **từ trên xuống**, điều kiện nào `true` **đầu tiên** thì chạy khối đó rồi **bỏ qua hết** các điều kiện còn lại — dù những điều kiện sau cũng có thể đúng.

⚠️ **Lỗi hay gặp:** quên dấu `{}`  khi khối lệnh chỉ có 1 dòng:

```csharp
if (diem >= 8)
    Console.WriteLine("Giỏi");
    Console.WriteLine("Chúc mừng!"); // ⚠️ dòng này LUÔN CHẠY, dù diem < 8 — vì nó KHÔNG nằm trong if!
```

✅ **Quy tắc an toàn:** luôn dùng `{}` dù chỉ có 1 dòng lệnh, tránh nhầm lẫn như trên.

---

## 2. `switch` — nhiều nhánh theo 1 giá trị cụ thể

### 🧠 Ví dụ trước

```csharp
int thu = 3;
string tenThu;

switch (thu)
{
    case 1:
        tenThu = "Chủ Nhật";
        break;
    case 2:
        tenThu = "Thứ Hai";
        break;
    case 3:
        tenThu = "Thứ Ba";
        break;
    default:
        tenThu = "Không xác định";
        break;
}

Console.WriteLine(tenThu);
```

📌 Dùng `switch` khi cần so sánh **1 biến** với **nhiều giá trị cụ thể** — thay vì viết `if (thu == 1) ... else if (thu == 2) ...` dài dòng. `break` để **dừng lại**, không chạy tiếp xuống `case` dưới (nếu quên `break`, C# sẽ báo lỗi compile — khác với một số ngôn ngữ khác tự "rớt" xuống case dưới).

### ✅ Cú pháp hiện đại hơn — `switch` dạng biểu thức (từ C# 8+)

```csharp
string tenThu = thu switch
{
    1 => "Chủ Nhật",
    2 => "Thứ Hai",
    3 => "Thứ Ba",
    _ => "Không xác định" // "_" nghĩa là "mọi trường hợp còn lại"
};
```

Gọn hơn nhiều — không cần `break`, không cần lặp lại tên biến (`tenThu = `) ở mỗi dòng, và **bắt buộc** phải có `_` (default) nếu không phủ hết mọi trường hợp (compiler sẽ cảnh báo).

---

## 3. Vòng lặp `for` — lặp theo số lần biết trước

### 🧠 Ví dụ trước

```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Lần lặp thứ {i}");
}
// In ra: Lần lặp thứ 0, 1, 2, 3, 4 (CHÚ Ý: bắt đầu từ 0, kết thúc TRƯỚC khi i = 5)
```

### 📌 Giải thích cấu trúc `for (khởi_tạo; điều_kiện; bước_nhảy)`

| Phần | Vai trò | Ví dụ |
|---|---|---|
| `int i = 0` | Chạy **1 lần duy nhất** lúc bắt đầu | khởi tạo biến đếm |
| `i < 5` | Kiểm tra **trước mỗi lần lặp** — còn `true` thì lặp tiếp | điều kiện dừng |
| `i++` | Chạy **sau mỗi lần lặp** | tăng biến đếm |

⚠️ **Lỗi hay gặp nhất — off-by-one (lệch 1):** nhầm lẫn giữa `i < 5` (lặp 5 lần: 0,1,2,3,4) và `i <= 5` (lặp 6 lần: 0,1,2,3,4,5). Luôn kiểm tra kỹ xem cần lặp đúng bao nhiêu lần.

---

## 4. Vòng lặp `while` và `do-while` — lặp theo điều kiện, không biết trước số lần

```csharp
int soLuong = 0;
while (soLuong < 3) // kiểm tra điều kiện TRƯỚC, có thể KHÔNG chạy lần nào nếu sai ngay từ đầu
{
    Console.WriteLine($"Đang xử lý đơn hàng thứ {soLuong}");
    soLuong++;
}
```

```csharp
int lua;
do
{
    Console.Write("Nhập số từ 1-10: ");
    lua = int.Parse(Console.ReadLine()!);
} while (lua < 1 || lua > 10); // kiểm tra SAU — do đó LUÔN CHẠY ÍT NHẤT 1 LẦN
```

| | `while` | `do-while` |
|---|---|---|
| Kiểm tra điều kiện | **Trước** khi chạy khối lệnh | **Sau** khi chạy khối lệnh |
| Số lần chạy tối thiểu | 0 lần (nếu điều kiện sai ngay từ đầu) | **Luôn ít nhất 1 lần** |

✅ `do-while` phù hợp cho tình huống **chắc chắn cần làm ít nhất 1 lần** trước khi kiểm tra — ví dụ: yêu cầu nhập lại cho tới khi đúng định dạng (phải hỏi ít nhất 1 lần).

⚠️ **Lỗi vòng lặp vô hạn:** quên cập nhật biến điều kiện (`soLuong++`) khiến điều kiện **không bao giờ** sai, chương trình treo mãi mãi. Luôn kiểm tra: có cách nào để vòng lặp **chắc chắn dừng lại** không?

---

## 5. `foreach` — lặp qua từng phần tử của 1 tập hợp

```csharp
string[] traiCay = { "Táo", "Cam", "Xoài" };

foreach (string tc in traiCay)
{
    Console.WriteLine(tc);
}
```

📌 `foreach` dùng để **duyệt qua từng phần tử** trong mảng/danh sách (học kỹ ở file 05), không cần tự quản lý biến đếm như `for` — đơn giản và ít lỗi hơn khi chỉ cần "lấy ra từng cái một", không cần biết vị trí (index) của nó.

---

## 6. `break` và `continue`

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 5)
        break; // THOÁT HẲN khỏi vòng lặp ngay lập tức

    if (i % 2 == 0)
        continue; // BỎ QUA phần còn lại của lần lặp này, nhảy tới lần lặp TIẾP THEO

    Console.WriteLine(i); // chỉ in ra số LẺ, và dừng hẳn khi i = 5
}
// In ra: 1, 3
```

| Từ khóa | Hành động |
|---|---|
| `break` | Thoát **hoàn toàn** khỏi vòng lặp, không quay lại nữa |
| `continue` | Bỏ qua phần còn lại **của lần lặp hiện tại**, vòng lặp vẫn tiếp tục ở lần kế tiếp |

---

## 7. Tóm tắt

| Khái niệm | Ghi nhớ |
|---|---|
| if/else | Kiểm tra từ trên xuống, điều kiện đúng ĐẦU TIÊN được chạy — luôn dùng `{}` dù 1 dòng |
| switch | So sánh 1 biến với nhiều giá trị cụ thể — có dạng biểu thức gọn (`switch` + `=>`) |
| for | Lặp biết trước số lần — cẩn thận lỗi off-by-one giữa `<` và `<=` |
| while | Kiểm tra TRƯỚC — có thể không chạy lần nào |
| do-while | Kiểm tra SAU — luôn chạy ít nhất 1 lần |
| foreach | Duyệt từng phần tử trong tập hợp, không cần quản lý index |
| break / continue | Thoát hẳn vòng lặp / bỏ qua lần lặp hiện tại, tiếp tục lần sau |

➡️ Tiếp theo: [04-Functions-And-Methods.md](04-Functions-And-Methods.md) — gói code thành những khối tái sử dụng được.
