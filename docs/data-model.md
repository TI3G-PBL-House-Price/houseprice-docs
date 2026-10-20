# Model & Skema Basis Data (Data Model & Schema) - HousePrice

Dokumen ini memuat spesifikasi arsitektur data, perancangan skema konseptual dan fisik, kamus data komprehensif (*data dictionary*), relasi antar-entitas, strategi pengindeksan (*indexing*), serta skrip DDL SQL untuk basis data **PostgreSQL** pada sistem HousePrice[cite: 1, 23].

---

## 1. Diagram Relasi Entitas (Entity-Relationship Model)

Basis data HousePrice terdiri dari **7 tabel relasional** yang terbagi ke dalam dua domain utama: **Manajemen Administratif & Artefak ML** (berelasi secara referensial) serta **Analitik & Telemetri Berlatensi Rendah** (*decoupled logging* untuk performa kueri cepat)[cite: 1]:

```
                      DIAGRAM RELASI BASIS DATA (ERD)
                      
   +-------------------------------------------------------------+
   |                           admins                            |
   +-------------------------------------------------------------+
   | PK  id             : UUID                                   |
   |     username       : VARCHAR(50)  [UNIQUE]                  |
   |     email          : VARCHAR(100) [UNIQUE]                  |
   |     password_hash  : VARCHAR(255)                           |
   |     full_name      : VARCHAR(100)                           |
   |     is_active      : BOOLEAN                                |
   |     created_at     : TIMESTAMPTZ                            |
   |     updated_at     : TIMESTAMPTZ                            |
   +-------------------------------------------------------------+
          | 1                      | 1                     | 1
          |                        |                       |
          | N                      | N                     | N
          v                        v                       v
   +-------------------+    +--------------------+  +--------------------+
   | dataset_metadata  |    |     ml_models      |  | system_audit_logs  |
   +-------------------+    +--------------------+  +--------------------+
   | PK id             |    | PK id              |  | PK id              |
   |    file_name      |    |    model_name      |  | FK admin_id        |
   |    file_path      |    |    version         |  |    action_type     |
   |    total_records  |    |    model_file_path |  |    module_name     |
   |    file_size_bytes|    |    algorithm_type  |  |    description     |
   | FK uploaded_by    |    |    r2_score        |  |    ip_address      |
   |    created_at     |    |    mae_score       |  |    user_agent      |
   +-------------------+    |    rmse_score      |  |    created_at      |
                            |    is_active       |  +--------------------+
                            | FK trained_by      |
                            |    created_at      |
                            |    updated_at      |
                            +--------------------+

  [ DOMAIN TELEMETRI & ANALITIK (DECOUPLED HIGH-THROUGHPUT TABLES) ]

   +-----------------------+  +-----------------------+  +----------------------+
   |   master_locations    |  |   market_statistics   |  |   prediction_logs    |
   +-----------------------+  +-----------------------+  +----------------------+
   | PK id         : SERIAL|  | PK id         : SERIAL|  | PK id          : UUID|
   |    city       : STR   |  |    city       : STR   |  |    city        : STR |
   |    district   : STR   |  |    district   : STR   |  |    district    : STR |
   |    is_active  : BOOL  |  |    avg_price  : NUM   |  |    surface_area: NUM |
   |    created_at : TIME  |  |    avg_sqm    : NUM   |  |    building_area:NUM |
   +-----------------------+  |    avg_surface: NUM   |  |    bed,bath,car: INT |
                              |    avg_build  : NUM   |  |    predicted_pr: NUM |
                              |    total_props: INT   |  |    model_vers  : STR |
                              |    updated_at : TIME  |  |    latency_ms  : INT |
                              +-----------------------+  |    client_ip   : STR |
                                                         |    created_at  : TIME|
                                                         +----------------------+
```

---

## 2. Kamus Data Lengkap (Data Dictionary)

