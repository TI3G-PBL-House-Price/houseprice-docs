# Panduan Sistem Desain (Design System) - HousePrice

Dokumen ini mendefinisikan spesifikasi visual, token desain, hierarki tipografi, penataan komponen antarmuka, serta standarisasi widget Flutter yang diselaraskan dengan arsitektur *Strict Gate Authentication*, 5 tab navigasi utama (Beranda, Prediksi, KPR, Notifikasi, Profil), alur Splash & Onboarding, serta tata kelola riwayat terintegrasi cloud[cite: 3, 8].

---

## 1. Fondasi Token Warna (Color Tokens)

Skema warna mengadopsi palet modern berlatar terang (*clean light theme*) dengan kontras tinggi, memanfaatkan perpaduan warna Midnight Navy dan Electric Blue gradasi[cite: 8].

```
                                  PALET WARNA RESMI
                                  
  [ #0B192E ] Dark Navy (Primary)         [ #0A58CA ] Electric Blue (Accent)
  [ #10B981 ] Success Green (Murah/Ideal) [ #EF4444 ] Alert Red (Mahal/Batas DSR)
  [ #F4F7FA ] Background Canvas           [ #EFF2F7 ] Input Field Background
  [ #E2E8F0 ] Border / Divider Line       [ #F59E0B ] Warning Amber (Notifikasi)
```

### A. Palet Warna Utama & Brand
* **Primary Dark Navy (`#0B192E`):** Warna kontainer gelap, kartu grafik pasar pada Beranda, header navigasi, dan teks judul utama[cite: 8].
* **Primary Blue Accent (`#0A58CA`):** Warna aksen interaktif, teks nominal hasil estimasi, tab aktif, dan penanda fokus form[cite: 8].
* **Gradient Button Primary:** Gradasi horizontal linear dari Navy pekat ke Biru Royal (`#001D4A` $\rightarrow$ `#0A58CA`) untuk tombol *Call-to-Action* (CTA) pada Landing Page, Login, dan Submit Prediksi[cite: 8].
* **App Canvas Background (`#F4F7FA`):** Warna latar belakang layar utama aplikasi[cite: 8].
* **Card Container (`#FFFFFF`):** Latar belakang permukaan kartu informasi dengan radius 16px dan bayangan tipis (*elevation 1*)[cite: 8].
* **Input Surface (`#EFF2F7`):** Latar belakang bidang isian teks, dropdown, dan input form autentikasi[cite: 8].
* **Border & Inactive Stroke (`#E2E8F0`):** Garis pembatas kartu, *divider*, dan batas isian input saat tidak aktif[cite: 8].

### B. Indikator Status & Semantik (Status Evaluation)
Warna ini digunakan secara konsisten untuk lencana status evaluasi harga pasar, notifikasi, dan rasio finansial[cite: 8]:
* **Murah / Nilai Baik / DSR Aman (`#10B981` / background `#E6F7F0`):** Menandai harga penawaran di bawah pasar wajar ($\Delta < -10\%$) serta rasio cicilan DSR kategori "Ideal" ($\le 40\%$)[cite: 8].
* **Wajar / Sesuai Pasar (`#0A58CA` / background `#EBF3FF`):** Menandai harga penawaran yang berada dalam rentang toleransi pasar ($-10\% \le \Delta \le +10\%$)[cite: 8].
* **Mahal / Di Atas Pasar (`#EF4444` / background `#FEE2E2`):** Menandai harga penawaran penjual yang melebihi batas toleransi ($\Delta > +10\%$) atau rasio cicilan yang melampaui batas aman finansial[cite: 8].
* **Badge Notifikasi Belum Dibaca (`#EF4444`):** Titik merah penanda pesan baru pada ikon lonceng navigasi[cite: 2, 7].
* **Notifikasi Tren & Sistem (`#0A58CA` / background `#EFF6FF`):** Menandai item kartu notifikasi pembaruan pasar atau model ML[cite: 2].

---

## 2. Tipografi (Typography)

Aplikasi menggunakan jenis huruf **`Plus Jakarta Sans`** dengan hierarki ukuran dan ketebalan berikut[cite: 8]:

| Tipe Gaya Teks | Ukuran | Bobot (*Weight*) | Penerapan pada Layar |
| :--- | :--- | :--- | :--- |
| **Display Nominal** | 30sp | ExtraBold (`w800`) | Angka cicilan per bulan (`Rp 8.420.000 / bln`) dan nominal hasil prediksi[cite: 8] |
| **Headline Bold** | 22sp | Bold (`w700`) | Judul layar: "Selamat Datang", "Estimasi Biaya Properti", "Kalkulator KPR"[cite: 8] |
| **Section Header** | 16sp | Bold (`w700`) | Judul seksi form ("Wilayah & Lokasi", "Informasi Akun")[cite: 8] |
| **Input Content** | 15sp | SemiBold (`w600`) | Nilai teks di dalam input box, dropdown, dan form auth[cite: 8] |
| **Body Regular** | 13sp – 14sp | Regular (`w400`) | Paragraf penjelasan onboarding, deskripsi notifikasi, dan subteks kartu[cite: 8] |
| **Caption & Units** | 11sp – 12sp | Medium (`w500`) | Penanda satuan ($m^2$, Tahun), lencana pil status, dan teks disclaimer[cite: 8] |

---

## 3. Komponen Antarmuka Modul Baru (Pre-Auth & Account)

### A. Splash & Onboarding Screen (`onboarding/`)
1. **Splash Screen:**
   * Logo HousePrice berukuran $120 \times 120$ dp terpusat secara vertikal.
   * Indikator pemuatan (*circular progress indicator*) kecil berwarna Electric Blue (`#0A58CA`) di bagian bawah[cite: 8].
2. **Onboarding Carousel Screen:**
   * **Ilustrasi / Vektor Visual:** Proporsi 45% tinggi layar di area atas.
   * **Slide Indicators:** 3 indikator titik (*dot indicators*) aktif memanjang ($24 \times 8$ dp warna Electric Blue) dan tidak aktif bulat ($8 \times 8$ dp warna abu-abu netral).
   * **Teks Edukasi Fitur:** Headline Bold (22sp) dan Body Regular (14sp) terpusat[cite: 8].
   * **Aksi Navigasi:** Tombol teks *"Lewati"* di pojok kanan atas AppBar dan tombol CTA *"Mulai Sekarang"* di bagian bawah.

### B. Landing Page (`landing_screen.dart`)
* **Header Ilustrasi Properti:** Visualisasi rumah modern Jakarta dengan latar belakang gradasi halus.
* **Teks Nilai Tambah:** Judul utama *"Solusi Cerdas Menilai Properti Idaman"* disertai subteks pengantar.
* **Tombol Aksi Utama:**
  * Tombol Masuk: `GradientButton` (Navy ke Blue) bertuliskan *"Masuk ke Akun"*[cite: 8].
  * Tombol Daftar: Tombol berkontur (*OutlinedButton*) berlatar putih dengan bingkai Dark Navy bertuliskan *"Daftar Akun Baru"*.

### C. Formulir Autentikasi (`login_screen.dart` & `register_screen.dart`)
* **Input Box Standar:**
  * Background `#EFF2F7`, border radius 12px, tanpa border tegas saat pasif, border Electric Blue 1.5px saat aktif (*focused*)[cite: 8].
  * Prefix Icon: Ikon Mail (`Icons.email_outlined`) untuk field email, Ikon Gembok (`Icons.lock_outline`) untuk field kata sandi.
  * Suffix Icon: Ikon toggle mata (`Icons.visibility` / `Icons.visibility_off`) khusus field kata sandi untuk menyamarkan teks.
* **Pemberitahuan Galat:** Teks peringatan berwarna Alert Red (`#EF4444`, 12sp) di bawah field yang gagal divalidasi[cite: 7, 8].

---

## 4. Komponen Form Spesifikasi Properti & Hasil

### A. Struktur Kartu Form Input Properti (`predict_form_screen.dart`)
Penataan komponen masukan mengelompokkan input ke dalam kartu-kartu terpisah (*card-grouped inputs*) berlatar putih dengan label pendukung di pojok kanan atas[cite: 8]:
1. **Seksi 1: Wilayah & Lokasi** (Header icon: Pin Peta | Badge: "DKI Jakarta")[cite: 8]
   * Dropdown Kota Administrasi (5 Wilayah Jakarta)[cite: 8].
   * Dropdown Kecamatan (Dinamis sesuai kota terpilih)[cite: 8].
