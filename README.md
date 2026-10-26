# HousePrice: Sistem Prediksi Harga Rumah Wilayah DKI Jakarta Berbasis Machine Learning & Mobile

[![PBL Semester 5](https://img.shields.io/badge/PBL-D4%20Teknik%20Informatika%20Polinema-blue.svg)](https://www.polinema.ac.id/)
[![Flutter](https://img.shields.io/badge/Mobile-Flutter%203.x-02569B?logo=flutter)](https://flutter.dev)
[![Flask](https://img.shields.io/badge/Backend-Flask%20Python-000000?logo=flask)](https://flask.palletsprojects.com/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2015+-4169E1?logo=postgresql)](https://www.postgresql.org/)
[![JWT](https://img.shields.io/badge/Auth-Strict%20Gate%20JWT-red.svg)](https://jwt.io/)
[![License](https://img.shields.io/badge/License-Academic%20PBL-green.svg)]()

> Repositori Pusat Dokumentasi & Standar Arsitektur Sistem untuk Proyek Terintegrasi Product-Based Learning (PBL) Semester 5, Jurusan Teknologi Informasi, Politeknik Negeri Malang (T.A. 2026/2027)[cite: 10].

---

## 📌 Ringkasan Eksekutif
Masyarakat dan keluarga muda sering menghadapi asimetri informasi saat membeli atau menjual rumah di DKI Jakarta, sehingga sulit menilai apakah harga penawaran tergolong murah, wajar, atau terlalu mahal[cite: 10]. 

**HousePrice** hadir sebagai solusi komprehensif terintegrasi berbasis keamanan ketat (*Strict Gate Authentication*)[cite: 3, 10]:
1. **Aplikasi Mobile (Flutter):** Mengimplementasikan alur Splash Screen dinamis, Carousel Onboarding, Landing Page, pendaftaran akun & login terenkripsi, visualisasi tren pasar, inferensi harga properti instan, evaluasi kelayakan harga penawaran pasar, kalkulator simulasi KPR (anuitas), pusat notifikasi in-app, serta profil pengguna yang memuat riwayat kalkulasi terintegrasi cloud[cite: 2, 8, 10].
2. **RESTful Inference & Services API (Flask):** Mesin inferensi cloud berlatensi rendah (< 1,5 detik) berbasis model Machine Learning Ensemble (*Random Forest Regressor*) terproteksi Bearer JWT Token, layanan otentikasi Bcrypt, agregasi statistik wilayah, serta manajemen pesan notifikasi[cite: 2, 4, 10].
3. **Web Dashboard Admin (Flask & Bootstrap):** Portal khusus administrator teknis untuk pemantauan performa model aktif ($R^2$, MAE, RMSE), manajemen dataset properti CSV, pemicu pelatihan ulang model (*retraining*), dan pencatatan jejak audit sistem (*system audit logs*)[cite: 1, 10].
4. **Basis Data Terpusat (PostgreSQL):** Arsitektur 9 tabel relasional yang mengelola kredensial admin, akun mobile pengguna terotentikasi, master referensi wilayah Jakarta, metadata dataset, registri model ML, statistik pasar, notifikasi pengguna, telemetri prediksi berelasi `user_id`, dan log audit keamanan[cite: 1, 2].

---

## 👥 Tim Pengembang (Kelompok 2 - TI 3G)

| Nama Mahasiswa | NIM | Peran Utama & Fokus MK Terintegrasi |
| :--- | :--- | :--- |
| **Abdul Rahman Hanif Darmawan** | 244107020232 | **Project Manager & QA Integration**, EDA & Visualisasi ML, Form Input Spesifikasi & Validasi Mobile, Uji Integrasi API & Keamanan Token JWT[cite: 7, 10] |
| **Muhammad Nuril Huda** | 244107020004 | **ML & Networking Engineer**, Training & Tuning Model Regresi (.pkl), *Slicing* Antarmuka Modul Baru (Splash, Onboard, Landing, Auth, Notif, Profil), HTTP Client AuthInterceptor & State Management[cite: 7, 9, 10] |
| **Nurfinka Lailasari** | 244107020211 | **QA Lead & Local Storage**, Pembersihan Missing Values ML, Penyimpanan Sesi Aman (`flutter_secure_storage`), Profil & Riwayat Prediksi Cloud, Defect Tracking & STR[cite: 7, 9, 10] |
| **Naufal Abid Aurizky** | 244107020212 | **Preprocessing & Business Logic**, Penanganan Outliers IQR & Feature Encoding ML, Logika Bisnis Evaluasi Harga Pasar, Formula Bunga Anuitas KPR, Endpoint Notifikasi API[cite: 7, 9, 10] |
| **Jasmine Nasywa Nabilah** | 244107020119 | **Data Duplicate & Mobile UI**, Pembersihan Baris Duplikat ML, UI Design System Mobile, Slicing Beranda Pasar Properti, Blackbox Testing (BVA/EP) Form[cite: 7, 9, 10] |

### 👨‍🏫 Dosen Pembimbing Mata Kuliah Terintegrasi
* **Pemrograman Mobile:** Habibie Ed Dien, S.Kom., M.T.[cite: 10]
* **Pembelajaran Mesin:** Usman Nurhasan, S.Kom., M.T.[cite: 10]
* **Penjaminan Mutu Perangkat Lunak:** M. Hasyim Ratsanjani, S.Kom., M.Kom.[cite: 10]

---

## 🗺️ Struktur Repositori Organisasi

Proyek ini dikelola secara modular pada organisasi GitHub resmi [**TI3G-PBL-House-Price**](https://github.com/TI3G-PBL-House-Price):

* [**`houseprice-docs`**](https://github.com/TI3G-PBL-House-Price/houseprice-docs) (Repositori Ini): Sumber kebenaran tunggal dokumentasi, mencakup PRD, diagram arsitektur C4, spesifikasi OpenAPI 3.0, skema DDL data model, kebijakan privasi & keamanan, serta panduan pengujian mutu PMPL[cite: 2, 3, 10].
* [**`houseprice-mobile-app`**](https://github.com/TI3G-PBL-House-Price/houseprice-mobile-app): Kode sumber aplikasi Android Flutter berbasis Clean Architecture, manajemen sesi aman (`flutter_secure_storage`), 5 Tab Bottom Navigation, dan state management BLoC/Cubit[cite: 3, 8, 10].
* [**`houseprice-backend-ml`**](https://github.com/TI3G-PBL-House-Price/houseprice-backend-ml): Layanan RESTful API Flask terproteksi JWT, controller Web Admin, skrip migrasi 9 tabel PostgreSQL, dan pemuat model produksi `.pkl`[cite: 2, 3, 10].
* [**`ml-model-testing`**](https://github.com/TI3G-PBL-House-Price/ml-model-testing): Notebook riset eksperimen Machine Learning, pipeline praproses tabular, komparasi algoritma (Random Forest vs XGBoost vs Linear Regression), dan evaluasi metrik ($R^2 \ge 0,80$)[cite: 9, 10].

---

## 📱 Alur Antarmuka & Fitur Utama Mobile

Aplikasi mobile dirancang dengan alur navigasi terproteksi penuh (*Strict Gate Flow*)[cite: 3]:
1. **Inisialisasi & Pengenalan:** `Splash Screen` memeriksa ketersediaan sesi $\rightarrow$ Pengguna baru diarahkan ke `3 Carousel Slide Onboarding` $\rightarrow$ `Landing Page`[cite: 3, 8].
2. **Gerbang Autentikasi (Strict Gate):** Layar `Login` dan `Register` untuk verifikasi kredensial pengguna baru dan penerbitan access token JWT[cite: 2, 8].
3. **5 Tab Dashboard Utama:**
   * **Beranda:** Ringkasan tren harga pasar properti dan filter kota administrasi DKI Jakarta[cite: 8].
   * **Prediksi:** Formulir spesifikasi rumah (LT, LB, kamar tidur, kamar mandi, carport, kecamatan, kota) untuk kalkulasi taksiran nilai pasar[cite: 2, 8].
   * **KPR:** Kalkulator simulasi pembiayaan perbankan berbasis metode anuitas dan evaluasi rasio kelayakan DSR[cite: 7, 8].
   * **Notifikasi:** Pusat informasi perkembangan tren harga per wilayah dan pembaruan sistem in-app[cite: 2, 8].
   * **Profil:** Identitas akun, daftar riwayat pencarian estimasi cloud personal, ubah kata sandi, dan tombol *Logout*[cite: 2, 8].

---

## 🚀 Quick Start & Panduan Deployment

Backend Flask dan basis data PostgreSQL dijalankan secara terisolasi menggunakan platform open-source **Coolify** dengan dukungan **Docker Compose**[cite: 3, 10]:

```bash
# Clone repositori backend
git clone [https://github.com/TI3G-PBL-House-Price/houseprice-backend-ml.git](https://github.com/TI3G-PBL-House-Price/houseprice-backend-ml.git)
cd houseprice-backend-ml

# Salin konfigurasi lingkungan
cp .env.example .env

# Jalankan kontainer backend dan database PostgreSQL
docker compose up -d --build
```