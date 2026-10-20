# ADR 0001: Pemilihan PostgreSQL sebagai Basis Data Relasional Cloud

- **Status:** Diterima (Accepted)
- **Tanggal:** 2026-10-05
- **Pengambil Keputusan:** Abdul Rahman Hanif Darmawan (Project Manager), Tim Pengembang HousePrice
- **Konsultasi:** Habibie Ed Dien, S.Kom., M.T., Usman Nurhasan, S.Kom., M.T., M. Hasyim Ratsanjani, S.Kom., M.Kom.

---

## 1. Konteks & Latar Belakang Masalah

Sistem HousePrice membutuhkan penyimpanan basis data terpusat di lingkungan cloud untuk mengelola akun administrator, metadata berkas dataset, registri versi model machine learning, master lokasi wilayah DKI Jakarta, agregasi statistik pasar, serta telemetri pencatatan log inferensi dan audit sistem[cite: 1, 23].

Kebutuhan teknis spesifik mencakup:
* Penanganan nilai nominal finansial properti berkapasitas besar (mencapai ratusan miliar Rupiah) dengan presisi desimal mutlak tanpa galat pembulatan floating-point[cite: 1, 27].
* Dukungan tipe data identitas unik global (*Universally Unique Identifier* / UUID) secara native untuk kunci primer (*Primary Key*) pada tabel transaksi dan metadata[cite: 1].
* Kemampuan pembuatan indeks parsial bersyarat (*Partial Indexing*) untuk menjamin integritas logika bisnis tingkat basis data (hanya boleh ada tepat satu model aktif)[cite: 1].
* Jaminan kepatuhan ACID (*Atomicity, Consistency, Isolation, Durability*) untuk pencatatan jejak audit administratif yang tidak dapat disangkal[cite: 1, 23].
* Kemudahan orkestrasi kontainer open-source pada platform Coolify dan Docker Compose[cite: 41].

---

## 2. Alternatif Basis Data yang Dipertimbangkan

1. **PostgreSQL 15+ (Terpilih)**
2. **MySQL 8.0**
3. **MongoDB 6.0+ (NoSQL Document Store)**
4. **SQLite 3 (Khusus Cloud Server)**

---

## 3. Matriks Evaluasi Perbandingan

| Kriteria Evaluasi | PostgreSQL 15+ | MySQL 8.0 | MongoDB 6.0+ | SQLite 3 (Cloud) |
| :--- | :--- | :--- | :--- | :--- |
| **Integritas Relasional & FK** | Sangat Kuat (ACID penuh)[cite: 1] | Sangat Kuat (InnoDB)[cite: 1] | Lemah (Manual / App level) | Kuat namun terbatas |
| **Dukungan Native Tipe UUID** | Native 128-bit (`UUID`)[cite: 1] | Menggunakan `BINARY(16)` atau string | Menggunakan `ObjectId` / String BSON | Disimpan sebagai `TEXT` |
| **Presisi Finansial Rupiah** | Native `NUMERIC(18,2)` presisi tinggi[cite: 1] | `DECIMAL(18,2)` | BSON Decimal128 (Raw) | `REAL` / IEEE 754 (Rentan galat) |
| **Fitur Partial Indexing** | Didukung penuh (`WHERE condition`)[cite: 1] | Terbatas / Tidak langsung | Partial index didukung | Didukung |
| **Konkurensi & Latensi Inferensi** | MVCC optimal untuk pembacaan cepat | Baik | Sangat Cepat untuk write bebas skema | Kunci tingkat berkas (*locking issue*) |
| **Dukungan Docker & Coolify** | Terintegrasi 1-klik di Coolify[cite: 41] | Didukung kontainer | Didukung kontainer | Berbasis berkas lokal |

---

## 4. Keputusan & Alasan Pemilihan (Rationale)

