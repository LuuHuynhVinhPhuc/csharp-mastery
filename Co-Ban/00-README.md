# C# Cơ Bản — Dành cho người mới tiếp cận

Đây là track dành cho người **mới học C# lần đầu** — tập trung vào những gì cần biết để **đọc hiểu và tự viết được code** cơ bản. Không đi vào cơ chế runtime, kiến trúc hệ thống hay tối ưu hiệu năng — những thứ đó nằm ở track [../Nang-Cao](../Nang-Cao), dành cho sau khi đã vững phần này.

## Cấu trúc tài liệu

| File | Nội dung |
|---|---|
| [01-Getting-Started.md](01-Getting-Started.md) | C#/.NET là gì, cài đặt, chương trình đầu tiên, cấu trúc 1 file C# |
| [02-Variables-And-DataTypes.md](02-Variables-And-DataTypes.md) | Biến, kiểu dữ liệu, ép kiểu, toán tử |
| [03-Control-Flow.md](03-Control-Flow.md) | if/else, switch, for, while, do-while, break/continue |
| [04-Functions-And-Methods.md](04-Functions-And-Methods.md) | Khai báo hàm, tham số, giá trị trả về, overload, ref/out |
| [05-Arrays-Strings-Collections.md](05-Arrays-Strings-Collections.md) | Mảng, string, List\<T\>, Dictionary\<TKey,TValue\> — dùng sao cho đúng |
| [06-OOP-Basics.md](06-OOP-Basics.md) | Class, Object, Constructor, Property, Kế thừa cơ bản |
| [07-Exception-Handling-Basics.md](07-Exception-Handling-Basics.md) | try/catch/finally, Exception là gì |
| [08-Intro-Async-And-LINQ.md](08-Intro-Async-And-LINQ.md) | async/await dùng sao cho đúng, LINQ cơ bản (Where/Select/OrderBy) |

## Lộ trình đọc

Đọc **tuần tự từ 01 → 08** — mỗi file xây dựng trên kiến thức của file trước, không nên nhảy cóc. Mỗi chủ đề đều có ví dụ chạy được ngay — gõ lại và chạy thử là cách học nhanh nhất, đừng chỉ đọc suông.

## Phạm vi (có chủ đích bỏ qua)

Track này **không** phải "mọi thứ cơ bản" — đây là những topic **trọng tâm nhất** để đọc hiểu và viết được code đúng. Một số thứ cố tình bỏ qua vì không thuộc "trọng tâm ngôn ngữ", bạn sẽ gặp khi làm bài tập/dự án thực tế và có thể tự tra cứu riêng:

- `Random` (sinh số ngẫu nhiên) — hay dùng trong bài tập (đoán số, xúc xắc...), nhưng là 1 class cụ thể, không phải khái niệm ngôn ngữ.
- Đọc/ghi file cơ bản (`File.ReadAllText`, `StreamReader`...) — thuộc nhóm I/O, không phải cú pháp nền tảng.
- Tổ chức project nhiều file/nhiều class/namespace thực tế — tài liệu này dùng ví dụ gọn trong 1 file để tập trung vào khái niệm.

---

## Quy ước

- 🧠 **Ví dụ trước** — nhìn code chạy được trước khi đọc giải thích.
- 📌 **Giải thích** — vì sao code hoạt động như vậy.
- ⚠️ **Lỗi hay gặp** — lỗi người mới thường mắc phải.
- ✅ **Ghi nhớ** — điều quan trọng nhất của mục đó.

➡️ Học xong track này, chuyển sang [Nâng Cao](../Nang-Cao/00-README.md) — hoặc xem lại [mục lục chung](../README.md).
