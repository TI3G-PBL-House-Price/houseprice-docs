# Product Requirements Document (PRD) - HousePrice

- **Nama Produk:** HousePrice (Prediksi Harga Rumah DKI Jakarta)
- **Kategori:** Mobile Intelligent System (Integrated Machine Learning)
- **Target Platform:** Android OS 10.0+ (Klien) & Web Browser (Admin)
- **Status Dokumen:** Approved (Baseline Semester 5)

---

## 1. Latar Belakang & Masalah Bisnis
Penetapan dan evaluasi harga rumah di wilayah DKI Jakarta memiliki tingkat volatilitas yang tinggi dipengaruhi oleh lokasi administratif, luas tanah/bangunan, dan fasilitas properti. Pembeli awam dan keluarga muda sering menghadapi risiko *overpricing* (membeli di atas harga pasar wajar), sementara penjual kesulitan menentukan nilai pasar yang kompetitif. 

HousePrice menjawab masalah ini dengan menyediakan taksiran harga berbasis bukti (*data-driven*) menggunakan machine learning yang dapat diakses secara transparan melalui gawai mobile.

---

## 2. Profil Pengguna (Personas)
1. **Calon Pembeli Properti (End User):** Masyarakat umum yang membutuhkan validasi harga pasar sebelum mengajukan penawaran, serta memerlukan kalkulasi cicilan KPR riil.
2. **Pemilik / Penjual Properti (End User):** Pengguna yang ingin menetapkan harga jual wajar properti miliknya berdasarkan atribut fisik rumah.
3. **Administrator Teknis (Admin):** Tim internal yang bertanggung jawab memantau metrik performa model inferensi, memperbarui dataset pasar, dan menjalankan pelatihan ulang model secara berkala.

---

## 3. Ruang Lingkup Sistem

### Yang Dikerjakan (In-Scope):
* Aplikasi mobile Flutter untuk input spesifikasi properti, inferensi harga, evaluasi penawaran, simulasi KPR, dan penyimpanan riwayat lokal.
* RESTful API Flask untuk melayani inferensi model Machine Learning dan statistik pasar.
* Model regresi berbasis ensemble (Random Forest) dengan target metrik $R^2 \ge 0.80$.
* Web Dashboard Admin untuk upload dataset, monitoring metrik ($R^2$, MAE, RMSE), dan retraining model.
* Skema basis data PostgreSQL (7 entitas) untuk logging dan manajemen model.

### Yang Tidak Dikerjakan (Out-of-Scope):
* Visualisasi peta spasial berupa *heatmap* sebaran harga.
* Sistem rekomendasi listing properti jual-beli spesifik (bukan marketplace).
* Transaksi finansial atau pengajuan kredit KPR langsung ke lembaga perbankan.
* Platform iOS (prioritas utama hanya Android).

---

## 4. Spesifikasi Fitur Fungsional

### A. Aplikasi Mobile (End User)
* **FR-MOB-01 (Beranda Pasar):** Menampilkan ringkasan rata-rata harga pasar DKI Jakarta, tren per kota administrasi, dan filter statistik wilayah.
* **FR-MOB-02 (Form Prediksi ML):** Form input parameter: Kota (Dropdown), Kecamatan (Dropdown dinamis), Luas Tanah ($m^2$), Luas Bangunan ($m^2$), Kamar Tidur, Kamar Mandi, dan Carport. Validasi masukan lokal mencegah angka negatif/nol.
* **FR-MOB-03 (Hasil Prediksi & Estimasi Nilai Pasar):** Menampilkan estimasi harga nominal (Rupiah) beserta rentang toleransi batas wajar (Lower-Upper Bound).
* **FR-MOB-04 (Cek Kewajaran Harga):** Membandingkan harga penawaran riil penjual terhadap nilai prediksi ML, menghasilkan label status: *Murah / Dibawah Pasar* (< -10%), *Wajar* (antara -10% s/d +10%), atau *Mahal / Diatas Pasar* (> +10%).
* **FR-MOB-05 (Kalkulator Simulasi KPR):** Menghitung estimasi angsuran bulanan berbasis metode bunga anuitas berdasarkan harga rumah, persentase Uang Muka (DP), suku bunga per tahun, dan tenor pinjaman (tahun).
* **FR-MOB-06 (Riwayat Prediksi Lokal):** Menyimpan histori komputasi ke database lokal perangkat (SQLite / SharedPreferences) yang dapat dilihat kembali atau dihapus secara offline tanpa akun.

### B. Web Dashboard Admin (Internal Team)
* **FR-WEB-01 (Autentikasi Admin):** Login berbasis session/token aman menggunakan password yang di-hash dengan Bcrypt.
* **FR-WEB-02 (Dashboard Monitoring):** Visualisasi metrik model aktif ($R^2$, MAE, RMSE), volume dataset latih/uji, dan total volume log inferensi.
* **FR-WEB-03 (Manajemen Dataset):** Mengunggah berkas `jakarta_house.csv` baru, pratinjau tabel 10 baris pertama, dan pengaturan rasio train-test split (default 80:20).
* **FR-WEB-04 (Pengelolaan & Retraining Model):** Memicu proses pipeline data preparation dan re-training model di server, membandingkan skor metrik model baru vs model aktif, serta konfirmasi switch model utama (.pkl).

### C. Cloud Services & REST API
* **FR-API-01 (`POST /api/v1/predict`):** Menerima payload JSON spesifikasi rumah, melakukan validasi schema, menjalankan inferensi model `.pkl`, mencatat ke tabel `prediction_logs`, dan mengembalikan nominal harga dalam < 1.5 detik.
* **FR-API-02 (`GET /api/v1/market-statistics`):** Menyediakan agregasi rata-rata harga dan luas per meter persegi untuk kebutuhan Beranda Mobile.
* **FR-API-03 (`POST /api/v1/retrain`):** Endpoint khusus admin untuk memicu retraining asynchronous.

---

## 5. Kebutuhan Non-Fungsional (NFR)
1. **Performance:** Waktu latensi inferensi API maksimal 1.500 ms (1,5 detik) pada jaringan 4G/WiFi stabil.
2. **Availability:** Layanan inferensi backend uptime target 99% selama masa uji evaluasi dosen.
3. **Security:** Tidak ada kredensial yang di-commit ke Git; password admin dienkripsi Bcrypt; API terlindungi dari request payload tidak valid via HTTP 400 Bad Request.
4. **Compatibility:** Aplikasi mobile kompatibel minimal pada Android API Level 29 (Android 10) hingga Android API Level 34+ (Android 14).