---
description: SOP untuk testing dan debugging di Laravel — termasuk Debugbar, Feature Test (CRUD endpoint), Unit Test, dan strategi test coverage pada fitur krusial.
---

# 04 — Testing & Debugging

> **Tujuan**: Menyiapkan environment testing dan debugging yang solid di Laravel. Termasuk instalasi Laravel Debugbar untuk development, penulisan Feature Test untuk endpoint CRUD, Unit Test untuk business logic, dan memastikan coverage test pada fitur-fitur krusial.

---

## Prasyarat

- Laravel sudah terinstall dengan database terhubung
- Autentikasi sudah di-setup (lihat `03_admin-dashboard-auth.md`)
- Minimal 1 fitur CRUD sudah ada (Model + Controller + Route)

---

## BAGIAN A: DEBUGGING

---

## Step 1 — Install Laravel Debugbar

### 1.1 — Install package (dev-only)

// turbo
```bash
composer require barryvdh/laravel-debugbar --dev
```

> [!IMPORTANT]
> Flag `--dev` memastikan Debugbar **hanya terinstall di environment development**, bukan production. Ini sangat penting untuk keamanan dan performa.

### 1.2 — Publish config (opsional)

// turbo
```bash
php artisan vendor:publish --provider="Barryvdh\Debugbar\ServiceProvider"
```

Ini akan membuat file `config/debugbar.php`.

### 1.3 — Konfigurasi di `.env`

Pastikan `.env` memiliki setting berikut:

```dotenv
APP_DEBUG=true
DEBUGBAR_ENABLED=true
```

> [!CAUTION]
> **JANGAN PERNAH** set `DEBUGBAR_ENABLED=true` atau `APP_DEBUG=true` di environment **production/staging**. Debugbar mengekspos query SQL, session data, request data, dan informasi sensitif lainnya.

### 1.4 — Konfigurasi Debugbar (`config/debugbar.php`)

Edit konfigurasi yang direkomendasikan:

```php
return [
    'enabled' => env('DEBUGBAR_ENABLED', false),
    'storage' => [
        'enabled' => true,
        'open' => false,
        'driver' => 'file',
        'path' => storage_path('debugbar'),
    ],
    'collectors' => [
        'phpinfo'         => true,
        'messages'        => true,
        'time'            => true,
        'memory'          => true,
        'exceptions'      => true,
        'log'             => true,
        'db'              => true,     // Query SQL — sangat berguna
        'views'           => true,     // View yang dirender
        'route'           => true,     // Route info
        'auth'            => true,     // Auth status
        'gate'            => true,     // Authorization gates
        'session'         => true,     // Session data
        'symfony_request' => true,
        'mail'            => true,
        'laravel'         => false,
        'events'          => false,    // Disable jika terlalu noisy
        'default_request' => false,
        'logs'            => false,
        'files'           => false,
        'config'          => false,
        'cache'           => true,
        'models'          => true,     // Model loaded — deteksi N+1
        'livewire'        => false,
    ],
];
```

### 1.5 — Verifikasi Debugbar berjalan

// turbo
```bash
php artisan serve
```

Buka `http://localhost:8000` di browser. Pastikan **Debugbar bar** muncul di bagian bawah halaman.

> [!TIP]
> **Fitur Debugbar yang paling berguna:**
> - **Queries**: Melihat semua SQL query yang dijalankan + execution time → deteksi N+1 problem
> - **Models**: Melihat jumlah model yang di-load
> - **Timeline**: Melihat waktu eksekusi setiap proses
> - **Exceptions**: Melihat error/exception yang terjadi
> - **Route**: Melihat route matching dan middleware

### 1.6 — Tambahkan debug helper di code (opsional)

Debugbar bisa digunakan langsung di code untuk debugging:

```php
use Barryvdh\Debugbar\Facades\Debugbar;

// Tambahkan pesan ke tab Messages
Debugbar::info('Ini informasi debug');
Debugbar::warning('Ini warning');
Debugbar::error('Ini error');

// Measure waktu eksekusi
Debugbar::startMeasure('proses-berat', 'Proses Berat');
// ... kode yang mau diukur ...
Debugbar::stopMeasure('proses-berat');

// Atau gunakan helper global (jika tersedia):
// debug($variable);     // sama seperti Debugbar::info()
// debugbar()->info($x);
```

