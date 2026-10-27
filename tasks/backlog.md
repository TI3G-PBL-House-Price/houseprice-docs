# Product Backlog Proyek (Tasks Backlog) - HousePrice

Dokumen ini memuat daftar inventaris seluruh kebutuhan kerja, tugas implementasi, dan pengujian sistem HousePrice yang terdistribusi ke dalam tiga mata kuliah terintegrasi (Pemrograman Mobile, Pembelajaran Mesin, dan Penjaminan Mutu Perangkat Lunak)[cite: 13].

---

## 1. Tata Kelola & Skala Prioritas Backlog

Setiap item backlog diklasifikasikan menggunakan metode prioritas **MoSCoW**:
* **Must Have (M):** Kebutuhan inti fungsional yang wajib tersedia agar produk dianggap berjalan (*runnable product*) pada evaluasi *Milestone* dan *Expo*[cite: 8, 14].
* **Should Have (S):** Fitur pendukung penting yang melengkapi pengalaman pengguna dan keandalan sistem.
* **Could Have (C):** Fitur pelengkap bernilai tambah yang dieksekusi bila waktu pengembangan memungkinkan[cite: 14].
* **Won't Have (W):** Fitur di luar ruang lingkup proyek (*out of scope*, seperti *heatmap* spasial, login publik cloud, dan pengajuan resmi KPR bank).

---

## 2. Backlog Mata Kuliah Pemrograman Mobile (Flutter)

Repositori: `houseprice-mobile-app` (Kondisi saat ini: Inisialisasi arsitektur Clean Architecture dan *slicing* UI mockup selesai)

| Kode Task | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target Sprint | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **MOB-01** | Fondasi Arsitektur | Inisialisasi struktur Clean Architecture (`core/` & `features/`), konfigurasi tema global (`app_colors.dart`, `app_typography.dart`, `app_sizes.dart`), dan scaffold navigasi BottomNavigationBar. | M | Sprint 1 | Jasmine | **Done** |
| **MOB-02** | Beranda (Fitur 1) | *Slicing* antarmuka Beranda dari desain UI: kartu statistik ringkasan pasar properti, tombol aksi cepat, dan daftar properti rekomendasi. | M | Sprint 1 | Jasmine | **Done** |
| **MOB-03** | Form Input (Fitur 2) | *Slicing* form 7 spesifikasi properti (luas tanah min. 30 m², bangunan, kamar, carport, dropdown wilayah) dan validasi masukan lokal. | M | Sprint 1 | Hanif | **Done** |
| **MOB-04** | Core Networking (Fitur 2) | Integrasi REST API Client dan State Management (Provider/BLoC) untuk menghubungkan form mobile ke backend inferensi cloud. | M | Sprint 2 | Nuril | To Do |
| **MOB-05** | Layar Hasil (Fitur 2 & 3) | Kartu nominal estimasi harga, rentang pasar wajar, ringkasan spesifikasi, evaluasi kewajaran, dan tombol tunggal **Hitung Ulang** (tombol simpan/bagikan ditiadakan). | M | Sprint 1 | Nurfinka & Nuril | **Done** |
| **MOB-06** | Cek Kewajaran (Fitur 3) | Implementasi komponen input harga penawaran penjual di dalam Layar Hasil dan logika evaluasi status (*Murah, Wajar, Mahal* deviasi $\pm 10\%$). | M | Sprint 1 | Naufal | **Done** |
| **MOB-07** | Simulasi KPR (Fitur 4) | Kalkulator cicilan KPR bunga anuitas bulanan, input pendapatan bulanan, komposisi pokok vs bunga, evaluasi DSR, dan layar hasil dengan tombol **Hitung Ulang**. | M | Sprint 1 | Naufal & Nuril | **Done** |
| **MOB-08** | Riwayat Perhitungan (Fitur 5) | Penyimpanan riwayat kalkulasi estimasi & KPR secara otomatis (*auto-record*) saat form disubmit, dengan proteksi autentikasi sebelum membuka riwayat. | M | Sprint 1 | Nurfinka & Nuril | **Done** |
| **MOB-09** | UI Polish & Error View | Penanganan indikator *loading shimmer*, dialog koneksi putus (*graceful degradation*), dan adaptabilitas layar Android. | S | Sprint 3 | Jasmine & Nuril | To Do |
| **MOB-10** | Pre-Auth Flow & Login Gate | Alur Splash Screen timer 3 detik, Onboarding Carousel 3 slide, Landing Page publik (menu bawah), serta tombol Masuk di AppBar. | M | Sprint 1 | Nuril | **Done** |

---

## 3. Backlog Mata Kuliah Pembelajaran Mesin & Backend

Trek Eksperimen ML: `ml-model-testing` | Trek Backend Produksi: `houseprice-backend-ml`

