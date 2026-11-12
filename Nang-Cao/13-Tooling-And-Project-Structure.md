# 13 — Tooling & Project Structure: Từ "biết code" tới "làm việc trong 1 dự án thật"

> Toàn bộ 12 file trước dạy ngôn ngữ và kiến trúc. File này dạy thứ khác hẳn: **mở 1 repo .NET thật ra, bạn thấy gì, và vì sao nó được tổ chức như vậy** — khoảng trống khiến nhiều người "giỏi lý thuyết" vẫn bỡ ngỡ ngày đầu đi làm.

---

## 1. `.csproj` — "giấy khai sinh" của 1 project

### 🧠 Ví dụ trước

```bash
dotnet new console -n MyApp
```

Lệnh này tạo ra 1 thư mục `MyApp/` chứa `Program.cs` và **1 file tên `MyApp.csproj`** — file nhỏ xíu nhưng mọi công cụ (IDE, `dotnet build`, CI/CD) đều đọc nó đầu tiên để biết "project này là gì, build thế nào".

### 📌 Giải thích từng dòng

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

</Project>
```

| Thẻ | Ý nghĩa |
|---|---|
| `Sdk="Microsoft.NET.Sdk"` | Dùng bộ quy tắc build chuẩn của .NET (khác `Microsoft.NET.Sdk.Web` cho ASP.NET Core project) |
| `OutputType` | `Exe` = chạy được trực tiếp (console app); `Library` = chỉ tạo ra `.dll` để project khác dùng, không tự chạy được |
| `TargetFramework` | Project build nhắm tới phiên bản .NET nào (`net9.0`, `net8.0`...) — quyết định API nào dùng được |
| `ImplicitUsings` | Tự động thêm sẵn 1 số `using` phổ biến (`System`, `System.Linq`...) vào mọi file, đỡ phải gõ tay |
| `Nullable` | Bật Nullable Reference Types (file 02) |

📌 Đây gọi là **SDK-style project** (từ .NET Core trở đi) — cực kỳ gọn so với định dạng `.csproj` cũ của .NET Framework (vài trăm dòng XML liệt kê **từng file** `.cs` một). SDK-style tự động **gom hết mọi file `.cs` trong thư mục** vào project — bạn thêm file mới, không cần sửa `.csproj` thủ công như trước đây.

### 📌 `.sln` — gom nhiều `.csproj` lại thành 1 "giải pháp"

```bash
dotnet new sln -n MyApp
dotnet sln add src/MyApp.Api/MyApp.Api.csproj
dotnet sln add src/MyApp.Domain/MyApp.Domain.csproj
```

File `.sln` (solution) không chứa code — nó chỉ là **danh sách các `.csproj`** thuộc cùng 1 hệ thống, để IDE (Visual Studio/Rider) mở **tất cả project liên quan cùng lúc** thay vì mở từng cái riêng lẻ. 1 repo thường có **1 `.sln`**, chứa **nhiều `.csproj`**.

---

## 2. Cấu trúc nhiều project — Clean Architecture nhìn từ ổ đĩa

### 🧠 Ví dụ trước

File 06 đã dạy Dependency Rule: Domain không được biết gì về Infrastructure. Nhưng nếu tất cả class nằm chung 1 project, **không có gì ngăn** 1 dev vô tình gõ `using Infrastructure;` ngay trong file ở tầng Domain — lỗi này chỉ phát hiện được lúc code review, nếu có ai để ý.

### 📌 Giải pháp: tách tầng thành project riêng, để compiler tự chặn

```
MyApp.sln
src/
├── MyApp.Domain/              (Library — không reference project nào khác)
│   └── MyApp.Domain.csproj
├── MyApp.Application/         (Library — chỉ reference Domain)
│   └── MyApp.Application.csproj
├── MyApp.Infrastructure/      (Library — reference Application + các package EF/SMTP...)
│   └── MyApp.Infrastructure.csproj
└── MyApp.Api/                 (Exe/Web — reference Application + Infrastructure, là "nhạc trưởng" ráp mọi thứ lại)
    └── MyApp.Api.csproj
```

```xml
<!-- MyApp.Application.csproj -->
<ItemGroup>
  <ProjectReference Include="..\MyApp.Domain\MyApp.Domain.csproj" />