> [!NOTE]
> Pastikan semua panggilan `Debugbar::*` di-wrap dengan pengecekan environment, atau gunakan code yang aman di production:
> ```php
> if (app()->hasDebugModeEnabled()) {
>     Debugbar::info('Debug message');
> }
> ```

---

## BAGIAN B: TESTING SETUP

---

## Step 2 — Konfigurasi Testing Environment

### 2.1 — Buat file `.env.testing`

Buat file `.env.testing` di root project:

```dotenv
APP_NAME="Laravel Test"
APP_ENV=testing
APP_DEBUG=true
APP_KEY=                            
# APP_KEY akan di-copy dari .env utama, atau generate baru

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db_test
DB_USERNAME=root
DB_PASSWORD=
```

> [!IMPORTANT]
> - Gunakan **database terpisah** untuk testing (`laravel_db_test`). JANGAN gunakan database development/production!
> - Tanyakan ke user untuk nama database testing dan kredensial MySQL.
> - `APP_KEY` bisa dikosongkan — Laravel akan menggunakan key dari `.env` utama saat testing.

### 2.2 — Buat database testing

// turbo
```bash
mysql -u root -e "CREATE DATABASE IF NOT EXISTS laravel_db_test CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

> Ganti `laravel_db_test` dengan nilai `DB_DATABASE` dari `.env.testing`.

### 2.3 — Konfigurasi `phpunit.xml`

Buka `phpunit.xml` dan pastikan environment variables sudah dikonfigurasi:

```xml
<phpunit>
    <!-- ... -->
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="APP_MAINTENANCE_DRIVER" value="file"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_STORE" value="array"/>
        <env name="DB_CONNECTION" value="mysql"/>
        <env name="DB_DATABASE" value="laravel_db_test"/>
        <env name="MAIL_MAILER" value="array"/>
        <env name="PULSE_ENABLED" value="false"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

> [!NOTE]
> - `BCRYPT_ROUNDS=4` mempercepat hashing saat test.
> - `CACHE_STORE=array` dan `SESSION_DRIVER=array` memastikan test terisolasi.
> - `QUEUE_CONNECTION=sync` agar job dijalankan langsung saat test.

### 2.4 — Pilih testing framework

Laravel mendukung dua testing framework:

| Framework | Kelebihan | Kapan Digunakan |
|-----------|-----------|-----------------|
| **Pest** | Syntax lebih bersih, chaining, less boilerplate | Recommended untuk project baru |
| **PHPUnit** | Lebih mature, class-based, familiar | Jika team sudah terbiasa |

**Default**: Gunakan **Pest** (built-in di Laravel 11+). Jika project menggunakan PHPUnit, sesuaikan syntax di contoh-contoh di bawah.

### 2.5 — Verifikasi Pest tersedia

// turbo
```bash
php artisan test --version
```

Jika Pest belum terinstall:

// turbo
```bash
composer require pestphp/pest --dev --with-all-dependencies
composer require pestphp/pest-plugin-laravel --dev
```

---

## BAGIAN C: FEATURE TEST (Endpoint CRUD)

---

## Step 3 — Template Feature Test untuk CRUD Endpoint

### 3.1 — Generate Feature Test

// turbo
```bash
php artisan make:test Admin/ExampleCrudTest --pest
```

> Ganti `ExampleCrud` dengan nama fitur/resource yang sedang ditest.
> Untuk PHPUnit, hilangkan flag `--pest`.

### 3.2 — Template Feature Test (Pest)

Buka `tests/Feature/Admin/ExampleCrudTest.php` dan gunakan template berikut:

