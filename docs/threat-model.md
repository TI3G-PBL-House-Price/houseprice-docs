# Pemodelan Ancaman Keamanan (Threat Model) - HousePrice

Dokumen ini mendefinisikan analisis ancaman keamanan perangkat lunak menggunakan kerangka kerja **STRIDE** (*Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege*) untuk mengidentifikasi potensi vektor serangan serta menetapkan kontrol mitigasi pada sistem HousePrice (aplikasi mobile Flutter, RESTful API Flask, Web Admin, dan PostgreSQL)[cite: 1, 23, 40].

---

## 1. Diagram Aliran Data & Batas Kepercayaan (Data Flow & Trust Boundaries)

Sistem HousePrice memiliki tiga zona batas kepercayaan (*Trust Boundaries*) yang memisahkan komponen publik dan komponen privat internal[cite: 23, 26, 40]:

```
+-----------------------------------------------------------------------------------+
| ZONA 1: UNTRUSTED ENVIRONMENT (Internet Publik)                                   |
|   [ Flutter Mobile Client ]              [ Web Browser Administrator ]            |
+-----------------------------------------------------------------------------------+
                               | (HTTPS / TLS 1.3)
                        [ TRUST BOUNDARY A ]
                               v
+-----------------------------------------------------------------------------------+
| ZONA 2: SEMI-TRUSTED DMZ (Reverse Proxy & Ingress)                                |
|   [ Reverse Proxy (Coolify / Traefik / Caddy) ]                                    |
|   - Terminasi SSL/TLS Let's Encrypt                                               |
|   - Pengendali Header Keamanan & CORS Policy                                      |
+-----------------------------------------------------------------------------------+
                               | (Internal Bridge Network)
                        [ TRUST BOUNDARY B ]
                               v
+-----------------------------------------------------------------------------------+
| ZONA 3: TRUSTED INTERNAL ZONE (Isolated Docker Network)                           |
|   +-----------------------------------------------------------------------------+ |
|   | Flask Application Service                                                   | |
|   | - Blueprints: /api/v1 (Public API) & /admin (Protected Admin)               | |
|   | - ML Runtime: Scikit-learn Pipeline Loader (.pkl)                           | |
|   +-----------------------------------------------------------------------------+ |
|                               | (Internal Socket / Port 5432)                     |
|                               v                                                   |
|   +-----------------------------------------------------------------------------+ |
|   | PostgreSQL Database Service (houseprice_db)                                 | |
|   | - admins, prediction_logs, system_audit_logs, ml_models                     | |
|   +-----------------------------------------------------------------------------+ |
+-----------------------------------------------------------------------------------+
```

---

## 2. Matriks Analisis Ancaman STRIDE & Mitigasi

Berikut adalah identifikasi ancaman spesifik pada setiap kategori kerangka kerja STRIDE beserta mekanisme pertahanan (*security controls*) yang diterapkan:

| Kategori Ancaman | Komponen Terdampak | Skenario Ancaman (*Threat Scenario*) | Dampak (*Impact*) | Strategi Mitigasi (*Mitigation Control*) |
| :--- | :--- | :--- | :--- | :--- |
| **Spoofing** (Pemalsuan Identitas) | Web Admin Authentication | Penyerang menebak kata sandi admin atau memalsukan sesi login untuk mengakses dashboard[cite: 1, 23]. | High | *Password hashing* menggunakan **Bcrypt** (*cost factor* 12), manajemen sesi berbasis *secure HTTP-only cookie*, dan pembatasan percobaan login (*rate limiting*)[cite: 1]. |
| **Spoofing** | REST API Inferensi | Penyerang berpura-pura menjadi aplikasi mobile resmi untuk mengirimkan spam inferensi[cite: 23, 27]. | Medium | Validasi header permintaan (*User-Agent enforcement*), validasi skema JSON masukan, serta pembatasan CORS pada rute admin[cite: 27, 40]. |
| **Tampering** (Manipulasi Data) | REST API Inferensi | Penyerang mengirimkan masukan SQL Injection melalui kolom teks `city` atau `district`[cite: 1, 24]. | High | Penerapan *Parameterized Queries* melalui ORM SQLAlchemy; masukan lokasi wajib dicocokkan dengan entri valid tabel `master_locations`[cite: 1, 24]. |
| **Tampering** | Pipeline Retraining ML | Pengunggahan berkas CSV berbahaya (*malicious CSV*) yang dirancang untuk memanipulasi bobot model ML[cite: 1, 23]. | Critical | Validasi integritas berkas CSV pada tabel `dataset_metadata`, pengecekan skema kolom, eliminasi baris duplikat, dan *outlier filtering* otomatis (IQR) sebelum training dijalankan[cite: 1, 27, 33]. |
| **Tampering** | Artefak Model `.pkl` | Modifikasi berkas serialisasi model di storage server[cite: 1, 27]. | Critical | Berkas `.pkl` disimpan di direktori terproteksi dengan hak akses baca (*read-only* bagi proses Flask); verifikasi *hash SHA-256* sebelum model dimuat ke memori[cite: 27]. |
| **Repudiation** (Penyangkalan Aksi) | Web Admin Operations | Administrator menghapus model ML aktif atau mengganti dataset lalu menyangkal tindakannya[cite: 1, 23]. | Medium | Pencatatan otomatis seluruh operasi administratif ke tabel `system_audit_logs` yang merekam `admin_id`, `action_type`, `ip_address`, dan waktu stempel kejadian[cite: 1]. |
| **Information Disclosure** (Kebocoran Data) | Jaringan Komunikasi | Penyadapan paket data (*eavesdropping*) pada jalur komunikasi antara mobile client dan server cloud[cite: 23, 26]. | High | Penegakan enkripsi saluran penuh menggunakan **HTTPS (TLS 1.3)** Let's Encrypt; penolakan lalu lintas teks polos (*cleartext disabled* pada Android)[cite: 40, 41]. |
| **Information Disclosure** | Repositori Kode | Kredensial basis data dan kunci rahasia (*SECRET_KEY*) terekspos di GitHub publik[cite: 41, 42]. | Critical | Penegakan berkas `.gitignore`, isolasi variabel lingkungan pada `.env`, dan penyediaan `.env.example` tanpa nilai rahasia aktual[cite: 41, 42]. |
| **Denial of Service** (Gang