# Engineering Guidelines & Ruleset - HousePrice

Dokumen ini memuat standar teknis pengembangan perangkat lunak, konvensi penamaan kode, aturan percabangan Git, dan alur pengujian mutu (QA) untuk seluruh anggota tim.

---

## 1. Aturan Percabangan Git (Branching Ruleset)

Repositori dikelola dengan pola **Git Flow Terstruktur**:

### A. Cabang Utama
* `main`: Cabang produksi terlindungi (*protected*). Hanya berisi kode stabil yang siap didemokan ke hadapan dosen penguji. Merge ke `main` wajib melalui Pull Request dan tagging rilis.
* `develop`: Cabang integrasi harian (*staging*). Seluruh cabang fitur bermuara di sini setelah lolos review.

### B. Penamaan Cabang Kerja (Feature Branches)
Format penamaan cabang wajib: `<tipe>/<nama-singkat-fitur>`

1. **Repositori `houseprice-mobile-app`:**
   * `feat/ui-design-system-home` (Jasmine)
   * `feat/property-form-input` (Hanif)
   * `feat/core-networking-state` (Nuril)
   * `feat/valuation-kpr-feature` (Naufal)
   * `feat/result-local-history` (Nurfinka)
   * `test/widget-form-bva` (Jasmine)

2. **Repositori `houseprice-backend-ml` & `ml-model-testing`:**
   * `feat/data-eda-visualization` (Hanif)
   * `feat/data-clean-duplicates` (Jasmine)
   * `feat/data-clean-missing` (Nurfinka)
   * `feat/data-outlier-encoding` (Naufal)
   * `feat/model-train-tuning` (Nuril)
   * `test/api-integration-perf` (Hanif)

---

## 2. Standar Pesan Commit (Conventional Commits)
Setiap commit harus mengikuti format standar: `<type>: <keterangan jelas dalam huruf kecil>`

* `feat:` Menambahkan fitur baru (contoh: `feat: implement kpr annuity formula`)
* `fix:` Memperbaiki bug/galat (contoh: `fix: handle null exception on district dropdown`)
* `test:` Menambahkan atau memodifikasi skrip pengujian (contoh: `test: add bva test cases for land area input`)
* `refactor:` Mengubah struktur kode tanpa mengubah fungsi luar (contoh: `refactor: extract property card into reusable widget`)
* `chore:` Penyesuaian build script, dependensi, atau env (contoh: `chore: update pubspec dependencies`)
* `docs:` Pembaruan file dokumentasi markdown (contoh: `docs: update api specification`)

---

## 3. Standar Implementasi Arsitektur

### A. Arsitektur Mobile (Flutter)
Menerapkan **Layered Architecture**:
1. **Presentation Layer:** Berisi Widget tampilan UI, Form input, Dialog error, dan State Management (Provider/BLoC).
2. **Business Logic Layer:** Validasi masukan lokal (misal: luas tanah $\ge 1$, kamar $\ge 0$), formula matematika KPR (anuitas bulanan), dan persentase rentang deviasi harga penawaran ($\pm 10\%$).
3. **Data/Network Layer:** Komunikasi HTTP RESTful API ke server cloud menggunakan package `http` atau `dio`, serialisasi model JSON, dan driver SQLite/SharedPreferences untuk penyimpanan offline.

### B. Arsitektur Backend (Flask & Cloud)
* Mengikuti prinsip modularitas (*Flask Blueprints*):
  * `blueprints/api/`: Endpoint inferensi dan statistik publik.
  * `blueprints/admin/`: Rute antarmuka Web Dashboard.
  * `services/ml_service.py`: Modul pemuat model `.pkl` dan eksekusi inferensi.
  * `services/db_service.py`: Operasi query PostgreSQL.
* Response time inferensi wajib di bawah **1.500 ms**.
* Semua request ke database wajib parameterized untuk mencegah serangan SQL Injection.

---

## 4. Standar Penjaminan Mutu Perangkat Lunak (PMPL)
Setiap fitur yang selesai dikembangkan tidak boleh langsung digabungkan ke `main` sebelum memenuhi kriteria verifikasi:
1. **Blackbox Testing (Form Input):** Wajib lulus uji *Boundary Value Analysis* (BVA) dan *Equivalence Partitioning* (EP) untuk masukan bernilai negatif, nol, atau karakter huruf.
2. **Business Logic Testing:** Rumus angsuran KPR bulanan wajib diverifikasi akurasinya terhadap kalkulator finansial standar bank.
3. **API Performance Testing:** Latensi endpoint `/api/v1/predict` diukur menggunakan skrip Postman Runner pada 50 kali permintaan berturut-turut dengan target rata-rata $< 1,5$ detik.
4. **Defect Tracking & STR:** Setiap bug dicatat pada lembar pelacak cacat (*Defect Tracking Log*) dan ditutup secara resmi sebelum penyusunan *Software Test Report* (STR).