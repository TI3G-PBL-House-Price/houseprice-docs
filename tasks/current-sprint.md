# Rencana Kerja Sprint Aktif (Current Sprint) - HousePrice

Dokumen ini memuat fokus kerja, papan aktivitas kanban, target capaian, serta pembagian tugas harian seluruh anggota tim untuk siklus **Sprint 1 (Minggu 5 s/d 8)** menuju demonstrasi **Milestone 1: Progres Tengah Semester** pasca penyesuaian kebutuhan *Strict Gate Authentication* dan penambahan modul antarmuka baru[cite: 8, 9].

---

## 1. Status & Realitas Repositori Terkini (Baseline Kondisi Riil)

* **Repositori `ml-model-testing` (Fokus Utama ML Saat Ini):**
  * Status: **Aktif (Fase Eksperimen)**[cite: 9].
  * Progres: Pengujian eksplorasi algoritma sedang berjalan[cite: 9]. Model *Random Forest Regressor* telah diuji coba[cite: 9]. Algoritma pembanding lain (seperti *Multiple Linear Regression* sebagai baseline dan *Gradient Boosting / XGBoost*) sedang disiapkan untuk diuji pada dataset `jakarta_house.csv` sebelum menetapkan model final[cite: 9].
* **Repositori `houseprice-backend-ml`:**
  * Status: **Belum Diinisialisasi (Kosong)**[cite: 9].
  * Rencana: Kode backend Flask, struktur Blueprints (`/auth`, `/predict`, `/notifications`, `/admin`), skema 9 tabel PostgreSQL, dan middleware otentikasi JWT akan dibangun setelah pemilihan model definitif[cite: 2, 3, 9].
* **Repositori `houseprice-mobile-app`:**
  * Status: **Selesai Slicing Fondasi & Re-Slicing Modul Baru**.
  * Progres: Slicing antarmuka awal (Beranda pasar, Form Spesifikasi Properti, Hasil Estimasi, dan Kalkulator KPR) telah selesai diimplementasikan[cite: 8, 9]. Namun, akibat penambahan requirement kritis dari dosen di pertengahan Sprint 1 (*Strict Gate Authentication*, Splash, Onboarding, Landing Page, Notifikasi, dan Profil), dibuka tugas *slicing* antarmuka lanjutan yang ditugaskan khusus kepada **@nurilhuda**[cite: 2, 8].

---

## 2. Target Capaian Sprint 1 (Sprint Goals)

Sprint 1 difokuskan untuk menyelesaikan fondasi pada ketiga repositori serta menuntaskan seluruh antarmuka pengguna sebelum masuk ke tahap integrasi API penuh[cite: 8, 9]:
1. **Machine Learning:** Menuntaskan tabel komparasi metrik uji ($R^2$, MAE, RMSE) antar-algoritma pada repositori `ml-model-testing`, memilih satu algoritma dengan performa terbaik ($R^2 \ge 0,80$), serta mengekspor artefak model final `.pkl`[cite: 9].
2. **Mobile (Flutter Slicing & Flow):**
   * Menyelesaikan *slicing* modul pre-otentikasi: Splash Screen, 3 Slide Onboarding Carousel, dan Landing Page[cite: 8].
   * Menyelesaikan *slicing* formulir otentikasi: Login dan Registrasi akun (Strict Gate) lengkap dengan validasi visual lokal[cite: 7, 8].
   * Menyelesaikan *slicing* modul pelengkap: Layar Notifikasi In-App dan Layar Profil Pengguna (termasuk kartu riwayat prediksi)[cite: 2, 8].
   * Memperbarui scaffold navigasi utama menjadi 5 Tab Bottom Navigation Bar (Beranda, Estimasi, KPR, Notifikasi, Profil)[cite: 8].
3. **Backend Skeleton:** Menginisialisasi repositori `houseprice-backend-ml`, menyiapkan struktur dasar Flask, migrasi skema tabel PostgreSQL (`users`, `notifications`, `prediction_logs`), dan endpoint dasar[cite: 2, 9].
4. **PMPL:** Menyusun matriks pengujian *Boundary Value Analysis* (BVA), *Equivalence Partitioning* (EP) untuk form auth dan properti, serta skenario verifikasi *Navigation Guard*[cite: 7, 9].

