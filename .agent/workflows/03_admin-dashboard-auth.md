---
description: SOP untuk mengintegrasikan autentikasi Laravel Breeze, layout dashboard admin dengan sidebar/navbar, route group dengan middleware auth, dan Controller dasar dashboard.
---

# 03 — Admin Dashboard & Authentication

> **Tujuan**: Mengintegrasikan sistem autentikasi menggunakan Laravel Breeze, membuat layout khusus dashboard admin (sidebar + navbar), memisahkan route menggunakan route group dengan middleware `auth`, dan membuat Controller dasar untuk halaman dashboard.

---

## Prasyarat

- Laravel sudah terinstall (lihat `01_laravel-backend-setup.md`)
- Frontend Blade + Vite + Tailwind CSS sudah di-setup (lihat `02_frontend-blade-vite.md`)
- Database MySQL sudah terhubung dan migration awal sudah jalan
- `npm run dev` bisa berjalan tanpa error

---

## Step 1 — Install Laravel Breeze

### 1.1 — Install package via Composer

// turbo
```bash
composer require laravel/breeze --dev
```

### 1.2 — Jalankan scaffolding Breeze (Blade stack)

// turbo
```bash
php artisan breeze:install blade
```

> [!IMPORTANT]
> Pilih **Blade** sebagai stack. Jangan pilih Vue, React, atau API — workflow ini menggunakan Blade templating.
>
> Jika `php artisan breeze:install` bersifat interaktif, gunakan flag tambahan jika tersedia, atau jawab sesuai opsi Blade.

### 1.3 — Install npm dependencies yang ditambahkan Breeze

// turbo
```bash
npm install
```

### 1.4 — Jalankan migration untuk tabel users

// turbo
```bash
php artisan migrate
```

### 1.5 — Verifikasi instalasi

// turbo
```bash
php artisan route:list --name=login
```

Pastikan route `login`, `register`, `logout`, dan `password.*` sudah terdaftar.

> [!NOTE]
> Laravel Breeze akan membuat/memodifikasi file-file berikut:
> - `routes/auth.php` — route autentikasi
> - `app/Http/Controllers/Auth/` — controller auth (Login, Register, Password, dll)
> - `resources/views/auth/` — view login, register, forgot password
> - `resources/views/layouts/` — layout bawaan Breeze (mungkin **menimpa** layout yang sudah dibuat di workflow 02)
>
> **Langkah selanjutnya akan menangani konflik ini.**

---

## Step 2 — Resolusi Konflik dengan Layout Workflow 02

> [!WARNING]
> Laravel Breeze kemungkinan akan **menimpa** file layout yang sudah ada dari workflow `02_frontend-blade-vite.md`. Langkah ini memastikan layout admin kustom tetap digunakan.

### 2.1 — Audit file yang berubah

Cek file yang di-overwrite oleh Breeze:

// turbo
```bash
git diff --name-only
```

Jika belum menggunakan Git, periksa secara manual file-file berikut:

- `resources/views/layouts/app.blade.php`
- `resources/views/layouts/navigation.blade.php`
- `resources/css/app.css`
- `resources/js/app.js`
- `tailwind.config.js`
- `vite.config.js`

### 2.2 — Strategi penanganan

Ada dua pendekatan. **Pilih salah satu** berdasarkan kebutuhan user:

#### Opsi A: Pisahkan layout admin dan layout Breeze (DIREKOMENDASIKAN)

Biarkan layout bawaan Breeze untuk halaman auth (`login`, `register`, dll), dan buat layout **terpisah** untuk admin dashboard.

**Struktur folder:**

```
resources/views/layouts/
├── app.blade.php           # Layout Breeze (untuk halaman auth & profil)
├── guest.blade.php         # Layout Breeze guest (login, register)
├── navigation.blade.php   # Navigation Breeze
└── admin.blade.php         # Layout kustom admin dashboard (BUAT BARU)
```

#### Opsi B: Override layout Breeze sepenuhnya

Ganti semua layout Breeze dengan layout kustom dari workflow 02. Lebih bersih tapi butuh penyesuaian lebih banyak di view auth.

> [!NOTE]
> Workflow ini menggunakan **Opsi A** sebagai default. Tanyakan ke user jika ingin menggunakan Opsi B.

### 2.3 — Restore CSS dan JS kustom

Jika Breeze menimpa `resources/css/app.css` dan `resources/js/app.js`, **gabungkan** konten Breeze dengan konten kustom dari workflow 02.

**Untuk `resources/css/app.css`**, pastikan berisi:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ============================================
   Custom Base Styles (dari workflow 02)
   ============================================ */
