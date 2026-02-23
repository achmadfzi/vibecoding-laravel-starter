---
description: Aturan dokumentasi kode Laravel yang wajib dipatuhi AI — standar PHPDoc blocks, inline comments, README.md, .env.example, dan format JSON API documentation.
---

# Laravel Documentation Rules

This document outlines the documentation standards, inline commenting rules, and strict PHPDoc guidelines to maintain clean, readable, and professional code in this Laravel project.

---

## 1. PHPDoc Blocks — Wajib di Setiap Class dan Method

### 1.1 — Aturan Umum

AI **WAJIB** menyertakan PHPDoc blocks pada **setiap** Class dan Method yang dibuat. PHPDoc harus ditulis **bersamaan** dengan kode, **bukan** ditambahkan setelahnya.

Setiap PHPDoc block **WAJIB** mengandung:
- **`@param`** — untuk setiap parameter method (tipe + nama + deskripsi singkat).
- **`@return`** — tipe return value, termasuk `void`.
- **`@throws`** — setiap exception yang mungkin dilempar oleh method (jika ada).

```php
// ✅ Benar — PHPDoc lengkap
/**
 * Buat user baru dan assign role yang dipilih.
 *
 * Password akan di-hash sebelum disimpan. Setelah user dibuat,
 * event UserCreated akan di-dispatch untuk proses lanjutan.
 *
 * @param  array{name: string, email: string, password: string, roles: string[]}  $data
 * @return \App\Models\User
 *
 * @throws \Illuminate\Database\QueryException  Jika email duplikat
 * @throws \InvalidArgumentException            Jika role tidak valid
 */
public function create(array $data): User
{
    // ...
}

// ❌ Salah — Tanpa PHPDoc
public function create(array $data): User
{
    // ...
}

// ❌ Salah — PHPDoc tidak lengkap (missing @param dan @throws)
/**
 * Buat user baru.
 *
 * @return \App\Models\User
 */
public function create(array $data): User
{
    // ...
}
```

### 1.2 — File yang WAJIB Punya PHPDoc

| File/Class | Level Dokumentasi | Wajib? |
|------------|-------------------|--------|
| **Controller** | Class + semua public method | 🔴 Wajib |
| **Model** | Class + `@property` + relationships + scopes + accessors | 🔴 Wajib |
| **Service** | Class + semua public method | 🔴 Wajib |
| **Repository** | Interface + implementasi public methods | 🔴 Wajib |
| **FormRequest** | Class + `rules()` + `messages()` | 🔴 Wajib |
| **Enum** | Class + setiap method | 🔴 Wajib |
| **Trait** | Class + semua public method | 🔴 Wajib |
| **Helper functions** | Setiap function | 🔴 Wajib |
| **DTO** | Class + `@property` | 🔴 Wajib |
| **Policy** | Class + semua method | 🟡 Penting |
| **Middleware** | Class + `handle()` | 🟡 Penting |
| **Seeder** | Class level saja | ⚡ Opsional |
| **Migration** | Tidak perlu | ❌ Skip |
| **Config** | Tidak perlu | ❌ Skip |

### 1.3 — PHPDoc Class Level

Setiap class **WAJIB** memiliki PHPDoc di atas deklarasi class:

```php
// ✅ Benar — Class-level PHPDoc
/**
 * Service layer untuk business logic terkait pembayaran.
 *
 * Menghandle pemrosesan pembayaran, kalkulasi pajak,
 * dan integrasi dengan payment gateway eksternal.
 *
 * @see \App\Repositories\Contracts\PaymentRepositoryInterface
 * @see \App\Models\Payment
 */
class PaymentService
{
    // ...
}

// ❌ Salah — Tanpa class-level PHPDoc
class PaymentService
{
    // ...
}
```

**Aturan class-level PHPDoc:**
- Baris pertama: **deskripsi singkat** (apa tanggung jawab class ini).
- Baris berikutnya (opsional): **detail tambahan** jika diperlukan.
- Gunakan `@see` untuk merujuk ke class terkait (repository, model, interface).
- Untuk **Model**: sertakan `@property` untuk setiap kolom dan `@property-read` untuk relasi.

