# Rencana & Pedoman Pengujian Mutu (Software Testing & QA Plan) - HousePrice

Dokumen ini memuat rencana pengujian, rancangan kasus uji (*test cases*), matriks pengujian fungsional blackbox (BVA & EP), pengujian logika bisnis, pengujian keamanan & integrasi API, pengujian alur navigasi (*Strict Gate Navigation Guard*), serta tata kelola pelaporan cacat perangkat lunak (*Software Test Report*) untuk memenuhi standar mata kuliah Penjaminan Mutu Perangkat Lunak (PMPL) pada proyek HousePrice.

---

## 1. Struktur Tim & Pembagian Tanggung Jawab QA (PMPL)

Setiap anggota tim memiliki peran khusus dalam eksekusi penjaminan mutu perangkat lunak:

| Anggota Tim | NIM | Fokus Pengujian PMPL | Ruang Lingkup & Metode |
| :--- | :--- | :--- | :--- |
| **Nurfinka Lailasari** | 244107020211 | **QA Lead & Defect Manager** | Manajemen Defect Tracking, verifikasi alur Splash/Onboarding, pengujian modul Profil & Riwayat Cloud, serta penyusunan *Software Test Report* (STR). |
| **Jasmine Nasywa Nabilah** | 244107020119 | **Functional Blackbox QA** | Pengujian Blackbox formulir autentikasi (Register & Login) serta form input properti menggunakan metode *Boundary Value Analysis* (BVA) dan *Equivalence Partitioning* (EP). |
| **Abdul Rahman Hanif Darmawan** | 244107020232 | **API, Security & Performance QA** | Pengujian integrasi RESTful API terproteksi JWT, validasi skema payload JSON, uji keamanan token (401 Unauthorized), dan uji latensi inferensi Postman Runner (< 1,5 detik). |
| **Muhammad Nuril Huda** | 244107020004 | **Usability & Navigation QA** | Pengujian *Navigation Guard* (Strict Gate), pengujian interaksi notifikasi in-app, pengujian state management (Bloc/Cubit), dan responsivitas UI pada perangkat fisik Android. |
| **Naufal Abid Aurizky** | 244107020212 | **Business Logic QA** | Pengujian logika bisnis matematis pada formula cicilan KPR metode anuitas, rasio DSR OJK, komparasi deviasi kewajaran harga, dan kalkulasi badge notifikasi. |

---

## 2. Pengujian Blackbox Form Input (EP & BVA)

### A. Formulir Autentikasi Pengguna (Register & Login Screen)

#### Matriks Partisi Kelas (Equivalence Partitioning - EP)
| Parameter Input | Partisi Valid | Partisi Tidak Valid (Ditolak) |
| :--- | :--- | :--- |
| **Nama Lengkap** | String teks alfabet: panjang $3 - 100$ karakter | Kurang dari 3 karakter, string kosong, karakter simbol berbahaya |
| **Alamat Email** | Format email baku (`user@domain.ext`), belum terdaftar di DB | Format tanpa `@`, tanpa domain (`user@.com`), email duplikat |
| **Kata Sandi (Register)** | String minimal 8 karakter, kombinasi huruf & angka | Kurang dari 8 karakter, spasi kosong (*whitespace only*) |
| **Konfirmasi Sandi** | Nilai string identik dengan kolom kata sandi | Nilai string berbeda dari kolom kata sandi |
| **Kredensial Login** | Pasangan email & sandi cocok dengan data di tabel `users` | Email tidak terdaftar, sandi salah, format email tidak valid |

#### Matriks Kasus Uji Nilai Batas (Boundary Value Analysis - BVA)
| Kode Uji | Parameter | Nilai Masukan Uji | Kategori Batas | Hasil yang Diharapkan |
| :--- | :--- | :--- | :--- | :--- |
| **TC-AUTH-01** | Nama Lengkap | `"Ab"` (2 karakter) | Batas bawah tidak valid | Pesan galat: "Nama lengkap minimal 3 karakter". |
| **TC-AUTH-02** | Nama Lengkap | `"Abi"` (3 karakter) | Batas bawah valid | Input diterima, tidak ada pesan galat. |
| **TC-AUTH-03** | Email | `"budi.santoso"` | Format salah (tanpa `@`) | Pesan galat: "Format alamat email tidak valid". |
| **TC-AUTH-04** | Email | `"budi@domain.com"` (duplikat) | Email sudah ada di DB | API mengembalikan 409 Conflict: "Email sudah terdaftar". |
| **TC-AUTH-05** | Kata Sandi | `7` karakter (`Pass12!`) | Batas bawah tidak valid | Pesan galat: "Kata sandi minimal 8 karakter". |
| **TC-AUTH-06** | Kata Sandi | `8` karakter (`Pass123!`) | Batas bawah valid | Input diterima, indikator keamanan valid. |
| **TC-AUTH-07** | Konfirmasi Sandi| Sandi: `Pass123!`, Konfirmasi: `Pass123?` | Nilai tidak cocok | Pesan galat: "Konfirmasi kata sandi tidak cocok". |
| **TC-AUTH-08** | Login Salah | Email benar, sandi keliru | Kredensial tidak valid | API 401 Unauthorized: "Email atau kata sandi salah". |

