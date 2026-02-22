# 🚀 VibeCoding Laravel Starter

**VibeCoding Laravel Starter** adalah *boilerplate* dan *starter kit* yang dirancang khusus untuk pengembangan aplikasi Laravel yang ditenagai oleh AI Agent (seperti Antigravity, Cursor, atau Cline). 

Repository ini tidak hanya berisi struktur dasar Laravel, tetapi juga dilengkapi dengan direktori `.agent/` yang berisi instruksi kerja (*workflows*), aturan (*rules*), dan *Standard Operating Procedures* (SOP) berformat Markdown. Ini memastikan AI dapat mengeksekusi tugas secara bertahap, konsisten, dan sesuai dengan *best practices* pengembangan Laravel.

Sangat cocok untuk mempercepat pengerjaan *project* berskala menengah hingga besar, seperti Sistem Informasi Manajemen Aset, aplikasi *dashboard* admin yang kompleks, maupun *RESTful API*.

---

## 📂 Struktur Direktori Agentic

Keunggulan utama dari *starter kit* ini ada pada folder `.agent/` yang menjadi "otak" panduan bagi AI:

```text
.agent/
  ├── rules/         # Aturan penulisan kode (PHP styling, penamaan variabel, dll)
  ├── skills/        # Kumpulan instruksi spesifik untuk fitur tertentu
  └── workflows/     # SOP langkah-demi-langkah untuk AI
      ├── 01_laravel-backend-setup.md
      ├── 02_frontend-blade-vite.md
      ├── 03_admin-dashboard-auth.md
      ├── 04_testing-and-debugging.md
      ├── 05_rbac-user-management.md
      ├── 06_deployment-optimization.md
      ├── 07_documentation-maintenance.md
      └── 08_artisan-playground.md
