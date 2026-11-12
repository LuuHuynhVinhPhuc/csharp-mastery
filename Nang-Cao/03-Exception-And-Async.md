# 03 — Exception Handling, Threading, và Async/Await

---

## 1. Exception Handling — không chỉ là `try/catch` cho qua chuyện

### 🧠 Ví dụ trước

```csharp
try
{
    ProcessOrder(order);
}
catch (Exception ex)
{
    // nuốt lỗi, không làm gì cả
}
```

Đoạn code này **còn tệ hơn** không có try/catch — vì khi `ProcessOrder` ném lỗi (ví dụ database mất kết nối, dữ liệu sai), chương trình **im lặng chạy tiếp** như không có chuyện gì, để lại hệ quả (đơn hàng không được xử lý) mà không ai biết, rất khó debug về sau.

### 📌 Bản chất: Exception là cơ chế "unwind the stack"

Khi một exception được `throw`, CLR làm việc sau:
1. Dừng ngay method hiện tại.
2. Tìm trong method đó có khối `try/catch` nào khớp với loại exception không.
3. Nếu không có, **"unwind" (tháo dỡ) stack frame đó** (pop ra khỏi Stack — nhớ lại file 01), rồi tìm tiếp lên method cha (người gọi method này), cứ thế lần lên.
4. Nếu đi tới tận cùng (`Main`) mà không ai catch → CLR in ra "Unhandled Exception" và **kill process**.

Đây là lý do exception "đắt" hơn nhiều so với `if/else` thông thường về mặt hiệu năng (phải build stack trace, tháo dỡ từng frame) — **không nên dùng exception để điều khiển luồng (control flow) bình thường**, chỉ dùng cho tình huống **thật sự bất thường**.

```csharp
// ⚠️ SAI: dùng exception để kiểm tra tồn tại — rất tốn hiệu năng nếu chạy nhiều lần
try { return dictionary[key]; }
catch (KeyNotFoundException) { return null; }

// ✅ ĐÚNG: dùng method được thiết kế sẵn để tránh exception
if (dictionary.TryGetValue(key, out var value)) return value;
return null;
```

### 1.1. Thứ tự catch — cụ thể trước, tổng quát sau

```csharp
try
{
    File.ReadAllText(path);
}
catch (FileNotFoundException ex)   // cụ thể nhất — bắt trước
{
    Console.WriteLine("File không tồn tại");
}
catch (IOException ex)             // tổng quát hơn (FileNotFoundException kế thừa từ đây)
{
    Console.WriteLine("Lỗi đọc/ghi file");
}
catch (Exception ex)               // tổng quát nhất — luôn để CUỐI CÙNG
{
    Console.WriteLine("Lỗi không xác định");
}
```

⚠️ Nếu đặt `catch (Exception ex)` lên trước, compiler sẽ báo lỗi "unreachable code" cho các catch cụ thể phía sau — vì `Exception` đã bắt hết mọi loại rồi.

### 1.2. `finally` — luôn chạy, kể cả khi có `return`

```csharp
int Test()
{
    try
    {
        return 1;
    }
    finally
    {
        Console.WriteLine("Finally vẫn chạy trước khi return 1 thực sự được trả về");
    }
}
```

📌 `finally` dùng để dọn dẹp tài nguyên (đóng file, đóng connection) **bất kể** method thoát ra bằng cách nào: return bình thường, throw exception, hay break ra khỏi vòng lặp. Đây chính là cơ chế nằm dưới từ khóa `using` đã nói ở file 01.

### 1.3. Custom Exception — khi nào nên tự tạo

```csharp
public class InsufficientBalanceException : Exception
{
    public decimal CurrentBalance { get; }
    public decimal RequiredAmount { get; }

    public InsufficientBalanceException(decimal current, decimal required)
        : base($"Số dư không đủ: hiện có {current}, cần {required}")
    {
        CurrentBalance = current;
        RequiredAmount = required;
    }
}
```

✅ **Khi nào dùng:** khi bên gọi (caller) cần **thông tin cụ thể để xử lý khác nhau** tùy loại lỗi nghiệp vụ (ví dụ: lỗi "không đủ tiền" cần hiển thị số dư, khác với lỗi "tài khoản khóa"). Nếu chỉ là lỗi chung chung không cần xử lý riêng, dùng `Exception` có sẵn là đủ, không cần bày vẽ thêm class.

