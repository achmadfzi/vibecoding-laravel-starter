---
description: Panduan AI untuk membuat Custom Artisan Command, script cronjob, seeder data dummy, dan penggunaan Laravel Tinker untuk debugging database.
---

# 08 — Artisan Playground

> **Tujuan**: Memberikan panduan lengkap untuk membuat custom Artisan command (cronjob, seeder khusus, utility script) dan menggunakan Laravel Tinker sebagai alat debugging cepat. Dokumen ini adalah "ruang eksperimen" bagi developer dan AI agent.

---

## Prasyarat

- Project Laravel sudah berjalan dan database terhubung
- Artisan CLI bisa dijalankan (`php artisan --version`)

---

## BAGIAN A: CUSTOM ARTISAN COMMAND

---

## Step 1 — Membuat Custom Command

### 1.1 — Generate command baru

// turbo
```bash
php artisan make:command NamaCommand
```

**Contoh:**

```bash
php artisan make:command CleanExpiredTokens
php artisan make:command GenerateDummyData
php artisan make:command SendMonthlyReport
php artisan make:command SyncExternalData
```

File akan dibuat di `app/Console/Commands/NamaCommand.php`.

### 1.2 — Anatomi Artisan Command

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;

/**
 * Deskripsi command: apa yang dilakukan.
 *
 * Usage:
 *   php artisan app:nama-command
 *   php artisan app:nama-command --option=value
 */
class NamaCommand extends Command
{
    /**
     * Nama dan signature command.
     *
     * Format: 'prefix:nama {argument} {--option}'
     *
     * @var string
     */
    protected $signature = 'app:nama-command';

    /**
     * Deskripsi yang muncul di `php artisan list`.
     *
     * @var string
     */
    protected $description = 'Deskripsi singkat command ini';

    /**
     * Eksekusi command.
     *
     * @return int  0 = success, 1 = failure
     */
    public function handle(): int
    {
        $this->info('Command berjalan...');

        // Logic di sini

        $this->info('✅ Selesai.');

        return Command::SUCCESS; // atau self::SUCCESS
    }
}
```

### 1.3 — Signature: Arguments & Options

```php
// Argument wajib
protected $signature = 'app:greet {name}';

// Argument opsional dengan default
protected $signature = 'app:greet {name=World}';

// Option (flag boolean)
protected $signature = 'app:cleanup {--force}';

// Option dengan value
protected $signature = 'app:export {--format=csv}';

// Option dengan shortcut
protected $signature = 'app:export {--F|format=csv}';

// Kombinasi
protected $signature = 'app:seed-data {model} {--count=10} {--fresh}';
```

Cara mengambil value:

```php
public function handle(): int
{
    $name   = $this->argument('name');       // Argument
    $format = $this->option('format');       // Option
    $force  = $this->option('force');        // Boolean option (true/false)

    return self::SUCCESS;
}
```

### 1.4 — Output & Interaksi

```php
public function handle(): int
{
    // Output berwarna
    $this->info('ℹ️  Informasi biasa');          // Hijau
    $this->warn('⚠️  Peringatan');               // Kuning
    $this->error('❌ Error terjadi');             // Merah
    $this->line('Teks biasa tanpa warna');
    $this->newLine(2);                           // 2 baris kosong

    // Progress bar
    $items = range(1, 100);
    $bar = $this->output->createProgressBar(count($items));
    $bar->start();
    foreach ($items as $item) {
        // Proses...
        $bar->advance();
    }
    $bar->finish();
    $this->newLine();

    // Konfirmasi
    if ($this->confirm('Lanjutkan proses?', true)) {
        $this->info('Melanjutkan...');
    }

    // Input dari user
    $email = $this->ask('Masukkan email:');
    $password = $this->secret('Masukkan password:');

    // Pilihan
    $role = $this->choice('Pilih role:', ['admin', 'operator', 'user'], 0);

    // Tabel output
    $this->table(
        ['ID', 'Name', 'Email'],
        \App\Models\User::all(['id', 'name', 'email'])->toArray()
    );

    return self::SUCCESS;
}
```

---

## Step 2 — Template: Cronjob Command

### 2.1 — Contoh: Hapus token kadaluarsa

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Carbon\Carbon;

/**
 * Menghapus personal access token yang sudah kadaluarsa.
 *
 * Dijalankan setiap hari via scheduler.
 *
 * Usage:
 *   php artisan app:clean-expired-tokens
 *   php artisan app:clean-expired-tokens --days=60
 */
class CleanExpiredTokens extends Command
{
    protected $signature = 'app:clean-expired-tokens {--days=30 : Hapus token lebih lama dari N hari}';

    protected $description = 'Hapus personal access token yang sudah kadaluarsa';

    public function handle(): int
    {
        $days = (int) $this->option('days');
        $cutoff = Carbon::now()->subDays($days);

        $this->info("Menghapus token lebih lama dari {$days} hari ({$cutoff->toDateString()})...");

        $deleted = DB::table('personal_access_tokens')
            ->where('last_used_at', '<', $cutoff)
            ->orWhereNull('last_used_at')
            ->where('created_at', '<', $cutoff)
            ->delete();

        $this->info("✅ {$deleted} token berhasil dihapus.");

        return self::SUCCESS;
    }
}
```