```php
<?php

use App\Models\User;
use App\Models\Example; // Ganti dengan Model yang ditest
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

/*
|--------------------------------------------------------------------------
| Setup: buat user authenticated untuk setiap test
|--------------------------------------------------------------------------
*/
beforeEach(function () {
    $this->user = User::factory()->create();
    $this->actingAs($this->user);
});

/*
|--------------------------------------------------------------------------
| INDEX — Menampilkan daftar resource
|--------------------------------------------------------------------------
*/
describe('GET /admin/examples (index)', function () {

    it('menampilkan halaman daftar', function () {
        $response = $this->get(route('admin.examples.index'));

        $response->assertStatus(200);
        $response->assertViewIs('admin.examples.index');
    });

    it('redirect ke login jika belum auth', function () {
        // Logout terlebih dahulu
        auth()->logout();

        $response = $this->get(route('admin.examples.index'));

        $response->assertRedirect(route('login'));
    });

    it('menampilkan data yang ada di database', function () {
        $examples = Example::factory()->count(3)->create();

        $response = $this->get(route('admin.examples.index'));

        $response->assertStatus(200);
        foreach ($examples as $example) {
            $response->assertSee($example->name);
        }
    });
});

/*
|--------------------------------------------------------------------------
| CREATE — Menampilkan form pembuatan resource baru
|--------------------------------------------------------------------------
*/
describe('GET /admin/examples/create (create)', function () {

    it('menampilkan form create', function () {
        $response = $this->get(route('admin.examples.create'));

        $response->assertStatus(200);
        $response->assertViewIs('admin.examples.create');
    });
});

/*
|--------------------------------------------------------------------------
| STORE — Menyimpan resource baru ke database
|--------------------------------------------------------------------------
*/
describe('POST /admin/examples (store)', function () {

    it('berhasil menyimpan data valid', function () {
        $data = [
            'name' => 'Test Example',
            'description' => 'Deskripsi test example',
            // Tambahkan field lain sesuai kebutuhan
        ];

        $response = $this->post(route('admin.examples.store'), $data);

        $response->assertRedirect(route('admin.examples.index'));
        $response->assertSessionHas('success');

        $this->assertDatabaseHas('examples', [
            'name' => 'Test Example',
        ]);
    });

    it('gagal menyimpan data invalid — validasi error', function () {
        $data = [
            'name' => '',  // required field kosong
        ];

        $response = $this->post(route('admin.examples.store'), $data);

        $response->assertSessionHasErrors(['name']);
        $this->assertDatabaseCount('examples', 0);
    });

    it('gagal menyimpan data duplikat (jika ada unique constraint)', function () {
        Example::factory()->create(['name' => 'Duplicate']);

        $data = ['name' => 'Duplicate'];

        $response = $this->post(route('admin.examples.store'), $data);

        $response->assertSessionHasErrors(['name']);
    });
});

/*
|--------------------------------------------------------------------------
| SHOW — Menampilkan detail resource
|--------------------------------------------------------------------------
*/
describe('GET /admin/examples/{id} (show)', function () {

    it('menampilkan detail resource yang ada', function () {
        $example = Example::factory()->create();

        $response = $this->get(route('admin.examples.show', $example));

        $response->assertStatus(200);
        $response->assertSee($example->name);
    });

    it('return 404 untuk resource yang tidak ada', function () {
        $response = $this->get(route('admin.examples.show', 99999));

        $response->assertStatus(404);
    });
});

/*
|--------------------------------------------------------------------------
| EDIT — Menampilkan form edit resource
|--------------------------------------------------------------------------
*/
describe('GET /admin/examples/{id}/edit (edit)', function () {

    it('menampilkan form edit dengan data yang benar', function () {
        $example = Example::factory()->create();

        $response = $this->get(route('admin.examples.edit', $example));

        $response->assertStatus(200);
        $response->assertViewIs('admin.examples.edit');
        $response->assertSee($example->name);
    });
});

/*
|--------------------------------------------------------------------------
| UPDATE — Mengupdate resource yang ada
|--------------------------------------------------------------------------
*/
describe('PUT /admin/examples/{id} (update)', function () {

    it('berhasil mengupdate data valid', function () {
        $example = Example::factory()->create(['name' => 'Old Name']);

        $data = [
            'name' => 'New Name',
            'description' => 'Updated description',
        ];

        $response = $this->put(route('admin.examples.update', $example), $data);

        $response->assertRedirect(route('admin.examples.index'));
        $response->assertSessionHas('success');

        $this->assertDatabaseHas('examples', [
            'id' => $example->id,
            'name' => 'New Name',
        ]);
    });

    it('gagal mengupdate dengan data invalid', function () {
        $example = Example::factory()->create();

        $data = ['name' => ''];

        $response = $this->put(route('admin.examples.update', $example), $data);

        $response->assertSessionHasErrors(['name']);
    });
});

/*
|--------------------------------------------------------------------------
| DELETE — Menghapus resource
|--------------------------------------------------------------------------
*/
describe('DELETE /admin/examples/{id} (destroy)', function () {

    it('berhasil menghapus resource', function () {
        $example = Example::factory()->create();

        $response = $this->delete(route('admin.examples.destroy', $example));

        $response->assertRedirect(route('admin.examples.index'));
        $response->assertSessionHas('success');

        $this->assertDatabaseMissing('examples', [
            'id' => $example->id,
        ]);
    });

    it('return 404 saat menghapus resource yang tidak ada', function () {
        $response = $this->delete(route('admin.examples.destroy', 99999));

        $response->assertStatus(404);
    });
});
```

