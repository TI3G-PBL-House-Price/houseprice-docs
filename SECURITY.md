# Kebijakan Keamanan Sistem (Security Policy) - HousePrice

Dokumen ini mendefinisikan standar keamanan informasi, manajemen kredensial, perlindungan identitas pengguna (*Strict Gate Authentication*), isolasi data transaksi, serta prosedur mitigasi kerentanan pada ekosistem proyek HousePrice (Mobile App Flutter, Backend REST API Flask, Web Admin, dan Basis Data PostgreSQL).

---

## 1. Kebijakan Pelaporan Kerentanan (Vulnerability Reporting)

Proyek ini dikembangkan dalam lingkup akademis terintegrasi (*Product-Based Learning*). Jika Anda menemukan celah keamanan (*vulnerability*), potensi kebocoran token, atau eksploitasi data:

1. **Dilarang Membuka Isu Publik:** Jangan melaporkan kerentanan melalui *Public GitHub Issue*.
2. **Kanal Pelaporan Privat:** Laporkan temuan secara langsung kepada *Project Manager & QA Integration* (Abdul Rahman Hanif Darmawan) atau tim pengembang melalui kanal komunikasi privat.
3. **Format Laporan:** Sertakan langkah reproduksi kerentanan (*Proof of Concept*), tingkat keparahan (*impact assessment*), serta komponen modul yang terdampak (`houseprice-mobile-app`, `houseprice-backend-ml`, atau infrastruktur cloud).
4. **Waktu Respons:** Tim akan meninjau laporan maksimal dalam 2 x 24 jam dan merilis perbaikan darurat melalui cabang `hotfix/*`.

---

## 2. Manajemen Lingkungan & Kredensial Rahasia (.env & Secrets)

Kebocoran kunci rahasia (*secret keys*) dan kredensial basis data merupakan risiko fatal dalam repositori Git:

### Aturan Berkas Konfigurasi Lingkungan
* Berkas `.env` aktual **dilarang keras** di-*commit* atau diunggah ke repositori GitHub mana pun (`houseprice-backend-ml`, `houseprice-mobile-app`, `ml-model-testing`, `houseprice-docs`).
* Aturan penyaringan berkas sensitif pada `.gitignore`:
  ```gitignore
  .env
  .env.*
  *.pem
  *.key
  instance/
  __pycache__/
  ```
* Setiap repositori hanya menyediakan template contoh publik bernama `.env.example` yang memuat parameter tanpa nilai rahasia aktual:
  ```ini
  # Environment Runtime
  FLASK_ENV=production
  SECRET_KEY=ganti-dengan-kunci-acak-minimal-32-karakter
  JWT_SECRET_KEY=ganti-dengan-kunci-jwt-hs256-panjang-dan-acak
  JWT_ACCESS_TOKEN_EXPIRES_DAYS=7

  # Database PostgreSQL Configuration
  DB_HOST=postgres
  DB_PORT=5432
  DB_NAME=houseprice_db
  DB_USER=houseprice_admin
  DB_PASSWORD=masukkan_password_aman_disini

  # ML Inference Configuration
  MODEL_PATH=models/rf_house_price_v1.pkl
  ALLOWED_ORIGINS=[https://admin.houseprice.id](https://admin.houseprice.id),[https://api.houseprice.id](https://api.houseprice.id)
  ```

---

## 3. Keamanan Otentikasi & Basis Data (Admin & Mobile Users)

Sistem mengelola dua entitas pengguna terpisah (`admins` dan `users`) dengan standar keamanan yang ketat:

1. **Enkripsi Kredensial (Bcrypt Hashing):**
   * Seluruh kata sandi (baik administrator web maupun pengguna mobile) tidak pernah disimpan dalam bentuk teks polos (*plaintext*).
   * Kata sandi wajib di-hash menggunakan algoritma **Bcrypt** dengan faktor kerja (*work factor/salt rounds*) minimal **12 putaran** (`$2b$12$...`).
2. **Otentikasi Berbasis Token (JWT - HS256):**
   * Sesi login pengguna mobile dikelola menggunakan JSON Web Token (JWT) dengan penandatanganan algoritma HS256.
   * Payload token hanya memuat informasi esensial non-sensitif (`user_id`, `email`, dan waktu kedaluwarsa `exp`). Dilarang keras menyematkan hash password atau data sensitif lain ke dalam payload JWT.
3. **Pencegahan Serangan SQL Injection:**
   * Seluruh operasi kueri basis data menuju PostgreSQL wajib menggunakan *Parameterized Queries* atau Object-Relational Mapping (SQLAlchemy) yang terisolasi.
   * Konkatenasi string mentah (*raw string formatting*) pada query SQL dilarang tanpa pengecualian.
