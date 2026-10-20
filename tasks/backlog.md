# Product Backlog Proyek (Tasks Backlog) - HousePrice

Dokumen ini memuat daftar inventaris seluruh kebutuhan kerja, tugas implementasi, dan pengujian sistem HousePrice yang terdistribusi ke dalam tiga mata kuliah terintegrasi (Pemrograman Mobile, Pembelajaran Mesin, dan Penjaminan Mutu Perangkat Lunak)[cite: 13, 19, 20].

---

## 1. Tata Kelola & Skala Prioritas Backlog

Setiap item backlog diklasifikasikan menggunakan metode prioritas **MoSCoW**:
* **Must Have (M):** Kebutuhan inti fungsional yang wajib tersedia agar produk dianggap berjalan (*runnable product*) pada evaluasi *Milestone* dan *Expo*[cite: 8, 14].
* **Should Have (S):** Fitur pendukung penting yang melengkapi pengalaman pengguna dan keandalan sistem[cite: 14, 20].
* **Could Have (C):** Fitur pelengkap bernilai tambah yang dieksekusi bila waktu pengembangan memungkinkan[cite: 14, 21].
* **Won't Have (W):** Fitur di luar ruang lingkup proyek (*out of scope*, seperti *heatmap* spasial dan transaksi KPR bank langsung)[cite: 21, 22].

---

## 2. Backlog Mata Kuliah Pemrograman Mobile (Flutter)

Repositori: `houseprice-mobile-app` (Kondisi saat ini: Desain UI siap, menunggu inisialisasi kode & *slicing*)[cite: 31]

| Kode Task | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target Sprint | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **MOB-01** | Fondasi Arsitektur | Inisialisasi proyek Flutter (`flutter create`), arsitektur folder berlapis (*presentation, business_logic, data*), konfigurasi rute, dan setup UI Design System sesuai mockup[cite: 25, 26, 31]. | M | Sprint 1 | Jasmine[cite: 19, 31] | **To Do** |
| **MOB-02** | Beranda (Fitur 1) | *Slicing* antarmuka Beranda dari desain UI: ringkasan statistik pasar properti dan filter pemilihan kota administrasi[cite: 24, 31]. | M | Sprint 1 | Jasmine[cite: 19, 31] | **To Do** |
| **MOB-03** | Form Input (Fitur 2) | *Slicing* form spesifikasi properti (luas tanah, luas bangunan, kamar tidur, kamar mandi, carport, dropdown wilayah) serta logika validasi input lokal (anti-minus & non-nol)[cite: 24, 27, 31]. | M | Sprint 1 | Hanif[cite: 19, 31] | **To Do** |
| **MOB-04** | Core Networking (Fitur 2) | Integrasi HTTP Client (REST API) dan State Management (Provider/BLoC) untuk menghubungkan form mobile ke backend inferensi cloud[cite: 26, 31]. | M | Sprint 2 | Nuril[cite: 19, 31] | To Do |
| **MOB-05** | Layar Hasil (Fitur 2) | Pembuatan kartu visual penampil nominal estimasi harga beserta rentang batas wajar toleransi (*Lower-Upper Bound*)[cite: 24, 25, 31]. | M | Sprint 2 | Nurfinka[cite: 19, 31] | To Do |
| **MOB-06** | Cek Kewajaran (Fitur 3) | Implementasi masukan harga penawaran penjual dan logika evaluasi status (*Murah, Wajar, Mahal*)[cite: 24, 25, 31]. | M | Sprint 2 | Naufal[cite: 19, 31] | To Do |
| **MOB-07** | Simulasi KPR (Fitur 4) | Pembuatan kalkulator cicilan KPR metode bunga anuitas bulanan berdasarkan variabel DP, bunga tahunan, dan tenor kredit[cite: 25, 26, 31]. | M | Sprint 2 | Naufal[cite: 19, 31] | To Do |
| **MOB-08** | Riwayat Lokal (Fitur 5) | Implementasi basis data lokal SQLite / SharedPreferences untuk menyimpan, membaca, dan menghapus histori simulasi secara *offline*[cite: 25, 31]. | S | Sprint 2 | Nurfinka[cite: 19, 31] | To Do |
| **MOB-09** | UI Polish & Error View | Penanganan indikator *loading shimmer*, dialog koneksi putus (*graceful degradation*), dan adaptabilitas layar Android[cite: 25, 27, 32]. | S | Sprint 3 | Jasmine & Nuril[cite: 19, 32] | To Do |