### 3.3 — Template Feature Test (PHPUnit) — Alternatif

Jika project menggunakan PHPUnit, gunakan template ini sebagai gantinya:

```php
<?php

namespace Tests\Feature\Admin;

use App\Models\User;
use App\Models\Example;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleCrudTest extends TestCase
{
    use RefreshDatabase;

    protected User $user;

    protected function setUp(): void
    {
        parent::setUp();
        $this->user = User::factory()->create();
    }

    /** @test */
    public function index_menampilkan_halaman_daftar(): void
    {
        $response = $this->actingAs($this->user)
            ->get(route('admin.examples.index'));

        $response->assertStatus(200);
        $response->assertViewIs('admin.examples.index');
    }

    /** @test */
    public function index_redirect_jika_belum_login(): void
    {
        $response = $this->get(route('admin.examples.index'));

        $response->assertRedirect(route('login'));
    }

    /** @test */
    public function store_berhasil_menyimpan_data_valid(): void
    {
        $data = [
            'name' => 'Test Example',
            'description' => 'Deskripsi test',
        ];

        $response = $this->actingAs($this->user)
            ->post(route('admin.examples.store'), $data);

        $response->assertRedirect(route('admin.examples.index'));
        $this->assertDatabaseHas('examples', ['name' => 'Test Example']);
    }

    /** @test */
    public function store_gagal_dengan_data_invalid(): void
    {
        $response = $this->actingAs($this->user)
            ->post(route('admin.examples.store'), ['name' => '']);

        $response->assertSessionHasErrors(['name']);
    }

    /** @test */
    public function update_berhasil_mengupdate_data(): void
    {
        $example = Example::factory()->create(['name' => 'Old']);

        $response = $this->actingAs($this->user)
            ->put(route('admin.examples.update', $example), ['name' => 'New']);

        $response->assertRedirect(route('admin.examples.index'));
        $this->assertDatabaseHas('examples', ['id' => $example->id, 'name' => 'New']);
    }

    /** @test */
    public function destroy_berhasil_menghapus_resource(): void
    {
        $example = Example::factory()->create();

        $response = $this->actingAs($this->user)
            ->delete(route('admin.examples.destroy', $example));

        $response->assertRedirect(route('admin.examples.index'));
        $this->assertDatabaseMissing('examples', ['id' => $example->id]);
    }
}
```

### 3.4 — Adaptasi template ke fitur nyata

> [!IMPORTANT]
> **Setiap kali membuat fitur CRUD baru**, copy template di atas dan sesuaikan:
> 1. Ganti `Example` dengan nama Model yang sebenarnya
> 2. Ganti `examples` dengan nama tabel/route resource
> 3. Sesuaikan `$data` dengan field yang ada di Model
> 4. Tambahkan test untuk edge case spesifik fitur tersebut

---

## BAGIAN D: UNIT TEST (Business Logic)

---

## Step 4 — Template Unit Test untuk Service Layer

### 4.1 — Generate Unit Test

// turbo
```bash
php artisan make:test Services/ExampleServiceTest --unit --pest
```

### 4.2 — Template Unit Test (Pest)

Buat file `tests/Unit/Services/ExampleServiceTest.php`:

