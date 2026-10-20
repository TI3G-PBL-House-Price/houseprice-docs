# Arsitektur Sistem Terintegrasi (System Architecture) - HousePrice

Dokumen ini mendefinisikan arsitektur perangkat lunak makro dan mikro sistem HousePrice, diagram kontainer model C4, struktur arsitektur berlapis (*layered architecture*) pada aplikasi mobile Flutter, alur kerja inferensi dan pelatihan backend Flask, serta topologi infrastruktur komputasi *cloud*[cite: 20, 23, 26].

---

## 1. Arsitektur Makro Sistem (C4 Container Diagram)

Sistem HousePrice dirancang dengan pola arsitektur layanan terpisah (*decoupled client-server architecture*) yang memisahkan antarmuka klien mobile dari mesin inferensi cloud dan portal manajemen admin[cite: 20, 23]:

```
                      DIAGRAM KONTAINER SISTEM (C4 LEVEL 2)
                      
  +--------------------------+               +--------------------------+
  |  Pengguna Mobile (Umum)  |               |   Administrator Sistem   |
  +--------------------------+               +--------------------------+
               |                                           |
               | (Aplikasi Android)                        | (Web Browser)
               v                                           v
  +--------------------------+               +--------------------------+
  |  Flutter Mobile Client   |               |   Web Admin Dashboard    |
  |  - Form Input & Validasi |               |   - Metrik Model (R²)    |
  |  - Cek Kewajaran & KPR   |               |   - Dataset CSV Manager  |
  |  - Local Storage SQLite  |               |   - ML Retrain Trigger   |
  +--------------------------+               +--------------------------+
               |                                           |
               | (HTTPS / JSON REST API)                   | (HTTPS / Form Data)
               +---------------------+---------------------+
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
                  |   | Blueprints: /api/v1 & /admin   | |
                  |   +--------------------------------+ |
                  |   | Preprocessing Pipeline (Scikit)| |
                  |   +--------------------------------+ |
                  |   | Inference Engine (Model .pkl)  | |
                  |   +--------------------------------+ |
                  +--------------------------------------+
                                     |
                                     | (TCP / Port 5432)
                                     v
                  +--------------------------------------+
                  |      PostgreSQL Database Service     |
                  |   - Schema: 7 Relational Tables    |
                  |   - Persistent Docker Volume         |
                  +--------------------------------------+
```

---

## 2. Arsitektur Aplikasi Mobile (Flutter Layered Architecture)

Aplikasi klien Android dibangun menggunakan pendekatan **Layered Architecture** untuk memastikan pemisahan tanggung jawab kode yang rapi serta memudahkan pengujian unit maupun widget[cite: 25, 26]:

```
  +--------------------------------------------------------------------------+
  | 1. PRESENTATION LAYER (UI & WIDGETS)                                     |
  |    - Screens: HomeScreen, PredictFormScreen, ResultScreen, KprScreen     |
  |    - Widgets: PropertyCard, MetricBadge, LocationDropdown, HistoryTile   |
  |    - State Management: Provider / BLoC (InputState, PredictionState)     |
  +--------------------------------------------------------------------------+
                                     | (Event / Intent)
                                     v
  +--------------------------------------------------------------------------+
  | 2. BUSINESS LOGIC LAYER (DOMAIN & RULES)                                 |
  |    - Form Input Validation: Pengecekan batas nilai ekstrem (BVA/EP)      |
  |    - Valuation Comparator: Evaluasi deviasi harga (Murah, Wajar, Mahal)  |
  |    - KPR Annuity Engine: Rumus kalkulasi angsuran bunga tetap bulanan    |
  +--------------------------------------------------------------------------+
                                     | (Method Call / Repository)
                                     v
  +--------------------------------------------------------------------------+
  | 3. DATA & NETWORK LAYER (INFRASTRUCTURE)                                 |
  |    - Remote Data Source: HTTP Client (RESTful API POST/GET)              |
  |    - Local Data Source: SQLite Helper & SharedPreferences                |
  |    - DTO & Mappers: Serialisasi JSON ke Dart Entity (HousePriceModel)    |
  +--------------------------------------------------------------------------+
```