### 2.2 — Contoh: Kirim laporan bulanan

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\User;
use Illuminate\Support\Facades\Mail;
use Illuminate\Support\Facades\Log;

/**
 * Mengirim email laporan bulanan ke semua admin.
 *
 * Dijalankan setiap tanggal 1 via scheduler.
 *
 * Usage:
 *   php artisan app:send-monthly-report
 *   php artisan app:send-monthly-report --test=admin@admin.com
 */
class SendMonthlyReport extends Command
{
    protected $signature = 'app:send-monthly-report {--test= : Kirim ke satu email saja (untuk testing)}';

    protected $description = 'Kirim laporan bulanan ke admin via email';

    public function handle(): int
    {
        $testEmail = $this->option('test');

        if ($testEmail) {
            $this->info("Mode test: mengirim ke {$testEmail}");
            $recipients = collect([['email' => $testEmail]]);
        } else {
            $recipients = User::role('admin')->get();
        }

        $bar = $this->output->createProgressBar($recipients->count());
        $bar->start();

        $sent = 0;
        $failed = 0;

        foreach ($recipients as $recipient) {
            try {
                // Mail::to($recipient->email)->send(new MonthlyReportMail());
                $sent++;
            } catch (\Exception $e) {
                Log::error("Gagal kirim report ke {$recipient->email}: {$e->getMessage()}");
                $failed++;
            }
            $bar->advance();
        }

        $bar->finish();
        $this->newLine(2);

        $this->info("✅ Berhasil kirim: {$sent}");
        if ($failed > 0) {
            $this->warn("⚠️  Gagal kirim: {$failed} (cek log)");
        }

        return $failed > 0 ? self::FAILURE : self::SUCCESS;
    }
}
```

### 2.3 — Contoh: Sync data dari API external

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;
use App\Models\Product;

/**
 * Sinkronisasi data produk dari API external.
 *
 * Dijalankan setiap 6 jam via scheduler.
 *
 * Usage:
 *   php artisan app:sync-products
 *   php artisan app:sync-products --dry-run
 */
class SyncExternalProducts extends Command
{
    protected $signature = 'app:sync-products {--dry-run : Tampilkan data tanpa menyimpan}';

    protected $description = 'Sync data produk dari API external';

    public function handle(): int
    {
        $dryRun = $this->option('dry-run');

        $this->info('Mengambil data dari API...');

        try {
            $response = Http::timeout(30)
                ->get('https://api.example.com/products');

            if ($response->failed()) {
                $this->error("❌ API error: {$response->status()}");
                return self::FAILURE;
            }

            $products = $response->json('data');
            $this->info("Ditemukan " . count($products) . " produk.");

            if ($dryRun) {
                $this->table(
                    ['ID', 'Name', 'Price'],
                    collect($products)->map(fn ($p) => [
                        $p['id'], $p['name'], $p['price']
                    ])->toArray()
                );
                $this->warn('⚠️  Dry run — tidak ada data yang disimpan.');
                return self::SUCCESS;
            }

            $bar = $this->output->createProgressBar(count($products));
            $bar->start();

            $created = 0;
            $updated = 0;

            foreach ($products as $data) {
                $product = Product::updateOrCreate(
                    ['external_id' => $data['id']],
                    [
                        'name'  => $data['name'],
                        'price' => $data['price'],
                    ]
                );

                $product->wasRecentlyCreated ? $created++ : $updated++;
                $bar->advance();
            }

            $bar->finish();
            $this->newLine(2);
            $this->info("✅ Created: {$created}, Updated: {$updated}");

            return self::SUCCESS;

        } catch (\Exception $e) {
            $this->error("❌ Exception: {$e->getMessage()}");
            Log::error('Sync products failed', ['error' => $e->getMessage()]);
            return self::FAILURE;
        }
    }
}
```