```php
<?php

use App\Services\ExampleService;
use App\Repositories\Contracts\ExampleRepositoryInterface;
use App\Models\Example;
use Illuminate\Database\Eloquent\Collection;

/*
|--------------------------------------------------------------------------
| Unit Test: ExampleService
|--------------------------------------------------------------------------
| Test business logic secara terisolasi menggunakan mock repository.
| TIDAK menggunakan RefreshDatabase — semua dependency di-mock.
*/

beforeEach(function () {
    // Mock repository
    $this->repository = Mockery::mock(ExampleRepositoryInterface::class);

    // Inject mock ke service
    $this->service = new ExampleService($this->repository);
});

afterEach(function () {
    Mockery::close();
});

/*
|--------------------------------------------------------------------------
| Test: getAllExamples
|--------------------------------------------------------------------------
*/
describe('getAllExamples', function () {

    it('mengembalikan semua data dari repository', function () {
        $expected = new Collection([
            new Example(['name' => 'Item 1']),
            new Example(['name' => 'Item 2']),
        ]);

        $this->repository
            ->shouldReceive('all')
            ->once()
            ->andReturn($expected);

        $result = $this->service->getAll();

        expect($result)->toBeInstanceOf(Collection::class);
        expect($result)->toHaveCount(2);
    });
});

/*
|--------------------------------------------------------------------------
| Test: createExample
|--------------------------------------------------------------------------
*/
describe('createExample', function () {

    it('berhasil membuat data baru', function () {
        $data = ['name' => 'New Item', 'description' => 'Desc'];
        $expected = new Example($data);

        $this->repository
            ->shouldReceive('create')
            ->once()
            ->with($data)
            ->andReturn($expected);

        $result = $this->service->create($data);

        expect($result)->toBeInstanceOf(Example::class);
        expect($result->name)->toBe('New Item');
    });
});

/*
|--------------------------------------------------------------------------
| Test: Custom Business Logic
|--------------------------------------------------------------------------
*/
describe('calculateSomething (contoh logic khusus)', function () {

    it('menghitung total dengan benar', function () {
        // Contoh: test logic kalkulasi di service
        // Ganti dengan logic bisnis yang sebenarnya

        $result = $this->service->calculateTotal(100, 0.11); // harga + PPN 11%

        expect($result)->toBe(111.0);
    });

    it('menolak input negatif', function () {
        expect(fn () => $this->service->calculateTotal(-100, 0.11))
            ->toThrow(\InvalidArgumentException::class);
    });
});
```

### 4.3 — Kapan menggunakan Unit Test vs Feature Test

| Aspek              | Unit Test                              | Feature Test                           |
|-------------------|---------------------------------------|---------------------------------------|
| **Scope**         | Satu class/method saja                | Full HTTP request cycle               |
| **Database**      | Tidak perlu (gunakan mock)            | Perlu (gunakan `RefreshDatabase`)     |
| **Speed**         | Sangat cepat                          | Lebih lambat                          |
| **Cocok untuk**   | Service, Helper, DTO, Enum logic      | Controller, Middleware, Endpoint      |
| **Dependency**    | Di-mock semua                         | Real (Eloquent, DB, session)          |

> [!TIP]
> **Aturan praktis:**
> - Semua logic di **Service layer** → Unit Test
> - Semua **endpoint / route** → Feature Test
> - Semua **validation** → Feature Test
> - Semua **helper / utility function** → Unit Test
> - Semua **Enum behavior** → Unit Test

---

## BAGIAN E: MENJALANKAN TEST

---

## Step 5 — Perintah-Perintah Test

### 5.1 — Jalankan semua test

// turbo
```bash
php artisan test
```

### 5.2 — Jalankan test spesifik file

// turbo
```bash
php artisan test --filter=ExampleCrudTest
```

### 5.3 — Jalankan test spesifik method/it block

```bash
php artisan test --filter="berhasil menyimpan data valid"
```

### 5.4 — Jalankan hanya Feature Test

// turbo
```bash
php artisan test --testsuite=Feature
```

### 5.5 — Jalankan hanya Unit Test

// turbo
```bash
php artisan test --testsuite=Unit
```

### 5.6 — Jalankan test dengan output verbose

```bash
php artisan test -v
```

### 5.7 — Jalankan test secara parallel (lebih cepat)

```bash
php artisan test --parallel
```

### 5.8 — Jalankan test dengan coverage report

```bash
php artisan test --coverage
```

> [!NOTE]
> Coverage membutuhkan **Xdebug** atau **PCOV** terinstall di PHP.
> Install PCOV (lebih ringan dari Xdebug):
> ```bash
> pecl install pcov
> ```
> Atau tambahkan di `php.ini`:
> ```ini
> extension=pcov
> pcov.enabled=1
> ```

