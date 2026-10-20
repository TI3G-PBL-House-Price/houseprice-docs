# Rencana Kerja Sprint Aktif (Current Sprint) - HousePrice

Dokumen ini memuat fokus kerja, papan aktivitas kanban, target capaian, serta pembagian tugas harian seluruh anggota tim untuk siklus **Sprint 1 (Minggu 5 s/d 8)** menuju demonstrasi **Milestone 1: Progres Tengah Semester**[cite: 8, 28].

---

## 1. Status & Realitas Repositori Terkini (Baseline Kondisi Riil)

* **Repositori `ml-model-testing` (Fokus Utama ML Saat Ini):**
  * Status: **Aktif (Fase Eksperimen)**[cite: 33].
  * Progres: Pengujian eksplorasi algoritma sedang berjalan[cite: 27, 33]. Model *Random Forest Regressor* telah diuji coba[cite: 27, 33]. Algoritma pembanding lain (seperti *Multiple Linear Regression* sebagai baseline dan *Gradient Boosting / XGBoost*) sedang disiapkan untuk diuji pada dataset `jakarta_house.csv` sebelum menetapkan model final[cite: 27, 33].
* **Repositori `houseprice-backend-ml`:**
  * Status: **Belum Diinisialisasi (Kosong)**.
  * Rencana: Kode backend Flask, struktur Blueprints, migrasi PostgreSQL, dan layanan inferensi akan dibangun ulang setelah model terbaik di repositori `ml-model-testing` terpilih secara definitif[cite: 1, 23, 27].
* **Repositori `houseprice-mobile-app`:**
  * Status: **Persiapan Slicing UI**.
  * Progres: Desain antarmuka (UI Mockup / Figma) telah tersedia, namun inisialisasi repositori proyek Flutter dan proses konversi desain ke kode (*slicing UI*) belum dimulai[cite: 31].

---

## 2. Target Capaian Sprint 1 (Sprint Goals)

Sprint 1 difokuskan untuk menyelesaikan fondasi pada ketiga repositori sebelum masuk ke tahap integrasi penuh[cite: 8, 31, 33]:
1. **Machine Learning:** Menuntaskan tabel komparasi metrik uji ($R^2$, MAE, RMSE) antar-algoritma pada repositori `ml-model-testing`, memilih satu algoritma dengan performa terbaik ($R^2 \ge 0,80$), serta mengekspor artefak model final `.pkl`[cite: 27, 33, 40].
2. **Mobile (Flutter):** Melakukan inisialisasi basis kode Flutter, menyusun arsitektur folder berlapis (*Layered Architecture*), serta menyelesaikan *slicing* UI untuk Beranda dan Form Spesifikasi Properti lengkap dengan validasi lokal[cite: 24, 25, 31].
3. **Backend Skeleton:** Menginisialisasi repositori `houseprice-backend-ml`, menyiapkan berkas dependensi (`requirements.txt`, `Dockerfile`), dan memuat model `.pkl` hasil eksperimen untuk melayani endpoint inferensi dasar[cite: 27, 41].
4. **PMPL:** Menyusun matriks uji *Boundary Value Analysis* (BVA) dan *Equivalence Partitioning* (EP) untuk memvalidasi masukan form properti saat selesai di-*slicing*[cite: 19, 27, 31].

---

## 3. Papan Tugas Kanban Sprint 1 (Sprint 1 Kanban Board)

```
+-----------------------------------------------------------------------------------+
| BACKLOG / TO DO           | IN PROGRESS                 | DONE                    |
+-----------------------------------------------------------------------------------+
| [MOB] Init Flutter Repo   | [ML] Eksperimen Model       | [DOC] Proposal Disetujui|
| [MOB] Slicing UI Beranda  |      (Random Forest vs      | [DOC] Dokumen Root Selesai|
| [MOB] Slicing Form Input  |       Baseline & Boosting)  | [DES] UI Design Mockup  |
| [BE]  Init Repo Flask     | [QA] Penyusunan Skenario    |                          |
| [BE]  Setup Docker & DB   |      Test Case BVA / EP     |                          |
| [ML]  Export Model .pkl   |                             |                          |
+-----------------------------------------------------------------------------------+
```

### Rincian Pembagian Tugas Anggota Tim:

#### A. Trek Sains Data & Machine Learning (Repo: `ml-model-testing`)
* **Abdul Rahman Hanif Darmawan:**
  * Menyusun dokumentasi perbandingan distribusi fitur pada notebook eksplorasi data (EDA)[cite: 19, 28, 33].
* **Jasmine Nasywa Nabilah:**
  * Memastikan penanganan 526 baris duplikat terintegrasi pada pipeline pembersihan data[cite: 19, 28, 33].
* **Nurfinka Lailasari:**
  * Mengeksekusi penanganan missing values pada atribut `surface_area` dan `building_area`[cite: 19, 29, 33].
* **Naufal Abid Aurizky:**
  * Mengonfigurasi pemfilteran pencilan (*outliers*) berbasis rentang IQR serta One-Hot Encoding fitur wilayah (`city`, `district`)[cite: 19, 27, 33].
* **Muhammad Nuril Huda:**
  * Melanjutkan pengujian algoritma komparasi (Random Forest vs Linear Regression vs Gradient Boosting/XGBoost)[cite: 19, 27, 33].
  * Melakukan pencatatan metrik evaluasi ($R^2$ Score, MAE, RMSE) pada masing-masing model untuk menentukan algoritma pemenang[cite: 19, 27, 33].

#### B. Trek Mobile App (Repo: `houseprice-mobile-app`)
* **Jasmine Nasywa Nabilah:**
  * Melakukan inisialisasi project Flutter (`flutter create`)[cite: 31, 34].
  * Mengonfigurasi UI Design System (skema palet warna, tipografi, dan aset ikon/gambar)[cite: 19, 31].
  * Memulai *slicing* tampilan layar Beranda (ringkasan statistik harga rumah)[cite: 19, 24, 31].
* **Abdul Rahman Hanif Darmawan:**
  * Memulai *slicing* antarmuka Form Spesifikasi Properti (field luas tanah, luas bangunan, jumlah kamar tidur, kamar mandi, carport, dan dropdown wilayah)[cite: 19, 24, 31].
  * Menulis logika validasi masukan form lokal (penolakan angka nol, minus, dan karakter huruf)[cite: 19, 27, 31].

#### C. Trek Backend & Basis Data (Repo: `houseprice-backend-ml`)
* **Abdul Rahman Hanif Darmawan & Muhammad Nuril Huda:**
  * Melakukan inisialisasi struktur repositori backend Flask setelah model ML terpilih[cite: 19, 28, 33].
  * Menyiapkan konfigurasi kontainer `Dockerfile` dan `docker-compose.yml` untuk lingkungan pengujian staging[cite: 41].

#### D. Trek Penjaminan Mutu Perangkat Lunak (PMPL)
* **Jasmine Nasywa Nabilah:**
  * Merancang tabel skenario uji coba Blackbox (BVA dan EP) untuk form properti berdasarkan layout desain yang ada[cite: 19, 27, 31].
* **Nurfinka Lailasari:**
  * Menyiapkan lembar pencatatan cacat (*Defect Tracking Sheet*) untuk mendokumentasikan setiap kendala yang muncul selama proses *slicing* dan pelatihan model[cite: 19, 29, 32].

---

## 4. Kriteria Penyelesaian Sprint 1 (Definition of Done)

Sprint 1 dinyatakan tuntas dan siap dipresentasikan pada evaluasi Milestone 1 jika memenuhi parameter berikut[cite: 8, 14]:
1. **Laporan Benchmark Model ML:** Tersedia tabel perbandingan performa antar-algoritma pada `ml-model-testing` dengan skor $R^2 \ge 0,80$ pada model terpilih[cite: 27, 40].
2. **Artefak Model Tersimpan:** Model terbaik berhasil diekspor ke dalam format biner `.pkl` menggunakan pustaka `joblib`[cite: 27, 33].
3. **Aplikasi Mobile Terpasang:** Aplikasi Flutter dapat dijalankan pada perangkat fisik Android atau emulator, dengan tampilan Beranda dan Form Input yang telah terpasang rapi sesuai desain UI[cite: 14, 27, 31].
4. **Validasi Form Berfungsi:** Form input menolak nilai tidak sah sesuai rancangan pengujian nilai batas BVA/EP[cite: 27, 31].