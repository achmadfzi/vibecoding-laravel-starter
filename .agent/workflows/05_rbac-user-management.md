---
description: SOP untuk membuat fitur RBAC & manajemen pengguna menggunakan spatie/laravel-permission — termasuk seeder role, CRUD user, middleware, dan Blade directives.
---

# 05 — RBAC & User Management

> **Tujuan**: Mengimplementasikan Role-Based Access Control (RBAC) dan fitur manajemen pengguna menggunakan package `spatie/laravel-permission`. Termasuk konfigurasi package, seeder role/permission, CRUD user dengan assignment role, middleware proteksi route, dan Blade directives untuk kontrol tampilan UI.

---

## Prasyarat

- Laravel sudah terinstall dengan autentikasi Breeze (lihat `03_admin-dashboard-auth.md`)
- Database MySQL terhubung dan migration sudah jalan
- Layout admin sudah tersedia (`layouts/admin.blade.php`)
- Minimal sudah ada 1 user di database (dari seeder)

---

## Step 1 — Install spatie/laravel-permission

### 1.1 — Install package via Composer

// turbo
```bash
composer require spatie/laravel-permission
```

### 1.2 — Publish migration dan config

// turbo
```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

Ini akan membuat:
- `database/migrations/xxxx_create_permission_tables.php`
- `config/permission.php`

### 1.3 — Jalankan migration

// turbo
```bash
php artisan migrate
```

### 1.4 — Verifikasi tabel terbuat

// turbo
```bash
php artisan migrate:status
```

Pastikan tabel berikut sudah ada:
- `roles`
- `permissions`
- `model_has_roles`
- `model_has_permissions`
- `role_has_permissions`

---

## Step 2 — Konfigurasi Model User

### 2.1 — Tambahkan trait HasRoles ke User model

Buka `app/Models/User.php` dan tambahkan trait:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Spatie\Permission\Traits\HasRoles; // ← Tambahkan import

class User extends Authenticatable
{
    use HasFactory, Notifiable, HasRoles; // ← Tambahkan HasRoles

    protected $fillable = [
        'name',
        'email',
        'password',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected function casts(): array
    {
        return [
            'email_verified_at' => 'datetime',
            'password' => 'hashed',
        ];
    }
}
```

### 2.2 — Konfigurasi `config/permission.php` (opsional)

Buka `config/permission.php` dan pastikan:

```php
'models' => [
    'permission' => Spatie\Permission\Models\Permission::class,
    'role' => Spatie\Permission\Models\Role::class,
],

// Cache — biarkan default, clear saat role/permission berubah
'cache' => [
    'expiration_time' => \DateInterval::createFromDateString('24 hours'),
    'key' => 'spatie.permission.cache',
    'store' => 'default',
],
```

---

## Step 3 — Buat Seeder untuk Role dan Permission

### 3.1 — Generate seeder

// turbo
```bash
php artisan make:seeder RolePermissionSeeder
```

### 3.2 — Edit RolePermissionSeeder

Buka `database/seeders/RolePermissionSeeder.php`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;
use Spatie\Permission\PermissionRegistrar;

