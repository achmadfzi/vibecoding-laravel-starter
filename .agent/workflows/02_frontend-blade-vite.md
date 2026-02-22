---
description: SOP untuk setup frontend Laravel menggunakan Blade templating, Vite bundler, dan Tailwind CSS — termasuk layout, komponen UI, dan halaman admin dashboard.
---

# 02 — Frontend Blade + Vite Setup

> **Tujuan**: Menyiapkan frontend Laravel menggunakan Blade templating engine dengan Vite sebagai bundler dan Tailwind CSS sebagai framework styling. Termasuk pembuatan layout utama, komponen UI reusable, dan halaman admin dashboard statis yang responsif.

---

## Prasyarat

- Laravel sudah terinstall (lihat `01_laravel-backend-setup.md`)
- Node.js 18+ dan npm sudah tersedia

// turbo
```bash
node -v && npm -v
```

> [!IMPORTANT]
> Jika Node.js belum terinstall, **hentikan proses** dan minta user untuk menginstall terlebih dahulu.

---

## Step 1 — Install Dependencies (Tailwind CSS + Vite Plugin)

### 1.1 — Install npm dependencies

// turbo
```bash
npm install
```

### 1.2 — Install Tailwind CSS v3 dan dependencies-nya

// turbo
```bash
npm install -D tailwindcss@3 postcss autoprefixer @tailwindcss/forms @tailwindcss/typography
```

### 1.3 — Inisialisasi konfigurasi Tailwind & PostCSS

// turbo
```bash
npx tailwindcss init -p
```

Ini akan membuat dua file:
- `tailwind.config.js`
- `postcss.config.js`

> [!NOTE]
> Jika user menginginkan Tailwind CSS v4, sesuaikan instalasi dan konfigurasi. Secara default, workflow ini menggunakan **Tailwind CSS v3** karena lebih stabil dan dokumentasinya lebih lengkap. Tanyakan ke user jika ragu.

---

## Step 2 — Konfigurasi `tailwind.config.js`

Buka dan edit `tailwind.config.js` menjadi:

```js
/** @type {import('tailwindcss').Config} */
export default {
    content: [
        "./resources/**/*.blade.php",
        "./resources/**/*.js",
        "./resources/**/*.vue",
    ],
    theme: {
        extend: {
            colors: {
                primary: {
                    50:  '#eff6ff',
                    100: '#dbeafe',
                    200: '#bfdbfe',
                    300: '#93c5fd',
                    400: '#60a5fa',
                    500: '#3b82f6',
                    600: '#2563eb',
                    700: '#1d4ed8',
                    800: '#1e40af',
                    900: '#1e3a8a',
                    950: '#172554',
                },
                secondary: {
                    50:  '#f8fafc',
                    100: '#f1f5f9',
                    200: '#e2e8f0',
                    300: '#cbd5e1',
                    400: '#94a3b8',
                    500: '#64748b',
                    600: '#475569',
                    700: '#334155',
                    800: '#1e293b',
                    900: '#0f172a',
                    950: '#020617',
                },
            },
            fontFamily: {
                sans: ['Inter', 'ui-sans-serif', 'system-ui', 'sans-serif'],
            },
        },
    },
    plugins: [
        require('@tailwindcss/forms'),
        require('@tailwindcss/typography'),
    ],
};
```

> [!NOTE]
> - Warna `primary` dan `secondary` bisa disesuaikan sesuai branding user. Tanyakan preferensi warna.
> - Font `Inter` digunakan sebagai default. Akan di-load via Google Fonts di layout.

---

## Step 3 — Konfigurasi `vite.config.js`