---

## Step 3 — Template: Seeder Data Dummy via Command

### 3.1 — Command untuk generate data dummy spesifik

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use App\Models\User;
use App\Models\Product;
use Illuminate\Support\Facades\Hash;

/**
 * Generate data dummy untuk testing dan development.
 *
 * Berbeda dari DatabaseSeeder, command ini bisa dijalankan kapanpun
 * tanpa mereset database, dan jumlahnya bisa dikustomisasi.
 *
 * Usage:
 *   php artisan app:seed-dummy users --count=50
 *   php artisan app:seed-dummy products --count=100
 *   php artisan app:seed-dummy all --count=25
 */
class SeedDummyData extends Command
{
    protected $signature = 'app:seed-dummy
                            {model : Model yang akan di-seed (users, products, all)}
                            {--count=10 : Jumlah data yang di-generate}
                            {--fresh : Hapus data existing sebelum seed}';

    protected $description = 'Generate data dummy untuk development';

    /**
     * Model yang didukung dan Factory-nya.
     *
     * @var array<string, class-string>
     */
    private array $supportedModels = [
        'users'    => User::class,
        'products' => Product::class,
        // Tambahkan model lain di sini
    ];

    public function handle(): int
    {
        $model = strtolower($this->argument('model'));
        $count = (int) $this->option('count');
        $fresh = $this->option('fresh');

        // Proteksi production
        if (app()->environment('production')) {
            $this->error('❌ Command ini tidak boleh dijalankan di production!');
            return self::FAILURE;
        }

        if ($model === 'all') {
            return $this->seedAll($count, $fresh);
        }

        if (!array_key_exists($model, $this->supportedModels)) {
            $this->error("❌ Model '{$model}' tidak didukung.");
            $this->info('Model yang tersedia: ' . implode(', ', array_keys($this->supportedModels)));
            return self::FAILURE;
        }

        return $this->seedModel($model, $count, $fresh);
    }

    /**
     * Seed satu model.
     */
    private function seedModel(string $key, int $count, bool $fresh): int
    {
        $modelClass = $this->supportedModels[$key];

        if ($fresh) {
            if (!$this->confirm("⚠️ Hapus semua data {$key} existing?")) {
                $this->info('Dibatalkan.');
                return self::SUCCESS;
            }
            $modelClass::truncate();
            $this->warn("🗑️  Data {$key} dihapus.");
        }

        $this->info("Generating {$count} {$key}...");

        $bar = $this->output->createProgressBar($count);
        $bar->start();

        $modelClass::factory()
            ->count($count)
            ->create()
            ->each(function () use ($bar) {
                $bar->advance();
            });

        // Alternatif tanpa progress bar per-item:
        // $modelClass::factory()->count($count)->create();

        $bar->finish();
        $this->newLine();
        $this->info("✅ {$count} {$key} berhasil dibuat.");

        return self::SUCCESS;
    }

