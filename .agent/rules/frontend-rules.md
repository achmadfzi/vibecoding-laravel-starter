---
description: Aturan coding frontend Laravel yang wajib dipatuhi AI — standar UI styling, struktur komponen Blade, konvensi Tailwind CSS, Vite bundler, dan best practices.
---

# Laravel Frontend Coding Rules

This document outlines the UI styling standards, component structuring, and best practices for the Laravel frontend project using the Blade templating engine, Tailwind CSS, and Vite.

---

## 1. Folder Structure — Organisasi File Frontend

Semua file frontend **WAJIB** mengikuti struktur folder standar berikut:

```
resources/
├── css/
│   └── app.css                   # Entry point CSS (Tailwind directives + custom)
├── js/
│   └── app.js                    # Entry point JS (Alpine.js, global scripts)
└── views/
    ├── layouts/                  # Layout utama (app.blade.php, guest.blade.php)
    ├── components/               # Blade components reusable
    ├── partials/                 # Partial views (alert, pagination, breadcrumb)
    ├── admin/                    # Halaman-halaman admin
    ├── auth/                     # Halaman autentikasi
    └── [feature]/                # Halaman per-fitur (users, products, dll)
```

**Aturan:**
- **JANGAN** meletakkan file view di root `resources/views/` langsung. Selalu gunakan subfolder.
- **Satu fitur = satu subfolder** di `views/`. Contoh: `views/users/`, `views/products/`.
- **Komponen reusable** harus di `views/components/`.
- **Partial views** (fragment kecil yang di-`@include`) harus di `views/partials/`.
- **Layout** hanya boleh di `views/layouts/`.

---

## 2. Blade Templating — Standar Penulisan

### 2.1 — Gunakan Component-Based Layout, BUKAN `@extends`

**WAJIB** gunakan Blade component syntax (`<x-layouts.app>`) sebagai standar layout.

```blade
{{-- ✅ Benar — Component-based layout --}}
<x-layouts.app title="Dashboard">
    <h2>Dashboard</h2>
    <p>Konten halaman...</p>
</x-layouts.app>

{{-- ❌ Salah — @extends legacy --}}
@extends('layouts.app')
@section('content')
    <h2>Dashboard</h2>
@endsection
```

**Pengecualian:** `@section` dan `@yield` hanya boleh digunakan untuk **optional sections** di dalam layout (seperti `breadcrumb`, `page-header`), bukan untuk konten utama.

### 2.2 — Blade Component Naming

| Tipe | Convention | Contoh File | Penggunaan |
|------|-----------|-------------|------------|
| Layout | kebab-case di `layouts/` | `layouts/app.blade.php` | `<x-layouts.app>` |
| Component | kebab-case | `components/stat-card.blade.php` | `<x-stat-card>` |
| Anonymous | kebab-case | `components/form/input.blade.php` | `<x-form.input>` |
| Nested | subfolder + kebab-case | `components/table/header.blade.php` | `<x-table.header>` |

**Aturan:**
- Nama file component selalu **kebab-case** (`stat-card.blade.php`, bukan `StatCard.blade.php`).
- Gunakan **subfolder** untuk mengelompokkan komponen terkait (`components/form/`, `components/table/`).
- **JANGAN** buat component dengan nama yang terlalu generik (`box.blade.php`, `item.blade.php`).

### 2.3 — Props dan Attributes pada Component

```blade
{{-- ✅ Benar — Props didefinisikan dengan jelas --}}
@props([
    'title',
    'subtitle' => null,
    'variant' => 'default',
    'icon' => null,
])

<div {{ $attributes->merge(['class' => 'card']) }}>
    @if($icon)
        <span>{!! $icon !!}</span>
    @endif
    <h3>{{ $title }}</h3>
    @if($subtitle)
        <p class="text-sm text-secondary-500">{{ $subtitle }}</p>
    @endif
    {{ $slot }}
</div>
```

**Aturan:**
- **WAJIB** gunakan `@props` untuk mendefinisikan props di setiap component.
- **WAJIB** berikan default value pada props opsional (`'subtitle' => null`).
- **WAJIB** gunakan `$attributes->merge()` untuk mendukung attribute forwarding.
- **JANGAN** hardcode class di component — gunakan `merge` agar bisa di-override.

### 2.4 — Blade Directives — Penggunaan yang Benar