Buka dan edit `vite.config.js` menjadi:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: [
                'resources/css/app.css',
                'resources/js/app.js',
            ],
            refresh: true,
        }),
    ],
    server: {
        hmr: {
            host: 'localhost',
        },
    },
});
```

> [!NOTE]
> `refresh: true` mengaktifkan auto-reload saat file Blade berubah — sangat berguna saat development.

---

## Step 4 — Setup CSS Entry Point

Buka `resources/css/app.css` dan ganti isinya dengan:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* ============================================
   Custom Base Styles
   ============================================ */
@layer base {
    body {
        @apply font-sans antialiased text-secondary-800 bg-gray-50;
    }

    h1 { @apply text-3xl font-bold tracking-tight; }
    h2 { @apply text-2xl font-semibold tracking-tight; }
    h3 { @apply text-xl font-semibold; }
    h4 { @apply text-lg font-medium; }
}

/* ============================================
   Reusable Component Classes
   ============================================ */
@layer components {
    /* Buttons */
    .btn {
        @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg
               transition-all duration-200 ease-in-out focus:outline-none focus:ring-2 focus:ring-offset-2
               disabled:opacity-50 disabled:cursor-not-allowed;
    }

    .btn-primary {
        @apply btn bg-primary-600 text-white hover:bg-primary-700 focus:ring-primary-500
               shadow-sm hover:shadow-md;
    }

    .btn-secondary {
        @apply btn bg-white text-secondary-700 border border-secondary-300
               hover:bg-secondary-50 focus:ring-secondary-500;
    }

    .btn-danger {
        @apply btn bg-red-600 text-white hover:bg-red-700 focus:ring-red-500;
    }

    .btn-sm {
        @apply px-3 py-1.5 text-xs;
    }

    .btn-lg {
        @apply px-6 py-3 text-base;
    }

    /* Cards */
    .card {
        @apply bg-white rounded-xl shadow-sm border border-secondary-200 overflow-hidden;
    }

    .card-body {
        @apply p-6;
    }

    .card-header {
        @apply px-6 py-4 border-b border-secondary-200 bg-secondary-50/50;
    }

    /* Form Inputs */
    .form-input {
        @apply block w-full rounded-lg border-secondary-300 shadow-sm
               focus:border-primary-500 focus:ring-primary-500 text-sm
               placeholder:text-secondary-400;
    }

    .form-label {
        @apply block text-sm font-medium text-secondary-700 mb-1;
    }

    .form-error {
        @apply text-sm text-red-600 mt-1;
    }

    /* Badges */
    .badge {
        @apply inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium;
    }

    .badge-success { @apply badge bg-green-100 text-green-800; }
    .badge-warning { @apply badge bg-yellow-100 text-yellow-800; }
    .badge-danger  { @apply badge bg-red-100 text-red-800; }
    .badge-info    { @apply badge bg-blue-100 text-blue-800; }

    /* Sidebar */
    .sidebar-link {
        @apply flex items-center gap-3 px-3 py-2.5 text-sm font-medium rounded-lg
               transition-colors duration-150;
    }

    .sidebar-link-active {
        @apply sidebar-link bg-primary-50 text-primary-700;
    }

    .sidebar-link-inactive {
        @apply sidebar-link text-secondary-600 hover:bg-secondary-100 hover:text-secondary-900;
    }
}

/* ============================================
   Utility Extensions
   ============================================ */
@layer utilities {
    .text-gradient {
        @apply bg-clip-text text-transparent bg-gradient-to-r from-primary-600 to-primary-400;
    }
}
```

---

## Step 5 — Setup JavaScript Entry Point

Buka `resources/js/app.js` dan ganti isinya dengan:

```js
import '../css/app.css';

// ============================================
// Alpine.js (Opsional, sangat direkomendasikan untuk interaktivitas Blade)
// ============================================
// Jika ingin menggunakan Alpine.js, uncomment baris berikut:
// import Alpine from 'alpinejs';
// window.Alpine = Alpine;
// Alpine.start();

// ============================================
// Global Scripts
// ============================================

// Mobile sidebar toggle
document.addEventListener('DOMContentLoaded', () => {
    const sidebarToggle = document.getElementById('sidebar-toggle');
    const sidebar = document.getElementById('sidebar');
    const sidebarOverlay = document.getElementById('sidebar-overlay');

    if (sidebarToggle && sidebar) {
        sidebarToggle.addEventListener('click', () => {
            sidebar.classList.toggle('-translate-x-full');
            sidebarOverlay?.classList.toggle('hidden');
        });

        sidebarOverlay?.addEventListener('click', () => {
            sidebar.classList.add('-translate-x-full');
            sidebarOverlay.classList.add('hidden');
        });
    }
});

console.log('App initialized.');
```

> [!TIP]
> **Alpine.js** sangat direkomendasikan untuk interaktivitas ringan (dropdown, modal, toggle) tanpa perlu framework SPA. Install dengan:
> ```bash
> npm install alpinejs
> ```
> Lalu uncomment baris Alpine.js di `app.js`.

---

## Step 6 — Buat Struktur Folder Blade

### 6.1 — Buat folder-folder yang dibutuhkan

// turbo
```powershell
New-Item -ItemType Directory -Force -Path "resources/views/layouts", "resources/views/components", "resources/views/admin", "resources/views/partials", "resources/views/auth"
```

### 6.2 — Penjelasan struktur folder