### 5.9 — Coverage dengan minimum threshold

```bash
php artisan test --coverage --min=80
```

Perintah ini akan **gagal** jika coverage di bawah 80%. Berguna untuk CI/CD pipeline.

---

## BAGIAN F: STRATEGI TEST COVERAGE

---

## Step 6 — Fitur yang WAJIB Di-test

> [!WARNING]
> AI agent **WAJIB** memastikan fitur-fitur berikut memiliki test coverage. Jika membuat fitur baru di kategori ini, **HARUS** diikuti dengan test.

### 6.1 — Prioritas Coverage

| Prioritas | Kategori                        | Jenis Test   | Minimum Coverage |
|-----------|--------------------------------|-------------|------------------|
| 🔴 P0    | Autentikasi (login/register/logout) | Feature | 100%            |
| 🔴 P0    | Otorisasi (middleware, policy, gate) | Feature | 100%            |
| 🔴 P0    | CRUD endpoint utama            | Feature      | 90%+             |
| 🟡 P1    | Form validation rules          | Feature      | 90%+             |
| 🟡 P1    | Business logic di Service      | Unit         | 85%+             |
| 🟡 P1    | Payment / transaksi keuangan   | Feature+Unit | 100%             |
| 🟢 P2    | Helper functions               | Unit         | 80%+             |
| 🟢 P2    | Enum behavior                  | Unit         | 80%+             |
| 🔵 P3    | View rendering (non-critical)  | Feature      | Optional         |
| 🔵 P3    | Static pages                   | Feature      | Optional         |

### 6.2 — Test Authentication (WAJIB Ada)

Buat file `tests/Feature/Auth/AuthenticationTest.php`:

```php
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

describe('Login', function () {

    it('menampilkan halaman login', function () {
        $this->get(route('login'))
            ->assertStatus(200);
    });

    it('berhasil login dengan kredensial valid', function () {
        $user = User::factory()->create();

        $this->post(route('login'), [
            'email' => $user->email,
            'password' => 'password', // default password dari UserFactory
        ])->assertRedirect(route('admin.dashboard'));

        $this->assertAuthenticated();
    });

    it('gagal login dengan password salah', function () {
        $user = User::factory()->create();

        $this->post(route('login'), [
            'email' => $user->email,
            'password' => 'wrong-password',
        ]);

        $this->assertGuest();
    });

    it('gagal login dengan email yang tidak terdaftar', function () {
        $this->post(route('login'), [
            'email' => 'nonexistent@example.com',
            'password' => 'password',
        ]);

        $this->assertGuest();
    });
});

describe('Logout', function () {

    it('berhasil logout', function () {
        $user = User::factory()->create();

        $this->actingAs($user)
            ->post(route('logout'))
            ->assertRedirect('/');

        $this->assertGuest();
    });
});

describe('Registration', function () {

    it('menampilkan halaman register', function () {
        $this->get(route('register'))
            ->assertStatus(200);
    });

    it('berhasil register dengan data valid', function () {
        $this->post(route('register'), [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ])->assertRedirect(route('admin.dashboard'));

        $this->assertAuthenticated();
        $this->assertDatabaseHas('users', ['email' => 'test@example.com']);
    });

    it('gagal register dengan email duplikat', function () {
        User::factory()->create(['email' => 'taken@example.com']);

        $this->post(route('register'), [
            'name' => 'Test',
            'email' => 'taken@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
        ])->assertSessionHasErrors(['email']);
    });
});

describe('Protected Routes', function () {

    it('redirect ke login jika mengakses admin tanpa auth', function () {
        $this->get(route('admin.dashboard'))
            ->assertRedirect(route('login'));
    });

    it('bisa mengakses admin setelah login', function () {
        $user = User::factory()->create();

        $this->actingAs($user)
            ->get(route('admin.dashboard'))
            ->assertStatus(200);
    });
});
```

### 6.3 — Test Checklist per Fitur CRUD

Setiap kali membuat fitur CRUD baru, pastikan semua test case berikut **ada dan passing**:

