# 🛠️ Hướng Dẫn Setup Môi Trường Laravel (Windows)

> **Trạng thái máy hiện tại:**
> - ✅ PHP 8.5.9 (đã có sẵn)
> - ✅ Composer 2.10.2 (đã có sẵn)
> - ❌ Thiếu PHP extensions: `zip`, `fileinfo`, `gd`, `pdo_mysql`, `pdo_sqlite`
> - ❌ Chưa có MySQL

---

## Bước 1: Cài Laragon

**Laragon** là phần mềm tạo môi trường lập trình trên Windows — tương tự XAMPP nhưng **gọn hơn, đẹp hơn, và sinh ra cho Laravel**. Cài xong là có đủ PHP (bật sẵn mọi extension), MySQL, Apache — không cần sửa gì thủ công.

### Tải và cài đặt

1. Vào trang: **https://laragon.org/download/**
2. Tải bản **Laragon Full** (khoảng ~150MB)
3. Cài đặt bình thường (Next → Next → Install)
4. Mở Laragon lên → nhấn **"Start All"** → Apache + MySQL tự chạy

### Sau khi cài xong, bạn sẽ có

| Thứ | Mô tả |
|-----|-------|
| **PHP** (bật đủ extensions) | Không cần sửa php.ini |
| **MySQL** | Database, chạy trên port 3306 |
| **Apache** | Web server |
| **Terminal** | Mở bằng cách click "Terminal" trong Laragon |
| **HeidiSQL** | Tool xem database có giao diện (giống MySQL Workbench nhưng nhẹ hơn) |

### Kiểm tra

Mở **Terminal của Laragon** (click nút "Terminal" trong cửa sổ Laragon), chạy:

```powershell
php -v
composer -V
php -m | findstr "zip fileinfo gd pdo_mysql pdo_sqlite"
```

**Kết quả mong đợi** — thấy đủ 5 extensions:
```
fileinfo
gd
pdo_mysql
pdo_sqlite
zip
```

> ⚠️ **Quan trọng:** Dùng **Terminal của Laragon** để Laragon dùng PHP của nó (đã bật đủ extensions). Nếu mở PowerShell bình thường, nó có thể dùng PHP cũ của bạn (thiếu extensions).

---

## Bước 2: Tạo project Laravel mới

Có **2 cách**, chọn cách nào cũng được:

### Cách 1 — Qua giao diện Laragon (đơn giản nhất)

1. Trong Laragon → Click **Menu** (nút ☰ ở góc) → **Quick app** → **Laravel**
2. Đặt tên: `my-first-app`
3. Laragon tự chạy `composer create-project` + tạo database + cấu hình virtual host luôn
4. Xong → mở trình duyệt vào **http://my-first-app.test** là thấy trang Welcome

### Cách 2 — Bằng lệnh trong Terminal (nên dùng cách này để hiểu)

Mở Terminal Laragon, chạy:

```powershell
cd D:\Users\Phuc\Desktop\Learn-Laravel
composer create-project laravel/laravel my-first-app
```

Đợi 1-3 phút cho Composer tải xong. Sau đó:

```powershell
cd my-first-app
php artisan serve
```

Mở trình duyệt → vào **http://localhost:8000** → thấy trang Welcome Laravel 🎉

> 💡 **`php artisan serve` hoạt động giống `npm run dev` bên Vite:**
> - Khởi động dev server
> - Tự reload khi bạn sửa code
> - Nhấn **Ctrl+C** để tắt

---

## Bước 3: Mở project trong VS Code

```powershell
code D:\Users\Phuc\Desktop\Learn-Laravel\my-first-app
```

Hoặc trong VS Code → File → Open Folder → chọn `my-first-app`.

### Extensions VS Code nên cài

| Extension | Tác dụng |
|-----------|---------|
| **PHP Intelephense** | Autocomplete + go to definition cho PHP |
| **Laravel Blade Snippets** | Highlight + snippet cho file `.blade.php` |
| **Laravel Extra Intellisense** | Autocomplete route names, view names |

---

## Bước 4: Thử sửa code đầu tiên

Mở file `routes/web.php` trong project, bạn sẽ thấy:

```php
Route::get('/', function () {
    return view('welcome');
});
```

**Thử thêm 1 route mới bên dưới:**

```php
Route::get('/hello', function () {
    return 'Xin chào! Đây là trang đầu tiên mình tự tạo 🎉';
});
```

Lưu file → mở trình duyệt → vào **http://localhost:8000/hello** → thấy dòng chữ.

**Chúc mừng! Bạn vừa viết dòng Laravel đầu tiên.** 🚀

---

## ✅ Checklist — Tự check sau khi làm xong

- [x] Đã cài Laragon, nhấn "Start All" thấy Apache + MySQL xanh
- [x] Mở Terminal Laragon, chạy `php -m` thấy đủ extensions
- [x] Chạy `composer create-project` tạo thành công `my-first-app`
- [x] Chạy `php artisan serve` → mở `http://localhost:8000` thấy trang Welcome
- [x] Thêm route `/hello` → mở trên trình duyệt thấy text tự viết
- [x] Mở project trong VS Code

---

## 📝 Ghi chú

- **Laragon vs PHP cũ:** Bạn đã có PHP cài sẵn ở `C:\Program Files\php-8.5.9-nts-Win32-vs17-x64\`, Laragon sẽ dùng bản PHP riêng của nó. Không xung đột — chỉ cần nhớ dùng Terminal của Laragon khi làm việc với Laravel.
- **Khi nào cần MySQL:** Ở Bài 3 (Migration + Model) trở đi. Laragon đã cài sẵn MySQL, chỉ cần "Start All" là có.
- **Sau khi xong bước này:** Quay lại `context.md`, bắt đầu **Bài 1** trong lộ trình.