@layer base {
    body {
        @apply font-sans antialiased text-gray-800 bg-gray-50;
    }
}

/* ============================================
   Custom Component Classes (dari workflow 02)
   Salin kembali dari 02_frontend-blade-vite.md Step 4
   ============================================ */
@layer components {
    .btn {
        @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg
               transition-all duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-offset-2
               disabled:opacity-50 disabled:cursor-not-allowed;
    }
    .btn-primary {
        @apply btn bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500 shadow-sm hover:shadow-md;
    }
    .btn-secondary {
        @apply btn bg-white text-gray-700 border border-gray-300 hover:bg-gray-50 focus:ring-gray-500;
    }
    .btn-danger {
        @apply btn bg-red-600 text-white hover:bg-red-700 focus:ring-red-500;
    }
    .btn-sm { @apply px-3 py-1.5 text-xs; }
    .btn-lg { @apply px-6 py-3 text-base; }

    .card {
        @apply bg-white rounded-xl shadow-sm border border-gray-200 overflow-hidden;
    }
    .card-body { @apply p-6; }
    .card-header {
        @apply px-6 py-4 border-b border-gray-200 bg-gray-50/50;
    }

    .form-input {
        @apply block w-full rounded-lg border-gray-300 shadow-sm
               focus:border-primary-500 focus:ring-primary-500 text-sm placeholder:text-gray-400;
    }
    .form-label { @apply block text-sm font-medium text-gray-700 mb-1; }
    .form-error { @apply text-sm text-red-600 mt-1; }

    .badge { @apply inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium; }
    .badge-success { @apply badge bg-green-100 text-green-800; }
    .badge-warning { @apply badge bg-yellow-100 text-yellow-800; }
    .badge-danger  { @apply badge bg-red-100 text-red-800; }
    .badge-info    { @apply badge bg-blue-100 text-blue-800; }

    .sidebar-link {
        @apply flex items-center gap-3 px-3 py-2.5 text-sm font-medium rounded-lg transition-colors duration-150;
    }
    .sidebar-link-active {
        @apply sidebar-link bg-primary-50 text-primary-700;
    }
    .sidebar-link-inactive {
        @apply sidebar-link text-gray-600 hover:bg-gray-100 hover:text-gray-900;
    }
}
```

**Untuk `tailwind.config.js`**, pastikan `content` paths dan custom colors (dari workflow 02) tetap ada. Gabungkan dengan konfigurasi Breeze jika ada tambahan.

---

## Step 3 — Buat Layout Admin Dashboard

### 3.1 — Buat file layout admin

Buat file `resources/views/layouts/admin.blade.php`:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? 'Dashboard' }} — {{ config('app.name') }}</title>

    {{-- Google Fonts --}}
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=inter:300,400,500,600,700,800" rel="stylesheet" />

    {{-- Vite Assets --}}
    @vite(['resources/css/app.css', 'resources/js/app.js'])

    @stack('styles')
</head>
<body class="min-h-screen bg-gray-50 font-sans antialiased">

    <div class="flex min-h-screen">
        {{-- Sidebar --}}
        @include('layouts.admin-sidebar')

        {{-- Main Content Wrapper --}}
        <div class="flex-1 flex flex-col lg:ml-64">
            {{-- Top Navbar --}}
            @include('layouts.admin-navbar')

            {{-- Page Content --}}
            <main class="flex-1 p-4 sm:p-6 lg:p-8">
                {{-- Flash Messages --}}
                @include('partials.alert')

                {{-- Page Header --}}
                @isset($header)
                    <div class="mb-6">
                        {{ $header }}
                    </div>
                @endisset

                {{-- Main Content Slot --}}
                {{ $slot }}
            </main>

            {{-- Footer --}}
            <footer class="border-t border-gray-200 bg-white">
                <div class="px-4 sm:px-6 lg:px-8 py-4">
                    <div class="flex flex-col sm:flex-row items-center justify-between gap-2">
                        <p class="text-sm text-gray-500">
                            &copy; {{ date('Y') }} {{ config('app.name') }}. All rights reserved.
                        </p>
                        <p class="text-xs text-gray-400">
                            Built with Laravel & Tailwind CSS
                        </p>
                    </div>
                </div>
            </footer>
        </div>
    </div>

    {{-- Mobile Sidebar Overlay --}}
    <div id="sidebar-overlay" class="fixed inset-0 bg-black/50 z-30 hidden lg:hidden"></div>

    @stack('scripts')
</body>
</html>
```