    /**
     * Seed semua model.
     */
    private function seedAll(int $count, bool $fresh): int
    {
        foreach (array_keys($this->supportedModels) as $key) {
            $result = $this->seedModel($key, $count, $fresh);
            if ($result !== self::SUCCESS) {
                return $result;
            }
            $this->newLine();
        }

        $this->info('🎉 Semua data dummy berhasil di-generate!');
        return self::SUCCESS;
    }
}
```

### 3.2 — Contoh menjalankan

```bash
# Seed 50 users
php artisan app:seed-dummy users --count=50

# Seed 100 products, hapus data lama dulu
php artisan app:seed-dummy products --count=100 --fresh

# Seed semua model, masing-masing 25
php artisan app:seed-dummy all --count=25
```

---

## Step 4 — Mendaftarkan Command ke Scheduler

### 4.1 — Laravel 11+ (routes/console.php)

Buka `routes/console.php`:

```php
<?php

use Illuminate\Support\Facades\Schedule;

/*
|--------------------------------------------------------------------------
| Console Routes / Scheduled Tasks
|--------------------------------------------------------------------------
|
| Definisikan scheduled command di sini.
| Pastikan cron `* * * * * php artisan schedule:run` sudah aktif di server.
|
*/

// Hapus token kadaluarsa — setiap hari jam 02:00
Schedule::command('app:clean-expired-tokens --days=30')
    ->dailyAt('02:00')
    ->timezone('Asia/Jakarta')
    ->withoutOverlapping()
    ->onOneServer()                          // Jika multi-server
    ->appendOutputTo(storage_path('logs/scheduler.log'));

// Kirim laporan bulanan — tanggal 1, jam 08:00
Schedule::command('app:send-monthly-report')
    ->monthlyOn(1, '08:00')
    ->timezone('Asia/Jakarta')
    ->appendOutputTo(storage_path('logs/scheduler.log'));

// Sync produk dari API — setiap 6 jam
Schedule::command('app:sync-products')
    ->everySixHours()
    ->withoutOverlapping()
    ->appendOutputTo(storage_path('logs/scheduler.log'));
```

### 4.2 — Laravel 10 (app/Console/Kernel.php)

```php
protected function schedule(Schedule $schedule): void
{
    $schedule->command('app:clean-expired-tokens --days=30')
        ->dailyAt('02:00')
        ->timezone('Asia/Jakarta');

    $schedule->command('app:send-monthly-report')
        ->monthlyOn(1, '08:00')
        ->timezone('Asia/Jakarta');
}
```

### 4.3 — Jadwal yang sering digunakan

| Method | Jadwal |
|--------|--------|
| `->everyMinute()` | Setiap menit |
| `->everyFiveMinutes()` | Setiap 5 menit |
| `->everyFifteenMinutes()` | Setiap 15 menit |
| `->everyThirtyMinutes()` | Setiap 30 menit |
| `->hourly()` | Setiap jam |
| `->everySixHours()` | Setiap 6 jam |
| `->daily()` | Setiap hari jam 00:00 |
| `->dailyAt('13:00')` | Setiap hari jam 13:00 |
| `->weekly()` | Setiap minggu (Senin 00:00) |
| `->weeklyOn(1, '08:00')` | Senin jam 08:00 |
| `->monthly()` | Setiap bulan tanggal 1, 00:00 |
| `->monthlyOn(1, '08:00')` | Tanggal 1, jam 08:00 |
| `->quarterly()` | Setiap 3 bulan |
| `->yearly()` | Setiap tahun |

### 4.4 — Options penting

```php
Schedule::command('app:nama')
    ->daily()
    ->timezone('Asia/Jakarta')           // Timezone lokal
    ->withoutOverlapping()               // Cegah running bersamaan
    ->onOneServer()                      // Hanya 1 server jika multi-server
    ->runInBackground()                  // Tidak blocking scheduler lain
    ->appendOutputTo('logs/cron.log')    // Log output ke file
    ->emailOutputOnFailure('dev@app.com') // Email jika gagal
    ->before(function () {               // Hook sebelum command jalan
        Log::info('Starting scheduled task...');
    })
    ->after(function () {                // Hook setelah command selesai
        Log::info('Scheduled task completed.');
    });
