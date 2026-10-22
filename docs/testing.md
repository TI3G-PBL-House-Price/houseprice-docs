# Rencana & Pedoman Pengujian Mutu (Software Testing & QA Plan) - HousePrice

Dokumen ini memuat rencana pengujian, rancangan kasus uji (*test cases*), matriks pengujian fungsional, pengujian logika bisnis, pengujian integrasi API, serta tata kelola pelaporan cacat perangkat lunak (*Software Test Report*) untuk memenuhi standar mata kuliah Penjaminan Mutu Perangkat Lunak (PMPL) pada proyek HousePrice[cite: 13].

---

## 1. Struktur Tim & Pembagian Tanggung Jawab QA (PMPL)

Setiap anggota tim memiliki peran khusus dalam eksekusi penjaminan mutu perangkat lunak:

| Anggota Tim | NIM | Fokus Pengujian PMPL | Ruang Lingkup & Metode |
| :--- | :--- | :--- | :--- |
| **Nurfinka Lailasari** | 244107020211 | **QA Lead & Defect Manager** | Manajemen Defect Tracking, verifikasi hasil prediksi, dan penyusunan *Software Test Report* (STR). |
| **Jasmine Nasywa Nabilah** | 244107020119 | **Functional Blackbox QA** | Pengujian Blackbox form input properti menggunakan metode *Boundary Value Analysis* (BVA) dan *Equivalence Partitioning* (EP). |
| **Abdul Rahman Hanif Darmawan** | 244107020232 | **API & Performance QA** | Pengujian integrasi RESTful API, validasi skema payload JSON, dan uji latensi inferensi Postman Runner (< 1,5 detik). |
| **Muhammad Nuril Huda** | 244107020004 | **Usability & UI QA** | Pengujian pengalaman pengguna (*Usability Testing*) dan responsivitas layout visual pada perangkat fisik Android. |
| **Naufal Abid Aurizky** | 244107020212 | **Business Logic QA** | Pengujian logika bisnis matematis pada formula cicilan KPR metode anuitas, rasio DSR OJK, dan formula deviasi kewajaran harga. |

---

## 2. Pengujian Blackbox Form Input: Equivalence Partitioning (EP) & Boundary Value Analysis (BVA)

Pengujian ini memastikan seluruh elemen masukan pada form spesifikasi properti menolak nilai tidak sah tanpa menyebabkan aplikasi mengalami penghentian mendadak (*crash*).

### A. Matriks Kelas Partisi (Equivalence Partitioning - EP)

| Parameter Input | Partisi Valid | Partisi Tidak Valid (Ditolak) |
| :--- | :--- | :--- |
| **Luas Tanah ($m^2$)** | Bilangan riil positif: $x \ge 30$ (Standar kaveling SHM DKI Jakarta)[cite: 1] | $x < 30$, bilangan negatif, karakter huruf/simbol |
| **Luas Bangunan ($m^2$)** | Bilangan riil positif: $x \ge 1$ | $x \le 0$, bilangan negatif, karakter huruf/simbol |
| **Kamar Tidur (`bed`)** | Bilangan bulat non-negatif: $x \in [0, 50]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 50) |
| **Kamar Mandi (`bath`)** | Bilangan bulat non-negatif: $x \in [0, 50]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 50) |
| **Carport** | Bilangan bulat non-negatif: $x \in [0, 30]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 30) |
| **Kota & Kecamatan** | Pilihan terdaftar pada daftar *dropdown* master data | Nilai kosong (*null/unselected*), teks manipulasi luar |

### B. Matriks Kasus Uji Nilai Batas (Boundary Value Analysis - BVA)