> [!NOTE]
> Layout admin menggunakan `@include` untuk sidebar dan navbar (bukan Blade component `<x-...>`).
> Ini disengaja agar lebih mudah dikelola terpisah dari komponen Breeze.
> Halaman yang menggunakan layout ini harus dibungkus dengan `<x-admin-layout>`.

### 3.2 — Buat Admin Sidebar

Buat file `resources/views/layouts/admin-sidebar.blade.php`:

```blade
{{-- Admin Sidebar --}}
<aside id="sidebar"
       class="fixed inset-y-0 left-0 z-40 w-64 bg-white border-r border-gray-200
              transform -translate-x-full lg:translate-x-0 transition-transform duration-300 ease-in-out">

    {{-- Brand / Logo --}}
    <div class="flex items-center gap-3 px-6 h-16 border-b border-gray-200">
        <div class="w-8 h-8 bg-primary-600 rounded-lg flex items-center justify-center shrink-0">
            <span class="text-white font-bold text-sm">{{ substr(config('app.name'), 0, 1) }}</span>
        </div>
        <span class="text-lg font-bold text-gray-900 truncate">{{ config('app.name') }}</span>
    </div>

    {{-- Navigation --}}
    <nav class="p-4 space-y-1 overflow-y-auto h-[calc(100vh-4rem)]">

        {{-- Section: Menu Utama --}}
        <p class="px-3 pt-2 pb-2 text-xs font-semibold text-gray-400 uppercase tracking-wider">
            Menu Utama
        </p>

        <a href="{{ route('admin.dashboard') }}"
           class="{{ request()->routeIs('admin.dashboard') ? 'sidebar-link-active' : 'sidebar-link-inactive' }}">
            <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-4 0a1 1 0 01-1-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 01-1 1h-2z" />
            </svg>
            <span>Dashboard</span>
        </a>

        {{-- Tambahkan menu item lain di sini sesuai fitur --}}
        {{-- Contoh:
        <a href="{{ route('admin.users.index') }}"
           class="{{ request()->routeIs('admin.users.*') ? 'sidebar-link-active' : 'sidebar-link-inactive' }}">
            <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.25 2.25 0 11-4.5 0 2.25 2.25 0 014.5 0z" />
            </svg>
            <span>Manajemen User</span>
        </a>
        --}}

        {{-- Section: Pengaturan --}}
        <p class="px-3 pt-6 pb-2 text-xs font-semibold text-gray-400 uppercase tracking-wider">
            Pengaturan
        </p>

        <a href="{{ route('profile.edit') }}"
           class="{{ request()->routeIs('profile.*') ? 'sidebar-link-active' : 'sidebar-link-inactive' }}">
            <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" />
            </svg>
            <span>Profil Saya</span>
        </a>

        <a href="#"
           class="sidebar-link-inactive">
            <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.066 2.573c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.573 1.066c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.066-2.573c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
            </svg>
            <span>Settings</span>
        </a>

        {{-- Spacer --}}
        <div class="flex-1"></div>

        {{-- Logout --}}
        <div class="pt-6 border-t border-gray-200 mt-6">
            <form method="POST" action="{{ route('logout') }}">
                @csrf
                <button type="submit"
                        class="sidebar-link-inactive w-full text-red-600 hover:bg-red-50 hover:text-red-700">
                    <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                              d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
                    </svg>
                    <span>Logout</span>
                </button>
            </form>
        </div>
    </nav>
</aside>
```

### 3.3 — Buat Admin Navbar (Top Header)

Buat file `resources/views/layouts/admin-navbar.blade.php`:

```blade
{{-- Top Navbar --}}
<header class="sticky top-0 z-20 bg-white/80 backdrop-blur-md border-b border-gray-200">
    <div class="flex items-center justify-between h-16 px-4 sm:px-6 lg:px-8">

        {{-- Left: Mobile toggle + Breadcrumb --}}
        <div class="flex items-center gap-4">
            {{-- Mobile Sidebar Toggle --}}
            <button id="sidebar-toggle"
                    class="lg:hidden p-2 rounded-lg text-gray-500 hover:bg-gray-100 hover:text-gray-700
                           transition-colors focus:outline-none focus:ring-2 focus:ring-primary-500"
                    aria-label="Toggle sidebar">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                          d="M4 6h16M4 12h16M4 18h16" />
                </svg>
            </button>

            {{-- Breadcrumb (opsional) --}}
            @isset($breadcrumb)
                <nav class="hidden sm:flex items-center gap-2 text-sm text-gray-500">
                    {{ $breadcrumb }}
                </nav>
            @endisset
        </div>

        {{-- Right: Notifications + User Menu --}}
        <div class="flex items-center gap-3">
            {{-- Notification Bell --}}
            <button class="relative p-2 rounded-lg text-gray-500 hover:bg-gray-100 hover:text-gray-700
                           transition-colors focus:outline-none focus:ring-2 focus:ring-primary-500"
                    aria-label="Notifications">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                          d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
                </svg>
                {{-- Badge: uncomment jika ada notifikasi --}}
                {{-- <span class="absolute top-1.5 right-1.5 w-2 h-2 bg-red-500 rounded-full ring-2 ring-white"></span> --}}
            </button>

            {{-- User Info + Dropdown --}}
            <div class="flex items-center gap-3 pl-3 border-l border-gray-200">
                {{-- Avatar --}}
                <div class="w-8 h-8 rounded-full bg-primary-100 flex items-center justify-center">
                    <span class="text-sm font-semibold text-primary-700">
                        {{ strtoupper(substr(Auth::user()->name, 0, 1)) }}
                    </span>
                </div>
                {{-- Name & Email --}}
                <div class="hidden sm:block">
                    <p class="text-sm font-medium text-gray-900 leading-tight">{{ Auth::user()->name }}</p>
                    <p class="text-xs text-gray-500 leading-tight">{{ Auth::user()->email }}</p>
                </div>
                {{-- Dropdown toggle (Alpine.js atau JS biasa) --}}
                {{-- Implementasi dropdown bisa ditambahkan di sini jika menggunakan Alpine.js --}}
            </div>
        </div>
    </div>
</header>
```

---

## Step 4 — Buat Admin Layout Component

Agar layout admin bisa digunakan dengan syntax `<x-admin-layout>`, buat Blade component class:

### 4.1 — Buat component class

// turbo
```bash
php artisan make:component AdminLayout --view
```

> Ini akan membuat file di `resources/views/components/admin-layout.blade.php`.

### 4.2 — Edit component view

Buka `resources/views/components/admin-layout.blade.php` dan **ganti isinya** agar merujuk ke layout admin:

```blade
@include('layouts.admin', ['slot' => $slot, 'title' => $title ?? 'Dashboard', 'header' => $header ?? null, 'breadcrumb' => $breadcrumb ?? null])
```

**Alternatif yang lebih baik**: ubah agar layout `admin.blade.php` sendiri menjadi component. Buat class component:

// turbo
```bash
php artisan make:component AdminLayout
```

Edit file `app/View/Components/AdminLayout.php`:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class AdminLayout extends Component
{
    public function __construct(
        public string $title = 'Dashboard'
    ) {}

    public function render(): View
    {
        return view('layouts.admin');
    }
}
```

Sekarang layout admin bisa digunakan di halaman blade seperti:

```blade
<x-admin-layout title="Dashboard">
    {{-- konten halaman --}}
</x-admin-layout>
```

---

## Step 5 — Buat Dashboard Controller

### 5.1 — Generate controller

// turbo
```bash
php artisan make:controller Admin/DashboardController
```

### 5.2 — Edit DashboardController

Buka `app/Http/Controllers/Admin/DashboardController.php` dan isi:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\View\View;

class DashboardController extends Controller
{
    /**
     * Tampilkan halaman utama dashboard admin.
     */
    public function index(): View
    {
        // Data statis untuk dashboard — ganti dengan query database nanti
        $stats = [
            [
                'label' => 'Total Users',
                'value' => '1,248',
                'change' => '+12.5%',
                'trend' => 'up',
                'icon' => 'users',
                'color' => 'blue',
            ],
            [
                'label' => 'Pendapatan',
                'value' => 'Rp 45,2jt',
                'change' => '+8.2%',
                'trend' => 'up',
                'icon' => 'currency',
                'color' => 'green',
            ],
            [
                'label' => 'Total Orders',
                'value' => '342',
                'change' => '-3.1%',
                'trend' => 'down',
                'icon' => 'shopping',
                'color' => 'purple',
            ],
            [
                'label' => 'Active Sessions',
                'value' => '89',
                'change' => 'Real-time',
                'trend' => 'neutral',
                'icon' => 'bolt',
                'color' => 'orange',
            ],
        ];

        $recentActivities = [
            [
                'user' => 'Ahmad Budi',
                'initials' => 'AB',
                'action' => 'Membuat pesanan baru',
                'status' => 'success',
                'status_label' => 'Selesai',
                'time' => '2 menit lalu',
            ],
            [
                'user' => 'Citra Sari',
                'initials' => 'CS',
                'action' => 'Update profil',
                'status' => 'info',
                'status_label' => 'Proses',
                'time' => '15 menit lalu',
            ],
            [
                'user' => 'Doni Rahmad',
                'initials' => 'DR',
                'action' => 'Pembayaran gagal',
                'status' => 'danger',
                'status_label' => 'Gagal',
                'time' => '1 jam lalu',
            ],
            [
                'user' => 'Eka Fitria',
                'initials' => 'EF',
                'action' => 'Register akun baru',
                'status' => 'warning',
                'status_label' => 'Pending',
                'time' => '2 jam lalu',
            ],
        ];

        return view('admin.dashboard', compact('stats', 'recentActivities'));
    }
}
```