class RolePermissionSeeder extends Seeder
{
    public function run(): void
    {
        // Reset cached roles and permissions
        app()[PermissionRegistrar::class]->forgetCachedPermissions();

        /*
        |----------------------------------------------------------------------
        | Define Permissions
        |----------------------------------------------------------------------
        | Gunakan format: "resource-action"
        | Sesuaikan dengan fitur/modul aplikasi
        */
        $permissions = [
            // User Management
            'user-list',
            'user-create',
            'user-edit',
            'user-delete',

            // Role Management
            'role-list',
            'role-create',
            'role-edit',
            'role-delete',

            // Dashboard
            'dashboard-view',

            // Tambahkan permission fitur lain di sini:
            // 'product-list',
            // 'product-create',
            // 'product-edit',
            // 'product-delete',
            // 'report-view',
            // 'report-export',
            // 'setting-manage',
        ];

        foreach ($permissions as $permission) {
            Permission::create(['name' => $permission]);
        }

        /*
        |----------------------------------------------------------------------
        | Define Roles & Assign Permissions
        |----------------------------------------------------------------------
        */

        // Super Admin — punya semua permission
        $superAdmin = Role::create(['name' => 'super-admin']);
        $superAdmin->givePermissionTo(Permission::all());

        // Admin — punya sebagian besar permission
        $admin = Role::create(['name' => 'admin']);
        $admin->givePermissionTo([
            'user-list',
            'user-create',
            'user-edit',
            'dashboard-view',
            // Tambahkan permission lain sesuai kebutuhan
        ]);

        // Operator — punya permission terbatas
        $operator = Role::create(['name' => 'operator']);
        $operator->givePermissionTo([
            'dashboard-view',
            // Tambahkan permission operasional di sini
        ]);

        // User biasa (opsional)
        $user = Role::create(['name' => 'user']);
        $user->givePermissionTo([
            'dashboard-view',
        ]);
    }
}
```

> [!IMPORTANT]
> **Tanyakan ke user** role dan permission apa saja yang dibutuhkan untuk aplikasi mereka. Daftar di atas adalah template default. Sesuaikan sebelum menjalankan seeder.

### 3.3 — Update DatabaseSeeder

Buka `database/seeders/DatabaseSeeder.php` dan tambahkan:

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
        // Jalankan RolePermissionSeeder terlebih dahulu
        $this->call(RolePermissionSeeder::class);

        // Buat Super Admin user
        $superAdmin = User::factory()->create([
            'name' => 'Super Admin',
            'email' => 'superadmin@admin.com',
            'password' => Hash::make('password'),
        ]);
        $superAdmin->assignRole('super-admin');

        // Buat Admin user
        $admin = User::factory()->create([
            'name' => 'Admin',
            'email' => 'admin@admin.com',
            'password' => Hash::make('password'),
        ]);
        $admin->assignRole('admin');

        // Buat Operator user
        $operator = User::factory()->create([
            'name' => 'Operator',
            'email' => 'operator@admin.com',
            'password' => Hash::make('password'),
        ]);
        $operator->assignRole('operator');
    }
}
```

### 3.4 — Jalankan seeder

// turbo
```bash
php artisan db:seed --class=RolePermissionSeeder
```

Atau untuk fresh seeding (reset database + seed):

```bash
php artisan migrate:fresh --seed
```

> [!CAUTION]
> `migrate:fresh --seed` akan **menghapus semua data** di database dan membuat ulang tabel. Gunakan hanya di environment development.

### 3.5 — Verifikasi seeder

// turbo
```bash
php artisan tinker --execute="echo 'Roles: ' . Spatie\Permission\Models\Role::pluck('name')->implode(', ') . PHP_EOL . 'Permissions: ' . Spatie\Permission\Models\Permission::count() . ' total';"
```

---

## Step 4 — Buat User Management (CRUD)

### 4.1 — Generate Controller

// turbo
```bash
php artisan make:controller Admin/UserController --resource
```

### 4.2 — Generate Form Request untuk validasi

// turbo
```bash
php artisan make:request Admin/StoreUserRequest
php artisan make:request Admin/UpdateUserRequest
```

### 4.3 — Edit StoreUserRequest

Buka `app/Http/Requests/Admin/StoreUserRequest.php`:

```php
<?php

namespace App\Http\Requests\Admin;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

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
            'email'    => ['required', 'string', 'email', 'max:255', 'unique:users,email'],
            'password' => ['required', 'confirmed', Password::defaults()],
            'roles'    => ['required', 'array', 'min:1'],
            'roles.*'  => ['string', 'exists:roles,name'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required'     => 'Nama wajib diisi.',
            'email.required'    => 'Email wajib diisi.',
            'email.unique'      => 'Email sudah digunakan.',
            'password.required' => 'Password wajib diisi.',
            'password.confirmed'=> 'Konfirmasi password tidak cocok.',
            'roles.required'    => 'Minimal pilih satu role.',
        ];
    }
}
```

### 4.4 — Edit UpdateUserRequest

Buka `app/Http/Requests/Admin/UpdateUserRequest.php`:

```php
<?php

namespace App\Http\Requests\Admin;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\Password;

class UpdateUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        return $this->user()->can('user-edit');
    }

    public function rules(): array
    {
        return [
            'name'     => ['required', 'string', 'max:255'],
            'email'    => [
                'required', 'string', 'email', 'max:255',
                Rule::unique('users', 'email')->ignore($this->route('user')),
            ],
            'password' => ['nullable', 'confirmed', Password::defaults()],
            'roles'    => ['required', 'array', 'min:1'],
            'roles.*'  => ['string', 'exists:roles,name'],
        ];
    }

    public function messages(): array
    {
        return [
            'name.required'  => 'Nama wajib diisi.',
            'email.required' => 'Email wajib diisi.',
            'email.unique'   => 'Email sudah digunakan.',
            'roles.required' => 'Minimal pilih satu role.',
        ];
    }
}
```