### 2.1 Tabel `admins`
Menyimpan kredensial dan profil administrator pengelola sistem dan model ML[cite: 1].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik global entitas admin[cite: 1]. | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11`[cite: 1] |
| `username` | `VARCHAR(50)` | `NOT NULL`, `UNIQUE` | Nama unik untuk login dashboard web[cite: 1]. | `admin_hanif`[cite: 1] |
| `email` | `VARCHAR(100)` | `NOT NULL`, `UNIQUE`, Format Email | Alamat surat elektronik terdaftar[cite: 1]. | `hanif@houseprice.id`[cite: 1] |
| `password_hash` | `VARCHAR(255)` | `NOT NULL` | Hash sandi terenkripsi algoritma Bcrypt[cite: 1]. | `$2b$12$e8m4...`[cite: 1] |
| `full_name` | `VARCHAR(100)` | `NOT NULL` | Nama lengkap personal administrator[cite: 1]. | `Abdul Rahman Hanif`[cite: 1] |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `TRUE` | Status hak akses aktif/nonaktif akun[cite: 1]. | `true`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel pembuatan akun admin[cite: 1]. | `2026-10-01 08:30:00+07`[cite: 1] |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel pembaruan data profil[cite: 1]. | `2026-10-01 08:30:00+07`[cite: 1] |

### 2.2 Tabel `master_locations`
Menyimpan referensi wilayah administratif legal di DKI Jakarta untuk sinkronisasi dropdown form mobile[cite: 1, 24].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY`, Autoincrement | ID sekuensial wilayah[cite: 1]. | `1` |
| `city` | `VARCHAR(50)` | `NOT NULL` | Nama kota administrasi di DKI Jakarta[cite: 1]. | `Jakarta Barat`[cite: 1] |
| `district` | `VARCHAR(50)` | `NOT NULL` | Nama kecamatan/kawasan perumahan[cite: 1]. | `Citra Garden`[cite: 1] |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `TRUE` | Status ketersediaan wilayah pada form mobile[cite: 1]. | `true`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu data referensi ditambahkan[cite: 1]. | `2026-10-01 09:00:00+07`[cite: 1] |

### 2.3 Tabel `dataset_metadata`
Mencatat riwayat berkas dataset CSV properti yang diunggah oleh admin[cite: 1, 23].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik catatan dataset[cite: 1]. | `b1ffcd88-8d1c-4ef8-bb6d-7cc0ce491b22`[cite: 1] |
| `file_name` | `VARCHAR(255)` | `NOT NULL` | Nama asli berkas CSV fisik[cite: 1]. | `jakarta_house_v1.csv`[cite: 1] |
| `file_path` | `VARCHAR(500)` | `NOT NULL` | Path direktori penyimpanan berkas di server[cite: 1]. | `/uploads/datasets/jakarta_house_v1.csv`[cite: 1] |
| `total_records` | `INTEGER` | `NOT NULL`, Check `(total_records > 0)` | Total baris data properti di dalam berkas[cite: 1]. | `10000`[cite: 1] |
| `file_size_bytes`| `BIGINT` | `NOT NULL` | Ukuran berkas CSV dalam satuan byte[cite: 1]. | `845620`[cite: 1] |
| `uploaded_by` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Relasi ke akun admin pengunggah berkas[cite: 1]. | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel unggah berkas[cite: 1]. | `2026-10-02 09:15:00+07`[cite: 1] |

