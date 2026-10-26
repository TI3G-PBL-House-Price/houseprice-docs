# Pemodelan Ancaman Keamanan (Threat Model) - HousePrice

Dokumen ini mendefinisikan analisis ancaman keamanan perangkat lunak menggunakan kerangka kerja **STRIDE** (*Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege*) untuk mengidentifikasi potensi vektor serangan serta menetapkan kontrol mitigasi pada sistem HousePrice pasca penerapan arsitektur *Strict Gate Authentication*[cite: 5].

---

## 1. Diagram Aliran Data & Batas Kepercayaan (Data Flow & Trust Boundaries)

Sistem HousePrice membagi area operasional ke dalam tiga zona batas kepercayaan (*Trust Boundaries*) untuk memisahkan entitas publik luar dari komputasi privat internal[cite: 5]:

```
+-----------------------------------------------------------------------------------+
| ZONA 1: UNTRUSTED ENVIRONMENT (Internet Publik)                                   |
|   [ Flutter Mobile Client ]              [ Web Browser Administrator ]            |
|   - Splash, Onboarding, Landing          - Dashboard Web Admin                    |
|   - Form Auth, ML Form, KPR, Profile                                              |
+-----------------------------------------------------------------------------------+
                               | (HTTPS / TLS 1.3 / Bearer JWT)
                        [ TRUST BOUNDARY A ]
                               v
+-----------------------------------------------------------------------------------+
| ZONA 2: SEMI-TRUSTED DMZ (Reverse Proxy & Ingress)                                |
|   [ Reverse Proxy (Coolify / Traefik) ]                                           |
|   - Terminasi SSL/TLS Let's Encrypt                                               |
|   - Pengendali Header Keamanan & CORS Policy                                      |
|   - Rate Limiting Gateway (/api/v1/auth/*)                                        |
+-----------------------------------------------------------------------------------+
                               | (Internal Bridge Network: houseprice-net)
                        [ TRUST BOUNDARY B ]
                               v
+-----------------------------------------------------------------------------------+
| ZONA 3: TRUSTED INTERNAL ZONE (Isolated Docker Network)                           |
|   +-----------------------------------------------------------------------------+ |
|   | Flask Application Service                                                   | |
|   | - Auth Middleware: Validasi JWT HS256 & Bcrypt Password Verifier            | |
|   | - Blueprints: /api/v1/auth, /api/v1/predict, /api/v1/notifications, /admin   | |
|   | - ML Runtime: Preprocessing Pipeline & Random Forest Engine (.pkl)          | |
|   +-----------------------------------------------------------------------------+ |
|                               | (Internal TCP / Port 5432)                        |
|                               v                                                   |
|   +-----------------------------------------------------------------------------+ |
|   | PostgreSQL Database Service (houseprice_db)                                 | |
|   | - admins, users, notifications, prediction_logs, system_audit_logs,         | |
|   |   master_locations, market_statistics, dataset_metadata, ml_models          | |
|   +-----------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------------+
```

---

## 2. Matriks Analisis Ancaman STRIDE & Kontrol Mitigasi

Berikut matriks evaluasi ancaman keamanan sistem berbasis skenario operasional aktual beserta kontrol pertahanan yang diterapkan[cite: 5]:

| Kategori Ancaman | Komponen Terdampak | Skenario Ancaman (*Threat Scenario*) | Dampak (*Impact*) | Strategi Mitigasi (*Mitigation Control*) |
| :--- | :--- | :--- | :--- | :--- |
| **Spoofing** (Pemalsuan Identitas) | Mobile User Authentication | Penyerang melakukan *brute force* atau *credential stuffing* pada formulir login mobile (`/api/v1/auth/login`). | High | Penerapan *rate limiting* berbasis IP, enkripsi kata sandi menggunakan **Bcrypt** (*salt factor* minimal 12 putaran), dan validasi kompleksitas sandi saat registrasi. |
| **Spoofing** | JWT Token Handling | Penyerang memalsukan signature JWT (*signature spoofing*) untuk menyamar sebagai pengguna sah. | Critical | Verifikasi signature token secara ketat menggunakan algoritma simetris **HS256** dan secret key berkekuatan tinggi (minimal 32 karakter acak) dari environment variable. |
| **Spoofing** | Web Admin Authentication | Penyerang menebak kredensial akun administrator untuk mengambil alih kontrol model ML[cite: 5]. | High | Hashing Bcrypt, manajemen sesi berbasis *secure HTTP-only cookie*, serta pemisahan tabel akun `admins` dan `users` secara fisik di database[cite: 5]. |
| **Tampering** (Manipulasi Data) | REST API Inferensi | Penyerang menyisipkan muatan SQL Injection melalui kolom teks `city` atau `district`[cite: 5]. | High | Penggunaan *Parameterized Queries* via SQLAlchemy ORM; validasi nilai wilayah wajib cocok dengan data aktif pada tabel `master_locations`[cite: 5]. |
| **Tampering** | Klien Mobile / Local Storage | Penyerang mencoba memodifikasi file token sesi atau flag status akun di perangkat yang telah di-*root*. | Medium | Penyimpanan token sesi wajib menggunakan pustaka **`flutter_secure_storage`** yang memanfaatkan enkripsi hardware **Android Keystore**; penolakan penyimpanan token di `SharedPreferences`. |
| **Tampering** | Pipeline Retraining ML | Pengunggahan berkas CSV manipulatif untuk merusak performa bobot model prediksi[cite: 5]. | Critical | Validasi integritas berkas di tabel `dataset_metadata`, verifikasi skema kolom, pembersihan duplikasi, serta pemfilteran *outlier* IQR otomatis sebelum *training*[cite: 5]. |
| **Tampering** | Artefak Model `.pkl` | Perubahan berkas biner serialisasi model di sistem berkas server[cite: 5]. | Critical | Direktori penyimpanan model dikonfigurasi *read-only* bagi proses Flask; verifikasi *hash SHA-256* berkas dilakukan sebelum inferensi dimuat[cite: 5]. |
| **Repudiation** (Penyangkalan Aksi) | Operasional Web Admin | Administrator mengeksekusi *retrain*, mengubah model aktif, atau menghapus dataset lalu menyangkal tindakannya[cite: 5]. | Medium | Pencatatan otomatis ke tabel `system_audit_logs` yang merekam `admin_id`, `action_type`, rincian deskripsi, alamat IP, dan waktu kejadian (*non-repudiation*)[cite: 5]. |
| **Repudiation** | Transaksi Prediksi Mobile | Pengguna menyangkal telah melakukan permintaan inferensi atau kalkulasi properti tertentu. | Low | Setiap request ke `/api/v1/predict` mencatat relasi foreign key `user_id`, parameter masukan fisik properti, nilai hasil, dan waktu transaksi ke tabel `prediction_logs`. |
| **Information Disclosure** (Kebocoran Data) | Jalur Komunikasi Jaringan | Penyadapan paket data (*eavesdropping* / *man-in-the-middle*) pada jaringan publik[cite: 5]. | High | Enkripsi saluran penuh menggunakan **HTTPS (TLS 1.3)** via reverse proxy Let's Encrypt; penonaktifan lalu lintas teks polos (*cleartext disabled*) pada aplikasi Android[cite: 5]. |
| **Information Disclosure** | Privasi Data Pengguna (PII) | Kebocoran data riwayat kalkulasi harga atau daftar notifikasi milik pengguna lain. | High | Penerapan kontrol otorisasi ketat (*multi-tenant data isolation*); kueri endpoint `/predict/history` dan `/notifications` wajib memfilter data berdasarkan `user_id` dari token JWT pemohon. |
| **Information Disclosure** | Repositori Kode Sumber | Kunci rahasia (`JWT_SECRET_KEY`, kredensial DB) tidak sengaja terunggah ke repositori GitHub[cite: 5]. | Critical | Konfigurasi ketat pada berkas `.gitignore`, isolasi variabel rahasia pada file `.env`, dan penyediaan `.env.example` tanpa nilai aktual[cite: 5]. |
| **Denial of Service** (Gangguan Layanan) | Endpoint Komputasi Inferensi | Serangan *traffic flooding* ke rute `/api/v1/predict` yang menghabiskan CPU server saat menjalankan model regresi. | High | Penerapan pembatasan volume request (*rate limiting* per token/IP) serta validasi skema masukan awal (JSON Schema) sebelum alokasi memori inferensi diproses. |
| **Denial of Service** | Basis Data PostgreSQL | Kueri pencarian riwayat prediksi atau notifikasi yang tidak terindeks menyebabkan *table scan* besar dan membebani I/O disk. | Medium | Penerapan indeks komposit B-Tree: `idx_prediction_logs_user_id` dan `idx_notifications_user_lookup` untuk menjamin latensi baca tetap rendah di bawah beban tinggi. |
| **Elevation of Privilege** (Eskalasi Hak Akses) | Akses Lintas Peran (Role Bypass) | Pengguna aplikasi mobile memanipulasi token JWT untuk mengakses rute administratif dashboard web (`/admin/*`). | Critical | Pemisahan mekanisme otentikasi secara tegas: token JWT mobile divalidasi oleh middleware `UserBearerAuth`, sedangkan Web Admin menggunakan verifikasi sesi independen berbasis tabel `admins`. |
| **Elevation of Privilege** | Insecure Direct Object References (IDOR) | Pengguna mengubah parameter path ID pada rute `/notifications/{id}/read` untuk menandai atau membaca notifikasi milik pengguna lain. | Medium | Validasi kepemilikan objek pada backend Flask: `WHERE id = :notif_id AND user_id = :current_user_id`; kembalikan `404 Not Found` jika entitas bukan milik akun terkait. |

---

## 3. Rencana Verifikasi Keamanan (Security Verification Plan)

Setiap mekanisme mitigasi di atas diverifikasi melalui rangkaian pengujian oleh tim Quality Assurance:

1. **Uji Penetrasi Autentikasi (Auth Pentest):**
   * Mengirim request tanpa token ke endpoint privat (`/api/v1/predict`, `/api/v1/notifications`, `/api/v1/auth/me`) $\rightarrow$ Memastikan respons konsisten `401 Unauthorized`.
   * Mengirim request dengan token JWT yang diubah payload-nya (*tampered signature*) $\rightarrow$ Memastikan server menolak token.
2. **Uji Validasi Batas & Sanitasi Masukan:**
   * Injeksi karakter SQL (`' OR '1'='1`) pada field `city`, `district`, email, dan password $\rightarrow$ Memastikan masukan diperlakukan sebagai teks literal aman.
   * Masukan angka negatif atau nol pada parameter luas area $\rightarrow$ Memastikan sistem merespons `400 Bad Request` tanpa *server crash*.
3. **Uji Isolasi Data (Multi-Tenant Verification):**
   * Pengguna A memanggil endpoint `/api/v1/predict/history` $\rightarrow$ Memastikan tidak ada catatan prediksi milik Pengguna B yang muncul dalam daftar respons.
   * Pengguna A mengirim permintaan PATCH ke notifikasi milik Pengguna B $\rightarrow$ Memastikan server mengembalikan kode status error hak akses.
4. **Audit Kebocoran Repositori:**
   * Melakukan pemindaian Git menggunakan alat deteksi rahasia (*secret scanner*) untuk memastikan tidak ada kunci privat, token aktif, atau berkas `.env` yang masuk ke riwayat *commit*.