---

## 3. Backlog Mata Kuliah Pembelajaran Mesin & Backend

Trek Eksperimen ML: `ml-model-testing` | Trek Backend Produksi: `houseprice-backend-ml`[cite: 33]

| Kode Task | Repositori | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **ML-01** | `ml-model-testing` | Eksplorasi Data (EDA) | Analisis statistik deskriptif dan visualisasi korelasi fitur pada dataset `jakarta_house.csv`[cite: 27, 28, 33]. | M | Sprint 1 | Hanif[cite: 19, 28, 33] | **Done** |
| **ML-02** | `ml-model-testing` | Data Cleaning (Duplicates) | Pembersihan dan verifikasi pembuangan 526 baris data ganda[cite: 27, 28, 33]. | M | Sprint 1 | Jasmine[cite: 19, 28, 33] | **Done** |
| **ML-03** | `ml-model-testing` | Data Cleaning (Missing) | Penanganan nilai kosong (*missing values*) pada atribut `surface_area` dan `building_area`[cite: 27, 29, 33]. | M | Sprint 1 | Nurfinka[cite: 19, 29, 33] | **In Progress** |
| **ML-04** | `ml-model-testing` | Preprocessing & Encoding | Pemfilteran *outlier* berbasis batas IQR, One-Hot Encoding fitur wilayah, penskalaan fitur numerik, dan pembagian dataset (80:20)[cite: 27, 29, 33]. | M | Sprint 1 | Naufal[cite: 19, 29, 33] | **In Progress** |
| **ML-05** | `ml-model-testing` | Benchmarking & Seleksi Model | Pengujian komparasi model regresi: Random Forest (telah diuji) vs Linear Regression vs Gradient Boosting/XGBoost, pencatatan metrik ($R^2 \ge 0,80$, MAE, RMSE), pemilihan model final, dan ekspor berkas serialisasi `.pkl`[cite: 27, 29, 33]. | M | Sprint 1 | Nuril[cite: 19, 29, 33] | **In Progress** |
| **ML-06** | `houseprice-backend-ml` | Setup Backend Skeleton | Inisialisasi struktur repositori Flask, Blueprints rute, konfigurasi virtual environment, `Dockerfile`, dan `docker-compose.yml`[cite: 23, 40, 41]. | M | Sprint 1-2 | Hanif & Nuril[cite: 19, 28, 33] | **To Do** |
| **ML-07** | `houseprice-backend-ml` | REST API Inferensi | Implementasi endpoint `POST /api/v1/predict`, `GET /api/v1/market-statistics`, dan `GET /api/v1/locations` menggunakan model `.pkl` terpilih[cite: 23, 27, 40]. | M | Sprint 2 | Hanif & Nuril[cite: 19, 28, 33] | **To Do** |
| **ML-08** | `houseprice-backend-ml` | Basis Data PostgreSQL | Migrasi DDL 7 tabel relasional, connection pool psycopg2/SQLAlchemy, dan pencatatan telemetri `prediction_logs`[cite: 1, 23]. | M | Sprint 2 | Hanif[cite: 19, 28] | **To Do** |
| **ML-09** | `houseprice-backend-ml` | Web Admin Dashboard | Antarmuka web pengelola: ringkasan metrik model aktif, monitoring volume data, manajemen unggah CSV, dan eksekusi retrain model[cite: 23, 28]. | S | Sprint 2 | Nuril & Hanif[cite: 19, 28] | **To Do** |
| **ML-10** | `houseprice-backend-ml` | Cloud Deployment | Penerapan backend dan PostgreSQL ke platform Coolify PaaS dengan otomatisasi SSL HTTPS Let's Encrypt[cite: 41]. | M | Sprint 2 | Hanif[cite: 19, 28] | **To Do** |

