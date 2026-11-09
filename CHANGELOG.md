# Catatan Perubahan (Changelog) - HousePrice

Seluruh pembaruan penting pada proyek HousePrice dicatat dalam berkas ini. Format pencatatan mengacu pada prinsip [Keep a Changelog](https://keepachangelog.com/id/1.0.0/) dan menganut penomoran versi [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

### Ditambahkan (Added - Backend Flask Sprint 1 / Milestone 1 Foundation)
- **Inisialisasi Arsitektur Flask Application Factory:**
  - Setup modularitas backend berbasis `create_app()` pada `app/__init__.py`.
  - Pemisahan inisialisasi ekstensi pada `app/extensions.py` (`SQLAlchemy`, `Bcrypt`, `Migrate`, `CORS`).
  - Registrasi Blueprints: `/api/v1/auth`, `/api/v1/predict`, `/api/v1/notifications`, `/api/v1` (market & locations), dan `/admin`.
- **Implementasi Skema 9 Tabel Basis Data PostgreSQL & Migrasi Alembic:**
  - Pembuatan model data relasional lengkap pada `app/models/` (`admins`, `users`, `notifications`, `master_locations`, `dataset_metadata`, `ml_models`, `market_statistics`, `prediction_logs`, `system_audit_logs`) sesuai `docs/docs/data-model.md`.
  - Pembentukan berkas migrasi awal Alembic (`migrations/versions/e70def023971_create_initial_9_tables.py`) lengkap dengan tipe UUID native, *foreign keys*, *composite indexes*, *check constraints*, dan *partial index* `idx_ml_models_active`.
- **Eksekusi Database Seeder Mandiri (`seed.py`):**
  - Pengisian otomatis seluruh 9 tabel PostgreSQL: 1 akun super admin (`admin_hanif` / `Admin123!`), 3 pengguna mobile sampel, 46 master lokasi wilayah DKI Jakarta, 9 data agregasi statistik pasar, 1 metadata dataset primer (10.000 baris), 2 model ML (Random Forest v1.0.0 aktif $R^2=82.72\%$ & Linear v0.9.0 standby), telemetri prediksi, notifikasi, dan jejak audit.
- **Implementasi Strict Gate Authentication & Penyelarasan Kontrak OpenAPI 3.0:**
  - Hashing kata sandi Bcrypt minimal 12 putaran (`rounds=12`).
  - Penerbitan dan validasi token JWT berbasis algoritma HS256 dengan masa aktif 7 hari.
  - Middleware penjaga akses `@token_required` pada `app/middlewares/auth_guard.py`.
  - Endpoint `POST /api/v1/auth/register` dengan validasi kelengkapan nama, keabsahan email, kekuatan sandi (min 8 karakter), dan kecocokan `confirm_password`.
  - Endpoint `POST /api/v1/auth/login` dengan struktur payload bersarang (`data.access_token`, `data.token_type`, `data.expires_in`, dan `data.user` memuat `phone_number`).
  - Endpoint `GET /api/v1/auth/me` untuk pengembalian data profil pengguna aktif terotentikasi.
- **Web Admin Dashboard & Manajemen Sesi (`/admin` - PR #1 Merged):**
  - Antarmuka web administratif modern bertema *Dark Zinc Engineering Console* (`login.html` dan `dashboard.html`) dengan tipografi *JetBrains Mono* + *Plus Jakarta Sans*.
  - Sistem autentikasi sesi admin terproteksi dekorator `@admin_login_required` pada rute `/admin/login` dan `/admin/dashboard`.
  - Tombol cepat *Isi Otomatis Kredensial Demo* untuk efisiensi demonstrasi presentasi.
  - Pemantauan 4 metrik KPI utama ($R^2$, MAE, Latensi $118$ ms, Volume Dataset), 2 grafik interaktif Chart.js, konsol simulasi *Live Inference Workbench* (rekam langsung ke `prediction_logs`), dan 4 tab penjelajah data (Telemetri Prediksi, Registri Model dengan aksi switch model, Statistik Pasar, dan Audit Trail).
  - Pencatatan aktivitas login, logout, retrain trigger, dan switch model ke `system_audit_logs`.
- **Implementasi Blueprint Master Data & Statistik Pasar API (`/api/v1`):**
  - Endpoint publik `GET /api/v1/locations` yang menyajikan data 46 kecamatan dikelompokkan ke dalam 5 kota administratif DKI Jakarta untuk kebutuhan *cascading dropdown* mobile.
  - Endpoint publik `GET /api/v1/market-statistics` yang mengembalikan agregasi harga pasar per $m^2$ dengan dukungan filter query parameter `?city=` dan `?district=`.
- **Otomasi Pengujian Mutu (PMPL Pytest Test Suite):**
  - Pembuatan direktori `tests/` memuat fixture `conftest.py`, `test_market_api.py`, dan `test_auth_api.py`.
  - Seluruh 7 pengujian otomatis integrasi API (locations, market stats, register, login, me, invalid auth) berhasil lulus 100% (*7 passed in 2.65s*).
- **Implementasi ML Service Layer & Prediction API (`/api/v1/predict` - PR #3 Merged):**
  - Modul `app/services/ml_service.py` untuk pemuatan artefak Scikit-Learn (`models_artefacts/model.pkl`) dengan *memory caching* dan *smart heuristic fallback* berbasis rata-rata harga pasar riil DKI Jakarta.
  - Perhitungan otomatis rentang batas toleransi kewajaran harga ($\pm 10\%$) serta pengukuran latensi eksekusi komputasi server ($< 1.500$ ms).
  - Endpoint `POST /api/v1/predict` (Strict Gate JWT) dengan validasi skema masukan fisik properti (`surface_area >= 1`, `building_area >= 1`, `bed >= 0`, `bath >= 0`, `carport >= 0`), penolakan `400 INVALID_INPUT_SCHEMA` dengan rincian `details`, serta pencatatan otomatis ke tabel `prediction_logs` yang terikat ke akun pengguna.
  - Endpoint `GET /api/v1/predict/history` (Strict Gate JWT) untuk menyajikan riwayat kalkulasi pengguna aktif dengan isolasi data antarakun dan dukungan paginasi parameter `limit` dan `offset`.
  - Penambahan 5 skenario uji otomatis pada `tests/test_predict_api.py`, meningkatkan cakupan pengujian Pytest menjadi 12 test cases dengan tingkat kelulusan 100% (*12 passed in 8.67s*).
- **Implementasi Profil Pengguna, FCM Token, Logout & In-App Notifications API:**
  - Endpoint `PUT /api/v1/auth/profile` (Strict Gate JWT) untuk pembaruan profil nama, kontak telepon, dan alur penggantian kata sandi dengan verifikasi hash sandi lama serta syarat kekuatan sandi baru (min 8 karakter).
  - Endpoint `POST /api/v1/auth/fcm-token` (Strict Gate JWT) untuk sinkronisasi token Firebase Cloud Messaging milik perangkat pengguna ke kolom `users.fcm_token`.
  - Endpoint `POST /api/v1/auth/logout` (Strict Gate JWT) untuk terminasi sesi aplikasi mobile dan pengosongan `fcm_token`.
  - Endpoint `GET /api/v1/notifications` (Strict Gate JWT) untuk memuat daftar notifikasi pengguna dengan kalkulasi dinamis jumlah pesan belum dibaca (`unread_count`).
  - Endpoint `PATCH /api/v1/notifications/<id>/read` (Strict Gate JWT) untuk menandai notifikasi telah dibaca dengan penjagaan akses kepemilikan data (404 jika bukan milik pengguna aktif).
  - Penambahan 6 skenario uji otomatis baru pada `tests/test_notifications_and_profile_api.py`, menghasilkan total 18 skenario uji Pytest lulus 100% (*18 passed in 22.54s*).
- **Implementasi Manajemen Dataset CSV Web Admin (FR-WEB-03):**
  - Formulir unggah berkas CSV (`/admin/dataset/upload`) dengan opsi rasio pemisahan *train-test split* (80:20, 70:30, 85:15).
  - Validasi format berkas `.csv`, penyimpanan ke direktori terisolasi `uploads/datasets/`, serta kalkulasi otomatis `total_records` dan `file_size_bytes` ke tabel `dataset_metadata`.
  - Pencatatan jejak audit aktivitas ke `system_audit_logs` (`DATASET_UPLOAD` - FR-WEB-05).
  - Integrasi tab kelima (*Dataset Manager*) pada Web Admin Dashboard yang menyajikan ringkasan metadata dataset aktif dan pratinjau struktur data tabular (10 baris pertama).
  - Penambahan 4 skenario uji otomatis pada `tests/test_admin_dataset.py`, meningkatkan total cakupan pengujian Pytest menjadi 22 test cases dengan tingkat kelulusan 100% (*22 passed in 24.16s*).
- **Orkestrasi Kontainerisasi & Panduan Handover:**
  - Konfigurasi multi-layanan `docker-compose.yml` (`db` PostgreSQL 15 Alpine dengan *healthcheck* dan `backend` Flask Gunicorn pada jaringan `houseprice-net`).
  - Penyiapan direktori paket `app/services/` sebagai kerangka layer logika bisnis dan ML runtime.
  - Penyusunan berkas panduan teknis dan alokasi tugas transisi Sprint 2 (`docs/tasks/backend-dev-notes.md`).

### Diubah & Diperbarui (Changed - Mobile App Sprint 1 Refinement)
- **Penyederhanaan Tombol Aksi Hasil Prediksi Harga Properti (`ResultPage`):**
  - Menghapus tombol **"Simpan"** dan tombol **"Bagikan Estimasi Nilai"**.
  - Menggantikannya dengan tombol aksi tunggal **"Hitung Ulang"** (`GradientButton` dengan ikon `Icons.refresh_rounded`) yang mengembalikan pengguna ke formulir input spesifikasi properti.
- **Penyederhanaan Tombol Aksi Hasil Simulasi KPR (`KprResultPage`):**
  - Menghapus tombol **"Simpan Hasil Simulasi ke Riwayat"**.
  - Menggantikannya dengan tombol aksi tunggal **"Hitung Ulang"** (`GradientButton` dengan ikon `Icons.refresh_rounded`) yang mengembalikan pengguna ke formulir kalkulator KPR.
- **Otomatisasi Penyimpanan Riwayat Perhitungan:**
  - Menambahkan method `HistoryBottomSheet.addRecord(...)` untuk otomatis menyimpan riwayat kalkulasi di urutan teratas tanpa memerlukan tombol simpan manual.
  - Form Prediksi (`PredictFormPage`) otomatis mencatat hasil estimasi nilai dan nama lokasi ke riwayat saat pengguna menekan tombol "Prediksi Harga".
  - Form KPR (`KprFormPage`) otomatis mencatat nominal angsuran per bulan dan tenor pinjaman ke riwayat saat pengguna menekan tombol "Hitung Simulasi KPR".
- **Pembaruan Alur Navigasi & Proteksi Akses Riwayat:**
  - Tombol **"Masuk"** ditambahkan di AppBar pojok kanan atas berdampingan dengan ikon riwayat pada `HomePage` dan `ResultPage` khusus bagi pengguna yang belum terautentikasi.
  - Akses riwayat kalkulasi (`HistoryBottomSheet.show`) kini wajib login terlebih dahulu; pengguna yang belum login akan dialihkan ke halaman login disertai pesan SnackBar edukatif.
  - Transisi Splash Screen diperbarui menjadi otomatis berpindah setelah timer 3 detik menuju Carousel Onboarding tanpa tombol masuk manual.
- **Penguatan Pengujian Otomatis (QA):**
  - Menambahkan pengujian widget untuk memastikan tombol "Hitung Ulang" tampil dan tombol "Simpan"/"Bagikan" tidak lagi muncul di layar hasil.
  - Menambahkan pengujian otomatisasi pencatatan riwayat kalkulasi lokal.
  - Seluruh 16 unit & widget tests dan analisis statis `flutter analyze` berhasil lulus 100%.

### Ditambahkan (Added - Mobile App Sprint 1)
- **Struktur Fondasi Clean Architecture:**
  - Setup folder arsitektur `lib/core/` (constants: `app_colors.dart`, `app_sizes.dart`, `app_typography.dart`; theme: `app_theme.dart`) dan `lib/features/` berorientasi fitur presentation (`pages/` dan `widgets/`).
  - Implementasi scaffold navigasi `MainNavigationWrapper` menggunakan Material 3 `NavigationBar` (Beranda, Prediksi, Riwayat, Favorit, Pengaturan).
- **Slicing Antarmuka & Validasi UI (UI Mockup):**
  - **Beranda (Home):** Header salam dinamis, kartu ringkasan pasar properti DKI Jakarta, tombol aksi cepat (Mulai Prediksi, Simulasi KPR, Cek Kewajaran, Riwayat), dan kartu rekomendasi properti.
  - **Form Prediksi Properti:** Form 7 parameter spesifikasi rumah lengkap dengan validasi lokal (luas tanah min. 30 m², luas bangunan, kamar tidur, kamar mandi, carport, dropdown wilayah kota administratif & kecamatan).
  - **Hasil Estimasi Properti:** Kartu nominal estimasi harga (Rp), rentang pasar wajar toleransi $\pm 10\%$, ringkasan spesifikasi, input evaluasi kewajaran harga penawaran penjual, dan tombol aksi "Hitung Ulang".
  - **Riwayat & Favorit:** Antarmuka daftar riwayat kalkulasi estimasi offline dan daftar properti tersimpan dengan kartu informasi interaktif.
  - **Pengaturan:** Halaman konfigurasi profil, preferensi notifikasi, mode gelap/terang, dan informasi pengembang PBL.

### Sedang Dikerjakan (In Progress - Target Menuju Milestone 2)
- **Machine Learning & Model Integration:**
  - Finalisasi artefak model serialisasi `.pkl` dari repositori `ml-model-testing` untuk disematkan ke `models_artefacts/model.pkl` (@nurilhuda).
  - Pembentukan service layer `app/services/ml_service.py` untuk pemuatan model dan evaluasi batas toleransi wajar $\pm 10\%$.
- **Backend & RESTful API (Flask):**
  - Endpoint `POST /api/v1/predict` (Strict Gate inferensi + pencatatan telemetri user) dan `GET /api/v1/predict/history`.
  - Endpoint pelengkap user profile (`PUT /api/v1/auth/profile`, `POST /api/v1/auth/fcm-token`, `POST /api/v1/auth/logout`).
  - Endpoint in-app notifications (`GET /api/v1/notifications`, `PATCH /api/v1/notifications/<id>/read`).
  - Web Admin CSV Dataset Management (Upload berkas & pratinjau tabel CSV).
- **Aplikasi Mobile (Flutter):**
  - Integrasi arsitektur State Management (Provider/BLoC) dan REST API client untuk menghubungkan form input ke endpoint backend di Sprint 2.

### Direncanakan (Planned - Target Milestone 2 & Final Expo)
- Integrasi server cloud berbasis Coolify dan konfigurasi SSL HTTPS Let's Encrypt.
- Fitur Cek Kewajaran Harga dengan indikator persentase deviasi penawaran penjual.
- Kalkulator simulasi pembiayaan KPR dengan metode bunga anuitas bulanan.
- Penyusunan Software Test Report (STR) komprehensif dan penutupan seluruh tiket cacat pada Defect Tracking Log.

---

## [0.1.0-proposal] - 2026-10-04

### Ditambahkan (Added)
- Dokumen Proposal Proyek PBL HousePrice yang disetujui untuk Program Studi D-IV Teknik Informatika Polinema.
- Dokumen Perencanaan Proyek mencakup pembagian fitur (Mobile, Web, Cloud) dan skema basis data 7 tabel.
- Diagram Entity Relationship Diagram (ERD) berbasis Eraser.io yang mendefinisikan entitas relasional sistem.
- Panduan percabangan Git (*Branching Ruleset*) untuk repositori mobile dan backend/ML.
- Penggunaan dataset properti DKI Jakarta dari Kaggle sebanyak 10.000 baris data sebagai data primer.
- Matriks alokasi peran dan tanggung jawab 5 anggota tim lintas 3 mata kuliah terintegrasi (Pemrograman Mobile, Pembelajaran Mesin, dan PMPL).
