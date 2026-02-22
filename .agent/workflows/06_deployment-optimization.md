---
description: SOP untuk mempersiapkan aplikasi Laravel sebelum deploy ke production — termasuk optimasi artisan, konfigurasi .env, storage symlink, queue worker, dan checklist keamanan.
---

# 06 — Deployment & Optimization

> **Tujuan**: Mempersiapkan aplikasi Laravel agar siap di-deploy ke server production (VPS, Railway, Laravel Forge, dll). Termasuk perintah optimasi Artisan, konfigurasi `.env` production, setup storage, queue worker, dan checklist keamanan sebelum go-live.

---

## ⛔ ATURAN MUTLAK

> [!CAUTION]
> **JANGAN PERNAH menjalankan perintah berikut di server production:**
> ```bash
> php artisan migrate:fresh        # ← MENGHAPUS SEMUA TABEL + DATA
> php artisan migrate:refresh      # ← DROP + RE-CREATE SEMUA TABEL
> php artisan db:wipe              # ← HAPUS SEMUA TABEL
> php artisan db:seed              # ← Hanya jalankan jika BENAR-BENAR diperlukan
> ```
>
> Perintah-perintah di atas akan **menghapus seluruh data production**. Ini **TIDAK BISA** di-undo.
>
> **Untuk production, HANYA gunakan:**
> ```bash
> php artisan migrate              # ← Menjalankan migration baru saja (aman)
> php artisan migrate --force      # ← Sama, tapi skip konfirmasi di production
> ```

---

## Prasyarat

- Aplikasi sudah berjalan dengan baik di local environment
- Semua test passing (`php artisan test`)
- Repository Git up-to-date
- Akses ke server production (SSH atau panel)

---

## BAGIAN A: KONFIGURASI ENVIRONMENT PRODUCTION

---

## Step 1 — Setup File `.env` Production

### 1.1 — Salin `.env.example` ke server

```bash
cp .env.example .env
```

### 1.2 — Edit `.env` untuk production

> [!WARNING]
> **AI agent WAJIB memastikan nilai-nilai berikut sudah benar sebelum deploy.** Setiap item yang salah bisa menyebabkan error, kebocoran data, atau performa buruk.

```dotenv
#----------------------------------------------------------------------
# Application
#----------------------------------------------------------------------
APP_NAME="NamaAplikasi"
APP_ENV=production                    # ← WAJIB production, bukan local
APP_KEY=                              # ← Generate via: php artisan key:generate
APP_DEBUG=false                       # ← WAJIB false! Jangan pernah true di production
APP_TIMEZONE=Asia/Jakarta
APP_URL=https://yourdomain.com        # ← Domain production (DENGAN https)

#----------------------------------------------------------------------
# Database
#----------------------------------------------------------------------
DB_CONNECTION=mysql
DB_HOST=127.0.0.1                     # ← Sesuaikan dengan host database server
DB_PORT=3306
DB_DATABASE=production_db_name
DB_USERNAME=production_db_user        # ← JANGAN pakai root di production
DB_PASSWORD=strong_random_password    # ← Password yang kuat

#----------------------------------------------------------------------
# Cache & Session
#----------------------------------------------------------------------
CACHE_STORE=redis                     # ← Gunakan Redis jika tersedia, fallback: file
SESSION_DRIVER=redis                  # ← Gunakan Redis jika tersedia, fallback: database
SESSION_LIFETIME=120
SESSION_ENCRYPT=true                  # ← Enkripsi session data

#----------------------------------------------------------------------
# Queue
#----------------------------------------------------------------------
QUEUE_CONNECTION=redis                # ← Gunakan Redis/database, bukan sync
# QUEUE_CONNECTION=database           # ← Alternatif jika Redis tidak tersedia

#----------------------------------------------------------------------
# Mail
#----------------------------------------------------------------------
MAIL_MAILER=smtp
MAIL_HOST=smtp.mailgun.org            # ← Sesuaikan dengan mail provider
MAIL_PORT=587
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="noreply@yourdomain.com"
MAIL_FROM_NAME="${APP_NAME}"

#----------------------------------------------------------------------
# Logging
#----------------------------------------------------------------------
LOG_CHANNEL=daily                     # ← daily agar log di-rotate per hari
LOG_LEVEL=error                       # ← Hanya log error di production (bukan debug)
LOG_DAILY_DAYS=14                     # ← Simpan log 14 hari

#----------------------------------------------------------------------
# Security
#----------------------------------------------------------------------
DEBUGBAR_ENABLED=false                # ← WAJIB false di production
TELESCOPE_ENABLED=false               # ← Matikan jika ada Laravel Telescope

#----------------------------------------------------------------------
# CORS (jika API)
#----------------------------------------------------------------------
FRONTEND_URL=https://frontend.yourdomain.com
```