| Folder                       | Fungsi                                                      |
|-----------------------------|-------------------------------------------------------------|
| `resources/views/layouts/`  | Layout utama Blade (app.blade.php, guest.blade.php)         |
| `resources/views/components/` | Blade components reusable (header, footer, sidebar, dll)  |
| `resources/views/admin/`    | Halaman-halaman admin (dashboard, settings, dll)            |
| `resources/views/partials/` | Partial views (alert, pagination, breadcrumb, dll)          |
| `resources/views/auth/`     | Halaman autentikasi (login, register, dll)                  |

---

## Step 7 — Buat Layout Utama (`app.blade.php`)

Buat file `resources/views/layouts/app.blade.php`:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? config('app.name', 'Laravel') }}</title>

    {{-- Google Fonts --}}
    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=inter:300,400,500,600,700,800" rel="stylesheet" />

    {{-- Vite Assets --}}
    @vite(['resources/css/app.css', 'resources/js/app.js'])

    {{-- Additional Head --}}
    @stack('styles')
</head>
<body class="min-h-screen bg-gray-50">

    <div class="flex min-h-screen">
        {{-- Sidebar --}}
        <x-sidebar />

        {{-- Main Content Area --}}
        <div class="flex-1 flex flex-col lg:ml-64">
            {{-- Header --}}
            <x-header />

            {{-- Page Content --}}
            <main class="flex-1 p-4 sm:p-6 lg:p-8">
                {{-- Breadcrumb (opsional) --}}
                @hasSection('breadcrumb')
                    <nav class="mb-6">
                        @yield('breadcrumb')
                    </nav>
                @endif

                {{-- Flash Messages --}}
                @include('partials.alert')

                {{-- Page Header --}}
                @hasSection('page-header')
                    <div class="mb-6">
                        @yield('page-header')
                    </div>
                @endif

                {{-- Main Slot --}}
                {{ $slot }}
            </main>

            {{-- Footer --}}
            <x-footer />
        </div>
    </div>

    {{-- Mobile Sidebar Overlay --}}
    <div id="sidebar-overlay" class="fixed inset-0 bg-black/50 z-30 hidden lg:hidden"></div>

    @stack('scripts')
</body>
</html>
```

> [!IMPORTANT]
> Layout ini menggunakan **Blade component syntax** (`<x-sidebar />`, `<x-header />`, `<x-footer />`).
> Layout menggunakan `{{ $slot }}` (component-based), bukan `@yield('content')`.
> Halaman yang menggunakan layout ini harus dibungkus dengan `<x-layouts.app>`.

---

## Step 8 — Buat Guest Layout (`guest.blade.php`)

Buat file `resources/views/layouts/guest.blade.php`:

```blade
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>{{ $title ?? config('app.name', 'Laravel') }}</title>

    <link rel="preconnect" href="https://fonts.bunny.net">
    <link href="https://fonts.bunny.net/css?family=inter:300,400,500,600,700,800" rel="stylesheet" />

    @vite(['resources/css/app.css', 'resources/js/app.js'])
    @stack('styles')
</head>
<body class="min-h-screen bg-gray-50 flex items-center justify-center p-4">
    <div class="w-full max-w-md">
        {{ $slot }}
    </div>

    @stack('scripts')
</body>
</html>
```

> [!NOTE]
> Layout ini digunakan untuk halaman publik yang tidak memerlukan sidebar/header (seperti login, register, forgot password).

---

## Step 9 — Buat Komponen UI Blade

### 9.1 — Sidebar Component

Buat file `resources/views/components/sidebar.blade.php`:

```blade
{{-- Sidebar --}}
<aside id="sidebar"
       class="fixed inset-y-0 left-0 z-40 w-64 bg-white border-r border-secondary-200
              transform -translate-x-full lg:translate-x-0 transition-transform duration-300 ease-in-out">

    {{-- Logo / Brand --}}
    <div class="flex items-center gap-3 px-6 h-16 border-b border-secondary-200">
        <div class="w-8 h-8 bg-primary-600 rounded-lg flex items-center justify-center">
            <span class="text-white font-bold text-sm">L</span>
        </div>
        <span class="text-lg font-bold text-secondary-900">{{ config('app.name') }}</span>
    </div>

    {{-- Navigation --}}
    <nav class="p-4 space-y-1 overflow-y-auto h-[calc(100vh-4rem)]">
        {{-- Main Menu --}}
        <p class="px-3 pt-4 pb-2 text-xs font-semibold text-secondary-400 uppercase tracking-wider">
            Menu Utama
        </p>

        <a href="{{ route('admin.dashboard') }}"
           class="{{ request()->routeIs('admin.dashboard') ? 'sidebar-link-active' : 'sidebar-link-inactive' }}">
            {{-- Icon: Dashboard --}}
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-4 0a1 1 0 01-1-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 01-1 1h-2z" />
            </svg>
            Dashboard
        </a>

        {{-- Placeholder menu items — tambahkan sesuai kebutuhan --}}
        {{--
        <a href="#" class="sidebar-link-inactive">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.25 2.25 0 11-4.5 0 2.25 2.25 0 014.5 0z" />
            </svg>
            Users
        </a>
        --}}

        {{-- Settings Section --}}
        <p class="px-3 pt-6 pb-2 text-xs font-semibold text-secondary-400 uppercase tracking-wider">
            Pengaturan
        </p>

        <a href="#" class="sidebar-link-inactive">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.066 2.573c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.573 1.066c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.066-2.573c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
            </svg>
            Settings
        </a>
    </nav>