### 1.4. Inner Exception — đừng nuốt lỗi gốc khi wrap lại

```csharp
try
{
    SaveToDatabase(order);
}
catch (SqlException ex)
{
    // ⚠️ SAI — ném exception mới, MẤT HOÀN TOÀN thông tin lỗi gốc (stack trace cũ biến mất)
    throw new ApplicationException("Lỗi khi lưu đơn hàng");

    // ✅ ĐÚNG — giữ lại exception gốc trong InnerException để debug sau này
    throw new ApplicationException("Lỗi khi lưu đơn hàng", ex);
}
```

Và phân biệt 2 cách re-throw:

```csharp
catch (Exception ex)
{
    throw;      // ✅ giữ nguyên TOÀN BỘ stack trace gốc — biết chính xác dòng nào gây lỗi
    throw ex;   // ⚠️ reset stack trace về vị trí này — mất dấu vết lỗi thật sự bắt đầu từ đâu
}
```

---

## 2. Threading — nền tảng trước khi học async/await

> Nhiều người học `async/await` mà bỏ qua Threading nên hiểu sai rằng "async = chạy đa luồng". **Không phải vậy** — xem kỹ mục 3 bên dưới.

### 🧠 Ví dụ trước

```csharp
var thread = new Thread(() => Console.WriteLine("Chạy trên thread khác"));
thread.Start();
Console.WriteLine("Chạy trên thread chính");
// Thứ tự in ra KHÔNG đảm bảo — 2 dòng chạy song song thật sự, trên 2 lõi CPU khác nhau (nếu có)
```

### 📌 Bản chất

Mỗi **Thread** là một luồng thực thi độc lập, có Stack riêng (nhớ lại file 01 — Stack là "theo từng thread"), nhưng **dùng chung Heap** với các thread khác trong cùng process. Chính vì dùng chung Heap, nhiều thread có thể cùng đọc/ghi 1 object → sinh ra **race condition**.

### 🧠 Ví dụ Race Condition

```csharp
int counter = 0;
void Increment() { for (int i = 0; i < 100_000; i++) counter++; }

var t1 = new Thread(Increment);
var t2 = new Thread(Increment);
t1.Start(); t2.Start();
t1.Join(); t2.Join();
Console.WriteLine(counter); // KỲ VỌNG 200,000 nhưng thường ra SỐ NHỎ HƠN!
```

### 📌 Vì sao sai

`counter++` **trông như 1 phép toán** nhưng thực chất CPU thực hiện **3 bước**: (1) đọc giá trị counter, (2) cộng 1, (3) ghi lại. Nếu t1 đọc counter = 50, đang cộng thì t2 cũng đọc counter = 50 (chưa thấy t1 ghi lại), cả 2 cùng ghi lại 51 → **mất đi 1 lần tăng**. Đây gọi là **race condition**.

### 2.1. `lock` và `Monitor` — độc quyền tương hỗ trong CÙNG 1 process

```csharp
private readonly object _lockObj = new();
int counter = 0;

void Increment()
{
    lock (_lockObj) // chỉ 1 thread được vào bên trong khối này tại 1 thời điểm
    {
        counter++;
    }
}
```

📌 `lock (_lockObj) { ... }` thực chất là cú pháp rút gọn (syntactic sugar) mà compiler viết lại thành:

```csharp
Monitor.Enter(_lockObj);
try
{
    counter++;
}
finally
{
    Monitor.Exit(_lockObj); // "finally" đảm bảo LUÔN mở khóa, kể cả khi có exception bên trong
}
```

`Monitor` là cơ chế **độc quyền tương hỗ (mutual exclusion)** cấp hệ điều hành nhưng chỉ hoạt động **trong phạm vi 1 process** — nó gắn 1 "chìa khóa ảo" lên object được truyền vào (`_lockObj`), chỉ 1 thread cầm được chìa khóa đó tại 1 thời điểm, thread khác gọi `Monitor.Enter` lên cùng object sẽ **bị block (ngủ)** cho tới khi chìa khóa được `Exit` trả lại.

⚠️ **Vì sao hầu như luôn dùng `lock` thay vì tự gọi `Monitor.Enter/Exit` tay:** quên bọc `try/finally` là lỗi rất dễ mắc — nếu exception xảy ra giữa khối code mà chưa kịp `Monitor.Exit`, **chìa khóa sẽ không bao giờ được trả lại**, mọi thread khác chờ mãi mãi (một dạng deadlock, xem mục 2.5). `lock` buộc bạn luôn có `finally` đúng cách, không thể viết sai.