> [!NOTE]
> Data `$stats` dan `$recentActivities` saat ini berupa static array. Nanti saat fitur sudah terimplementasi, ganti dengan query Eloquent yang sesuai.

---

## Step 6 — Setup Route dengan Route Group & Middleware Auth

### 6.1 — Edit `routes/web.php`

Buka `routes/web.php` dan ubah isinya menjadi:

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Admin\DashboardController;
use App\Http\Controllers\ProfileController;

/*
|--------------------------------------------------------------------------
| Public Routes (tanpa auth)
|--------------------------------------------------------------------------
*/

Route::get('/', function () {
    return redirect()->route('login');
});

/*
|--------------------------------------------------------------------------
| Authenticated Routes (memerlukan login)
|--------------------------------------------------------------------------
*/

Route::middleware(['auth', 'verified'])->group(function () {

    /*
    |----------------------------------------------------------------------
    | Admin Dashboard Routes
    |----------------------------------------------------------------------
    */
    Route::prefix('admin')->name('admin.')->group(function () {

        // Dashboard
        Route::get('/dashboard', [DashboardController::class, 'index'])
            ->name('dashboard');

        // Tambahkan resource routes lain di sini:
        // Route::resource('users', Admin\UserController::class);
        // Route::resource('settings', Admin\SettingController::class);
    });

    /*
    |----------------------------------------------------------------------
    | Profile Routes (dari Breeze)
    |----------------------------------------------------------------------
    */
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});