### 1.4 — PHPDoc Model — Format Detail

```php
/**
 * Model Payment — representasi tabel `payments` di database.
 *
 * Menyimpan data transaksi pembayaran yang terkait dengan order.
 *
 * @property int              $id
 * @property string           $invoice_number    Nomor invoice unik
 * @property float            $amount            Jumlah pembayaran
 * @property string           $status            Status: pending, paid, failed, refunded
 * @property string|null      $payment_method    Metode pembayaran (bca, mandiri, gopay, dll)
 * @property int              $order_id          Foreign key ke tabel orders
 * @property int              $user_id           Foreign key ke tabel users
 * @property \Carbon\Carbon   $paid_at           Waktu pembayaran dikonfirmasi
 * @property \Carbon\Carbon   $created_at
 * @property \Carbon\Carbon   $updated_at
 *
 * @property-read \App\Models\Order  $order   Order terkait
 * @property-read \App\Models\User   $user    User yang membayar
 */
class Payment extends Model
{
    // ...
}
```

### 1.5 — PHPDoc Constructor dengan Dependency Injection

```php
/**
 * Buat instance PaymentService.
 *
 * @param  \App\Repositories\Contracts\PaymentRepositoryInterface  $repository
 * @param  \App\Services\TaxCalculatorService                      $taxService
 */
public function __construct(
    protected PaymentRepositoryInterface $repository,
    protected TaxCalculatorService $taxService,
) {}
```

### 1.6 — Format @param, @return, @throws yang Benar

```php
/**
 * Proses pembayaran untuk order tertentu.
 *
 * Method ini akan: (1) validasi amount, (2) buat record payment,
 * (3) kirim ke payment gateway, (4) update status order.
 *
 * @param  \App\Models\Order  $order        Order yang akan dibayar
 * @param  float              $amount       Jumlah pembayaran (harus positif)
 * @param  string             $method       Metode pembayaran: 'bca', 'mandiri', 'gopay'
 * @param  array{callback_url?: string, metadata?: array}  $options  Opsi tambahan
 * @return \App\Models\Payment  Payment record yang baru dibuat
 *
 * @throws \App\Exceptions\InsufficientBalanceException  Jika saldo tidak cukup
 * @throws \App\Exceptions\PaymentGatewayException       Jika gateway error
 * @throws \InvalidArgumentException                     Jika amount negatif
 */
public function processPayment(
    Order $order,
    float $amount,
    string $method,
    array $options = []
): Payment {
    // ...
}
```

**Aturan format:**
- `@param`: gunakan **FQCN** (Fully Qualified Class Name) dengan `\` prefix untuk tipe class.
- `@param`: **align** tipe, nama, dan deskripsi agar mudah dibaca.
- `@return`: sebutkan tipe spesifik, bukan hanya `mixed`.
- `@throws`: satu baris per exception, sertakan kondisi kapan exception dilempar.
- Gunakan `array{key: type}` notation untuk mendeskripsikan struktur array.

### 1.7 — Anti-Pattern PHPDoc (JANGAN Lakukan)

```php
// ❌ PHPDoc yang hanya mengulang nama method (redundan, tidak berguna)
/**
 * Get user.
 */
public function getUser(): User {}

// ❌ @param tanpa deskripsi
/**
 * @param $id
 * @param $name
 */
public function update($id, $name) {}

// ❌ Terlalu verbose / deskripsi novel
/**
 * This method is responsible for the complex and intricate process of
 * retrieving a singular user entity from the database persistence layer
 * by utilizing the unique identifier...
 */
public function findById(int $id): ?User {}

// ❌ Menggunakan tipe generik
/**
 * @param  mixed  $data
 * @return mixed
 */
public function process($data) {}

// ✅ Yang benar — singkat, spesifik, informatif
/**
 * Ambil user berdasarkan ID beserta role dan permission-nya.
 *
 * @param  int  $id  ID user yang dicari
 * @return \App\Models\User|null  User instance, atau null jika tidak ditemukan
 */
