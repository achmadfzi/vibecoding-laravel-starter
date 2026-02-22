---
description: SOP untuk AI dalam men-generate dokumentasi project Laravel — termasuk README.md komprehensif, PHPDoc blocks, dan dokumentasi API otomatis menggunakan Scribe.
---

# 07 — Documentation & Maintenance

> **Tujuan**: Men-generate dan memelihara dokumentasi project Laravel yang komprehensif. Termasuk file README.md, PHPDoc blocks di semua Controller/Model/Service, changelog, dan dokumentasi API otomatis menggunakan Scribe (jika project berupa REST API).

---

## Prasyarat

- Project Laravel sudah berjalan dan minimal memiliki 1 fitur CRUD
- Setidaknya sudah melalui workflow 01–06

---

## BAGIAN A: README.md

---

## Step 1 — Buat File README.md

### 1.1 — Template README.md

Buat atau replace file `README.md` di root project menggunakan template berikut. **Sesuaikan semua placeholder** `{{...}}` dengan informasi project yang sebenarnya.

````markdown
# {{APP_NAME}}

{{Deskripsi singkat aplikasi dalam 1–2 kalimat. Jelaskan apa fungsi utama dan siapa target penggunanya.}}

---

## 📋 Daftar Isi

- [Tentang Project](#tentang-project)
- [Tech Stack](#tech-stack)
- [Requirements](#requirements)
- [Instalasi Lokal](#instalasi-lokal)
- [Konfigurasi](#konfigurasi)
- [Menjalankan Aplikasi](#menjalankan-aplikasi)
- [Testing](#testing)
- [Deployment](#deployment)
- [Struktur Project](#struktur-project)
- [Kontributor](#kontributor)
- [Lisensi](#lisensi)

---

## Tentang Project

{{Deskripsi detail tentang project: latar belakang, masalah yang diselesaikan, fitur utama.}}

### Fitur Utama

- 🔐 Autentikasi (Login, Register, Logout)
- 👥 Manajemen User dengan RBAC (Role-Based Access Control)
- 📊 Dashboard Admin responsif
- {{Tambahkan fitur lain sesuai aplikasi}}

---

## Tech Stack

| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| [Laravel](https://laravel.com) | 11.x | Backend framework |
| [PHP](https://php.net) | 8.2+ | Server-side language |
| [MySQL](https://mysql.com) | 8.0+ | Database |
| [Tailwind CSS](https://tailwindcss.com) | 3.x | Styling framework |
| [Vite](https://vitejs.dev) | 5.x | Frontend bundler |
| [Laravel Breeze](https://laravel.com/docs/starter-kits) | Latest | Authentication scaffolding |
| [Spatie Permission](https://spatie.be/docs/laravel-permission) | 6.x | Role & Permission management |

---

## Requirements

Pastikan tools berikut sudah terinstall di sistem Anda:

- **PHP** >= 8.2 dengan extensions: `mbstring`, `xml`, `ctype`, `json`, `bcmath`, `openssl`, `pdo_mysql`
- **Composer** >= 2.x
- **Node.js** >= 18.x
- **npm** >= 9.x
- **MySQL** >= 8.0
- **Git** >= 2.x

---

## Instalasi Lokal

### 1. Clone repository

```bash
git clone {{REPO_URL}} {{PROJECT_FOLDER}}
cd {{PROJECT_FOLDER}}
```

### 2. Install dependencies

```bash
composer install
npm install
```

### 3. Setup environment

```bash
cp .env.example .env
php artisan key:generate
```

### 4. Konfigurasi database

Edit file `.env` dan sesuaikan konfigurasi database:

```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE={{DB_NAME}}
DB_USERNAME=root
DB_PASSWORD=
```

### 5. Buat database

```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS {{DB_NAME}} CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

### 6. Jalankan migration & seeder

```bash
php artisan migrate --seed
```

### 7. Buat storage symlink

```bash
php artisan storage:link
```

---

## Menjalankan Aplikasi

Buka **dua terminal** terpisah:

**Terminal 1 — Laravel server:**
```bash
php artisan serve
```

**Terminal 2 — Vite (frontend hot-reload):**
```bash
npm run dev
```

Buka browser: [http://localhost:8000](http://localhost:8000)

### Akun Default

| Role | Email | Password |
|------|-------|----------|
| Super Admin | `superadmin@admin.com` | `password` |
| Admin | `admin@admin.com` | `password` |
| Operator | `operator@admin.com` | `password` |

---

## Testing

```bash
# Jalankan semua test
php artisan test

# Jalankan test spesifik
php artisan test --filter=NamaTest

# Test dengan coverage
php artisan test --coverage
```

---

## Deployment

Lihat [Deployment Guide](docs/deployment.md) atau jalankan:

```bash
./deploy.sh
```

Ringkasan perintah deploy:

```bash
composer install --no-dev --optimize-autoloader
npm ci && npm run build
php artisan migrate --force
php artisan optimize
```

> ⚠️ Pastikan `APP_DEBUG=false` dan `APP_ENV=production` di server.

---

## Struktur Project

```
app/
├── DTOs/                    # Data Transfer Objects
├── Enums/                   # Enum classes
├── Helpers/                 # Global helper functions
├── Http/
│   ├── Controllers/Admin/   # Admin controllers
│   ├── Requests/Admin/      # Form request validation
│   └── Resources/           # API resources
├── Models/                  # Eloquent models
├── Repositories/            # Data access layer
│   └── Contracts/           # Repository interfaces
├── Services/                # Business logic layer
└── Traits/                  # Reusable traits

resources/views/
├── admin/                   # Admin pages
├── auth/                    # Auth pages (Breeze)
├── components/              # Blade components
├── layouts/                 # Layout templates
└── partials/                # Partial views
```

---

## Kontributor

| Nama | Role |
|------|------|
| {{Nama}} | {{Role}} |

---

## Lisensi

{{Tipe lisensi, misal: MIT License, Proprietary, dll.}}
````

### 1.2 — Instruksi untuk AI

> [!IMPORTANT]
> Saat mengisi template README:
> 1. **Tanyakan ke user** untuk: nama project, deskripsi, URL repository, nama kontributor, dan lisensi.
> 2. **Jangan meninggalkan placeholder** `{{...}}`. Semua harus diisi atau dihapus.
> 3. **Update Tech Stack** berdasarkan `composer.json` dan `package.json` yang sebenarnya.
> 4. **Update Struktur Project** berdasarkan folder yang benar-benar ada.
> 5. **Update Akun Default** berdasarkan `DatabaseSeeder.php`.
> 6. **Verifikasi** langkah instalasi bisa diikuti dari nol (fresh clone) tanpa error.

---

## BAGIAN B: PHPDoc BLOCKS

---

## Step 2 — Standar PHPDoc untuk Controller

### 2.1 — Template PHPDoc Controller

Setiap Controller **WAJIB** memiliki PHPDoc block di:
- **Class level** — deskripsi controller
- **Setiap public method** — deskripsi, parameter, return type

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Http\Requests\Admin\StoreUserRequest;
use App\Http\Requests\Admin\UpdateUserRequest;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\View\View;
use Spatie\Permission\Models\Role;

/**
 * Controller untuk manajemen user.
 *
 * Menangani operasi CRUD pada resource User, termasuk
 * assignment role menggunakan Spatie Permission.
 *
 * @see \App\Models\User
 * @see \App\Http\Requests\Admin\StoreUserRequest
 * @see \App\Http\Requests\Admin\UpdateUserRequest
 */
class UserController extends Controller
{
    /**
     * Menampilkan daftar semua user dengan pagination.
     *
     * @return \Illuminate\View\View
     */
    public function index(): View
    {
        // ...
    }

    /**
     * Menampilkan form untuk membuat user baru.
     *
     * @return \Illuminate\View\View
     */
    public function create(): View
    {
        // ...
    }

    /**
     * Menyimpan user baru ke database dan assign role.
     *
     * @param  \App\Http\Requests\Admin\StoreUserRequest  $request
     * @return \Illuminate\Http\RedirectResponse
     */
    public function store(StoreUserRequest $request): RedirectResponse
    {
        // ...
    }

    /**
     * Menampilkan detail user beserta role dan permission-nya.
     *
     * @param  \App\Models\User  $user
     * @return \Illuminate\View\View
     */
    public function show(User $user): View
    {
        // ...
    }

    /**
     * Menampilkan form untuk mengedit user.
     *
     * @param  \App\Models\User  $user
     * @return \Illuminate\View\View
     */
    public function edit(User $user): View
    {
        // ...
    }

    /**
     * Mengupdate data user dan sync role-nya.
     *
     * Password hanya diupdate jika field password diisi.
     *
     * @param  \App\Http\Requests\Admin\UpdateUserRequest  $request
     * @param  \App\Models\User  $user
     * @return \Illuminate\Http\RedirectResponse
     */
    public function update(UpdateUserRequest $request, User $user): RedirectResponse
    {
        // ...
    }

    /**
     * Menghapus user dari database.
     *
     * User tidak dapat menghapus akun sendiri (self-delete protection).
     *
     * @param  \App\Models\User  $user
     * @return \Illuminate\Http\RedirectResponse
     *
     * @throws \Illuminate\Database\Eloquent\ModelNotFoundException
     */
    public function destroy(User $user): RedirectResponse
    {
        // ...
    }
}
```

---

## Step 3 — Standar PHPDoc untuk Model

### 3.1 — Template PHPDoc Model

Setiap Model **WAJIB** memiliki PHPDoc block yang mendokumentasikan:
- Deskripsi model
- Properties (kolom database) menggunakan `@property`
- Relasi menggunakan `@property-read`
- Method kustom

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;
use Illuminate\Database\Eloquent\SoftDeletes;

/**
 * Model Example — representasi tabel `examples` di database.
 *
 * Menyimpan data contoh yang terkait dengan user. Mendukung
 * soft delete untuk keamanan data.
 *
 * @property int         $id
 * @property string      $name          Nama item
 * @property string|null $description   Deskripsi opsional
 * @property string      $status        Status: active, inactive, pending
 * @property int         $user_id       Foreign key ke tabel users
 * @property \Carbon\Carbon      $created_at
 * @property \Carbon\Carbon      $updated_at
 * @property \Carbon\Carbon|null $deleted_at
 *
 * @property-read \App\Models\User           $user       User pemilik
 * @property-read \Illuminate\Database\Eloquent\Collection|\App\Models\ExampleItem[] $items Item terkait
 *
 * @method static \Database\Factories\ExampleFactory factory(...$parameters)
 */
class Example extends Model
{
    use HasFactory, SoftDeletes;

    /**
     * Kolom yang dapat diisi secara mass-assignment.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'name',
        'description',
        'status',
        'user_id',
    ];

    /**
     * Casting kolom ke tipe data tertentu.
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'created_at' => 'datetime',
            'updated_at' => 'datetime',
            'deleted_at' => 'datetime',
        ];
    }

    /*
    |--------------------------------------------------------------------------
    | Relationships
    |--------------------------------------------------------------------------
    */

    /**
     * Relasi ke User pemilik.
     *
     * @return \Illuminate\Database\Eloquent\Relations\BelongsTo
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Relasi ke item-item terkait.
     *
     * @return \Illuminate\Database\Eloquent\Relations\HasMany
     */
    public function items(): HasMany
    {
        return $this->hasMany(ExampleItem::class);
    }

    /*
    |--------------------------------------------------------------------------
    | Scopes
    |--------------------------------------------------------------------------
    */

    /**
     * Scope: filter berdasarkan status aktif.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('status', 'active');
    }

    /*
    |--------------------------------------------------------------------------
    | Accessors & Mutators
    |--------------------------------------------------------------------------
    */

    /**
     * Accessor: dapatkan label status dalam Bahasa Indonesia.
     *
     * @return string
     */
    public function getStatusLabelAttribute(): string
    {
        return match ($this->status) {
            'active'   => 'Aktif',
            'inactive' => 'Tidak Aktif',
            'pending'  => 'Menunggu',
            default    => $this->status,
        };
    }
}
```

---

## Step 4 — Standar PHPDoc untuk Service

### 4.1 — Template PHPDoc Service

```php
<?php

namespace App\Services;

use App\Models\Example;
use App\Repositories\Contracts\ExampleRepositoryInterface;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Pagination\LengthAwarePaginator;

/**
 * Service layer untuk business logic terkait Example.
 *
 * Menghandle operasi bisnis seperti pembuatan, validasi logik khusus,
 * dan kalkulasi. Menggunakan Repository pattern untuk akses data.
 *
 * @see \App\Repositories\Contracts\ExampleRepositoryInterface
 */
class ExampleService
{
    /**
     * Buat instance ExampleService.
     *
     * @param  \App\Repositories\Contracts\ExampleRepositoryInterface  $repository
     */
    public function __construct(
        protected ExampleRepositoryInterface $repository
    ) {}

    /**
     * Ambil semua example dengan pagination.
     *
     * @param  int  $perPage  Jumlah item per halaman
     * @return \Illuminate\Pagination\LengthAwarePaginator
     */
    public function getAllPaginated(int $perPage = 15): LengthAwarePaginator
    {
        return $this->repository->paginate($perPage);
    }

    /**
     * Buat example baru setelah business logic validation.
     *
     * @param  array{name: string, description?: string, status?: string}  $data
     * @return \App\Models\Example
     *
     * @throws \InvalidArgumentException  Jika data tidak valid
     */
    public function create(array $data): Example
    {
        // Business logic sebelum create
        return $this->repository->create($data);
    }

    /**
     * Hitung total dengan pajak.
     *
     * @param  float  $amount   Jumlah dasar (harus positif)
     * @param  float  $taxRate  Tarif pajak dalam desimal (misal 0.11 untuk 11%)
     * @return float  Total setelah pajak
     *
     * @throws \InvalidArgumentException  Jika amount negatif
     */
    public function calculateTotal(float $amount, float $taxRate): float
    {
        if ($amount < 0) {
            throw new \InvalidArgumentException('Amount tidak boleh negatif.');
        }

        return $amount + ($amount * $taxRate);
    }
}
```

---

## Step 5 — Standar PHPDoc untuk FormRequest & Repository

### 5.1 — FormRequest

```php
/**
 * Form request validation untuk membuat User baru.
 *
 * Memvalidasi input: name, email, password, dan roles.
 * Authorization dicek berdasarkan permission 'user-create'.
 *
 * @see \App\Http\Controllers\Admin\UserController::store()
 */
class StoreUserRequest extends FormRequest
{
    /**
     * Cek apakah user memiliki izin untuk request ini.
     *
     * @return bool
     */
    public function authorize(): bool { /* ... */ }

    /**
     * Aturan validasi yang diterapkan.
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array|string>
     */
    public function rules(): array { /* ... */ }

    /**
     * Pesan error kustom dalam Bahasa Indonesia.
     *
     * @return array<string, string>
     */
    public function messages(): array { /* ... */ }
}
```

### 5.2 — Repository Interface

```php
/**
 * Contract untuk ExampleRepository.
 *
 * Mendefinisikan operasi data access yang harus diimplementasikan
 * oleh concrete repository class.
 *
 * @see \App\Repositories\ExampleRepository
 */
interface ExampleRepositoryInterface extends BaseRepositoryInterface
{
    /**
     * Cari example berdasarkan status.
     *
     * @param  string  $status  Status filter: 'active', 'inactive', 'pending'
     * @return \Illuminate\Database\Eloquent\Collection
     */
    public function findByStatus(string $status): Collection;
}
```

---

## Step 6 — Aturan Wajib PHPDoc

> [!WARNING]
> **AI agent WAJIB mengikuti aturan berikut saat menulis atau memodifikasi kode PHP:**

### 6.1 — File yang WAJIB punya PHPDoc

| File/Class | Level Dokumentasi | Wajib? |
|------------|-------------------|--------|
| Controller | Class + semua public method | ✅ Wajib |
| Model | Class + `@property` + relationships + scopes + accessors | ✅ Wajib |
| Service | Class + semua public method | ✅ Wajib |
| Repository | Interface + implementasi public methods | ✅ Wajib |
| FormRequest | Class + `rules()` + `messages()` | ✅ Wajib |
| Enum | Class + setiap method | ✅ Wajib |
| Trait | Class + semua public method | ✅ Wajib |
| Helper functions | Setiap function | ✅ Wajib |
| Migration | Tidak perlu | ❌ Opsional |
| Seeder | Class level saja | ⚡ Opsional |
| Config | Tidak perlu | ❌ Skip |

### 6.2 — Checklist PHPDoc per method

Setiap method WAJIB memiliki:

- [ ] **Deskripsi singkat** (1 baris pertama) — apa yang dilakukan method ini
- [ ] **Deskripsi detail** (opsional) — penjelasan tambahan jika kompleks
- [ ] **`@param`** untuk setiap parameter — tipe + nama + deskripsi
- [ ] **`@return`** — tipe return value
- [ ] **`@throws`** — exception yang mungkin dilempar (jika ada)
- [ ] **`@see`** — referensi ke class/method terkait (opsional)

### 6.3 — Format yang benar

```php
/**
 * Deskripsi singkat dalam satu baris.          ← WAJIB
 *
 * Deskripsi detail yang lebih panjang jika     ← OPSIONAL
 * diperlukan. Bisa multi-baris.
 *
 * @param  string  $name   Nama pengguna        ← WAJIB (per parameter)
 * @param  int     $age    Usia pengguna
 * @return \App\Models\User                      ← WAJIB
 *
 * @throws \InvalidArgumentException  Jika...    ← JIKA ADA
 * @see \App\Services\UserService                ← OPSIONAL
 */
```

### 6.4 — Anti-pattern (JANGAN lakukan)

```php
// ❌ Terlalu singkat / tidak berguna
/** Get user */
public function getUser() {}

// ❌ Tidak ada parameter description
/** @param $id */

// ❌ Redundant dengan method signature
/** @return View Returns a view */

// ✅ Yang benar:
/**
 * Ambil user berdasarkan ID beserta role-nya.
 *
 * @param  int  $id  ID user yang dicari
 * @return \App\Models\User|null  User instance atau null jika tidak ditemukan
 */
```

---

## BAGIAN C: DOKUMENTASI API (Scribe)

---

## Step 7 — Install Scribe (Hanya untuk REST API)

> [!NOTE]
> Step 7–10 **HANYA** diperlukan jika project memiliki REST API endpoints (`routes/api.php`). Jika project hanya web (Blade), skip ke Step 11.
>
> **Tanyakan ke user** apakah project mereka memiliki REST API sebelum menginstall Scribe.

### 7.1 — Install Scribe

// turbo
```bash
composer require --dev knuckleswtf/scribe
```

### 7.2 — Publish config

// turbo
```bash
php artisan vendor:publish --tag=scribe-config
```

Ini akan membuat file `config/scribe.php`.

### 7.3 — Konfigurasi dasar Scribe

Buka `config/scribe.php` dan edit:

```php
return [
    'title' => env('APP_NAME', 'Laravel') . ' API Documentation',

    'description' => 'Dokumentasi lengkap REST API untuk ' . env('APP_NAME', 'Laravel'),

    'base_url' => env('APP_URL', 'http://localhost:8000'),

    'routes' => [
        [
            'match' => [
                'prefixes' => ['api/*'],      // Hanya dokumentasikan route /api/*
                'domains' => ['*'],
            ],
            'include' => [],
            'exclude' => [],
        ],
    ],

    'type' => 'static',                       // Generate HTML statis
    'static' => [
        'output_path' => 'public/docs',       // Output di public/docs
    ],

    'auth' => [
        'enabled' => true,
        'default' => false,
        'in' => 'bearer',                     // Token via Authorization header
        'name' => 'Authorization',
        'use_value' => 'Bearer {ACCESS_TOKEN}',
        'placeholder' => '{ACCESS_TOKEN}',
        'extra_info' => 'Dapatkan token melalui endpoint login.',
    ],

    'example_languages' => [
        'bash',
        'javascript',
        'php',
    ],

    // Logo (opsional)
    'logo' => false,

    // Postman collection
    'postman' => [
        'enabled' => true,
        'overrides' => [],
    ],

    // OpenAPI spec
    'openapi' => [
        'enabled' => true,
        'overrides' => [],
    ],
];
```

---

## Step 8 — Menambahkan Anotasi API di Controller

### 8.1 — Template API Controller dengan anotasi Scribe

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\Admin\StoreUserRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Resources\Json\AnonymousResourceCollection;

/**
 * @group User Management
 *
 * API untuk mengelola data pengguna.
 * Memerlukan autentikasi Bearer token.
 */
class UserApiController extends Controller
{
    /**
     * Daftar semua user
     *
     * Mengambil daftar user dengan pagination.
     * Mendukung filter berdasarkan role.
     *
     * @authenticated
     *
     * @queryParam page integer Nomor halaman. Example: 1
     * @queryParam per_page integer Jumlah item per halaman. Example: 15
     * @queryParam role string Filter berdasarkan role. Example: admin
     *
     * @response 200 scenario="Success" {
     *   "data": [
     *     {
     *       "id": 1,
     *       "name": "Admin",
     *       "email": "admin@admin.com",
     *       "roles": ["admin"],
     *       "created_at": "2026-01-15T10:00:00.000000Z"
     *     }
     *   ],
     *   "meta": {
     *     "current_page": 1,
     *     "total": 25,
     *     "per_page": 15
     *   }
     * }
     *
     * @response 401 scenario="Unauthenticated" {
     *   "message": "Unauthenticated."
     * }
     */
    public function index(): AnonymousResourceCollection
    {
        $users = User::with('roles')->paginate(request('per_page', 15));

        return UserResource::collection($users);
    }

    /**
     * Buat user baru
     *
     * Membuat akun pengguna baru dan assign role.
     *
     * @authenticated
     *
     * @bodyParam name string required Nama lengkap user. Example: John Doe
     * @bodyParam email string required Email unik. Example: john@example.com
     * @bodyParam password string required Password minimal 8 karakter. Example: password123
     * @bodyParam password_confirmation string required Konfirmasi password. Example: password123
     * @bodyParam roles string[] required Daftar role. Example: ["admin"]
     *
     * @response 201 scenario="Created" {
     *   "data": {
     *     "id": 2,
     *     "name": "John Doe",
     *     "email": "john@example.com",
     *     "roles": ["admin"],
     *     "created_at": "2026-02-22T10:00:00.000000Z"
     *   }
     * }
     *
     * @response 422 scenario="Validation Error" {
     *   "message": "The email has already been taken.",
     *   "errors": {
     *     "email": ["The email has already been taken."]
     *   }
     * }
     */
    public function store(StoreUserRequest $request): JsonResponse
    {
        // ...
    }

    /**
     * Detail user
     *
     * Mengambil detail satu user berdasarkan ID.
     *
     * @authenticated
     *
     * @urlParam user integer required ID user. Example: 1
     *
     * @response 200 scenario="Found" {
     *   "data": {
     *     "id": 1,
     *     "name": "Admin",
     *     "email": "admin@admin.com",
     *     "roles": ["admin"],
     *     "permissions": ["user-list", "user-create"],
     *     "created_at": "2026-01-15T10:00:00.000000Z"
     *   }
     * }
     *
     * @response 404 scenario="Not Found" {
     *   "message": "User not found."
     * }
     */
    public function show(User $user): UserResource
    {
        // ...
    }

    /**
     * Hapus user
     *
     * Menghapus user dari database. User tidak dapat menghapus dirinya sendiri.
     *
     * @authenticated
     *
     * @urlParam user integer required ID user yang akan dihapus. Example: 2
     *
     * @response 200 scenario="Deleted" {
     *   "message": "User berhasil dihapus."
     * }
     *
     * @response 403 scenario="Self Delete" {
     *   "message": "Anda tidak dapat menghapus akun sendiri."
     * }
     */
    public function destroy(User $user): JsonResponse
    {
        // ...
    }
}
```

### 8.2 — Anotasi Scribe yang sering digunakan

| Anotasi | Fungsi | Contoh |
|---------|--------|--------|
| `@group` | Kelompokkan endpoint | `@group User Management` |
| `@authenticated` | Endpoint butuh auth | `@authenticated` |
| `@unauthenticated` | Endpoint publik | `@unauthenticated` |
| `@queryParam` | Parameter di URL query | `@queryParam page integer Example: 1` |
| `@urlParam` | Parameter di URL path | `@urlParam id integer required Example: 1` |
| `@bodyParam` | Parameter di request body | `@bodyParam name string required Example: John` |
| `@response` | Contoh response | `@response 200 scenario="OK" {...}` |
| `@responseFile` | Response dari file JSON | `@responseFile responses/users.json` |
| `@header` | Custom header required | `@header X-Custom-Header string required` |

---

## Step 9 — Generate Dokumentasi API

### 9.1 — Generate docs

// turbo
```bash
php artisan scribe:generate
```

### 9.2 — Akses dokumentasi

Buka di browser:
- **HTML docs**: `http://localhost:8000/docs`
- **Postman collection**: `public/docs/collection.json`
- **OpenAPI spec**: `public/docs/openapi.yaml`

### 9.3 — Re-generate setelah perubahan

Setiap kali endpoint API berubah:

```bash
php artisan scribe:generate
```

> [!TIP]
> Tambahkan perintah ini ke deploy script agar dokumentasi selalu up-to-date:
> ```bash
> php artisan scribe:generate --no-interaction 2>/dev/null || true
> ```

---

## Step 10 — Exclude Docs dari Git (Opsional)

Jika tidak ingin commit file dokumentasi hasil generate, tambahkan ke `.gitignore`:

```
public/docs/
.scribe/
```

---

## BAGIAN D: CHANGELOG & MAINTENANCE

---

## Step 11 — Buat File CHANGELOG.md

### 11.1 — Template CHANGELOG.md

Buat file `CHANGELOG.md` di root project:

```markdown
# Changelog

Semua perubahan penting pada project ini didokumentasikan di file ini.

Format berdasarkan [Keep a Changelog](https://keepachangelog.com/id-ID/1.1.0/),
dan project ini mengikuti [Semantic Versioning](https://semver.org/lang/id/).

## [Unreleased]

### Ditambahkan
- Fitur yang sedang dikembangkan

---

## [1.0.0] - YYYY-MM-DD

### Ditambahkan
- Setup project Laravel dengan Clean Architecture
- Autentikasi menggunakan Laravel Breeze (login, register, logout)
- Dashboard admin dengan layout responsif (sidebar + navbar)
- RBAC menggunakan Spatie Permission (super-admin, admin, operator)
- CRUD manajemen user dengan assignment role
- Testing framework (Pest) dengan template Feature & Unit test
- Deploy script dan checklist production
- Dokumentasi project (README, PHPDoc, CHANGELOG)

### Keamanan
- Middleware permission pada route admin
- FormRequest authorization
- Blade directives untuk kontrol UI berdasarkan permission
- Self-delete protection pada user management
```

### 11.2 — Aturan update CHANGELOG

> [!IMPORTANT]
> AI agent **WAJIB** menambahkan entry ke CHANGELOG setiap kali:
> - Menambah fitur baru → `### Ditambahkan`
> - Mengubah fitur yang sudah ada → `### Diubah`
> - Menghapus fitur → `### Dihapus`
> - Memperbaiki bug → `### Diperbaiki`
> - Perbaikan keamanan → `### Keamanan`
> - Perubahan yang tidak backward-compatible → `### Berubah (Breaking)`

---

## Step 12 — Maintenance Commands

### 12.1 — Perintah maintenance rutin

| Kapan | Perintah | Fungsi |
|-------|----------|--------|
| Setelah deploy | `php artisan optimize` | Cache config/route/view |
| Debug production | `php artisan config:clear` | Clear config cache |
| Update permission | `php artisan permission:cache-reset` | Reset permission cache |
| Update dependencies | `composer update` | Update PHP packages |
| Check outdated | `composer outdated` | Cek package yang perlu diupdate |
| Clear log | Hapus isi `storage/logs/*.log` | Bersihkan log files |
| Check routes | `php artisan route:list` | Daftar semua route |
| Check schedule | `php artisan schedule:list` | Daftar scheduled tasks |

---

## Checklist Verifikasi Akhir

### README.md
- [ ] File `README.md` ada di root project
- [ ] Semua placeholder `{{...}}` sudah diganti
- [ ] Deskripsi project jelas dan informatif
- [ ] Tech stack sesuai dengan package yang terinstall
- [ ] Langkah instalasi bisa diikuti dari nol tanpa error
- [ ] Akun default tercantum dan sesuai dengan seeder

### PHPDoc
- [ ] Semua Controller memiliki PHPDoc (class + methods)
- [ ] Semua Model memiliki PHPDoc (class + `@property` + relations)
- [ ] Semua Service memiliki PHPDoc (class + methods)
- [ ] Semua FormRequest memiliki PHPDoc
- [ ] Semua Repository interface memiliki PHPDoc
- [ ] Setiap method memiliki `@param`, `@return`, dan `@throws` (jika ada)
- [ ] Tidak ada placeholder atau `TODO` di PHPDoc

### API Documentation (jika ada API)
- [ ] Scribe terinstall dan terkonfigurasi
- [ ] Semua endpoint memiliki anotasi (`@group`, `@bodyParam`, `@response`)
- [ ] `php artisan scribe:generate` berjalan tanpa error
- [ ] Dokumentasi bisa diakses di `/docs`
- [ ] Postman collection tersedia
- [ ] OpenAPI spec tersedia

### Changelog
- [ ] `CHANGELOG.md` ada di root project
- [ ] Entry versi 1.0.0 mencakup semua fitur yang sudah dibuat

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **PHPDoc WAJIB ditulis saat membuat kode**, bukan setelahnya. Jangan pernah membuat Controller/Model/Service tanpa PHPDoc.
> 2. **README harus selalu up-to-date.** Setiap kali menambahkan fitur baru, update section yang relevan di README (Tech Stack, Fitur, Struktur, dll).
> 3. **CHANGELOG harus diupdate setiap merilis fitur.** Ini bukan opsional — ini adalah catatan historis penting.
> 4. **Scribe hanya untuk project API.** Tanyakan ke user sebelum menginstall. Jangan install jika tidak ada `routes/api.php`.
> 5. **Jangan meninggalkan placeholder.** Semua `{{...}}`, `TODO`, dan `FIXME` harus diselesaikan sebelum commit.
> 6. **PHPDoc bukan cerita** — gunakan bahasa teknis yang ringkas. Satu baris untuk deskripsi singkat sudah cukup untuk method sederhana.
> 7. **Verifikasi README** dengan cara: bayangkan developer baru clone project ini. Apakah mereka bisa menjalankan aplikasi hanya dengan membaca README? Jika tidak, README belum lengkap.