```blade
{{-- ✅ Output ter-escape (default, SELALU gunakan ini) --}}
{{ $user->name }}

{{-- ⚠️ Output TIDAK ter-escape — hanya untuk HTML yang sudah di-sanitize --}}
{!! $trustedHtml !!}

{{-- ✅ Kondisional --}}
@if($users->isEmpty())
    <p>Tidak ada data.</p>
@endif

{{-- ✅ Loop dengan @forelse (lebih baik daripada @foreach + @if) --}}
@forelse($users as $user)
    <tr>...</tr>
@empty
    <tr><td colspan="5">Tidak ada data.</td></tr>
@endforelse

{{-- ✅ CSRF di setiap form --}}
<form method="POST" action="{{ route('users.store') }}">
    @csrf
    ...
</form>

{{-- ✅ Method spoofing --}}
<form method="POST" action="{{ route('users.destroy', $user) }}">
    @csrf
    @method('DELETE')
    ...
</form>
```

**Aturan:**
- **WAJIB** gunakan `{{ }}` (escaped) untuk semua output. **JANGAN** gunakan `{!! !!}` kecuali untuk SVG icons atau HTML yang sudah di-sanitize.
- **WAJIB** gunakan `@forelse` daripada `@foreach` + `@if(count())` untuk menampilkan empty state.
- **WAJIB** sertakan `@csrf` di setiap form.
- **WAJIB** gunakan `@method('PUT')` / `@method('DELETE')` untuk HTTP method yang sesuai.
- **JANGAN** gunakan inline PHP (`@php ... @endphp`) di Blade kecuali untuk operasi sangat sederhana. Logic harus di Controller/Service.

### 2.5 — Partial Views dan @include

```blade
{{-- ✅ Benar — include partial dengan data eksplisit --}}
@include('partials.alert')
@include('partials.user-row', ['user' => $user, 'showActions' => true])

{{-- ❌ Salah — include tanpa konteks yang jelas --}}
@include('partials.user-row')  {{-- Mengandalkan variabel parent scope --}}
```

**Aturan:**
- **WAJIB** pass data secara eksplisit ke partial jika partial membutuhkan variabel selain global/session.
- **Gunakan component** (`<x-...>`) untuk elemen interaktif/reusable, gunakan **partial** (`@include`) hanya untuk fragment sederhana (alert, pagination).
- **JANGAN** buat partial yang menerima lebih dari 4 parameter — jadikan Blade Component.

---

## 3. Tailwind CSS — Konvensi Styling

### 3.1 — Design System: Gunakan Custom Classes di `app.css`

**WAJIB** definisikan reusable style di `@layer components` pada `app.css`, **JANGAN** tulis class Tailwind yang panjang berulang kali.

```css
/* ✅ Benar — Definisi di app.css */
@layer components {
    .btn-primary {
        @apply inline-flex items-center justify-center px-4 py-2 text-sm font-medium
               rounded-lg bg-primary-600 text-white hover:bg-primary-700
               transition-all duration-200 focus:outline-none focus:ring-2
               focus:ring-offset-2 focus:ring-primary-500
               disabled:opacity-50 disabled:cursor-not-allowed;
    }
}
```

```blade
{{-- ✅ Benar — Pakai custom class --}}
<button class="btn-primary">Simpan</button>

{{-- ❌ Salah — Inline class yang sangat panjang, duplikasi di banyak tempat --}}
<button class="inline-flex items-center justify-center px-4 py-2 text-sm font-medium rounded-lg bg-primary-600 text-white hover:bg-primary-700 transition-all duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary-500 disabled:opacity-50 disabled:cursor-not-allowed">
    Simpan
</button>
```

**Aturan:**
- **Elemen yang digunakan lebih dari 2 kali** harus didefinisikan sebagai custom class di `@layer components`.
- **Jangan duplikasi** class Tailwind yang panjang (>5 utility classes) di banyak tempat.
- **Prefix custom class** berdasarkan jenisnya: `btn-*`, `card-*`, `form-*`, `badge-*`, `sidebar-*`.

### 3.2 — Color System: Gunakan Design Tokens

**WAJIB** gunakan warna dari design system (`primary-*`, `secondary-*`), **JANGAN** hardcode warna Tailwind generik.

