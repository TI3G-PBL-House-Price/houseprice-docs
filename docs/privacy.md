# Kebijakan Privasi & Pemisahan Data (Data Privacy Policy) - HousePrice

Dokumen ini mendefinisikan prinsip perlindungan privasi pengguna, batasan pengumpulan informasi, pemisahan penyimpanan data (*data segregation* antara penyimpanan lokal perangkat dan basis data cloud), serta retensi data pada sistem HousePrice.

---

## 1. Prinsip Privasi Berkelanjutan (Privacy by Design)

Sistem HousePrice menerapkan prinsip minimasi data (*data minimization*) dan privasi sejak perancangan (*privacy by design*) sesuai kaidah pelindungan data pribadi:

* **Tanpa Registrasi Pengguna Akhir:** Pengguna umum pada aplikasi mobile dapat mengakses fitur prediksi harga rumah, pengecekan kewajaran penawaran, kalkulator KPR, dan statistik pasar tanpa membuat akun, tanpa login, dan tanpa menyerahkan identitas diri[cite: 20, 24, 40].
* **Nol Pengumpulan Informasi Pribadi (Zero PII):** Sistem tidak meminta, mencatat, maupun menyimpan data identitas individu (*Personally Identifiable Information* / PII) seperti Nama Lengkap, Nomor Induk Kependudukan (NIK), alamat email pengguna akhir, nomor telepon, atau data rekening keuangan/perbankan[cite: 24, 25].
* **Kedaulatan Data di Tangan Pengguna:** Riwayat penelusuran properti dan simulasi kredit disimpan secara lokal di dalam ruang penyimpanan aman (*application sandbox*) milik perangkat pengguna sendiri[cite: 25, 31].

---

## 2. Pemisahan Penyimpanan Data: Lokal vs Cloud

Pemisahan tanggung jawab penyimpanan dilakukan secara ketat untuk melindungi kerahasiaan kalkulasi finansial pengguna sekaligus menjaga keandalan analisis backend:

```
                  ARSITEKTUR PEMISAHAN DATA (DATA SEGREGATION)
                  
  [ PERANGKAT MOBILE PENGGUNA ]                     [ SERVER CLOUD (POSTGRESQL) ]
  +-------------------------------------+           +-------------------------------------+
  | Local Storage (SQLite / Prefs)      |           | prediction_logs                     |
  | - Catatan histori simulasi pribadi  |           | - Parameter fisik properti          |
  | - Nilai penawaran riil penjual      |           | - Nilai estimasi nominal model      |
  | - Rincian kalkulasi angsuran KPR    |  Request  | - Versi model ML (misal: v1.0.0)    |
  | - Persentase Down Payment (DP)      | --------> | - Latensi inferensi (ms)            |
  | - Riwayat preferensi tenor & bunga  |           | - Alamat IP klien (Audit/Security)  |
  +-------------------------------------+           +-------------------------------------+
       |                                                         |
  (Tersimpan offline di memori ponsel)             (Agregasi statistik pasar & audit)
  (Hanya dapat diakses oleh pemilik HP)            (Tanpa identitas nama/profil pengguna)
```

### A. Penyimpanan Lokal Klien (Client-Side Storage)
* **Teknologi Penyimpanan:** SQLite Database dan SharedPreferences / EncryptedSharedPreferences pada Android[cite: 25, 31].
* **Ruang Lingkup Data:**
  * Riwayat estimasi properti yang disimpan secara sadar oleh pengguna (*Save Calculation*)[cite: 25].
  * Variabel finansial simulasi KPR (persentase DP, suku bunga, tenor kredit, dan nominal cicilan bulanan)[cite: 25].
  * Riwayat penawaran harga penjual pada fitur Cek Kewajaran[cite: 24, 25].
* **Batasan Akses:** Seluruh data lokal tersimpan di dalam direktori internal aplikasi (`/data/data/com.polinema.houseprice/`) yang terisolasi dari aplikasi lain di perangkat Android pengguna.

### B. Penyimpanan Cloud Server (Server-Side Storage)
* **Teknologi Penyimpanan:** PostgreSQL Database pada tabel `prediction_logs`[cite: 1, 23].
* **Ruang Lingkup Data yang Dikirim:**