</aside>
```

### 9.2 — Header Component

Buat file `resources/views/components/header.blade.php`:

```blade
{{-- Top Header Bar --}}
<header class="sticky top-0 z-20 bg-white/80 backdrop-blur-md border-b border-secondary-200">
    <div class="flex items-center justify-between h-16 px-4 sm:px-6 lg:px-8">
        {{-- Left: Mobile menu toggle + Page title --}}
        <div class="flex items-center gap-4">
            {{-- Mobile Sidebar Toggle --}}
            <button id="sidebar-toggle"
                    class="lg:hidden p-2 rounded-lg text-secondary-500 hover:bg-secondary-100 hover:text-secondary-700
                           transition-colors focus:outline-none focus:ring-2 focus:ring-primary-500">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
                </svg>
            </button>

            {{-- Page Title --}}
            <h1 class="text-lg font-semibold text-secondary-900 hidden sm:block">
                {{ $title ?? '' }}
            </h1>
        </div>

        {{-- Right: Actions --}}
        <div class="flex items-center gap-3">
            {{-- Notifications --}}
            <button class="relative p-2 rounded-lg text-secondary-500 hover:bg-secondary-100 hover:text-secondary-700
                           transition-colors focus:outline-none focus:ring-2 focus:ring-primary-500">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                          d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9" />
                </svg>
                {{-- Notification badge --}}
                {{-- <span class="absolute top-1.5 right-1.5 w-2 h-2 bg-red-500 rounded-full"></span> --}}
            </button>

            {{-- User Dropdown --}}
            <div class="flex items-center gap-3 pl-3 border-l border-secondary-200">
                <div class="w-8 h-8 rounded-full bg-primary-100 flex items-center justify-center">
                    <span class="text-sm font-semibold text-primary-700">
                        {{ substr(auth()->user()->name ?? 'A', 0, 1) }}
                    </span>
                </div>
                <div class="hidden sm:block">
                    <p class="text-sm font-medium text-secondary-900">{{ auth()->user()->name ?? 'Admin' }}</p>
                    <p class="text-xs text-secondary-500">{{ auth()->user()->email ?? 'admin@app.com' }}</p>
                </div>
            </div>
        </div>
    </div>
</header>
```

### 9.3 — Footer Component

Buat file `resources/views/components/footer.blade.php`:

```blade
{{-- Footer --}}
<footer class="border-t border-secondary-200 bg-white">
    <div class="px-4 sm:px-6 lg:px-8 py-4">
        <div class="flex flex-col sm:flex-row items-center justify-between gap-2">
            <p class="text-sm text-secondary-500">
                &copy; {{ date('Y') }} {{ config('app.name') }}. All rights reserved.
            </p>
            <p class="text-xs text-secondary-400">
                Built with Laravel & Tailwind CSS
            </p>
        </div>
    </div>
</footer>
```

### 9.4 — Alert Partial

Buat file `resources/views/partials/alert.blade.php`:

```blade
{{-- Flash Message Alerts --}}
@if (session('success'))
    <div class="mb-4 p-4 rounded-lg bg-green-50 border border-green-200 text-green-800 text-sm flex items-center gap-3"
         role="alert">
        <svg class="w-5 h-5 text-green-500 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        {{ session('success') }}
    </div>
@endif

@if (session('error'))
    <div class="mb-4 p-4 rounded-lg bg-red-50 border border-red-200 text-red-800 text-sm flex items-center gap-3"
         role="alert">
        <svg class="w-5 h-5 text-red-500 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        {{ session('error') }}
    </div>
@endif