#### `Monitor.TryEnter` — thử lấy khóa, không chờ vô thời hạn

```csharp
if (Monitor.TryEnter(_lockObj, TimeSpan.FromSeconds(2))) // chờ TỐI ĐA 2 giây
{
    try { counter++; }
    finally { Monitor.Exit(_lockObj); }
}
else
{
    Console.WriteLine("Không lấy được khóa sau 2 giây — bỏ qua, tránh treo vô thời hạn");
}
```

✅ Dùng khi bạn **không chấp nhận chờ đợi vô hạn** — ví dụ 1 thao tác UI không được phép "đứng hình", thà bỏ qua hoặc báo lỗi còn hơn treo app.

#### `Monitor.Wait` / `Pulse` — cơ chế báo hiệu giữa các thread (Producer-Consumer)

### 🧠 Ví dụ trước

Một thread "sản xuất" dữ liệu bỏ vào hàng đợi, một thread khác "tiêu thụ" — thread tiêu thụ cần biết **khi nào** có dữ liệu mới để xử lý, thay vì liên tục kiểm tra lãng phí CPU (gọi là *busy-waiting*, hình dung như liên tục hỏi "có chưa, có chưa" thay vì ngồi yên chờ được gọi tên).

```csharp
private readonly object _lockObj = new();
private readonly Queue<int> _queue = new();

void Produce(int item)
{
    lock (_lockObj)
    {
        _queue.Enqueue(item);
        Monitor.Pulse(_lockObj); // "đánh thức" 1 thread đang Wait, báo "có hàng mới rồi"
    }
}

void Consume()
{
    lock (_lockObj)
    {
        while (_queue.Count == 0)
            Monitor.Wait(_lockObj); // NGỦ ở đây, nhả khóa ra cho thread khác, tới khi được Pulse mới tỉnh lại và lấy khóa lại
        var item = _queue.Dequeue();
    }
}
```

📌 `Monitor.Wait` đặc biệt ở chỗ: nó **tạm nhả khóa đang giữ** rồi cho thread "ngủ", để thread khác (như `Produce`) vào được khối `lock` cùng object — nếu không nhả khóa, `Produce` không bao giờ vào được để `Pulse` đánh thức `Consume`, gây deadlock ngay lập tức. Khi được `Pulse`, thread ngủ tỉnh dậy, **tự động lấy lại khóa** rồi tiếp tục chạy từ đúng chỗ `Wait()`. Vòng `while` (không phải `if`) là bắt buộc — vì giữa lúc được đánh thức và lúc thật sự chạy tiếp, có thể một thread khác đã "nẫng tay trên" hết hàng trong queue rồi, cần kiểm tra lại điều kiện cho chắc (gọi là *spurious wakeup* / tranh chấp lại tài nguyên).

> ✅ Trong code thực tế hiện đại, hiếm khi cần tự viết tay `Wait`/`Pulse` như trên — `BlockingCollection<T>` hoặc `Channel<T>` (file 02 có nhắc `ConcurrentCollection`) đã đóng gói sẵn đúng pattern Producer-Consumer này, an toàn và ít lỗi hơn tự viết tay.

### 2.2. Mutex — độc quyền tương hỗ GIỮA NHIỀU PROCESS

### 🧠 Ví dụ trước

Bạn muốn đảm bảo ứng dụng Desktop của mình **chỉ được mở 1 lần duy nhất** trên máy (user bấm chạy file .exe 2 lần, lần thứ 2 phải bị chặn hoặc focus về cửa sổ cũ) — `lock`/`Monitor` **không giải quyết được** vì đó là 2 process hoàn toàn độc lập, không dùng chung bộ nhớ để chia sẻ 1 object `_lockObj`.

```csharp
bool isNewInstance;
using var mutex = new Mutex(initiallyOwned: true, "Global\\MyApp_SingleInstance_Mutex", out isNewInstance);

if (!isNewInstance)
{
    Console.WriteLine("Ứng dụng đã đang chạy rồi!");
    return; // thoát ngay, không cho chạy process thứ 2
}

// ... chạy ứng dụng bình thường
```

### 📌 Bản chất