| Kode Uji | Parameter | Nilai Masukan Uji | Kategori Batas | Hasil yang Diharapkan |
| :--- | :--- | :--- | :--- | :--- |
| **TC-BVA-01** | Luas Tanah | `29` | Batas bawah tidak valid | Form memunculkan pesan galat: "Luas tanah minimal 30 m² (standar SHM)"[cite: 1]. |
| **TC-BVA-02** | Luas Tanah | `30` | Batas bawah valid | Input diterima, form lolos validasi[cite: 1]. |
| **TC-BVA-03** | Luas Tanah | `-5` | Nilai negatif ekstrem | Form memunculkan peringatan kesalahan input. |
| **TC-BVA-04** | Luas Bangunan | `0` | Batas bawah tidak valid | Form memunculkan pesan galat: "Luas bangunan minimal 1 m²". |
| **TC-BVA-05** | Luas Bangunan | `1` | Batas bawah valid | Input diterima, tombol prediksi aktif. |
| **TC-BVA-06** | Kamar Tidur | `-1` | Nilai negatif tidak valid | Nilai ditolak, input otomatis di-reset ke 0. |
| **TC-BVA-07** | Kamar Tidur | `0` | Batas bawah valid | Diterima (untuk rumah tipe studio). |
| **TC-BVA-08** | Carport | `0` | Batas bawah valid | Diterima (untuk rumah tanpa garasi/carport). |
| **TC-BVA-09** | Tipe Data | `"abc"` pada kolom luas | Karakter non-numerik | Kolom form tidak mengizinkan pengetikan huruf (*numeric only*). |

---

## 3. Pengujian Logika Bisnis (Business Logic Testing)

### A. Pengujian Logika Cek Kewajaran Harga (Layar Hasil)
Formula penentuan status kewajaran harga membandingkan penawaran aktual penjual ($P_{offer}$) terhadap estimasi model ML ($P_{pred}$):

$$\Delta\% = \frac{P_{offer} - P_{pred}}{P_{pred}} \times 100\%$$

| Skenario Uji | $P_{pred}$ (Model) | $P_{offer}$ (Penjual) | Selisih $\Delta\%$ | Label Evaluasi yang Diharapkan |
| :--- | :--- | :--- | :--- | :--- |
| **TC-LOG-01** | Rp 2.000.000.000 | Rp 1.700.000.000 | -15,0% | **Dibawah Pasar / Murah** ($\Delta < -10\%$) |
| **TC-LOG-02** | Rp 2.000.000.000 | Rp 1.800.000.000 | -10,0% | **Wajar / Sesuai Pasar** (Batas bawah toleransi) |
| **TC-LOG-03** | Rp 2.000.000.000 | Rp 2.050.000.000 | +2,5% | **Wajar / Sesuai Pasar** (Dalam batas deviasi $\pm 10\%$) |
| **TC-LOG-04** | Rp 2.000.000.000 | Rp 2.200.000.000 | +10,0% | **Wajar / Sesuai Pasar** (Batas atas toleransi) |
| **TC-LOG-05** | Rp 2.000.000.000 | Rp 2.400.000.000 | +20,0% | **Diatas Pasar / Mahal** ($\Delta > +10\%$) |

### B. Pengujian Logika Rumus Bunga Anuitas KPR
Perhitungan angsuran bulanan ($A$) dari plafon pinjaman pokok ($P$), suku bunga bulanan ($i = \text{bunga tahunan} / 12$), dan total bulan pembayaran ($n = \text{tenor} \times 12$):

$$A = P \times \frac{i(1 + i)^n}{(1 + i)^n - 1}$$

| Parameter Uji | Kasus Uji Standar KPR | Toleransi Selisih |
| :--- | :--- | :--- |
| **Harga Rumah / DP** | Rp 1.000.000.000 / Uang Muka 20% (Plafon $P$ = Rp 800.000.000) | - |
| **Suku Bunga / Tenor** | Bunga 6% per tahun ($i = 0,005$ per bulan) / Tenor 15 tahun ($n = 180$ bulan) | - |
| **Hasil Hitung Sistem** | Rp 6.750.707 per bulan | - |
| **Nilai Benchmark Bank**| Rp 6.750.707 per bulan | **Rp 0 (Presisi Identik)** |

### C. Pengujian Kelayakan Rasio Finansial (Debt Service Ratio - DSR)
Pengujian kapasitas pembayaran angsuran terhadap pendapatan bulanan debitur berdasarkan standar OJK[cite: 2, 6]:

$$\text{DSR} = \frac{\text{Angsuran Bulanan}}{\text{Pendapatan Bersih Bulanan}} \times 100\%$$