---

### B. Formulir Spesifikasi Properti (Predict Form Screen)

#### Matriks Partisi Kelas (Equivalence Partitioning - EP)
| Parameter Input | Partisi Valid | Partisi Tidak Valid (Ditolak) |
| :--- | :--- | :--- |
| **Luas Tanah ($m^2$)** | Bilangan riil positif: $x \ge 30$ (Standar kaveling SHM DKI) | $x < 30$, bilangan negatif, karakter huruf/simbol |
| **Luas Bangunan ($m^2$)** | Bilangan riil positif: $x \ge 1$ | $x \le 0$, bilangan negatif, karakter huruf/simbol |
| **Kamar Tidur (`bed`)** | Bilangan bulat non-negatif: $x \in [0, 50]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 50) |
| **Kamar Mandi (`bath`)** | Bilangan bulat non-negatif: $x \in [0, 50]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 50) |
| **Carport** | Bilangan bulat non-negatif: $x \in [0, 30]$ | $x < 0$, desimal, nilai di luar rentang ekstrem (> 30) |
| **Kota & Kecamatan** | Pilihan terdaftar pada daftar *dropdown* master data | Nilai kosong (*null/unselected*), teks manipulasi luar |

#### Matriks Kasus Uji Nilai Batas (Boundary Value Analysis - BVA)
| Kode Uji | Parameter | Nilai Masukan Uji | Kategori Batas | Hasil yang Diharapkan |
| :--- | :--- | :--- | :--- | :--- |
| **TC-BVA-01** | Luas Tanah | `29` | Batas bawah tidak valid | Pesan galat: "Luas tanah minimal 30 m² (standar SHM)". |
| **TC-BVA-02** | Luas Tanah | `30` | Batas bawah valid | Input diterima, form lolos validasi. |
| **TC-BVA-03** | Luas Tanah | `-5` | Nilai negatif ekstrem | Form memunculkan peringatan kesalahan input. |
| **TC-BVA-04** | Luas Bangunan | `0` | Batas bawah tidak valid | Pesan galat: "Luas bangunan minimal 1 m²". |
| **TC-BVA-05** | Luas Bangunan | `1` | Batas bawah valid | Input diterima, tombol prediksi aktif. |
| **TC-BVA-06** | Kamar Tidur | `-1` | Nilai negatif tidak valid | Nilai ditolak, input otomatis di-reset ke 0. |
| **TC-BVA-07** | Kamar Tidur | `0` | Batas bawah valid | Diterima (untuk rumah tipe studio). |
| **TC-BVA-08** | Carport | `0` | Batas bawah valid | Diterima (untuk rumah tanpa garasi/carport). |
| **TC-BVA-09** | Tipe Data | `"abc"` pada kolom luas | Karakter non-numerik | Kolom form tidak mengizinkan pengetikan huruf (*numeric only*). |

---

## 3. Pengujian Alur Navigasi & Sesi (Strict Gate Navigation Guard)

Pengujian ini memastikan tidak ada celah bagi pengguna anonim untuk mengakses fitur privat sebelum melewati autentikasi:

| Kode Uji | Kondisi Awal (*Initial State*) | Aksi Pengguna | Hasil yang Diharapkan (*Expected Outcome*) |
| :--- | :--- | :--- | :--- |
| **TC-NAV-01** | Instalasi Baru (*First Launch*) | Membuka aplikasi | Menampilkan Splash $\rightarrow$ otomatis beralih ke `OnboardingScreen` (3 carousel). |
| **TC-NAV-02** | Selesai Onboarding | Menekan tombol "Mulai Sekarang" / "Lewati" | Nilai `is_first_launch` di SharedPreferences menjadi `false` $\rightarrow$ navigasi ke `LandingScreen`. |
| **TC-NAV-03** | Belum Login (*Unauthenticated*) | Membuka aplikasi kedua kali | Splash screen mendeteksi tidak ada token $\rightarrow$ langsung menuju `LandingScreen`. |
| **TC-NAV-04** | Halaman Landing | Menekan tombol "Masuk" / "Daftar" | Navigasi tepat menuju `LoginScreen` atau `RegisterScreen`. |
| **TC-NAV-05** | Login Berhasil | Kredensial valid dimasukkan | Token JWT tersimpan di `flutter_secure_storage` $\rightarrow$ navigasi ke `MainNavigationScreen` (Beranda). |
| **TC-NAV-06** | Sesi Aktif (*Authenticated*) | Menutup & membuka kembali aplikasi | Splash screen mendeteksi token valid di secure storage $\rightarrow$ *bypass* landing page $\rightarrow$ langsung ke Beranda. |
| **TC-NAV-07** | Token Kedaluwarsa (*Expired*) | Membuka fitur prediksi setelah token expired | API mengembalikan 401 $\rightarrow$ `AuthInterceptor` menghapus token lokal $\rightarrow$ redirect paksa ke `LandingScreen` disertai notifikasi "Sesi telah berakhir". |
| **TC-NAV-08** | Aksi Logout | Memilih opsi "Keluar" pada menu Profil | Dialog konfirmasi muncul $\rightarrow$ token dihapus dari secure storage $\rightarrow$ navigasi kembali ke `LandingScreen`. |

---

## 4. Pengujian Logika Bisnis (Business Logic Testing)

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
Pengujian kapasitas pembayaran angsuran terhadap pendapatan bulanan debitur berdasarkan standar OJK:

$$\text{DSR} = \frac{\text{Angsuran Bulanan}}{\text{Pendapatan Bersih Bulanan}} \times 100\%$$

| Kode Uji | Angsuran Bulanan | Pendapatan Bersih | DSR Terhitung | Hasil Evaluasi Sistem |
| :--- | :--- | :--- | :--- | :--- |
| **TC-LOG-06** | Rp 8.420.000 | Rp 25.000.000 | 33,68% | **Lencana "Ideal"** (DSR $\le 40\%$, kapasitas sangat sehat) |
| **TC-LOG-07** | Rp 8.420.000 | Rp 21.050.000 | 40,00% | **Lencana "Ideal"** (Batas ambang maksimum OJK 40%) |
| **TC-LOG-08** | Rp 8.420.000 | Rp 15.000.000 | 56,13% | **Peringatan Risiko** (DSR $> 40\%$, melebihi batas aman kredit) |

---

## 5. Pengujian Integrasi & Keamanan API (Postman Runner)

Pengujian integrasi dijalankan menggunakan **Postman Collection Runner** dengan rangkaian skenario:

| Kategori API | Endpoint | Kondisi Uji | Status HTTP | Kriteria Kelulusan |
| :--- | :--- | :--- | :---: | :--- |
| **Auth** | `POST /api/v1/auth/register` | Payload valid, email unik | `201 Created` | Respons memuat `data.id` dan pesan sukses registrasi. |
| **Auth** | `POST /api/v1/auth/register` | Email sudah terdaftar | `409 Conflict` | Respons error format JSON konsisten. |
| **Auth** | `POST /api/v1/auth/login` | Email & password benar | `200 OK` | Mengembalikan `access_token` JWT yang valid. |
| **Auth** | `POST /api/v1/auth/login` | Password salah | `401 Unauthorized` | Akses ditolak, tidak menerbitkan token. |
| **Auth** | `GET /api/v1/auth/me` | Header `Bearer <valid_token>` | `200 OK` | Mengembalikan profil pengguna sesuai pemilik token. |
| **Strict Gate** | `POST /api/v1/predict` | Tanpa Header Authorization | `401 Unauthorized` | Request inferensi ditolak sebelum komputasi ML dimulai. |
| **Strict Gate** | `POST /api/v1/predict` | Token palsu / tampered | `401 Unauthorized` | Server memverifikasi integritas HS256 dan menolak request. |
| **ML Inference**| `POST /api/v1/predict` | Header Bearer valid + JSON valid | `200 OK` | Latensi $< 1.500$ ms, nilai prediksi tercatat dengan `user_id` di `prediction_logs`. |
| **History** | `GET /api/v1/predict/history`| Header Bearer Pengguna A | `200 OK` | Hanya mengembalikan daftar riwayat milik Pengguna A (*data isolation*). |
| **Notifikasi** | `GET /api/v1/notifications` | Header Bearer valid | `200 OK` | Menampilkan array pesan notifikasi beserta `unread_count`. |
| **Notifikasi** | `PATCH /notifications/:id/read`| ID notifikasi milik Pengguna B diakses oleh A | `404 Not Found` | Proteksi IDOR berhasil, mencegah manipulasi pesan antar-pengguna. |