`Mutex` là cơ chế độc quyền tương hỗ **do hệ điều hành quản lý** (không chỉ CLR), nên nó hoạt động được **xuyên suốt nhiều process khác nhau**, miễn là chúng cùng dùng chung 1 **tên (name)** — ở ví dụ trên là chuỗi `"Global\\MyApp_SingleInstance_Mutex"`. Process thứ 2 cố tạo `Mutex` cùng tên sẽ nhận biết ngay "đã có ai giữ cái tên này rồi" qua tham số `out isNewInstance`.

⚠️ `Mutex` **chậm hơn đáng kể** so với `lock`/`Monitor` (vì phải đi qua lời gọi hệ điều hành thật sự — kernel-mode object — thay vì chỉ xử lý trong phạm vi CLR/user-mode). ✅ **Quy tắc:** chỉ dùng `Mutex` khi thật sự cần đồng bộ **giữa các process khác nhau** (single-instance app, hoặc 2 ứng dụng riêng biệt cùng ghi 1 file chung trên đĩa). Nếu chỉ đồng bộ giữa các thread trong cùng 1 ứng dụng, `lock`/`Monitor` luôn là lựa chọn đúng và nhanh hơn.

### 2.3. Semaphore / SemaphoreSlim — giới hạn SỐ LƯỢNG thread được vào cùng lúc

### 🧠 Ví dụ trước

Gọi 1 API bên ngoài giới hạn tối đa **3 kết nối đồng thời** (vượt quá sẽ bị từ chối hoặc phạt). Bạn có 100 request cần gọi — `lock` không phù hợp (chỉ cho **1** thread vào, trong khi ở đây muốn cho phép **tối đa 3** cùng lúc).

```csharp
private readonly SemaphoreSlim _semaphore = new(initialCount: 3, maxCount: 3); // tối đa 3 "chỗ trống"

async Task CallExternalApiAsync(int requestId)
{
    await _semaphore.WaitAsync(); // XIN 1 "chỗ trống" — nếu đã đủ 3 thread khác đang chạy, CHỜ ở đây
    try
    {
        Console.WriteLine($"Request {requestId} đang gọi API...");
        await Task.Delay(1000); // giả lập gọi API thật
    }
    finally
    {
        _semaphore.Release(); // TRẢ LẠI "chỗ trống" cho thread khác đang chờ dùng
    }
}

var tasks = Enumerable.Range(1, 100).Select(id => CallExternalApiAsync(id));
await Task.WhenAll(tasks); // 100 tác vụ, nhưng CHỈ TỐI ĐA 3 CÁI chạy "song song" tại bất kỳ thời điểm nào
```

### 📌 Bản chất

`Semaphore` quản lý 1 "bộ đếm chỗ trống" (gọi là *count*) — mỗi lần `WaitAsync()`/`Wait()`, bộ đếm giảm 1 (nếu còn chỗ, thread đi tiếp ngay; nếu hết chỗ — về 0 — thread phải **chờ**); mỗi lần `Release()`, bộ đếm tăng lại 1, đánh thức 1 thread đang chờ (nếu có). Khởi tạo `new(3, 3)` nghĩa là "tối đa 3 chỗ, hiện có sẵn 3 chỗ trống".

| | `Semaphore` | `SemaphoreSlim` |
|---|---|---|
| Phạm vi | Có bản **named**, dùng được **giữa nhiều process** (giống Mutex) | Chỉ trong **cùng 1 process**, nhẹ hơn, nhanh hơn |
| Hỗ trợ async | Không có `WaitAsync` | **Có** `WaitAsync()` — không block thread thật trong lúc chờ (sẽ giải thích rõ ở mục 3 bên dưới — async không tốn thread) |
| Khi nào dùng | Giới hạn truy cập tài nguyên dùng chung **giữa nhiều ứng dụng/process** | **99% trường hợp thực tế trong 1 ứng dụng .NET** — giới hạn số lượng gọi API/DB đồng thời, giới hạn số tác vụ nặng chạy song song |

✅ **Phân biệt nhanh 3 công cụ đã học:** `lock`/`Monitor` = đúng **1** thread vào, trong **cùng process**. `Mutex` = đúng **1** thread/process vào, nhưng hoạt động **xuyên process**. `Semaphore`/`SemaphoreSlim` = cho phép **N** thread vào cùng lúc (N ≥ 1 tùy cấu hình) — bản chất `Semaphore` với `maxCount: 1` sẽ hoạt động gần giống `Mutex`/`lock`, nhưng 2 công cụ kia vẫn tối ưu hơn hẳn cho đúng trường hợp "chỉ 1 thread" của chúng.