```
✅ INDEX
   ├── [ ] Halaman tampil (200) saat authenticated
   ├── [ ] Redirect ke login saat guest
   └── [ ] Data dari database tampil di halaman

✅ CREATE
   └── [ ] Form create tampil (200)

✅ STORE
   ├── [ ] Berhasil simpan data valid → redirect + flash success
   ├── [ ] Gagal simpan data kosong → validation errors
   ├── [ ] Gagal simpan data duplikat (jika ada unique)
   └── [ ] Data tersimpan di database (assertDatabaseHas)

✅ SHOW
   ├── [ ] Detail tampil (200) untuk ID yang ada
   └── [ ] Return 404 untuk ID yang tidak ada

✅ EDIT
   └── [ ] Form edit tampil (200) dengan data yang benar

✅ UPDATE
   ├── [ ] Berhasil update data valid → redirect + flash success
   ├── [ ] Gagal update data invalid → validation errors
   └── [ ] Data ter-update di database

✅ DELETE
   ├── [ ] Berhasil hapus → redirect + flash success
   ├── [ ] Data terhapus dari database (assertDatabaseMissing)
   └── [ ] Return 404 untuk ID yang tidak ada
```

---

## BAGIAN G: HELPER & BEST PRACTICES

---

## Step 7 — Testing Helper & Utilities

### 7.1 — Test Helper Functions

Jika ada helper di `app/Helpers/helpers.php`, buat test-nya:

```php
<?php
// tests/Unit/Helpers/HelpersTest.php

describe('format_rupiah', function () {
    it('format angka ke rupiah dengan prefix', function () {
        expect(format_rupiah(1500000))->toBe('Rp 1.500.000');
    });

    it('format angka ke rupiah tanpa prefix', function () {
        expect(format_rupiah(1500000, false))->toBe('1.500.000');
    });

    it('format angka nol', function () {
        expect(format_rupiah(0))->toBe('Rp 0');
    });
});

describe('generate_code', function () {
    it('generate kode dengan prefix default', function () {
        $code = generate_code();
        expect($code)->toStartWith('TRX-');
    });

    it('generate kode dengan prefix kustom', function () {
        $code = generate_code('INV');
        expect($code)->toStartWith('INV-');
    });

    it('generate kode unik setiap kali dipanggil', function () {
        $code1 = generate_code();
        $code2 = generate_code();
        expect($code1)->not->toBe($code2);
    });
});
```

### 7.2 — Test Enum

```php
<?php
// tests/Unit/Enums/StatusEnumTest.php

use App\Enums\StatusEnum;

describe('StatusEnum', function () {
    it('memiliki value yang benar', function () {
        expect(StatusEnum::ACTIVE->value)->toBe('active');
        expect(StatusEnum::INACTIVE->value)->toBe('inactive');
        expect(StatusEnum::PENDING->value)->toBe('pending');
    });

    it('mengembalikan label yang benar', function () {
        expect(StatusEnum::ACTIVE->label())->toBe('Aktif');
        expect(StatusEnum::INACTIVE->label())->toBe('Tidak Aktif');
        expect(StatusEnum::PENDING->label())->toBe('Menunggu');
    });

    it('bisa dibuat dari string value', function () {
        $status = StatusEnum::from('active');
        expect($status)->toBe(StatusEnum::ACTIVE);
    });

    it('throw error untuk value yang tidak valid', function () {
        expect(fn () => StatusEnum::from('invalid'))
            ->toThrow(\ValueError::class);
    });
});
```

---

## Step 8 — Common Testing Patterns & Tips

### 8.1 — Assertion yang sering digunakan

```php
// HTTP Response
$response->assertStatus(200);
$response->assertOk();                    // alias 200
$response->assertCreated();               // 201
$response->assertNoContent();             // 204
$response->assertNotFound();              // 404
$response->assertForbidden();             // 403
$response->assertUnauthorized();          // 401
$response->assertRedirect('/path');
$response->assertRedirectToRoute('route.name');

// View
$response->assertViewIs('view.name');
$response->assertViewHas('key');
$response->assertViewHas('key', 'value');
$response->assertSee('text');
$response->assertDontSee('text');

// Session
$response->assertSessionHas('key');
$response->assertSessionHas('success');
$response->assertSessionHasErrors(['field']);
$response->assertSessionDoesntHaveErrors();

// Database
$this->assertDatabaseHas('table', ['column' => 'value']);
$this->assertDatabaseMissing('table', ['column' => 'value']);
$this->assertDatabaseCount('table', 5);

// Auth
$this->assertAuthenticated();
$this->assertGuest();
$this->assertAuthenticatedAs($user);

// Pest expectations
expect($value)->toBe('exact');
expect($value)->toEqual('loose');
expect($value)->toBeTrue();
expect($value)->toBeFalse();
expect($value)->toBeNull();
expect($value)->not->toBeNull();
expect($value)->toBeInstanceOf(SomeClass::class);
expect($value)->toHaveCount(3);
expect($value)->toContain('item');
expect($value)->toStartWith('prefix');
expect($value)->toMatchArray(['key' => 'value']);
expect(fn() => something())->toThrow(Exception::class);
```