### 4.5 — Edit UserController

Buka `app/Http/Controllers/Admin/UserController.php`:

```php
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Http\Requests\Admin\StoreUserRequest;
use App\Http\Requests\Admin\UpdateUserRequest;
use App\Models\User;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Hash;
use Illuminate\View\View;
use Spatie\Permission\Models\Role;

class UserController extends Controller
{
    /**
     * Daftar semua user.
     */
    public function index(): View
    {
        $users = User::with('roles')
            ->latest()
            ->paginate(15);

        return view('admin.users.index', compact('users'));
    }

    /**
     * Form tambah user baru.
     */
    public function create(): View
    {
        $roles = Role::orderBy('name')->pluck('name', 'name');

        return view('admin.users.create', compact('roles'));
    }

    /**
     * Simpan user baru.
     */
    public function store(StoreUserRequest $request): RedirectResponse
    {
        $user = User::create([
            'name'     => $request->name,
            'email'    => $request->email,
            'password' => Hash::make($request->password),
        ]);

        $user->assignRole($request->roles);

        return redirect()
            ->route('admin.users.index')
            ->with('success', 'User berhasil ditambahkan.');
    }

    /**
     * Detail user.
     */
    public function show(User $user): View
    {
        $user->load('roles', 'permissions');

        return view('admin.users.show', compact('user'));
    }

    /**
     * Form edit user.
     */
    public function edit(User $user): View
    {
        $roles = Role::orderBy('name')->pluck('name', 'name');
        $userRoles = $user->roles->pluck('name')->toArray();

        return view('admin.users.edit', compact('user', 'roles', 'userRoles'));
    }

    /**
     * Update user.
     */
    public function update(UpdateUserRequest $request, User $user): RedirectResponse
    {
        $data = [
            'name'  => $request->name,
            'email' => $request->email,
        ];

        // Update password hanya jika diisi
        if ($request->filled('password')) {
            $data['password'] = Hash::make($request->password);
        }

        $user->update($data);

        // Sync roles (hapus lama, assign baru)
        $user->syncRoles($request->roles);

        return redirect()
            ->route('admin.users.index')
            ->with('success', 'User berhasil diperbarui.');
    }

    /**
     * Hapus user.
     */
    public function destroy(User $user): RedirectResponse
    {
        // Jangan hapus diri sendiri
        if ($user->id === auth()->id()) {
            return redirect()
                ->route('admin.users.index')
                ->with('error', 'Anda tidak dapat menghapus akun sendiri.');
        }

        $user->delete();

        return redirect()
            ->route('admin.users.index')
            ->with('success', 'User berhasil dihapus.');
    }
}
```

---

## Step 5 — Setup Route dengan Middleware Permission

### 5.1 — Daftarkan middleware Spatie

**Laravel 11+** — Buka `bootstrap/app.php` dan daftarkan alias middleware:

```php
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__ . '/../routes/web.php',
        commands: __DIR__ . '/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->alias([
            'role'       => \Spatie\Permission\Middleware\RoleMiddleware::class,
            'permission' => \Spatie\Permission\Middleware\PermissionMiddleware::class,
            'role_or_permission' => \Spatie\Permission\Middleware\RoleOrPermissionMiddleware::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })
    ->create();
```

> [!NOTE]
> Pada **Laravel 10** atau sebelumnya, daftarkan di `$middlewareAliases` dalam `app/Http/Kernel.php`.

### 5.2 — Tambahkan route CRUD User di `routes/web.php`

Tambahkan route user management di dalam route group `admin`:

