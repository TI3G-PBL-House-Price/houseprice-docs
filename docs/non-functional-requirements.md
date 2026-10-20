# Kebutuhan Non-Fungsional (Non-Functional Requirements) - HousePrice

Dokumen ini menetapkan batasan kualitas, kriteria performa, keandalan, keamanan, serta kompatibilitas operasional yang wajib dipenuhi oleh sistem HousePrice (aplikasi mobile Flutter, layanan RESTful API Flask, Web Admin, dan basis data PostgreSQL).

---

## 1. Kinerja & Waktu Respons (Performance & Latency)

* **Latensi Inferensi API (`POST /api/v1/predict`):** Waktu total pemrosesan permintaan dari penerimaan payload JSON, validasi skema masukan, inferensi model machine learning `.pkl`, pencatatan log pada basis data, hingga pengiriman respons balik wajib berada di bawah **1.500 ms (1,5 detik)** pada kondisi koneksi jaringan 4G/WiFi stabil[cite: 27, 40].
* **Waktu Muat Statistik Pasar (`GET /api/v1/market-statistics`):** Waktu respons penyajian data agregasi harga wilayah ke halaman beranda mobile maksimal **800 ms** menggunakan optimasi kueri terindeks pada tabel `market_statistics`[cite: 1, 24].
* **Kelancaran Antarmuka Pengguna Mobile (UI Frame Rate):** Aplikasi mobile wajib mempertahankan kecepatan rendering pada **60 frame per detik (fps)** tanpa *jank* atau *freeze* saat melakukan transisi navigasi antar-layar maupun *scrolling* riwayat[cite: 27, 29].
* **Indikator Proses Asinkron:** Setiap proses yang membutuhkan waktu komputasi jaringan di atas 300 ms wajib menampilkan indikator visual (*loading indicator* atau animasi *shimmer*) untuk memberikan umpan balik langsung kepada pengguna[cite: 25, 27].

---

## 2. Kualitas Pemodelan Machine Learning (Model Accuracy & Quality)

* **Koefisien Determinasi ($R^2$ Score):** Model regresi utama yang diterapkan pada lingkungan produksi (*inference engine*) wajib mencapai skor **$R^2 \ge 0,80$** terhadap data uji validasi (*test-set evaluation*)[cite: 27, 40].
* **Batas Kesalahan Prediksi (MAE & RMSE):** Nilai Mean Absolute Error (MAE) dan Root Mean Squared Error (RMSE) harus berada pada rentang batas deviasi yang wajar terhadap sebaran harga properti Jakarta setelah eliminasi data pencilan (*outliers*)[cite: 27, 29].
* **Serialisasi & Ukuran Artefak:** Model machine learning diserialisasi menggunakan pustaka `joblib` dengan ukuran berkas `.pkl` tidak melebihi **150 MB** agar proses pemuatan ke memori RAM server berlangsung secara instan[cite: 27, 33].

---

## 3. Keandalan & Ketersediaan Sistem (Reliability & Availability)

* **Tingkat Ketersediaan Layanan (Service Uptime):** Layanan inferensi backend cloud menargetkan ketersediaan sebesar **99%** selama masa uji coba operasional dan evaluasi berkala oleh dosen pembimbing[cite: 40].
* **Penanganan Kegagalan Jaringan (Graceful Degradation):** Jika sambungan internet terputus atau server backend mengalami kendala, aplikasi mobile tidak boleh terhenti mendadak (*crash*). Antarmuka wajib menampilkan dialog penanganan galat ramah (*friendly error notification*)[cite: 27, 40].
* **Operasional Fitur Mandiri (Offline Capability):** Fitur Kalkulator Simulasi KPR dan Riwayat Prediksi Lokal wajib dapat beroperasi penuh secara *offline* tanpa memerlukan sambungan jaringan internet aktif[cite: 25, 40].

---

## 4. Keamanan Sistem & Perlindungan Data (Security & Data Protection)

* **Kerahasiaan Komunikasi Jaringan:** Seluruh saluran komunikasi data antara klien mobile dan server backend wajib menggunakan enkripsi **HTTPS (TLS 1.3)** yang diterbitkan secara otomatis oleh sertifikat Let's Encrypt melalui reverse proxy Coolify[cite: 41].
* **Keamanan Kredensial Administrator:** Kata sandi akun administrator pada tabel `admins` wajib dienkripsi searah menggunakan algoritma **Bcrypt** dengan *salt work factor* minimal 12 putaran (`$2b$12$...`)[cite: 1, 40].
* **Pencegahan Injeksi Kode (SQL Injection Prevention):** Seluruh operasi manipulasi dan penarikan data ke PostgreSQL wajib menggunakan kueri berparameter (*parameterized queries*) melalui ORM SQLAlchemy atau abstraksi database yang aman[cite: 1, 40].
* **Sanitasi Masukan API:** Seluruh parameter input numerik dan teks pada endpoint inferensi wajib divalidasi skemanya sebelum dieksekusi oleh model, menolak format yang salah dengan respons status `400 Bad Request`[cite: 27, 40].
* **Prinsip Minimasi Data Pribadi (*Privacy by Design*):** Pengguna aplikasi mobile tidak memerlukan autentikasi login atau pengisian identitas pribadi (seperti nama lengkap, surel, nomor telepon), menjaga privasi penuh bagi masyarakat umum[cite: 20, 24].

---

## 5. Kompatibilitas & Portabilitas Lingkungan (Compatibility & Portability)

* **Kompatibilitas Sistem Operasi Klien Mobile:** Aplikasi Android mendukung batas minimum versi **Android API Level 29 (Android 10)** hingga target versi **Android API Level 34+ (Android 14)**[cite: 40].
* **Responsivitas Perangkat Klien:** Antarmuka mobile dirancang adaptif untuk mendukung layar ponsel dengan resolusi standar (mulai dari aspek rasio 16:9, 18:9, hingga 20:9) dengan kepadatan piksel yang bervariasi (HD, Full HD+) tanpa pemotongan elemen teks (*text overflow*)[cite: 27, 29, 32].
* **Portabilitas Lingkungan Server (Containerization):** Seluruh konfigurasi runtime backend (Python 3.10+, dependensi pustaka ML, dan Gunicorn) serta PostgreSQL dikemas ke dalam berkas `Dockerfile` dan `docker-compose.yml` agar dapat dijalankan secara konsisten pada lingkungan lokal, server pengujian, maupun VPS hosting[cite: 41].

---

## 6. Kemudahan Pemeliharaan & Pengujian (Maintainability & Testability)

* **Keterpisahan Lapisan Arsitektur (Layered Separation):** Kode aplikasi mobile terpisah secara ketat menjadi tiga lapisan: *Presentation Layer*, *Business Logic Layer*, dan *Data Layer* untuk memastikan kemudahan pelacakan cacat kode[cite: 25, 26, 40].
* **Modularitas Backend:** Backend Flask dirancang menggunakan pola modular *Flask Blueprints* untuk memisahkan logika rute API publik dengan rute dashboard administrasi[cite: 40].
* **Ketertelusuran Cacat Mutu (Defect Traceability):** Setiap galat atau *bug* yang ditemukan selama proses pengujian PMPL wajib tercatat dengan kode identitas unik pada *Defect Tracking Log* dan diverifikasi penutupannya sebelum penyusunan *Software Test Report* (STR)[cite: 28, 29, 32].