/*
|--------------------------------------------------------------------------
| Auth Routes (dari Breeze)
|--------------------------------------------------------------------------
*/
require __DIR__ . '/auth.php';
```

### 6.2 — Penjelasan struktur route

```
routes/web.php
│
├── GET /                          → Redirect ke login
│
├── middleware(['auth', 'verified'])
│   │
│   ├── prefix('admin')            → Semua URL dimulai /admin/...
│   │   ├── GET /admin/dashboard   → DashboardController@index
│   │   ├── (future routes...)
│   │   └── ...
│   │
│   └── Profile routes             → ProfileController (dari Breeze)
│       ├── GET /profile
│       ├── PATCH /profile
│       └── DELETE /profile
│
└── require auth.php               → Route login/register/logout (dari Breeze)
```

### 6.3 — Verifikasi route

// turbo
```bash
php artisan route:list --columns=method,uri,name,middleware
```

Pastikan route `admin.dashboard` memiliki middleware `auth` dan `verified`.

---

## Step 7 — Update Dashboard View untuk Menggunakan Data dari Controller

### 7.1 — Update `resources/views/admin/dashboard.blade.php`

Ganti isi `resources/views/admin/dashboard.blade.php` agar menggunakan data dari controller:

```blade
<x-admin-layout title="Dashboard">
    {{-- Page Header --}}
    <x-slot name="header">
        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
            <div>
                <h2 class="text-2xl font-bold text-gray-900">Dashboard</h2>
                <p class="text-sm text-gray-500 mt-1">
                    Selamat datang kembali, {{ Auth::user()->name }}!
                </p>
            </div>
            <div class="flex items-center gap-2">
                <button class="btn-secondary btn-sm">
                    <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                              d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
                    </svg>
                    Export
                </button>
                <button class="btn-primary btn-sm">
                    <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                    </svg>
                    Tambah Baru
                </button>
            </div>
        </div>
    </x-slot>

    {{-- Stats Cards --}}
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-6 mb-6 sm:mb-8">
        @foreach ($stats as $stat)
            <div class="card">
                <div class="card-body">
                    <div class="flex items-center justify-between">
                        <div>
                            <p class="text-sm font-medium text-gray-500">{{ $stat['label'] }}</p>
                            <p class="text-2xl sm:text-3xl font-bold text-gray-900 mt-1">{{ $stat['value'] }}</p>
                            <p class="text-xs mt-2 flex items-center gap-1
                                {{ $stat['trend'] === 'up' ? 'text-green-600' : ($stat['trend'] === 'down' ? 'text-red-600' : 'text-gray-500') }}">
                                @if ($stat['trend'] === 'up')
                                    <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                                    </svg>
                                @elseif ($stat['trend'] === 'down')
                                    <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
                                    </svg>
                                @endif
                                {{ $stat['change'] }}
                            </p>
                        </div>
                        <div class="w-12 h-12 bg-{{ $stat['color'] }}-100 rounded-xl flex items-center justify-center shrink-0">
                            @switch($stat['icon'])
                                @case('users')
                                    <svg class="w-6 h-6 text-{{ $stat['color'] }}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197" />
                                    </svg>
                                    @break
                                @case('currency')
                                    <svg class="w-6 h-6 text-{{ $stat['color'] }}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                                    </svg>
                                    @break
                                @case('shopping')
                                    <svg class="w-6 h-6 text-{{ $stat['color'] }}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
                                    </svg>
                                    @break
                                @case('bolt')
                                    <svg class="w-6 h-6 text-{{ $stat['color'] }}-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                              d="M13 10V3L4 14h7v7l9-11h-7z" />
                                    </svg>
                                    @break
                            @endswitch
                        </div>
                    </div>
                </div>
            </div>
        @endforeach
    </div>

    {{-- Content Grid: Activity Table + Quick Actions --}}
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 sm:gap-6">
        {{-- Recent Activity Table --}}
        <div class="lg:col-span-2 card">
            <div class="card-header">
                <div class="flex items-center justify-between">
                    <h3 class="text-base font-semibold text-gray-900">Aktivitas Terbaru</h3>
                    <a href="#" class="text-sm text-primary-600 hover:text-primary-700 font-medium">Lihat Semua</a>
                </div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-sm">
                    <thead>
                        <tr class="border-b border-gray-200">
                            <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider">User</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider">Aksi</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider hidden sm:table-cell">Status</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider hidden md:table-cell">Waktu</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-gray-100">
                        @forelse ($recentActivities as $activity)
                            <tr class="hover:bg-gray-50 transition-colors">
                                <td class="px-6 py-4">
                                    <div class="flex items-center gap-3">
                                        <div class="w-8 h-8 rounded-full bg-primary-100 flex items-center justify-center shrink-0">
                                            <span class="text-xs font-semibold text-primary-700">{{ $activity['initials'] }}</span>
                                        </div>
                                        <span class="font-medium text-gray-900">{{ $activity['user'] }}</span>
                                    </div>
                                </td>
                                <td class="px-6 py-4 text-gray-600">{{ $activity['action'] }}</td>
                                <td class="px-6 py-4 hidden sm:table-cell">
                                    <span class="badge-{{ $activity['status'] }}">{{ $activity['status_label'] }}</span>
                                </td>
                                <td class="px-6 py-4 text-gray-500 hidden md:table-cell">{{ $activity['time'] }}</td>
                            </tr>
                        @empty
                            <tr>
                                <td colspan="4" class="px-6 py-8 text-center text-gray-400">
                                    Belum ada aktivitas terbaru.
                                </td>
                            </tr>
                        @endforelse
                    </tbody>
                </table>
            </div>
        </div>

        {{-- System Info Panel --}}
        <div class="card">
            <div class="card-header">
                <h3 class="text-base font-semibold text-gray-900">Info Sistem</h3>
            </div>
            <div class="card-body space-y-3">
                <div class="flex items-center justify-between">
                    <span class="text-sm text-gray-500">Laravel</span>
                    <span class="text-sm font-medium text-gray-900">{{ app()->version() }}</span>
                </div>
                <div class="flex items-center justify-between">
                    <span class="text-sm text-gray-500">PHP</span>
                    <span class="text-sm font-medium text-gray-900">{{ phpversion() }}</span>
                </div>
                <div class="flex items-center justify-between">
                    <span class="text-sm text-gray-500">Environment</span>
                    <span class="badge-info">{{ app()->environment() }}</span>
                </div>
                <div class="flex items-center justify-between">
                    <span class="text-sm text-gray-500">Login sebagai</span>
                    <span class="text-sm font-medium text-gray-900">{{ Auth::user()->email }}</span>
                </div>
                <div class="flex items-center justify-between">
                    <span class="text-sm text-gray-500">Server Time</span>
                    <span class="text-sm font-medium text-gray-900">{{ now()->format('d M Y, H:i') }}</span>
                </div>
            </div>
        </div>
    </div>