---

## 6. Pengujian Pengalaman Pengguna (Usability & Responsiveness)

Pengujian dieksekusi langsung pada perangkat fisik Android:

1. **Uji Bebas Overflow:** Memastikan layar Onboarding, form Login, form Register, form Prediksi, dan layar Profil bebas dari galat *RenderFlex overflow* saat keyboard virtual terbuka.
2. **Uji Transisi Carousel Onboarding:** Gestur geser (*swipe*) kartu perkenalan berjalan halus pada 60 fps dan indikator titik (*dot indicators*) sinkron dengan halaman aktif.
3. **Uji Penanganan Sinyal Buruk (*Network Timeout*):** Memastikan aplikasi menampilkan SnackBar pemberitahuan saat koneksi internet terputus di tengah proses login atau komputasi prediksi, tanpa memicu layar merah crash.
4. **Uji Mode Gelap/Terang (*Theme Adaptation*):** Komponen input teks dan kartu statistik beranda memiliki kontras warna yang memenuhi standar keterbacaan WCAG AA.

---

## 7. Prosedur Pelacakan Cacat (Defect Tracking) & Format STR

### A. Siklus Hidup Cacat (Defect Lifecycle)
`Tercatat (Open)` $\rightarrow$ `Diperiksa PM (Assigned)` $\rightarrow$ `Diperbaiki Developer (Fixed)` $\rightarrow$ `Diverifikasi QA (Retested)` $\rightarrow$ `Selesai (Closed)`

### B. Format Tabel Defect Tracking Log

| Defect ID | Tanggal Temuan | Modul | Deskripsi Kendala | Tingkat Keparahan | Penanggung Jawab | Status Akhir |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **DEF-01** | 2026-10-10 | Mobile Form | Masukan angka desimal koma menyebabkan crash parsing integer | High | Hanif | Closed |
| **DEF-02** | 2026-10-12 | API Backend | Latensi melebihi 2 detik karena model dimuat ulang pada setiap request | Critical | Nuril | Closed |
| **DEF-03** | 2026-10-15 | KPR Calc | Bunga 0% menghasilkan error division by zero | Medium | Naufal | Closed |
| **DEF-04** | 2026-10-26 | Mobile Auth | Token JWT tidak terhapus dari secure storage saat logout | High | Finka | Closed |
| **DEF-05** | 2026-10-26 | API Auth | Endpoint `/predict` dapat diakses tanpa menyertakan header token | Critical | Hanif | Closed |
| **DEF-06** | 2026-10-26 | Mobile UI | Carousel onboarding mengalami overflow pada layar beresolusi kecil | Medium | Jasmine | Closed |

### C. Kerangka Dokumen Software Test Report (STR)
1. **Ringkasan Eksekutif Pengujian:** Total skenario yang dieksekusi, rasio kelulusan (*Passed vs Failed*), serta cakupan modul.
2. **Hasil Uji Keamanan Autentikasi:** Bukti pengujian *Strict Gate*, verifikasi token JWT, dan proteksi isolasi data riwayat prediksi.
3. **Metrik Keberhasilan Model ML:** Rekapitulasi skor evaluasi model aktif ($R^2 \ge 0,80$, MAE, RMSE).
4. **Kinerja & Latensi Layanan:** Hasil pengujian Postman Collection Runner (distribusi waktu tanggap API inferensi $< 1,5$ detik).
5. **Rekapitulasi Tiket Defect:** Status penyelesaian cacat perangkat lunak mulai dari DEF-01 hingga tiket terbaru.
6. **Pernyataan Kesiapan Rilis:** Rekomendasi formal dari QA Lead mengenai kelaikan rilis build APK klien dan server staging backend.