public function findById(int $id): ?User {}
```

---

## 2. Inline Comments — Hanya untuk Logika Bisnis yang Rumit

### 2.1 — Prinsip Inti

> **Kode yang baik adalah dokumentasi terbaik.** Inline comments hanya diperlukan ketika kode tidak bisa menjelaskan _mengapa_ sesuatu dilakukan.

AI **HANYA** boleh menulis inline comments pada **logika bisnis yang rumit atau non-obvious**. **JANGAN** mengomentari kode dasar yang sudah jelas dari konteksnya.

### 2.2 — Contoh: Kapan HARUS Comment

```php
// ✅ BENAR — Menjelaskan business rule yang tidak obvious
public function calculateDiscount(Order $order): float
{
    $discount = 0;

    // Diskon loyalty: pelanggan dengan 10+ transaksi mendapat 5% tambahan
    if ($order->user->completed_orders_count >= 10) {
        $discount += 0.05;
    }

    // Diskon gabungan tidak boleh melebihi 30% sesuai kebijakan perusahaan
    return min($discount, 0.30);
}

// ✅ BENAR — Menjelaskan workaround atau edge case
public function syncInventory(Product $product): void
{
    // Lock row untuk mencegah race condition pada concurrent checkout
    $product = Product::lockForUpdate()->find($product->id);

    // Spatie activity log v4 membutuhkan manual flush sebelum bulk update
    // Ref: https://github.com/spatie/laravel-activitylog/issues/XXX
    activity()->disableLogging();

    $product->update(['stock' => $this->calculateActualStock($product)]);

    activity()->enableLogging();
}

// ✅ BENAR — Menjelaskan alasan teknis
public function generateReport(): void
{
    // Gunakan chunk untuk menghindari memory overflow pada dataset besar (100k+ rows)
    User::active()->chunk(500, function ($users) {
        // ...
    });
}
```

### 2.3 — Contoh: Kapan JANGAN Comment

```php
// ❌ SALAH — Mengomentari kode dasar yang sudah jelas
public function store(StoreUserRequest $request): RedirectResponse
{
    // Ambil data yang sudah divalidasi
    $data = $request->validated();

    // Buat user baru
    $user = $this->userService->create($data);

    // Redirect ke halaman index dengan flash message
    return redirect()
        ->route('admin.users.index')
        ->with('success', 'User berhasil ditambahkan.');
}

// ✅ BENAR — Tanpa comment karena kodenya sudah self-explanatory
public function store(StoreUserRequest $request): RedirectResponse
{
    $this->userService->create($request->validated());

    return redirect()
        ->route('admin.users.index')
        ->with('success', 'User berhasil ditambahkan.');
}
```

```php
// ❌ SALAH — Comment redundan
// Set name
$user->name = $data['name'];

// Set email
$user->email = $data['email'];

// Save user
$user->save();

// ❌ SALAH — Comment yang mengulang kode
// Jika user aktif
if ($user->isActive()) {
    // Kirim email
    Mail::to($user)->send(new WelcomeMail());
}
```

### 2.4 — Aturan Inline Comment

| Situasi | Comment? | Alasan |
|---------|----------|--------|
| Business rule yang kompleks (diskon, kalkulasi, threshold) | ✅ Ya | Tidak bisa dipahami dari kode saja |
| Workaround / hack / bug fix | ✅ Ya | Perlu konteks mengapa solusi ini dipilih |
| Edge case yang tidak obvious | ✅ Ya | Pembaca perlu tahu batasan/kondisi khusus |
| Regex pattern yang kompleks | ✅ Ya | Regex sulit dibaca tanpa penjelasan |
| Performance optimization (chunking, caching, locking) | ✅ Ya | Perlu menjelaskan alasan optimasi |
| TODO / FIXME sementara | ✅ Ya | Tapi harus resolved sebelum merge |
| CRUD dasar (create, read, update, delete) | ❌ Tidak | Self-explanatory |
| Assignment variabel sederhana | ❌ Tidak | Jelas dari konteks |
| Memanggil method dengan nama deskriptif | ❌ Tidak | Nama method sudah cukup |
| Return / redirect standar | ❌ Tidak | Pattern Laravel umum |
| Eloquent query sederhana | ❌ Tidak | Framework convention |

### 2.5 — Section Separator di Model dan Class Besar

Gunakan section separator untuk mengorganisasi bagian-bagian class besar:

```php
class User extends Model
{
    /*
    |--------------------------------------------------------------------------
    | Relationships
    |--------------------------------------------------------------------------
    */