```blade
{{-- ✅ Benar — Pakai design token --}}
<button class="bg-primary-600 hover:bg-primary-700 text-white">Simpan</button>
<p class="text-secondary-500">Deskripsi</p>

{{-- ❌ Salah — Hardcode warna generik --}}
<button class="bg-blue-600 hover:bg-blue-700 text-white">Simpan</button>
<p class="text-gray-500">Deskripsi</p>
```

**Aturan:**
- Gunakan `primary-*` untuk warna aksen utama (CTA, link, active state).
- Gunakan `secondary-*` untuk warna netral (teks, border, background).
- Gunakan warna semantik untuk status: `green-*` (success), `red-*` (danger/error), `yellow-*` (warning), `blue-*` (info).
- **JANGAN** gunakan `gray-*` secara langsung. Gunakan `secondary-*` sebagai pengganti.
- **Pengecualian:** `bg-gray-50` diperbolehkan untuk background page utama.

### 3.3 — Responsive Design: Mobile-First

**WAJIB** desain mobile-first. Gunakan breakpoint Tailwind secara progresif.

```blade
{{-- ✅ Benar — Mobile-first: base = mobile, lalu tambahkan untuk layar besar --}}
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-6">
    ...
</div>

<div class="p-4 sm:p-6 lg:p-8">
    ...
</div>

{{-- ✅ Tampilkan/sembunyikan berdasarkan ukuran layar --}}
<span class="hidden sm:block">Teks lengkap</span>
<span class="sm:hidden">Singkat</span>
```

**Aturan:**
- **Base styles** (tanpa prefix) = mobile.
- `sm:` = tablet (≥640px), `md:` = tablet landscape (≥768px), `lg:` = desktop (≥1024px), `xl:` = desktop besar (≥1280px).
- **JANGAN** desain desktop-first lalu override dengan `sm:` override.
- **WAJIB** test tampilan di mobile (≤375px), tablet (768px), dan desktop (1280px+).

### 3.4 — Spacing dan Sizing: Konsistensi

| Konteks | Standard | Contoh |
|---------|----------|--------|
| Padding konten halaman | `p-4 sm:p-6 lg:p-8` | `<main class="p-4 sm:p-6 lg:p-8">` |
| Padding card body | `p-6` | `<div class="card-body">` → `@apply p-6` |
| Gap antar elemen | `gap-4` atau `gap-6` | `<div class="grid gap-4 sm:gap-6">` |
| Margin bottom section | `mb-6` atau `mb-8` | `<div class="mb-6 sm:mb-8">` |
| Border radius | `rounded-lg` atau `rounded-xl` | Card: `rounded-xl`, Button: `rounded-lg` |
| Icon size | `w-4 h-4` (sm), `w-5 h-5` (md), `w-6 h-6` (lg) | Sesuai konteks |

**Aturan:**
- **JANGAN** campur unit spacing yang tidak konsisten (misal `p-3` dan `p-7` dalam satu layout).
- Gunakan kelipatan 4 Tailwind (`4`, `6`, `8`, `12`, `16`) untuk spacing utama.
- **JANGAN** gunakan arbitrary values (`p-[13px]`) kecuali benar-benar diperlukan.

### 3.5 — Typography: Hierarki yang Jelas

```css
/* Sudah didefinisikan di @layer base pada app.css */
h1 { @apply text-3xl font-bold tracking-tight; }
h2 { @apply text-2xl font-semibold tracking-tight; }
h3 { @apply text-xl font-semibold; }
h4 { @apply text-lg font-medium; }
```

**Aturan:**
- **WAJIB** gunakan hierarki heading yang benar (`h1` > `h2` > `h3` > `h4`).
- **HANYA satu `h1`** per halaman (biasanya page title).
- Body text: `text-sm` untuk form/table, `text-base` untuk konten utama.
- Warna teks: `text-secondary-900` (heading), `text-secondary-700` (body), `text-secondary-500` (muted/helper).
- Font: `font-sans` (Inter) — sudah di-set di `@layer base`.

### 3.6 — Transitions dan Animations

```blade
{{-- ✅ Benar — Transisi halus --}}
<button class="btn-primary transition-all duration-200 ease-in-out hover:shadow-md">
    Simpan
</button>

<a class="sidebar-link-inactive transition-colors duration-150">
    Dashboard
</a>

{{-- ❌ Salah — Tanpa transisi (terasa kaku) --}}
<button class="bg-primary-600 hover:bg-primary-700">
    Simpan
</button>
```

