# Kebijakan Keamanan Sistem (Security Policy) - HousePrice

Dokumen ini mendefinisikan standar keamanan informasi, perlindungan data, manajemen kredensial, serta prosedur penanganan kerentanan pada ekosistem proyek HousePrice (Mobile App, Backend API, Web Admin, dan Basis Data).

---

## 1. Kebijakan Pelaporan Kerentanan (Vulnerability Reporting)

Proyek ini dikembangkan dalam lingkup akademis terintegrasi (*Product-Based Learning*). Jika Anda menemukan celah keamanan (*vulnerability*), kebocoran token, atau potensi eksploitasi data:

1. **Dilarang Membuka Isu Publik:** Jangan membuat laporan kerentanan melalui *Public GitHub Issue*.
2. **Kanal Pelaporan:** Laporkan temuan secara privat langsung kepada *Project Manager* (Abdul Rahman Hanif Darmawan) atau anggota tim QA[cite: 19, 28].
3. **Format Laporan:** Sertakan langkah reproduksi kerentanan (*Proof of Concept*), tingkat keparahan (*impact assessment*), serta komponen modul yang terdampak.
4. **Waktu Respons:** Tim akan meninjau laporan maksimal dalam 2 x 24 jam dan merilis perbaikan darurat melalui cabang `hotfix/*`[cite: 37].

---

## 2. Manajemen Lingkungan & Kredensial Rahasia (.env & Secrets)

Kebocoran kredensial merupakan risiko fatal dalam repositori Git publik maupun organisasi:

### Aturan Berkas Konfigurasi Lingkungan
* Berkas `.env` asli **dilarang keras** di-*commit* atau diunggah ke repositori GitHub mana pun (`houseprice-backend-ml`, `houseprice-mobile-app`, `ml-model-testing`).
* Berkas `.gitignore` pada repositori backend wajib menyaring berkas sensitif:
  ```gitignore
  .env
  .env.*
  *.pem
  *.key
  instance/
  __pycache__/
  ```
* Setiap repositori hanya menyediakan template contoh publik bernama `.env.example` yang berisi daftar variabel tanpa memuat nilai rahasia aktual:
  ```ini
  # Environment Runtime
  FLASK_ENV=production
  SECRET_KEY=ganti-dengan-kunci-rahasia-acak-minimal-32-karakter
  
  # Database PostgreSQL Configuration
  DB_HOST=postgres
  DB_PORT=5432
  DB_NAME=houseprice_db
  DB_USER=houseprice_admin
  DB_PASSWORD=masukkan_password_aman_disini
  
  # ML Inference Configuration
  MODEL_PATH=models/rf_house_price_v1.pkl
  ALLOWED_ORIGINS=[https://admin.houseprice.id](https://admin.houseprice.id)
  ```

---

## 3. Keamanan Basis Data & Autentikasi Web Admin

Pengelolaan data master dan artefak model machine learning diatur melalui Web Admin dengan mekanisme pengamanan berlapis:

1. **Hashing Kata Sandi Administrator:**
   * Kata sandi akun administrator pada tabel `admins` tidak pernah disimpan dalam bentuk teks polos (*plaintext*)[cite: 1].
   * Seluruh kata sandi wajib di-hash menggunakan algoritma **Bcrypt** dengan *salt work factor* minimal 12 putaran (`$2b$12$...`)[cite: 1].
2. **Perlindungan Terhadap Serangan SQL Injection:**
   * Seluruh kueri basis data menuju PostgreSQL wajib menggunakan *Parameterized Queries* atau Object Relational Mapping (SQLAlchemy) yang terisolasi.
   * Dilarang menggunakan konkatenasi string mentah (*raw string formatting*) saat menyusun perintah SQL.
3. **Pencatatan Audit Sistem (*Audit Logging*):**
   * Setiap aktivitas administratif kritis (unggah dataset CSV baru, eksekusi retraining model, perubahan versi model aktif) dicatat secara otomatis ke tabel `system_audit_logs`[cite: 1].
   * Atribut audit mencakup: `admin_id`, `action_type`, `description`, `ip_address`, dan `user_agent` untuk menjamin *non-repudiation*[cite: 1].

---

## 4. Keamanan Layanan RESTful API & Komputasi Cloud

Endpoint backend melayani inferensi publik untuk aplikasi mobile Flutter serta kebutuhan manajemen admin[cite: 20, 23]:

1. **Enkripsi Saluran Komunikasi (Wajib HTTPS):**
   * Seluruh lalu lintas data antara aplikasi mobile Flutter dan server cloud wajib melalui protokol HTTPS terenkripsi TLS 1.3 via reverse proxy Coolify (Let's Encrypt).
   * Aplikasi Flutter secara tegas menolak lalu lintas teks polos (*cleartext traffic* dinonaktifkan pada `AndroidManifest.xml`).
2. **Validasi Skema Masukan (Input Sanitization):**
   * Endpoint `POST /api/v1/predict` menerapkan skema validasi ketat (menggunakan Marshmallow atau Pydantic) sebelum data diteruskan ke model Machine Learning.
   * Masukan bernilai tidak masuk akal (luas area bernilai negatif, karakter khusus, atau teks pada kolom numerik) langsung ditolak dengan kode status `400 Bad Request` tanpa memicu galat internal server (`500 Internal Server Error`).
3. **Kebijakan Akses Lintas Domain (CORS Policy):**
   * Pengendalian *Cross-Origin Resource Sharing* (CORS) diaktifkan secara selektif.
   * Jalur API publik `/api/v1/*` dapat diakses oleh klien mobile, sementara antarmuka administratif web dibatasi hanya untuk domain tepercaya yang terdaftar di konfigurasi `ALLOWED_ORIGINS`.

---

## 5. Privasi & Perlindungan Data Pengguna Mobile (*Privacy by Design*)

Aplikasi mobile dirancang dengan prinsip minimasi data (*data minimization*):

* **Tanpa Pendaftaran Akun Mobile:** Pengguna umum tidak diwajibkan mendaftarkan identitas pribadi (nama, email, atau nomor telepon) untuk menggunakan fitur estimasi harga, cek kewajaran, dan simulasi KPR[cite: 20, 24, 25].
* **Penyimpanan Lokal Mandiri:** Riwayat simulasi harga dan kalkulasi KPR disimpan secara eksklusif pada penyimpanan lokal perangkat pengguna (SQLite / SharedPreferences)[cite: 25, 31].
* **Data Log Server:** Data yang terekam pada tabel `prediction_logs` di basis data PostgreSQL hanya berupa parameter fisik properti, nilai estimasi, versi model, waktu komputasi, dan alamat IP klien untuk analisis telemetri performa sistem[cite: 1].