| Kode Task | Repositori | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ML-01** | `ml-model-testing` | Eksplorasi Data (EDA) | Analisis statistik deskriptif dan visualisasi korelasi fitur pada dataset `jakarta_house.csv`. | M | Sprint 1 | Hanif | **Done** |
| **ML-02** | `ml-model-testing` | Data Cleaning (Duplicates) | Pembersihan dan verifikasi pembuangan 526 baris data ganda. | M | Sprint 1 | Jasmine | **Done** |
| **ML-03** | `ml-model-testing` | Data Cleaning (Missing) | Penanganan nilai kosong (*missing values*) pada atribut `surface_area` dan `building_area`. | M | Sprint 1 | Nurfinka | **In Progress** |
| **ML-04** | `ml-model-testing` | Preprocessing & Encoding | Pemfilteran *outlier* berbasis batas IQR, One-Hot Encoding fitur wilayah, penskalaan numerik, dan pemisahan train-test (80:20). | M | Sprint 1 | Naufal | **In Progress** |
| **ML-05** | `ml-model-testing` | Benchmarking & Seleksi Model | Pengujian komparasi model regresi: Random Forest vs Baseline & Gradient Boosting, evaluasi metrik ($R^2 \ge 0,80$), dan ekspor biner `.pkl`. | M | Sprint 1 | Nuril | **In Progress** |
| **ML-06** | `houseprice-backend-ml` | Setup Backend Skeleton | Inisialisasi struktur repositori Flask, Blueprints rute, dependensi Python, `Dockerfile`, dan `docker-compose.yml`. | M | Sprint 1-2 | Hanif & Nuril | **To Do** |
| **ML-07** | `houseprice-backend-ml` | REST API Inferensi | Implementasi endpoint `POST /api/v1/predict`, `GET /api/v1/market-statistics`, dan `GET /api/v1/locations` menggunakan model `.pkl` terpilih. | M | Sprint 2 | Hanif & Nuril | **To Do** |
| **ML-08** | `houseprice-backend-ml` | Basis Data PostgreSQL | Migrasi DDL 7 tabel relasional, connection pool, dan pencatatan telemetri `prediction_logs`[cite: 1]. | M | Sprint 2 | Hanif | **To Do** |
| **ML-09** | `houseprice-backend-ml` | Web Admin Dashboard | Antarmuka web pengelola: ringkasan metrik model aktif, monitoring volume data, manajemen unggah CSV, dan eksekusi retrain model. | S | Sprint 2 | Nuril & Hanif | **To Do** |
| **ML-10** | `houseprice-backend-ml` | Cloud Deployment | Penerapan backend dan PostgreSQL ke platform Coolify PaaS dengan sertifikat SSL HTTPS Let's Encrypt. | M | Sprint 2 | Hanif | **To Do** |

---

## 4. Backlog Mata Kuliah Penjaminan Mutu Perangkat Lunak (PMPL)

Keterlibatan lintas repositori

| Kode Task | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **QA-01** | Uji Form Blackbox | Perancangan skenario uji dan eksekusi BVA (luas tanah min. 30 m²) serta EP pada form input properti mobile. | M | Sprint 1 | Jasmine | **To Do** |
| **QA-02** | Uji Logika Bisnis | Verifikasi matematis kalkulasi bunga anuitas KPR, evaluasi rasio DSR OJK $\le 40\%$, dan formula deviasi kewajaran harga ($\pm 10\%$). | M | Sprint 2 | Naufal | **To Do** |
| **QA-03** | Uji Integrasi & Latensi API | Pengujian otomatisasi endpoint inferensi backend via Postman Runner untuk memverifikasi latensi $< 1,5$ detik dan respon kode status HTTP. | M | Sprint 2 | Hanif | **To Do** |
| **QA-04** | Uji Pengalaman Pengguna | Pengujian Usability dan responsivitas layout visual pada perangkat fisik Android (bebas galat *overflow*). | M | Sprint 3 | Nuril | **To Do** |
| **QA-05** | Defect Management & STR | Setup lembar pelacakan cacat (*Defect Tracking Log*) dan penyusunan laporan formal *Software Test Report* (STR). | M | Sprint 1-3 | Nurfinka | **In Progress** |

---

## 5. Alokasi Milestone PBL (Garis Waktu 16 Minggu)

* **Sprint 0 (Minggu 1-4) [SELESAI]:** Penyusunan dan persetujuan proposal produk, perencanaan skema ERD, studi kelayakan dataset, dan pembagian peran tim[cite: 8].
* **Sprint 1 (Minggu 5-8) [AKTIF]:** Selesainya benchmarking model ML di `ml-model-testing`, inisiasi proyek Flutter & *slicing* UI dasar, pembuatan kerangka awal Flask backend, dan uji BVA form[cite: 8]. *(Milestone 1: Demo Progres Tengah Semester)*[cite: 8]
* **Sprint 2 (Minggu 9-12):** Integrasi penuh REST API Flask, database PostgreSQL, fitur KPR & DSR, Cek Kewajaran, penyimpanan riwayat lokal SQLite, dan uji performa API[cite: 8]. *(Milestone 2: Demo Produk Fungsional)*[cite: 8]
* **Sprint 3 (Minggu 13-16):** Uji coba menyeluruh QA, penutupan bug, penyusunan Software Test Report (STR), deploy cloud Coolify, dan persiapan Expo PBL[cite: 8]. *(Milestone 3: Demo Day Final)*[cite: 8]