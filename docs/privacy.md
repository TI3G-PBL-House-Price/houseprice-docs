# Kebijakan Privasi & Tata Kelola Data (Data Privacy Policy) - HousePrice

Dokumen ini mendefinisikan prinsip perlindungan privasi pengguna, klasifikasi informasi, pemisahan penyimpanan data (*data segregation* antara penyimpanan aman perangkat lokal dan basis data cloud), isolasi multi-pengguna (*data isolation*), serta kebijakan retensi data pada ekosistem HousePrice pasca penerapan arsitektur *Strict Gate Authentication*[cite: 3, 4, 6].

---

## 1. Prinsip Tata Kelola & Privasi Data (Privacy by Design)

Sistem HousePrice menerapkan prinsip minimasi data (*data minimization*) dan kepatuhan perlindungan privasi data pribadi (*data protection by design*)[cite: 6]:

* **Pengumpulan Data Minimal Terkendali (*Controlled Minimal PII*):** Penerapan *Strict Gate Authentication* mewajibkan pengguna mendaftarkan akun untuk mengakses layanan[cite: 2, 4]. Data pribadi yang dihimpun dibatasi secara ketat hanya untuk kebutuhan operasional akun: Nama Lengkap, Alamat Email aktif (sebagai pengenal unik autentikasi), Nomor Telepon (opsional), dan Token Perangkat FCM (untuk penyaluran notifikasi tren pasar)[cite: 2, 4].
* **Nol Pengumpulan Data Finansial Sensitif (*Zero Sensitive Financial Data*):** Sistem tidak meminta, mencatat, maupun menyimpan informasi perbankan rahasia seperti Nomor Induk Kependudukan (NIK), nomor rekening bank, kartu kredit, slip gaji, atau riwayat kelayakan pinjaman riil[cite: 6]. Seluruh parameter pada kalkulator KPR murni berupa simulasi matematis anuitas yang dihitung secara mandiri[cite: 3, 6].
* **Kedaulatan & Kendali Pengguna (*User Data Sovereignty*):** Pengguna memiliki hak penuh untuk memperbarui profil, mengganti kata sandi keamanan, membersihkan histori prediksi, serta keluar dari sesi aplikasi (*safe logout*) dengan penghapusan token secara tuntas dari perangkat[cite: 2, 4].

---

## 2. Arsitektur Pemisahan Penyimpanan Data (Data Segregation)

Pemisahan tanggung jawab penyimpanan dirancang secara terisolasi guna menjamin integritas autentikasi, efisiensi komputasi ML, dan privasi catatan komputasi pengguna[cite: 3, 6]:

```
                  ARSITEKTUR PEMISAHAN DATA (DATA SEGREGATION)
                  
  [ PERANGKAT MOBILE PENGGUNA ]                     [ BASIS DATA CLOUD (POSTGRESQL) ]
  +-------------------------------------+           +-------------------------------------+
  | Flutter Secure Storage (Keystore)   |           | users                               |
  | - JWT Access Token (HS256)          |           | - Identitas akun (Nama, Email, Telp)|
  | - Refresh Token / Auth Session      |           | - Hash Sandi Bcrypt ($2b$12$)       |
  +-------------------------------------+           | - FCM Device Registration Token     |
  | SharedPreferences (Plain Sandbox)   |           +-------------------------------------+
  | - Flag Onboarding (is_first_launch) |                              | (1 to N)
  | - Cache preferensi tema tampilan    |                              v
  +-------------------------------------+           +-------------------------------------+
                     |                              | notifications                       |
                     | (Request dengan              | - Pesan tren harga pasar & rilis    |
                     |  Bearer JWT Token)           | - Status keterbacaan (is_read)      |
                     +----------------------------> +-------------------------------------+
                                                                       | (1 to N)
                                                                       v
                                                    +-------------------------------------+
                                                    | prediction_logs                     |
                                                    | - Relasi foreign key: user_id       |
                                                    | - Parameter fisik (LT, LB, kamar)   |
                                                    | - Nominal estimasi harga model ML   |
                                                    | - Telemetri (latensi & client IP)   |
                                                    +-------------------------------------+
```

### A. Penyimpanan Klien Mobile (Client-Side Storage)
1. **Ruang Penyimpanan Terenkripsi (*Encrypted Sandbox*):**
   * Menggunakan pustaka `flutter_secure_storage` yang terintegrasi dengan perangkat keras keamanan Android Keystore[cite: 3, 4].
   * Menyimpan sesi token otentikasi JWT[cite: 3, 4]. Dilarang keras menyimpan token pada *SharedPreferences* biasa guna mencegah ekstraksi sesi pada perangkat yang telah di-*root*[cite: 4, 5].
2. **Penyimpanan Preferensi Antarmuka (*Application Cache*):**
   * Menggunakan `SharedPreferences` terbatas untuk data status non-sensitif: penanda apakah pengguna telah menyelesaikan layar onboarding (*carousel*) dan pengaturan tampilan[cite: 2, 3].

### B. Penyimpanan Server Cloud (Server-Side Storage)
1. **Entitas Kredensial & Pengguna (`users`):**
   * Menyimpan identitas terdaftar[cite: 2]. Kata sandi dienkripsi satu arah menggunakan algoritma **Bcrypt** dengan faktor kerja (*work factor*) 12 *salt rounds*[cite: 4]. Teks kata sandi asli (*plaintext*) tidak pernah dicatat atau disimpan pada media penyimpanan mana pun[cite: 4].
