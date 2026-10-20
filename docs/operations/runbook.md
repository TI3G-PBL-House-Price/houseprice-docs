# Panduan Operasional & Penanganan Insiden (Operations Runbook) - HousePrice

Dokumen ini memuat prosedur standar operasional (*Standard Operating Procedures* / SOP), verifikasi kesehatan sistem (*health checks*), panduan penanganan gangguan (*incident playbooks*), serta pemeliharaan berkala untuk ekosistem produksi HousePrice (Coolify PaaS, Docker Compose, Flask Backend, PostgreSQL, dan Model ML)[cite: 1, 23, 41].

---

## 1. Tim Tanggap Insiden & Matriks Eskalasi

Jika terjadi kegagalan operasional pada lingkungan *staging* atau produksi, eskalasi dilakukan secara terstruktur sesuai domain tanggung jawab anggota tim[cite: 19, 28, 29]:

| Tingkat Keparahan | Dampak Insiden | Target Respons | Penanggung Jawab Utama | Kontak & Eskalasi |
| :--- | :--- | :--- | :--- | :--- |
| **P1 - Kritis (Blocker)** | Server inferensi mati total, basis data tidak dapat diakses, atau demonstrasi live terhenti[cite: 27, 40]. | $< 15$ menit | Abdul Rahman Hanif (PM) & Nuril Huda (ML/Backend)[cite: 19, 28, 29] | Eksekusi cabang `hotfix/*` langsung ke `main`[cite: 37]. |
| **P2 - Mayor (Degraded)** | Latensi inferensi $> 1.500$ ms atau sinkronisasi filter kecamatan gagal dimuat[cite: 24, 27]. | $< 1$ jam | Nuril Huda (ML) & Hanif (API QA)[cite: 19, 28, 29] | Investigasi alokasi RAM container & restart worker Gunicorn[cite: 41]. |
| **P3 - Minor (Low Impact)** | Galat kalkulasi simulasi KPR atau tampilan teks terpotong pada perangkat tipe tertentu[cite: 25, 29]. | $< 4$ jam | Naufal Abid (Logic) & Jasmine Nasywa (UI)[cite: 19, 29] | Dicatat pada Defect Tracking Log untuk sprint berjalan[cite: 19, 29]. |

---

## 2. Pemeriksaan Kesehatan Rutin (Health Checks)

Sebelum sesi pengujian dosen atau demonstrasi rilis, jalankan prosedur verifikasi cepat (*sanity check*)[cite: 8, 28]:

### A. Endpoint Pemeriksaan Kesehatan Sistem
Kueri endpoint kesehatan server untuk memastikan Flask dan koneksi basis data aktif[cite: 23, 40]:
```bash
curl -i [https://api.houseprice.id/healthz](https://api.houseprice.id/healthz)
```
*Respons normal yang diharapkan (HTTP 200 OK):*
```json
{
  "status": "healthy",
  "database": "connected",
  "active_model": "Random Forest Regressor v1.0.0",
  "timestamp": "2026-10-20T11:00:00Z"
}
```

### B. Uji Inferensi Langsung (Smoke Test Inference)
Kirimkan satu payload sampel untuk menguji pipeline preprocessing dan model `.pkl`[cite: 24, 27]:
```bash
curl -X POST [https://api.houseprice.id/api/v1/predict](https://api.houseprice.id/api/v1/predict) \
  -H "Content-Type: application/json" \
  -d '{
    "city": "Jakarta Barat",
    "district": "Citra Garden",
    "surface_area": 100,
    "building_area": 120,
    "bed": 3,
    "bath": 2,
    "carport": 1
  }'
```
*Indikator lolos:* HTTP 200 OK, nilai `predicted_price` berupa angka nominal positif, dan `latency_ms` $< 1.500$ ms[cite: 1, 27].

---

## 3. Panduan Penanganan Gangguan (Incident Playbooks)

### Skenario A: Latensi Inferensi Melebihi Batas (> 1.500 ms) atau Timeout
* **Gejala:** Aplikasi mobile menampilkan pesan putus koneksi (*Request Timeout*) atau spinner loading berputar lebih dari 2 detik[cite: 25, 27].
* **Akar Masalah Potensial:** Terjadi *memory leak* pada proses Flask, proses Gunicorn kekurangan worker thread, atau berkas model `.pkl` dimuat ulang pada setiap request[cite: 27, 40].
* **Langkah Pemulihan:**
  1. Masuk ke terminal VPS via SSH atau buka antarmuka terminal Coolify[cite: 41].
  2. Periksa status penggunaan resource container:
     ```bash
     docker stats houseprice-backend-1
     ```
  3. Periksa log aplikasi untuk mencari bottleneck:
     ```bash
     docker logs --tail 100 houseprice-backend-1 | grep "WARNING"
     ```
  4. Lakukan *graceful reload* pada worker Gunicorn tanpa mematikan basis data:
     ```bash
     docker compose restart backend
     ```