```

### 4.5 — Verifikasi scheduled command

// turbo
```bash
php artisan schedule:list
```

// turbo
```bash
php artisan schedule:test
```

> `schedule:test` memungkinkan Anda memilih dan menjalankan satu scheduled command secara manual untuk testing.

---

## BAGIAN B: LARAVEL TINKER

---

## Step 5 — Dasar Penggunaan Tinker

### 5.1 — Masuk ke Tinker

// turbo
```bash
php artisan tinker
```

Tinker adalah REPL (Read-Eval-Print Loop) yang memungkinkan interaksi langsung dengan aplikasi Laravel.

### 5.2 — Keluar dari Tinker

```php
exit
// atau
quit
// atau tekan Ctrl+C
```

---

## Step 6 — Operasi Database Cepat via Tinker

### 6.1 — Query data

```php
// Ambil semua user
User::all();

// Ambil user pertama
User::first();

// Cari user berdasarkan ID
User::find(1);

// Cari user berdasarkan email
User::where('email', 'admin@admin.com')->first();

// Ambil 5 user terbaru
User::latest()->take(5)->get();

// Ambil user dengan role-nya (eager loading)
User::with('roles')->find(1);

// Count
User::count();
User::where('email', 'like', '%@admin.com')->count();

// Pluck (ambil kolom tertentu saja)
User::pluck('email');
User::pluck('name', 'id'); // ['id' => 'name']
```

### 6.2 — Create data

```php
// Buat user baru
$user = User::create([
    'name' => 'Test User',
    'email' => 'test@example.com',
    'password' => bcrypt('password'),
]);

// Buat user via factory
User::factory()->create();

// Buat 10 user via factory
User::factory()->count(10)->create();

// Buat user dengan role
$user = User::create([
    'name' => 'Admin Baru',
    'email' => 'newadmin@admin.com',
    'password' => bcrypt('password'),
]);
$user->assignRole('admin');
```

### 6.3 — Update data

```php
// Update satu user
$user = User::find(1);
$user->update(['name' => 'Nama Baru']);

// Update massal
User::where('email', 'like', '%@test.com')->update(['name' => 'Test Account']);

// Toggle status
$user = User::find(1);
$user->is_active = !$user->is_active;
$user->save();
```

### 6.4 — Delete data

```php
// Hapus satu user
User::find(5)->delete();

// Hapus massal
User::where('email', 'like', '%@test.com')->delete();

// Soft delete (jika model pakai SoftDeletes)
User::find(5)->delete();          // Soft delete
User::withTrashed()->find(5);     // Lihat termasuk yang soft-deleted
User::onlyTrashed()->get();       // Hanya yang soft-deleted
User::find(5)->forceDelete();     // Hapus permanen
```

### 6.5 — Relasi dan role (Spatie Permission)

```php
// Lihat role user
$user = User::find(1);
$user->roles->pluck('name');

// Lihat permission user
$user->getAllPermissions()->pluck('name');

// Assign role
$user->assignRole('admin');

// Remove role
$user->removeRole('admin');

// Sync roles (hapus lama, set baru)
$user->syncRoles(['admin', 'operator']);

// Cek role
$user->hasRole('admin');              // true/false
$user->hasAnyRole(['admin', 'operator']); // true/false

// Lihat semua role
\Spatie\Permission\Models\Role::pluck('name');

// Lihat semua permission
\Spatie\Permission\Models\Permission::pluck('name');

// Lihat permission dari satu role
$role = \Spatie\Permission\Models\Role::findByName('admin');
$role->permissions->pluck('name');
```

---

## Step 7 — Debugging dengan Tinker

### 7.1 — Cek environment & config

```php
// Cek environment
app()->environment();                    // "local", "production", etc.

// Cek config
config('app.timezone');
config('database.default');
config('mail.default');
config('permission.cache.expiration_time');

