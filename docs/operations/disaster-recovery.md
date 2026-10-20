# Rencana Pemulihan Bencana & Pencadangan (Disaster Recovery Plan) - HousePrice

Dokumen ini mendefinisikan strategi pencadangan data (*backup*), prosedur pemulihan sistem dari kegagalan fatal (*disaster recovery*), target waktu dan titik pemulihan (RTO & RPO), serta mitigasi kehilangan artefak model Machine Learning pada ekosistem HousePrice[cite: 1, 23, 41].

---

## 1. Parameter Pemulihan Sistem (RTO & RPO Objectives)

Untuk menjamin ketersediaan layanan inferensi dan perlindungan data operasional, sistem menetapkan dua target metrik utama[cite: 40]:

* **Recovery Point Objective (RPO) = 24 Jam:**
  Batas toleransi maksimal kehilangan data transaksi atau log inferensi jika terjadi kerusakan total pada media penyimpanan server adalah data dalam 24 jam terakhir.
* **Recovery Time Objective (RTO) = 30 Menit:**
  Waktu maksimal yang dibutuhkan tim teknis untuk memulihkan seluruh infrastruktur (basis data, backend Flask, dan model ML) hingga kembali beroperasi penuh melayani aplikasi mobile setelah insiden terjadi.

---

## 2. Klasifikasi Data & Strategi Pencadangan (Backup Strategy)

Komponen sistem dibagi ke dalam tiga kategori aset dengan perlakuan pencadangan yang berbeda[cite: 1, 41]:

| Kategori Aset | Komponen / Lokasi | Frekuensi Backup | Metode & Perkakas | Lokasi Penyimpanan Cadangan |
| :--- | :--- | :--- | :--- | :--- |
| **Basis Data Relasional** | PostgreSQL (`houseprice_db`: 7 tabel relasional)[cite: 1] | Harian (Setiap pukul 02:00 WIB) | `pg_dump` terkompresi (`.sql.gz`) via Cron/Coolify[cite: 41] | Remote Storage / Cloud Bucket & Direktori Lokal Terpisah |
| **Artefak Model ML** | Berkas serialisasi model (`/app/models/*.pkl`)[cite: 1, 27] | Setiap rilis versi model baru | Git LFS / GitHub Release Assets & Cold Storage[cite: 27, 33] | Repositori `ml-model-testing` & Arsip Server |
| **Dataset Mentah** | Berkas CSV (`/app/uploads/datasets/`)[cite: 1] | Setiap ada unggahan baru | Pengarsipan arsip tarball (`.tar.gz`)[cite: 1] | Repositori dataset tim & Direktori `/backups/` |
| **Konfigurasi Lingkungan** | File `.env` dan Docker Compose[cite: 41] | Setiap ada perubahan konfigurasi | Salinan terenkripsi manual oleh PM | Password Vault / Berkas Rahasia PM |

---

## 3. Otomatisasi Skrip Pencadangan Basis Data (Backup Script)

Skrip shell berikut dijalankan secara otomatis setiap hari menggunakan *cron job* pada server VPS untuk mencadangkan basis data PostgreSQL[cite: 1, 41]:

```bash
#!/usr/bin/env bash
# ==============================================================================
# Skrip Pencadangan Otomatis PostgreSQL - HousePrice
# Simpan di: /opt/scripts/backup_houseprice_db.sh
# ==============================================================================
set -e

BACKUP_DIR="/var/backups/houseprice/db"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
BACKUP_FILE="${BACKUP_DIR}/houseprice_db_${TIMESTAMP}.sql.gz"
RETENTION_DAYS=7

# Buat direktori jika belum tersedia
mkdir -p "${BACKUP_DIR}"

# Jalankan dump database dari dalam container Docker
echo "[$(date)] Memulai pencadangan basis data PostgreSQL..."
docker exec houseprice-postgres pg_dump -U houseprice_admin -d houseprice_db | gzip > "${BACKUP_FILE}"

# Verifikasi integritas berkas cadangan
if [ -s "${BACKUP_FILE}" ]; then
    echo "[$(date)] Cadangan berhasil dibuat: ${BACKUP_FILE} ($(du -h "${BACKUP_FILE}" | cut -f1))"
else
    echo "[$(date)] Galat: Berkas cadangan kosong!" >&2
    exit 1
fi

# Rotasi pembersihan berkas cadangan yang berusia lebih dari 7 hari
echo "[$(date)] Membersihkan berkas cadangan lama (> ${RETENTION_DAYS} hari)..."
find "${BACKUP_DIR}" -type f -name "*.sql.gz" -mtime +${RETENTION_DAYS} -delete

echo "[$(date)] Prosedur pencadangan selesai."
```