4. **Pencatatan Audit Administratif (*Audit Logging*):**
   * Setiap aktivitas kritis di Web Admin (unggah dataset CSV baru, eksekusi retraining model, aktivasi versi model baru) dicatat secara otomatis ke tabel `system_audit_logs`.
   * Atribut pencatatan mencakup: `admin_id`, `action_type`, `module_name`, `description`, `ip_address`, dan `user_agent` untuk menjamin prinsip *non-repudiation*.

---

## 4. Keamanan Layanan RESTful API & Komputasi Cloud

Layanan API backend mengoperasikan endpoint publik dan endpoint bisnis privat yang dilindungi:

1. **Enkripsi Saluran Komunikasi (Wajib HTTPS):**
   * Seluruh komunikasi antara aplikasi klien mobile Flutter dan server cloud wajib melalui protokol HTTPS terenkripsi TLS 1.3 via reverse proxy Coolify (Let's Encrypt).
   * Aplikasi Flutter secara tegas menolak lalu lintas teks polos (*cleartext HTTP traffic* dinonaktifkan pada `AndroidManifest.xml`).
2. **Proteksi Endpoint Privat (Strict Gate Middleware):**
   * Endpoint inferensi (`POST /api/v1/predict`), riwayat pengguna, notifikasi, dan profil dilindungi oleh middleware decorator otentikasi.
   * Permintaan tanpa header `Authorization: Bearer <token>` atau dengan token kedaluwarsa/palsu langsung ditolak dengan kode status `401 Unauthorized`.
3. **Validasi & Sanitasi Skema Masukan (Input Sanitization):**
   * Seluruh masukan JSON divalidasi ketat pada *gateway layer* menggunakan library skema (Marshmallow/Pydantic) sebelum dieksekusi oleh pipeline Machine Learning.
   * Nilai di luar batas logika (misal: luas tanah $\le 0$, kamar bernilai negatif, atau karakter non-numerik) langsung ditolak dengan kode status `400 Bad Request`.
4. **Mitigasi Serangan Brute Force (Rate Limiting):**
   * Endpoint pendaftaran (`/api/v1/auth/register`) dan masuk akun (`/api/v1/auth/login`) menerapkan pembatasan frekuensi request (*rate limiting*) berbasis IP untuk memitigasi serangan *credential stuffing* dan *brute force*.
5. **Kebijakan Akses Lintas Domain (CORS Policy):**
   * Jalur API mobile dikonfigurasi menerima request dari klien aplikasi terdaftar.
   * Antarmuka web administratif dibatasi secara ketat hanya dapat diakses oleh domain dashboard resmi melalui konfigurasi `ALLOWED_ORIGINS`.

---

## 5. Keamanan Klien Mobile & Privasi Data Pengguna (*Privacy & Data Isolation*)

Penerapan *Strict Gate Authentication* diimbangi dengan standar perlindungan privasi data pribadi (*Personally Identifiable Information / PII*):

1. **Penyimpanan Token Aman (*Secure Storage*):**
   * Token otentikasi JWT pada perangkat Android **WAJIB** disimpan menggunakan pustaka `flutter_secure_storage`.
   * Pustaka ini memanfaatkan mekanisme perangkat keras **Android Keystore** untuk mengenkripsi data token dengan algoritma AES-GCM.
   * Dilarang keras menyimpan token otentikasi atau kredensial pengguna pada `SharedPreferences` biasa yang tidak terenkripsi.
2. **Pembersihan Sesi (*Safe Logout*):**
   * Saat pengguna menekan tombol *Logout*, seluruh token JWT yang tersimpan di *Secure Storage* dihapus tuntas dari memori lokal dan router navigasi mengembalikan status tampilan ke `LandingScreen`.
3. **Isolasi Data Riwayat Prediksi (*Multi-Tenant Isolation*):**
   * Setiap catatan transaksi pada tabel `prediction_logs` terikat secara relasional pada atribut `user_id`.
   * Endpoint riwayat (`GET /api/v1/predict/history`) memvalidasi kepemilikan data melalui token aktif; pengguna tidak dapat melihat atau mengakses riwayat prediksi milik akun lain (*authorization enforcement*).
4. **Perlindungan Data Kontak & Notifikasi:**
   * Nomor kontak dan token perangkat FCM yang tersimpan di tabel `users` hanya digunakan untuk kebutuhan fungsional aplikasi dan dilindungi dari eksposur endpoint publik.