// Cek .env value (HANYA jika config belum di-cache)
env('APP_DEBUG');
env('DB_DATABASE');
```

### 7.2 — Test query dan debug SQL

```php
// Lihat SQL query yang akan dijalankan (tanpa execute)
User::where('email', 'like', '%admin%')->toSql();
// Output: "select * from `users` where `email` like ?"

// Lihat query dengan binding
User::where('email', 'like', '%admin%')->toRawSql();
// Output: "select * from `users` where `email` like '%admin%'"

// Enable query log
DB::enableQueryLog();
User::with('roles')->where('id', '<', 10)->get();
DB::getQueryLog();
// Output: array of executed queries with time
```

### 7.3 — Test helper & service

```php
// Test route
route('admin.dashboard');                // "/admin/dashboard"
route('admin.users.edit', ['user' => 1]); // "/admin/users/1/edit"

// Test date/time
now();                                    // Current datetime
now()->format('d M Y H:i');
now()->subDays(30);
now()->diffForHumans(now()->subHours(3)); // "3 hours after"

// Test string helper
\Illuminate\Support\Str::slug('Hello World');   // "hello-world"
\Illuminate\Support\Str::random(32);
\Illuminate\Support\Str::uuid();

// Test path
storage_path('app/public');
public_path('build');
base_path('.env');
```

### 7.4 — Test cache

```php
// Set cache
cache()->put('test_key', 'test_value', now()->addMinutes(5));

// Get cache
cache()->get('test_key');

// Forget cache
cache()->forget('test_key');

// Clear all cache
cache()->flush();

// Check if cached
cache()->has('test_key');
```

### 7.5 — Test mail (tanpa benar-benar kirim)

```php
// Preview mail di browser
// Tambahkan route sementara di web.php:
// Route::get('/mail-preview', fn() => new \App\Mail\YourMailable());

// Atau via Tinker, cek mail object
$mail = new \App\Mail\WelcomeMail($user);
$mail->render(); // Output HTML
```

---

## Step 8 — One-Liner Tinker (Tanpa Masuk REPL)

Untuk eksekusi cepat tanpa masuk ke REPL, gunakan `--execute`:

```bash
# Hitung user
php artisan tinker --execute="echo User::count();"

# Lihat semua role
php artisan tinker --execute="echo \Spatie\Permission\Models\Role::pluck('name');"

# Cek environment
php artisan tinker --execute="echo app()->environment();"

# Reset password user tertentu
php artisan tinker --execute="User::where('email','admin@admin.com')->update(['password'=>bcrypt('newpassword')]);"

# Buat user cepat
php artisan tinker --execute="\$u = User::create(['name'=>'Quick User','email'=>'quick@test.com','password'=>bcrypt('password')]); \$u->assignRole('admin'); echo \$u->id;"

# Cek config
php artisan tinker --execute="echo config('app.timezone');"

