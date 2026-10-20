# Glosarium Istilah Sistem (Glossary) - HousePrice

Dokumen ini memuat definisi baku mengenai istilah domain properti, sains data, rekayasa perangkat lunak, serta penjaminan mutu perangkat lunak (PMPL) yang digunakan secara konsisten di seluruh dokumentasi dan basis kode HousePrice.

---

## 1. Domain Properti & Finansial (Real Estate & Mortgage)

* **Luas Tanah (`surface_area`):** Total ukuran luas bidang tapak tanah properti yang dinyatakan dalam satuan meter persegi ($m^2$)[cite: 1, 24, 27].
* **Luas Bangunan (`building_area`):** Total luas lantai fisik bangunan tertutup yang dapat dihuni dalam satuan meter persegi ($m^2$)[cite: 1, 24, 27].
* **Kamar Tidur (`bed`):** Jumlah ruangan yang diperuntukkan secara khusus sebagai tempat tidur pada unit properti[cite: 1, 24].
* **Kamar Mandi (`bath`):** Jumlah fasilitas sanitasi kamar mandi yang tersedia pada unit properti[cite: 1, 24].
* **Carport:** Area parkir kendaraan roda empat semi-terbuka di pekarangan properti (beratap atau tanpa atap)[cite: 1, 24].
* **Lokasi Administratif (`city` & `district`):** Wilayah yurisdiksi geografis properti yang terdaftar di DKI Jakarta, mencakup 5 kota administrasi (`city`) dan 253 area kecamatan/kawasan (`district`)[cite: 1, 24, 27].
* **Harga Penawaran (*Offer Price*):** Nilai nominal riil dalam Rupiah yang diajukan oleh pemilik atau pengembang properti saat menawarkan unit rumah ke pasar[cite: 24, 25].
* **Nilai Estimasi Pasar (*Fair Market Estimate*):** Nilai taksiran harga properti berbasis data historis yang dihasilkan oleh model Machine Learning[cite: 20, 24, 25].
* **Deviasi Kewajaran Harga:** Rentang persentase selisih antara harga penawaran penjual terhadap estimasi model ML[cite: 25, 40]:
  * *Murah / Di Bawah Pasar:* Selisih deviasi $< -10\%$ dari harga estimasi[cite: 25, 40].
  * *Wajar / Sesuai Pasar:* Selisih deviasi berada pada rentang $-10\%$ hingga $+10\%$[cite: 25, 40].
  * *Mahal / Di Atas Pasar:* Selisih deviasi $> +10\%$ dari harga estimasi[cite: 25, 40].
* **Kredit Pemilikan Rumah (KPR):** Fasilitas pembiayaan kredit perbankan jangka panjang untuk pembelian unit properti[cite: 20, 25].
* **Uang Muka (*Down Payment* / DP):** Pembayaran awal tunai dari pembeli yang dinyatakan dalam persentase tertentu terhadap total harga properti[cite: 24, 25].
* **Plafon Kredit:** Nilai pokok pembiayaan bersih yang dipinjamkan oleh bank setelah dikurangi uang muka[cite: 25].
* **Suku Bunga Anuitas:** Skema perhitungan bunga perbankan di mana jumlah cicilan bulanan (pokok + bunga) bernilai tetap sepanjang masa pinjaman[cite: 25, 26, 37].
* **Tenor:** Jangka waktu periode pelunasan fasilitas pembiayaan kredit properti dalam satuan tahun[cite: 25].

---

## 2. Sains Data & Machine Learning

