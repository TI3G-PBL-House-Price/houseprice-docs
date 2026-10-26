# Model & Skema Basis Data (Data Model & Schema) - HousePrice

Dokumen ini memuat spesifikasi arsitektur data, perancangan skema konseptual dan fisik, kamus data komprehensif, relasi antar-entitas, strategi pengindeksan, serta skrip DDL SQL untuk basis data **PostgreSQL** pada sistem HousePrice.

---

## 1. Diagram Relasi Entitas (Entity-Relationship Model)

Basis data HousePrice terdiri dari **9 tabel relasional** yang terbagi ke dalam tiga domain utama:
1. **Domain Administratif & Artefak ML:** Berelasi secara referensial untuk mengelola data latih, riwayat model regresi, dan audit operasional internal.
2. **Domain Pengguna & Notifikasi Mobile (Strict Gate):** Mengelola autentikasi pengguna aplikasi mobile, token sesi, preferensi perangkat, pemberitahuan pasar, serta kepemilikan riwayat kalkulasi.
3. **Domain Referensi & Telemetri Analitik:** Menyediakan data master wilayah legal Jakarta, agregasi statistik pasar beranda, dan pencatatan komputasi inferensi model berlatensi rendah.

```
                      DIAGRAM RELASI BASIS DATA (ERD)

  [ DOMAIN ADMINISTRATIF & ML ]           [ DOMAIN PENGGUNA MOBILE (STRICT GATE) ]

+-----------------------------+          +-----------------------------+
|           admins            |          |            users            |
+-----------------------------+          +-----------------------------+
| PK  id            : UUID    |          | PK  id            : UUID    |
|     username      : STR(50) |          |     full_name     : STR(100)|
|     email         : STR(100)|          |     email         : STR(100)|
|     password_hash : STR(255)|          |     password_hash : STR(255)|
|     full_name     : STR(100)|          |     phone_number  : STR(20) |
|     is_active     : BOOL    |          |     fcm_token     : TEXT    |
|     created_at    : TIME    |          |     is_active     : BOOL    |
|     updated_at    : TIME    |          |     created_at    : TIME    |
+-----------------------------+          |     updated_at    : TIME    |
   | 1         | 1         | 1           +-----------------------------+
   |           |           |                | 1                   | 1
   | N         | N         | N              |                     |
   v           v           v                | N                   | N
+---------+ +---------+ +---------+         v                     v
| dataset | |   ml    | | system  |    +---------------+    +---------------+
|metadata | | models  | |audit_log|    | notifications |    |prediction_logs|
+---------+ +---------+ +---------+    +---------------+    +---------------+
|PK id    | |PK id    | |PK id    |    |PK id    : UUID|    |PK id    : UUID|
|   file  | |   model | |FK admin |    |FK user_id:UUID|    |FK user_id:UUID|
|FK upload| |FK train | |   action|    |   title : STR |    |   city  : STR |
|   ...   | |   ...   | |   ...   |    |   is_read:BOOL|    |   price : NUM |
+---------+ +---------+ +---------+    +---------------+    +---------------+
                                                                    ^
  [ DOMAIN REFERENSI & TELEMETRI ]                                  |
                                                                    |
+-----------------------------+          +--------------------------+--+
|      master_locations       |          |      market_statistics      |
+-----------------------------+          +-----------------------------+
| PK  id            : SERIAL  |          | PK  id            : SERIAL  |
|     city          : STR(50) |          |     city          : STR(50) |
|     district      : STR(50) |          |     district      : STR(50) |
|     is_active     : BOOL    |          |     avg_price     : NUM     |
|     created_at    : TIME    |          |     avg_price_sqm : NUM     |
+-----------------------------+          |     updated_at    : TIME    |
                                         +-----------------------------+
```

---

## 2. Kamus Data Lengkap (Data Dictionary)