```php
use App\Http\Controllers\Admin\UserController;

Route::middleware(['auth', 'verified'])->group(function () {

    Route::prefix('admin')->name('admin.')->group(function () {

        // Dashboard (semua role bisa akses)
        Route::get('/dashboard', [DashboardController::class, 'index'])
            ->name('dashboard');

        // User Management — hanya yang punya permission
        Route::resource('users', UserController::class)
            ->middleware('permission:user-list|user-create|user-edit|user-delete');

        // Atau lebih granular per action:
        // Route::middleware('permission:user-list')->get('/users', [UserController::class, 'index'])->name('users.index');
        // Route::middleware('permission:user-create')->get('/users/create', [UserController::class, 'create'])->name('users.create');
        // Route::middleware('permission:user-create')->post('/users', [UserController::class, 'store'])->name('users.store');
        // Route::middleware('permission:user-edit')->get('/users/{user}/edit', [UserController::class, 'edit'])->name('users.edit');
        // Route::middleware('permission:user-edit')->put('/users/{user}', [UserController::class, 'update'])->name('users.update');
        // Route::middleware('permission:user-delete')->delete('/users/{user}', [UserController::class, 'destroy'])->name('users.destroy');
    });

    // Profile routes
    Route::get('/profile', [ProfileController::class, 'edit'])->name('profile.edit');
    Route::patch('/profile', [ProfileController::class, 'update'])->name('profile.update');
    Route::delete('/profile', [ProfileController::class, 'destroy'])->name('profile.destroy');
});
```

> [!TIP]
> **Strategi middleware:**
> - Gunakan `permission:` untuk kontrol granular (recommended).
> - Gunakan `role:` untuk kontrol berbasis role keseluruhan.
> - Gunakan `role_or_permission:` untuk kombinasi.
>
> **Contoh:**
> ```php
> ->middleware('role:super-admin')           // Hanya super-admin
> ->middleware('role:admin|super-admin')     // Admin atau super-admin
> ->middleware('permission:user-list')       // Siapapun yang punya permission user-list
> ```

### 5.3 — Verifikasi route

// turbo
```bash
php artisan route:list --name=admin.users
```

---

## Step 6 — Buat View Blade untuk CRUD User

### 6.1 — Buat folder views

// turbo
```powershell
New-Item -ItemType Directory -Force -Path "resources/views/admin/users"
```

### 6.2 — Index View (Daftar User)

Buat file `resources/views/admin/users/index.blade.php`:

```blade
<x-admin-layout title="Manajemen User">
    <x-slot name="header">
        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4">
            <div>
                <h2 class="text-2xl font-bold text-gray-900">Manajemen User</h2>
                <p class="text-sm text-gray-500 mt-1">Kelola semua pengguna aplikasi</p>
            </div>
            @can('user-create')
                <a href="{{ route('admin.users.create') }}" class="btn-primary btn-sm">
                    <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                    </svg>
                    Tambah User
                </a>
            @endcan
        </div>
    </x-slot>

    <div class="card">
        <div class="overflow-x-auto">
            <table class="w-full text-sm">
                <thead>
                    <tr class="border-b border-gray-200 bg-gray-50/50">
                        <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider">#</th>
                        <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider">Nama</th>
                        <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider hidden sm:table-cell">Email</th>
                        <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider hidden md:table-cell">Role</th>
                        <th class="text-left px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider hidden lg:table-cell">Dibuat</th>
                        <th class="text-right px-6 py-3 text-xs font-semibold text-gray-500 uppercase tracking-wider">Aksi</th>
                    </tr>
                </thead>
                <tbody class="divide-y divide-gray-100">
                    @forelse ($users as $index => $user)
                        <tr class="hover:bg-gray-50 transition-colors">
                            <td class="px-6 py-4 text-gray-500">
                                {{ $users->firstItem() + $index }}
                            </td>
                            <td class="px-6 py-4">
                                <div class="flex items-center gap-3">
                                    <div class="w-8 h-8 rounded-full bg-primary-100 flex items-center justify-center shrink-0">
                                        <span class="text-xs font-semibold text-primary-700">
                                            {{ strtoupper(substr($user->name, 0, 2)) }}
                                        </span>
                                    </div>
                                    <span class="font-medium text-gray-900">{{ $user->name }}</span>
                                </div>
                            </td>
                            <td class="px-6 py-4 text-gray-600 hidden sm:table-cell">{{ $user->email }}</td>
                            <td class="px-6 py-4 hidden md:table-cell">
                                @foreach ($user->roles as $role)
                                    @php
                                        $badgeClass = match($role->name) {
                                            'super-admin' => 'badge-danger',
                                            'admin'       => 'badge-info',
                                            'operator'    => 'badge-warning',
                                            default       => 'badge-success',
                                        };
                                    @endphp
                                    <span class="{{ $badgeClass }}">{{ ucfirst($role->name) }}</span>
                                @endforeach
                            </td>
                            <td class="px-6 py-4 text-gray-500 hidden lg:table-cell">
                                {{ $user->created_at->format('d M Y') }}
                            </td>
                            <td class="px-6 py-4">
                                <div class="flex items-center justify-end gap-2">
                                    @can('user-edit')
                                        <a href="{{ route('admin.users.edit', $user) }}"
                                           class="p-1.5 rounded-lg text-gray-500 hover:bg-blue-50 hover:text-blue-600 transition-colors"
                                           title="Edit">
                                            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                      d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                                            </svg>
                                        </a>
                                    @endcan

                                    @can('user-delete')
                                        @if ($user->id !== auth()->id())
                                            <form method="POST" action="{{ route('admin.users.destroy', $user) }}"
                                                  onsubmit="return confirm('Yakin ingin menghapus user {{ $user->name }}?')">
                                                @csrf
                                                @method('DELETE')
                                                <button type="submit"
                                                        class="p-1.5 rounded-lg text-gray-500 hover:bg-red-50 hover:text-red-600 transition-colors"
                                                        title="Hapus">
                                                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                                              d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                                                    </svg>
                                                </button>
                                            </form>
                                        @endif
                                    @endcan
                                </div>
                            </td>
                        </tr>
                    @empty
                        <tr>
                            <td colspan="6" class="px-6 py-12 text-center text-gray-400">
                                <svg class="w-12 h-12 mx-auto mb-3 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                                          d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197" />
                                </svg>
                                Belum ada user.
                            </td>
                        </tr>
                    @endforelse
                </tbody>
            </table>
        </div>

        {{-- Pagination --}}
        @if ($users->hasPages())
            <div class="px-6 py-4 border-t border-gray-200">
                {{ $users->links() }}
            </div>
        @endif
    </div>
</x-admin-layout>
```

