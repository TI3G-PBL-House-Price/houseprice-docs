# Rekapitulasi Sprint 0: Identifikasi Masalah & Proposal (Sprint 0 Summary) - HousePrice

- **Periode Pelaksanaan:** Minggu 1 s/d Minggu 4 (Fase Identifikasi & Perencanaan)[cite: 7, 8]
- **Status Evaluasi:** Disetujui Dosen Pembimbing (*Approved Proposal Baseline*)[cite: 8, 10]
- **Target Luaran:** Proposal Proyek PBL, Dokumen Perencanaan Fitur, Skema Basis Data (ERD), dan Aturan Percabangan Git (*Branching Ruleset*)[cite: 1, 8, 31]

---

## 1. Ringkasan Eksekutif & Identifikasi Masalah

Pada fase Sprint 0, tim mengidentifikasi persoalan nyata di sektor pasar properti DKI Jakarta, khususnya asimetri informasi yang dialami calon pembeli rumah dan keluarga muda dari luar daerah yang belum memahami nilai pasar wajar[cite: 20, 21]. Di sisi lain, pemilik rumah sering mengalami kesulitan menentukan valuasi harga jual yang kompetitif dan proporsional terhadap atribut fisik bangunan miliknya[cite: 20, 21].

Sistem **HousePrice** dirumuskan sebagai solusi berupa aplikasi terintegrasi berbasis *Machine Learning* dan *Mobile* yang memberikan taksiran nilai pasar secara objektif, menyediakan indikator evaluasi kewajaran penawaran (*Murah, Wajar, Mahal*), serta menyertakan kalkulator simulasi pembiayaan KPR berbasis bunga anuitas[cite: 20, 24, 25].

---

## 2. Capaian & Artefak yang Dihasilkan (Sprint Deliverables)

Seluruh target luaran pada fase inisiasi dan perencanaan telah selesai disusun dan diverifikasi[cite: 8]:

| Artefak / Dokumen | Deskripsi & Ruang Lingkup | Status Verifikasi |
| :--- | :--- | :--- |
| **Proposal PBL Resmi** | Dokumen 12 halaman memuat latar belakang, rumusan masalah, ruang lingkup *in-scope/out-of-scope*, studi kelayakan data, serta rancangan teknis awal[cite: 18, 20, 21, 22]. | **Disetujui** (Minggu 4)[cite: 8, 10] |
| **Dokumen Perencanaan (Excel)** | Perincian kebutuhan fitur Mobile (5 fitur), Web Admin (4 modul), Cloud/API (3 modul), serta kamus data awal[cite: 1, 23, 24]. | **Selesai**[cite: 1, 23, 24] |
| **Entity Relationship Diagram (ERD)** | Perancangan arsitektur basis data 7 tabel relasional pada platform Eraser.io (`admins`, `master_locations`, `dataset_metadata`, `ml_models`, `market_statistics`, `prediction_logs`, `system_audit_logs`)[cite: 1]. | **Selesai & Tervalidasi**[cite: 1] |
| **Studi Kelayakan Dataset** | Penelusuran dan validasi dataset `jakarta_house.csv` dari Kaggle yang memuat 10.000 baris data dan 8 atribut properti DKI Jakarta[cite: 21, 27]. | **Selesai & Layak**[cite: 21, 27] |
| **Branching Ruleset Guideline** | Penyusunan aturan alur percabangan Git (*Git Flow*), pembagian *branch* fitur per anggota tim, serta konvensi penulisan pesan *commit*[cite: 31, 33, 37]. | **Selesai & Diterapkan**[cite: 31, 33, 37] |
| **Desain Antarmuka (UI Mockup)** | Perancangan prototipe visual dan alur antarmuka pengguna (*design system*) untuk layar Beranda, Form Input, Hasil Prediksi, dan Kalkulator KPR[cite: 24, 25, 31]. | **Selesai (Aset Siap Slicing)**[cite: 31] |

---

## 3. Matriks Pembagian Peran Tim & Keterlibatan Mata Kuliah

Tim Kelompok 2 (Kelas TI-2G) menyepakati distribusi tanggung jawab yang seimbang lintas 3 mata kuliah terintegrasi[cite: 13, 19, 28]:

* **Dosen Pengampu Terintegrasi:**
  * Pemrograman Mobile: Habibie Ed Dien, S.Kom., M.T.[cite: 20]
  * Pembelajaran Mesin: Usman Nurhasan, S.Kom., M.T.[cite: 20]
  * Penjaminan Mutu Perangkat Lunak: M. Hasyim Ratsanjani, S.Kom., M.Kom.[cite: 20]