---

## 3. Papan Tugas Kanban Sprint 1 (Sprint 1 Kanban Board)

```
+---------------------------------------------------------------------------------------------------------+
| BACKLOG / TO DO                     | IN PROGRESS                             | DONE                    |
+---------------------------------------------------------------------------------------------------------+
| [MOB] Slicing Splash & Onboard      | [MOB] Slicing UI Landing Page           | [DOC] Proposal Baseline |
|       (@nurilhuda)                  |       (@nurilhuda)                      | [DOC] Revisi PRD & Specs|
| [MOB] Slicing Form Login & Register | [ML]  Eksperimen Komparasi Model        | [DES] UI Design Mockup  |
|       (@nurilhuda)                  |       (Random Forest vs XGBoost)        | [MOB] Init Flutter Repo |
| [MOB] Slicing Notifikasi & Profil   | [QA]  Penyusunan Test Case Auth & BVA   | [MOB] Slicing Beranda   |
|       (@nurilhuda)                  | [BE]  Inisialisasi Blueprint Auth & DB  | [MOB] Slicing Form ML   |
| [BE]  Setup Docker & DB Schema      |                                         | [MOB] Slicing KPR Calc  |
| [ML]  Export Model Final .pkl       |                                         |                         |
+---------------------------------------------------------------------------------------------------------+
```

### Rincian Pembagian Tugas Anggota Tim:

#### A. Trek Mobile App (Repo: `houseprice-mobile-app`)
* **Muhammad Nuril Huda (@nurilhuda):**
  * **[TASK-MOB-01] Slicing UI Splash Screen & Onboarding Carousel:**
    * Mengimplementasikan Splash Screen dengan animasi loading dan inisialisasi state[cite: 8].
    * Membangun 3 kartu *carousel slider* proposisi fitur (Prediksi ML, Cek Kewajaran, Kalkulator KPR) lengkap dengan *dot indicators* dan tombol aksi Lewati/Mulai[cite: 8].
  * **[TASK-MOB-02] Slicing UI Landing Page:**
    * Mengimplementasikan halaman pembuka sebelum otentikasi dengan tombol aksi Masuk (`GradientButton`) dan Daftar Akun (`OutlinedButton`)[cite: 8].
  * **[TASK-MOB-03] Slicing UI Formulir Login & Register:**
    * Membangun form login dan register dengan styling abu-abu rounded (`#EFF2F7`), prefix icon, icon toggle penyamaran kata sandi (*obscure text*), serta label penanganan pesan galat[cite: 8].
  * **[TASK-MOB-04] Slicing UI Pusat Notifikasi In-App:**
    * Membangun tampilan daftar pesan notifikasi tren harga pasar, status belum dibaca (*soft blue background*), dan badge indikator merah[cite: 8].
  * **[TASK-MOB-05] Slicing UI Layar Profil & Riwayat Cloud:**
    * Membangun kartu informasi pengguna terdaftar, kartu list riwayat estimasi personal, serta tombol Logout dengan dialog konfirmasi[cite: 7, 8].
  * **[TASK-MOB-06] Konfigurasi 5 Tab Bottom Navigation:**
    * Memperbarui komponen `main_navigation.dart` agar mengintegrasikan 5 tab: Beranda, Prediksi, KPR, Notifikasi, dan Profil[cite: 8].
* **Jasmine Nasywa Nabilah:**
  * Mengonfigurasi aset gambar vektor dan palet warna baru pada `app_colors.dart` untuk modul autentikasi dan notifikasi[cite: 8, 9].
  * Melakukan review tampilan UI untuk memastikan tidak ada *RenderFlex overflow* saat keyboard virtual terbuka pada layar auth dan form properti[cite: 7].
* **Abdul Rahman Hanif Darmawan:**
  * Memastikan logika validasi lokal form spesifikasi properti terhubung dengan benar ke alur navigasi hasil estimasi[cite: 8, 9].