* **Target Variable (`price`):** Variabel terikat berupa nilai kontinu harga rumah dalam satuan nominal Rupiah yang diprediksi oleh algoritma regresi[cite: 1, 27, 33].
* **Predictor Features ($X$):** Kumpulan atribut independen input (`surface_area`, `building_area`, `bed`, `bath`, `carport`, `city`, `district`) yang menjadi masukan model[cite: 1, 24, 27, 33].
* **Analisis Data Eksploratif (*Exploratory Data Analysis* / EDA):** Investigasi awal dataset menggunakan statistik deskriptif dan grafik sebaran untuk mengenali korelasi serta pola distribusi data[cite: 19, 27, 28, 33].
* **Imputasi Nilai Kosong (*Missing Value Imputation*):** Pengisian baris data atribut yang tidak memiliki nilai (seperti pada `surface_area` dan `building_area`) menggunakan estimasi statistik[cite: 19, 27, 29, 33].
* **Pembersihan Data Duplikat (*Duplicate Removal*):** Proses identifikasi dan eliminasi baris observasi identik pada dataset mentah[cite: 19, 27, 28, 33].
* **Interquartile Range (IQR):** Metode pemfilteran statistik untuk mendeteksi dan membuang data pencilan (*outliers*) ekstrem pada atribut harga dan luas properti berdasarkan selisih kuartil atas ($Q_3$) dan kuartil bawah ($Q_1$)[cite: 27, 29, 33].
* **One-Hot Encoding (OHE):** Konversi fitur non-numerik kategorikal wilayah (`district`, `city`) menjadi matriks representasi biner bernilai 0 atau 1[cite: 27, 29, 33].
* **StandardScaler:** Transformasi normalisasi fitur numerik agar terdistribusi dengan nilai rata-rata (*mean*) 0 dan standar deviasi 1[cite: 27, 29, 33].
* **Train-Test Split:** Pembagian dataset secara terpisah dengan proporsi 80% sebagai data latih (*training set*) dan 20% sebagai data uji validasi (*test set*)[cite: 27, 29, 33].
* **Random Forest Regressor:** Algoritma pemelajaran mesin berbasis himpunan pohon keputusan (*ensemble decision trees*) yang menggabungkan banyak prediksi untuk menghasilkan estimasi nilai non-linear yang stabil[cite: 27, 29, 33].
* **Serialisasi Model (`.pkl`):** Pembekuan struktur model machine learning dan bobot parameter ke dalam berkas biner menggunakan pustaka `joblib` agar dapat dimuat seketika di backend[cite: 27, 29, 33].
* **Koefisien Determinasi ($R^2$ Score):** Metrik statistik yang mengukur proporsi variansi nilai target harga yang mampu dijelaskan oleh variabel independen model (kriteria keberhasilan: $R^2 \ge 0,80$)[cite: 27, 40].
* **Mean Absolute Error (MAE):** Rata-rata selisih mutlak antara harga riil aktual dengan nilai harga taksiran model dalam nominal Rupiah[cite: 27, 29, 33].
* **Root Mean Squared Error (RMSE):** Akar kuadrat dari rata-rata kuadrat selisih prediksi yang memberi bobot lebih tinggi pada kesalahan prediksi berskala besar[cite: 27, 29, 33].

---

## 3. Rekayasa Perangkat Lunak, PMPL, & Arsitektur Cloud

* **Boundary Value Analysis (BVA):** Teknik pengujian *blackbox* yang berfokus pada batas ekstrem nilai input (misal: luas tanah bernilai 0, 1, dan angka batas maksimal)[cite: 19, 27, 29, 31].
* **Equivalence Partitioning (EP):** Pembagian kelas data masukan ke dalam kelompok partisi valid (angka riil positif) dan partisi tidak valid (angka negatif, karakter huruf, simbol)[cite: 19, 27, 29, 31].
* **Usability Testing:** Pengujian pengalaman pengguna secara langsung pada layar ponsel pintar Android fisik guna menilai kemudahan form input dan responsivitas layout[cite: 19, 27, 29, 32].
* **Software Test Report (STR):** Dokumen formal rekapitulasi hasil pengujian perangkat lunak yang merangkum status test case, temuan kegagalan, dan persentase kelulusan sistem[cite: 19, 29, 32].
* **Defect Tracking:** Tata kelola pencatatan dan pelacakan siklus hidup kendala/cacat (*bugs*) dari status penemuan (*Open*), perbaikan (*Fixed*), hingga verifikasi ulang (*Closed*)[cite: 19, 29, 32].
* **RESTful API:** Antarmuka layanan web berbasis standar protokol HTTP yang menghubungkan aplikasi klien mobile dengan server inferensi cloud menggunakan payload pertukaran format JSON[cite: 20, 26, 27].
* **Inference Latency:** Total waktu durasi komputasi yang dihabiskan server untuk menerima request HTTP, memproses inferensi model ML, dan mengirimkan respons JSON kembali ke aplikasi (standar batas: $< 1.500$ ms)[cite: 27, 33, 40].
* **PostgreSQL:** Sistem manajemen basis data relasional (*RDBMS*) tingkat enterprise yang digunakan untuk mengelola data operasional sistem, log inferensi, dan metadata registrasi model[cite: 1, 23].
* **Coolify:** Platform *Platform-as-a-Service* (PaaS) open-source mandiri (*self-hosted*) untuk orkestrasi container Docker dan manajemen sertifikat HTTPS Let's Encrypt secara otomatis[cite: 41].
* **Docker Compose:** Perkakas pendefinisian dan orkestrasi multi-container untuk menjalankan container layanan backend Flask dan basis data PostgreSQL secara terisolasi[cite: 41].