    public function orders(): HasMany { /* ... */ }

    /*
    |--------------------------------------------------------------------------
    | Scopes
    |--------------------------------------------------------------------------
    */

    public function scopeActive(Builder $query): Builder { /* ... */ }

    /*
    |--------------------------------------------------------------------------
    | Accessors & Mutators
    |--------------------------------------------------------------------------
    */

    public function getFullNameAttribute(): string { /* ... */ }

    /*
    |--------------------------------------------------------------------------
    | Business Logic
    |--------------------------------------------------------------------------
    */

    public function isEligibleForDiscount(): bool { /* ... */ }
}
```

---

## 3. README.md — Wajib Update Setiap Perubahan Signifikan

### 3.1 — Kapan AI WAJIB Update README.md

AI **WAJIB** memperbarui file `README.md` setiap kali:

| Trigger | Section yang Harus Diupdate |
|---------|----------------------------|
| Menambahkan package baru (`composer require` / `npm install`) | **Tech Stack** — tambahkan baris baru di tabel |
| Menambahkan fitur baru | **Fitur Utama** — tambahkan bullet point baru |
| Mengubah environment variable | **Konfigurasi** — perbarui daftar variabel |
| Mengubah langkah instalasi | **Instalasi Lokal** — perbarui step yang berubah |
| Menambahkan artisan command custom | **Menjalankan Aplikasi** atau section baru |
| Mengubah struktur folder | **Struktur Project** — perbarui tree diagram |
| Menambahkan akun seeder baru | **Akun Default** — perbarui tabel akun |
| Menambahkan API endpoint baru | Tambahkan link ke API documentation |

### 3.2 — Contoh Update Tech Stack

```markdown
<!-- SEBELUM -->
## Tech Stack

| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| Laravel | 11.x | Backend framework |
| Tailwind CSS | 3.x | Styling framework |

<!-- SESUDAH — Setelah install package baru -->
## Tech Stack

| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| Laravel | 11.x | Backend framework |
| Tailwind CSS | 3.x | Styling framework |
| Laravel Excel | 3.1 | Export/import data ke Excel |
| Intervention Image | 3.x | Image processing & thumbnail |
```

### 3.3 — Aturan README

- **JANGAN** meninggalkan placeholder `{{...}}` yang belum diisi.
- **JANGAN** menambahkan sesuatu ke Tech Stack tanpa benar-benar menginstall package-nya.
- **JANGAN** mendeskripsikan fitur yang belum diimplementasikan sebagai "sudah ada".
- **WAJIB** verifikasi bahwa langkah instalasi bisa diikuti dari nol (fresh clone) tanpa error.
- **WAJIB** update section **Akun Default** jika mengubah `DatabaseSeeder.php`.

---

## 4. .env.example — Wajib Sinkron dengan .env

### 4.1 — Kapan AI WAJIB Update .env.example

AI **WAJIB** memperbarui file `.env.example` **setiap kali**:

- Menambahkan variabel environment baru di `.env`.
- Menambahkan package yang memerlukan konfigurasi environment.
- Mengubah atau menghapus variabel environment yang ada.

```dotenv
# ✅ BENAR — .env.example selalu sinkron dengan .env

# === App ===
APP_NAME="Laravel App"
APP_ENV=local
APP_KEY=
APP_DEBUG=true
APP_URL=http://localhost:8000

# === Database ===
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_app
DB_USERNAME=root
DB_PASSWORD=

# === Mail (Mailtrap untuk development) ===
MAIL_MAILER=smtp
MAIL_HOST=sandbox.smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_FROM_ADDRESS="noreply@example.com"
MAIL_FROM_NAME="${APP_NAME}"