### Skenario B: Basis Data PostgreSQL Mengalami Crash / Connection Refused
* **Gejala:** Endpoint API mengembalikan HTTP 500 dengan pesan `psycopg2.OperationalError: could not connect to server`[cite: 23, 40].
* **Langkah Pemulihan:**
  1. Periksa ketersediaan container database:
     ```bash
     docker ps -a | grep postgres
     ```
  2. Jika statusnya `Exited`, periksa penyebab matinya kontainer:
     ```bash
     docker logs --tail 50 houseprice-postgres-1
     ```
  3. Nyalakan kembali kontainer database dan verifikasi integritas persistent volume[cite: 41]:
     ```bash
     docker compose up -d postgres
     ```
  4. Pastikan port internal 5432 menerima koneksi dari kontainer Flask[cite: 41]:
     ```bash
     docker compose exec backend python -c "import psycopg2; print('DB Connected')"
     ```

### Skenario C: Berkas Model `.pkl` Tidak Ditemukan atau Rusak (Model Load Failure)
* **Gejala:** Backend menyala tetapi endpoint inferensi gagal dengan galat `FileNotFoundError: models/rf_house_price_v1.pkl`[cite: 1, 27].
* **Langkah Pemulihan:**
  1. Periksa ketersediaan berkas fisik model pada direktori kontainer:
     ```bash
     docker compose exec backend ls -lh /app/models/
     ```
  2. Jika berkas hilang atau berukuran 0 byte, salin model stabil terakhir dari artefak rilis `ml-model-testing` atau repositori git[cite: 27, 33]:
     ```bash
     docker cp ./backup_models/rf_house_price_v1.pkl houseprice-backend-1:/app/models/
     ```
  3. Pastikan kolom `is_active` pada tabel `ml_models` mengarah ke nama file yang ada secara fisik[cite: 1].
  4. Muat ulang aplikasi backend[cite: 41]:
     ```bash
     docker compose restart backend
     ```

### Skenario D: Galat Sertifikat SSL / HTTPS (Let's Encrypt Expired)
* **Gejala:** Aplikasi Android menolak menghubungkan diri ke server karena peringatan `CERTIFICATE_VERIFY_FAILED`[cite: 40].
* **Langkah Pemulihan:**
  1. Buka antarmuka web Coolify Dashboard di browser[cite: 41].
  2. Masuk ke menu proyek **HousePrice Backend** $\rightarrow$ **Settings** $\rightarrow$ **Domains**[cite: 41].
  3. Klik tombol **Force Renew SSL Certificate**[cite: 41].
  4. Traefik reverse proxy akan meminta sertifikat baru ke Let's Encrypt CA dan memasangnya seketika dalam kurun waktu 30 detik tanpa downtime[cite: 41].

---

## 4. Prosedur Operasional Rutin & Pemeliharaan

### A. Prosedur Rollback Versi Model ML (Emergency Model Switch)
Jika model baru yang diaktifkan admin ternyata memberikan estimasi yang bias atau tidak wajar di lapangan, lakukan rollback instan melalui basis data PostgreSQL[cite: 1, 23]:

```sql
-- 1. Matikan model bermasalah saat ini
UPDATE ml_models SET is_active = FALSE WHERE version = 'v1.1.0';

-- 2. Aktifkan kembali model versi stabil sebelumnya (v1.0.0)
UPDATE ml_models SET is_active = TRUE WHERE version = 'v1.0.0';

-- 3. Catat aksi darurat ini ke audit log sistem
INSERT INTO system_audit_logs (id, admin_id, action_type, module_name, description)
VALUES (
    uuid_generate_v4(),
    'a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11',
    'SWITCH_MODEL',
    'ML_REGISTRY',
    'Rollback darurat ke model v1.0.0 karena anomali estimasi'
);
```
Setelah kueri berhasil dieksekusi, restart kontainer backend agar file model biner yang baru dimuat ke memori[cite: 1, 41].

### B. Pembersihan & Pengarsipan Log Inferensi Berkala
Jalankan kueri pemeliharaan retensi log pada akhir pekan agar ukuran penyimpanan basis data PostgreSQL tetap ramping[cite: 1]:
```sql
-- Menghapus log inferensi yang telah berusia lebih dari 90 hari
DELETE FROM prediction_logs 
WHERE created_at < NOW() - INTERVAL '90 days';

-- Mengoptimalkan kembali ruang disk tabel
VACUUM ANALYZE prediction_logs;
```

### C. Alur Perbaikan Mendesak (Hotfix Workflow)
Jika ditemukan cacat fungsi fatal selama sesi demonstrasi berlangsung[cite: 37]:
```bash
# 1. Buat cabang perbaikan darurat langsung dari cabang main
git checkout main
git pull origin main
git checkout -b hotfix/fix-critical-bug

# 2. Perbaiki kode, uji lokal, dan commit
git add .
git commit -m "fix: resolve critical parsing error during demo"

# 3. Gabungkan kembali ke main dan berikan git tag baru
git checkout main
git merge hotfix/fix-critical-bug
git tag -a v1.0.1 -m "Hotfix: resolve critical parsing error"
git push origin main --tags

# 4. Sinkronkan perbaikan tersebut ke develop
git checkout develop
git merge hotfix/fix-critical-bug
git push origin develop
```