Tim memutuskan untuk menggunakan **PostgreSQL 15+** sebagai *Single Source of Truth* penyimpanan basis data cloud backend dengan pertimbangan teknis berikut[cite: 1, 23]:

1. **Dukungan Native Tipe UUID:** 5 dari 7 tabel skema sistem (`admins`, `dataset_metadata`, `ml_models`, `prediction_logs`, dan `system_audit_logs`) menggunakan tipe data UUID sebagai Primary Key[cite: 1]. PostgreSQL mendukung tipe data `UUID` secara native (16 byte hemat penyimpanan) bersama ekstensi `uuid-ossp` (`uuid_generate_v4()`)[cite: 1].
2. **Kebutuhan Indeks Parsial Unik (Conditional Unique Index):** Pada tabel `ml_models`, sistem mengharuskan bahwa hanya boleh ada tepat 1 baris model yang berstatus aktif (`is_active = TRUE`) untuk melayani inferensi API[cite: 1]. PostgreSQL memungkinkan penegakan aturan bisnis ini langsung pada tingkat mesin basis data:
   ```sql
   CREATE UNIQUE INDEX idx_ml_models_active ON ml_models (is_active) WHERE is_active = TRUE;
   ```
   Aturan ini mencegah inkonsistensi status model ganda secara deterministik tanpa membebani logika aplikasi Flask[cite: 1].
3. **Akurasi Tipe Data Numerik Finansial:** Kolom harga properti dan metrik galat dievaluasi menggunakan `NUMERIC(18,2)` yang menjamin ketepatan nilai nominal rupiah hingga batas triliun tanpa risiko *floating point precision loss*[cite: 1].
4. **Pemisahan Peran dengan SQLite:** SQLite tetap digunakan pada proyek ini, namun difokuskan secara eksklusif pada **klien mobile Flutter** untuk penyimpanan riwayat pencarian luring (*offline-first cache*)[cite: 25, 31]. Menggunakan SQLite pada backend cloud ditolak karena risiko *table locking* saat melayani banyak permintaan inferensi paralel dari aplikasi mobile[cite: 27, 40].
5. **Penolakan MongoDB:** Format dataset properti bersifat tabular terstruktur ketat (`jakarta_house.csv`)[cite: 27] dan membutuhkan integritas referensial antar-tabel audit (`admins` $\rightarrow$ `dataset_metadata`, `admins` $\rightarrow$ `ml_models`), sehingga penggunaan basis data dokumen NoSQL tidak relevan dan meningkatkan kompleksitas validasi aplikasi[cite: 1, 23].

---

## 5. Konsekuensi Positif & Mitigasi Risiko

### Konsekuensi Positif (Benefits)
* Integritas data operasional, registri model, dan log inferensi terjamin penuh oleh kepatuhan ACID[cite: 1, 23].
* Kemudahan integrasi dengan ORM SQLAlchemy dan driver `psycopg2-binary` pada ekosistem Python Flask[cite: 23, 41].
* Kemudahan migrasi dan pencadangan terotomatisasi melalui utilitas kontainer Docker (`pg_dump`)[cite: 41].

### Risiko & Mitigasi (Risks & Mitigations)
* **Konsumsi Memori RAM:** PostgreSQL membutuhkan alokasi RAM yang lebih besar dibandingkan SQLite.
  * *Mitigasi:* Pengaturan alokasi buffer memori (`shared_buffers = 128MB`, `work_mem = 4MB`) pada berkas konfigurasi PostgreSQL di Docker Compose disesuaikan agar berjalan optimal pada server VPS berkapasitas 1–2 GB RAM[cite: 41].
* **Akumulasi Log Inferensi:** Tabel `prediction_logs` dapat membesar seiring banyaknya pengujian dan permintaan aplikasi[cite: 1].
  * *Mitigasi:* Pembuatan indeks B-Tree pada kolom `created_at` serta penyiapan prosedur pembersihan berkala (*data retention job*) untuk log yang berusia lebih dari 90 hari[cite: 1].