**Aturan:**
- **WAJIB** tambahkan `transition-*` pada elemen interaktif (button, link, card hover).
- Durasi standar: `duration-150` (cepat, untuk hover), `duration-200` (normal), `duration-300` (smooth, untuk sidebar/modal).
- **JANGAN** gunakan durasi lebih dari `duration-500` — terasa lambat.
- Gunakan `ease-in-out` sebagai default timing function.

---

## 4. Blade Component Design — Pattern Standar

### 4.1 — Stat Card Component

```blade
{{-- resources/views/components/stat-card.blade.php --}}
@props([
    'title',
    'value',
    'trend' => null,
    'trendUp' => true,
    'icon' => null,
    'iconBg' => 'bg-blue-100',
    'iconColor' => 'text-blue-600',
])

<div class="card">
    <div class="card-body">
        <div class="flex items-center justify-between">
            <div>
                <p class="text-sm font-medium text-secondary-500">{{ $title }}</p>
                <p class="text-2xl sm:text-3xl font-bold text-secondary-900 mt-1">{{ $value }}</p>
                @if($trend)
                    <p class="text-xs {{ $trendUp ? 'text-green-600' : 'text-red-600' }} mt-2 flex items-center gap-1">
                        {{ $trend }}
                    </p>
                @endif
            </div>
            @if($icon)
                <div class="w-12 h-12 {{ $iconBg }} rounded-xl flex items-center justify-center">
                    {!! $icon !!}
                </div>
            @endif
        </div>
    </div>
</div>
```

### 4.2 — Form Input Component

```blade
{{-- resources/views/components/form/input.blade.php --}}
@props([
    'label',
    'name',
    'type' => 'text',
    'value' => null,
    'required' => false,
    'placeholder' => '',
    'hint' => null,
])

<div>
    <label for="{{ $name }}" class="form-label">
        {{ $label }}
        @if($required)
            <span class="text-red-500">*</span>
        @endif
    </label>

    <input
        type="{{ $type }}"
        id="{{ $name }}"
        name="{{ $name }}"
        value="{{ old($name, $value) }}"
        placeholder="{{ $placeholder }}"
        {{ $required ? 'required' : '' }}
        {{ $attributes->merge(['class' => 'form-input' . ($errors->has($name) ? ' border-red-500 focus:border-red-500 focus:ring-red-500' : '')]) }}
    >

    @if($hint && !$errors->has($name))
        <p class="text-xs text-secondary-400 mt-1">{{ $hint }}</p>
    @endif

    @error($name)
        <p class="form-error">{{ $message }}</p>
    @enderror
</div>
```

### 4.3 — Data Table Pattern

```blade
{{-- Pattern standar untuk tabel data --}}
<div class="card">
    <div class="card-header">
        <div class="flex items-center justify-between">
            <h3 class="text-base font-semibold text-secondary-900">{{ $title }}</h3>
            <div class="flex items-center gap-2">
                {{-- Search, filter, action buttons --}}
            </div>
        </div>
    </div>
    <div class="overflow-x-auto">
        <table class="w-full text-sm">
            <thead>
                <tr class="border-b border-secondary-200">
                    <th class="text-left px-6 py-3 text-xs font-semibold text-secondary-500 uppercase tracking-wider">
                        Kolom
                    </th>
                </tr>
            </thead>
            <tbody class="divide-y divide-secondary-100">
                @forelse($items as $item)
                    <tr class="hover:bg-secondary-50 transition-colors">
                        <td class="px-6 py-4">{{ $item->name }}</td>
                    </tr>
                @empty
                    <tr>
                        <td colspan="100%" class="px-6 py-12 text-center text-secondary-400">
                            <p class="text-sm">Tidak ada data ditemukan.</p>
                        </td>
                    </tr>
                @endforelse
            </tbody>
        </table>
    </div>
    @if($items->hasPages())
        <div class="px-6 py-4 border-t border-secondary-200">
            {{ $items->links() }}
        </div>
    @endif
</div>
```

---

## 5. Vite — Asset Management

### 5.1 — Entry Points

```js
// vite.config.js — entry points standar
laravel({
    input: [
        'resources/css/app.css',
        'resources/js/app.js',
    ],
    refresh: true,
}),
```

**Aturan:**
- Gunakan `@vite()` directive di Blade layout untuk memuat assets.
- **JANGAN** gunakan `<link>` atau `<script>` manual untuk file lokal. Selalu melalui Vite.
- **BOLEH** gunakan `<link>` untuk CDN eksternal (Google Fonts, dsb.).