### Penjabaran Komponen Tiap Lapisan:
1. **Presentation Layer:** Mengelola siklus hidup antarmuka pengguna, pemrosesan event input form, visualisasi grafik harga pasar, serta penanganan animasi transisi layar[cite: 24, 25].
2. **Business Logic Layer:** Mengisolasi aturan kalkulasi matematis murni tanpa dependensi widget UI:
   * *Formula Deviasi Kewajaran:* Menghitung persentase simpangan penawaran penjual terhadap hasil taksiran model ML ($\Delta\%$)[cite: 24, 25].
   * *Formula Bunga Anuitas KPR:* Menghitung besaran cicilan rutin pokok dan bunga per bulan berbasis tenor dan suku bunga[cite: 25, 26].
3. **Data Layer:** Berfungsi sebagai jembatan *dual-source*:
   * Mengirim parameter fisik rumah ke endpoint `/api/v1/predict` melalui protokol HTTP asinkron[cite: 23, 26].
   * Menyimpan dan membaca riwayat kalkulasi pada basis data lokal SQLite perangkat secara *offline-first*[cite: 25, 31].

---

## 3. Alur Komputasi Inferensi Backend (Inference Pipeline)

Setiap permintaan estimasi harga yang dikirim dari klien mobile diproses melalui alur sekuensial yang ketat pada backend Flask[cite: 23, 26]:

```
  [ Klien Mobile ] 
         | 
         | 1. POST /api/v1/predict (JSON Payload: LT, LB, bed, bath, carport, loc)
         v
  +--------------------------------------------------------------------------+
  | [ Ingress & Validation Gateway ]                                         |
  | - Validasi tipe data & format JSON menggunakan Marshmallow / Pydantic    |
  | - Pengecekan rentang validitas angka (LT >= 1, LB >= 1, kamar >= 0)      |
  +--------------------------------------------------------------------------+
         | (Lolos Validasi)
         v
  +--------------------------------------------------------------------------+
  | [ Feature Transformation Pipeline ]                                      |
  | - One-Hot Encoding fitur kategorikal (city, district) via encoder map    |
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
  | - Simpan masukan, output nominal, latensi, dan IP ke `prediction_logs`   |
  | - Kembalikan status HTTP 200 OK dengan JSON payload standar ke Mobile    |
  +--------------------------------------------------------------------------+
         |
         | 2. JSON Response (predicted_price, lower_bound, upper_bound, latency_ms)
         v
  [ Klien Mobile ] (Tampilkan Hasil Prediksi di Layar)
```

---

## 4. Alur Manajemen & Retraining Model ML (Admin Pipeline)

Sistem menyediakan alur pembaruan model secara terprogram melalui antarmuka Web Admin[cite: 20, 23]:

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
```

---

## 5. Topologi Jaringan & Komposisi Infrastruktur (Deployment Topology)

Komponen server dijalankan secara terisolasi menggunakan **Coolify** dan **Docker Compose** pada satu node Virtual Private Server (VPS)[cite: 41]:

1. **Ingress Layer (Port 80 & 443):**
   * Menggunakan reverse proxy Traefik bawaan Coolify untuk manajemen sertifikat SSL/TLS gratis (Let's Encrypt).
   * Melakukan terminasi SSL dan meneruskan trafik aman ke container internal Flask via port HTTP 5000.
2. **Application Network (`houseprice-net`):**
   * Jaringan jembatan (*Docker bridge network*) privat yang menghubungkan container `flask-backend` dan container `postgres-db`.
   * Port 5432 basis data PostgreSQL ditutup dari internet publik dan hanya dapat diakses melalui nama host internal `postgres-db`.
3. **Persistent Storage:**
   * `postgres-data`: Docker volume untuk menyimpan berkas basis data PostgreSQL agar data tidak hilang saat container di-restart.
   * `model-storage`: Direktori server terproteksi (`/app/models/`) untuk menyimpan artefak serialisasi model `.pkl`.
   * `dataset-storage`: Direktori server (`/app/uploads/datasets/`) untuk menyimpan riwayat berkas CSV yang diunggah admin[cite: 1].