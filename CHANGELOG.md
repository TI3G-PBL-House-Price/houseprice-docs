# Catatan Perubahan (Changelog) - HousePrice

Seluruh pembaruan penting pada proyek HousePrice dicatat dalam berkas ini. Format pencatatan mengacu pada prinsip [Keep a Changelog](https://keepachangelog.com/id/1.0.0/) dan menganut penomoran versi [Semantic Versioning](https://semver.org/).

---

## [Unreleased]

### Sedang Dikerjakan (In Progress - Target Milestone 1 / Minggu 8)
- **Machine Learning & Data Pipeline:**
  - Analisis data eksploratif (EDA) dan visualisasi korelasi fitur pada dataset `jakarta_house.csv`.
  - Pembersihan baris data ganda (penanganan 526 baris duplikat).
  - Imputasi nilai kosong (*missing values*) pada kolom `surface_area` dan `building_area`.
  - Penanganan nilai pencilan (*outliers*) menggunakan ambang batas Interquartile Range (IQR).
  - Penskalaan fitur, One-Hot Encoding wilayah, dan pembagian dataset train-test (80:20).
  - Pelatihan model regresi dasar dan algoritma ensemble (Random Forest Regressor) hingga ekspor file serialisasi `.pkl`.
- **Aplikasi Mobile (Flutter):**
  - Implementasi UI Design System dasar dan navigasi aplikasi.
  - Tampilan Beranda (Home) ringkasan data pasar properti Jakarta.
  - Form input spesifikasi properti lengkap dengan validasi lokal (pencegahan input angka minus, nol, atau teks).
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