---

## 4. Backlog Mata Kuliah Penjaminan Mutu Perangkat Lunak (PMPL)

Keterlibatan lintas repositori[cite: 19, 31, 33]

| Kode Task | Modul / Fitur | Deskripsi Pekerjaan | Prioritas | Target | Penanggung Jawab | Status Riil |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **QA-01** | Uji Form Blackbox | Perancangan skenario uji dan eksekusi *Boundary Value Analysis* (BVA) serta *Equivalence Partitioning* (EP) pada form input properti setelah selesai di-*slicing*[cite: 19, 27, 31]. | M | Sprint 1 | Jasmine[cite: 19, 29, 31] | **To Do** |
| **QA-02** | Uji Logika Bisnis | Verifikasi matematis kalkulasi amortisasi bunga anuitas KPR dan formula deviasi kewajaran harga ($\pm 10\%$)[cite: 19, 25, 32]. | M | Sprint 2 | Naufal[cite: 19, 29, 32] | **To Do** |
| **QA-03** | Uji Integrasi & Latensi API | Pengujian otomatisasi endpoint inferensi backend via Postman Runner untuk memverifikasi latensi $< 1,5$ detik dan penanganan error 400 Bad Request[cite: 19, 27, 33]. | M | Sprint 2 | Hanif[cite: 19, 28, 33] | **To Do** |
| **QA-04** | Uji Pengalaman Pengguna | Pengujian Usability dan responsivitas layout visual pada perangkat fisik Android (bebas galat *overflow*)[cite: 19, 27, 32]. | M | Sprint 3 | Nuril[cite: 19, 29, 32] | **To Do** |
| **QA-05** | Defect Management & STR | Setup lembar pelacakan cacat (*Defect Tracking Log*) dan penyusunan laporan formal *Software Test Report* (STR)[cite: 19, 29, 32]. | M | Sprint 1-3 | Nurfinka[cite: 19, 29, 32] | **In Progress** |

---

## 5. Alokasi Milestone PBL (Garis Waktu 16 Minggu)

Distribusi target capaian mengacu pada ketentuan perkuliahan semester 5[cite: 8, 28]:

* **Sprint 0 (Minggu 1-4) [SELESAI]:** Penyusunan dan persetujuan proposal produk, perencanaan skema ERD, studi kelayakan dataset, dan pembagian peran tim[cite: 8, 28].
* **Sprint 1 (Minggu 5-8) [AKTIF]:** Selesainya benchmarking model ML di `ml-model-testing`, inisiasi proyek Flutter & *slicing* UI dasar, pembuatan kerangka awal Flask backend, dan uji BVA form[cite: 8, 28, 31, 33]. *(Milestone 1: Demo Progres Tengah Semester)*[cite: 8, 28]
* **Sprint 2 (Minggu 9-12):** Integrasi penuh REST API Flask, database PostgreSQL, fitur KPR, Cek Kewajaran, penyimpanan riwayat lokal, dan uji performa API[cite: 8, 28, 31, 33]. *(Milestone 2: Demo Produk Fungsional)*[cite: 8, 28]
* **Sprint 3 (Minggu 13-16):** Uji coba menyeluruh QA, penutupan bug, penyusunan Software Test Report (STR), deploy cloud Coolify, dan persiapan Expo PBL[cite: 8, 28, 32, 41]. *(Milestone 3: Demo Day Final)*[cite: 8, 28]