@if (session('warning'))
    <div class="mb-4 p-4 rounded-lg bg-yellow-50 border border-yellow-200 text-yellow-800 text-sm flex items-center gap-3"
         role="alert">
        <svg class="w-5 h-5 text-yellow-500 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
        </svg>
        {{ session('warning') }}
    </div>
@endif

{{-- Validation Errors --}}
@if ($errors->any())
    <div class="mb-4 p-4 rounded-lg bg-red-50 border border-red-200 text-red-800 text-sm" role="alert">
        <p class="font-medium mb-2">Terdapat {{ $errors->count() }} kesalahan:</p>
        <ul class="list-disc list-inside space-y-1">
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

---

## Step 10 — Buat Halaman Admin Dashboard

### 10.1 — Buat file dashboard view

Buat file `resources/views/admin/dashboard.blade.php`:

```blade
<x-layouts.app title="Dashboard">
    {{-- Page Header --}}
    @section('page-header')
        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
            <div>
                <h2 class="text-2xl font-bold text-secondary-900">Dashboard</h2>
                <p class="text-sm text-secondary-500 mt-1">Selamat datang kembali, {{ auth()->user()->name ?? 'Admin' }}!</p>
            </div>
            <div class="flex items-center gap-2">
                <button class="btn-secondary btn-sm">
                    <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4" />
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
    @endsection

    {{-- Stats Cards --}}
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-6 mb-6 sm:mb-8">
        {{-- Card: Total Users --}}
        <div class="card">
            <div class="card-body">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-sm font-medium text-secondary-500">Total Users</p>
                        <p class="text-2xl sm:text-3xl font-bold text-secondary-900 mt-1">1,248</p>
                        <p class="text-xs text-green-600 mt-2 flex items-center gap-1">
                            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                            </svg>
                            +12.5% dari bulan lalu
                        </p>
                    </div>
                    <div class="w-12 h-12 bg-blue-100 rounded-xl flex items-center justify-center">
                        <svg class="w-6 h-6 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                  d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.25 2.25 0 11-4.5 0 2.25 2.25 0 014.5 0z" />
                        </svg>
                    </div>
                </div>
            </div>
        </div>

        {{-- Card: Revenue --}}
        <div class="card">
            <div class="card-body">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-sm font-medium text-secondary-500">Pendapatan</p>
                        <p class="text-2xl sm:text-3xl font-bold text-secondary-900 mt-1">Rp 45,2jt</p>
                        <p class="text-xs text-green-600 mt-2 flex items-center gap-1">
                            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 10l7-7m0 0l7 7m-7-7v18" />
                            </svg>
                            +8.2% dari bulan lalu
                        </p>
                    </div>
                    <div class="w-12 h-12 bg-green-100 rounded-xl flex items-center justify-center">
                        <svg class="w-6 h-6 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                  d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                        </svg>
                    </div>
                </div>
            </div>
        </div>

        {{-- Card: Orders --}}
        <div class="card">
            <div class="card-body">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-sm font-medium text-secondary-500">Total Orders</p>
                        <p class="text-2xl sm:text-3xl font-bold text-secondary-900 mt-1">342</p>
                        <p class="text-xs text-red-600 mt-2 flex items-center gap-1">
                            <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3" />
                            </svg>
                            -3.1% dari bulan lalu
                        </p>
                    </div>
                    <div class="w-12 h-12 bg-purple-100 rounded-xl flex items-center justify-center">
                        <svg class="w-6 h-6 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                  d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
                        </svg>
                    </div>
                </div>
            </div>
        </div>

        {{-- Card: Active Sessions --}}
        <div class="card">
            <div class="card-body">
                <div class="flex items-center justify-between">
                    <div>
                        <p class="text-sm font-medium text-secondary-500">Active Sessions</p>
                        <p class="text-2xl sm:text-3xl font-bold text-secondary-900 mt-1">89</p>
                        <p class="text-xs text-secondary-500 mt-2">Real-time</p>
                    </div>
                    <div class="w-12 h-12 bg-orange-100 rounded-xl flex items-center justify-center">
                        <svg class="w-6 h-6 text-orange-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                  d="M13 10V3L4 14h7v7l9-11h-7z" />
                        </svg>
                    </div>
                </div>
            </div>
        </div>
    </div>

    {{-- Content Grid --}}
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-4 sm:gap-6">
        {{-- Recent Activity Table --}}
        <div class="lg:col-span-2 card">
            <div class="card-header">
                <div class="flex items-center justify-between">
                    <h3 class="text-base font-semibold text-secondary-900">Aktivitas Terbaru</h3>
                    <a href="#" class="text-sm text-primary-600 hover:text-primary-700 font-medium">Lihat Semua</a>
                </div>
            </div>
            <div class="overflow-x-auto">
                <table class="w-full text-sm">
                    <thead>
                        <tr class="border-b border-secondary-200">
                            <th class="text-left px-6 py-3 text-xs font-semibold text-secondary-500 uppercase tracking-wider">User</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-secondary-500 uppercase tracking-wider">Aksi</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-secondary-500 uppercase tracking-wider hidden sm:table-cell">Status</th>
                            <th class="text-left px-6 py-3 text-xs font-semibold text-secondary-500 uppercase tracking-wider hidden md:table-cell">Waktu</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-secondary-100">
                        <tr class="hover:bg-secondary-50 transition-colors">
                            <td class="px-6 py-4">
                                <div class="flex items-center gap-3">
                                    <div class="w-8 h-8 rounded-full bg-primary-100 flex items-center justify-center">
                                        <span class="text-xs font-semibold text-primary-700">AB</span>
                                    </div>
                                    <span class="font-medium text-secondary-900">Ahmad Budi</span>
                                </div>
                            </td>
                            <td class="px-6 py-4 text-secondary-600">Membuat pesanan baru</td>
                            <td class="px-6 py-4 hidden sm:table-cell"><span class="badge-success">Selesai</span></td>
                            <td class="px-6 py-4 text-secondary-500 hidden md:table-cell">2 menit lalu</td>
                        </tr>
                        <tr class="hover:bg-secondary-50 transition-colors">
                            <td class="px-6 py-4">
                                <div class="flex items-center gap-3">
                                    <div class="w-8 h-8 rounded-full bg-green-100 flex items-center justify-center">
                                        <span class="text-xs font-semibold text-green-700">CS</span>
                                    </div>
                                    <span class="font-medium text-secondary-900">Citra Sari</span>
                                </div>
                            </td>
                            <td class="px-6 py-4 text-secondary-600">Update profil</td>
                            <td class="px-6 py-4 hidden sm:table-cell"><span class="badge-info">Proses</span></td>
                            <td class="px-6 py-4 text-secondary-500 hidden md:table-cell">15 menit lalu</td>
                        </tr>
                        <tr class="hover:bg-secondary-50 transition-colors">
                            <td class="px-6 py-4">
                                <div class="flex items-center gap-3">
                                    <div class="w-8 h-8 rounded-full bg-orange-100 flex items-center justify-center">
                                        <span class="text-xs font-semibold text-orange-700">DR</span>
                                    </div>
                                    <span class="font-medium text-secondary-900">Doni Rahmad</span>
                                </div>
                            </td>
                            <td class="px-6 py-4 text-secondary-600">Pembayaran gagal</td>
                            <td class="px-6 py-4 hidden sm:table-cell"><span class="badge-danger">Gagal</span></td>
                            <td class="px-6 py-4 text-secondary-500 hidden md:table-cell">1 jam lalu</td>
                        </tr>
                        <tr class="hover:bg-secondary-50 transition-colors">
                            <td class="px-6 py-4">
                                <div class="flex items-center gap-3">
                                    <div class="w-8 h-8 rounded-full bg-purple-100 flex items-center justify-center">
                                        <span class="text-xs font-semibold text-purple-700">EF</span>
                                    </div>
                                    <span class="font-medium text-secondary-900">Eka Fitria</span>
                                </div>
                            </td>
                            <td class="px-6 py-4 text-secondary-600">Register akun baru</td>
                            <td class="px-6 py-4 hidden sm:table-cell"><span class="badge-warning">Pending</span></td>
                            <td class="px-6 py-4 text-secondary-500 hidden md:table-cell">2 jam lalu</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        {{-- Quick Stats / Info Panel --}}
        <div class="space-y-4 sm:space-y-6">
            {{-- Quick Links --}}
            <div class="card">
                <div class="card-header">
                    <h3 class="text-base font-semibold text-secondary-900">Aksi Cepat</h3>
                </div>
                <div class="card-body space-y-3">
                    <a href="#" class="flex items-center gap-3 p-3 rounded-lg hover:bg-secondary-50 transition-colors group">
                        <div class="w-10 h-10 bg-primary-100 rounded-lg flex items-center justify-center group-hover:bg-primary-200 transition-colors">
                            <svg class="w-5 h-5 text-primary-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm font-medium text-secondary-900">Tambah User</p>
                            <p class="text-xs text-secondary-500">Buat akun pengguna baru</p>
                        </div>
                    </a>
                    <a href="#" class="flex items-center gap-3 p-3 rounded-lg hover:bg-secondary-50 transition-colors group">
                        <div class="w-10 h-10 bg-green-100 rounded-lg flex items-center justify-center group-hover:bg-green-200 transition-colors">
                            <svg class="w-5 h-5 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                      d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm font-medium text-secondary-900">Lihat Laporan</p>
                            <p class="text-xs text-secondary-500">Laporan bulan ini</p>
                        </div>
                    </a>
                    <a href="#" class="flex items-center gap-3 p-3 rounded-lg hover:bg-secondary-50 transition-colors group">
                        <div class="w-10 h-10 bg-purple-100 rounded-lg flex items-center justify-center group-hover:bg-purple-200 transition-colors">
                            <svg class="w-5 h-5 text-purple-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                      d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.066 2.573c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.573 1.066c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.066-2.573c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" />
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm font-medium text-secondary-900">Pengaturan</p>
                            <p class="text-xs text-secondary-500">Konfigurasi aplikasi</p>
                        </div>
                    </a>
                </div>
            </div>

            {{-- System Info --}}
            <div class="card">
                <div class="card-header">
                    <h3 class="text-base font-semibold text-secondary-900">Info Sistem</h3>
                </div>
                <div class="card-body space-y-3">
                    <div class="flex items-center justify-between">
                        <span class="text-sm text-secondary-500">Laravel</span>
                        <span class="text-sm font-medium text-secondary-900">{{ app()->version() }}</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-sm text-secondary-500">PHP</span>
                        <span class="text-sm font-medium text-secondary-900">{{ phpversion() }}</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-sm text-secondary-500">Environment</span>
                        <span class="badge-info">{{ app()->environment() }}</span>
                    </div>
                    <div class="flex items-center justify-between">
                        <span class="text-sm text-secondary-500">Server Time</span>
                        <span class="text-sm font-medium text-secondary-900">{{ now()->format('d M Y, H:i') }}</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-layouts.app>
```

