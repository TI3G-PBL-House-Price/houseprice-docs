# Glosarium Istilah Sistem (Glossary) - HousePrice

Dokumen ini memuat definisi baku mengenai istilah domain properti, sains data, rekayasa perangkat lunak, sistem autentikasi, serta penjaminan mutu perangkat lunak (PMPL) yang digunakan secara konsisten di seluruh dokumentasi dan basis kode HousePrice[cite: 11].

---

## 1. Domain Properti & Finansial (Real Estate & Mortgage)

* **Luas Tanah (`surface_area`):** Total ukuran luas bidang tapak tanah properti yang dinyatakan dalam satuan meter persegi ($m^2$)[cite: 1, 11].
* **Luas Bangunan (`building_area`):** Total luas lantai fisik bangunan tertutup yang dapat dihuni dalam satuan meter persegi ($m^2$)[cite: 1, 11].
* **Kamar Tidur (`bed`):** Jumlah ruangan yang diperuntukkan secara khusus sebagai tempat tidur pada unit properti[cite: 1, 11].
* **Kamar Mandi (`bath`):** Jumlah fasilitas sanitasi kamar mandi yang tersedia pada unit properti[cite: 1, 11].
* **Carport:** Area parkir kendaraan roda empat semi-terbuka di pekarangan properti (beratap atau tanpa atap)[cite: 1, 11].
* **Lokasi Administratif (`city` & `district`):** Wilayah yurisdiksi geografis properti yang terdaftar di DKI Jakarta, mencakup 5 kota administrasi (`city`) dan 253 area kecamatan/kawasan (`district`)[cite: 1, 11].
* **Harga Penawaran (*Offer Price*):** Nilai nominal riil dalam Rupiah yang diajukan oleh pemilik atau agen properti saat menawarkan unit rumah ke pasar[cite: 1, 11].
* **Nilai Estimasi Pasar (*Fair Market Estimate*):** Nilai taksiran harga wajar properti berbasis data historis yang dihasilkan oleh komputasi model Machine Learning[cite: 2, 11].
* **Deviasi Kewajaran Harga:** Rentang persentase selisih antara harga penawaran riil penjual terhadap hasil taksiran model ML[cite: 7, 11]:
  * *Murah / Di Bawah Pasar:* Selisih deviasi $< -10\%$ dari harga estimasi[cite: 7, 11].
  * *Wajar / Sesuai Pasar:* Selisih deviasi berada pada rentang $-10\%$ hingga $+10\%$ (batas toleransi wajar)[cite: 7, 11].
  * *Mahal / Di Atas Pasar:* Selisih deviasi $> +10\%$ dari harga estimasi[cite: 7, 11].
* **Kredit Pemilikan Rumah (KPR):** Fasilitas pembiayaan kredit perbankan jangka panjang untuk pembelian unit rumah tinggal[cite: 1, 11].
* **Uang Muka (*Down Payment* / DP):** Pembayaran awal tunai oleh pembeli yang dihitung sebagai persentase tertentu terhadap total nilai transaksi properti[cite: 1, 11].
* **Plafon Kredit:** Besaran nominal pokok pembiayaan bersih yang dipinjamkan oleh bank setelah dikurangi setoran uang muka[cite: 1, 11].
* **Suku Bunga Anuitas:** Metode perhitungan bunga perbankan di mana jumlah angsuran rutin bulanan (gabungan pokok dan bunga) memiliki nilai tetap sepanjang masa kredit berjalan[cite: 1, 7, 11].
* **Tenor:** Jangka waktu periode pelunasan pembiayaan kredit properti yang dinyatakan dalam satuan tahun[cite: 1, 11].
* **Debt Service Ratio (DSR):** Rasio persentase perbandingan antara total beban cicilan bulanan terhadap total pendapatan bersih debitur berdasarkan acuan regulasi OJK (kategori "Ideal" jika DSR $\le 40\%$)[cite: 7, 8].

---

## 2. Sains Data & Machine Learning