### 2.4. `Interlocked` — thao tác nguyên tử (atomic) không cần lock cả khối

```csharp
long counter = 0;
Interlocked.Increment(ref counter); // cộng 1, ĐẢM BẢO không bị race condition, KHÔNG CẦN lock
Interlocked.Add(ref counter, 10);
Interlocked.CompareExchange(ref counter, newValue: 100, comparand: 0); // chỉ gán 100 NẾU counter hiện đang = 0
```

📌 Với những phép toán cực kỳ đơn giản trên 1 biến số (tăng/giảm/gán có điều kiện), CPU hiện đại hỗ trợ sẵn các lệnh **atomic** (không thể bị ngắt giữa chừng bởi thread khác) ở tầng phần cứng. `Interlocked` tận dụng trực tiếp các lệnh này — **nhanh hơn `lock` rất nhiều** (không cần xin/trả khóa, không có thread nào phải "ngủ chờ"), nhưng chỉ áp dụng được cho phép toán đơn lẻ như trên, không dùng được khi cần bảo vệ **cả 1 khối logic nhiều bước** (lúc đó vẫn phải quay về `lock`).

### 2.5. Deadlock — khi 2 thread chờ nhau mãi mãi

```csharp
object lockA = new(), lockB = new();

// Thread 1:
lock (lockA) { Thread.Sleep(100); lock (lockB) { /* ... */ } }
// Thread 2:
lock (lockB) { Thread.Sleep(100); lock (lockA) { /* ... */ } }
// Thread 1 giữ A, chờ B. Thread 2 giữ B, chờ A. → KẸT VĨNH VIỄN.
```

✅ **Quy tắc tránh deadlock:** luôn lock theo **một thứ tự cố định, nhất quán** ở mọi nơi trong code (ví dụ luôn lock A trước B), không bao giờ đảo ngược thứ tự ở chỗ khác.

### 2.6. ThreadPool — vì sao không nên tự `new Thread()` tùy tiện

📌 Tạo 1 `Thread` thật sự khá "đắt": tốn ~1MB Stack (file 01), tốn thời gian hệ điều hành khởi tạo. Nếu code tạo hàng ngàn Thread cho hàng ngàn tác vụ ngắn, hệ thống sẽ ngộp. **ThreadPool** giải quyết việc này bằng cách **giữ sẵn một nhóm thread tái sử dụng** — bạn "gửi việc" vào hàng đợi (queue), ThreadPool giao cho thread rảnh xử lý, xong thì thread đó quay lại hàng đợi chờ việc tiếp theo thay vì bị hủy. `Task` (nói bên dưới) mặc định chạy trên ThreadPool.

---

## 3. Async/Await — hiểu lầm lớn nhất của người mới

### 🧠 Ví dụ trước

```csharp
async Task<string> GetDataAsync()
{
    Console.WriteLine($"Bắt đầu — Thread ID: {Environment.CurrentManagedThreadId}");
    var result = await httpClient.GetStringAsync("https://api.example.com/data");
    Console.WriteLine($"Kết thúc — Thread ID: {Environment.CurrentManagedThreadId}");
    return result;
}
```

Chạy thử sẽ thấy: **2 dòng `Console.WriteLine` có thể in ra 2 Thread ID khác nhau!** Điều này khiến nhiều người nghĩ "async tạo ra thread mới để chạy song song". **Sai hoàn toàn.**

### 📌 Bản chất: Async không tạo thread mới — nó là cơ chế "nhường chỗ" (non-blocking I/O)

Hình dung: bạn ra quán cà phê gọi món rồi **ngồi ì chờ** tới lúc có món (= blocking/đồng bộ) — trong lúc chờ bạn không làm được gì khác. Async giống việc bạn gọi món, nhận số thứ tự, **đi làm việc khác** (ngồi đọc sách), khi có món thì được gọi tên quay lại lấy. Bạn **không cần một người khác đứng chờ thay bạn** (không tốn thêm 1 "thread" nào) — bạn chỉ đơn giản là không ngồi chờ vô ích.