### 2.4 Tabel `ml_models`
Menyimpan registri model regresi Machine Learning, metrik evaluasi, dan versi produksi aktif[cite: 1, 23].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik artefak model ML[cite: 1]. | `c2eedf77-7e2b-4ef8-bb6d-8dd1df502c33`[cite: 1] |
| `model_name` | `VARCHAR(100)` | `NOT NULL` | Nama representatif algoritma model[cite: 1]. | `Random Forest Regressor`[cite: 1] |
| `version` | `VARCHAR(20)` | `NOT NULL` | Penomoran versi model (Semantic)[cite: 1]. | `v1.0.0`[cite: 1] |
| `model_file_path`| `VARCHAR(500)`| `NOT NULL` | Lokasi berkas serialisasi biner `.pkl`[cite: 1]. | `/models/rf_house_price_v1.pkl`[cite: 1] |
| `algorithm_type`| `VARCHAR(50)` | `NOT NULL` | Golongan algoritma (`Ensemble`/`Linear`)[cite: 1]. | `Ensemble`[cite: 1] |
| `r2_score` | `NUMERIC(6,4)` | `NOT NULL`, Check `(r2_score <= 1.0)` | Skor koefisien determinasi evaluasi uji[cite: 1]. | `0.8272`[cite: 1] |
| `mae_score` | `NUMERIC(18,2)`| `NOT NULL`, Check `(mae_score >= 0)` | Nilai Mean Absolute Error dalam Rupiah[cite: 1]. | `1063034875.00`[cite: 1] |
| `rmse_score` | `NUMERIC(18,2)`| `NOT NULL`, Check `(rmse_score >= 0)`| Nilai Root Mean Squared Error dalam Rupiah[cite: 1]. | `1808105786.00`[cite: 1] |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `FALSE` | Penanda model utama inferensi API[cite: 1]. | `true`[cite: 1] |
| `trained_by` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Relasi admin yang melatih model[cite: 1]. | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu pelatihan model selesai[cite: 1]. | `2026-10-02 10:45:00+07`[cite: 1] |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu status aktif/nonaktif diperbarui[cite: 1]. | `2026-10-02 10:45:00+07`[cite: 1] |

### 2.5 Tabel `market_statistics`
Menyimpan agregasi rata-rata harga dan luas per wilayah untuk konsumsi cepat Beranda Mobile[cite: 1, 24].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY`, Autoincrement | ID sekuensial statistik pasar[cite: 1]. | `1` |
| `city` | `VARCHAR(50)` | `NOT NULL` | Nama kota administrasi wilayah data[cite: 1]. | `Jakarta Barat`[cite: 1] |
| `district` | `VARCHAR(50)` | `NULLABLE` | Nama kecamatan (`NULL` jika rata-rata kota)[cite: 1].| `Citra Garden`[cite: 1] |
| `avg_price` | `NUMERIC(18,2)`| `NOT NULL`, Check `(avg_price >= 0)` | Rata-rata nominal harga rumah (Rp)[cite: 1]. | `2450000000.00`[cite: 1] |
| `avg_price_per_sqm`| `NUMERIC(18,2)`| `NOT NULL`, Check `(avg_price_per_sqm >= 0)` | Rata-rata harga per meter persegi ($m^2$)[cite: 1]. | `18500000.00`[cite: 1] |
| `avg_surface_area`| `NUMERIC(8,2)` | `NOT NULL`, Check `(avg_surface_area > 0)` | Rata-rata luas tanah properti ($m^2$)[cite: 1]. | `125.50`[cite: 1] |
| `avg_building_area`| `NUMERIC(8,2)`| `NOT NULL`, Check `(avg_building_area > 0)` | Rata-rata luas bangunan properti ($m^2$)[cite: 1]. | `140.00`[cite: 1] |
| `total_properties`| `INTEGER` | `NOT NULL`, Check `(total_properties > 0)` | Volume sampel properti dalam kalkulasi[cite: 1]. | `450`[cite: 1] |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel agregasi diperbarui[cite: 1]. | `2026-10-03 14:00:00+07`[cite: 1] |

### 2.6 Tabel `prediction_logs`
Mencatat telemetri setiap permintaan inferensi prediksi harga yang masuk dari aplikasi mobile[cite: 1, 23].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal transaksi prediksi[cite: 1]. | `d3ffea66-6f3a-4ef8-bb6d-9ee2ea613d44`[cite: 1] |
| `city` | `VARCHAR(50)` | `NOT NULL` | Masukan nama kota administrasi[cite: 1]. | `Jakarta Barat`[cite: 1] |
| `district` | `VARCHAR(50)` | `NOT NULL` | Masukan nama kecamatan properti[cite: 1]. | `Citra Garden`[cite: 1] |
| `surface_area` | `NUMERIC(8,2)` | `NOT NULL`, Check `(surface_area >= 1)` | Masukan luas tanah properti ($m^2$)[cite: 1]. | `100.00`[cite: 1] |
| `building_area`| `NUMERIC(8,2)` | `NOT NULL`, Check `(building_area >= 1)` | Masukan luas bangunan properti ($m^2$)[cite: 1]. | `120.00`[cite: 1] |
| `bed` | `INTEGER` | `NOT NULL`, Check `(bed >= 0)` | Masukan jumlah kamar tidur[cite: 1]. | `3`[cite: 1] |
| `bath` | `INTEGER` | `NOT NULL`, Check `(bath >= 0)` | Masukan jumlah fasilitas kamar mandi[cite: 1]. | `2`[cite: 1] |
| `carport` | `INTEGER` | `NOT NULL`, Check `(carport >= 0)` | Masukan kapasitas ruang parkir mobil[cite: 1]. | `1` |
| `predicted_price`| `NUMERIC(18,2)`| `NOT NULL`, Check `(predicted_price >= 0)` | Estimasi nominal harga dari model ML[cite: 1]. | `2150000000.00`[cite: 1] |
| `model_version`| `VARCHAR(20)` | `NOT NULL` | Versi model ML yang mengeksekusi[cite: 1]. | `v1.0.0`[cite: 1] |
| `latency_ms` | `INTEGER` | `NULLABLE`, Check `(latency_ms >= 0)` | Waktu komputasi server dalam milidetik[cite: 1]. | `120`[cite: 1] |
| `client_ip` | `VARCHAR(45)` | `NULLABLE` | Alamat IP pemohon (IPv4 / IPv6)[cite: 1]. | `180.252.164.21`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel permintaan inferensi[cite: 1]. | `2026-10-04 11:20:15+07`[cite: 1] |

### 2.7 Tabel `system_audit_logs`
Merekam jejak operasional administrator untuk akuntabilitas sistem (*security compliance*)[cite: 1, 23].

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik entri catatan audit[cite: 1]. | `e4aaeb55-5f2b-4ef8-bb6d-0ff3fa724e55`[cite: 1] |
| `admin_id` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Akun admin yang mengeksekusi operasi[cite: 1]. | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11`[cite: 1] |
| `action_type` | `VARCHAR(50)` | `NOT NULL` | Jenis aksi (`UPLOAD`, `RETRAIN`, `SWITCH`)[cite: 1]. | `SWITCH_MODEL`[cite: 1] |
| `module_name` | `VARCHAR(50)` | `NOT NULL` | Modul yang ditargetkan (`ML_REGISTRY`)[cite: 1]. | `ML_REGISTRY`[cite: 1] |
| `description` | `TEXT` | `NOT NULL` | Penjelasan tekstual detail kegiatan[cite: 1]. | `Mengaktifkan model RF v1.0.0`[cite: 1] |
| `ip_address` | `VARCHAR(45)` | `NULLABLE` | Alamat IP perangkat pengelola[cite: 1]. | `192.168.1.15`[cite: 1] |
| `user_agent` | `TEXT` | `NULLABLE` | Data identitas peramban perangkat admin[cite: 1]. | `Mozilla/5.0...`[cite: 1] |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel presisi operasi audit[cite: 1]. | `2026-10-04 14:10:00+07`[cite: 1] |