---

## Step 11 — Setup Route untuk Dashboard

### 11.1 — Buat route di `routes/web.php`

Tambahkan route berikut:

```php
use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return redirect()->route('admin.dashboard');
});

Route::prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', function () {
        return view('admin.dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> Route ini belum menggunakan middleware auth. Saat mengintegrasikan autentikasi nanti, tambahkan middleware `auth` ke route group admin:
> ```php
> Route::prefix('admin')->name('admin.')->middleware('auth')->group(function () {
>     // ...
> });
> ```

---

## Step 12 — Build & Verifikasi

### 12.1 — Build assets via Vite (development mode)

// turbo
```bash
npm run dev
```

> Biarkan proses ini berjalan di terminal terpisah. Vite akan melakukan hot-reload.

### 12.2 — Jalankan Laravel server (terminal terpisah)

// turbo
```bash
php artisan serve
```

### 12.3 — Buka browser

Buka `http://localhost:8000` dan pastikan:
- Dashboard tampil dengan benar
- Sidebar tampil di desktop, tersembunyi di mobile
- Header sticky di atas
- Cards statistik tampil grid responsif (1 kolom di mobile, 2 di tablet, 4 di desktop)
- Tabel aktivitas scrollable horizontal di layar kecil

### 12.4 — Cek responsivitas