### 6.3 — Create View (Form Tambah User)

Buat file `resources/views/admin/users/create.blade.php`:

```blade
<x-admin-layout title="Tambah User">
    <x-slot name="header">
        <div class="flex items-center gap-4">
            <a href="{{ route('admin.users.index') }}"
               class="p-2 rounded-lg text-gray-500 hover:bg-gray-100 transition-colors">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
                </svg>
            </a>
            <div>
                <h2 class="text-2xl font-bold text-gray-900">Tambah User Baru</h2>
                <p class="text-sm text-gray-500 mt-1">Buat akun pengguna baru</p>
            </div>
        </div>
    </x-slot>

    <div class="max-w-2xl">
        <div class="card">
            <div class="card-body">
                <form method="POST" action="{{ route('admin.users.store') }}" class="space-y-6">
                    @csrf

                    {{-- Nama --}}
                    <div>
                        <label for="name" class="form-label">Nama Lengkap <span class="text-red-500">*</span></label>
                        <input type="text" id="name" name="name" value="{{ old('name') }}"
                               class="form-input @error('name') border-red-500 @enderror"
                               placeholder="Masukkan nama lengkap"
                               required autofocus>
                        @error('name') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Email --}}
                    <div>
                        <label for="email" class="form-label">Email <span class="text-red-500">*</span></label>
                        <input type="email" id="email" name="email" value="{{ old('email') }}"
                               class="form-input @error('email') border-red-500 @enderror"
                               placeholder="email@example.com"
                               required>
                        @error('email') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Password --}}
                    <div>
                        <label for="password" class="form-label">Password <span class="text-red-500">*</span></label>
                        <input type="password" id="password" name="password"
                               class="form-input @error('password') border-red-500 @enderror"
                               placeholder="Minimal 8 karakter"
                               required>
                        @error('password') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Confirm Password --}}
                    <div>
                        <label for="password_confirmation" class="form-label">Konfirmasi Password <span class="text-red-500">*</span></label>
                        <input type="password" id="password_confirmation" name="password_confirmation"
                               class="form-input"
                               placeholder="Ulangi password"
                               required>
                    </div>

                    {{-- Roles --}}
                    <div>
                        <label class="form-label">Role <span class="text-red-500">*</span></label>
                        <div class="mt-2 space-y-2">
                            @foreach ($roles as $role)
                                <label class="flex items-center gap-3 p-3 rounded-lg border border-gray-200 hover:bg-gray-50 cursor-pointer transition-colors">
                                    <input type="checkbox" name="roles[]" value="{{ $role }}"
                                           class="rounded border-gray-300 text-primary-600 focus:ring-primary-500"
                                           {{ in_array($role, old('roles', [])) ? 'checked' : '' }}>
                                    <div>
                                        <span class="text-sm font-medium text-gray-900">{{ ucfirst($role) }}</span>
                                    </div>
                                </label>
                            @endforeach
                        </div>
                        @error('roles') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Actions --}}
                    <div class="flex items-center gap-3 pt-4 border-t border-gray-200">
                        <button type="submit" class="btn-primary">
                            <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                            </svg>
                            Simpan User
                        </button>
                        <a href="{{ route('admin.users.index') }}" class="btn-secondary">Batal</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</x-admin-layout>
```

