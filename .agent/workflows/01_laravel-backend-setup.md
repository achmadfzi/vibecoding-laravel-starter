---
description: SOP untuk menginisialisasi project Laravel baru dengan Clean Architecture, koneksi database MySQL, dan konfigurasi standar production-ready.
---

# 01 — Laravel Backend Setup

> **Tujuan**: Menginisialisasi project Laravel baru dari nol hingga siap digunakan sebagai backend API/web app dengan arsitektur bersih, koneksi database MySQL, dan konfigurasi standar production-ready.

---

## Prasyarat

Pastikan tools berikut sudah terinstall di sistem sebelum memulai:

| Tool       | Minimum Version | Cara Cek              |
|------------|----------------|-----------------------|
| PHP        | 8.2+           | `php -v`              |
| Composer   | 2.x            | `composer --version`  |
| MySQL      | 8.0+           | `mysql --version`     |
| Node.js    | 18+ (opsional) | `node -v`             |
| Git        | 2.x            | `git --version`       |

> [!IMPORTANT]
> Jika salah satu prasyarat belum terpenuhi, **hentikan proses** dan minta user untuk menginstall terlebih dahulu.

---

## Step 1 — Instalasi Laravel via Composer

// turbo
```bash
composer create-project laravel/laravel . --prefer-dist
```

> [!NOTE]
> Perintah ini menginstall Laravel ke **direktori saat ini** (`.`). Pastikan direktori kosong sebelum menjalankan.
> Jika direktori tidak kosong, tanyakan ke user apakah ingin membersihkan terlebih dahulu atau menggunakan direktori baru.

**Verifikasi instalasi berhasil:**

// turbo
```bash
php artisan --version
```

Pastikan output menunjukkan versi Laravel (misal: `Laravel Framework 11.x.x` atau `12.x.x`).

---

## Step 2 — Setup File `.env` dan Koneksi Database MySQL

### 2.1 — Salin file environment

// turbo
```bash
copy .env.example .env
```

### 2.2 — Generate application key

// turbo
```bash
php artisan key:generate
```

### 2.3 — Konfigurasi koneksi database

Buka file `.env` dan ubah bagian database menjadi:

```dotenv
APP_NAME="NamaAplikasi"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=root
DB_PASSWORD=
```

> [!IMPORTANT]
> - Tanyakan ke user untuk nilai `APP_NAME`, `DB_DATABASE`, `DB_USERNAME`, dan `DB_PASSWORD`.
> - Jika user tidak memberikan, gunakan default: `APP_NAME=LaravelApp`, `DB_DATABASE=laravel_db`, `DB_USERNAME=root`, `DB_PASSWORD=` (kosong).
> - Pastikan database sudah dibuat di MySQL sebelum melanjutkan ke step migrasi.

### 2.4 — Buat database (jika belum ada)

// turbo
```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS nama_database CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

> Ganti `nama_database` dengan nilai `DB_DATABASE` dari `.env`. Jika MySQL memerlukan password, gunakan flag `-p` dan masukkan password.

### 2.5 — Tes koneksi database

// turbo
```bash
php artisan db:show
```

Jika koneksi berhasil, output akan menampilkan informasi database. Jika gagal, periksa kembali kredensial di `.env`.

---

## Step 3 — Konfigurasi `config/app.php`

Buka file `config/app.php` dan ubah nilai berikut:

```php
'timezone' => 'Asia/Jakarta',

'locale' => 'id',

'fallback_locale' => 'en',