</ItemGroup>
```

✅ **Đây là điểm mấu chốt**: `ProjectReference` quyết định project nào **được phép** `using` project nào. Nếu `MyApp.Domain.csproj` **không có** `ProjectReference` nào trỏ tới `MyApp.Infrastructure`, thì dù dev có cố tình gõ `using MyApp.Infrastructure;` trong code của Domain, **compiler sẽ báo lỗi ngay lập tức** ("không tìm thấy namespace") — Dependency Rule giờ không còn là "quy ước anh em tự giác tuân theo" nữa, mà là **ràng buộc vật lý compiler ép buộc**. Đây chính là lý do các dự án áp dụng Clean Architecture nghiêm túc luôn tách nhiều project thay vì gộp chung 1 project với thư mục `Domain/`, `Application/`... — thư mục không ngăn được ai `using` bậy, project reference thì có.

⚠️ **Hệ quả cần nhớ:** `MyApp.Domain.csproj` **không được phép** cài package `Microsoft.EntityFrameworkCore` hay bất kỳ package nào thuộc về "chi tiết kỹ thuật" — nếu bạn thấy mình cần cài EF Core vào project Domain để code chạy được, đó là dấu hiệu rõ ràng đang vi phạm Dependency Rule, cần xem lại thiết kế (thường là do đặt entity EF-specific — ví dụ navigation property kiểu `DbSet` — thẳng vào Domain).

---

## 3. NuGet — quản lý thư viện bên thứ 3

### 🧠 Ví dụ trước

```bash
dotnet add package Newtonsoft.Json --version 13.0.3
```

Chạy xong, mở `.csproj` lên sẽ thấy tự động thêm:

```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

### 📌 Giải thích

**NuGet** là kho package (thư viện đóng gói sẵn) chính thức của .NET — giống `npm` của JavaScript hay `pip` của Python. `dotnet add package` tải package về (cache ở máy, nằm trong `~/.nuget/packages`), rồi ghi 1 dòng `PackageReference` vào `.csproj` để đánh dấu "project này cần package này, đúng version này".

| Lệnh | Vai trò |
|---|---|
| `dotnet add package <Tên>` | Thêm 1 package mới vào project hiện tại |
| `dotnet remove package <Tên>` | Gỡ package khỏi project |
| `dotnet list package` | Xem project đang dùng những package nào |
| `dotnet restore` | Tải về tất cả package mà `.csproj` yêu cầu nhưng máy chưa có cache — tự động chạy ngầm mỗi lần `build`/`run` |

### ⚠️ Version conflict — khi 2 project cần 2 version khác nhau của cùng 1 package

Nếu `MyApp.Api` cần `Serilog 3.1.0` còn `MyApp.Infrastructure` cần `Serilog 3.0.0`, chỉ **1 version duy nhất** được nạp thật sự lúc chạy (gọi là *version unification* — không thể có 2 bản của cùng 1 assembly cùng lúc). Thuật toán NuGet chọn version cụ thể nào khá tinh vi (direct reference thường được ưu tiên hơn transitive — "nearest wins" — và giữa các version request khác nhau thường sẽ ngả về version cao hơn để thỏa mãn mọi ràng buộc), nên **đừng đoán** version nào sẽ thắng — dùng `dotnet list package --include-transitive` để xem chính xác version nào thật sự được chọn, nếu thấy hành vi lạ sau khi thêm package.

✅ **Central Package Management** (tính năng mới hơn): thay vì ghi version lặp lại trong từng `.csproj`, tạo 1 file `Directory.Packages.props` ở gốc repo, khai báo version **1 lần duy nhất** cho mọi project dùng chung — tránh tình trạng 5 project tham chiếu cùng 1 package nhưng mỗi cái 1 version khác nhau do quên đồng bộ tay.

---

## 4. `dotnet` CLI — các lệnh dùng hằng ngày ngoài `new`/`run`

```bash
dotnet restore          # tải về mọi package còn thiếu (thường không cần gọi tay, build/run tự làm)
dotnet build            # biên dịch ra .dll, KHÔNG chạy
dotnet run               # build (nếu cần) RỒI chạy luôn
dotnet test              # chạy toàn bộ unit test trong project test (file 09)
dotnet publish -c Release -o ./publish   # đóng gói bản production, tối ưu, sẵn sàng deploy
dotnet tool install -g dotnet-ef          # cài 1 "global tool" — ở đây là CLI của EF Core (file 05, dùng cho migrations)
```

📌 Phân biệt nhanh: `build` chỉ biên dịch (dùng khi muốn kiểm tra code có lỗi compile không, hoặc trong bước CI trước khi test); `run` build xong chạy ngay (dùng lúc code/debug hằng ngày); `publish` tạo ra bản **đóng gói để triển khai thật** — khác `build` ở chỗ thường tối ưu hóa hơn (`-c Release` thay vì `Debug` mặc định), có thể kèm trim/self-contained tùy cấu hình.

---

## 5. Môi trường chạy (Environment) — vì sao code không được hardcode connection string

### 🧠 Ví dụ trước

