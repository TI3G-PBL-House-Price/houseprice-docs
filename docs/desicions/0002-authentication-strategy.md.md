# ADR 0002: Strategi Otentikasi & Otorisasi Sistem (Web Admin vs Mobile Client)

- **Status:** Diterima (Accepted)
- **Tanggal:** 2026-10-06
- **Pengambil Keputusan:** Abdul Rahman Hanif Darmawan (Project Manager), Tim Pengembang HousePrice
- **Konsultasi:** Habibie Ed Dien, S.Kom., M.T., Usman Nurhasan, S.Kom., M.T., M. Hasyim Ratsanjani, S.Kom., M.Kom.

---

## 1. Konteks & Latar Belakang Masalah

Sistem HousePrice melayani dua tipe pemangku kepentingan (*stakeholders*) dengan karakteristik interaksi dan kebutuhan keamanan yang sangat bertolak belakang[cite: 20, 24]:

1. **Pengguna Umum Aplikasi Mobile (Flutter):** Masyarakat umum yang memerlukan estimasi harga rumah, evaluasi kewajaran penawaran, dan simulasi KPR secara cepat, intuitif, dan tanpa hambatan registrasi (*frictionless experience*)[cite: 20, 24].
2. **Administrator Teknis Web Dashboard (Flask):** Tim internal pengelola sistem yang memiliki hak istimewa untuk mengunggah dataset properti baru, memantau telemetri inferensi, mengeksekusi pelatihan ulang model machine learning, dan mengaktifkan versi model produksi (`.pkl`)[cite: 1, 23].

Tantangan arsitektur yang harus diselesaikan:
* Menghilangkan friksi pendaftaran bagi pengguna mobile tanpa membuka celah serangan eksploitasi inferensi (*DDoS / Scraping*)[cite: 27, 40].
* Mencegah pengumpulan data pribadi (*Personally Identifiable Information* / PII) pada pengguna akhir demi kepatuhan privasi (*Privacy by Design*)[cite: 20, 24].
* Menerapkan mekanisme otentikasi yang kuat dan tahan uji pada Web Admin untuk mencegah pengambilalihan kontrol model machine learning (*Elevation of Privilege*)[cite: 1, 23].

---

## 2. Alternatif Strategi yang Dipertimbangkan

### Opsi A: Otentikasi Terpadu Berbasis Akun Pengguna Wajib (Full Mandatory Login)
* *Deskripsi:* Seluruh pengguna (baik mobile maupun web) wajib mendaftar akun menggunakan email dan kata sandi sebelum dapat mengakses fitur apa pun.
* *Evaluasi:* Ditolak. Mewajibkan registrasi pada aplikasi mobile menciptakan friksi adopsi yang tinggi bagi calon pembeli/penjual rumah yang hanya ingin mengecek estimasi harga secara cepat[cite: 20, 24]. Selain itu, opsi ini membebani basis data PostgreSQL dengan pengelolaan tabel profil pengguna umum yang sebenarnya tidak diperlukan oleh logika bisnis utama[cite: 1, 24].

### Opsi B: Hardcoded API Key pada Aplikasi Mobile
* *Deskripsi:* Menyematkan kunci API statis (*static API key*) ke dalam kode sumber Flutter untuk memvalidasi permintaan inferensi.
* *Evaluasi:* Ditolak. Berkas APK Android mudah didekompilasi melalui proses *reverse engineering*, sehingga *hardcoded key* pasti akan terekspos dan tidak memberikan jaminan keamanan nyata[cite: 27, 40].

### Opsi C: Strategi Pemisahan Akses Ganda / Dual-Tier Access (Terpilih)
* *Deskripsi:* 
  * **Tingkat Klien Mobile (Public Stateless Access):** Endpoint publik (`/api/v1/predict`, `/api/v1/market-statistics`, `/api/v1/locations`) dapat diakses tanpa login (*guest mode*), diproteksi dengan pembatasan laju (*Rate Limiting*) dan validasi skema input ketat[cite: 27, 40].
  * **Tingkat Web Admin (Stateful Session & JWT Authentication):** Portal administratif dan rute mutasi model (`/admin/*`, `/api/v1/admin/*`) diproteksi penuh menggunakan kata sandi ter-hash Bcrypt serta otentikasi berbasis *Secure HTTP-Only Cookie* / JWT Bearer Token[cite: 1, 23].

---

## 3. Matriks Evaluasi Alternatif

