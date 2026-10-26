# Agent Instructions & Project Context (HousePrice)

Berkas ini adalah panduan operasional kontekstual untuk AI Coding Assistant (LLM/Agent) saat membaca, mengedit, atau menghasilkan kode pada repositori proyek HousePrice.

---

## 1. Identitas, Arsitektur & Batasan Proyek
- Proyek ini adalah tugas akhir perkuliahan **Project-Based Learning (PBL) Semester 5 Jurusan Teknologi Informasi Polinema**.
- Model pembelajaran: *Product-Based*. Seluruh luaran kode harus berjalan penuh (*runnable*), teruji, dan siap dideploy ke server staging/production.
- Struktur repositori di GitHub Organization (`TI3G-PBL-House-Price`):
  1. `houseprice-docs`: Repositori dokumentasi terpusat (PRD, Arsitektur, Kontrak OpenAPI, Standar SQA, Database DDL).
  2. `houseprice-mobile-app`: Aplikasi klien mobile berbasis Flutter.
  3. `houseprice-backend-ml`: Layanan RESTful API, inferensi model ML, dan Dashboard Web Admin berbasis Flask & PostgreSQL.
  4. `ml-model-testing`: Repositori eksperimen, exploratory data analysis (EDA), dan pelatihan model Machine Learning.
- **Prinsip Akses Mobile (Strict Gate Authentication):** Seluruh modul fungsional utama (Beranda, Prediksi ML, Cek Kewajaran, Kalkulator KPR, Notifikasi, dan Profil) berada di balik autentikasi ketat. Pengguna wajib memiliki akun dan login aktif untuk dapat mengakses sistem.

---

## 2. Struktur Tim & Aturan Kepemilikan Kode
Agent wajib menjaga konsistensi tanggung jawab 5 anggota tim pada setiap kode dan dokumen yang dihasilkan:
- **Abdul Rahman Hanif Darmawan (PM & QA Integration):** Pemilik arsitektur otentikasi JWT backend, form auth & form input spesifikasi rumah di mobile, skrip pengujian integrasi API (Postman/Newman), serta koordinasi kualitas sistem.
- **Muhammad Nuril Huda (ML & Networking):** Pemilik arsitektur training model regresi (`.pkl`), HTTP Networking mobile (`AuthInterceptor`), State Management global (Bloc/Cubit) untuk auth & notifikasi, dan controller backend API.
- **Nurfinka Lailasari (Data Cleaning & Local Storage):** Pemilik pembersihan missing values dataset, manajemen penyimpanan sesi aman (`flutter_secure_storage`), layar Splash & Onboarding, modul Profil Pengguna & Riwayat Prediksi personal, serta Software Test Report (STR).
- **Naufal Abid Aurizky (Preprocessing & Business Logic):** Pemilik outlier handling IQR, feature encoding/scaling, logika bisnis evaluasi kelayakan harga properti, kalkulator KPR (anuitas), dan endpoint layanan notifikasi.
- **Jasmine Nasywa Nabila (Data Duplicate & Mobile UI):** Pemilik penanganan data duplikat, UI Design System mobile (komponen form auth, carousel onboarding, landing page), layar Beranda statistik pasar properti, dan Blackbox Testing (BVA/EP).

---

## 3. Alur Pengguna & Modul Aplikasi Mobile

AI Agent harus mengikuti hierarki alur antarmuka mobile berikut:

1. **Splash Screen:** Memeriksa status pertama kali buka aplikasi (`SharedPreferences`) dan ketersediaan JWT token aktif di penyimpanan aman (`flutter_secure_storage`).
2. **Onboarding Screen:** Ditampilkan khusus bagi pengguna baru. Memuat 3 *slide carousel* edukasi fitur (Prediksi ML, Evaluasi Harga, Kalkulator KPR).
3. **Landing Page:** Tampilan sambutan awal sebelum otentikasi. Menyediakan pintu masuk menuju layar Login atau Registrasi.
4. **Auth Module (Strict Gate):**
   - **Register Screen:** Input nama lengkap, email, password, dan konfirmasi password.
   - **Login Screen:** Input email dan password untuk memperoleh Bearer JWT Token.
5. **Main Navigation (Bottom Navigation Bar):**
   - **Beranda (Home):** Visualisasi tren rata-rata harga per meter persegi di wilayah DKI Jakarta.
   - **Prediksi Harga (ML Core):** Formulir spesifikasi rumah (LT, LB, kamar tidur, kamar mandi, carport, kecamatan, kota) untuk kalkulasi taksiran harga pasar.
   - **Cek Kewajaran:** Komparasi harga penawaran riil terhadap hasil estimasi ML (Murah, Wajar, Mahal).
   - **Kalkulator KPR:** Simulasi angsuran perbankan berbasis metode anuitas.
   - **Notifikasi:** Daftar pesan pembaruan tren properti dan pengumuman sistem.
   - **Profil Pengguna:** Informasi akun, riwayat prediksi personal pengguna, ubah kata sandi, dan tombol *Logout*.

