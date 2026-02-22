---
description: Aturan coding backend Laravel yang wajib dipatuhi AI — standar PSR-12, type hinting, naming conventions, arsitektur, Eloquent, dan keamanan.
---

# Laravel Backend Coding Rules

This document outlines the coding standards, architectural patterns, and best practices for the enterprise Laravel backend project using PHP, Eloquent ORM, and MVC/Service-Oriented and Clean Architecture principles.

---

## 1. Coding Standards — PSR-12

Semua kode PHP **WAJIB** mengikuti standar [PSR-12](https://www.php-fig.org/psr/psr-12/).

- **Indentasi**: gunakan 4 spasi, bukan tab.
- **Batas baris**: maksimal 120 karakter per baris.
- **Opening brace `{`**: pada baris baru untuk class dan method, pada baris yang sama untuk control structure (`if`, `for`, `foreach`, `while`, `switch`).
- **Satu statement per baris**. Jangan menggabungkan multiple statement.
- **Satu class per file**. Nama file harus sama persis dengan nama class (`UserController.php` → `class UserController`).
- **Spasi setelah keyword**: `if (`, `foreach (`, `return `, bukan `if(` atau `foreach(`.
- **Tidak ada trailing whitespace** di akhir baris.
- **File PHP HARUS** diakhiri dengan satu baris kosong (newline).
- **Tidak ada closing tag `?>`** di file PHP yang hanya berisi kode PHP.
- **Import `use`** dikelompokkan berdasarkan jenis dan diurutkan alfabet:
  1. PHP core classes
  2. Framework classes (Illuminate, Laravel)
  3. Third-party packages
  4. Application classes

```php
// ✅ Benar
use Carbon\Carbon;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use App\Models\User;
use App\Services\UserService;
```

---

## 2. Type Hinting — Wajib di Semua Tempat

Type hinting **WAJIB** digunakan pada **semua** parameter method, return type, dan property.

- **Parameter method**: selalu gunakan type hint.
- **Return type**: selalu deklarasikan, termasuk `void`.
- **Property type** (PHP 8.x): selalu deklarasikan tipe pada class property.
- **Nullable type**: gunakan `?Type` jika value bisa `null`.
- **Union type**: gunakan `Type1|Type2` jika diperlukan (PHP 8.0+).
- **Array shape**: dokumentasikan struktur array menggunakan PHPDoc `@param array{key: type}`.

```php
// ✅ Benar — semua parameter dan return di-type-hint
public function findActiveByRole(string $role, int $limit = 10): Collection
{
    return User::role($role)->where('is_active', true)->limit($limit)->get();
}

// ✅ Property type declaration
protected UserRepositoryInterface $repository;
protected string $defaultRole = 'user';

// ✅ Nullable
public function findByEmail(string $email): ?User
{
    return User::where('email', $email)->first();
}

// ❌ Salah — tanpa type hint
public function findUser($id)
{
    return User::find($id);
}
```

**Aturan khusus:**
- Jangan gunakan `mixed` kecuali benar-benar polimorfik.
- Jangan gunakan `array` tanpa PHPDoc yang menjelaskan strukturnya.
- Selalu gunakan return type `void` untuk method yang tidak mengembalikan nilai.
- Gunakan `self` atau `static` sebagai return type jika method mengembalikan instance objek yang sama.

---

## 3. Naming Conventions

### 3.1 — Class

| Tipe | Convention | Contoh |
|------|-----------|--------|
| Model | Singular, PascalCase | `User`, `ProductCategory`, `OrderItem` |
| Controller | Singular + `Controller` | `UserController`, `ProductController` |
| Service | Singular + `Service` | `UserService`, `PaymentService` |
| Repository | Singular + `Repository` | `UserRepository`, `OrderRepository` |
| Repository Interface | Singular + `RepositoryInterface` | `UserRepositoryInterface` |
| Form Request | Action + Model + `Request` | `StoreUserRequest`, `UpdateProductRequest` |
| Resource | Singular + `Resource` | `UserResource`, `ProductResource` |
| Event | Past tense PascalCase | `UserCreated`, `OrderShipped` |
| Listener | Imperative PascalCase | `SendWelcomeEmail`, `UpdateInventory` |
| Job | Imperative PascalCase | `ProcessPayment`, `GenerateReport` |
| Mail | Descriptive PascalCase | `WelcomeMail`, `InvoiceMail` |
| Notification | Descriptive PascalCase | `OrderConfirmation`, `PasswordReset` |
| Policy | Singular + `Policy` | `UserPolicy`, `PostPolicy` |
| Middleware | Descriptive PascalCase | `EnsureEmailIsVerified`, `CheckRole` |
| Enum | Singular PascalCase | `UserStatus`, `OrderType`, `PaymentMethod` |
| Trait | Adjective/ability PascalCase | `HasRoles`, `Searchable`, `Auditable` |
| DTO | Descriptive + `DTO` | `CreateUserDTO`, `UpdateProductDTO` |

### 3.2 — Method

| Konteks | Convention | Contoh |
|---------|-----------|--------|
| Controller CRUD | Standard Laravel verbs | `index`, `create`, `store`, `show`, `edit`, `update`, `destroy` |
| Service | camelCase, verb-first | `createUser()`, `calculateTotal()`, `syncRoles()` |
| Repository | camelCase, verb-first | `findById()`, `findByEmail()`, `getActivePaginated()` |
| Scope (Model) | camelCase, prefix `scope` | `scopeActive()`, `scopeByRole()` |
| Accessor | camelCase, format baru | `statusLabel(): Attribute` |
| Boolean method | camelCase, prefix `is`/`has`/`can` | `isActive()`, `hasPermission()`, `canEdit()` |
| Private/helper | camelCase | `formatResponse()`, `buildQuery()` |

### 3.3 — Database

| Tipe | Convention | Contoh |
|------|-----------|--------|
| Tabel | snake_case, plural | `users`, `product_categories`, `order_items` |
| Kolom | snake_case, singular | `first_name`, `is_active`, `created_at` |
| Foreign key | singular_model + `_id` | `user_id`, `category_id`, `parent_id` |
| Pivot table | singular alphabetical, snake_case | `role_user`, `category_product` |
| Migration | descriptive snake_case | `create_users_table`, `add_phone_to_users_table` |
| Boolean column | prefix `is_` atau `has_` | `is_active`, `is_verified`, `has_subscription` |
| Timestamp column | suffix `_at` | `published_at`, `verified_at`, `deleted_at` |

### 3.4 — Variabel dan Konstanta

| Tipe | Convention | Contoh |
|------|-----------|--------|
| Variable | camelCase | `$userData`, `$totalAmount`, `$isActive` |
| Class constant | UPPER_SNAKE_CASE | `MAX_LOGIN_ATTEMPTS`, `DEFAULT_PAGINATION` |
| Config key | snake_case | `config('app.timezone')`, `config('services.payment.key')` |
| Route name | dot notation, lowercase | `admin.users.index`, `api.products.store` |
| Route URL | kebab-case, plural | `/admin/users`, `/api/product-categories` |

---

## 4. Architecture — Controller, Form Request, Service

### 4.1 — Controller: Thin Controller, TANPA Business Logic

Controller **HANYA** boleh berisi:
- Memanggil Form Request (validasi otomatis)
- Memanggil Service class
- Mengembalikan response (view atau JSON)

**DILARANG di Controller:**
- ❌ Query database langsung (Eloquent/DB facade)
- ❌ Validasi manual (`$request->validate()`) — gunakan Form Request
- ❌ Business logic (kalkulasi, kondisi bisnis, dsb.)
- ❌ Manipulasi data kompleks
- ❌ Try-catch yang menutupi error (kecuali untuk response formatting khusus API)

```php
// ✅ Benar — Controller tipis
class UserController extends Controller
{
    public function __construct(
        protected UserService $userService
    ) {}

    public function store(StoreUserRequest $request): RedirectResponse
    {
        $this->userService->create($request->validated());

        return redirect()
            ->route('admin.users.index')
            ->with('success', 'User berhasil ditambahkan.');
    }
}

// ❌ Salah — Controller gemuk
class UserController extends Controller
{
    public function store(Request $request)
    {
        $request->validate([                    // ❌ Validasi di controller
            'name' => 'required',
            'email' => 'required|email|unique:users',
        ]);

        $user = new User();                     // ❌ Query langsung
        $user->name = $request->name;
        $user->email = $request->email;
        $user->password = Hash::make($request->password);

        if ($request->role === 'admin') {       // ❌ Business logic
            $user->level = 10;
            $user->is_premium = true;
        }

        $user->save();                          // ❌ Persistence di controller
        $user->assignRole($request->role);

        return redirect()->route('admin.users.index');
    }
}
```

### 4.2 — Form Request: Semua Validasi di Sini

- **WAJIB** buat Form Request untuk setiap operasi `store` dan `update`.
- Gunakan `authorize()` untuk cek permission.
- Gunakan `messages()` untuk pesan error Bahasa Indonesia.
- Gunakan `prepareForValidation()` jika perlu normalisasi input sebelum validasi.

```php
class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('user-create');
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => ['required', 'email', 'unique:users,email'],
            'password' => ['required', 'confirmed', Password::defaults()],
            'roles'    => ['required', 'array', 'min:1'],
            'roles.*'  => ['string', 'exists:roles,name'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required' => 'Nama wajib diisi.',
            'email.unique'  => 'Email sudah digunakan.',
        ];
    }

    protected function prepareForValidation(): void
    {
        $this->merge([
            'email' => strtolower(trim($this->email)),
        ]);
    }
}
```

### 4.3 — Service: Semua Business Logic di Sini

- Satu Service per domain/entity.
- Inject Repository via constructor (dependency injection).
- Return model/DTO/collection, **bukan** response atau redirect.
- Handle exception di sini jika perlu wrapping logic.

```php
class UserService
{
    public function __construct(
        protected UserRepositoryInterface $repository
    ) {}

    public function create(array $data): User
    {
        $user = $this->repository->create([
            'name'     => $data['name'],
            'email'    => $data['email'],
            'password' => Hash::make($data['password']),
        ]);

        $user->assignRole($data['roles']);

        // Business logic tambahan:
        // - Kirim welcome email
        // - Log aktivitas
        // - Trigger event

        return $user;
    }
}
```

### 4.4 — Alur arsitektur yang benar

```
Request → Middleware → FormRequest (validasi + authorize)
       → Controller (thin, hanya orchestrate)
       → Service (business logic)
       → Repository (data access)
       → Model (Eloquent)
       → Response (View / JSON)
```

---

## 5. Eloquent ORM — Aturan Penggunaan

### 5.1 — WAJIB hindari N+1 Query

N+1 adalah masalah performa paling umum di Laravel. **WAJIB** gunakan eager loading.

```php
// ❌ N+1 problem — 1 query users + N query roles
$users = User::all();
foreach ($users as $user) {
    echo $user->roles->first()->name;  // Query per iterasi!
}

// ✅ Eager loading — hanya 2 query total
$users = User::with('roles')->get();
foreach ($users as $user) {
    echo $user->roles->first()->name;  // Sudah di-load
}

// ✅ Eager loading dengan constraint
$users = User::with(['roles', 'orders' => function ($query) {
    $query->where('status', 'completed')->latest()->limit(5);
}])->get();

// ✅ Eager loading di pagination
$users = User::with('roles')->latest()->paginate(15);
```

**Aturan:**
- Selalu gunakan `with()` jika akan mengakses relasi di loop.
- Gunakan `withCount()` untuk menghitung relasi tanpa load data.
- Aktifkan `Model::preventLazyLoading()` di `AppServiceProvider` saat development:

```php
// app/Providers/AppServiceProvider.php
public function boot(): void
{
    Model::preventLazyLoading(!$this->app->isProduction());
}
```

### 5.2 — Gunakan Query Scopes

Jangan duplikasi query condition. Gunakan scope.

```php
// ❌ Duplikasi di banyak tempat
User::where('is_active', true)->where('email_verified_at', '!=', null)->get();

// ✅ Gunakan scope
// Di Model:
public function scopeActive(Builder $query): Builder
{
    return $query->where('is_active', true);
}

public function scopeVerified(Builder $query): Builder
{
    return $query->whereNotNull('email_verified_at');
}

// Di Controller/Service:
User::active()->verified()->get();
```

### 5.3 — Gunakan Mass Assignment dengan benar

- Selalu definisikan `$fillable` di Model.
- **JANGAN** gunakan `$guarded = []` (mengizinkan semua field).
- **JANGAN** pernah pass `$request->all()` langsung ke `create()` atau `update()`.

```php
// ✅ Benar
protected $fillable = ['name', 'email', 'password'];

// Dan gunakan validated data:
User::create($request->validated());

// ❌ Salah — mengizinkan semua field
protected $guarded = [];

// ❌ Salah — pass semua input tanpa filter
User::create($request->all());
```

### 5.4 — Aturan Eloquent lainnya

- **Selalu gunakan `$casts`** untuk kolom date, boolean, enum, json, decimal.
- **Gunakan `firstOrCreate()` / `updateOrCreate()`** untuk upsert, hindari manual check + create.
- **Gunakan chunking** untuk memproses data besar: `chunk()`, `chunkById()`, `lazy()`, `cursor()`.
- **Hindari `DB::raw()`** kecuali benar-benar diperlukan. Gunakan Eloquent method.
- **Hapus model via Eloquent** (`$model->delete()`), bukan via raw query, agar event dan observer ter-trigger.

---

## 6. Security Rules

### 6.1 — Mass Assignment Protection

- **WAJIB** definisikan `$fillable` di setiap Model.
- **JANGAN** gunakan `$guarded = []`.
- **JANGAN** pass `$request->all()` ke Eloquent. Gunakan `$request->validated()` atau `$request->only([...])`.

### 6.2 — Authorization — Policies & Gates

- **WAJIB** gunakan Policies untuk otorisasi resource-based.
- **WAJIB** gunakan `authorize()` di Form Request atau `$this->authorize()` di Controller.
- **JANGAN** hanya mengandalkan Blade directives (`@can`) — selalu ada server-side check.

```php
// ✅ Policy
class UserPolicy
{
    public function update(User $authUser, User $targetUser): bool
    {
        return $authUser->hasPermissionTo('user-edit');
    }

    public function delete(User $authUser, User $targetUser): bool
    {
        // Tidak boleh hapus diri sendiri
        if ($authUser->id === $targetUser->id) {
            return false;
        }

        return $authUser->hasPermissionTo('user-delete');
    }
}

// ✅ Gate (untuk otorisasi sederhana non-resource)
Gate::define('access-admin-panel', function (User $user): bool {
    return $user->hasAnyRole(['super-admin', 'admin']);
});
```

### 6.3 — SQL Injection Prevention

- **JANGAN** pernah interpolasi variabel ke raw query.
- Gunakan parameter binding jika terpaksa pakai raw query.

```php
// ❌ BERBAHAYA — SQL injection
DB::select("SELECT * FROM users WHERE email = '$email'");

// ✅ Aman — parameter binding
DB::select("SELECT * FROM users WHERE email = ?", [$email]);

// ✅ Paling aman — Eloquent
User::where('email', $email)->first();
```

### 6.4 — XSS Prevention

- Blade `{{ }}` sudah auto-escape. **JANGAN** gunakan `{!! !!}` kecuali benar-benar diperlukan dan data sudah di-sanitize.
- Selalu gunakan `e()` helper jika perlu manual escape.

### 6.5 — CSRF

- **Semua form POST/PUT/PATCH/DELETE** harus menyertakan `@csrf`.
- API yang diproteksi harus menggunakan token-based auth (Sanctum/Passport), bukan session CSRF.

### 6.6 — Password & Sensitive Data

- **WAJIB** hash password menggunakan `Hash::make()` atau `bcrypt()`.
- **JANGAN** pernah log, dump, atau return password dalam response.
- **JANGAN** simpan API key/secret di kode. Gunakan `.env` dan `config()`.
- Gunakan `$hidden` di Model untuk menyembunyikan field sensitif dari serialization:

```php
protected $hidden = [
    'password',
    'remember_token',
    'two_factor_secret',
];
```

---

## 7. Error Handling

- **JANGAN** gunakan `try-catch` yang menelan error tanpa logging.
- **Gunakan custom exceptions** untuk error domain-specific.
- **Log error** dengan konteks yang cukup: `Log::error($message, ['context' => $data])`.
- **JANGAN** return response `500` dengan stack trace di production (`APP_DEBUG=false`).
- **Gunakan `abort()`** untuk HTTP error standar: `abort(404)`, `abort(403)`.

```php
// ❌ Salah — error ditelan
try {
    $user = $this->service->create($data);
} catch (\Exception $e) {
    // Diam-diam gagal tanpa log
    return redirect()->back();
}

// ✅ Benar
try {
    $user = $this->service->create($data);
} catch (\Exception $e) {
    Log::error('Failed to create user', [
        'data'  => $data,
        'error' => $e->getMessage(),
    ]);

    return redirect()->back()
        ->with('error', 'Gagal membuat user. Silakan coba lagi.')
        ->withInput();
}
```

---

## 8. General Code Quality Rules

- **JANGAN** gunakan `dd()` atau `dump()` di kode yang di-commit. Gunakan `Log::debug()`.
- **JANGAN** gunakan `env()` di luar file `config/`. Gunakan `config()`.
- **JANGAN** hardcode values. Gunakan config, enum, atau konstanta.
- **WAJIB** gunakan Dependency Injection via constructor, bukan `app()` atau facade jika memungkinkan.
- **WAJIB** gunakan Early Return untuk mengurangi nesting:

```php
// ❌ Nesting dalam
public function process(User $user): void
{
    if ($user->isActive()) {
        if ($user->hasPermission('process')) {
            if ($user->balance > 0) {
                // Logic di sini... 3 level nesting
            }
        }
    }
}

// ✅ Early return
public function process(User $user): void
{
    if (!$user->isActive()) return;
    if (!$user->hasPermission('process')) return;
    if ($user->balance <= 0) return;

    // Logic di sini... 0 nesting
}
```

- **JANGAN** buat method lebih dari 30 baris. Jika lebih, pecah menjadi method yang lebih kecil.
- **JANGAN** buat class lebih dari 300 baris. Jika lebih, pecah tanggung jawabnya.
- **Setiap method hanya boleh punya satu tanggung jawab** (Single Responsibility).
- **Selalu sertakan PHPDoc** sesuai standar yang didefinisikan di `07_documentation-maintenance.md`.

---

## Ringkasan Aturan

| # | Aturan | Prioritas |
|---|--------|-----------|
| 1 | Ikuti PSR-12 | 🔴 Wajib |
| 2 | Type hint di semua parameter, return, property | 🔴 Wajib |
| 3 | Naming convention konsisten (lihat tabel) | 🔴 Wajib |
| 4 | Controller tipis — TANPA business logic & validasi | 🔴 Wajib |
| 5 | Validasi di Form Request, logic di Service | 🔴 Wajib |
| 6 | Eager loading — hindari N+1 | 🔴 Wajib |
| 7 | `$fillable` di setiap Model — JANGAN `$guarded=[]` | 🔴 Wajib |
| 8 | Gunakan `$request->validated()`, bukan `$request->all()` | 🔴 Wajib |
| 9 | Policy/Gate untuk authorization | 🔴 Wajib |
| 10 | Hash password, sembunyikan field sensitif | 🔴 Wajib |
| 11 | JANGAN `dd()`, `dump()`, `env()` di luar config | 🟡 Penting |
| 12 | PHPDoc di semua class dan public method | 🟡 Penting |
| 13 | Early return, max 30 baris per method | 🟡 Penting |
| 14 | Dependency Injection via constructor | 🟡 Penting |