'faker_locale' => 'id_ID',
```

> [!NOTE]
> - `timezone`: Sesuaikan dengan zona waktu user. Default `Asia/Jakarta` (WIB). Opsi lain: `Asia/Makassar` (WITA), `Asia/Jayapura` (WIT).
> - `locale`: Bahasa utama aplikasi. Gunakan `id` untuk Bahasa Indonesia.
> - `faker_locale`: Agar data dummy dari Factory/Seeder menggunakan format Indonesia (nama, alamat, dll).
> - Tanyakan ke user jika timezone atau locale perlu disesuaikan.

---

## Step 4 — Setup Clean Architecture (Folder Structure)

### 4.1 — Buat struktur folder tambahan

// turbo
```bash
mkdir -p app/Services
mkdir -p app/Repositories/Contracts
mkdir -p app/DTOs
mkdir -p app/Enums
mkdir -p app/Traits
mkdir -p app/Helpers
mkdir -p app/Http/Requests
mkdir -p app/Http/Resources
mkdir -p app/Exceptions
```

> [!NOTE]
> Pada Windows, gunakan `mkdir` tanpa `-p` per folder, atau gunakan PowerShell:
> ```powershell
> New-Item -ItemType Directory -Force -Path app/Services, app/Repositories/Contracts, app/DTOs, app/Enums, app/Traits, app/Helpers, app/Http/Requests, app/Http/Resources, app/Exceptions
> ```

### 4.2 — Penjelasan setiap folder

| Folder                       | Fungsi                                                                                          |
|-----------------------------|-------------------------------------------------------------------------------------------------|
| `app/Services/`            | Business logic layer. Setiap fitur utama memiliki satu Service class.                          |
| `app/Repositories/`        | Data access layer. Abstraksi query database, memisahkan Eloquent dari business logic.          |
| `app/Repositories/Contracts/` | Interface untuk Repository pattern. Memungkinkan dependency injection & testability.          |
| `app/DTOs/`                | Data Transfer Objects. Untuk passing data antar layer secara type-safe.                        |
| `app/Enums/`               | Enum classes untuk konstanta (status, role, tipe, dsb).                                        |
| `app/Traits/`              | Reusable traits untuk Model atau Controller (misal: `HasUuid`, `Filterable`, `ApiResponse`).   |
| `app/Helpers/`             | Helper functions/class global (misal: formatting, string manipulation).                        |
| `app/Http/Requests/`       | Form Request validation classes (sudah bawaan Laravel, pastikan digunakan).                    |
| `app/Http/Resources/`      | API Resource classes untuk transformasi response JSON (sudah bawaan Laravel).                  |
| `app/Exceptions/`          | Custom exception classes (sudah bawaan Laravel, tambahkan exception spesifik di sini).         |

### 4.3 — Buat base Repository Interface

Buat file `app/Repositories/Contracts/BaseRepositoryInterface.php`:

```php
<?php

namespace App\Repositories\Contracts;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Pagination\LengthAwarePaginator;

interface BaseRepositoryInterface
{
    public function all(): Collection;
    public function find(int|string $id): ?Model;
    public function create(array $data): Model;
    public function update(int|string $id, array $data): Model;
    public function delete(int|string $id): bool;
    public function paginate(int $perPage = 15): LengthAwarePaginator;
}
```

### 4.4 — Buat base Repository Implementation

Buat file `app/Repositories/BaseRepository.php`:

```php
<?php

namespace App\Repositories;

use App\Repositories\Contracts\BaseRepositoryInterface;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Pagination\LengthAwarePaginator;

abstract class BaseRepository implements BaseRepositoryInterface
{
    public function __construct(
        protected Model $model
    ) {}

    public function all(): Collection
    {
        return $this->model->all();
    }

    public function find(int|string $id): ?Model
    {
        return $this->model->find($id);
    }

    public function create(array $data): Model
    {
        return $this->model->create($data);
    }

    public function update(int|string $id, array $data): Model
    {
        $record = $this->model->findOrFail($id);
        $record->update($data);
        return $record->fresh();
    }

    public function delete(int|string $id): bool
    {
        return $this->model->findOrFail($id)->delete();
    }

    public function paginate(int $perPage = 15): LengthAwarePaginator
    {
        return $this->model->paginate($perPage);
    }
}
```

### 4.5 — Buat API Response Trait

Buat file `app/Traits/ApiResponse.php`:

```php
<?php

namespace App\Traits;

use Illuminate\Http\JsonResponse;