</x-admin-layout>
```

---

## Step 8 — Redirect Setelah Login ke Dashboard Admin

### 8.1 — Edit redirect setelah login

Breeze menggunakan `RouteServiceProvider::HOME` atau constant `HOME` untuk redirect setelah login. Temukan dan ubah nilainya.

**Pada Laravel 11+ (Breeze terbaru)**, cari di `app/Http/Controllers/Auth/AuthenticatedSessionController.php` atau `app/Providers/RouteServiceProvider.php`:

```php
// Cari baris ini:
public const HOME = '/dashboard';

// Ubah menjadi:
public const HOME = '/admin/dashboard';
```

**Jika tidak ada `RouteServiceProvider.php`** (Laravel 11+), cari di `app/Http/Controllers/Auth/AuthenticatedSessionController.php`:

```php
// Pada method store(), cari:
return redirect()->intended(route('dashboard', absolute: false));

// Ubah menjadi:
return redirect()->intended(route('admin.dashboard', absolute: false));
```

**Lakukan juga di** `app/Http/Controllers/Auth/RegisteredUserController.php`:

```php
// Pada method store(), cari redirect setelah register dan ubah ke:
return redirect(route('admin.dashboard', absolute: false));
```

> [!IMPORTANT]
> Periksa **semua file controller di** `app/Http/Controllers/Auth/` untuk memastikan tidak ada yang masih redirect ke `/dashboard` lama. Gunakan pencarian:
> ```bash
> grep -r "route('dashboard'" app/Http/Controllers/Auth/
> ```

### 8.2 — Hapus route `/dashboard` lama (jika ada)

Cek apakah Breeze membuat route `/dashboard` di `routes/web.php`. Jika ada, **hapus** karena sudah diganti oleh `/admin/dashboard`:

```php
// HAPUS baris ini jika ada:
Route::get('/dashboard', function () {
    return view('dashboard');
})->middleware(['auth', 'verified'])->name('dashboard');
```

---

## Step 9 — Buat User Seeder (Opsional, untuk Testing)

### 9.1 — Edit Database Seeder

Buka `database/seeders/DatabaseSeeder.php`:

```php
<?php

namespace Database\Seeders;

use App\Models\User;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        // Admin user untuk testing
        User::factory()->create([
            'name' => 'Admin',
            'email' => 'admin@admin.com',
            'password' => Hash::make('password'),
        ]);

        // Tambahkan dummy users lain (opsional)
        // User::factory(10)->create();
    }
}
```

### 9.2 — Jalankan seeder

// turbo
```bash
php artisan db:seed
```

> [!NOTE]
> Akun testing default:
> - **Email**: `admin@admin.com`
> - **Password**: `password`
>
> Tanyakan ke user jika ingin kredensial berbeda.

---

## Step 10 — Verifikasi End-to-End

### 10.1 — Jalankan dev server + Vite

Buka dua terminal terpisah:

**Terminal 1:**
// turbo
```bash
npm run dev
```

**Terminal 2:**
// turbo
```bash
php artisan serve
```

### 10.2 — Test flow autentikasi

Buka browser dan lakukan tes berikut:

| # | Test Case                                      | Expected Result                                         |
|---|-----------------------------------------------|---------------------------------------------------------|
| 1 | Buka `http://localhost:8000`                  | Redirect ke halaman login                               |
| 2 | Buka `http://localhost:8000/admin/dashboard`  | Redirect ke login (belum auth)                          |
| 3 | Register akun baru                             | Berhasil, redirect ke `/admin/dashboard`                |
| 4 | Logout                                         | Kembali ke halaman login                                |
| 5 | Login dengan akun yang sudah register          | Masuk ke dashboard                                      |
| 6 | Login dengan `admin@admin.com` / `password`    | Masuk ke dashboard (jika seeder sudah jalan)            |
| 7 | Dashboard menampilkan nama user yang login     | Nama user tampil di header & welcome message            |
| 8 | Sidebar responsive — toggle di mobile          | Sidebar muncul/hilang dengan animasi                    |
| 9 | Klik "Profil Saya" di sidebar                 | Navigasi ke halaman profile Breeze                      |
| 10| Klik Logout di sidebar                        | Logout dan redirect ke login                            |

### 10.3 — Cek responsivitas dashboard

Gunakan browser DevTools (F12) dan verifikasi:

| Breakpoint | Lebar  | Verifikasi                                          |
|-----------|--------|-----------------------------------------------------|
| Mobile    | 375px  | Sidebar hidden, toggle button, stat cards 1 kolom   |
| Tablet    | 768px  | Stat cards 2 kolom, tabel kolom partial hidden      |
| Desktop   | 1024px | Sidebar visible, stat cards 4 kolom, semua kolom    |

---

## Checklist Verifikasi Akhir