### 8.2 — Factory Tips

Pastikan setiap Model yang di-test memiliki Factory:

```bash
php artisan make:factory ExampleFactory --model=Example
```

Contoh factory:

```php
<?php

namespace Database\Factories;

use App\Models\Example;
use Illuminate\Database\Eloquent\Factories\Factory;

class ExampleFactory extends Factory
{
    protected $model = Example::class;

    public function definition(): array
    {
        return [
            'name' => fake()->sentence(3),
            'description' => fake()->paragraph(),
            'status' => fake()->randomElement(['active', 'inactive', 'pending']),
            'created_at' => fake()->dateTimeBetween('-1 year', 'now'),
        ];
    }

    // State methods untuk variasi data
    public function active(): static
    {
        return $this->state(fn () => ['status' => 'active']);
    }

    public function inactive(): static
    {
        return $this->state(fn () => ['status' => 'inactive']);
    }
}
```

---

## Checklist Verifikasi Akhir

- [ ] Laravel Debugbar terinstall (`--dev` only) dan muncul di browser
- [ ] `config/debugbar.php` — collectors dikonfigurasi (terutama `db`, `models`)
- [ ] `.env.testing` dibuat dengan **database terpisah**
- [ ] `phpunit.xml` dikonfigurasi sesuai environment testing
- [ ] Database testing sudah dibuat (`laravel_db_test`)
- [ ] `php artisan test` berjalan tanpa error
- [ ] Feature Test autentikasi (login/register/logout) **ada dan passing**
- [ ] Feature Test CRUD template tersedia dan bisa diadaptasi
- [ ] Unit Test template untuk Service layer tersedia
- [ ] Unit Test untuk Helper functions tersedia
- [ ] Unit Test untuk Enum tersedia
- [ ] Factory tersedia untuk setiap Model yang di-test
- [ ] Coverage bisa dijalankan (`php artisan test --coverage`)
- [ ] Fitur P0 (auth, otorisasi) memiliki coverage 100%

---

## Ringkasan Struktur Test

```
tests/
├── Feature/
│   ├── Auth/
│   │   └── AuthenticationTest.php         # Login, Register, Logout, Protected Routes
│   ├── Admin/
│   │   └── ExampleCrudTest.php            # Template CRUD test (copy per fitur)
│   └── ...
├── Unit/
│   ├── Services/
│   │   └── ExampleServiceTest.php         # Unit test Service layer
│   ├── Helpers/
│   │   └── HelpersTest.php                # Test helper functions
│   ├── Enums/
│   │   └── StatusEnumTest.php             # Test Enum behavior
│   └── ...
├── Pest.php                               # Pest configuration
└── TestCase.php                           # Base test case
```

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **WAJIB buat test** untuk setiap fitur CRUD baru. Tidak boleh skip testing.
> 2. **Fitur autentikasi dan otorisasi** harus memiliki test coverage **100%**. Ini non-negotiable.
> 3. **Selalu gunakan `RefreshDatabase`** di Feature Test agar database bersih setiap test.
> 4. **Selalu gunakan Factory** untuk membuat data test, jangan insert manual.
> 5. **Jalankan `php artisan test`** setelah setiap perubahan signifikan. Jika ada test gagal, **perbaiki sebelum lanjut**.
> 6. **Debugbar = development only**. Pastikan `DEBUGBAR_ENABLED=false` di `.env.production`.
> 7. **Gunakan database terpisah** untuk testing. Jangan pernah jalankan test di database development.
> 8. **Test tidak hanya happy path**. Test juga error cases: validation error, 404, unauthorized, duplikat data.