trait ApiResponse
{
    protected function successResponse(mixed $data = null, string $message = 'Success', int $code = 200): JsonResponse
    {
        return response()->json([
            'success' => true,
            'message' => $message,
            'data'    => $data,
        ], $code);
    }

    protected function errorResponse(string $message = 'Error', int $code = 400, mixed $errors = null): JsonResponse
    {
        $response = [
            'success' => false,
            'message' => $message,
        ];

        if ($errors !== null) {
            $response['errors'] = $errors;
        }

        return response()->json($response, $code);
    }

    protected function createdResponse(mixed $data = null, string $message = 'Created successfully'): JsonResponse
    {
        return $this->successResponse($data, $message, 201);
    }

    protected function noContentResponse(): JsonResponse
    {
        return response()->json(null, 204);
    }
}
```

### 4.6 — Buat base Enum untuk Status (contoh)

Buat file `app/Enums/StatusEnum.php`:

```php
<?php

namespace App\Enums;

enum StatusEnum: string
{
    case ACTIVE   = 'active';
    case INACTIVE = 'inactive';
    case PENDING  = 'pending';

    public function label(): string
    {
        return match ($this) {
            self::ACTIVE   => 'Aktif',
            self::INACTIVE => 'Tidak Aktif',
            self::PENDING  => 'Menunggu',
        };
    }
}
```

### 4.7 — Buat Helper file (opsional, jika dibutuhkan)

Buat file `app/Helpers/helpers.php`:

```php
<?php

if (!function_exists('format_rupiah')) {
    /**
     * Format angka ke format Rupiah.
     */
    function format_rupiah(int|float $amount, bool $withPrefix = true): string
    {
        $formatted = number_format($amount, 0, ',', '.');
        return $withPrefix ? "Rp {$formatted}" : $formatted;
    }
}

if (!function_exists('generate_code')) {
    /**
     * Generate kode unik dengan prefix.
     */
    function generate_code(string $prefix = 'TRX'): string
    {
        return $prefix . '-' . strtoupper(uniqid());
    }
}
```

Lalu daftarkan di `composer.json` agar autoload:

```json
"autoload": {
    "psr-4": {
        "App\\": "app/",
        "Database\\Factories\\": "database/factories/",
        "Database\\Seeders\\": "database/seeders/"
    },
    "files": [
        "app/Helpers/helpers.php"
    ]
},
```

Setelah mengedit `composer.json`, jalankan:

// turbo
```bash
composer dump-autoload
```

---

## Step 5 — Daftarkan Repository Binding di Service Provider

### 5.1 — Buat RepositoryServiceProvider

// turbo
```bash
php artisan make:provider RepositoryServiceProvider
```

### 5.2 — Edit RepositoryServiceProvider

Buka file `app/Providers/RepositoryServiceProvider.php` dan isi:

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class RepositoryServiceProvider extends ServiceProvider
{
    /**
     * Daftarkan binding Repository Interface -> Implementation di sini.
     *
     * Contoh:
     * $this->app->bind(
     *     \App\Repositories\Contracts\UserRepositoryInterface::class,
     *     \App\Repositories\UserRepository::class
     * );
     */
    public function register(): void
    {
        // Tambahkan binding repository di sini sesuai kebutuhan fitur
    }

    public function boot(): void
    {
        //
    }
}
```

### 5.3 — Daftarkan provider di `bootstrap/providers.php`

