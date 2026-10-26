# Product Requirements Document (PRD) - HousePrice

- **Nama Produk:** HousePrice (Prediksi Harga Rumah DKI Jakarta)
- **Kategori:** Mobile Intelligent System (Integrated Machine Learning)
- **Target Platform:** Android OS 10.0+ (Klien Mobile) & Web Browser (Web Admin)
- **Status Dokumen:** Revised Baseline (Sprint 1 - Strict Gate Architecture)

---

## 1. Latar Belakang & Masalah Bisnis
Penetapan dan evaluasi harga rumah di wilayah DKI Jakarta memiliki tingkat volatilitas yang tinggi dipengaruhi oleh lokasi administratif, luas tanah/bangunan, dan fasilitas properti. Pembeli awam dan keluarga muda sering menghadapi risiko *overpricing* (membeli di atas harga pasar wajar), sementara penjual kesulitan menentukan nilai pasar yang kompetitif. 

HousePrice menjawab masalah ini dengan menyediakan taksiran harga berbasis bukti (*data-driven*) menggunakan machine learning yang dapat diakses secara transparan melalui gawai mobile. Untuk menjamin privasi data pencarian, personalisasi riwayat simulasi KPR, serta penyaluran notifikasi tren properti, sistem menerapkan model akses terotentikasi penuh (*Strict Gate Authentication*).

---

## 2. Profil Pengguna (Personas)
1. **Pengguna Terdaftar Mobile (Registered Mobile User):** Calon pembeli atau pemilik rumah di DKI Jakarta yang telah membuat akun untuk melakukan estimasi harga properti, mengevaluasi kewajaran penawaran penjual, menghitung simulasi angsuran KPR, memantau riwayat kalkulasi personal, serta menerima notifikasi perkembangan pasar.
2. **Administrator Teknis (Admin):** Tim pengelola sistem yang bertanggung jawab memantau metrik performa model inferensi ($R^2$, MAE, RMSE), mengelola berkas dataset pasar, menjalankan pelatihan ulang model secara berkala, dan memeriksa jejak audit operasional melalui Web Dashboard.

---

## 3. Ruang Lingkup Sistem

### Yang Dikerjakan (In-Scope):
- **Alur Antarmuka Mobile (Flutter):**
  - Alur pre-otentikasi: Splash Screen dinamis, Onboarding *carousel* edukatif, dan Landing Page sambutan.
  - Alur otentikasi *Strict Gate*: Registrasi akun baru, Login kredensial, proteksi sesi via JWT, dan penyimpanan token aman (*Secure Storage*).
  - Alur fungsional inti: Beranda tren pasar properti, Form input prediksi harga ML, Analisis kelayakan penawaran pasar, dan Kalkulator simulasi KPR (metode anuitas).
  - Modul personal: Pusat pesan notifikasi tren properti serta Profil pengguna yang memuat riwayat komputasi terpusat di cloud.
- **Backend & Cloud API (Flask & PostgreSQL):**
  - Modul otentikasi JWT (hashing kata sandi *bcrypt* minimal 12 salt rounds).
  - Endpoint inferensi regresi properti terproteksi *Bearer Token*.
  - Penyedia agregasi statistik harga pasar properti per wilayah DKI Jakarta.
  - Layanan penyaluran notifikasi in-app bagi pengguna.
- **Machine Learning Core:**
  - Model regresi berbasis ensemble (*Random Forest Regressor*) dengan target performa $R^2 \ge 0.80$.
- **Web Dashboard Admin:**
  - Monitoring metrik performa model aktif, manajemen dataset properti (`.csv`), pemicu *retraining* model, dan log audit sistem.
- **Skema Basis Data PostgreSQL (9 Entitas):**
  - `admins`, `users`, `master_locations`, `dataset_metadata`, `ml_models`, `market_statistics`, `notifications`, `system_audit_logs`, serta relasi kepemilikan data pada `prediction_logs`.