* **Target Variable (`price`):** Variabel terikat berupa nilai kontinu harga jual rumah dalam nominal Rupiah yang menjadi target inferensi algoritma regresi[cite: 1, 11].
* **Predictor Features ($X$):** Kumpulan atribut input independen (`surface_area`, `building_area`, `bed`, `bath`, `carport`, `city`, `district`) yang diproses oleh model regresi[cite: 1, 11].
* **Analisis Data Eksploratif (*Exploratory Data Analysis* / EDA):** Investigasi awal dataset menggunakan ringkasan statistik deskriptif dan grafik sebaran untuk mengenali pola korelasi dan distribusi data properti[cite: 9, 11].
* **Imputasi Nilai Kosong (*Missing Value Imputation*):** Prosedur pengisian baris data yang kosong pada atribut luas tanah dan luas bangunan menggunakan estimasi statistik[cite: 9, 11].
* **Pembersihan Data Duplikat (*Duplicate Removal*):** Proses pendeteksian dan eliminasi baris data properti yang identik pada dataset mentah[cite: 9, 11].
* **Interquartile Range (IQR):** Metode pemfilteran statistik untuk mendeteksi dan mengeliminasi data pencilan (*outliers*) ekstrem pada atribut harga dan dimensi properti berdasarkan rentang kuartil atas ($Q_3$) dan kuartil bawah ($Q_1$)[cite: 9, 11].
* **One-Hot Encoding (OHE):** Transformasi fitur kategorikal non-numerik (kota dan kecamatan) menjadi representasi vektor biner bernilai 0 atau 1[cite: 3, 11].
* **StandardScaler:** Prosedur normalisasi fitur numerik kontinu agar memiliki nilai rata-rata (*mean*) 0 dan standar deviasi 1[cite: 3, 11].
* **Train-Test Split:** Pembagian dataset secara acak terkontrol dengan proporsi 80% untuk data pelatihan (*training set*) dan 20% untuk data evaluasi validasi (*test set*)[cite: 1, 11].
* **Random Forest Regressor:** Algoritma pembelajaran mesin berbasis himpunan pohon keputusan (*ensemble decision trees*) yang menggabungkan banyak estimator untuk menghasilkan taksiran regresi non-linear yang stabil[cite: 1, 9, 11].
* **Serialisasi Model (`.pkl`):** Penyimpanan representasi biner arsitektur model dan parameter bobot yang telah dilatih menggunakan pustaka `joblib` agar dapat dimuat seketika di backend Flask[cite: 9, 11].
* **Koefisien Determinasi ($R^2$ Score):** Metrik evaluasi yang mengukur persentase variasi nilai target harga yang mampu dijelaskan oleh variabel fitur model (kriteria kelulusan target: $R^2 \ge 0,80$)[cite: 1, 9, 11].
* **Mean Absolute Error (MAE):** Rata-rata selisih nilai absolut antara harga penawaran riil aktual dengan estimasi komputasi model dalam satuan Rupiah[cite: 1, 9, 11].
* **Root Mean Squared Error (RMSE):** Akar kuadrat dari rerata kuadrat selisih prediksi yang memberikan penalti lebih tinggi pada galat prediksi berskala besar[cite: 1, 9, 11].

---

## 3. Rekayasa Perangkat Lunak, Arsitektur & Antarmuka Mobile

* **Strict Gate Authentication:** Model keamanan gerbang ketat di mana seluruh fungsionalitas utama aplikasi (Beranda, Estimasi ML, Cek Kewajaran, KPR, Notifikasi, dan Profil) diproteksi penuh dan hanya dapat diakses setelah pengguna memiliki akun serta login aktif[cite: 3, 8].
* **Onboarding Carousel:** Tampilan visual interaktif berbasis geser (*swipe*) pada peluncuran pertama aplikasi yang menyajikan proposisi nilai produk sebelum mengarahkan pengguna ke Landing Page[cite: 7, 8].
* **Landing Page:** Halaman pembuka sebelum otentikasi yang berfungsi sebagai etalase pengantar dan menyediakan navigasi bercabang menuju form Login atau form Pendaftaran akun baru[cite: 7, 8].
* **In-App Notifications:** Sistem penyaluran pemberitahuan perkembangan tren properti di DKI Jakarta dan informasi rilis model yang tersimpan di tabel `notifications` serta dapat ditandai status keterbacaannya oleh pengguna[cite: 2, 8].
* **Navigation Guard / Route Guard:** Komponen penentu alur rute pada aplikasi Flutter yang memeriksa keberadaan token sesi aktif sebelum mengizinkan perpindahan menuju dashboard antarmuka utama[cite: 3, 7].
* **AuthInterceptor:** Komponen middleware pada HTTP Client (Dio) yang otomatis menyematkan header `Authorization: Bearer <token>` pada setiap request privat, serta mencegat respons HTTP 401 Unauthorized untuk memicu prosedur *auto-logout*[cite: 3, 7].
* **Multi-Tenant Data Isolation:** Pola isolasi data relasional di mana entitas privat pengguna (seperti riwayat prediksi dan notifikasi) secara ketat dibatasi pemanggilannya menggunakan klausa `WHERE user_id = :current_user_id` untuk mencegah celah *Insecure Direct Object References* (IDOR)[cite: 5, 6].
* **Inference Latency:** Waktu yang dihabiskan server cloud untuk menerima request HTTP JSON, menjalankan pipeline praproses, mengeksekusi model ML `.pkl`, mencatat log, dan mengembalikan respons ke perangkat (standar batas: $< 1.500$ ms)[cite: 2, 7, 11].
* **RESTful API:** Protokol antarmuka pertukaran data antara aplikasi klien mobile dan server backend Flask menggunakan metode standar HTTP dan muatan berformat JSON terstruktur[cite: 2, 11].

