# Panduan Sistem Desain (Design System) - HousePrice

Dokumen ini mendefinisikan spesifikasi visual, token desain, penataan komponen form, serta standarisasi widget Flutter yang diselaraskan dengan estetika visual *mockup* antarmuka serta ruang lingkup fungsional resmi HousePrice (tanpa autentikasi cloud, 3 tab navigasi utama, serta penambahan komponen Cek Kewajaran Harga dan penyimpanan riwayat lokal)[cite: 1, 2, 3, 6, 7].

---

## 1. Fondasi Token Warna (Color Tokens)

Skema warna mengadopsi palet modern berlatar terang (*clean light theme*) dengan kontras tinggi, memanfaatkan perpaduan warna Midnight Navy dan Electric Blue gradasi[cite: 1, 7].

```
                                  PALET WARNA RESMI
                                  
  [ #0B192E ] Dark Navy (Primary)         [ #0A58CA ] Electric Blue (Accent)
  [ #10B981 ] Success Green (Murah/Ideal) [ #EF4444 ] Alert Red (Mahal/Batas DSR)
  [ #F4F7FA ] Background Canvas          [ #EFF2F7 ] Input Field Background
```

### A. Palet Warna Utama & Brand
* **Primary Dark Navy (`#0B192E`):** Warna kontainer gelap, kartu grafik pasar pada Beranda, header, dan teks tombol utama[cite: 2, 7].
* **Primary Blue Accent (`#0A58CA`):** Warna aksen interaktif, teks nominal hasil estimasi, tab aktif, dan penanda fokus form[cite: 1, 3].
* **Gradient Button Primary:** Gradasi horizontal linear dari Navy pekat ke Biru Royal (`#001D4A` $\rightarrow$ `#0A58CA`) untuk tombol *Call-to-Action* (CTA)[cite: 1, 6].
* **App Canvas Background (`#F4F7FA`):** Warna latar belakang layar utama aplikasi[cite: 1, 2].
* **Card Container (`#FFFFFF`):** Latar belakang permukaan kartu informasi dengan radius 16px dan bayangan tipis[cite: 1, 2].
* **Input Surface (`#EFF2F7`):** Latar belakang bidang isian teks, dropdown, dan stepper counter[cite: 1, 6].

### B. Indikator Status & Semantik (Status Evaluation)
Warna ini digunakan secara konsisten untuk lencana status evaluasi harga pasar dan rasio finansial[cite: 2, 3]:
* **Murah / Nilai Baik / DSR Aman (`#10B981` / background `#E6F7F0`):** Menandai harga penawaran di bawah pasar wajar ($\Delta < -10\%$) serta rasio cicilan DSR kategori "Ideal" ($\le 40\%$)[cite: 2].
* **Wajar / Sesuai Pasar (`#0A58CA` / background `#EBF3FF`):** Menandai harga penawaran yang berada dalam rentang toleransi pasar ($-10\% \le \Delta \le +10\%$)[cite: 3].
* **Mahal / Di Atas Pasar (`#EF4444` / background `#FEE2E2`):** Menandai harga penawaran penjual yang melebihi batas toleransi ($\Delta > +10\%$) atau rasio cicilan yang melampaui batas aman finansial[cite: 2].

---

## 2. Tipografi (Typography)

Aplikasi menggunakan jenis huruf **`Plus Jakarta Sans`** dengan hierarki ukuran dan ketebalan berikut[cite: 1, 2]:

| Tipe Gaya Teks | Ukuran | Bobot (Weight) | Penerapan pada Layar |
| :--- | :--- | :--- | :--- |
| **Display Nominal** | 30sp | ExtraBold (`w800`) | Angka cicilan per bulan (`Rp 8.420.000 / bln`)[cite: 2] |
| **Headline Bold** | 22sp | Bold (`w700`) | Judul halaman: "Estimasi Biaya Properti", "Kalkulator KPR"[cite: 1, 6] |
| **Section Header** | 16sp | Bold (`w700`) | Judul kartu seksi form ("Wilayah & Lokasi", "Tata Ruang")[cite: 1] |
| **Input Content** | 15sp | SemiBold (`w600`) | Nilai teks di dalam input box dan dropdown[cite: 1, 6] |
| **Body Regular** | 13sp – 14sp | Regular (`w400`) | Sub-heading penjelasan dan deskripsi kartu[cite: 1, 7] |
| **Caption & Units** | 11sp – 12sp | Medium (`w500`) | Penanda satuan ($m^2$, Tahun), lencana pil, dan teks disclaimer[cite: 1, 2] |

---

## 3. Komponen Form & Penataan Kartu (Form Layout Pattern)

Penataan komponen masukan mengelompokkan input ke dalam kartu-kartu terpisah (*card-grouped inputs*) berlatar putih dengan label pendukung di pojok kanan atas[cite: 1]:

### A. Struktur Kartu Form Input Properti (`predict_form_screen.dart`)
1. **Seksi 1: Wilayah & Lokasi** (Header icon: Pin Peta | Badge: "DKI Jakarta")[cite: 1]
   * Dropdown Kota Administrasi (5 Wilayah Jakarta)[cite: 1].
   * Dropdown Kecamatan (Dinamis sesuai kota terpilih)[cite: 1].
2. **Seksi 2: Tata Ruang, Kamar & Garasi/Carport** (Header icon: Denah | Badge: "Spesifikasi")[cite: 1]
   * Field Jumlah Kamar Tidur (Prefix icon: Tempat tidur)[cite: 1].
   * Field Jumlah Kamar Mandi (Prefix icon: Shower/Bak mandi)[cite: 1].
   * Field Kapasitas Garasi/Carport (Prefix icon: Mobil)[cite: 1].
3. **Seksi 3: Dimensi Luas Tanah & Bangunan** (Header icon: Penggaris siku | Badge: "Metric (m²)")[cite: 1]
   * Field Luas Tanah (Prefix icon: Bidang luas, Suffix: $m^2$, Subtext validasi: *"Min. 30 m²"*)[cite: 1].
   * Field Luas Bangunan (Prefix icon: Rumah, Suffix: $m^2$, Subtext validasi: *"Akumulasi seluruh lantai"*)[cite: 1].

### B. Struktur Kartu Form Kalkulator KPR (`kpr_screen.dart`)
* **Pemilih Periode:** Tab segmen atas ("Bulanan" vs "Tahunan")[cite: 6].
* **Input Finansial:**
  * Nilai Properti / Rumah (Rp)[cite: 6].
  * Nilai Uang Muka / DP (Rp)[cite: 6].
  * Suku Bunga KPR (% per tahun) disertai catatan suku bunga pasar terkini[cite: 6].
  * Jangka Waktu Kredit / Tenor (Tahun)[cite: 6].
  * Pendapatan Bersih Bulanan (Rp)[cite: 6].
* **Kotak Pemandu:** Informasi rasio rekomendasi cicilan maksimal $30\% - 40\%$ dari total pendapatan[cite: 2, 6].

---

## 4. Desain Layar Hasil & Penyesuaian Fitur Kunci

### A. Layar Hasil Estimasi (`result_screen.dart`)
Menata kembali susunan komponen hasil inferensi agar mencakup Fitur Cek Kewajaran secara lengkap[cite: 3]:

```
+-------------------------------------------------------------+
| [<-] Hasil Valuasi Properti                 [Ikon Riwayat]  |
+-------------------------------------------------------------+
| CARD 1: ESTIMASI NILAI PASAR                                |
| Rentang Harga Pasar Wajar:                                  |
| Rp 2,45 M  sampai  Rp 2,80 M                                |
| Harga Rata-Rata Bangunan: Rp 18,5 Jt / m2                   |
+-------------------------------------------------------------+
| CARD 2: CEK KEWAJARAN HARGA PENAWARAN (FITUR 3)             |
| "Masukkan harga penawaran penjual untuk evaluasi"           |
| [ Input Nominal Penawaran: Rp 2.500.000.000 ]               |
| [ Tombol: Evaluasi Kewajaran ]                              |
| Status: [ Lencana Hijau: Wajar / Sesuai Pasar (Deviasi 2%) ]|
+-------------------------------------------------------------+
| CARD 3: RINGKASAN SPESIFIKASI PROPERTI                      |
| Kota: Jakarta Selatan        | Kecamatan: Kebayoran Baru    |
| Luas Tanah: 140 m2           | Luas Bangunan: 120 m2        |
| Kamar Tidur: 3               | Kamar Mandi: 2               |
+-------------------------------------------------------------+
| [ Tombol Putih: Hitung Ulang ]  [ Tombol Putih: Simpan ]    |
| [ TOMBOL GRADASI NAVY-BLUE: Bagikan Estimasi Nilai ]        |
+-------------------------------------------------------------+
```

### B. Layar Hasil KPR (`kpr_result_screen.dart`)
* **Pembersihan Out-of-Scope:** Tombol utama yang semula *"Ajukan Pre-Approval KPR"* diubah menjadi **`[ Simpan Hasil Simulasi ke Riwayat ]`** yang menyimpan rincian ke SQLite lokal[cite: 2].
* **Kartu Komposisi Angsuran:** Visualisasi bar proporsi persentase Pokok vs Bunga, ringkasan Total Pinjaman Pokok, Total Estimasi Bunga, dan Total Beban Pengeluaran KPR[cite: 2].
* **Status DSR:** Kotak evaluasi kapasitas pembayaran dengan badge lencana "Ideal" jika DSR $\le 40\%$[cite: 2].

---

## 5. Sistem Navigasi & Struktur Berkas Flutter

Aplikasi menggunakan **3 Tab Navigasi Bawah Konsisten** di seluruh alur utama, tanpa tab profil mandiri[cite: 1, 6, 7]:
1. **Beranda:** Menampilkan statistik rata-rata harga per meter persegi, diagram batang per wilayah, dan tombol pintas navigasi[cite: 7].
2. **Estimasi:** Form pengisian spesifikasi properti untuk memprediksi harga pasar[cite: 1].
3. **Simulasi KPR:** Kalkulator angsuran bulanan dan evaluasi kelayakan kredit berdasarkan penghasilan[cite: 6].

> **Akses Riwayat Lokal:** Ikon dokumen/jam pada pojok kanan atas AppBar di Beranda dan Layar Hasil berfungsi untuk membuka lembar riwayat simulasi tersimpan dari SQLite lokal[cite: 3, 5, 7].

### Pemetaan Berkas Implementasi Flutter (`houseprice-mobile-app`):
```text
lib/
├── core/
│   ├── theme/
│   │   ├── app_colors.dart        # Definisi kode hex palet warna
│   │   └── app_typography.dart    # Konfigurasi TextStyle Plus Jakarta Sans
│   └── widgets/
│       ├── gradient_button.dart   # Tombol gradasi Navy ke Electric Blue
│       ├── custom_text_field.dart # Form field rounded grey dengan prefix icon
│       └── status_badge.dart      # Lencana pil status (Murah/Wajar/Mahal/Ideal)
└── presentation/
    ├── screens/
    │   ├── main_navigation.dart   # Scaffold pembungkus 3 tab BottomNavBar
    │   ├── home/
    │   │   └── home_screen.dart   # Slicing grafik & statistik pasar
    │   ├── predict/
    │   │   ├── predict_form_screen.dart
    │   │   └── result_screen.dart # Hasil valuasi & komponen Cek Kewajaran
    │   ├── kpr/
    │   │   ├── kpr_form_screen.dart
    │   │   └── kpr_result_screen.dart
    │   └── history/
    │       └── history_bottom_sheet.dart # Daftar riwayat tersimpan lokal
```