---

## 3. Strategi Pengindeksan Basis Data (Indexing Strategy)

Untuk menjamin waktu tanggap kueri tetap di bawah standar performa NFR ($< 800$ ms untuk statistik dan $< 1.500$ ms untuk inferensi)[cite: 27, 40], indeks berikut dibuat secara spesifik:

1. **`idx_market_stats_location` (Composite Index):**
   * *Tabel:* `market_statistics (city, district)`
   * *Alasan:* Mengoptimalkan kueri filter beranda mobile saat pengguna memilih kota atau kecamatan tertentu[cite: 24].
2. **`idx_prediction_logs_created_at` (B-Tree Index):**
   * *Tabel:* `prediction_logs (created_at DESC)`
   * *Alasan:* Mempercepat penarikan data grafik tren volume inferensi harian pada dashboard admin dan prosedur retensi data[cite: 23].
3. **`idx_ml_models_active` (Partial Unique Index):**
   * *Tabel:* `ml_models (is_active) WHERE is_active = TRUE`
   * *Alasan:* Menjamin bahwa hanya ada **tepat satu model** yang dapat berstatus aktif (`TRUE`) sebagai mesin inferensi produksi pada satu waktu[cite: 1].
4. **`idx_master_locations_lookup`:**
   * *Tabel:* `master_locations (city, is_active)`
   * *Alasan:* Mempercepat pengisian opsi dropdown kota dan kecamatan secara cascading pada form properti mobile[cite: 24].