# === Payment Gateway (contoh: Midtrans) ===
# Ditambahkan saat install midtrans/laravel
MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
MIDTRANS_IS_PRODUCTION=false

# === Cloud Storage (contoh: S3) ===
# Ditambahkan saat install league/flysystem-aws-s3-v3
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=ap-southeast-1
AWS_BUCKET=
```

### 4.2 — Aturan .env.example

| Aturan | Detail |
|--------|--------|
| **WAJIB** sinkron | Setiap variabel di `.env` harus ada di `.env.example` |
| **JANGAN** taruh secret | `.env.example` tidak boleh berisi API key / password asli |
| **Berikan default value** | Gunakan value placeholder yang masuk akal (`localhost`, `false`, kosong) |
| **Kelompokkan** | Pisahkan variabel per section dengan komentar header (`# === Section ===`) |
| **Sertakan komentar** | Tambahkan komentar untuk variabel non-standar menjelaskan asal/tujuannya |
| **Urut** | Ikuti urutan: App → Database → Cache → Mail → Queue → External Services |

### 4.3 — Workflow Saat Menambahkan Package Baru

Setiap kali AI menjalankan `composer require` atau `npm install` untuk package baru, AI **WAJIB** melakukan langkah berikut secara berurutan:

```
1. Install package       → composer require vendor/package
2. Update .env.example   → Tambahkan variabel environment yang diperlukan
3. Update .env           → Isi value aktual (development)
4. Update README.md      → Tambahkan ke section Tech Stack
5. Update CHANGELOG.md   → Catat di section [Unreleased] → ### Ditambahkan
```

**Contoh:**

```bash
# 1. Install
composer require maatwebsite/laravel-excel
```

```dotenv
# 2. .env.example — tambahkan jika ada variabel baru
# (Laravel Excel tidak perlu env var, jadi skip)
```

```markdown
<!-- 4. README.md — Update Tech Stack -->
| [Laravel Excel](https://laravel-excel.com) | 3.1 | Export/import data ke Excel |
```

```markdown
<!-- 5. CHANGELOG.md -->
## [Unreleased]
### Ditambahkan
- Export/import data menggunakan Laravel Excel
```

---

## 5. REST API Documentation — Format JSON yang Konsisten

### 5.1 — Response Wrapper Standar

Semua API response **WAJIB** mengikuti format wrapper yang konsisten:

```json
// ✅ BENAR — Success response (single resource)
{
    "success": true,
    "message": "User berhasil dibuat.",
    "data": {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com",
        "roles": ["admin"],
        "created_at": "2026-02-23T08:00:00.000000Z"
    }
}

// ✅ BENAR — Success response (collection/paginated)
{
    "success": true,
    "message": "Daftar user berhasil diambil.",
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com"
        }
    ],
    "meta": {
        "current_page": 1,
        "per_page": 15,
        "total": 42,
        "last_page": 3
    }
}

// ✅ BENAR — Error response
{
    "success": false,
    "message": "Validasi gagal.",
    "errors": {
        "email": ["Email sudah digunakan."],
        "password": ["Password minimal 8 karakter."]
    }
}

// ✅ BENAR — Error response (non-validation)
{
    "success": false,
    "message": "User tidak ditemukan.",
    "errors": null
}
```

### 5.2 — HTTP Status Code Standar

| Status Code | Kondisi | Contoh |
|-------------|---------|--------|
| `200 OK` | Request berhasil (GET, PUT, DELETE) | Daftar user, update user, hapus user |
| `201 Created` | Resource baru dibuat (POST) | Buat user baru |
| `204 No Content` | Berhasil tanpa response body | Soft delete tanpa response |
| `400 Bad Request` | Request tidak valid | Parameter salah format |
| `401 Unauthorized` | Belum login / token expired | Token tidak valid |
| `403 Forbidden` | Login tapi tidak punya izin | Akses resource milik orang lain |
| `404 Not Found` | Resource tidak ditemukan | User ID tidak ada |
| `422 Unprocessable Entity` | Validasi gagal | Email duplikat |
| `429 Too Many Requests` | Rate limit exceeded | Terlalu banyak request |
| `500 Internal Server Error` | Server error | Bug di kode |