### Yang Tidak Dikerjakan (Out-of-Scope):
- Akses fitur kalkulasi prediksi tanpa akun (*guest / anonymous access* ditiadakan).
- Visualisasi spasial interaktif berupa GIS *heatmap* berbasis koordinat lintang/bujur.
- Fitur transaksi jual-beli properti langsung atau listing iklan mandiri (bukan *marketplace* properti).
- Integrasi persetujuan pembiayaan KPR langsung ke core-banking lembaga perbankan.
- Pengembangan native untuk platform iOS (prioritas difokuskan penuh pada platform Android).

---

## 4. Spesifikasi Fitur Fungsional

### A. Aplikasi Mobile (Flutter Client)
- **FR-MOB-01 (Splash & Session Verification):** Memeriksa status pertama kali instalasi dan memvalidasi keberadaan JWT token di *Secure Storage* untuk menentukan navigasi berikutnya (Onboarding, Landing Page, atau Beranda).
- **FR-MOB-02 (Onboarding Carousel):** Menyajikan 3 kartu geser interaktif yang memaparkan proposisi nilai aplikasi (Estimasi Harga Akurat, Evaluasi Pasar Realistis, Simulasi KPR Cepat) dilengkapi opsi lewati (*Skip*) dan tombol aksi selesai.
- **FR-MOB-03 (Landing Page):** Halaman pembuka sebelum otentikasi yang menyediakan opsi navigasi terarah menuju form pendaftaran akun atau form masuk akun.
- **FR-MOB-04 (Autentikasi Akun - Strict Gate):**
  - **Pendaftaran (Register):** Validasi format email, kelengkapan nama, dan konfirmasi kata sandi minimal 8 karakter.
  - **Masuk Akun (Login):** Autentikasi email dan kata sandi, penerimaan token JWT, serta penyimpanan sesi terenkripsi.
- **FR-MOB-05 (Beranda Tren Pasar):** Menyajikan statistik agregasi rata-rata harga per meter persegi ($m^2$) berdasarkan penyaringan kota administrasi di DKI Jakarta.
- **FR-MOB-06 (Form Prediksi ML):** Formulir masukan parameter fisik properti: Kota (Dropdown), Kecamatan (Dropdown dinamis dari master data), Luas Tanah ($m^2$), Luas Bangunan ($m^2$), Jumlah Kamar Tidur, Kamar Mandi, dan Carport. Validasi lokal mencegah masukan kosong, nol, atau bernilai negatif.
- **FR-MOB-07 (Hasil Prediksi & Estimasi Nilai Pasar):** Menampilkan nilai taksiran pasar properti dalam nominal Rupiah beserta rentang batas deviasi wajar (*Lower* dan *Upper Bound*).
- **FR-MOB-08 (Cek Kewajaran Harga):** Mengomparasikan nominal penawaran aktual penjual dengan estimasi model ML, menampilkan evaluasi status:
  - *Murah / Dibawah Pasar:* Selisih penawaran < -10% terhadap prediksi.
  - *Wajar / Sesuai Pasar:* Selisih penawaran berada di antara rentang -10% s/d +10%.
  - *Mahal / Diatas Pasar:* Selisih penawaran > +10% terhadap prediksi.
- **FR-MOB-09 (Kalkulator Simulasi KPR):** Menghitung proyeksi angsuran pinjaman bulanan berbasis metode anuitas dengan parameter: harga properti, persentase uang muka (DP), suku bunga tetap tahunan, dan tenor cicilan (tahun).
- **FR-MOB-10 (Pusat Notifikasi):** Menampilkan daftar pemberitahuan tren harga pasar terbaru dan rilis sistem, disertai pembaruan status baca saat pesan dibuka.
- **FR-MOB-11 (Profil Pengguna & Riwayat Cloud):** Menampilkan identitas akun yang sedang aktif, daftar riwayat pencarian prediksi yang terikat pada `user_id` di server, fasilitas penggantian kata sandi, dan fungsi *Logout* yang menghapus token lokal.

