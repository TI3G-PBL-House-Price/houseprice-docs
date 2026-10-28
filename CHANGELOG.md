# Catatan Perubahan (Changelog) - HousePrice

Seluruh pembaruan penting pada proyek HousePrice dicatat dalam berkas ini. Format pencatatan mengacu pada prinsip [Keep a Changelog](https://keepachangelog.com/id/1.0.0/) dan menganut penomoran versi [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

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

### Sedang Dikerjakan (In Progress - Target Milestone 1 / Minggu 8)
- **Machine Learning & Data Pipeline:**
  - Analisis data eksploratif (EDA) dan visualisasi korelasi fitur pada dataset `jakarta_house.csv`.
  - Pembersihan baris data ganda (penanganan 526 baris duplikat).
  - Imputasi nilai kosong (*missing values*) pada kolom `surface_area` dan `building_area`.
  - Penanganan nilai pencilan (*outliers*) menggunakan ambang batas Interquartile Range (IQR).
  - Penskalaan fitur, One-Hot Encoding wilayah, dan pembagian dataset train-test (80:20).
  - Pelatihan model regresi dasar dan algoritma ensemble (Random Forest Regressor) hingga ekspor file serialisasi `.pkl`.
- **Aplikasi Mobile (Flutter):**
  - Integrasi arsitektur State Management (Provider/BLoC) dan REST API client untuk menghubungkan form input ke backend ML di Sprint 2.
- **Backend & RESTful API (Flask):**
  - Endpoint `POST /api/v1/predict` untuk inferensi estimasi harga rumah.
  - Endpoint `GET /api/v1/market-statistics` untuk penyediaan agregasi data ke mobile.
  - Konfigurasi migrasi skema awal PostgreSQL (tabel `admins`, `master_locations`, `ml_models`, `prediction_logs`).
- **Penjaminan Mutu Perangkat Lunak (PMPL):**
  - Pembuatan test case Blackbox (Boundary Value Analysis & Equivalence Partitioning) untuk form properti.
  - Skrip pengujian otomatis integrasi API dan uji latensi respons inferensi via Postman Runner.

### Direncanakan (Planned - Target Milestone 2 & Final Expo)
- Integrasi server cloud berbasis Coolify dan konfigurasi SSL HTTPS Let's Encrypt.
- Fitur Cek Kewajaran Harga dengan indikator persentase deviasi penawaran penjual.
- Kalkulator simulasi pembiayaan KPR dengan metode bunga anuitas bulanan.
- Penyimpanan riwayat prediksi lokal pengguna (SQLite / SharedPreferences).
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