### 5.3 — API Naming Convention

```
# ✅ BENAR — RESTful, plural, kebab-case
GET     /api/users                  → List users
POST    /api/users                  → Create user
GET     /api/users/{user}           → Show user detail
PUT     /api/users/{user}           → Update user
DELETE  /api/users/{user}           → Delete user
GET     /api/users/{user}/orders    → List user's orders

GET     /api/product-categories     → List product categories
POST    /api/product-categories     → Create product category

# ❌ SALAH — Anti-pattern
GET     /api/getUsers               → Verb di URL
POST    /api/user/create            → Verb di URL
GET     /api/User/1                 → PascalCase
GET     /api/product_categories     → snake_case
```

### 5.4 — Scribe API Annotation Standar

Jika project menggunakan REST API, setiap endpoint **WAJIB** memiliki anotasi Scribe:

```php
/**
 * Daftar semua user
 *
 * Mengambil daftar user yang terdaftar dengan pagination.
 * Mendukung filter berdasarkan role dan status.
 *
 * @authenticated
 * @group User Management
 *
 * @queryParam  page      integer  Nomor halaman. Example: 1
 * @queryParam  per_page  integer  Jumlah item per halaman (max: 100). Example: 15
 * @queryParam  role      string   Filter berdasarkan role. Example: admin
 * @queryParam  search    string   Pencarian berdasarkan nama/email. Example: john
 *
 * @response 200 scenario="Success" {
 *   "success": true,
 *   "message": "Daftar user berhasil diambil.",
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
 *     "per_page": 15,
 *     "last_page": 2
 *   }
 * }
 *
 * @response 401 scenario="Unauthenticated" {
 *   "message": "Unauthenticated."
 * }
 */
public function index(): AnonymousResourceCollection
{
    // ...
}
```

**Aturan anotasi:**
- `@group` — WAJIB, kelompokkan endpoint berdasarkan domain (User Management, Order, Payment).
- `@authenticated` — WAJIB jika endpoint memerlukan token.
- `@queryParam` / `@bodyParam` / `@urlParam` — WAJIB untuk setiap parameter.
- `@response` — WAJIB minimal 1 success scenario + 1 error scenario.
- `Example:` — WAJIB sertakan contoh value yang realistis, bukan "string" atau "integer".

### 5.5 — JSON Format Rules

| Aturan | Contoh Benar | Contoh Salah |
|--------|-------------|-------------|
| Key menggunakan **snake_case** | `"created_at"`, `"user_id"` | `"createdAt"`, `"userId"` |
| Boolean sebagai `true`/`false` | `"is_active": true` | `"is_active": "true"` |
| Null ditulis sebagai `null` | `"deleted_at": null` | `"deleted_at": ""` |
| Date format **ISO 8601** | `"2026-02-23T08:00:00.000000Z"` | `"23-02-2026"` |
| ID sebagai integer | `"id": 1` | `"id": "1"` |
| Monetary sebagai integer (cents) ATAU string | `"amount": 150000` atau `"amount": "150000.00"` | `"amount": 150000.50` (float precision issue) |
| Array kosong sebagai `[]` | `"items": []` | `"items": null` (jika field wajib array) |
| Nesting max 3 level | `data.user.role` | `data.user.profile.address.city.district` |

---

## 6. CHANGELOG.md — Wajib Update Setiap Fitur Baru

### 6.1 — Format Standar

```markdown
## [Unreleased]

### Ditambahkan
- Fitur export data ke Excel menggunakan Laravel Excel
- API endpoint untuk manajemen produk

### Diubah
- Refactor UserService untuk mendukung bulk operations

### Diperbaiki
- Fix pagination yang salah pada halaman user list

### Dihapus
- Hapus endpoint deprecated `/api/v1/legacy-users`

### Keamanan
- Upgrade spatie/laravel-permission ke v6.4 untuk patch CVE-XXXX
```