Với I/O (gọi API, đọc file, truy vấn database): CPU **không cần làm gì cả** trong lúc chờ phản hồi từ mạng/đĩa — nó chỉ ngồi đợi tín hiệu phần cứng báo "xong rồi". `await` tận dụng đúng khoảng trống này: thread hiện tại được **trả lại cho ThreadPool** để đi làm việc khác, thay vì bị "khóa cứng" (blocked) ngồi chờ vô ích. Khi I/O hoàn tất, một thread **bất kỳ nào đó** rảnh trong ThreadPool (có thể là thread khác với lúc đầu) sẽ được giao tiếp tục đoạn code sau `await` — đây là lý do Thread ID đổi khác.

### 📌 Compiler biến `async` method thành gì? — State Machine

```csharp
async Task<int> Example()
{
    int a = 1;
    await Task.Delay(1000);
    int b = a + 1;
    return b;
}
```

Trình biên dịch **viết lại toàn bộ method này** thành một class ẩn implement `IAsyncStateMachine`, với các biến local (`a`, `b`) trở thành **field của class đó** (bị "hoisted" lên Heap — nhắc lại điểm đã nói ở file 01!), và method được chia thành nhiều "bước" dựa theo vị trí các `await`. Mỗi lần `await` xong, state machine nhớ "mình đang ở bước nào" để tiếp tục đúng chỗ. Đây là lý do `async` method **luôn tốn ít nhất 1 lần cấp phát Heap** (cho state machine object) — nhẹ nhưng không miễn phí, nên **không nên** đánh dấu `async` cho những method cực kỳ ngắn, chạy hàng triệu lần trong vòng lặp nóng (hot path).

### 3.1. `Task` vs `Thread` — khác nhau thế nào

| | `Thread` | `Task` |
|---|---|---|
| Là gì | 1 luồng thực thi thật sự của OS | 1 "lời hứa" (promise) về một công việc sẽ hoàn thành, có thể chạy trên ThreadPool **hoặc không tốn thread nào cả** (khi đang chờ I/O) |
| Dùng cho | CPU-bound thật sự cần chạy song song (tính toán nặng) | I/O-bound (gọi API, DB, file) — không cần giữ thread trong lúc chờ |

⚠️ `Task.Run(() => HeavyComputation())` là dùng cho **CPU-bound** (việc tính toán nặng, cần mượn 1 thread ThreadPool để chạy song song thật sự). Dùng `Task.Run` để bọc quanh code I/O-bound (vốn đã có bản `...Async`) là **thừa thãi và phản tác dụng** — vừa tốn thêm 1 thread không cần thiết, vừa không giải quyết được gì.

### 3.2. Deadlock kinh điển: `.Result` / `.Wait()` trong code đồng bộ

### 🧠 Ví dụ trước (lỗi rất hay gặp khi mới học async)

```csharp
// Trong 1 ASP.NET (Framework cũ) controller action ĐỒNG BỘ:
public ActionResult Index()
{
    var data = GetDataAsync().Result; // ⚠️ TREO ỨNG DỤNG (deadlock) trong nhiều trường hợp!
    return View(data);
}
```

### 📌 Vì sao treo

Trong môi trường có `SynchronizationContext` (như ASP.NET Framework cũ, UI app WPF/WinForms), code sau `await` mặc định cố gắng **quay lại đúng context ban đầu** để tiếp tục chạy (ví dụ quay lại UI thread). Nhưng nếu context đó đang **bị block** bởi chính `.Result` (đang ngồi chờ đồng bộ), thì code sau `await` **không bao giờ có cơ hội chạy** — vì context đang kẹt chờ chính kết quả đó → **deadlock vĩnh viễn**.

✅ **2 cách khắc phục:**

```csharp
// Cách 1 (khuyến nghị): async xuyên suốt từ đầu tới cuối — "async all the way"
public async Task<ActionResult> Index()
{
    var data = await GetDataAsync();
    return View(data);
}

// Cách 2: nếu bắt buộc phải gọi đồng bộ, dùng ConfigureAwait(false) trong thư viện
// để nói "không cần quay lại context gốc" — giảm nguy cơ deadlock (nhưng cách 1 vẫn tốt hơn)
var data = GetDataAsync().ConfigureAwait(false).GetAwaiter().GetResult();
```