2. **Seksi 2: Tata Ruang, Kamar & Garasi/Carport** (Header icon: Denah | Badge: "Spesifikasi")[cite: 8]
   * Field Jumlah Kamar Tidur (Prefix icon: Tempat tidur)[cite: 8].
   * Field Jumlah Kamar Mandi (Prefix icon: Shower/Bak mandi)[cite: 8].
   * Field Kapasitas Garasi/Carport (Prefix icon: Mobil)[cite: 8].
3. **Seksi 3: Dimensi Luas Tanah & Bangunan** (Header icon: Penggaris siku | Badge: "Metric (m²)")[cite: 8]
   * Field Luas Tanah (Prefix icon: Bidang luas, Suffix: $m^2$, Subtext validasi: *"Min. 30 m²"*)[cite: 8].
   * Field Luas Bangunan (Prefix icon: Rumah, Suffix: $m^2$, Subtext validasi: *"Akumulasi seluruh lantai"*)[cite: 8].

### B. Layar Hasil Estimasi (`result_screen.dart`)
Menata susunan komponen hasil inferensi model ML dan evaluasi kelayakan harga penawaran[cite: 8]:

```
+-------------------------------------------------------------+
| [<-] Hasil Valuasi Properti         [Riwayat] [Masuk (Tamu)]|
+-------------------------------------------------------------+
| CARD 1: ESTIMASI NILAI PASAR (HASIL MACHINE LEARNING)       |
| Rentang Harga Pasar Wajar:                                  |
| Rp 2,45 M  sampai  Rp 2,80 M                                |
| Harga Rata-Rata Bangunan: Rp 18,5 Jt / m2                   |
| Status Pencatatan: [ Otomatis Tersimpan di Riwayat ]        |
+-------------------------------------------------------------+
| CARD 2: CEK KEWAJARAN HARGA PENAWARAN                       |
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
| [ TOMBOL GRADASI NAVY-BLUE: Hitung Ulang ]                  |
+-------------------------------------------------------------+
```
*(Catatan: Tombol "Simpan" dan "Bagikan Estimasi Nilai" telah dihapus; sistem otomatis menyimpan setiap perhitungan ke riwayat kalkulasi pengguna saat tombol kalkulasi/prediksi ditekan).*

### C. Layar Hasil Simulasi KPR (`kpr_result_screen.dart`)
* **Hero Angsuran:** Kartu gelap Navy dengan estimasi cicilan per bulan (`Rp 8.420.000 / bulan`).
* **Komposisi Angsuran:** Visualisasi bar proporsi pokok (66,2%) vs bunga (33,8%) serta rincian total beban pembiayaan.
* **Evaluasi DSR:** Kartu kapasitas bayar dengan rasio Debt Service Ratio dan lencana status Ideal ($\le 40\%$).
* **Tombol Aksi Utama:** Tombol tunggal **"Hitung Ulang"** (`GradientButton`) dengan ikon `Icons.refresh_rounded` untuk kembali ke formulir kalkulator.

### D. Akses Riwayat Kalkulasi & Proteksi Autentikasi (`history_bottom_sheet.dart`)
* **Penyimpanan Otomatis (*Auto-Save*):** Setiap kali formulir prediksi atau simulasi KPR disubmit, data perhitungan otomatis dicatat ke riwayat teratas (`addRecord`) tanpa perlu tombol simpan manual.
* **Proteksi Autentikasi (*Auth Guard*):** Pengguna wajib terautentikasi (login) untuk membuka riwayat. Jika pengguna anonim/tamu menekan ikon riwayat, sistem menampilkan SnackBar edukatif 2 detik dan mengarahkan pengguna ke halaman Login.

### E. Layar Notifikasi & Profil Pengguna
1. **Layar Notifikasi (`notification_screen.dart`):**
   * *List Tile Notifikasi:* Kartu putih berlatar belakang biru lembut jika pesan belum dibaca (`#EFF6FF`), dan berlatar putih penuh jika sudah dibaca[cite: 8].
   * Informasi memuat ikon kategori, judul notifikasi (SemiBold 14sp), deskripsi pesan (Regular 13sp), serta stempel waktu (Caption 11sp)[cite: 8].