| Kode Uji | Angsuran Bulanan | Pendapatan Bersih | DSR Terhitung | Hasil Evaluasi Sistem |
| :--- | :--- | :--- | :--- | :--- |
| **TC-LOG-06** | Rp 8.420.000 | Rp 25.000.000 | 33,68% | **Lencana "Ideal"** (DSR $\le 40\%$, kapasitas sangat sehat)[cite: 2] |
| **TC-LOG-07** | Rp 8.420.000 | Rp 21.050.000 | 40,00% | **Lencana "Ideal"** (Batas ambang maksimum OJK 40%)[cite: 2] |
| **TC-LOG-08** | Rp 8.420.000 | Rp 15.000.000 | 56,13% | **Peringatan Risiko** (DSR $> 40\%$, melebihi batas aman kredit)[cite: 2] |

---

## 4. Pengujian Integrasi & Kinerja API (Integration & Performance QA)

Pengujian dilakukan menggunakan **Postman Collection Runner** pada endpoint inferensi utama:

* **Target Endpoint:** `POST /api/v1/predict`
* **Volume Eksekusi:** 50 permintaan berturut-turut (*sequential iteration*)
* **Kriteria Keberhasilan Kinerja:** Rata-rata waktu latensi inferensi backend $< 1.500$ ms (1,5 detik).
* **Pengujian Skema Galat:**
  * Payload JSON tanpa atribut `surface_area` wajib mengembalikan kode status **`400 Bad Request`** beserta rincian validasi.
  * Method selain `POST` (seperti `GET` pada `/api/v1/predict`) wajib mengembalikan kode status **`405 Method Not Allowed`**.

---

## 5. Pengujian Pengalaman Pengguna (Usability & Responsiveness Testing)

Pengujian dieksekusi langsung pada perangkat fisik Android:

1. **Uji Bebas Overflow:** Memastikan form input, kartu hasil estimasi, kartu cek kewajaran, dan kalkulator KPR bebas dari galat *RenderFlex overflow* saat keyboard virtual aktif.
2. **Uji Navigasi Antarmuka:** Alur perpindahan dari Beranda $\rightarrow$ Form Prediksi $\rightarrow$ Layar Hasil & Cek Kewajaran $\rightarrow$ Kalkulator KPR $\rightarrow$ Riwayat Lokal berjalan lancar tanpa *lag*.
3. **Uji Mode Offline:** Memastikan fitur Kalkulator KPR dan pembacaan riwayat estimasi lokal tetap dapat diakses penuh saat perangkat berada dalam mode pesawat (*offline*).

---

## 6. Prosedur Pelacakan Cacat (Defect Tracking) & Format STR

### A. Siklus Hidup Cacat (Defect Lifecycle)
`Tercatat (Open)` $\rightarrow$ `Diperiksa PM (Assigned)` $\rightarrow$ `Diperbaiki Developer (Fixed)` $\rightarrow$ `Diverifikasi QA (Retested)` $\rightarrow$ `Selesai (Closed)`

### B. Format Tabel Defect Tracking Log

| Defect ID | Tanggal Temuan | Modul | Deskripsi Kendala | Tingkat Keparahan | Penanggung Jawab | Status Akhir |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DEF-01** | 2026-10-10 | Mobile Form | Masukan angka desimal koma menyebabkan crash parsing integer | High | Hanif | Closed |
| **DEF-02** | 2026-10-12 | API Backend | Latensi melebihi 2 detik karena model dimuat ulang pada setiap request | Critical | Nuril | Closed |
| **DEF-03** | 2026-10-15 | KPR Calc | Bunga 0% menghasilkan error division by zero | Medium | Naufal | Closed |

### C. Kerangka Dokumen Software Test Report (STR)
1. **Ringkasan Pengujian:** Total test cases yang dieksekusi, jumlah lulus (*Passed*), dan jumlah gagal (*Failed*).
2. **Metrik Keberhasilan:** Bukti evaluasi metrik model Machine Learning ($R^2 \ge 0,80$, MAE, RMSE).
3. **Hasil Uji Beban & Latensi:** Grafik distribusi waktu tanggap API inferensi dari Postman Runner.
4. **Log Penutupan Cacat Mutu:** Rekapitulasi seluruh tiket *defect* yang berhasil ditutup sebelum demonstrasi final.
5. **Kesimpulan Kesiapan Rilis:** Pernyataan formal persetujuan QA Lead untuk peluncuran rilis APK dan backend.