| Kriteria Penilaian | Opsi A (Mandatory Login) | Opsi B (Static API Key) | Opsi C (Dual-Tier Access) |
| :--- | :--- | :--- | :--- |
| **Kenyamanan Pengguna Mobile (UX)** | Rendah (Ada barrier registrasi) | Tinggi (Langsung pakai) | **Sangat Tinggi (Frictionless)**[cite: 20, 24] |
| **Kepatuhan Privasi (Privacy by Design)** | Rendah (Mengharuskan simpan PII) | Tinggi (Tanpa PII) | **Sangat Tinggi (Zero PII Mobile)**[cite: 24, 25] |
| **Ketahanan Keamanan Mobile** | Tinggi | Sangat Rendah (Key bocor di APK) | **Tinggi (Rate Limited & Validated)**[cite: 27, 40] |
| **Keamanan Rute Administratif** | Tinggi | Tergantung implementasi | **Sangat Tinggi (Bcrypt + Protected Role)**[cite: 1, 23] |
| **Kompleksitas Pengembangan** | Tinggi (Butuh modul auth mobile) | Rendah | **Moderat & Terfokus pada Inti Sistem** |

---

## 4. Keputusan & Alasan Pemilihan (Rationale)

Tim memutuskan untuk mengadopsi **Opsi C: Strategi Pemisahan Akses Ganda (Dual-Tier Access Strategy)** dengan pertimbangan teknis berikut:

1. **Minimasi Data Pribadi Pengguna Mobile:**
   Aplikasi mobile HousePrice difokuskan murni sebagai alat bantu pengambilan keputusan (*decision support tool*)[cite: 20, 21]. Seluruh data riwayat pencarian disimpan di dalam SQLite lokal perangkat pengguna, sehingga backend tidak memerlukan identitas personal pengguna[cite: 25, 31].
2. **Pengamanan Akses Publik Melalui Kontrol Perilaku (Behavioral Controls):**
   Sebagai pengganti login pada aplikasi mobile, integritas layanan backend dijaga melalui:
   * *Rate Limiting:* Membatasi setiap alamat IP pemohon maksimal 60 permintaan inferensi per menit guna menangkal serangan DoS dan pemindaian otomatis (*scraping*)[cite: 1, 27].
   * *Strict Input Validation:* Validasi skema JSON masukan (menggunakan Marshmallow/Pydantic) menolak data anomali sebelum dieksekusi oleh model machine learning[cite: 27, 40].
3. **Standar Enkripsi Kata Sandi Administrator (Bcrypt):**
   Pada tabel `admins`, kata sandi dienkripsi menggunakan algoritma **Bcrypt** dengan *salt work factor* 12 putaran (`$2b$12$...`)[cite: 1]. Bcrypt memiliki mekanisme *adaptive cost* yang secara efektif melindungi akun dari serangan *brute force* dan *rainbow table dictionary attack*[cite: 1].
4. **Isolasi Rute Melalui Flask Blueprints:**
   Pemisahan rute secara tegas:
   * `blueprints/api/`: Rute publik terbuka untuk inferensi klien mobile[cite: 23, 40].
   * `blueprints/admin/`: Rute tertutup yang menerapkan dekorator otentikasi wajib (`@login_required` / `@admin_required`), memverifikasi sesi login admin sebelum mengizinkan pengunggahan dataset atau retraining model[cite: 1, 23].

---

## 5. Konsekuensi Positif & Mitigasi Risiko

### Konsekuensi Positif (Benefits)
* Pengguna mobile dapat langsung menggunakan aplikasi secara instan tanpa hambatan aktivasi akun atau verifikasi surel[cite: 20, 24].
* Arsitektur backend lebih ramping dan hemat memori karena tidak perlu mengelola ratusan sesi pengguna umum[cite: 1, 23].
* Keamanan modul administratif terpusat pada tabel `admins` dan terekam penuh di tabel `system_audit_logs`[cite: 1].

### Risiko & Mitigasi (Risks & Mitigations)
* **Penyalahgunaan API Inferensi oleh Pihak Luar:**
  * *Mitigasi:* Mengonfigurasi modul `Flask-Limiter` dengan penyimpanan berbasis memori/Redis untuk menegakkan kuota permintaan per menit per alamat IP (`client_ip`)[cite: 1, 27].
* **Pembajakan Sesi Admin (Session Hijacking):**
  * *Mitigasi:* Cookie sesi admin wajib dikonfigurasi dengan flag `HttpOnly`, `Secure` (hanya via HTTPS), dan `SameSite=Strict` untuk menangkal serangan Cross-Site Scripting (XSS) dan Cross-Site Request Forgery (CSRF)[cite: 40, 41].