### 1.3 — Checklist `.env` production

| Item | Nilai yang Benar | ❌ Kesalahan Umum |
|------|-------------------|-------------------|
| `APP_ENV` | `production` | `local` atau `testing` |
| `APP_DEBUG` | `false` | `true` — mengekspos stack trace ke user |
| `APP_URL` | `https://...` | `http://localhost:8000` |
| `DB_USERNAME` | User khusus | `root` |
| `DB_PASSWORD` | Password kuat | Kosong atau `password` |
| `LOG_LEVEL` | `error` atau `warning` | `debug` — log terlalu banyak |
| `DEBUGBAR_ENABLED` | `false` | `true` — mengekspos data sensitif |
| `SESSION_ENCRYPT` | `true` | `false` |

---

## Step 2 — Generate Application Key

```bash
php artisan key:generate
```

> [!IMPORTANT]
> - `APP_KEY` wajib di-generate **sekali** saat setup awal di server.
> - **JANGAN re-generate** `APP_KEY` setelah aplikasi live — ini akan menginvalidasi semua session, cookie, dan data terenkripsi.
> - Backup `APP_KEY` di tempat yang aman (password manager, vault).

---

## BAGIAN B: OPTIMASI ARTISAN

---

## Step 3 — Perintah Optimasi untuk Production

### 3.1 — Clear semua cache terlebih dahulu

```bash
php artisan optimize:clear
```

Perintah ini menjalankan:
- `config:clear`
- `route:clear`
- `view:clear`
- `event:clear`
- `cache:clear`

### 3.2 — Cache konfigurasi

```bash
php artisan config:cache
```

**Efek:** Menggabungkan semua file config menjadi satu file cached. Mempercepat loading config secara signifikan.

> [!WARNING]
> Setelah `config:cache`, panggilan `env()` di luar file config **TIDAK AKAN BEKERJA**. Pastikan semua penggunaan `env()` hanya ada di file-file `config/*.php`, bukan di controller/service/model.
>
> **Cara cek:**
> ```bash
> grep -r "env(" app/ --include="*.php" -l
> ```
> Jika ada hasil, pindahkan panggilan `env()` ke file config yang sesuai.

### 3.3 — Cache route

```bash
php artisan route:cache
```

**Efek:** Mempercepat route resolution. Sangat efektif untuk aplikasi dengan banyak route.

> [!NOTE]
> Route cache **tidak support** route yang menggunakan closure. Semua route harus menggunakan Controller. Jika ada route closure, akan error saat caching.
>
> **Cara cek:**
> ```bash
> grep -n "Route::" routes/web.php | grep "function"
> ```
> Jika ada closure, konversi ke Controller terlebih dahulu.

### 3.4 — Cache view (Blade template)

```bash
php artisan view:cache
```

**Efek:** Pre-compile semua Blade template. Menghilangkan overhead kompilasi saat request pertama.

### 3.5 — Cache event (opsional)

```bash
php artisan event:cache
```

### 3.6 — Satu perintah untuk semua optimasi