### 2.1 Tabel `admins`
Menyimpan kredensial dan profil administrator pengelola sistem dashboard web.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik global entitas admin | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11` |
| `username` | `VARCHAR(50)` | `NOT NULL`, `UNIQUE` | Nama unik untuk login dashboard web | `admin_hanif` |
| `email` | `VARCHAR(100)` | `NOT NULL`, `UNIQUE`, Format Email | Alamat surat elektronik terdaftar | `hanif@houseprice.id` |
| `password_hash` | `VARCHAR(255)` | `NOT NULL` | Hash sandi terenkripsi algoritma Bcrypt | `$2b$12$e8m4...` |
| `full_name` | `VARCHAR(100)` | `NOT NULL` | Nama lengkap personal administrator | `Abdul Rahman Hanif` |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `TRUE` | Status hak akses aktif/nonaktif akun | `true` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel pembuatan akun admin | `2026-10-01 08:30:00+07` |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel pembaruan data profil | `2026-10-01 08:30:00+07` |

### 2.2 Tabel `users`
Menyimpan data identitas, kredensial autentikasi *Strict Gate*, dan sesi akun pengguna mobile.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik global akun pengguna mobile | `f1bbcd22-4e1b-4df8-bb6d-9aa1cd621a01` |
| `full_name` | `VARCHAR(100)` | `NOT NULL` | Nama lengkap pemilik akun pengguna | `Budi Santoso` |
| `email` | `VARCHAR(100)` | `NOT NULL`, `UNIQUE`, Format Email | Alamat email aktif untuk login mobile | `budi.santoso@gmail.com` |
| `password_hash` | `VARCHAR(255)` | `NOT NULL` | Hash sandi terenkripsi Bcrypt minimal 12 salt rounds | `$2b$12$7k9X...` |
| `phone_number` | `VARCHAR(20)` | `NULLABLE` | Nomor telepon atau WhatsApp kontak pengguna | `081234567890` |
| `fcm_token` | `TEXT` | `NULLABLE` | Token registrasi perangkat untuk push notification | `fM7sK_d92...` |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `TRUE` | Status keaktifan akun pengguna | `true` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel registrasi akun | `2026-10-26 10:00:00+07` |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel perubahan profil atau sandi | `2026-10-26 10:00:00+07` |

### 2.3 Tabel `notifications`
Menyimpan pesan notifikasi in-app, pengumuman tren properti, dan pembaruan sistem per pengguna.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik catatan notifikasi | `a2ccdb33-5e2c-4ef8-bb6d-0bb2ce732b12` |
| `user_id` | `UUID` | `NOT NULL`, `REFERENCES users(id) ON DELETE CASCADE` | Relasi akun pengguna penerima notifikasi | `f1bbcd22-4e1b-4df8-bb6d-9aa1cd621a01` |
| `title` | `VARCHAR(150)` | `NOT NULL` | Judul pesan notifikasi | `Tren Harga Jakarta Barat Naik` |
| `message` | `TEXT` | `NOT NULL` | Konten penjelasan detail pemberitahuan | `Harga properti Cengkareng naik 3.2% bulan ini.` |
| `type` | `VARCHAR(50)` | `NOT NULL`, Default `'SYSTEM'` | Kategori pesan (`SYSTEM`, `MARKET_TREND`, `PRICE_ALERT`) | `MARKET_TREND` |
| `is_read` | `BOOLEAN` | `NOT NULL`, Default `FALSE` | Status pesan telah dibaca oleh pengguna | `false` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel notifikasi diterbitkan | `2026-10-26 11:30:00+07` |

### 2.4 Tabel `master_locations`
Menyimpan referensi wilayah administratif legal di DKI Jakarta untuk sinkronisasi dropdown form mobile.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY`, Autoincrement | ID sekuensial wilayah | `1` |
| `city` | `VARCHAR(50)` | `NOT NULL` | Nama kota administrasi di DKI Jakarta | `Jakarta Barat` |
| `district` | `VARCHAR(50)` | `NOT NULL` | Nama kecamatan atau kawasan terdaftar | `Citra Garden` |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `TRUE` | Status ketersediaan wilayah pada form mobile | `true` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu data referensi ditambahkan | `2026-10-01 09:00:00+07` |