#### B. Trek Sains Data & Machine Learning (Repo: `ml-model-testing`)
* **Muhammad Nuril Huda:**
  * Menyelesaikan pengujian algoritma komparasi (Random Forest vs Linear Regression vs Gradient Boosting/XGBoost)[cite: 9].
  * Mencatat perbandingan metrik evaluasi ($R^2$ Score, MAE, RMSE) untuk menentukan artefak model terbaik[cite: 9].
* **Abdul Rahman Hanif Darmawan:**
  * Mendokumentasikan hasil perbandingan performa model dan distribusi galat prediksi pada notebook EDA[cite: 9].
* **Naufal Abid Aurizky:**
  * Mengonfigurasi pipeline praproses fitur: pemfilteran *outlier* IQR dan skema encoding area wilayah (`master_locations`)[cite: 1, 9].
* **Nurfinka Lailasari:**
  * Menuntaskan evaluasi penanganan missing values pada atribut luas tanah dan luas bangunan[cite: 9].
* **Jasmine Nasywa Nabilah:**
  * Memverifikasi eliminasi baris duplikat pada dataset properti latih[cite: 9].

#### C. Trek Backend & Basis Data (Repo: `houseprice-backend-ml`)
* **Abdul Rahman Hanif Darmawan & Muhammad Nuril Huda:**
  * Melakukan inisialisasi struktur repositori backend Flask (Blueprint `/api/v1/auth`, `/api/v1/predict`, `/api/v1/notifications`)[cite: 2, 3, 9].
  * Menyiapkan skrip DDL PostgreSQL untuk membentuk 9 tabel (termasuk tabel baru `users` dan `notifications`)[cite: 1, 2].
  * Menyiapkan konfigurasi `Dockerfile` dan `docker-compose.yml` untuk deployment di VPS Coolify[cite: 3, 9].

#### D. Trek Penjaminan Mutu Perangkat Lunak (PMPL)
* **Nurfinka Lailasari:**
  * Menyiapkan matriks pelacakan cacat (*Defect Tracking Log*) untuk mendokumentasikan kendala pengujian antarmuka auth, form, dan latensi model[cite: 7, 9].
* **Jasmine Nasywa Nabilah:**
  * Menyusun tabel skenario uji Blackbox (BVA dan EP) untuk formulir registrasi, formulir login, dan validasi form properti[cite: 7, 9].
* **Abdul Rahman Hanif Darmawan:**
  * Menyiapkan skrip uji coba Postman Collection Runner untuk pengujian rute endpoint auth dan inferensi terproteksi Bearer Token[cite: 2, 7].

---

## 4. Kriteria Penyelesaian Sprint 1 (Definition of Done)

Sprint 1 dinyatakan tuntas dan siap dipresentasikan pada evaluasi Milestone 1 jika memenuhi parameter berikut[cite: 9]:
1. **Model ML Terpilih & Terekspor:** Model regresi terbaik telah ditentukan dengan metrik $R^2 \ge 0,80$ dan berhasil diekspor menjadi berkas biner `.pkl` menggunakan `joblib`[cite: 9].
2. **Kelengkapan Slicing Antarmuka Mobile:** Aplikasi Flutter berjalan lancar pada perangkat Android dengan alur navigasi lengkap:
   * `Splash Screen` $\rightarrow$ `Onboarding Carousel` $\rightarrow$ `Landing Page` $\rightarrow$ `Login / Register` $\rightarrow$ `5 Tab Dashboard` (Beranda, Prediksi ML, KPR, Notifikasi, Profil)[cite: 7, 8].
3. **Validasi Formulir Berfungsi:** Seluruh form (pendaftaran, login, dan spesifikasi properti) menolak masukan kosong atau tidak valid secara visual di sisi klien sesuai matriks BVA/EP[cite: 7, 9].
4. **Struktur Backend & DB Siap:** Repositori Flask telah terinisialisasi dan skema 9 tabel PostgreSQL berhasil dieksekusi melalui skrip migrasi DDL[cite: 1, 2].