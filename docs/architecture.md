# Arsitektur Sistem Terintegrasi (System Architecture) - HousePrice

Dokumen ini mendefinisikan arsitektur perangkat lunak makro dan mikro sistem HousePrice, diagram kontainer model C4, struktur arsitektur berlapis (*layered Clean Architecture*) pada aplikasi mobile Flutter, alur kerja inferensi terproteksi JWT, pipeline retraining backend Flask, serta topologi infrastruktur komputasi *cloud*.

---

## 1. Arsitektur Makro Sistem (C4 Container Diagram)

Sistem HousePrice menerapkan pola arsitektur layanan terpisah (*decoupled client-server architecture*) dengan gerbang keamanan ketat (*Strict Gate Authentication*) yang memisahkan antarmuka klien mobile dari mesin inferensi cloud dan portal manajemen admin:

```
                      DIAGRAM KONTAINER SISTEM (C4 LEVEL 2)
                      
  +-------------------------------+             +--------------------------+
  |    Pengguna Mobile Klien      |             |   Administrator Sistem   |
  | (Publik Baru / Terdaftar)     |             +--------------------------+
  +-------------------------------+                          |
                  |                                          | (Web Browser)
                  | (Aplikasi Android)                       v
                  v                             +--------------------------+
  +-------------------------------+             |   Web Admin Dashboard    |
  |     Flutter Mobile Client     |             |   - Metrik Model (R²)    |
  |  - Splash, Onboard, Landing   |             |   - Dataset CSV Manager  |
  |  - Auth (Strict Gate JWT)     |             |   - ML Retrain Trigger   |
  |  - Form Prediksi, Cek, KPR    |             |   - Audit Logs Viewer    |
  |  - Notifications & Profile    |             +--------------------------+
  |  - Secure Storage & Intercept |                          |
  +-------------------------------+                          |
                  |                                          |
                  | (HTTPS / Bearer JWT JSON API)            | (HTTPS / Form Data)
                  +---------------------+--------------------+
                                        |
                                        v
                     +--------------------------------------+
                     |   Reverse Proxy (Coolify / Traefik)  |
                     |   - Auto SSL/TLS Let's Encrypt       |
                     |   - CORS & HTTP Security Headers     |
                     +--------------------------------------+
                                        | (Internal Docker Network)
                                        v
                     +--------------------------------------+
                     |   Flask Application Service (WSGI)   |
                     |   +--------------------------------+ |
                     |   | Blueprints:                    | |
                     |   | - /api/v1/auth (Mobile User)   | |
                     |   | - /api/v1/predict (ML Core)    | |
                     |   | - /api/v1/notifications        | |
                     |   | - /api/v1/market-stats & locs  | |
                     |   | - /admin (Dashboard & Retrain) | |
                     |   +--------------------------------+ |
                     |   | Auth Middleware (JWT HS256)    | |
                     |   +--------------------------------+ |
                     |   | Preprocessing & Scikit Engine  | |
                     |   +--------------------------------+ |
                     |   | Model Serialized (.pkl)        | |
                     |   +--------------------------------+ |
                     +--------------------------------------+
                                        |
                                        | (TCP / Port 5432)
                                        v
                     +--------------------------------------+
                     |      PostgreSQL Database Service     |
                     |   - Schema: 9 Relational Tables      |
                     |   - Strict User & Prediction Link    |
                     |   - Persistent Docker Volume         |
                     +--------------------------------------+
```

---

## 2. Arsitektur Aplikasi Mobile (Flutter Clean Architecture & Route Guard)

Aplikasi klien Android dibangun menggunakan pendekatan **Feature-First Clean Architecture** dengan pembatasan rute (*Navigation Guard*) untuk memastikan fitur inti hanya dapat diakses setelah sesi pengguna terverifikasi:

```
  +--------------------------------------------------------------------------+
  | 0. ROUTING & NAVIGATION GUARD (AUTH GATEWAY)                             |
  |    - Splash State Checker -> Onboarding / Landing / Main Nav             |
  |    - Unauthenticated: LandingScreen, LoginScreen, RegisterScreen         |
  |    - Authenticated: Bottom Navigation Dashboard                          |
  +--------------------------------------------------------------------------+
                                     |
                                     v
  +--------------------------------------------------------------------------+
  | 1. PRESENTATION LAYER (UI, WIDGETS & STATE MANAGEMENT)                   |
  |    - Features:                                                           |
  |      * Onboarding: SplashScreen, OnboardingCarousel, LandingScreen       |
  |      * Auth: LoginScreen, RegisterScreen                                 |
  |      * Property & ML: HomeScreen, PredictFormScreen, ResultScreen        |
  |      * Finance: KprCalculatorScreen                                      |
  |      * Personal: NotificationScreen, ProfileScreen (Cloud History)       |
  |    - State Management: BLoC / Cubit                                      |
  |      (AuthBloc, PredictionBloc, NotificationBloc, ProfileBloc)           |
  +--------------------------------------------------------------------------+
                                     | (Event / Intent)
                                     v
  +--------------------------------------------------------------------------+
  | 2. DOMAIN LAYER (BUSINESS LOGIC & USE CASES)                             |
  |    - Use Cases: LoginUser, RegisterUser, PredictHousePrice,              |
  |                 CalculateKprAnnuity, EvaluatePriceFairness,              |
  |                 FetchUserHistory, FetchNotifications                     |
  |    - Business Rules:                                                     |
  |      * Valuation Comparator: Deviasi harga (-10% s/d +10%)               |
  |      * KPR Annuity Engine: Rumus kalkulasi angsuran bunga tetap bulanan  |
  +--------------------------------------------------------------------------+
                                     | (Method Call / Repository)
                                     v
  +--------------------------------------------------------------------------+
  | 3. DATA & NETWORK LAYER (INFRASTRUCTURE & PERSISTENCE)                   |
  |    - Remote Data Source: Dio HTTP Client dengan `AuthInterceptor`        |
  |      (Auto-inject `Authorization: Bearer <token>` & auto-logout on 401)  |
  |    - Local Data Source:                                                  |
  |      * `flutter_secure_storage`: Penyimpanan terenkripsi JWT token       |
  |      * `shared_preferences`: Flag status onboarding pertama buka        |
  |    - DTO & Mappers: UserModel, PredictionModel, NotificationModel        |
  +--------------------------------------------------------------------------+
```

### Penjabaran Komponen Tiap Lapisan:
1. **Routing & Navigation Guard:** Memvalidasi status sesi saat aplikasi dibuka:
   * Jika baru pertama kali diinstal $\rightarrow$ diarahkan ke `OnboardingScreen`.
   * Jika belum terotentikasi $\rightarrow$ diarahkan ke `LandingScreen`, `LoginScreen`, atau `RegisterScreen`.
   * Jika token valid $\rightarrow$ diarahkan langsung ke `MainNavigationScreen` (Beranda).
2. **Presentation Layer:** Mengelola siklus hidup widget, form validasi lokal, pemrosesan event input, visualisasi statistik pasar, serta animasi transisi. Seluruh state dikelola secara reaktif menggunakan BLoC/Cubit.
3. **Domain Layer:** Mengisolasi logika bisnis independen tanpa dependensi pada UI atau library eksternal. Berisi kalkulasi matematis deviasi kewajaran harga pasar dan rumus angsuran KPR berbasis anuitas.
4. **Data Layer:** Berfungsi sebagai jembatan *dual-source*:
   * Mengirim request terproteksi ke backend API melalui Dio dengan `AuthInterceptor` yang otomatis menangani penyematan Bearer Token dan merespons status `401 Unauthorized`.
   * Mengelola penyimpanan sesi lokal menggunakan `flutter_secure_storage` untuk menjamin keamanan token JWT di lingkungan Android Keystore.

---

## 3. Alur Komputasi Inferensi Backend (Strict Gate Inference Pipeline)

Setiap permintaan estimasi harga yang dikirim dari klien mobile diproses melalui alur sekuensial yang terlindungi di backend Flask:

```
  [ Klien Mobile ] 
         | 
         | 1. POST /api/v1/predict 
         |    Headers: Authorization: Bearer <jwt_token>
         |    JSON Payload: {city, district, surface_area, building_area, bed, bath, carport}
         v
  +--------------------------------------------------------------------------+
  | [ Ingress & JWT Authentication Middleware ]                              |
  | - Pengecekan keberadaan header Authorization: Bearer                     |
  | - Verifikasi integritas tanda tangan JWT & masa berlaku (exp) via HS256  |
  | - Ekstraksi identitas `user_id` dari payload token                       |
  | - Jika token tidak valid / kedaluwarsa -> Tolak dengan HTTP 401          |
  +--------------------------------------------------------------------------+
         | (Token Terverifikasi, user_id Didapatkan)
         v
  +--------------------------------------------------------------------------+
  | [ Input Validation Gateway ]                                             |
  | - Validasi tipe data & format JSON menggunakan Marshmallow / Pydantic    |
  | - Pengecekan batas nilai ekstrem (surface_area >= 1, building_area >= 1)  |
  | - Pengecekan keberadaan lokasi pada tabel `master_locations`             |
  +--------------------------------------------------------------------------+
         | (Lolos Validasi Skema)
         v
  +--------------------------------------------------------------------------+
  | [ Feature Transformation Pipeline ]                                      |
  | - One-Hot Encoding fitur wilayah (city, district) via encoder map        |
  | - Feature Scaling numerik menggunakan StandardScaler yang tersimpan      |
  +--------------------------------------------------------------------------+
         | (Matriks Fitur Siap Inferensi)
         v
  +--------------------------------------------------------------------------+
  | [ Model Execution Engine ]                                               |
  | - Model Random Forest Regressor aktif (.pkl) memproses inferensi         |
  | - Kalkulasi rentang toleransi harga wajar (Lower Bound s/d Upper Bound)   |
  +--------------------------------------------------------------------------+
         | (Nilai Estimasi Dihasilkan)
         v
  +--------------------------------------------------------------------------+
  | [ Async DB Logger & Response Dispatcher ]                                |
  | - Simpan masukan fisik, nominal hasil, latensi, IP, serta `user_id`      |
  |   secara relasional ke tabel `prediction_logs`                           |
  | - Kembalikan status HTTP 200 OK dengan JSON payload standar ke Mobile    |
  +--------------------------------------------------------------------------+
         |
         | 2. JSON Response (prediction_id, predicted_price, tolerance_range, latency_ms)
         v
  [ Klien Mobile ] (Tampilkan Hasil Prediksi & Tambahkan ke Riwayat Pengguna)
```

---

## 4. Alur Manajemen & Retraining Model ML (Admin Pipeline)

Sistem menyediakan alur pembaruan model secara terprogram melalui antarmuka Web Admin:

```
  [ Web Admin ] ---> Unggah CSV Baru ---> Validasi Skema & Simpan Metadata
                            |
                            v
                  Pembersihan Duplikat & Imputasi Nilai Kosong
                            |
                            v
                  Pemfilteran Outlier Berbasis IQR (Harga & Luas)
                            |
                            v
                  Pemisahan Data (80% Train, 20% Test Split)
                            |
                            v
                  Pelatihan Ulang Model Regresi (Random Forest)
                            |
                            v
                  Evaluasi Metrik Uji (Hitung R², MAE, RMSE)
                            |
                            v
                  Simpan Entri Model ke `ml_models` (Status Nonaktif)
                            |
                            v
  [ Web Admin ] <--- Tampilkan Komparasi Metrik (Model Baru vs Model Aktif)
                            | (Admin Menyetujui Pengaktifan)
                            v
                  Perbarui `is_active = TRUE` & Muat Ulang File .pkl
                            |
                            v
                  Catat Jejak Operasi ke `system_audit_logs`
```

---

## 5. Topologi Jaringan & Komposisi Infrastruktur (Deployment Topology)

Komponen server dijalankan secara terisolasi menggunakan **Coolify** dan **Docker Compose** pada satu node Virtual Private Server (VPS):

1. **Ingress Layer (Port 80 & 443):**
   * Menggunakan reverse proxy Traefik bawaan Coolify untuk manajemen sertifikat SSL/TLS otomatis (*Let's Encrypt*).
   * Melakukan terminasi SSL dan meneruskan trafik aman ke container internal Flask via port HTTP 5000.
   * Mengelola kebijakan CORS untuk membatasi origin akses API yang tidak sah.
2. **Application Network (`houseprice-net`):**
   * Jaringan jembatan (*Docker bridge network*) privat yang menghubungkan container `flask-backend` dan container `postgres-db`.
   * Port 5432 basis data PostgreSQL ditutup dari internet publik dan hanya dapat diakses melalui host internal `postgres-db`.
3. **Keamanan Kredensial & Enkripsi:**
   * Kunci rahasia JWT (`JWT_SECRET_KEY`) dan kredensial basis data diinjeksikan secara terisolasi melalui *environment variables* (`.env`).
   * Kata sandi akun pengguna mobile dan akun admin dienkripsi satu arah menggunakan modul `bcrypt` sebelum disimpan ke basis data.
4. **Persistent Storage:**
   * `postgres-data`: Docker volume untuk menyimpan data relasional PostgreSQL (9 tabel) agar persistensi data terjamin saat container di-restart.
   * `model-storage`: Direktori server terproteksi (`/app/models/`) untuk menyimpan artefak serialisasi model `.pkl`.
   * `dataset-storage`: Direktori server (`/app/uploads/datasets/`) untuk menyimpan riwayat berkas CSV yang diunggah admin.