### 6.4 — Edit View (Form Edit User)

Buat file `resources/views/admin/users/edit.blade.php`:

```blade
<x-admin-layout title="Edit User">
    <x-slot name="header">
        <div class="flex items-center gap-4">
            <a href="{{ route('admin.users.index') }}"
               class="p-2 rounded-lg text-gray-500 hover:bg-gray-100 transition-colors">
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
                </svg>
            </a>
            <div>
                <h2 class="text-2xl font-bold text-gray-900">Edit User</h2>
                <p class="text-sm text-gray-500 mt-1">Edit data pengguna: {{ $user->name }}</p>
            </div>
        </div>
    </x-slot>

    <div class="max-w-2xl">
        <div class="card">
            <div class="card-body">
                <form method="POST" action="{{ route('admin.users.update', $user) }}" class="space-y-6">
                    @csrf
                    @method('PUT')

                    {{-- Nama --}}
                    <div>
                        <label for="name" class="form-label">Nama Lengkap <span class="text-red-500">*</span></label>
                        <input type="text" id="name" name="name" value="{{ old('name', $user->name) }}"
                               class="form-input @error('name') border-red-500 @enderror"
                               required>
                        @error('name') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Email --}}
                    <div>
                        <label for="email" class="form-label">Email <span class="text-red-500">*</span></label>
                        <input type="email" id="email" name="email" value="{{ old('email', $user->email) }}"
                               class="form-input @error('email') border-red-500 @enderror"
                               required>
                        @error('email') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Password (opsional) --}}
                    <div>
                        <label for="password" class="form-label">Password Baru</label>
                        <input type="password" id="password" name="password"
                               class="form-input @error('password') border-red-500 @enderror"
                               placeholder="Kosongkan jika tidak ingin mengubah">
                        @error('password') <p class="form-error">{{ $message }}</p> @enderror
                        <p class="text-xs text-gray-400 mt-1">Biarkan kosong jika tidak ingin mengubah password.</p>
                    </div>

                    {{-- Confirm Password --}}
                    <div>
                        <label for="password_confirmation" class="form-label">Konfirmasi Password Baru</label>
                        <input type="password" id="password_confirmation" name="password_confirmation"
                               class="form-input"
                               placeholder="Ulangi password baru">
                    </div>

                    {{-- Roles --}}
                    <div>
                        <label class="form-label">Role <span class="text-red-500">*</span></label>
                        <div class="mt-2 space-y-2">
                            @foreach ($roles as $role)
                                <label class="flex items-center gap-3 p-3 rounded-lg border border-gray-200 hover:bg-gray-50 cursor-pointer transition-colors">
                                    <input type="checkbox" name="roles[]" value="{{ $role }}"
                                           class="rounded border-gray-300 text-primary-600 focus:ring-primary-500"
                                           {{ in_array($role, old('roles', $userRoles)) ? 'checked' : '' }}>
                                    <div>
                                        <span class="text-sm font-medium text-gray-900">{{ ucfirst($role) }}</span>
                                    </div>
                                </label>
                            @endforeach
                        </div>
                        @error('roles') <p class="form-error">{{ $message }}</p> @enderror
                    </div>

                    {{-- Actions --}}
                    <div class="flex items-center gap-3 pt-4 border-t border-gray-200">
                        <button type="submit" class="btn-primary">
                            <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                            </svg>
                            Update User
                        </button>
                        <a href="{{ route('admin.users.index') }}" class="btn-secondary">Batal</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</x-admin-layout>
```