### 2.5 Tabel `dataset_metadata`
Mencatat riwayat berkas dataset CSV properti yang diunggah melalui Web Admin.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik catatan dataset | `b1ffcd88-8d1c-4ef8-bb6d-7cc0ce491b22` |
| `file_name` | `VARCHAR(255)` | `NOT NULL` | Nama asli berkas CSV fisik | `jakarta_house_v1.csv` |
| `file_path` | `VARCHAR(500)` | `NOT NULL` | Path direktori penyimpanan berkas di server | `/uploads/datasets/jakarta_house_v1.csv` |
| `total_records` | `INTEGER` | `NOT NULL`, Check `(total_records > 0)` | Total baris data properti di dalam berkas | `10000` |
| `file_size_bytes`| `BIGINT` | `NOT NULL` | Ukuran berkas CSV dalam satuan byte | `845620` |
| `uploaded_by` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Relasi ke akun admin pengunggah berkas | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel unggah berkas | `2026-10-02 09:15:00+07` |

### 2.6 Tabel `ml_models`
Menyimpan registri model regresi Machine Learning, metrik evaluasi, dan status versi produksi aktif.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik artefak model ML | `c2eedf77-7e2b-4ef8-bb6d-8dd1df502c33` |
| `model_name` | `VARCHAR(100)` | `NOT NULL` | Nama representatif algoritma model | `Random Forest Regressor` |
| `version` | `VARCHAR(20)` | `NOT NULL` | Penomoran versi model | `v1.0.0` |
| `model_file_path`| `VARCHAR(500)`| `NOT NULL` | Lokasi berkas serialisasi biner `.pkl` | `/models/rf_house_price_v1.pkl` |
| `algorithm_type`| `VARCHAR(50)` | `NOT NULL` | Golongan algoritma (`Ensemble` atau `Linear`) | `Ensemble` |
| `r2_score` | `NUMERIC(6,4)` | `NOT NULL`, Check `(r2_score <= 1.0)` | Skor koefisien determinasi evaluasi uji | `0.8272` |
| `mae_score` | `NUMERIC(18,2)`| `NOT NULL`, Check `(mae_score >= 0)` | Nilai Mean Absolute Error dalam Rupiah | `1063034875.00` |
| `rmse_score` | `NUMERIC(18,2)`| `NOT NULL`, Check `(rmse_score >= 0)`| Nilai Root Mean Squared Error dalam Rupiah | `1808105786.00` |
| `is_active` | `BOOLEAN` | `NOT NULL`, Default `FALSE` | Penanda model utama inferensi API | `true` |
| `trained_by` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Relasi admin yang melatih model | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu pelatihan model selesai | `2026-10-02 10:45:00+07` |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu status aktif/nonaktif diperbarui | `2026-10-02 10:45:00+07` |