### 5.2 — Import CSS di JavaScript

```js
// ✅ Benar — CSS di-import melalui JS entry point
// resources/js/app.js
import '../css/app.css';
```

**Aturan:**
- CSS di-import via `app.js`, bukan terpisah di Blade (kecuali untuk Google Fonts).
- **JANGAN** buat file CSS per halaman. Semua styling melalui Tailwind utility classes atau `@layer` di `app.css`.
- **Pengecualian:** CSS vendor pihak ketiga boleh di-import terpisah jika ukurannya besar.

### 5.3 — JavaScript Modular

```js
// ✅ Benar — Modular, fungsionalitas terpisah
// resources/js/app.js
import '../css/app.css';
import './modules/sidebar.js';
import './modules/dropdown.js';
import './modules/modal.js';

// resources/js/modules/sidebar.js
export function initSidebar() {
    const toggle = document.getElementById('sidebar-toggle');
    // ...
}

document.addEventListener('DOMContentLoaded', initSidebar);
```

**Aturan:**
- **JANGAN** tulis semua JavaScript di satu file `app.js` yang besar (>100 baris).
- Pecah fungsionalitas ke `resources/js/modules/`.
- Gunakan ES Module (`import`/`export`).
- **JANGAN** gunakan jQuery. Gunakan vanilla JS atau Alpine.js.

### 5.4 — Alpine.js untuk Interaktivitas

Alpine.js adalah **standar** untuk interaktivitas ringan di Blade.

```blade
{{-- ✅ Benar — Alpine.js untuk dropdown --}}
<div x-data="{ open: false }" class="relative">
    <button @click="open = !open" class="btn-secondary">
        Menu
        <svg :class="{ 'rotate-180': open }" class="w-4 h-4 transition-transform">...</svg>
    </button>

    <div x-show="open"
         x-transition:enter="transition ease-out duration-200"
         x-transition:enter-start="opacity-0 scale-95"
         x-transition:enter-end="opacity-100 scale-100"
         x-transition:leave="transition ease-in duration-150"
         x-transition:leave-start="opacity-100 scale-100"
         x-transition:leave-end="opacity-0 scale-95"
         @click.away="open = false"
         class="absolute right-0 mt-2 w-48 bg-white rounded-lg shadow-lg border border-secondary-200 z-50">
        ...
    </div>
</div>
```

**Aturan:**
- Gunakan Alpine.js untuk: dropdown, modal, toggle, tab, accordion, form interaction.
- **JANGAN** campur Alpine.js dengan jQuery.
- **JANGAN** buat logic bisnis di Alpine.js — hanya untuk UI state.
- Tambahkan `x-transition` pada semua elemen yang ditampilkan/disembunyikan.
- Gunakan `x-cloak` untuk mencegah flash of unstyled content.

---

## 6. Form Handling — Standar UI Form

### 6.1 — Struktur Form yang Benar

```blade
<form method="POST" action="{{ route('users.store') }}" class="space-y-6">
    @csrf

    {{-- Form fields menggunakan component --}}
    <x-form.input label="Nama Lengkap" name="name" required />
    <x-form.input label="Email" name="email" type="email" required />

    {{-- Form actions di bawah --}}
    <div class="flex items-center justify-end gap-3 pt-4 border-t border-secondary-200">
        <a href="{{ route('users.index') }}" class="btn-secondary">Batal</a>
        <button type="submit" class="btn-primary">Simpan</button>
    </div>
</form>
```

**Aturan:**
- **WAJIB** sertakan `@csrf` di setiap form.
- **WAJIB** gunakan `old()` helper untuk mempertahankan input saat validasi gagal.
- **WAJIB** tampilkan error per field menggunakan `@error($name)`.
- Form actions (Simpan, Batal) selalu di bagian bawah form dengan separator `border-t`.
- Button submit selalu di **kanan**. Button cancel di **kiri**.

### 6.2 — Validasi Error Display

```blade
{{-- ✅ Global validation errors (di partials/alert.blade.php) --}}
@if ($errors->any())
    <div class="mb-4 p-4 rounded-lg bg-red-50 border border-red-200 text-red-800 text-sm">
        <p class="font-medium mb-2">Terdapat {{ $errors->count() }} kesalahan:</p>
        <ul class="list-disc list-inside space-y-1">
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

{{-- ✅ Inline field error --}}
@error('email')
    <p class="form-error">{{ $message }}</p>
@enderror
```