# Lihat 5 user terakhir
php artisan tinker --execute="echo User::latest()->take(5)->get(['id','name','email'])->toJson(JSON_PRETTY_PRINT);"
```

> [!TIP]
> `--execute` sangat berguna untuk scripting dan CI/CD. Anda bisa menjalankan operasi database tanpa masuk ke REPL interaktif.

---

## BAGIAN C: UTILITY COMMANDS

---

## Step 9 — Template: Database Maintenance Commands

### 9.1 — Backup database via command

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Process;
use Carbon\Carbon;

/**
 * Backup database MySQL ke file SQL.
 *
 * Usage:
 *   php artisan app:db-backup
 *   php artisan app:db-backup --compress
 */
class DatabaseBackup extends Command
{
    protected $signature = 'app:db-backup {--compress : Compress hasil backup ke .gz}';

    protected $description = 'Backup database ke file SQL';

    public function handle(): int
    {
        $database = config('database.connections.mysql.database');
        $username = config('database.connections.mysql.username');
        $password = config('database.connections.mysql.password');
        $host     = config('database.connections.mysql.host');
        $port     = config('database.connections.mysql.port');

        $timestamp = Carbon::now()->format('Y-m-d_His');
        $filename  = "backup_{$database}_{$timestamp}.sql";
        $filepath  = storage_path("app/backups/{$filename}");

        // Buat folder backups jika belum ada
        if (!is_dir(dirname($filepath))) {
            mkdir(dirname($filepath), 0755, true);
        }

        $this->info("Backing up database: {$database}");

        $command = sprintf(
            'mysqldump -h%s -P%s -u%s -p%s %s > %s',
            $host, $port, $username, $password, $database, $filepath
        );

        $result = Process::run($command);

        if ($result->failed()) {
            $this->error("❌ Backup gagal: {$result->errorOutput()}");
            return self::FAILURE;
        }

        // Compress jika diminta
        if ($this->option('compress')) {
            Process::run("gzip {$filepath}");
            $filepath .= '.gz';
        }

        $size = $this->formatBytes(filesize($filepath));
        $this->info("✅ Backup berhasil: {$filepath} ({$size})");

        return self::SUCCESS;
    }

    /**
     * Format byte ke human-readable.
     */
    private function formatBytes(int $bytes): string
    {
        $units = ['B', 'KB', 'MB', 'GB'];
        $i = 0;
        while ($bytes >= 1024 && $i < count($units) - 1) {
            $bytes /= 1024;
            $i++;
        }
        return round($bytes, 2) . ' ' . $units[$i];
    }
}
```

### 9.2 — Health check command

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Http;

/**
 * Cek kesehatan aplikasi — database, cache, storage, dan queue.
 *
 * Usage:
 *   php artisan app:health-check
 */
class HealthCheck extends Command
{
    protected $signature = 'app:health-check';

    protected $description = 'Cek kesehatan sistem (DB, cache, storage, queue)';

    public function handle(): int
    {
        $results = [];

        // 1. Database
        try {
            DB::connection()->getPDO();
            $results[] = ['Database', '✅ Connected', config('database.default')];
        } catch (\Exception $e) {
            $results[] = ['Database', '❌ Failed', $e->getMessage()];
        }

        // 2. Cache
        try {
            Cache::put('health_check', true, 10);
            $value = Cache::get('health_check');
            Cache::forget('health_check');
            $results[] = ['Cache', $value ? '✅ Working' : '❌ Failed', config('cache.default')];
        } catch (\Exception $e) {
            $results[] = ['Cache', '❌ Failed', $e->getMessage()];
        }

        // 3. Storage writable
        $storagePath = storage_path('logs');
        $results[] = [
            'Storage',
            is_writable($storagePath) ? '✅ Writable' : '❌ Not Writable',
            $storagePath
        ];

        // 4. Storage symlink
        $symlinkPath = public_path('storage');
        $results[] = [
            'Storage Link',
            is_link($symlinkPath) ? '✅ Exists' : '⚠️ Missing',
            $symlinkPath
        ];

        // 5. APP_DEBUG
        $debug = config('app.debug');
        $results[] = [
            'APP_DEBUG',
            $debug ? '⚠️ true (matikan di production!)' : '✅ false',
            config('app.env')
        ];

        // 6. APP_KEY
        $key = config('app.key');
        $results[] = [
            'APP_KEY',
            $key ? '✅ Set' : '❌ Missing',
            $key ? substr($key, 0, 12) . '...' : 'NOT SET'
        ];

        // Output tabel
        $this->table(['Check', 'Status', 'Detail'], $results);

        $hasFailure = collect($results)->contains(fn ($r) => str_contains($r[1], '❌'));

        if ($hasFailure) {
            $this->error('❌ Ada masalah yang perlu diperbaiki!');
            return self::FAILURE;
        }

        $this->info('✅ Semua pengecekan OK!');
        return self::SUCCESS;
    }
}
```

---

## Step 10 — Artisan Commands Cheatsheet

### 10.1 — Command yang sering digunakan

| Perintah | Fungsi |
|----------|--------|
| `php artisan list` | Daftar semua command yang tersedia |
| `php artisan help <command>` | Help untuk command tertentu |
| `php artisan make:model Nama -mfcs` | Model + Migration + Factory + Controller + Seeder |
| `php artisan make:controller Api/NamaController --api` | Controller API (tanpa create/edit) |
| `php artisan make:resource NamaResource` | API Resource transformer |
| `php artisan make:request NamaRequest` | Form Request validation |
| `php artisan make:middleware NamaMiddleware` | Custom middleware |
| `php artisan make:mail NamaMail --markdown=emails.nama` | Mailable dengan Markdown template |
| `php artisan make:notification NamaNotification` | Notification class |
| `php artisan make:event NamaEvent` | Event class |
| `php artisan make:listener NamaListener --event=NamaEvent` | Event listener |
| `php artisan make:job NamaJob` | Queue job |
| `php artisan make:policy NamaPolicy --model=Nama` | Authorization policy |
| `php artisan make:rule NamaRule` | Custom validation rule |
| `php artisan make:enum NamaEnum` | Enum class (PHP 8.1+) |

### 10.2 — Shortcut flag untuk `make:model`

```bash
# Model + Migration
php artisan make:model Product -m