Tambahkan `RepositoryServiceProvider` ke array providers:

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\RepositoryServiceProvider::class,
];
```

> [!NOTE]
> Pada Laravel 11+, providers didaftarkan di `bootstrap/providers.php`.
> Pada Laravel 10 atau sebelumnya, daftarkan di array `'providers'` dalam `config/app.php`.

---

## Step 6 — Konfigurasi Tambahan (Opsional tapi Direkomendasikan)

### 6.1 — Setup CORS (jika digunakan sebagai API backend)

Buka `config/cors.php` dan sesuaikan:

```php
'paths' => ['api/*'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:3000')],
'allowed_methods' => ['*'],
'allowed_headers' => ['*'],
'supports_credentials' => true,
```

Tambahkan di `.env`:

```dotenv
FRONTEND_URL=http://localhost:3000
```

### 6.2 — Setup API Rate Limiting

Buka `app/Providers/AppServiceProvider.php` dan tambahkan di method `boot()`:

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Http\Request;

public function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

### 6.3 — Setup Logging Configuration

Di `.env`, pastikan konfigurasi log sudah sesuai:

```dotenv
LOG_CHANNEL=daily
LOG_LEVEL=debug
```

---

## Step 7 — Jalankan Initial Migration

// turbo
```bash
php artisan migrate
```

**Verifikasi migrasi berhasil:**

// turbo
```bash
php artisan migrate:status
```

Pastikan semua migration berstatus `Ran`.

> [!CAUTION]
> Jika migrasi gagal, periksa hal berikut:
> 1. Database sudah dibuat dan bisa diakses.
> 2. Kredensial `DB_*` di `.env` sudah benar.
> 3. MySQL service sedang berjalan.
> 4. Jalankan `php artisan config:clear` lalu coba lagi.

---

## Step 8 — Verifikasi Akhir

### 8.1 — Jalankan development server

// turbo
```bash
php artisan serve
```

Buka `http://localhost:8000` di browser dan pastikan halaman Laravel welcome ditampilkan.

### 8.2 — Checklist verifikasi

Sebelum melanjutkan ke development fitur, pastikan semua item berikut terpenuhi:

- [ ] Laravel terinstall dan `php artisan --version` menampilkan versi
- [ ] File `.env` terkonfigurasi dengan benar
- [ ] `APP_KEY` sudah di-generate
- [ ] Koneksi database berhasil (`php artisan db:show`)
- [ ] `config/app.php` — timezone, locale, faker_locale sudah diubah
- [ ] Struktur folder Clean Architecture sudah dibuat
- [ ] `BaseRepositoryInterface` dan `BaseRepository` sudah dibuat
- [ ] `ApiResponse` trait sudah dibuat
- [ ] `RepositoryServiceProvider` sudah dibuat dan didaftarkan
- [ ] `composer dump-autoload` sudah dijalankan (jika ada Helper)
- [ ] Initial migration berhasil (`php artisan migrate:status` semua `Ran`)
- [ ] Development server berjalan tanpa error

---

## Ringkasan Struktur Folder Akhir

```
app/
├── DTOs/                          # Data Transfer Objects
├── Enums/
│   └── StatusEnum.php             # Contoh Enum
├── Exceptions/                    # Custom Exceptions
├── Helpers/
│   └── helpers.php                # Global helper functions
├── Http/
│   ├── Controllers/               # Controllers (bawaan Laravel)
│   ├── Middleware/                 # Middleware (bawaan Laravel)
│   ├── Requests/                  # Form Request Validation
│   └── Resources/                 # API Resources
├── Models/                        # Eloquent Models (bawaan Laravel)
├── Providers/
│   ├── AppServiceProvider.php     # App Service Provider
│   └── RepositoryServiceProvider.php  # Repository bindings
├── Repositories/
│   ├── Contracts/
│   │   └── BaseRepositoryInterface.php
│   └── BaseRepository.php
├── Services/                      # Business Logic Layer
└── Traits/
    └── ApiResponse.php            # Standar API response format
```

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **Selalu tanyakan ke user** sebelum menggunakan nilai default untuk `APP_NAME`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, dan `timezone`.
> 2. **Jangan skip verifikasi**. Setiap step memiliki verifikasi — pastikan dijalankan.
> 3. **Jika ada error**, jangan lanjutkan ke step berikutnya. Debug terlebih dahulu.
> 4. **Folder `Services/` dan `Repositories/`** bersifat template. Isinya akan bertambah seiring development fitur.
> 5. **Helper file bersifat opsional**. Hanya buat jika user membutuhkannya.
> 6. **Gunakan `// turbo` annotation** — command yang sudah ditandai `// turbo` aman untuk di-auto-run.