Gunakan browser DevTools (F12) dan test pada breakpoint berikut:

| Breakpoint | Lebar  | Yang Harus Dicek                                            |
|-----------|--------|-------------------------------------------------------------|
| Mobile    | 375px  | Sidebar hidden, toggle button tampil, cards 1 kolom         |
| Tablet    | 768px  | Cards 2 kolom, tabel partial column hidden                  |
| Desktop   | 1024px | Sidebar visible, cards 4 kolom, semua kolom tabel tampil    |
| Wide      | 1280px | Layout tetap proporsional, tidak stretch berlebihan          |

---

## Step 13 — Build Production (Opsional)

Jika ingin build untuk production:

// turbo
```bash
npm run build
```

Verifikasi folder `public/build/` terisi dengan file CSS dan JS yang sudah di-minify.

---

## Checklist Verifikasi Akhir

- [ ] Tailwind CSS terinstall dan terkonfigurasi (`tailwind.config.js`)
- [ ] Vite terkonfigurasi (`vite.config.js`) dengan `refresh: true`
- [ ] `resources/css/app.css` berisi directive Tailwind + custom component classes
- [ ] `resources/js/app.js` berisi script sidebar toggle
- [ ] Folder Blade terstruktur: `layouts/`, `components/`, `admin/`, `partials/`
- [ ] Layout `app.blade.php` menggunakan component syntax (`<x-sidebar />`)
- [ ] Layout `guest.blade.php` tersedia untuk halaman publik
- [ ] Komponen `sidebar.blade.php` — responsif, collapsible di mobile
- [ ] Komponen `header.blade.php` — sticky, mobile toggle, user info
- [ ] Komponen `footer.blade.php` — sederhana, responsif
- [ ] Partial `alert.blade.php` — support success/error/warning/validation
- [ ] Halaman `admin/dashboard.blade.php` — stat cards + activity table + quick actions
- [ ] Route `/admin/dashboard` terdaftar dan bisa diakses
- [ ] `npm run dev` berjalan tanpa error
- [ ] Tampilan responsif di semua breakpoint (375px, 768px, 1024px, 1280px)