- [ ] Laravel Breeze terinstall (`composer show laravel/breeze`)
- [ ] Route auth terdaftar: `login`, `register`, `logout`, `password.*`
- [ ] Konflik layout dengan workflow 02 sudah diresolved
- [ ] CSS custom classes (`.btn-*`, `.card`, `.sidebar-link-*`) tetap berfungsi
- [ ] Layout admin terpisah di `layouts/admin.blade.php`
- [ ] Admin sidebar di `layouts/admin-sidebar.blade.php` — termasuk logout & profil
- [ ] Admin navbar di `layouts/admin-navbar.blade.php` — termasuk user info
- [ ] `AdminLayout` component terdaftar dan bisa digunakan sebagai `<x-admin-layout>`
- [ ] `DashboardController` dibuat di `app/Http/Controllers/Admin/`
- [ ] Route group `admin.*` menggunakan middleware `auth` + `verified`
- [ ] Redirect setelah login mengarah ke `/admin/dashboard`
- [ ] Route `/dashboard` lama dari Breeze sudah dihapus
- [ ] User seeder berjalan — bisa login dengan `admin@admin.com`
- [ ] Flow login → dashboard → profil → logout berfungsi end-to-end
- [ ] Dashboard responsif di semua breakpoint

---

## Ringkasan Struktur File

```
app/
├── Http/
│   └── Controllers/
│       └── Admin/
│           └── DashboardController.php    # Dashboard controller
├── View/
│   └── Components/
│       └── AdminLayout.php                # Admin layout component class

resources/views/
├── layouts/
│   ├── admin.blade.php                    # Layout utama admin (sidebar + navbar + footer)
│   ├── admin-sidebar.blade.php            # Sidebar navigation admin
│   ├── admin-navbar.blade.php             # Top navbar admin
│   ├── app.blade.php                      # Layout Breeze (untuk auth/profil)
│   └── guest.blade.php                    # Layout Breeze guest (login/register)
├── components/
│   └── admin-layout.blade.php             # Layout component blade (auto-generated)
├── admin/
│   └── dashboard.blade.php                # Halaman dashboard (pakai data dari Controller)
├── partials/
│   └── alert.blade.php                    # Flash message (dari workflow 02)
└── auth/                                  # View auth dari Breeze (login, register, dll)

routes/
├── web.php                                # Route utama (public + admin group + profil)
└── auth.php                               # Route autentikasi Breeze

database/seeders/
└── DatabaseSeeder.php                     # Seeder dengan admin user testing
```

---

## Diagram Alur Autentikasi

```
User mengakses URL
        │
        ▼
┌───────────────────┐
│  Route Middleware  │
│  auth + verified   │
├───────────────────┤
│  Sudah login?     │
└────────┬──────────┘
         │
    ┌────┴────┐
    │         │
   YES       NO
    │         │
    ▼         ▼
┌────────┐  ┌──────────────┐
│  Admin │  │ Redirect ke  │
│Dashboard│  │ /login       │
└────────┘  └──────┬───────┘
                   │
                   ▼
            ┌──────────────┐
            │ Login Form   │
            │ (Breeze)     │
            └──────┬───────┘
                   │ Submit
                   ▼
            ┌──────────────┐
            │ Validasi     │
            │ Credentials  │
            └──────┬───────┘
                   │
              ┌────┴────┐
              │         │
            Valid    Invalid
              │         │
              ▼         ▼
        ┌──────────┐  ┌──────────┐
        │ Redirect │  │ Error    │
        │ /admin/  │  │ kembali  │
        │ dashboard│  │ ke login │
        └──────────┘  └──────────┘
```

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **Pastikan Breeze tidak merusak setup workflow 02.** Selalu cek dan restore `app.css`, `tailwind.config.js`, dan `app.js` setelah install Breeze.
> 2. **Jangan campur layout admin dengan layout Breeze.** Gunakan `<x-admin-layout>` untuk halaman admin, dan biarkan `<x-app-layout>` / `<x-guest-layout>` untuk halaman auth/profil bawaan Breeze.
> 3. **Redirect setelah login HARUS ke `/admin/dashboard`**, bukan `/dashboard`. Cek semua controller auth.
> 4. **Selalu test flow lengkap**: register → login → dashboard → profil → logout → login lagi.
> 5. **Sidebar harus menampilkan route `admin.*`** yang aktif dengan class `sidebar-link-active`. Tambahkan menu baru di sidebar setiap kali route admin baru dibuat.
> 6. **Credential seeder default**: `admin@admin.com` / `password`. Tanyakan ke user jika ingin berbeda.