| Anggota Tim | NIM | Peran Utama | Pembelajaran Mesin | Pemrograman Mobile | Penjaminan Mutu (PMPL) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Abdul Rahman Hanif Darmawan** | 244107020232 | Project Manager & QA Integration[cite: 19, 28] | Exploratory Data Analysis (EDA) & Visualisasi Korelasi Fitur[cite: 19, 28] | Form Input Spesifikasi Properti & Validasi Masukan Lokal[cite: 19, 28] | Pengujian Integrasi RESTful API & Pengujian Latensi Postman (< 1,5s)[cite: 19, 28] |
| **Muhammad Nuril Huda** | 244107020004 | ML Model Engineer & Mobile Networking[cite: 19, 29] | Seleksi Algoritma, Tuning, Evaluasi ($R^2 \ge 0,80$), & Ekspor `.pkl`[cite: 19, 29] | Prediksi Core ML, REST Client Networking, & State Management[cite: 19, 29] | Pengujian Pengalaman Pengguna (*Usability*) & Responsivitas Layar[cite: 19, 29] |
| **Nurfinka Lailasari** | 244107020211 | ML Data Quality & Mobile Storage / QA Lead[cite: 19, 29] | Penanganan Nilai Kosong (*Missing Values*) Luas Tanah/Bangunan[cite: 19, 29] | Layar Hasil Prediksi & Penyimpanan Riwayat Lokal (SQLite/Prefs)[cite: 19, 29] | Manajemen Defect Tracking & Penyusunan Software Test Report (STR)[cite: 19, 29] |
| **Naufal Abid Aurizky** | 244107020212 | ML Preprocessing & Mobile Logic Developer[cite: 19, 29] | Outlier Handling IQR, One-Hot Encoding, Scaling, & Split Dataset[cite: 19, 29] | Logika Bisnis Cek Kewajaran Harga & Formula Bunga Anuitas KPR[cite: 19, 29] | Pengujian Logika Bisnis (*Business Logic Testing*) KPR & Kewajaran[cite: 19, 29] |
| **Jasmine Nasywa Nabilah** | 244107020119 | ML Data Duplicate & Mobile UI / Design System[cite: 19, 29] | Identifikasi & Pembuangan 526 Baris Data Duplikat[cite: 19, 29] | Tampilan Beranda Statistik Pasar Properti & UI Design System[cite: 19, 29] | Pengujian Blackbox Form Input (BVA & Equivalence Partitioning)[cite: 19, 29] |

---

## 4. Evaluasi Awal Dataset (`jakarta_house.csv`)

Pemeriksaan awal terhadap dataset publik Kaggle menghasilkan data ringkasan berikut[cite: 21, 27]:
* **Volume Data:** 10.000 baris observasi dengan 8 atribut fitur (`price`, `bed`, `bath`, `carport`, `surface_area`, `building_area`, `district`, `city`)[cite: 27].
* **Temuan Baris Duplikat:** Terdapat 526 baris data ganda yang wajib dieliminasi pada tahap awal pembersihan[cite: 27].
* **Temuan Nilai Kosong:** Terdapat 3 baris kosong pada `surface_area` dan 5 baris kosong pada `building_area` yang membutuhkan penanganan imputasi statistik[cite: 27].
* **Sebaran Pencilan (*Outliers*):** Variasi harga properti merentang dari Rp 1,85 juta hingga Rp 900 miliar, sehingga pemfilteran nilai ekstrem menggunakan metode rentang antar-kuartil (*Interquartile Range* / IQR) menjadi prasyarat wajib sebelum pelatihan model regresi[cite: 27].

---

## 5. Keputusan Transisi ke Sprint 1

Dengan disetujuinya dokumen Proposal pada batas akhir Minggu 4, tim secara resmi menutup Sprint 0 dan beralih ke pelaksanaan **Sprint 1 (Minggu 5 s/d 8)** dengan fokus utama[cite: 8, 10]:
1. Melakukan komparasi model regresi pada repositori `ml-model-testing` untuk memilih algoritma final terbaik (*Random Forest* vs model pembanding)[cite: 27, 33].
2. Memulai inisialisasi basis kode Flutter dan *slicing* antarmuka UI Beranda serta Form Input di repositori `houseprice-mobile-app`[cite: 31].
3. Mempersiapkan struktur awal *backend skeleton* di repositori `houseprice-backend-ml`[cite: 23, 41].