---

## Ringkasan Struktur File yang Dibuat

```
resources/
├── css/
│   └── app.css                        # Tailwind directives + custom components
├── js/
│   └── app.js                         # JS entry point + sidebar toggle
└── views/
    ├── layouts/
    │   ├── app.blade.php              # Layout utama (sidebar + header + footer)
    │   └── guest.blade.php            # Layout publik (login, register)
    ├── components/
    │   ├── sidebar.blade.php          # Sidebar navigation
    │   ├── header.blade.php           # Top header bar
    │   └── footer.blade.php           # Footer
    ├── partials/
    │   └── alert.blade.php            # Flash message alerts
    ├── admin/
    │   └── dashboard.blade.php        # Admin dashboard page
    └── auth/                          # (Disiapkan untuk halaman auth)

tailwind.config.js                     # Tailwind configuration
vite.config.js                         # Vite bundler configuration
postcss.config.js                      # PostCSS configuration (auto-generated)
```

---

## Panduan Responsivitas untuk AI Agent

> [!WARNING]
> **WAJIB** diikuti saat membuat atau memodifikasi komponen Blade.

### Prinsip Utama

1. **Mobile-first**: Selalu mulai styling dari layar kecil, lalu tambahkan breakpoint (`sm:`, `md:`, `lg:`, `xl:`)
2. **Jangan hardcode width**: Gunakan `w-full`, `max-w-*`, atau `flex-1` alih-alih `w-[500px]`
3. **Grid responsif**: Gunakan pattern `grid-cols-1 sm:grid-cols-2 lg:grid-cols-4`
4. **Tabel scrollable**: Selalu bungkus `<table>` dengan `<div class="overflow-x-auto">`
5. **Hidden/Show**: Gunakan `hidden sm:block` atau `sm:hidden` untuk toggle visibility per breakpoint
6. **Padding responsif**: Gunakan `p-4 sm:p-6 lg:p-8` untuk spacing yang proporsional
7. **Font size responsif**: Gunakan `text-2xl sm:text-3xl` untuk heading yang adaptif

### Breakpoint Tailwind CSS

| Prefix | Min Width | Target Device        |
|--------|----------|----------------------|
| (none) | 0px      | Mobile (default)     |
| `sm:`  | 640px    | Small tablet         |
| `md:`  | 768px    | Tablet               |
| `lg:`  | 1024px   | Desktop              |
| `xl:`  | 1280px   | Large desktop        |
| `2xl:` | 1536px   | Wide screen          |

### Checklist per Komponen Baru

Setiap kali membuat komponen atau halaman baru, pastikan:

- [ ] Tampilan tidak overflow horizontal di `375px`
- [ ] Text tidak terlalu kecil atau terpotong di mobile
- [ ] Tombol dan link cukup besar untuk di-tap (min 44x44px touch target)
- [ ] Gambar menggunakan `max-w-full h-auto` atau `object-cover`
- [ ] Form input full-width di mobile (`w-full`)
- [ ] Modal/dialog tidak lebih lebar dari viewport mobile

---

## Catatan Penting untuk AI Agent

> [!CAUTION]
> 1. **Selalu gunakan `@vite` directive** di layout, bukan `<link>` / `<script>` manual.
> 2. **Jangan skip responsivitas**. Setiap komponen HARUS dicek di minimal 3 breakpoint: mobile (375px), tablet (768px), desktop (1024px).
> 3. **Gunakan CSS component classes** yang sudah didefinisikan di `app.css` (`.btn-primary`, `.card`, `.form-input`, dll) — jangan buat class baru dengan styling yang sama.
> 4. **Sidebar harus collapsible** di mobile. Pastikan toggle button dan overlay berfungsi.
> 5. **Jangan buat inline style**. Selalu gunakan Tailwind classes.
> 6. **Biarkan `npm run dev` berjalan** di background saat development untuk hot-reload.