---

## 7. Accessibility (a11y) — Standar Minimum

**WAJIB** memenuhi standar aksesibilitas berikut:

| Aturan | Implementasi |
|--------|-------------|
| Label pada setiap input | `<label for="name">` yang terhubung dengan `id` input |
| Alt text pada gambar | `<img alt="Deskripsi gambar">` |
| Role pada alert | `<div role="alert">` untuk flash messages |
| Focus visible | Gunakan `focus:ring-2 focus:ring-primary-500` pada elemen interaktif |
| Keyboard navigable | Semua aksi harus bisa diakses via keyboard (Tab, Enter, Escape) |
| Color contrast | Teks `text-secondary-500` atau lebih gelap di atas `bg-white` |
| Aria labels | `aria-label` pada icon-only buttons |
| Semantic HTML | Gunakan `<nav>`, `<main>`, `<aside>`, `<header>`, `<footer>` |

```blade
{{-- ✅ Benar — Accessible icon button --}}
<button aria-label="Hapus item" class="p-2 text-red-600 hover:bg-red-50 rounded-lg transition-colors">
    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
    </svg>
</button>

{{-- ❌ Salah — Tidak accessible --}}
<button class="p-2 text-red-600">
    <svg>...</svg>
</button>
```

---

## 8. Icons — Standar Penggunaan

**WAJIB** gunakan **Heroicons** (outline style) sebagai standar icon library.

```blade
{{-- ✅ Benar — Heroicons inline SVG --}}
<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
          d="M12 4v16m8-8H4" />
</svg>
```

**Aturan:**
- Gunakan **outline** style untuk konsistensi.
- **JANGAN** campur icon library (Heroicons + Font Awesome + Lucide di satu project).
- Ukuran standar: `w-4 h-4` (inline text), `w-5 h-5` (button/nav), `w-6 h-6` (feature card).
- Warna icon mengikuti warna teks parent: `text-current` atau `currentColor`.

---

## 9. Flash Messages & Feedback — UX Standards

### 9.1 — Flash Session dari Controller

```php
// ✅ Standar flash message
return redirect()->route('users.index')->with('success', 'User berhasil ditambahkan.');
return redirect()->back()->with('error', 'Gagal menyimpan data.')->withInput();
return redirect()->back()->with('warning', 'Data sudah ada sebelumnya.');
```

### 9.2 — Tampilan Alert

**WAJIB** gunakan partial `partials/alert.blade.php` yang sudah disediakan. Jangan buat alert ad-hoc.

| Session Key | Warna | Penggunaan |
|-------------|-------|------------|
| `success` | Green (`bg-green-50`, `text-green-800`) | Operasi berhasil |
| `error` | Red (`bg-red-50`, `text-red-800`) | Operasi gagal |
| `warning` | Yellow (`bg-yellow-50`, `text-yellow-800`) | Peringatan |
| `info` | Blue (`bg-blue-50`, `text-blue-800`) | Informasi umum |

### 9.3 — Empty State

**WAJIB** tampilkan empty state yang informatif saat data kosong.

```blade
{{-- ✅ Benar — Empty state yang informatif --}}
@forelse($users as $user)
    ...
@empty
    <div class="text-center py-12">
        <svg class="w-12 h-12 text-secondary-300 mx-auto mb-4">...</svg>
        <h3 class="text-sm font-medium text-secondary-900">Belum ada data</h3>
        <p class="text-sm text-secondary-500 mt-1">Mulai dengan menambahkan data baru.</p>
        <a href="{{ route('users.create') }}" class="btn-primary btn-sm mt-4">
            Tambah User
        </a>
    </div>
@endforelse
```

---

## 10. Delete Confirmation — Pattern Standar

**WAJIB** tampilkan konfirmasi sebelum menghapus data.