2. **Entitas Riwayat Prediksi (`prediction_logs`):**
   * Menyimpan parameter fisik properti, nilai estimasi model ML, waktu inferensi, dan alamat IP klien untuk analisis telemetri[cite: 2, 6].
   * Dilengkapi kolom relasi `user_id` (Foreign Key merujuk ke `users.id`) untuk mendukung penarikan riwayat kalkulasi personal pada menu Profil pengguna[cite: 2].
3. **Entitas Notifikasi Sistem (`notifications`):**
   * Menyimpan pesan pemberitahuan perkembangan harga properti di wilayah DKI Jakarta dan pengumuman sistem yang ditujukan kepada akun pengguna[cite: 2, 3].

---

## 3. Isolasi Data Multi-Pengguna (Data Privacy Isolation)

Penerapan otentikasi *Strict Gate* menjamin prinsip isolasi data antar-pengguna (*multi-tenant segregation*) di tingkat backend API[cite: 2, 5]:

* **Otorisasi Berbasis Token Kriptografis:** Setiap permintaan data riwayat (`GET /api/v1/predict/history`) dan notifikasi (`GET /api/v1/notifications`) wajib menyertakan header `Authorization: Bearer <token>`[cite: 2, 3].
* **Pencegahan Akses Ilegal (Anti-IDOR):**
  * Kueri basis data pada backend Flask memfilter data secara wajib menggunakan klausa `WHERE user_id = :current_user_id`, di mana nilai `user_id` diekstrak langsung dari payload token JWT yang telah tervalidasi tanda tangannya (*signature*)[cite: 3, 5].
  * Pengguna tidak dapat melihat, mengubah, maupun menghapus riwayat komputasi atau notifikasi milik akun lain[cite: 4, 5].
* **Pemisahan Peran Administratif:** Akun administrator pada tabel `admins` terpisah secara fisik dari tabel `users`[cite: 2, 5]. Administrator hanya dapat memantau metrik performa model, volume dataset, dan jejak audit sistem, tanpa memiliki hak akses melihat kata sandi pengguna atau mengubah riwayat komputasi privat pengguna[cite: 2, 4].

---

## 4. Kebijakan Retensi & Hak Pemilik Data (Data Retention & Rights)

Sistem memberikan transparansi dan kontrol penuh kepada pemilik akun atas data mereka[cite: 4]:

| Hak Pengguna (*User Rights*) | Mekanisme Pelaksanaan dalam Sistem | Penanganan di Basis Data |
| :--- | :--- | :--- |
| **Hak Koreksi (*Right to Rectification*)** | Pengguna dapat memperbarui nama lengkap, nomor kontak, atau mengganti kata sandi melalui menu Profil Mobile (`PUT /api/v1/auth/profile`)[cite: 2]. | Memperbarui kolom `full_name`, `phone_number`, `password_hash`, serta memperbarui stempel waktu `updated_at`[cite: 2]. |
| **Hak Menghapus Riwayat (*Data Cleansing*)** | Pengguna dapat menghapus riwayat kalkulasi prediksi tertentu dari daftar tampilan menu profilnya[cite: 2]. | Menghapus baris terkait pada `prediction_logs` atau memutuskan keterikatan relasi `user_id`[cite: 2]. |
| **Hak Penghapusan Akun (*Right to Erasure*)** | Pengguna berhak meminta penonaktifan akun secara permanen (*delete account*) melalui permohonan ke pengelola sistem. | Penghapusan akun mengeksekusi aksi berjenjang: seluruh notifikasi terhapus permanen (`ON DELETE CASCADE`), sedangkan log inferensi properti dilepas kaitan identitasnya (`ON DELETE SET NULL`) agar tidak merusak validitas statistik agregasi pasar[cite: 2]. |
| **Pembersihan Sesi Lokal (*Safe Logout*)** | Pengguna memilih opsi *Logout* pada menu Pengaturan Profil[cite: 2]. | Seluruh token JWT pada `flutter_secure_storage` dihapus tuntas dan state aplikasi dialihkan kembali ke `LandingScreen`[cite: 3, 4]. |

---

## 5. Integrasi Layanan Pihak Ketiga & Keamanan Transmisi

1. **Enkripsi Saluran Komunikasi (Wajib TLS 1.3):**
   * Seluruh pertukaran data antara aplikasi mobile dan server cloud dienkripsi menggunakan protokol HTTPS berbasis sertifikat resmi Let's Encrypt[cite: 3, 4].
   * Komunikasi teks polos (*cleartext HTTP*) dinonaktifkan sepenuhnya pada konfigurasi jaringan aplikasi klien[cite: 4].
2. **Firebase Cloud Messaging (FCM):**
   * Token perangkat FCM yang tersimpan pada atribut `users.fcm_token` hanya digunakan untuk mendistribusikan push notification terkait tren properti[cite: 2, 4].
   * Tidak ada data identitas sensitif (seperti kata sandi atau nominal estimasi harga) yang disertakan di dalam muatan data *push notification payload*.
3. **Tanpa Broker Data Komersial:**
   * Sistem HousePrice tidak menyematkan pustaka pelacak pihak ketiga (*third-party commercial trackers*), SDK iklan, maupun menjual agregasi data perilaku pengguna kepada entitas luar mana pun.