### B. Web Dashboard Admin (Flask Web Interface)
- **FR-WEB-01 (Autentikasi Admin):** Login sesi aman khusus akun administrator yang terverifikasi pada tabel `admins`.
- **FR-WEB-02 (Dashboard Monitoring):** Menampilkan metrik model aktif ($R^2$, MAE, RMSE), total volume dataset pelatihan, serta grafik frekuensi inferensi dari klien mobile.
- **FR-WEB-03 (Manajemen Dataset):** Unggah berkas `jakarta_house.csv`, pratinjau struktur data tabular, dan penentuan rasio pemisahan data latih/uji (*train-test split*).
- **FR-WEB-04 (Retraining & Switch Model):** Menjalankan pipeline pelatihan ulang algoritma regresi, menampilkan tabel perbandingan skor model baru vs model aktif, serta konfirmasi penetapan berkas `.pkl` utama.
- **FR-WEB-05 (Audit Aktivitas):** Pencatatan otomatis setiap tindakan administratif (unggah dataset, pergantian model, penghapusan berkas) ke tabel `system_audit_logs`.

### C. Cloud Services & RESTful API
- **FR-API-01 (`POST /api/v1/auth/register`):** Mendaftarkan akun mobile baru, memvalidasi keunikan email, dan menyimpan hash kata sandi *bcrypt* ke tabel `users`.
- **FR-API-02 (`POST /api/v1/auth/login`):** Memvalidasi kredensial pengguna mobile dan menerbitkan access token JWT (HS256) bervaliditas terbatas.
- **FR-API-03 (`GET /api/v1/auth/me`):** Mengembalikan data identitas profil pengguna berdasarkan header otentikasi Bearer Token.
- **FR-API-04 (`POST /api/v1/predict`):** Menerima spesifikasi rumah via JSON, memvalidasi otentikasi Bearer Token JWT, mengekstrak identitas pemohon (`user_id`), mengeksekusi inferensi model `.pkl`, mencatat transaksi ke tabel `prediction_logs`, dan mengembalikan nilai prediksi dalam waktu < 1.5 detik.
- **FR-API-05 (`GET /api/v1/market-statistics`):** Menyediakan ringkasan rata-rata harga dan luas bangunan per wilayah untuk kebutuhan beranda mobile.
- **FR-API-06 (`GET /api/v1/notifications`):** Mengambil pesan notifikasi yang relevan untuk akun pengguna yang sedang terautentikasi.
- **FR-API-07 (`POST /api/v1/retrain`):** Endpoint privat internal admin untuk mengeksekusi pelatihan ulang model secara terprogram.

---

## 5. Kebutuhan Non-Fungsional (NFR)

### A. Kinerja & Keandalan (Performance & Reliability)
- **Latensi Komputasi:** Waktu respons inferensi endpoint `/api/v1/predict` tidak melebihi 1.500 ms (1,5 detik) pada kondisi koneksi internet normal.
- **Ketersediaan Layanan (Uptime):** Backend API menargetkan ketersediaan 99% selama masa pengujian sprint dan demonstrasi evaluasi akhir.

### B. Keamanan & Privasi (Security & Privacy)
- **Proteksi Akses (Strict Gate):** Endpoint bisnis privat menolak akses anonim dan mengembalikan kode status `401 Unauthorized` jika token tidak disertakan atau kedaluwarsa.
- **Enkripsi Kredensial:** Seluruh kata sandi pengguna wajib dienkripsi searah menggunakan *bcrypt* dengan faktor kerja (*cost factor*) minimal 12 *salt rounds*.
- **Penyimpanan Klien:** Token otentikasi JWT pada perangkat mobile wajib disimpan dalam ruang aman OS (*Keystore* pada Android melalui library `flutter_secure_storage`).
- **Isolasi Data (Data Isolation):** Riwayat prediksi properti diproteksi secara relasional di tingkat database; pengguna hanya memiliki izin melihat data komputasi miliknya sendiri.
- **Sanitasi Kredensial Repositori:** Variabel lingkungan dan kunci rahasia (`JWT_SECRET_KEY`, kredensial PostgreSQL) dilarang dimasukkan ke sistem kontrol versi Git.

### C. Kompatibilitas Perangkat (Device Compatibility)
- Aplikasi klien mobile mendukung penuh sistem operasi Android minimal versi 10.0 (API Level 29) hingga versi 14+ (API Level 34).
- Antarmuka web admin kompatibel pada peramban web modern berbasis Chromium (Chrome, Edge) dan Mozilla Firefox.