---

## Step 7 — Update Sidebar dengan Menu User Management

Buka `resources/views/layouts/admin-sidebar.blade.php` dan tambahkan menu item di section "Menu Utama":

```blade
{{-- User Management — hanya tampil jika punya permission --}}
@can('user-list')
    <a href="{{ route('admin.users.index') }}"
       class="{{ request()->routeIs('admin.users.*') ? 'sidebar-link-active' : 'sidebar-link-inactive' }}">
        <svg class="w-5 h-5 shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.25 2.25 0 11-4.5 0 2.25 2.25 0 014.5 0z" />
        </svg>
        <span>Manajemen User</span>
    </a>
@endcan
```

---

## Step 8 — Blade Directives Reference

### 8.1 — Directive yang tersedia dari Spatie

| Directive | Fungsi | Contoh |
|-----------|--------|--------|
| `@role('name')` | Cek apakah user memiliki role tertentu | `@role('admin') ... @endrole` |
| `@hasrole('name')` | Sama dengan `@role` | `@hasrole('super-admin') ... @endhasrole` |
| `@hasanyrole('a\|b')` | Cek apakah user punya salah satu role | `@hasanyrole('admin\|super-admin') ... @endhasanyrole` |
| `@hasallroles('a\|b')` | Cek apakah user punya semua role | `@hasallroles('admin\|editor') ... @endhasallroles` |
| `@unlessrole('name')` | Kebalikan dari `@role` | `@unlessrole('admin') ... @endunlessrole` |
| `@can('permission')` | Cek permission (bawaan Laravel) | `@can('user-edit') ... @endcan` |
| `@cannot('permission')` | Kebalikan dari `@can` | `@cannot('user-delete') ... @endcannot` |
| `@canany(['a','b'])` | Cek salah satu permission | `@canany(['user-edit','user-delete']) ... @endcanany` |

### 8.2 — Contoh penggunaan di Blade

```blade
{{-- Tampilkan tombol hanya untuk role tertentu --}}
@role('super-admin')
    <a href="/admin/settings" class="btn-danger">System Settings</a>
@endrole

{{-- Tampilkan menu hanya jika punya permission --}}
@can('user-create')
    <a href="{{ route('admin.users.create') }}" class="btn-primary">
        Tambah User
    </a>
@endcan

{{-- Tampilkan konten untuk admin ATAU super-admin --}}
@hasanyrole('admin|super-admin')
    <div class="card">
        <div class="card-body">
            <p>Panel khusus Admin</p>
        </div>
    </div>
@endhasanyrole

{{-- Sembunyikan tombol delete dari operator --}}
@can('user-delete')
    <form method="POST" action="{{ route('admin.users.destroy', $user) }}">
        @csrf
        @method('DELETE')
        <button class="btn-danger btn-sm">Hapus</button>
    </form>
@endcan

{{-- Pesan jika tidak punya akses --}}
@cannot('report-view')
    <div class="p-4 bg-yellow-50 rounded-lg text-yellow-800 text-sm">
        Anda tidak memiliki akses untuk melihat laporan.
    </div>
@endcannot
```

### 8.3 — Aturan wajib penggunaan directive

> [!WARNING]
> **AI agent WAJIB mengikuti aturan ini saat membuat view:**
> 1. **Tombol aksi** (create, edit, delete) harus dibungkus `@can('permission-name')`.
> 2. **Menu sidebar** harus dibungkus `@can()` atau `@role()` sesuai permission.
> 3. **Formulir berbahaya** (delete, change role) harus dibungkus `@can()` DAN ada konfirmasi JavaScript.
> 4. **Jangan hanya mengandalkan Blade directives** untuk keamanan — selalu validasi juga di **Controller/FormRequest** dan **Middleware**.
> 5. Blade directives = **UI layer protection** (menyembunyikan tombol). Middleware + FormRequest = **server-side protection** (mencegah aksi).

---

## Step 9 — Handle Unauthorized Access (403)

### 9.1 — Buat halaman 403 kustom

Buat file `resources/views/errors/403.blade.php`:

```blade
<x-admin-layout title="Akses Ditolak">
    <div class="flex flex-col items-center justify-center py-16 sm:py-24">
        <div class="w-20 h-20 bg-red-100 rounded-full flex items-center justify-center mb-6">
            <svg class="w-10 h-10 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
            </svg>
        </div>
        <h2 class="text-2xl font-bold text-gray-900 mb-2">Akses Ditolak</h2>
        <p class="text-gray-500 mb-6 text-center max-w-md">
            {{ $exception->getMessage() ?: 'Anda tidak memiliki izin untuk mengakses halaman ini.' }}
        </p>
        <a href="{{ route('admin.dashboard') }}" class="btn-primary">
            <svg class="w-4 h-4 mr-1.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
            </svg>
            Kembali ke Dashboard
        </a>
    </div>
</x-admin-layout>
```

---

## Step 10 — Verifikasi End-to-End

### 10.1 — Test case matrix

| # | Test Case | Login Sebagai | Expected |
|---|-----------|--------------|----------|
| 1 | Akses `/admin/users` | `superadmin@admin.com` | ✅ Tampil daftar user |
| 2 | Akses `/admin/users` | `admin@admin.com` | ✅ Tampil daftar user |
| 3 | Akses `/admin/users` | `operator@admin.com` | ❌ 403 Forbidden |
| 4 | Klik "Tambah User" | super-admin | ✅ Form create tampil |
| 5 | Submit form create | super-admin | ✅ User tersimpan + role assigned |
| 6 | Submit form dengan email duplikat | super-admin | ❌ Validation error |
| 7 | Edit user + ubah role | super-admin | ✅ Data ter-update + role sync |
| 8 | Hapus user lain | super-admin | ✅ User terhapus |
| 9 | Hapus diri sendiri | super-admin | ❌ Error "tidak dapat menghapus akun sendiri" |
| 10 | Tombol edit/delete visible | admin (punya permission) | ✅ Tampil |
| 11 | Tombol edit/delete hidden | operator (tanpa permission) | ✅ Tidak tampil |

### 10.2 — Jalankan test

// turbo
```bash
php artisan test --filter=UserCrudTest
```

// turbo
```bash
php artisan test
```

---

## Checklist Verifikasi Akhir

- [ ] `spatie/laravel-permission` terinstall
- [ ] Migration permission tables sudah jalan (5 tabel)
- [ ] Trait `HasRoles` ditambahkan di Model User
- [ ] Middleware alias (`role`, `permission`, `role_or_permission`) terdaftar
- [ ] `RolePermissionSeeder` dibuat dengan role & permission sesuai kebutuhan
- [ ] `DatabaseSeeder` membuat user testing per role
- [ ] `UserController` CRUD lengkap (index, create, store, edit, update, destroy)
- [ ] `StoreUserRequest` & `UpdateUserRequest` — validasi + authorization
- [ ] Route `admin.users.*` menggunakan middleware permission
- [ ] View index — tabel user responsif + pagination
- [ ] View create — form + role checkbox
- [ ] View edit — form pre-filled + role checkbox
- [ ] Sidebar menampilkan menu "Manajemen User" hanya jika `@can('user-list')`
- [ ] Tombol aksi (create/edit/delete) dibungkus `@can`
- [ ] Halaman 403 kustom tersedia
- [ ] Redirect setelah login/register sudah ke `/admin/dashboard`
- [ ] Self-delete protection (tidak bisa hapus akun sendiri)
- [ ] Test case #1–#11 passing

---

## Catatan Penting untuk AI Agent

> [!WARNING]
> 1. **Tanyakan ke user** role dan permission apa yang dibutuhkan sebelum menjalankan seeder. Jangan gunakan default tanpa konfirmasi.
> 2. **Proteksi 3 lapis** wajib diterapkan: Middleware (route) + FormRequest (authorize) + Blade directive (UI).
> 3. **Jangan pernah hanya mengandalkan `@can` di Blade** — itu hanya menyembunyikan UI, user masih bisa hit endpoint langsung.
> 4. **Clear permission cache** setelah seeding atau mengubah role/permission:
>    ```bash
>    php artisan permission:cache-reset
>    ```
> 5. **Super-admin harus bisa mengakses semua** — gunakan `Gate::before()` di `AuthServiceProvider` jika diperlukan:
>    ```php
>    Gate::before(function ($user, $ability) {
>        return $user->hasRole('super-admin') ? true : null;
>    });
>    ```
> 6. **Self-delete protection** sudah ada di controller. Pastikan juga di UI, tombol hapus tidak muncul untuk user yang sedang login.