### 2.7 Tabel `market_statistics`
Menyimpan agregasi rata-rata harga dan luas per wilayah untuk kebutuhan tampilan Beranda Mobile.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `SERIAL` | `PRIMARY KEY`, Autoincrement | ID sekuensial statistik pasar | `1` |
| `city` | `VARCHAR(50)` | `NOT NULL` | Nama kota administrasi wilayah data | `Jakarta Barat` |
| `district` | `VARCHAR(50)` | `NULLABLE` | Nama kecamatan (`NULL` jika rata-rata kota) | `Citra Garden` |
| `avg_price` | `NUMERIC(18,2)`| `NOT NULL`, Check `(avg_price >= 0)` | Rata-rata nominal harga rumah (Rp) | `2450000000.00` |
| `avg_price_per_sqm`| `NUMERIC(18,2)`| `NOT NULL`, Check `(avg_price_per_sqm >= 0)` | Rata-rata harga per meter persegi ($m^2$) | `18500000.00` |
| `avg_surface_area`| `NUMERIC(8,2)` | `NOT NULL`, Check `(avg_surface_area > 0)` | Rata-rata luas tanah properti ($m^2$) | `125.50` |
| `avg_building_area`| `NUMERIC(8,2)`| `NOT NULL`, Check `(avg_building_area > 0)` | Rata-rata luas bangunan properti ($m^2$) | `140.00` |
| `total_properties`| `INTEGER` | `NOT NULL`, Check `(total_properties > 0)` | Volume sampel properti dalam kalkulasi | `450` |
| `updated_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel agregasi diperbarui | `2026-10-03 14:00:00+07` |

### 2.8 Tabel `prediction_logs`
Mencatat telemetri permintaan inferensi prediksi harga yang terikat pada akun pemohon.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal transaksi prediksi | `d3ffea66-6f3a-4ef8-bb6d-9ee2ea613d44` |
| `user_id` | `UUID` | `NULLABLE`, `REFERENCES users(id) ON DELETE SET NULL` | Relasi akun pengguna mobile yang meminta prediksi | `f1bbcd22-4e1b-4df8-bb6d-9aa1cd621a01` |
| `city` | `VARCHAR(50)` | `NOT NULL` | Masukan nama kota administrasi | `Jakarta Barat` |
| `district` | `VARCHAR(50)` | `NOT NULL` | Masukan nama kecamatan properti | `Citra Garden` |
| `surface_area` | `NUMERIC(8,2)` | `NOT NULL`, Check `(surface_area >= 1)` | Masukan luas tanah properti ($m^2$) | `100.00` |
| `building_area`| `NUMERIC(8,2)` | `NOT NULL`, Check `(building_area >= 1)` | Masukan luas bangunan properti ($m^2$) | `120.00` |
| `bed` | `INTEGER` | `NOT NULL`, Check `(bed >= 0)` | Masukan jumlah kamar tidur | `3` |
| `bath` | `INTEGER` | `NOT NULL`, Check `(bath >= 0)` | Masukan jumlah fasilitas kamar mandi | `2` |
| `carport` | `INTEGER` | `NOT NULL`, Check `(carport >= 0)` | Masukan kapasitas ruang parkir mobil | `1` |
| `predicted_price`| `NUMERIC(18,2)`| `NOT NULL`, Check `(predicted_price >= 0)` | Estimasi nominal harga dari model ML | `2150000000.00` |
| `model_version`| `VARCHAR(20)` | `NOT NULL` | Versi model ML yang mengeksekusi | `v1.0.0` |
| `latency_ms` | `INTEGER` | `NULLABLE`, Check `(latency_ms >= 0)` | Waktu komputasi server dalam milidetik | `120` |
| `client_ip` | `VARCHAR(45)` | `NULLABLE` | Alamat IP pemohon | `180.252.164.21` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel permintaan inferensi | `2026-10-26 14:15:00+07` |

### 2.9 Tabel `system_audit_logs`
Merekam jejak operasional administrator untuk kepatuhan akuntabilitas sistem.

| Nama Kolom | Tipe Data | Constraint / Validasi | Deskripsi & Tujuan | Contoh Data |
| :--- | :--- | :--- | :--- | :--- |
| `id` | `UUID` | `PRIMARY KEY`, Default `gen_random_uuid()` | Pengenal unik entri catatan audit | `e4aaeb55-5f2b-4ef8-bb6d-0ff3fa724e55` |
| `admin_id` | `UUID` | `NOT NULL`, `REFERENCES admins(id)` | Akun admin yang mengeksekusi operasi | `a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11` |
| `action_type` | `VARCHAR(50)` | `NOT NULL` | Jenis aksi (`UPLOAD_DATASET`, `RETRAIN_MODEL`, `SWITCH_MODEL`) | `SWITCH_MODEL` |
| `module_name` | `VARCHAR(50)` | `NOT NULL` | Modul yang ditargetkan | `ML_REGISTRY` |
| `description` | `TEXT` | `NOT NULL` | Penjelasan tekstual detail kegiatan | `Mengaktifkan model RF v1.0.0` |
| `ip_address` | `VARCHAR(45)` | `NULLABLE` | Alamat IP perangkat pengelola | `192.168.1.15` |
| `user_agent` | `TEXT` | `NULLABLE` | Data identitas peramban perangkat admin | `Mozilla/5.0...` |
| `created_at` | `TIMESTAMPTZ` | `NOT NULL`, Default `CURRENT_TIMESTAMP` | Waktu stempel presisi operasi audit | `2026-10-04 14:10:00+07` |

---

## 3. Strategi Pengindeksan Basis Data (Indexing Strategy)

Untuk menjamin waktu tanggap kueri tetap di bawah standar performa NFR ($< 800$ ms untuk analitik dan $< 1.500$ ms untuk inferensi), indeks berikut dikonfigurasikan secara spesifik:

1. **`idx_users_email` (B-Tree Unique Index):**
   * *Tabel:* `users (email)`
   * *Alasan:* Menjamin kecepatan kueri verifikasi kredensial saat proses login pengguna mobile.
2. **`idx_prediction_logs_user_id` (B-Tree Index):**
   * *Tabel:* `prediction_logs (user_id, created_at DESC)`
   * *Alasan:* Mengoptimalkan pemanggilan riwayat prediksi properti pada menu Profil pengguna tanpa membebani tabel telemetri secara keseluruhan.
3. **`idx_notifications_user_lookup` (Composite B-Tree Index):**
   * *Tabel:* `notifications (user_id, is_read, created_at DESC)`
   * *Alasan:* Mempercepat kueri pengambilan daftar notifikasi aktif dan penghitungan jumlah pesan yang belum dibaca (*unread counter badge*).
4. **`idx_market_stats_location` (Composite Index):**
   * *Tabel:* `market_statistics (city, district)`
   * *Alasan:* Mengoptimalkan kueri filter beranda mobile saat pengguna memilih kota atau kecamatan tertentu.
5. **`idx_prediction_logs_created_at` (B-Tree Index):**
   * *Tabel:* `prediction_logs (created_at DESC)`
   * *Alasan:* Mempercepat penarikan data grafik tren volume inferensi harian pada dashboard admin.
6. **`idx_ml_models_active` (Partial Unique Index):**
   * *Tabel:* `ml_models (is_active) WHERE is_active = TRUE`
   * *Alasan:* Menjamin bahwa hanya ada tepat satu model yang berstatus aktif sebagai mesin inferensi produksi pada satu waktu.
7. **`idx_master_locations_lookup`:**
   * *Tabel:* `master_locations (city, is_active)`
   * *Alasan:* Mempercepat pengisian opsi dropdown kota dan kecamatan secara cascading pada form properti mobile.

---

## 4. Skrip DDL PostgreSQL (Schema Migration Script)

Skrip SQL berikut dapat langsung dieksekusi pada basis data PostgreSQL untuk membentuk struktur seluruh 9 tabel, constraint, foreign key, dan indeks relasional:

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

-- 2. TABEL: users (Mobile Strict Gate Authentication)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    phone_number VARCHAR(20),
    fcm_token TEXT,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE UNIQUE INDEX idx_users_email ON users (email);

-- 3. TABEL: notifications
CREATE TABLE notifications (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(150) NOT NULL,
    message TEXT NOT NULL,
    type VARCHAR(50) NOT NULL DEFAULT 'SYSTEM',
    is_read BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_notifications_user_lookup ON notifications (user_id, is_read, created_at DESC);

-- 4. TABEL: master_locations
CREATE TABLE master_locations (
    id SERIAL PRIMARY KEY,
    city VARCHAR(50) NOT NULL,
    district VARCHAR(50) NOT NULL,
    is_active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX idx_master_locations_lookup ON master_locations (city, is_active);

-- 5. TABEL: dataset_metadata
CREATE TABLE dataset_metadata (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(500) NOT NULL,
    total_records INTEGER NOT NULL CHECK (total_records > 0),
    file_size_bytes BIGINT NOT NULL,
    uploaded_by UUID NOT NULL REFERENCES admins(id) ON DELETE RESTRICT,
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- 6. TABEL: ml_models
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
CREATE UNIQUE INDEX idx_ml_models_active ON ml_models (is_active) WHERE is_active = TRUE;

-- 7. TABEL: market_statistics
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

-- 8. TABEL: prediction_logs
CREATE TABLE prediction_logs (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    user_id UUID REFERENCES users(id) ON DELETE SET NULL,
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
CREATE INDEX idx_prediction_logs_user_id ON prediction_logs (user_id, created_at DESC);

-- 9. TABEL: system_audit_logs
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