---

## 4. Konvensi Teknis Wajib (Rules for Code Generation)

### A. Python & Flask Backend
- Gunakan **Python 3.10+**.
- **Keamanan Kredensial:** Seluruh kata sandi pengguna wajib di-hash menggunakan `bcrypt` dengan minimal 12 *salt rounds*. Dilarang menyimpan kata sandi dalam bentuk *plain text*.
- **Mekanisme Autentikasi:** Menerapkan JSON Web Token (JWT) dengan algoritma HS256. Setiap token wajib memuat payload `user_id`, `email`, dan waktu kedaluwarsa (`exp`).
- **Middleware Proteksi:** Wajib menggunakan decorator otentikasi (misal `@token_required`) untuk memvalidasi header `Authorization: Bearer <token>` pada endpoint privat (`/api/v1/predict`, `/api/v1/auth/me`, `/api/v1/auth/profile`, `/api/v1/notifications/*`).
- **Pencatatan Inferensi:** Endpoint `POST /api/v1/predict` wajib mengekstrak `user_id` dari token JWT yang terverifikasi dan menyimpannya ke tabel `prediction_logs`.
- **Serialisasi Model ML:** Berkas model disimpan murni menggunakan `joblib` (`.pkl`).
- **Format Respons API:** Seluruh endpoint wajib mengembalikan struktur JSON konsisten:
  ```json
  {
    "status": "success",
    "data": {},
    "message": "Deskripsi respons yang jelas"
  }
  ```
- **Error Handling:** Kembalikan status HTTP standar: `400 Bad Request` (validasi payload), `401 Unauthorized` (token tidak valid/kedaluwarsa), `403 Forbidden` (akses dilarang), `404 Not Found`, dan `500 Internal Server Error`.

### B. Mobile Flutter
- Gunakan **Flutter 3.x** dengan kepatuhan penuh terhadap **Sound Null Safety**.
- **Arsitektur Folder (Clean Architecture):** Wajib dipisahkan per fitur di bawah `lib/features/`:
  - `auth/` (data, domain, presentation)
  - `onboarding/` (presentation)
  - `home/` (data, domain, presentation)
  - `prediction/` (data, domain, presentation)
  - `kpr/` (data, domain, presentation)
  - `notification/` (data, domain, presentation)
  - `profile/` (data, domain, presentation)
- **Keamanan Token Lokal:** JWT access token dan refresh token **WAJIB** disimpan menggunakan `flutter_secure_storage`. Dilarang keras menggunakan `SharedPreferences` biasa untuk data sensitif atau token otentikasi.
- **HTTP Client & Interceptor:** Gunakan instance Dio/Http terpusat dengan implementasi `AuthInterceptor`:
  - Otomatis menyematkan header `Authorization: Bearer <token>` pada setiap request ke endpoint terproteksi.
  - Tangani respons HTTP `401 Unauthorized` dengan memicu *event logout*, menghapus token dari secure storage, dan me-redirect antarmuka pengguna kembali ke Landing Page.
- **State Management:** Gunakan Bloc/Cubit secara konsisten untuk mengelola `AuthState`, `PredictionState`, dan `NotificationState`.
- **Penanganan Galat Antarmuka:** Tangani `SocketException` dan `TimeoutException` secara ramah melalui SnackBar atau komponen Dialog informatif; dilarang membiarkan aplikasi crash atau menampilkan layar merah.

### C. Basis Data (PostgreSQL) & Lingkungan Kerja
- Database utama adalah **PostgreSQL**. Dilarang mengasumsikan MySQL atau SQLite untuk backend API.
- Skema tabel wajib mencakup:
  - Akun & Otentikasi: `admins`, `users`
  - Referensi & Log: `master_locations`, `market_statistics`, `notifications`, `system_audit_logs`
  - Machine Learning: `dataset_metadata`, `ml_models`, `prediction_logs` (berelasi ke `users.id`)
- **Environment Variables:** Konfigurasi rahasia (`DATABASE_URL`, `JWT_SECRET_KEY`, `FLASK_ENV`, `PORT`) wajib dimuat menggunakan `python-dotenv` dari berkas `.env` dan tidak boleh dimasukkan secara langsung ke dalam repositori git.