---

## 4. Skrip DDL PostgreSQL (Schema Migration Script)

Skrip SQL berikut dapat langsung dieksekusi pada basis data PostgreSQL untuk membentuk struktur seluruh tabel, constraint, dan indeks relasional:

```sql
-- Mengaktifkan ekstensi UUID generator
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 1. TABEL: admins
CREATE TABLE admins (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    full_name VARCHAR(100) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 2. TABEL: master_locations
CREATE TABLE master_locations (
    id SERIAL PRIMARY KEY,
    city VARCHAR(50) NOT NULL,
    district VARCHAR(50) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_master_locations_lookup ON master_locations (city, is_active);

-- 3. TABEL: dataset_metadata
CREATE TABLE dataset_metadata (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    total_records INTEGER NOT NULL CHECK (total_records > 0),
    file_size_bytes BIGINT NOT NULL,
    uploaded_by UUID NOT NULL REFERENCES admins(id) ON DELETE RESTRICT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 4. TABEL: ml_models
CREATE TABLE ml_models (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    model_name VARCHAR(100) NOT NULL,
    version VARCHAR(20) NOT NULL,
    model_file_path VARCHAR(500) NOT NULL,
    algorithm_type VARCHAR(50) NOT NULL,
    r2_score NUMERIC(6,4) NOT NULL CHECK (r2_score <= 1.0),
    mae_score NUMERIC(18,2) NOT NULL CHECK (mae_score >= 0),
    rmse_score NUMERIC(18,2) NOT NULL CHECK (rmse_score >= 0),
    is_active BOOLEAN NOT NULL DEFAULT FALSE,
    trained_by UUID NOT NULL REFERENCES admins(id) ON DELETE RESTRICT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
-- Partial index untuk menjamin hanya 1 model yang aktif
CREATE UNIQUE INDEX idx_ml_models_active ON ml_models (is_active) WHERE is_active = TRUE;

-- 5. TABEL: market_statistics
CREATE TABLE market_statistics (
    id SERIAL PRIMARY KEY,
    city VARCHAR(50) NOT NULL,
    district VARCHAR(50),
    avg_price NUMERIC(18,2) NOT NULL CHECK (avg_price >= 0),
    avg_price_per_sqm NUMERIC(18,2) NOT NULL CHECK (avg_price_per_sqm >= 0),
    avg_surface_area NUMERIC(8,2) NOT NULL CHECK (avg_surface_area > 0),
    avg_building_area NUMERIC(8,2) NOT NULL CHECK (avg_building_area > 0),
    total_properties INTEGER NOT NULL CHECK (total_properties > 0),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_market_stats_location ON market_statistics (city, district);

-- 6. TABEL: prediction_logs
CREATE TABLE prediction_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    city VARCHAR(50) NOT NULL,
    district VARCHAR(50) NOT NULL,
    surface_area NUMERIC(8,2) NOT NULL CHECK (surface_area >= 1),
    building_area NUMERIC(8,2) NOT NULL CHECK (building_area >= 1),
    bed INTEGER NOT NULL CHECK (bed >= 0),
    bath INTEGER NOT NULL CHECK (bath >= 0),
    carport INTEGER NOT NULL CHECK (carport >= 0),
    predicted_price NUMERIC(18,2) NOT NULL CHECK (predicted_price >= 0),
    model_version VARCHAR(20) NOT NULL,
    latency_ms INTEGER CHECK (latency_ms >= 0),
    client_ip VARCHAR(45),
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_prediction_logs_created_at ON prediction_logs (created_at DESC);

-- 7. TABEL: system_audit_logs
CREATE TABLE system_audit_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    admin_id UUID NOT NULL REFERENCES admins(id) ON DELETE RESTRICT,
    action_type VARCHAR(50) NOT NULL,
    module_name VARCHAR(50) NOT NULL,
    description TEXT NOT NULL,
    ip_address VARCHAR(45),
    user_agent TEXT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_audit_logs_created_at ON system_audit_logs (created_at DESC);
```