```bash
php artisan optimize
```

Perintah `optimize` menjalankan `config:cache`, `route:cache`, dan `view:cache` sekaligus.

### 3.7 — Ringkasan perintah optimasi

| Perintah | Fungsi | Wajib? |
|----------|--------|--------|
| `php artisan optimize:clear` | Bersihkan semua cache | ✅ Jalankan pertama |
| `php artisan optimize` | Cache config + route + view | ✅ Wajib |
| `php artisan config:cache` | Cache konfigurasi | ✅ (included in optimize) |
| `php artisan route:cache` | Cache routing | ✅ (included in optimize) |
| `php artisan view:cache` | Cache Blade views | ✅ (included in optimize) |
| `php artisan event:cache` | Cache event listeners | ⚡ Opsional |
| `php artisan icons:cache` | Cache icon (jika pakai Blade Icons) | ⚡ Opsional |

---

## BAGIAN C: SETUP STORAGE & FILE SYSTEM

---

## Step 4 — Buat Storage Symlink

### 4.1 — Jalankan perintah symlink

```bash
php artisan storage:link
```

**Efek:** Membuat symbolic link dari `public/storage` → `storage/app/public`. Wajib agar file yang diupload user bisa diakses via URL publik.

### 4.2 — Verifikasi symlink

```bash
ls -la public/storage
```

Pastikan output menunjukkan symlink:
```
public/storage -> /path/to/project/storage/app/public
```

### 4.3 — Set permission folder storage

```bash
chmod -R 775 storage bootstrap/cache
chown -R www-data:www-data storage bootstrap/cache
```

> Ganti `www-data` dengan user web server yang sesuai:
> - **nginx**: biasanya `www-data` atau `nginx`
> - **Apache**: biasanya `www-data` atau `apache`
> - **Laravel Forge**: `forge`

> [!IMPORTANT]
> Jika permission salah, Laravel tidak bisa menulis log, session, cache, atau file upload. Error umum: `Permission denied` atau `Failed to open stream`.

---

## BAGIAN D: DATABASE MIGRATION DI PRODUCTION

---

## Step 5 — Jalankan Migration dengan Aman

### 5.1 — Cek status migration

```bash
php artisan migrate:status
```

### 5.2 — Jalankan migration baru

```bash
php artisan migrate --force
```

> Flag `--force` diperlukan karena Laravel di environment `production` akan menampilkan konfirmasi sebelum migrate. Flag ini men-skip konfirmasi (aman untuk CI/CD pipeline).

### 5.3 — Panduan migration di production

| Situasi | Perintah | Aman? |
|---------|----------|-------|
| Deploy pertama kali | `php artisan migrate --force` | ✅ |
| Deploy update (ada migration baru) | `php artisan migrate --force` | ✅ |
| Perlu rollback 1 step | `php artisan migrate:rollback --step=1` | ⚠️ Hati-hati |
| Perlu reset database | **JANGAN** — backup dulu, lalu restore | 🔴 |
| Perlu ubah kolom | Buat migration baru, **JANGAN** edit migration lama | ✅ |

> [!CAUTION]
> **JANGAN PERNAH:**
> - `migrate:fresh` — menghapus semua tabel
> - `migrate:refresh` — rollback semua lalu migrate ulang
> - `migrate:reset` — rollback semua migration
> - Edit file migration yang sudah pernah dijalankan di production
>
> **Jika perlu mengubah struktur tabel**, buat migration baru:
> ```bash
> php artisan make:migration add_phone_column_to_users_table --table=users
> ```

---

## BAGIAN E: QUEUE WORKER

---

## Step 6 — Setup Queue Worker (Jika Menggunakan Queue)

### 6.1 — Cek konfigurasi queue

Pastikan `.env` production menggunakan driver selain `sync`:

```dotenv
QUEUE_CONNECTION=redis     # atau database
```

Jika menggunakan driver `database`, buat tabel queue:

```bash
php artisan queue:table
php artisan migrate --force
```

### 6.2 — Jalankan queue worker

**Untuk testing:**
```bash
php artisan queue:work --tries=3 --timeout=90
```

**Untuk production**, gunakan **Supervisor** agar worker berjalan terus-menerus dan auto-restart jika crash.

### 6.3 — Buat konfigurasi Supervisor

Buat file `/etc/supervisor/conf.d/laravel-worker.conf`:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/project/artisan queue:work redis --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/path/to/project/storage/logs/worker.log
stopwaitsecs=3600
```

> [!NOTE]
> Sesuaikan:
> - `/path/to/project/` dengan path absolut ke project Laravel
> - `user=www-data` dengan user web server
> - `numprocs=2` — jumlah worker process (sesuaikan dengan beban)
> - `redis` dengan driver queue yang digunakan

### 6.4 — Start Supervisor

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

### 6.5 — Restart worker setelah deploy

> [!IMPORTANT]
> Setiap kali deploy kode baru, worker HARUS di-restart agar menggunakan kode terbaru:
> ```bash
> php artisan queue:restart
> ```
> Perintah ini men-signal semua worker untuk gracefully restart setelah job yang sedang berjalan selesai.

---

## BAGIAN F: TASK SCHEDULER (Cron Job)

---

## Step 7 — Setup Task Scheduler (Jika Menggunakan Schedule)

### 7.1 — Tambahkan cron entry di server

```bash
crontab -e
```

Tambahkan baris:
```
* * * * * cd /path/to/project && php artisan schedule:run >> /dev/null 2>&1
```

### 7.2 — Verifikasi schedule terdaftar

```bash
php artisan schedule:list
```

> [!NOTE]
> Cron ini menjalankan `schedule:run` setiap menit. Laravel sendiri yang menentukan command mana yang perlu dijalankan berdasarkan jadwal yang didefinisikan di `app/Console/Kernel.php` atau `routes/console.php` (Laravel 11+).

---

## BAGIAN G: BUILD FRONTEND ASSETS

---

## Step 8 — Build Assets untuk Production

### 8.1 — Install npm dependencies

```bash
npm ci
```

> Gunakan `npm ci` (clean install) di production, bukan `npm install`. `npm ci` lebih cepat dan deterministik karena menginstall dari `package-lock.json`.

### 8.2 — Build production assets

```bash
npm run build
```

**Efek:** Menghasilkan file CSS dan JS yang sudah di-minify dan hashed di `public/build/`.

### 8.3 — Verifikasi build

```bash
ls -la public/build/assets/
```

Pastikan ada file `.css` dan `.js` dengan hash di nama file.

> [!IMPORTANT]
> **JANGAN jalankan `npm run dev` di production.** Dev server Vite hanya untuk development.

---

## BAGIAN H: CHECKLIST KEAMANAN

---

## Step 9 — Security Checklist

### 9.1 — Checklist wajib sebelum go-live

| # | Item | Verifikasi | Status |
|---|------|-----------|--------|
| 1 | `APP_DEBUG=false` | `grep APP_DEBUG .env` | [ ] |
| 2 | `APP_ENV=production` | `grep APP_ENV .env` | [ ] |
| 3 | HTTPS aktif | URL pakai `https://` | [ ] |
| 4 | `DEBUGBAR_ENABLED=false` | `grep DEBUGBAR .env` | [ ] |
| 5 | DB user bukan `root` | `grep DB_USERNAME .env` | [ ] |
| 6 | DB password kuat | Min 16 karakter, random | [ ] |
| 7 | `APP_KEY` sudah di-generate | Tidak kosong di `.env` | [ ] |
| 8 | File `.env` tidak di-commit ke Git | Cek `.gitignore` | [ ] |
| 9 | `storage/` & `bootstrap/cache/` writable | Permission 775 | [ ] |
| 10 | Symlink storage aktif | `ls -la public/storage` | [ ] |
| 11 | Config cached | `php artisan config:cache` | [ ] |
| 12 | Route cached | `php artisan route:cache` | [ ] |
| 13 | View cached | `php artisan view:cache` | [ ] |
| 14 | CORS dikonfigurasi (jika API) | Cek `config/cors.php` | [ ] |
| 15 | Rate limiting aktif | Cek `AppServiceProvider` | [ ] |
| 16 | Session encrypted | `SESSION_ENCRYPT=true` | [ ] |
| 17 | Log level = warning/error | `LOG_LEVEL=error` | [ ] |
| 18 | Tidak ada `dd()` / `dump()` di code | `grep -r "dd(" app/` | [ ] |
| 19 | `composer install --no-dev` | Dev package tidak terinstall | [ ] |