Project ASP.NET Core mới tạo luôn có sẵn:

```
appsettings.json                 ← cấu hình MẶC ĐỊNH, dùng chung mọi môi trường
appsettings.Development.json     ← GHI ĐÈ lên trên, chỉ áp dụng khi chạy ở máy dev
```

```json
// appsettings.json
{ "ConnectionStrings": { "Default": "Server=prod-db;Database=MyApp;..." } }

// appsettings.Development.json
{ "ConnectionStrings": { "Default": "Server=localhost;Database=MyApp_Dev;..." } }
```

### 📌 Giải thích

ASP.NET Core đọc biến môi trường `ASPNETCORE_ENVIRONMENT` để quyết định **file config nào được nạp chồng lên sau cùng**: luôn đọc `appsettings.json` trước, rồi đọc tiếp `appsettings.{Environment}.json` **đè lên** những key trùng tên. Nhờ vậy, code **không bao giờ cần biết** mình đang chạy ở đâu — chỉ cần inject `IConfiguration`/Options Pattern (file 04) như bình thường, giá trị đúng môi trường tự động có mặt.

⚠️ **Hiểu lầm phổ biến:** `ASPNETCORE_ENVIRONMENT` **không có giá trị mặc định là `Development`** ở bất kỳ đâu — nếu biến này hoàn toàn không được set, ASP.NET Core luôn coi là **`Production`** (an toàn hơn khi lỡ quên set). Lý do bạn thấy app chạy ở `Development` lúc code tại máy dev hoàn toàn là vì **`launchSettings.json`** (mục ngay bên dưới) **chủ động set** biến môi trường này mỗi khi bấm F5/`dotnet run` qua 1 profile cụ thể — không phải hành vi mặc định của `dotnet run`. Chạy `dotnet run --no-launch-profile`, hoặc chạy trực tiếp file `.dll` đã publish mà quên set biến này, app sẽ tự coi là `Production` dù đang ở máy dev — đây là nguồn gốc của không ít bug "tự nhiên đọc nhầm config".

⚠️ **Quy tắc bắt buộc:** `appsettings.json` (file commit vào Git) **không bao giờ** được chứa secret thật (connection string production, API key, mật khẩu) — vì file này nằm trong source control, ai clone repo cũng đọc được. Secret thật nên để trong **biến môi trường** của server thật, hoặc dịch vụ quản lý secret chuyên dụng (Azure Key Vault, AWS Secrets Manager...), hoặc dùng `dotnet user-secrets` (chỉ lưu trên máy dev cá nhân, không commit) khi phát triển local.

### 📌 `launchSettings.json` — cấu hình riêng cho lúc debug ở máy dev

```json
// Properties/launchSettings.json
{
  "profiles": {
    "https": {
      "applicationUrl": "https://localhost:7001;http://localhost:5001",
      "environmentVariables": { "ASPNETCORE_ENVIRONMENT": "Development" }
    }
  }
}
```

📌 File này **chỉ ảnh hưởng lúc bấm F5/`dotnet run` ở máy dev** (quyết định port chạy, environment variable set sẵn...) — **không hề được dùng** khi deploy production (lúc đó server thật tự set `ASPNETCORE_ENVIRONMENT=Production` theo cách riêng, ví dụ qua biến môi trường của container/VM). Đây là file hay bị commit nhầm chứa thông tin cá nhân (port riêng, số điện thoại test...) — nên kiểm tra trước khi commit dù không chứa secret nhạy cảm.

---

## 6. Tóm tắt chương

| Khái niệm | Ghi nhớ cốt lõi |
|---|---|
| `.csproj` | "Giấy khai sinh" của 1 project — SDK-style tự gom file `.cs`, không cần liệt kê tay như .NET Framework cũ |
| `.sln` | Danh sách các `.csproj` thuộc cùng hệ thống, để IDE mở tất cả cùng lúc |
| ProjectReference | Quyết định project nào được `using` project nào — biến Dependency Rule (file 06) thành ràng buộc compiler, không chỉ là quy ước |
| NuGet / PackageReference | Quản lý thư viện bên thứ 3, version unification khi conflict, Central Package Management để đồng bộ version |
| dotnet build/run/publish | Chỉ biên dịch / biên dịch rồi chạy / đóng gói bản deploy tối ưu |
| appsettings theo môi trường | File sau đè lên file trước theo `ASPNETCORE_ENVIRONMENT` — code không cần biết đang chạy ở đâu |
| Secret | Không bao giờ commit secret thật vào `appsettings.json` — dùng biến môi trường/Key Vault/user-secrets |

➡️ Quay lại [00-README.md](00-README.md) để xem toàn bộ lộ trình.