# Model + Migration + Factory + Controller + Seeder
php artisan make:model Product -mfcs

# Model + Migration + Factory + Resource Controller + Seeder + FormRequest + Policy
php artisan make:model Product --all
# Sama dengan:
php artisan make:model Product -mfcsrp
```

| Flag | Generates |
|------|-----------|
| `-m` | Migration |
| `-f` | Factory |
| `-c` | Controller |
| `-s` | Seeder |
| `-r` | Resource Controller (with all 7 methods) |
| `-p` | Policy |
| `-R` | Form Requests (StoreRequest + UpdateRequest) |
| `--all` | Semua di atas |
| `--api` | API Controller (tanpa create/edit) |
| `--pivot` | Pivot model (tanpa timestamps) |

---

## Checklist Verifikasi

- [ ] Custom command bisa dibuat via `php artisan make:command`
- [ ] Command terdaftar di `php artisan list` (namespace `app:`)
- [ ] Command bisa dijalankan manual via terminal
- [ ] Scheduler terdaftar di `php artisan schedule:list`
- [ ] Tinker bisa dijalankan (`php artisan tinker`)
- [ ] Operasi CRUD via Tinker berfungsi
- [ ] One-liner `--execute` berfungsi
- [ ] Proteksi production ada di command yang destructive (cek `app()->environment()`)
- [ ] Command punya PHPDoc sesuai standar workflow 07

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **Semua custom command HARUS menggunakan prefix `app:`** pada signature (misal `app:clean-tokens`, bukan `clean-tokens`). Ini mencegah konflik dengan command Laravel/package bawaan.
> 2. **Command destructive WAJIB punya proteksi production:**
>    ```php
>    if (app()->environment('production')) {
>        $this->error('Tidak boleh dijalankan di production!');
>        return self::FAILURE;
>    }
>    ```
> 3. **Selalu return exit code**: `self::SUCCESS` (0) untuk sukses, `self::FAILURE` (1) untuk gagal. Ini penting untuk scheduler dan CI/CD.
> 4. **Gunakan progress bar** untuk command yang memproses banyak data. Ini membantu monitoring dan UX.
> 5. **Log output** dari scheduled command ke file menggunakan `->appendOutputTo()`. Jangan hanya mengandalkan `$this->info()`.
> 6. **Tinker bukan untuk production.** Jangan mengeksekusi operasi berbahaya (delete, truncate) via Tinker di server production tanpa konfirmasi user terlebih dahulu.
> 7. **Gunakan `--execute`** untuk operasi one-liner yang bisa diotomasi, tapi **hati-hati** dengan quoting di shell. Selalu wrap dalam single quotes jika ada variabel PHP (`\$`).
> 8. **Tanyakan ke user** apakah mereka butuh cronjob/scheduled task sebelum membuat command. Jangan buat command yang tidak akan digunakan.
