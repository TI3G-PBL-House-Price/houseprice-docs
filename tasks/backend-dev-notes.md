# Catatan Pengembangan Backend & Panduan Handover Teknis (Sprint 1 ke Sprint 2) - HousePrice

- **Dokumen:** Catatan Teknis & Panduan Handover Modul Backend Flask
- **Status:** Aktif (Transisi Milestone 1 menuju Milestone 2)
- **Penulis:** Abdul Rahman Hanif Darmawan (@abdulrahmanhanif) — *Project Manager & Backend/DB Lead*
- **Target Pembaca / Pelaksana:**
  - Muhammad Nuril Huda (@nurilhuda) — *ML & Networking Engineer*
  - Naufal Abid Aurizky (@naufalabid) — *Preprocessing & Business Logic*
  - Nurfinka Lailasari (@nurfinkalailasari) — *QA Lead & Local Storage*
  - Jasmine Nasywa Nabilah (@jasminenasywa) — *Mobile UI & Data Binding*

---

## 1. Ringkasan Status Backend Terkini (Baseline Milestone 1)

Inisiasi repositori backend [`houseprice-backend-ml`](file:///c:/Users/HAMI/Downloads/PBL%20SMT%205/Repository/houseprice-backend-ml) telah berhasil diselesaikan dengan cakupan:
1. **Arsitektur Flask Application Factory:** 
   - Pola modular berbasis `create_app()` pada `app/__init__.py`.
   - Pemisahan inisialisasi ekstensi pada `app/extensions.py` (`SQLAlchemy`, `Bcrypt`, `Migrate`, `CORS`).
2. **Skema Basis Data Relasional 9 Tabel:**
   - Seluruh model basis data PostgreSQL telah didefinisikan pada `app/models/` sesuai spesifikasi `docs/docs/data-model.md` (`admins`, `users`, `notifications`, `master_locations`, `dataset_metadata`, `ml_models`, `market_statistics`, `prediction_logs`, `system_audit_logs`).
   - Berkas migrasi awal Alembic `migrations/versions/e70def023971_create_initial_9_tables.py` telah terbentuk dengan *constraints*, *composite indexes*, dan *partial index* `idx_ml_models_active`.
3. **Strict Gate Authentication Sesuai OpenAPI 3.0:**
   - Enkripsi kata sandi Bcrypt minimal 12 putaran (`rounds=12`).
   - Token JWT berbasis algoritma HS256 dengan masa aktif terkonfigurasi.
   - Endpoint `POST /api/v1/auth/register` telah memvalidasi kelengkapan nama, keabsahan email, kekuatan sandi, dan kecocokan `confirm_password`.
   - Endpoint `POST /api/v1/auth/login` telah diselaraskan dengan kontrak `openapi.yaml`, mengembalikan token dan identitas user di dalam objek `data` bersarang (`access_token`, `token_type`, `expires_in`, `user`).
   - Endpoint `GET /api/v1/auth/me` terproteksi dekorator `@token_required`.
4. **Orkestrasi Kontainerisasi (`docker-compose.yml`):**
   - Mendefinisikan layanan `db` (PostgreSQL 15 Alpine dengan *healthcheck*) dan `backend` (Flask Gunicorn).
5. **Struktur Layer Layanan:**
   - Direktori `app/services/` telah disiapkan untuk pemisahan logika bisnis dan *runtime* model ML.

---

## 2. Rincian Handover Tugas Rekan Tim untuk Sprint 2 (Milestone 2)

### A. Muhammad Nuril Huda (@nurilhuda) — ML Runtime & Networking
* **Integrasi Model Machine Learning (`app/services/ml_service.py`):**
  - Setelah eksperimen komparasi model di `ml-model-testing` selesai dan artefak terbaik diekspor ke format `.pkl` (disimpan di `models_artefacts/model.pkl`):
  - Buat class layanan di `app/services/ml_service.py` untuk memuat model menggunakan `joblib.load()`.
  - Implementasikan transformasi fitur: *One-Hot Encoding* wilayah dan *StandardScaler* numerik sesuai pipeline latih.
  - Tambahkan kalkulasi rentang deviasi wajar $\pm 10\%$:
    $$\text{Lower Bound} = \text{Predicted Price} \times 0.90$$
    $$\text{Upper Bound} = \text{Predicted Price} \times 1.10$$
* **Endpoint Inferensi & Riwayat (`app/blueprints/predict/routes.py`):**
  - **`POST /api/v1/predict` (Strict Gate):**
    - Pasang dekorator `@token_required`.
    - Validasi masukan: `surface_area >= 1`, `building_area >= 1`, `bed >= 0`, `bath >= 0`, `carport >= 0`.
    - Eksekusi inferensi melalui `MLService`.
    - Simpan riwayat ke tabel `prediction_logs` dengan mengikat `user_id = current_user.id`, `latency_ms`, dan `client_ip`.
    - Pastikan total waktu eksekusi server $< 1.500$ ms.
  - **`GET /api/v1/predict/history` (Strict Gate):**
    - Pasang dekorator `@token_required`.
    - Ambil data dari `prediction_logs` di mana `user_id == current_user.id` terurut `created_at DESC` dengan parameter paginasi `limit` (default 20) dan `offset` (default 0).
* **Sinkronisasi Klien Mobile (Flutter):**
  - Sesuaikan `AuthInterceptor` pada aplikasi Flutter agar menangkap token dari struktur baru `response.data['data']['access_token']`.
  - Simpan token ke `flutter_secure_storage`.

---

### B. Naufal Abid Aurizky (@naufalabid) — Master Data & Logika Bisnis
* **Blueprint Master Wilayah & Statistik Pasar:**
  - Buat blueprint baru (misal: `app/blueprints/market/routes.py`) dan daftarkan di `app/__init__.py`.
  - **`GET /api/v1/locations`:**
    - Ambil data dari tabel `master_locations` di mana `is_active = TRUE`.
    - Format output JSON: Kelompokkan kota beserta daftar kecamatannya (`city`, array `districts`) untuk kebutuhan *cascading dropdown* di mobile.
  - **`GET /api/v1/market-statistics`:**
    - Ambil agregasi statistik harga rata-rata properti per wilayah dari tabel `market_statistics` dengan dukungan filter query parameter `city` dan `district`.
* **Pusat Notifikasi (`app/blueprints/notifications/routes.py`):**
  - **`GET /api/v1/notifications` (Strict Gate):**
    - Pasang dekorator `@token_required`.
    - Ambil daftar notifikasi milik `current_user.id` dari tabel `notifications`.
    - Hitung jumlah pesan belum dibaca (`unread_count = count(is_read == FALSE)`).
  - **`PATCH /api/v1/notifications/<uuid:id>/read` (Strict Gate):**
    - Perbarui status `is_read = TRUE` hanya jika pesan tersebut berelasi dengan `current_user.id`.

---

### C. Nurfinka Lailasari (@nurfinkalailasari) — QA Lead & API Test Automation
* **Otomasi Pengujian Integrasi API (Postman / Pytest):**
  - Buat koleksi uji otomatis untuk memvalidasi skenario positif dan negatif:
    - Register: Validasi email ganda (409), password beda (400), format salah (400), sukses (201).
    - Login: Kredensial salah (401), akun nonaktif (403), sukses (200).
    - Predict: Akses tanpa token (401), token kedaluwarsa (401), luas tanah 0 (400), sukses (200).
  - Verifikasi target NFR: Catat latensi endpoint inferensi `/predict` pada 50 kali eksekusi beruntun (target rata-rata $< 1,5$ detik).
* **Pencatatan Defect Tracking:**
  - Dokumentasikan setiap perbedaan respons atau kegagalan kontrak ke dalam lembar *Defect Tracking Log* untuk bahan penyusunan *Software Test Report* (STR).

---

### D. Jasmine Nasywa Nabilah (@jasminenasywa) — Slicing & Mobile Data Binding
* **Konsumsi API pada Form Properti & Beranda:**
  - Integrasikan respons dari `GET /api/v1/locations` ke komponen dropdown kota dan kecamatan dinamis pada form properti mobile.
  - Hubungkan kartu ringkasan tren pasar di Beranda Mobile dengan data dari `GET /api/v1/market-statistics`.

---

## 3. Standar Format Respons Kontrak API (Wajib Dipatuhi)

Untuk menjaga konsistensi integrasi dengan aplikasi mobile Flutter, seluruh rute baru **wajib mengikuti format standar `openapi.yaml`**:

### Format Respons Sukses:
```json
{
  "status": "success",
  "message": "Pesan deskriptif keberhasilan operasi",
  "data": { ... }
}
```

### Format Respons Galat (Error):
```json
{
  "status": "error",
  "error_code": "KODE_ERROR_STANDAR",
  "message": "Penjelasan galat yang dapat dipahami pengguna",
  "details": [
    "Rincian spesifik parameter yang bermasalah (opsional)"
  ]
}
```
*Daftar `error_code` umum:* `UNAUTHORIZED`, `TOKEN_EXPIRED`, `INVALID_CREDENTIALS`, `INVALID_INPUT_SCHEMA`, `RESOURCE_NOT_FOUND`, `DUPLICATE_RESOURCE`, `INTERNAL_SERVER_ERROR`.

---

## 4. Konvensi Git & Alur Kolaborasi

1. **Selalu buat cabang kerja baru dari `develop`:**
   - Format penamaan cabang: `feat/<nama-singkat-fitur>` atau `fix/<nama-bug>`.
   - Contoh: `feat/ml-inference-service` (Nuril), `feat/master-location-endpoints` (Naufal), `test/api-postman-collection` (Nurfinka).
2. **Gunakan format *Conventional Commits*:**
   - `feat: implement prediction endpoint with token auth`
   - `fix: resolve null check on district query param`
   - `test: add automated test cases for login endpoint`
3. **Buka Pull Request (PR) ke cabang `develop`:**
   - Wajib ditinjau dan disetujui minimal oleh *Project Manager* (@abdulrahmanhanif) sebelum digabungkan.