---

## 4. Prosedur Pemulihan Bencana (Disaster Recovery Procedures)

### Skenario 1: Kerusakan Kontainer Basis Data (Database Corruption / Data Loss)
Jika tabel terhapus atau volume Docker mengalami korupsi data[cite: 1, 41]:

1. Hentikan container backend untuk mencegah inkonsistensi request baru:
   ```bash
   docker compose stop backend
   ```
2. Bersihkan volume basis data yang rusak dan buat ulang kontainer:
   ```bash
   docker compose down -v
   docker compose up -d postgres
   ```
3. Tunggu hingga status kontainer database siap (*healthy*)[cite: 41]:
   ```bash
   docker compose exec postgres pg_isready -U houseprice_admin -d houseprice_db
   ```
4. Pulihkan data dari berkas cadangan kompresi terbaru:
   ```bash
   gunzip -c /var/backups/houseprice/db/houseprice_db_latest.sql.gz | \
   docker exec -i houseprice-postgres psql -U houseprice_admin -d houseprice_db
   ```
5. Nyalakan kembali container backend dan periksa ketersediaan API:
   ```bash
   docker compose start backend
   curl -i http://localhost:5000/healthz
   ```

### Skenario 2: Kegagalan Total Server VPS (Full Server Outage / Hardware Failure)
Jika server VPS mati total atau penyedia cloud mengalami *outage* permanen:

1. Siapkan mesin VPS Ubuntu baru dan pasang Docker / Coolify sesuai panduan `deployment.md`[cite: 41].
2. Gandakan repositori backend dari GitHub Organization:
   ```bash
   git clone [https://github.com/PBL-House-Price/houseprice-backend-ml.git](https://github.com/PBL-House-Price/houseprice-backend-ml.git)
   cd houseprice-backend-ml
   ```
3. Pulihkan berkas konfigurasi `.env` dari salinan arsip rahasia Project Manager[cite: 41].
4. Gandakan artefak model ML stabil (`rf_house_price_v1.pkl`) ke folder `models/`[cite: 1, 27].
5. Jalankan `docker compose up -d`[cite: 41].
6. Pulihkan basis data dari *off-site backup* terbaru menggunakan langkah pada Skenario 1.
7. Alihkan rekaman DNS Domain (`api.houseprice.id`) ke alamat IP publik VPS baru[cite: 41].
8. Traefik reverse proxy pada Coolify akan otomatis menerbitkan sertifikat SSL Let's Encrypt baru begitu DNS terpropagasi[cite: 41].

### Skenario 3: Kerusakan Berkas Serialisasi Model ML (.pkl Corrupted)
Jika berkas biner model `.pkl` pada server mengalami kegagalan baca saat proses inferensi[cite: 1, 27]:

1. Sistem inferensi akan menangkap galat pemuatan dan mengembalikan kode status `500 Internal Server Error` dengan pesan spesifik[cite: 27].
2. Buka antarmuka tabel `ml_models` untuk melihat metadata model versi sebelumnya yang stabil[cite: 1].
3. Unduh berkas artefak model cadangan dari repositori `ml-model-testing` (folder `/releases/`)[cite: 33]:
   ```bash
   curl -L -o /app/models/rf_house_price_v1.pkl [https://github.com/PBL-House-Price/ml-model-testing/raw/main/models/rf_house_price_v1.pkl](https://github.com/PBL-House-Price/ml-model-testing/raw/main/models/rf_house_price_v1.pkl)
   ```
4. Ubah flag `is_active` di tabel `ml_models` ke versi cadangan yang valid[cite: 1].
5. Muat ulang worker Flask backend (`docker compose restart backend`)[cite: 41].

---

## 5. Pengujian & Simulasi Rutin Pemulihan (Drill & Testing)

Rencana pemulihan bencana wajib diuji secara praktis minimal satu kali sebelum pelaksanaan evaluasi akhir (Minggu 14)[cite: 8, 28]:

* **Simulasi Uji:** Menjalankan pemulihan database dari berkas dump di komputer pengembang lokal untuk memastikan skrip `pg_dump` dan `psql` tidak menghasilkan galat sintaks atau pelanggaran constraint foreign key[cite: 1].
* **Verifikasi Integritas Data:** Memastikan seluruh data referensi kota (`master_locations`), metrik model (`ml_models`), dan log audit (`system_audit_logs`) kembali persis seperti kondisi sebelum insiden[cite: 1].