### 9.2 — Perintah verifikasi keamanan cepat

```bash
# Cek penggunaan dd()/dump() di codebase
grep -rn "dd(" app/ resources/ routes/ --include="*.php" --include="*.blade.php"
grep -rn "dump(" app/ resources/ routes/ --include="*.php" --include="*.blade.php"

# Cek env() di luar config (tidak akan bekerja setelah config:cache)
grep -rn "env(" app/ --include="*.php"

# Cek route closure (tidak bisa di-cache)
grep -n "function" routes/web.php routes/api.php 2>/dev/null

# Pastikan .env tidak di-track Git
git ls-files .env
# Harus kosong (tidak ada output)
```

---

## BAGIAN I: DEPLOYMENT SCRIPT

---

## Step 10 — Script Deploy Lengkap

### 10.1 — Deploy script manual (untuk VPS via SSH)

Buat file `deploy.sh` di root project:

```bash
#!/bin/bash
set -e

echo "🚀 Starting deployment..."

# 1. Pull latest code
echo "📥 Pulling latest code..."
git pull origin main

# 2. Install PHP dependencies (tanpa dev)
echo "📦 Installing Composer dependencies..."
composer install --no-dev --optimize-autoloader --no-interaction

# 3. Install & build frontend
echo "🎨 Building frontend assets..."
npm ci
npm run build

# 4. Run migrations
echo "🗄️ Running migrations..."
php artisan migrate --force

# 5. Clear & optimize
echo "⚡ Optimizing application..."
php artisan optimize:clear
php artisan optimize

# 6. Storage symlink (hanya perlu sekali, tapi aman dijalankan ulang)
php artisan storage:link 2>/dev/null || true

# 7. Restart queue worker (jika ada)
echo "🔄 Restarting queue workers..."
php artisan queue:restart 2>/dev/null || true

# 8. Clear permission cache (jika pakai Spatie)
echo "🔑 Resetting permission cache..."
php artisan permission:cache-reset 2>/dev/null || true

echo "✅ Deployment complete!"
```

Buat executable:
```bash
chmod +x deploy.sh
```

Jalankan:
```bash
./deploy.sh
```

### 10.2 — Urutan perintah deploy (tanpa script)

Jika tidak menggunakan script, jalankan perintah berikut **secara berurutan**:

```bash
# 1. Pull kode terbaru
git pull origin main

# 2. Install dependencies (tanpa dev packages)
composer install --no-dev --optimize-autoloader --no-interaction

# 3. Build frontend
npm ci && npm run build

# 4. Jalankan migration baru
php artisan migrate --force

# 5. Clear semua cache
php artisan optimize:clear

# 6. Cache ulang untuk production
php artisan optimize

# 7. Storage link (sekali saja)
php artisan storage:link

# 8. Restart queue (jika ada)
php artisan queue:restart

# 9. Reset permission cache (jika pakai Spatie)
php artisan permission:cache-reset
```

### 10.3 — Deploy script untuk platform tertentu

#### Laravel Forge

Forge menyediakan deployment script bawaan. Edit di dashboard Forge:

```bash
cd /home/forge/yourdomain.com
git pull origin main
composer install --no-dev --optimize-autoloader --no-interaction
npm ci && npm run build
php artisan migrate --force
php artisan optimize:clear
php artisan optimize
php artisan queue:restart
```

#### Railway

Tambahkan di `Procfile`:
```
web: php artisan optimize && php artisan migrate --force && php-fpm
```

Atau set build command di Railway dashboard:
```
composer install --no-dev --optimize-autoloader && npm ci && npm run build && php artisan optimize
```

---

## BAGIAN J: MONITORING & MAINTENANCE

---

## Step 11 — Post-Deploy Monitoring

### 11.1 — Cek aplikasi berjalan

```bash
curl -I https://yourdomain.com
```

Pastikan response `HTTP/2 200`.

### 11.2 — Monitor log error

```bash
tail -f storage/logs/laravel.log
```

### 11.3 — Maintenance mode

**Aktifkan maintenance mode sebelum deploy besar:**

```bash
php artisan down --secret="bypass-secret-token"
```

> Flag `--secret` memungkinkan akses halaman selama maintenance dengan menambahkan token di URL: `https://yourdomain.com/bypass-secret-token`

**Nonaktifkan setelah deploy selesai:**

```bash
php artisan up
```

---

## Checklist Verifikasi Akhir

### Pre-Deployment

- [ ] Semua test passing (`php artisan test`)
- [ ] Tidak ada `dd()` / `dump()` di codebase
- [ ] Tidak ada `env()` di luar file config
- [ ] Tidak ada route closure (semua pakai Controller)
- [ ] `.env` production sudah disiapkan
- [ ] `APP_DEBUG=false`, `APP_ENV=production`
- [ ] Database production sudah dibuat
- [ ] DB user bukan `root`, password kuat

### Deployment

- [ ] `composer install --no-dev` berhasil
- [ ] `npm ci && npm run build` berhasil
- [ ] `php artisan migrate --force` berhasil
- [ ] `php artisan optimize` berhasil
- [ ] `php artisan storage:link` sudah dijalankan
- [ ] Permission `storage/` dan `bootstrap/cache/` sudah benar

### Post-Deployment

- [ ] Website bisa diakses via browser
- [ ] HTTPS berfungsi
- [ ] Login/register berfungsi
- [ ] File upload berfungsi (storage symlink)
- [ ] Queue berjalan (jika ada)
- [ ] Cron/scheduler berjalan (jika ada)
- [ ] Tidak ada error di `storage/logs/laravel.log`
- [ ] Debugbar **TIDAK** muncul di browser

---

## Catatan Penting untuk AI Agent

> [!CAUTION]
> 1. **JANGAN PERNAH** jalankan `migrate:fresh`, `migrate:refresh`, `migrate:reset`, atau `db:wipe` di production. Ini menghapus **SEMUA DATA**.
> 2. **JANGAN** set `APP_DEBUG=true` di production. Stack trace akan mengekspos path, database credentials, dan informasi sensitif.
> 3. **JANGAN** jalankan `npm run dev` di production. Gunakan `npm run build`.
> 4. **JANGAN** gunakan `root` sebagai DB user di production.
> 5. **JANGAN** commit file `.env` ke Git.
> 6. **Selalu** jalankan `php artisan optimize:clear` **sebelum** `php artisan optimize` saat deploy.
> 7. **Selalu** jalankan `php artisan queue:restart` setelah deploy jika ada queue worker.
> 8. **Selalu** backup database sebelum menjalankan migration di production:
>    ```bash
>    mysqldump -u user -p database_name > backup_$(date +%Y%m%d_%H%M%S).sql
>    ```
> 9. **Tanyakan ke user** platform deployment yang digunakan (VPS, Forge, Railway, dll) dan sesuaikan script.
> 10. **Jangan asumsikan** Redis tersedia — tanyakan dulu. Jika tidak ada, gunakan `file` untuk cache dan `database` untuk queue.