2. **Layar Profil Pengguna (`profile_screen.dart`):**
   * **Header Profil:** Avatar inisial nama berlatar Electric Blue, nama lengkap pengguna (Bold 18sp), dan alamat email akun (Regular 13sp)[cite: 8].
   * **Kartu Riwayat Prediksi Personal:** Menampilkan daftar kartu ringkasan inferensi sebelumnya yang diambil dari cloud API (`GET /api/v1/predict/history`)[cite: 2, 4].
   * **Menu Aksi:** Tombol Ubah Kata Sandi dan Tombol Logout berwarna Alert Red (`#EF4444`) dengan dialog konfirmasi[cite: 7, 8].

---

## 5. Sistem Navigasi & Struktur Berkas Flutter

Aplikasi menggunakan **5 Tab Navigasi Bawah Konsisten (Bottom Navigation Bar)** untuk seluruh alur pengguna terdaftar[cite: 3]:
1. **Beranda:** Menampilkan grafik tren rata-rata harga pasar dan filter wilayah Jakarta[cite: 8].
2. **Prediksi:** Formulir spesifikasi rumah untuk kalkulasi taksiran harga pasar berbasis ML[cite: 8].
3. **KPR:** Kalkulator simulasi angsuran pembiayaan perbankan berbasis metode bunga anuitas[cite: 8].
4. **Notifikasi:** Daftar pemberitahuan pembaruan pasar dan sistem, dilengkapi *badge counter*[cite: 2, 7].
5. **Profil:** Rincian akun pengguna terdaftar, riwayat kalkulasi cloud, dan pengaturan keamanan[cite: 2, 3].

### Pemetaan Berkas Implementasi Flutter (`houseprice-mobile-app`):
```text
lib/
├── core/
│   ├── network/
│   │   ├── api_client.dart            # Inisialisasi Dio
│   │   └── auth_interceptor.dart      # Injeksi Bearer JWT & penanganan 401
│   ├── storage/
│   │   └── secure_storage_helper.dart # Wrapper flutter_secure_storage
│   ├── theme/
│   │   ├── app_colors.dart            # Definisi kode hex palet warna
│   │   └── app_typography.dart        # Konfigurasi TextStyle Plus Jakarta Sans
│   └── widgets/
│       ├── gradient_button.dart       # Tombol gradasi Navy ke Electric Blue
│       ├── custom_text_field.dart     # Form field rounded grey dengan prefix/suffix icon
│       └── status_badge.dart          # Lencana pil status (Murah/Wajar/Mahal/Ideal)
└── features/
    ├── onboarding/
    │   └── presentation/screens/
    │       ├── splash_screen.dart     # Cek sesi token & flag first launch
    │       ├── onboarding_screen.dart # 3 Slide Carousel interaktif
    │       └── landing_screen.dart    # Sambutan & pemilihan login/register
    ├── auth/
    │   ├── data/                      # Repositori & remote data source auth API
    │   ├── domain/                    # Use case: LoginUser, RegisterUser
    │   └── presentation/screens/
    │       ├── login_screen.dart      # Form masuk akun (Strict Gate)
    │       └── register_screen.dart   # Form pendaftaran akun baru
    ├── home/
    │   └── presentation/screens/
    │       └── home_screen.dart       # Grafik statistik pasar Jakarta
    ├── prediction/
    │   └── presentation/screens/
    │       ├── predict_form_screen.dart
    │       └── result_screen.dart     # Valuasi ML & Cek Kewajaran Harga
    ├── kpr/
    │   └── presentation/screens/
    │       ├── kpr_form_screen.dart   # Kalkulator input pembiayaan KPR
    │       └── kpr_result_screen.dart # Rincian angsuran & status DSR
    ├── notification/
    │   └── presentation/screens/
    │       └── notification_screen.dart # Daftar notifikasi tren & sistem
    └── profile/
        └── presentation/screens/
            ├── profile_screen.dart    # Tampilan akun & riwayat cloud
            └── change_password_screen.dart
```