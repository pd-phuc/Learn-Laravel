# Bài 2: Laravel Basics — Nền Tảng Cốt Lõi

Bài này trình bày toàn bộ kiến thức nền tảng của Laravel mà bạn cần nắm **trước khi** bắt tay vào code bài tập. Mỗi phần đều có đối chiếu với source code thực tế trong `ref/mst-checkscam`.

---

## Mục lục

1. [Cấu trúc thư mục Laravel](#1-cấu-trúc-thư-mục-laravel)
2. [Vòng đời một Request](#2-vòng-đời-một-request)
3. [Routing](#3-routing)
4. [Controller](#4-controller)
5. [Blade Template](#5-blade-template)
6. [Request và Response](#6-request-và-response)

---

## 1. Cấu trúc thư mục Laravel

Khi bạn chạy `composer create-project`, Laravel tạo ra rất nhiều folder. Không cần nhớ hết — chỉ cần hiểu rõ **6 folder quan trọng nhất** dưới đây:

```
my-first-app/
├── app/                    ← CODE PHP CHÍNH
│   ├── Http/
│   │   ├── Controllers/    ← Nơi đặt Controller (xử lý logic)
│   │   └── Middleware/     ← Bộ lọc request (kiểm tra auth, chặn spam...)
│   ├── Models/             ← Nơi đặt Model (tương tác database)
│   └── Helpers/            ← (tùy chọn) Hàm tiện ích tự viết
│
├── routes/                 ← ĐỊNH NGHĨA ĐƯỜNG DẪN (URL)
│   └── web.php             ← Tất cả route cho web nằm ở đây
│
├── resources/              ← GIAO DIỆN
│   └── views/              ← File Blade template (.blade.php)
│       ├── layouts/        ← Layout dùng chung (header, footer)
│       └── home.blade.php  ← Trang cụ thể
│
├── database/               ← MỌI THỨ LIÊN QUAN DATABASE
│   ├── migrations/         ← File tạo/sửa bảng
│   ├── seeders/            ← File tạo dữ liệu mẫu
│   └── factories/          ← File sinh dữ liệu giả (fake)
│
├── config/                 ← CÀI ĐẶT HỆ THỐNG
│   ├── app.php             ← Cấu hình chung (tên app, timezone...)
│   ├── database.php        ← Cấu hình kết nối DB
│   └── ...
│
├── public/                 ← THƯ MỤC WEB ROOT
│   └── index.php           ← Entry point — mọi request đều đi qua đây
│
├── .env                    ← BIẾN MÔI TRƯỜNG (DB password, API key...)
├── artisan                 ← CLI tool của Laravel
├── composer.json           ← Danh sách dependencies PHP
└── package.json            ← Danh sách dependencies JS (nếu dùng Vite)
```

**Đối chiếu với `mst-checkscam`:**

Mở thư mục `ref/mst-checkscam` bạn sẽ thấy cấu trúc giống hệt. Dev đã tổ chức code theo đúng convention:

| Thư mục | Trong mst-checkscam chứa gì |
|---------|----------------------------|
| `app/Http/Controllers/` | `HomeController`, `SearchController`, `ReportController`... + folder `Admin/` riêng |
| `app/Models/` | `Report`, `Comment`, `User`, `Insurance`... (10 model) |
| `app/Helpers/` | `StringHelper`, `FileHelper`, `StatsHelper` — các hàm tiện ích dùng lại nhiều nơi |
| `routes/web.php` | ~187 dòng khai báo tất cả URL của web |
| `resources/views/` | `home.blade.php`, folder `admin/`, `scammer/`, `reports/`... |
| `database/migrations/` | 16 file migration tạo các bảng |
| `config/` | Ngoài config mặc định còn thêm `newfeed.php`, `seotools.php` |

### Quy tắc quan trọng

- **Không sửa file trong `vendor/`** — đây là code của các package (giống `node_modules`), Composer quản lý tự động.
- **Không commit `.env`** — file này chứa mật khẩu, API key. Mỗi môi trường (dev, staging, production) có `.env` riêng.
- **`public/` là thư mục duy nhất** mà web server expose ra ngoài. User không bao giờ truy cập trực tiếp vào `app/` hay `config/`.

---

## 2. Vòng đời một Request

Khi người dùng gõ URL trên trình duyệt, Laravel xử lý theo thứ tự sau:

```
Trình duyệt gửi request (VD: GET /hello)
    │
    ▼
public/index.php              ← Mọi request đều đi qua đây
    │
    ▼
Middleware                    ← Bộ lọc (kiểm tra auth, maintenance mode...)
    │
    ▼
routes/web.php                ← Tìm route khớp với URL
    │
    ▼
Controller@method             ← Chạy hàm xử lý logic
    │
    ├── Model (đọc/ghi DB)
    │
    ▼
View (.blade.php)             ← Render HTML
    │
    ▼
Response → Trình duyệt       ← Trả HTML về cho người dùng
```

**Ví dụ thực tế trong `mst-checkscam`:**

Khi người dùng vào `https://checkscam.vn/` (trang chủ):

1. `public/index.php` nhận request
2. Middleware `CheckMaintenanceMode` kiểm tra web có đang bảo trì không
3. `routes/web.php` tìm thấy: `Route::get('/', [HomeController::class, 'index'])`
4. `HomeController@index` chạy: query thống kê, lấy báo cáo mới nhất, lấy lịch sử tìm kiếm
5. Controller gọi `return view('home', compact(...))` → render file `resources/views/home.blade.php`
6. HTML trả về cho trình duyệt

Hiểu flow này là hiểu cách Laravel hoạt động. Mọi tính năng đều đi theo đúng luồng này.

---

## 3. Routing

### Route là gì?

Route = bảng ánh xạ giữa **URL** và **code xử lý**. Khi user truy cập một URL, Laravel dò trong file `routes/web.php` để tìm route khớp, rồi chạy code tương ứng.

### Cú pháp cơ bản

```php
// Cấu trúc: Route::{method}('{url}', {xử_lý});

// Dạng 1: Closure (hàm trực tiếp) — dùng cho trang đơn giản
Route::get('/hello', function () {
    return 'Hello World';
});

// Dạng 2: Trỏ đến Controller — dùng cho mọi trang phức tạp
Route::get('/', [HomeController::class, 'index']);
//              ↑ class Controller      ↑ tên method sẽ gọi
```

### HTTP Methods

```php
Route::get('/page', ...);       // Lấy dữ liệu, hiển thị trang
Route::post('/submit', ...);    // Gửi dữ liệu (form submit)
Route::put('/update/{id}', ...);    // Cập nhật toàn bộ
Route::patch('/toggle/{id}', ...);  // Cập nhật một phần
Route::delete('/delete/{id}', ...); // Xóa
```

Trong thực tế, bạn sẽ dùng `get` và `post` nhiều nhất. `put`, `patch`, `delete` dùng trong admin panel.

### Route Parameters

Khi URL có phần động (VD: mỗi bài phốt có slug khác nhau):

```php
// {slug} là parameter — Laravel tự trích xuất giá trị từ URL
Route::get('/{slug}', [ReportController::class, 'show'])->name('scammer.show');

// Trong Controller, nhận parameter qua tham số hàm:
public function show(string $slug)
{
    // $slug = giá trị từ URL, VD: "nguyen-van-a-x8hf3k2l"
    $report = Report::where('slug', $slug)->firstOrFail();
    return view('scammer.index', compact('report'));
}
```

**Trong `mst-checkscam`:** Route cuối cùng trong `web.php` (dòng 186) là `Route::get('/{slug}', ...)` — đây là catch-all route, bắt mọi URL không khớp route nào ở trên, coi nó như slug của bài phốt.

### Named Routes

Đặt tên cho route để không phải hardcode URL:

```php
// Đặt tên:
Route::get('/', [HomeController::class, 'index'])->name('home');
Route::get('/to-cao-lua-dao', ...)->name('reports');

// Dùng tên thay vì URL cứng:
return redirect()->route('home');                  // thay vì redirect('/');
$url = route('scammer.show', ['slug' => 'abc']);   // sinh URL: /abc
```

Lợi ích: nếu sau này đổi URL (VD: `/to-cao-lua-dao` thành `/bao-cao`), chỉ cần sửa 1 chỗ trong route, không phải tìm sửa khắp nơi.

### Route Groups

Khi nhiều route có chung prefix, middleware, hoặc name:

```php
// Trong mst-checkscam — tất cả route admin:
Route::name('admin.')->prefix('admin')->group(function () {
    // Các route trong group tự động có:
    // - URL bắt đầu bằng /admin/...
    // - Tên bắt đầu bằng admin....

    Route::middleware('auth')->group(function () {
        // Các route trong đây YÊU CẦU ĐĂNG NHẬP
        Route::get('/', [DashboardController::class, 'index'])->name('dashboard');
        // → URL thực: /admin
        // → Tên thực: admin.dashboard
        // → Phải đăng nhập mới vào được

        Route::get('/reports', [AdminReportController::class, 'index'])->name('reports.index');
        // → URL thực: /admin/reports
        // → Tên thực: admin.reports.index
    });

    // Route login nằm NGOÀI middleware auth (vì chưa đăng nhập thì làm sao check auth)
    Route::get('/login', [AuthController::class, 'showFormLogin'])->name('auth.login');
});
```

**Phân tích:** Dev dùng `prefix('admin')` để gom mọi URL admin vào `/admin/...`, `middleware('auth')` để bảo vệ, `name('admin.')` để đặt tên nhất quán. Đây là pattern chuẩn mà bạn sẽ gặp trong mọi dự án Laravel.

---

## 4. Controller

### Controller là gì?

Controller là nơi đặt **logic xử lý**. Thay vì nhét code vào file route (sẽ rất dài và lộn xộn), mỗi nhóm tính năng có một Controller riêng.

### Tạo Controller

```bash
php artisan make:controller HomeController
```

Lệnh này tạo file `app/Http/Controllers/HomeController.php` với nội dung cơ bản.

### Cấu trúc một Controller

```php
<?php

namespace App\Http\Controllers;           // Khai báo namespace

use App\Models\Report;                    // Import Model cần dùng
use Illuminate\Http\Request;              // Import class Request

class HomeController extends Controller   // Kế thừa Controller base
{
    // Mỗi method = 1 trang hoặc 1 hành động
    public function index(Request $request)
    {
        // 1. Xử lý logic (query DB, tính toán...)
        $stats = [
            'total_reports' => Report::where('status', 'approved')->count(),
        ];
        $latestReports = Report::where('status', 'approved')
            ->latest()
            ->limit(5)
            ->get();

        // 2. Trả về View kèm dữ liệu
        return view('home', compact('stats', 'latestReports'));
        //         ↑ tên view    ↑ biến truyền sang view
    }
}
```

### Kết nối Route với Controller

```php
// routes/web.php
use App\Http\Controllers\HomeController;

Route::get('/', [HomeController::class, 'index'])->name('home');
//                ↑ tên class            ↑ tên method
```

Khi user truy cập `/`, Laravel gọi `HomeController` → chạy method `index()`.

### Truyền dữ liệu sang View

Có 3 cách, kết quả giống nhau:

```php
// Cách 1: compact() — phổ biến nhất, mst-checkscam dùng cách này
return view('home', compact('stats', 'latestReports'));
// compact() tự tạo array ['stats' => $stats, 'latestReports' => $latestReports]

// Cách 2: Mảng thủ công
return view('home', [
    'stats' => $stats,
    'latestReports' => $latestReports,
]);

// Cách 3: with()
return view('home')->with('stats', $stats)->with('latestReports', $latestReports);
```

### Tổ chức Controller trong `mst-checkscam`

```
app/Http/Controllers/
├── Controller.php              ← Base class (không chứa logic)
├── HomeController.php          ← Trang chủ
├── SearchController.php        ← Tra cứu scammer
├── ReportController.php        ← Gửi + xem tố cáo
├── CommentController.php       ← Bình luận
├── PostController.php          ← Bài viết
├── InsuranceController.php     ← Bảo hiểm
├── NewfeedController.php       ← Khu mua bán
├── SocialiteController.php     ← Google OAuth
├── SitemapController.php       ← Tạo sitemap
└── Admin/                      ← Folder riêng cho admin
    ├── AuthController.php
    ├── DashboardController.php
    ├── AdminReportController.php
    ├── AdminCommentController.php
    ├── AdminInsuranceController.php
    ├── AdminPostController.php
    ├── AdminBannerController.php
    ├── AdminUserController.php
    ├── AdminSettingController.php
    └── AdminSearchAnalyticsController.php
```

Quy tắc đặt tên:
- Tên Controller = **danh từ + "Controller"** (VD: `ReportController`, `CommentController`)
- Mỗi Controller quản lý 1 resource (1 đối tượng nghiệp vụ)
- Controller admin đặt trong subfolder `Admin/` với prefix `Admin` trong tên

---

## 5. Blade Template

### Blade là gì?

Blade là template engine của Laravel — nó cho phép bạn viết PHP trong file HTML một cách gọn gàng, thay vì dùng `<?php echo ... ?>` truyền thống.

File Blade có đuôi `.blade.php` và nằm trong `resources/views/`.

### Hiển thị biến

```blade
{{-- Cú pháp Blade --}}
{{ $variable }}

{{-- Tương đương PHP thuần: --}}
<?php echo htmlspecialchars($variable); ?>
```

`{{ }}` tự động **escape HTML** để chống XSS (tấn công chèn script). Nếu muốn hiển thị HTML thô (đã tin tưởng an toàn):

```blade
{!! $htmlContent !!}     {{-- KHÔNG escape, cẩn thận khi dùng --}}
```

### Cấu trúc điều kiện

```blade
@if($report->status == 'approved')
    <span class="text-green">Đã duyệt</span>
@elseif($report->status == 'pending')
    <span class="text-yellow">Chờ duyệt</span>
@else
    <span class="text-red">Từ chối</span>
@endif
```

### Vòng lặp

```blade
{{-- Duyệt mảng/collection --}}
@foreach($latestReports as $report)
    <div class="card">
        <h3>{{ $report->target_name }}</h3>
        <p>{{ $report->description }}</p>
    </div>
@endforeach

{{-- Hiển thị message khi mảng rỗng --}}
@forelse($reports as $report)
    <div>{{ $report->target_name }}</div>
@empty
    <p>Không có dữ liệu.</p>
@endforelse
```

### Layout: Kế thừa template

Đây là cách **tái sử dụng** cấu trúc trang (header, footer, sidebar...) mà không phải copy-paste.

**Bước 1 — Tạo layout chính** (`resources/views/layouts/app.blade.php`):

```blade
<!DOCTYPE html>
<html lang="vi">
<head>
    <title>@yield('title', 'Trang chủ')</title>
</head>
<body>
    <header>
        {{-- Navbar chung cho mọi trang --}}
        <nav>...</nav>
    </header>

    <main>
        @yield('content')       {{-- Chỗ trống: trang con sẽ điền nội dung vào đây --}}
    </main>

    <footer>
        {{-- Footer chung --}}
    </footer>
</body>
</html>
```

**Bước 2 — Trang con kế thừa layout:**

```blade
{{-- resources/views/home.blade.php --}}
@extends('layouts.app')           {{-- Kế thừa layout --}}

@section('title', 'Trang chủ')    {{-- Điền vào @yield('title') --}}

@section('content')                {{-- Điền vào @yield('content') --}}
    <h1>Chào mừng đến CheckScam</h1>
    <p>Tổng báo cáo: {{ $stats['total_reports'] }}</p>

    @foreach($latestReports as $report)
        <div>{{ $report->target_name }}</div>
    @endforeach
@endsection
```

Kết quả: HTML cuối cùng trả về cho trình duyệt sẽ là layout + nội dung trang con ghép lại.

**Trong `mst-checkscam`:**
- Layout chính: `resources/views/layouts/app.blade.php` — dùng `@yield('content')` và Blade Components (`<x-head />`, `<x-header />`, `<x-footer />`)
- Admin có layout riêng: `resources/views/admin/layouts/`
- Mỗi trang con dùng `@extends('layouts.app')` + `@section('content')`

### Blade Components

Ngoài `@extends/@yield`, Laravel còn hỗ trợ Components — cách tái sử dụng UI hiện đại hơn:

```blade
{{-- Tạo component: resources/views/components/header.blade.php --}}
<header>
    <nav>
        <a href="{{ route('home') }}">Trang chủ</a>
        <a href="{{ route('reports') }}">Tố cáo</a>
    </nav>
</header>

{{-- Sử dụng: --}}
<x-header />       {{-- Laravel tự tìm file components/header.blade.php --}}
```

`mst-checkscam` dùng Components cho `<x-head />`, `<x-header />`, `<x-footer />` — những phần xuất hiện trên mọi trang.

### Include (nhúng partial)

Dùng khi muốn tách 1 phần HTML nhỏ ra file riêng:

```blade
{{-- resources/views/partials/report-card.blade.php --}}
<div class="card">
    <h3>{{ $report->target_name }}</h3>
    <p>{{ $report->description }}</p>
</div>

{{-- Nhúng vào trang chính: --}}
@foreach($reports as $report)
    @include('partials.report-card', ['report' => $report])
@endforeach
```

---

## 6. Request và Response

### Nhận dữ liệu từ Request

Khi user gửi form hoặc truy cập URL có query string, Controller nhận dữ liệu qua object `$request`:

```php
public function index(Request $request)
{
    // Từ query string: /search?q=0987654321
    $query = $request->query('q', '');       // '' là giá trị mặc định nếu không có 'q'

    // Từ form POST:
    $name = $request->input('reporter_name');

    // Kiểm tra có parameter không:
    if ($request->filled('status')) {         // filled = có và không rỗng
        // ...
    }

    // Lấy IP người dùng:
    $ip = $request->ip();

    // Kiểm tra request AJAX:
    if ($request->ajax()) {
        return response()->json(['data' => ...]);
    }
}
```

**Trong `mst-checkscam`:**
- `SearchController` dùng `$request->query('q')` để lấy từ khóa tìm kiếm
- `ReportController` dùng `$request->ip()` để chống spam (giới hạn 3 lần/IP/ngày)
- `HomeController` dùng `$request->ajax()` để phân biệt request thường và AJAX (load more)
- `AdminReportController` dùng `$request->filled('status')` để lọc theo trạng thái

### Các kiểu Response

```php
// 1. Trả về View (phổ biến nhất)
return view('home', compact('data'));

// 2. Trả text/HTML thẳng
return 'Hello World';

// 3. Trả JSON (cho API hoặc AJAX)
return response()->json([
    'message' => 'Thành công',
    'data' => $report,
]);

// 4. Redirect (chuyển hướng sang trang khác)
return redirect()->route('home');
return redirect()->back();                              // quay lại trang trước
return redirect()->route('home')->with('success', 'Đã lưu!');   // redirect kèm flash message

// 5. Lỗi 404
abort(404);
// hoặc dùng firstOrFail() — tự trả 404 nếu không tìm thấy:
$report = Report::where('slug', $slug)->firstOrFail();
```

### Flash Session — Thông báo 1 lần

Flash session là dữ liệu tồn tại **chỉ trong 1 request tiếp theo**, dùng để hiển thị thông báo sau khi submit form:

```php
// Trong Controller:
return redirect()->route('home')->with('success', 'Báo cáo đã được gửi!');

// Trong Blade view:
@if(session('success'))
    <div class="alert-success">{{ session('success') }}</div>
@endif
```

`mst-checkscam` dùng pattern này ở khắp nơi: sau khi duyệt báo cáo, xóa comment, login thành công...

---

## Tổng kết: Bức tranh toàn cảnh

Tất cả kiến thức trên kết nối với nhau theo flow:

```
User truy cập URL
    │
    ▼
[routes/web.php]    ← ROUTING: tìm route khớp, có thể qua middleware
    │
    ▼
[Controller]        ← Nhận Request, xử lý logic, gọi Model
    │
    ▼
[Blade View]        ← Nhận dữ liệu từ Controller, render HTML
    │                  Dùng @extends để kế thừa layout
    │                  Dùng {{ }} để hiển thị biến
    │                  Dùng @foreach, @if để logic hiển thị
    ▼
Response            ← HTML/JSON/Redirect trả về cho trình duyệt
```

Bài tiếp theo sẽ đi vào **thực hành**: tạo Controller, viết Blade view, dựng lại mini version trang chủ CheckScam. Nhưng trước hết, hãy đọc kỹ bài này và thử đối chiếu từng phần với code trong `ref/mst-checkscam` để thấy lý thuyết áp dụng thực tế ra sao.

### Bài tập tự đọc code (không cần code, chỉ đọc hiểu)

1. Mở `ref/mst-checkscam/routes/web.php` — đọc và trả lời: có bao nhiêu route `GET`? Bao nhiêu `POST`? Route nào có middleware?
2. Mở `ref/mst-checkscam/app/Http/Controllers/HomeController.php` — chỉ ra đâu là phần query dữ liệu, đâu là phần trả về view.
3. Mở `ref/mst-checkscam/resources/views/layouts/app.blade.php` — tìm `@yield` và giải thích nó chờ nhận gì từ trang con.
4. Mở `ref/mst-checkscam/app/Http/Controllers/CommentController.php` — tìm chỗ dùng `$request->ip()`, `$request->ajax()`, `$request->validate()` và giải thích mỗi chỗ làm gì.