### 6.2 — Aturan CHANGELOG

AI **WAJIB** menambahkan entry ke CHANGELOG setiap kali:

| Aksi | Category |
|------|----------|
| Menambah fitur baru | `### Ditambahkan` |
| Mengubah fitur yang sudah ada | `### Diubah` |
| Menghapus fitur | `### Dihapus` |
| Memperbaiki bug | `### Diperbaiki` |
| Perbaikan keamanan | `### Keamanan` |
| Breaking changes | `### Berubah (Breaking)` |

---

## 7. General Documentation Rules

### 7.1 — Bahasa

- PHPDoc dan inline comments: **Bahasa Indonesia** (konsisten dengan codebase).
- Deskripsi `@param` dan `@return`: Bahasa Indonesia singkat.
- `@throws`: Bahasa Indonesia, jelaskan kondisi kapan exception dilempar.
- README.md: **Bahasa Indonesia** (kecuali user request English).
- API error messages: **Bahasa Indonesia** (untuk user-facing), English untuk technical errors.

### 7.2 — Larangan Umum

- **JANGAN** meninggalkan `TODO`, `FIXME`, atau `HACK` di kode yang di-commit, kecuali disertai issue tracker reference (`// TODO(#123): ...`).
- **JANGAN** mendokumentasikan kode yang di-comment-out. Hapus kode tersebut.
- **JANGAN** menulis PHPDoc yang hanya mengulang nama method tanpa value tambah.
- **JANGAN** menggunakan `@inheritDoc` tanpa verifikasi bahwa parent class benar-benar punya PHPDoc.
- **JANGAN** meninggalkan `dd()`, `dump()`, `var_dump()` di kode. Gunakan `Log::debug()`.
- **JANGAN** buat dokumentasi untuk fitur yang belum diimplementasikan.

### 7.3 — Checklist Sebelum Commit

Sebelum menyelesaikan sebuah task, AI **WAJIB** memverifikasi:

- [ ] Semua class baru memiliki class-level PHPDoc
- [ ] Semua method publik baru memiliki `@param`, `@return`, dan `@throws` (jika ada)
- [ ] Inline comments hanya ada pada logika bisnis yang rumit
- [ ] `.env.example` sinkron dengan `.env` (jika ada variabel baru)
- [ ] `README.md` sudah diupdate (jika ada package/fitur baru)
- [ ] `CHANGELOG.md` sudah diupdate (jika ada perubahan fitur)
- [ ] Tidak ada placeholder `{{...}}`, `TODO`, atau `FIXME` yang belum resolved
- [ ] Tidak ada `dd()`, `dump()`, atau console.log di kode

---

## Ringkasan Aturan

| # | Aturan | Prioritas |
|---|--------|-----------|
| 1 | PHPDoc block di setiap Class (class-level description) | 🔴 Wajib |
| 2 | `@param`, `@return`, `@throws` di setiap public method | 🔴 Wajib |
| 3 | `@property` dan `@property-read` di setiap Model | 🔴 Wajib |
| 4 | Inline comments HANYA untuk logika bisnis yang rumit | 🔴 Wajib |
| 5 | JANGAN comment kode dasar (CRUD, assignment, redirect) | 🔴 Wajib |
| 6 | Update `.env.example` setiap tambah variabel environment | 🔴 Wajib |
| 7 | Update `README.md` setiap tambah package/fitur baru | 🔴 Wajib |
| 8 | Update `CHANGELOG.md` setiap perubahan fitur | 🟡 Penting |
| 9 | JSON response format konsisten (snake_case, ISO 8601) | 🟡 Penting |
| 10 | Response wrapper standar (`success`, `message`, `data`) | 🟡 Penting |
| 11 | Scribe annotations di setiap API endpoint | 🟡 Penting |
| 12 | Section separator di class besar (Relationships, Scopes, dll) | 🟡 Penting |
| 13 | JANGAN tinggalkan `TODO` / `FIXME` tanpa issue reference | 🟡 Penting |
| 14 | PHPDoc ditulis bersamaan dengan kode, bukan setelahnya | 🔴 Wajib |