---

## 4. Keamanan, Kriptografi & Infrastruktur Cloud

* **JSON Web Token (JWT):** Standar token kriptografis ringkas (RFC 7519) yang memuat klaim identitas pengguna (`user_id`, `email`, waktu kedaluwarsa `exp`) dan ditandatangani menggunakan algoritma simetris HS256 untuk otentikasi stateless di backend[cite: 2, 4].
* **Bcrypt Password Hashing:** Fungsi *hash* kriptografis satu arah adaptif dengan faktor kerja (*salt rounds*) minimal 12 putaran untuk mengamankan kata sandi dari serangan *brute force* dan *rainbow table*[cite: 1, 4].
* **Flutter Secure Storage:** Pustaka penyimpanan lokal terenkripsi di perangkat Android yang memanfaatkan perlindungan perangkat keras **Android Keystore** (algoritma AES-GCM) untuk menyimpan token JWT[cite: 4, 8].
* **Firebase Cloud Messaging (FCM) Token:** Pengenal unik perangkat mobile yang diterbitkan oleh layanan Google Firebase untuk mendistribusikan pesan *push notification* ke aplikasi klien[cite: 2, 4].
* **PostgreSQL:** Sistem manajemen basis data relasional (*RDBMS*) terpusat yang mengelola 9 tabel entitas (termasuk tabel baru `users` dan `notifications`)[cite: 1, 2, 11].
* **Coolify:** Platform *Platform-as-a-Service* (PaaS) open-source mandiri (*self-hosted*) untuk menjalankan orkestrasi container Docker dan penerbitan sertifikat SSL/TLS HTTPS otomatis berbasis Let's Encrypt[cite: 3, 11].
* **Docker Compose:** Perkakas pendefinisian multi-kontainer untuk mengisolasi layanan backend Flask dan PostgreSQL pada jaringan privat internal (*bridge network*)[cite: 3, 11].

---

## 5. Penjaminan Mutu Perangkat Lunak (PMPL)

* **Boundary Value Analysis (BVA):** Teknik perancangan kasus uji *blackbox* yang berfokus pada batas ekstrem nilai input (misal: luas tanah bernilai 29, 30, dan nilai negatif)[cite: 7, 11].
* **Equivalence Partitioning (EP):** Pembagian kelompok data masukan menjadi partisi kelas valid (diterima oleh sistem) dan partisi kelas tidak valid (ditolak dengan pesan galat)[cite: 7, 11].
* **Usability Testing:** Pengujian pengalaman pengguna secara langsung pada perangkat fisik Android guna memastikan kelancaran gestur *carousel*, kejelasan hierarki teks, dan ketiadaan kendala *RenderFlex overflow*[cite: 7, 8, 11].
* **Software Test Report (STR):** Dokumen rekapitulasi formal hasil pelaksanaan pengujian yang mencakup statistik kasus uji lolos/gagal, bukti metrik model ML, dan kesimpulan kelayakan rilis build aplikasi[cite: 7, 11].
* **Defect Tracking:** Tata kelola pencatatan dan pemantauan siklus hidup kendala perangkat lunak dari status *Open*, *Assigned*, *Fixed*, hingga *Closed*[cite: 7, 11].