> 📌 Ghi chú: ASP.NET Core (hiện đại) **không có `SynchronizationContext`** mặc định, nên lỗi deadlock kiểu này ít gặp hơn hẳn — nhưng thói quen "async all the way", không trộn lẫn `.Result`/`.Wait()` với `await`, vẫn luôn là best practice ở mọi nơi.

### 3.3. `CancellationToken` — hủy tác vụ đúng cách

```csharp
async Task ProcessAsync(CancellationToken ct)
{
    for (int i = 0; i < 1000; i++)
    {
        ct.ThrowIfCancellationRequested(); // kiểm tra, nếu bị yêu cầu hủy thì ném OperationCanceledException
        await Task.Delay(100, ct);         // nhiều API async nhận thẳng CancellationToken
    }
}

var cts = new CancellationTokenSource();
cts.CancelAfter(TimeSpan.FromSeconds(5)); // tự hủy sau 5 giây
await ProcessAsync(cts.Token);
```

✅ **Quy tắc:** mọi method async chạy lâu (vòng lặp, gọi API, xử lý nền) nên nhận `CancellationToken` làm tham số cuối cùng, để caller (ví dụ: người dùng bấm "Hủy", hoặc request HTTP bị client đóng kết nối) có cách dừng công việc giữa chừng mà không phải chờ nó chạy xong vô ích.

### 3.4. `Task.WhenAll` — chạy song song nhiều tác vụ I/O

```csharp
// ⚠️ CHẬM — chạy tuần tự, tổng thời gian = tổng 3 lần gọi
var r1 = await GetDataAsync(1);
var r2 = await GetDataAsync(2);
var r3 = await GetDataAsync(3);

// ✅ NHANH — khởi động cả 3 gần như cùng lúc, tổng thời gian ≈ thời gian của lần CHẬM NHẤT
var t1 = GetDataAsync(1);
var t2 = GetDataAsync(2);
var t3 = GetDataAsync(3);
await Task.WhenAll(t1, t2, t3);
```

⚠️ Lưu ý: cách 2 chỉ hiệu quả vì đây là **I/O-bound** (3 request mạng độc lập, không ai chờ ai). Nếu đổi thành CPU-bound tính toán nặng trên máy chỉ có ít lõi CPU, chạy "song song" quá nhiều việc cùng lúc có thể không nhanh hơn bao nhiêu, thậm chí chậm hơn do tranh chấp tài nguyên CPU.

---

## 4. Tóm tắt chương

| Khái niệm | Ghi nhớ cốt lõi |
|---|---|
| Exception | Cơ chế unwind stack — đắt về hiệu năng, không dùng cho control flow bình thường |
| throw vs throw ex | `throw` giữ stack trace gốc, `throw ex` làm mất dấu vết |
| Race condition | Nhiều thread cùng đọc/ghi 1 vùng Heap không đồng bộ → kết quả sai |
| lock/Monitor | Đảm bảo chỉ 1 thread vào khối code tại 1 thời điểm, trong CÙNG 1 process |
| Monitor.Wait/Pulse | Cho thread "ngủ" và nhả khóa, đánh thức bằng Pulse — nền tảng Producer-Consumer |
| Mutex | Giống lock nhưng hoạt động XUYÊN nhiều process (named mutex) — chậm hơn, dùng khi thật sự cần |
| Semaphore/SemaphoreSlim | Cho phép TỐI ĐA N thread vào cùng lúc (không chỉ 1) — SemaphoreSlim có WaitAsync, dùng phổ biến nhất |
| Interlocked | Thao tác atomic cực nhanh cho 1 phép toán đơn giản (tăng/giảm/so sánh-gán), không cần lock cả khối |
| Deadlock | 2+ thread chờ chéo nhau mãi mãi — tránh bằng cách lock theo thứ tự cố định |
| Async/Await | KHÔNG tạo thread mới — nhường thread lại cho ThreadPool trong lúc chờ I/O |
| async method | Bị compiler biến thành state machine, biến local bị hoisted lên Heap |
| Task vs Thread | Task cho I/O-bound (không tốn thread lúc chờ), Thread/Task.Run cho CPU-bound thật sự |
| .Result/.Wait() | Tránh trộn với async — nguy cơ deadlock, luôn "async all the way" |

➡️ Tiếp theo: [04-DotNet-Platform.md](04-DotNet-Platform.md) — CLR chi tiết hơn, Dependency Injection, và ASP.NET Core.
