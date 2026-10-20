# Agent Instructions & Project Context (HousePrice)

Berkas ini adalah panduan operasional kontekstual untuk AI Coding Assistant (LLM/Agent) saat membaca, mengedit, atau menghasilkan kode pada proyek HousePrice.

---

## 1. Identitas & Batasan Proyek
- Proyek ini adalah tugas akhir perkuliahan **Project-Based Learning (PBL) Semester 5 Jurusan Teknologi Informasi Polinema**.
- Model pembelajaran: *Product-Based*. Kode harus menghasilkan produk nyata yang berjalan penuh (*runnable*), bukan sekadar mockup atau skrip demo parsial.
- Kode dibagi ke dalam 3 repositori di GitHub Organization:
  1. `houseprice-mobile-app` (Flutter)
  2. `houseprice-backend-ml` (Flask & PostgreSQL)
  3. `ml-model-testing` (Jupyter Notebooks / Python ML)

---

## 2. Struktur Tim & Aturan Kepemilikan Kode
Agent harus menghormati pembagian tanggung jawab 5 anggota tim saat menyarankan atau menyusun kode:
* **Abdul Rahman Hanif Darmawan (PM & QA Integration):** Pemilik form input mobile, validasi data, script pengujian integrasi API backend, dan performa Postman.
* **Muhammad Nuril Huda (ML & Networking):** Pemilik arsitektur training model regresi, pipeline tuning, file serialisasi `.pkl`, dan HTTP networking/State Management pada Flutter.
* **Nurfinka Lailasari (Data Cleaning & Local Storage):** Pemilik penanganan missing values dataset, layar UI hasil prediksi, database SQLite/SharedPreferences mobile, dan Software Test Report (STR).
* **Naufal Abid Aurizky (Preprocessing & Business Logic):** Pemilik outlier handling IQR, One-Hot Encoding, Feature Scaling, logika bisnis cek kewajaran harga, dan formula kalkulator KPR.
* **Jasmine Nasywa Nabila (Data Duplicate & Mobile UI):** Pemilik penanganan baris duplikat dataset, UI Design System, layar Beranda pasar properti, dan Blackbox Testing (BVA/EP).

---

## 3. Konvensi Teknis Wajib (Rules for Code Generation)

### A. Python & Flask Backend
* Gunakan **Python 3.10+**.
* Skema database PostgreSQL dioperasikan menggunakan ORM (SQLAlchemy) atau connection pool psycopg2 yang aman dari SQL Injection.
* Model ML diserialisasi murni menggunakan `joblib`.
* Jangan pernah menyematkan path lokal absolut (misal: `C:\Users\Admin\...`). Selalu gunakan `pathlib.Path` atau `os.path.join`.
* Seluruh endpoint API wajib mengembalikan response berformat JSON standar:
  ```json
  {
    "status": "success",
    "data": {},
    "message": "Deskripsi pesan respons"
  }

### B. Mobile Flutter
* Gunakan Flutter versi stabil (3.x) dengan Sound Null Safety.
* Struktur arsitektur berlapis: presentation/, business_logic/, data/.
* Hindari meletakkan logika kalkulasi langsung di dalam widget build().
* Tangani error handling jaringan (SocketException, TimeoutException) secara ramah pada UI (tampilkan SnackBar/Dialog, bukan layar merah crash).

### C. Basis Data & Environment Variables
* Database yang digunakan adalah PostgreSQL. Dilarang mengasumsikan MySQL atau SQLite untuk backend cloud.
* Segala konfigurasi rahasia (DB password, secret key) WAJIB dimuat melalui modul python-dotenv dari file .env.