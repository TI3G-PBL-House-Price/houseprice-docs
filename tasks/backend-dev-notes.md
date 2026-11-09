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

## 2. Rincian Penugasan & Integrasi Modul Sprint 2 (Milestone 2)

> [!NOTE]
> Seluruh kerangka web backend, endpoint RESTful API (`auth`, `locations`, `market-statistics`, `predict`, `notifications`), integrasi PostgreSQL, Web Admin Dashboard, dan otomasi pengujian Pytest dikerjakan langsung secara terpusat oleh **Abdul Rahman Hanif Darmawan (@abdulrahmanhanif)**. Muhammad Nuril Huda difokuskan penuh pada penuntasan eksperimen ML dan pengeksporan model biner `.pkl`.

### A. Muhammad Nuril Huda (@nurilhuda) — Model ML Artefak & Client Networking
* **Penyediaan Artefak Model Machine Learning (`models_artefacts/model.pkl`):**
  - Tuntaskan eksperimen komparasi model regresi pada repositori `ml-model-testing` (Random Forest, Linear Regression, Gradient Boosting/XGBoost).
  - Ekspor model biner terbaik ($R^2 \ge 80\%$) ke format biner `model.pkl` lengkap dengan pipeline praproses fitur (One-Hot Encoding wilayah dan StandardScaler numerik).
  - Letakkan berkas model pada direktori `models_artefacts/model.pkl` di repositori backend agar dapat dimuat oleh `app/services/ml_service.py`.
* **Sinkronisasi Klien Mobile (Flutter):**
  - Sesuaikan `AuthInterceptor` pada repositori `houseprice-mobile-app` agar menangkap token JWT dari struktur respons baru: `response.data['data']['access_token']`.
  - Simpan token secara aman ke `flutter_secure_storage`.
  - Integrasikan pemanggilan `POST /api/v1/predict` dan konsumsi riwayat kalkulasi pada aplikasi mobile.

---

### B. Abdul Rahman Hanif Darmawan (@abdulrahmanhanif) — Backend, Web Admin & Test Automation
* **ML Service Layer & Predict API (`app/services/ml_service.py` & `app/blueprints/predict/`):**
  - Mengimplementasikan loader model `joblib.load()` dengan penanganan graceful fallback praproses.
  - Menghitung rentang deviasi toleransi wajar $\pm 10\%$:
    $$\text{Lower Bound} = \text{Predicted Price} \times 0.90$$
    $$\text{Upper Bound} = \text{Predicted Price} \times 1.10$$
  - Menyediakan endpoint `POST /api/v1/predict` (Strict Gate, simpan ke `prediction_logs`, latensi server $< 1.500$ ms) dan `GET /api/v1/predict/history` (paginasi per pengguna).
* **Master Lokasi & Statistik Pasar (`app/blueprints/market/` - Selesai):**
  - Menyediakan `GET /api/v1/locations` (data 46 kecamatan dikelompokkan per 5 kota administratif DKI Jakarta).
  - Menyediakan `GET /api/v1/market-statistics` (rata-rata harga/m² dengan filter query `city` dan `district`).
* **Pusat Notifikasi & Profil (`app/blueprints/notifications/` & `app/blueprints/auth/`):**
  - Endpoint `PUT /api/v1/auth/profile` dan `POST /api/v1/auth/fcm-token`.
  - Endpoint `GET /api/v1/notifications` (`unread_count`) dan `PATCH /api/v1/notifications/<uuid:id>/read`.
* **Web Admin Console (`/admin` - PR #1 Merged):**
  - Dashboard monokromatik gelap, proteksi sesi admin, 4 KPI cards, 2 visualisasi Chart.js, konsol simulasi *Live Inference Workbench*, dan 4 tab data explorer.
* **Otomasi Pengujian Integrasi API (Pytest Test Suite - Selesai):**
  - Pembuatan automated test suite pada direktori `tests/` (`conftest.py`, `test_market_api.py`, `test_auth_api.py`) dengan 7 skenario uji 100% lulus.

---

### C. Naufal Abid Aurizky (@naufalabid) & Jasmine Nasywa Nabilah (@jasminenasywa) — Mobile UI & Data Binding
* **Konsumsi API pada Form Properti & Beranda:**
  - Integrasikan respons dari `GET /api/v1/locations` ke komponen dropdown kota dan kecamatan dinamis pada form spesifikasi properti mobile.
  - Hubungkan kartu ringkasan tren pasar di Beranda Mobile dengan data dari `GET /api/v1/market-statistics`.
  - Validasi alur Cek Kewajaran deviasi $\pm 10\%$ pada tampilan mobile.

---

### D. Nurfinka Lailasari (@nurfinkalailasari) — QA Lead & Defect Tracking
* **Pencatatan Defect Tracking:**
  - Dokumentasikan setiap perbedaan respons atau kegagalan kontrak integrasi backend-mobile ke dalam lembar *Defect Tracking Log* untuk bahan penyusunan *Software Test Report* (STR).

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