```blade
{{-- ✅ Benar — Form delete dengan konfirmasi JavaScript --}}
<form method="POST" action="{{ route('users.destroy', $user) }}"
      onsubmit="return confirm('Yakin ingin menghapus {{ $user->name }}?')">
    @csrf
    @method('DELETE')
    <button type="submit" class="btn-danger btn-sm">Hapus</button>
</form>

{{-- ✅ Lebih baik — Gunakan Alpine.js modal konfirmasi --}}
<div x-data="{ showDelete: false }">
    <button @click="showDelete = true" class="btn-danger btn-sm">Hapus</button>

    {{-- Modal Konfirmasi --}}
    <div x-show="showDelete" x-cloak
         class="fixed inset-0 z-50 flex items-center justify-center bg-black/50">
        <div class="bg-white rounded-xl shadow-xl p-6 max-w-sm w-full mx-4">
            <h3 class="text-lg font-semibold text-secondary-900">Konfirmasi Hapus</h3>
            <p class="text-sm text-secondary-500 mt-2">Apakah Anda yakin ingin menghapus data ini? Tindakan ini tidak dapat dibatalkan.</p>
            <div class="flex justify-end gap-3 mt-6">
                <button @click="showDelete = false" class="btn-secondary">Batal</button>
                <form method="POST" action="{{ route('users.destroy', $user) }}">
                    @csrf
                    @method('DELETE')
                    <button type="submit" class="btn-danger">Hapus</button>
                </form>
            </div>
        </div>
    </div>
</div>
```

---

## 11. Performance — Frontend Optimization

- **Gunakan `@vite()`** — Vite melakukan tree-shaking, minification, dan code splitting otomatis di production.
- **Lazy load gambar**: `<img loading="lazy" alt="...">`.
- **Gunakan `@once`** untuk script/style yang hanya perlu di-render sekali:

```blade
@once
    @push('scripts')
        <script src="https://cdn.example.com/library.js"></script>
    @endpush
@endonce
```

- **Hindari inline `<style>`** dan `<script>` di Blade. Gunakan `@push('styles')` / `@push('scripts')`.
- **Pagination**: Selalu gunakan pagination (`->paginate()`) untuk data list. **JANGAN** tampilkan semua data sekaligus.
- **SVG inline** untuk icons (bukan font icons) — lebih cepat dan tidak perlu HTTP request tambahan.

---

## 12. General Code Quality Rules — Frontend

- **JANGAN** tulis CSS inline (`style="..."`) di Blade. Gunakan Tailwind utility classes.
- **JANGAN** tulis JavaScript inline (`onclick="..."`) di Blade. Gunakan Alpine.js atau event listener.
- **JANGAN** hardcode URL. Gunakan `route()` atau `url()` helper.
- **JANGAN** hardcode teks label. Pertimbangkan localization jika multi-bahasa (`__('messages.key')`).
- **WAJIB** gunakan `{{ asset('...') }}` untuk static files di `public/`.
- **WAJIB** gunakan `@stack` dan `@push` untuk page-specific styles/scripts, bukan langsung di Blade.
- **Setiap halaman harus punya `<title>` yang unik** — pass via props layout (`title="Dashboard"`).
- **JANGAN** biarkan console.log di production code. Gunakan hanya saat development.

---

## Ringkasan Aturan

| # | Aturan | Prioritas |
|---|--------|-----------|
| 1 | Component-based layout (`<x-layouts.app>`) | 🔴 Wajib |
| 2 | Blade component untuk elemen reusable, dengan `@props` | 🔴 Wajib |
| 3 | `{{ }}` escaped output — JANGAN `{!! !!}` sembarangan | 🔴 Wajib |
| 4 | Design tokens (`primary-*`, `secondary-*`) — JANGAN hardcode warna | 🔴 Wajib |
| 5 | Custom classes di `@layer components` untuk elemen berulang | 🔴 Wajib |
| 6 | Mobile-first responsive design | 🔴 Wajib |
| 7 | `@csrf` di setiap form, `@method` untuk PUT/DELETE | 🔴 Wajib |
| 8 | `old()` helper dan `@error` di setiap form field | 🔴 Wajib |
| 9 | Transition pada semua elemen interaktif | 🟡 Penting |
| 10 | `@forelse` + empty state informatif | 🟡 Penting |
| 11 | Heroicons (outline) sebagai standar icon | 🟡 Penting |
| 12 | Delete confirmation sebelum hapus data | 🟡 Penting |
| 13 | Alpine.js untuk interaktivitas, BUKAN jQuery | 🟡 Penting |
| 14 | Accessibility: label, aria, focus, semantic HTML | 🟡 Penting |
| 15 | `@vite()` untuk assets, JANGAN manual `<link>`/`<script>` | 🟡 Penting |
| 16 | JANGAN inline CSS/JS — gunakan Tailwind + Alpine.js | 🟡 Penting |
