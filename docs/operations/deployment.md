# Panduan Penerapan & Infrastruktur (Deployment Guide) - HousePrice

Dokumen ini memuat panduan lengkap penerapan (*deployment*) sistem HousePrice menggunakan platform open-source **Coolify (Self-Hosted PaaS)** sebagai lingkungan produksi utama berbasis cloud, serta template kontainerisasi **Docker Compose** sebagai alternatif mandiri (*fallback*) untuk lingkungan lokal atau demonstrasi server kampus[cite: 41].

---

## 1. Ikhtisar Arsitektur Penerapan (Deployment Overview)

Sistem HousePrice dirancang untuk dapat dijalankan secara konsisten pada berbagai lingkungan melalui teknologi kontainer Docker[cite: 41]:

```
                     TOPOLOGI SERVER CLOUD (COOLIFY PAAS)
                     
   Internet (HTTPS Port 443 / HTTP Port 80)
                     |
                     v
   +-----------------------------------------------------------------+
   | Reverse Proxy (Traefik v3 bawaan Coolify)                       |
   | - Otomatisasi Sertifikat SSL/TLS Let's Encrypt                  |
   | - Terminasi SSL & Pengalihan Otomatis HTTP -> HTTPS             |
   +-----------------------------------------------------------------+
                     |
         +-----------+-----------+ (Jaringan Jembatan Docker Privat)
         |                       |
         v                       v
   +-----------------------+   +-----------------------+
   | houseprice-backend    |   | houseprice-postgres   |
   | - Python 3.10+ Flask  |   | - PostgreSQL 15+      |
   | - Gunicorn (4 Worker) |<->| - Persistent Storage  |
   | - Scikit-Learn .pkl   |   | - Port 5432 (Privat)  |
   | - Port 5000 (Internal)|   |                       |
   +-----------------------+   +-----------------------+
```

### Spesifikasi Minimum Server (VPS / Cloud Compute):
* **Sistem Operasi:** Ubuntu 22.04 LTS / 24.04 LTS (x86_64)
* **CPU:** Minimal 1 vCPU (Disarankan 2 vCPU untuk retraining model)
* **RAM:** Minimal 2 GB (Rekomendasi 4 GB untuk menampung Coolify + Flask ML + PostgreSQL)
* **Penyimpanan:** Minimal 25 GB SSD
* **Jaringan:** Alamat IP Publik Statis dengan Port 80 dan 443 terbuka

---

## 2. Pilihan 1: Penerapan Cloud Menggunakan Coolify (Direkomendasikan)

Coolify adalah PaaS (*Platform-as-a-Service*) open-source mandiri yang menyediakan antarmuka visual (GUI) modern untuk orkestrasi kontainer, konfigurasi domain, penyediaan SSL gratis, dan integrasi repositori Git[cite: 41].

### Langkah 1: Instalasi Coolify pada VPS Ubuntu
Akses VPS Anda melalui terminal SSH, lalu jalankan skrip instalasi resmi Coolify satu baris:
```bash
curl -fsSL [https://cdn.coollabs.io/coolify/install.sh](https://cdn.coollabs.io/coolify/install.sh) | bash
```
Setelah instalasi selesai, buka peramban (*browser*) dan akses antarmuka Coolify melalui URL: `http://<IP-VPS-ANDA>:8000` untuk melakukan registrasi akun administrator root pertama kali[cite: 41].

### Langkah 2: Penyediaan Basis Data PostgreSQL
1. Pada dashboard Coolify, pilih menu **Projects** $\rightarrow$ **Default** $\rightarrow$ **Production Environment**[cite: 41].
2. Klik **+ Add Resource** $\rightarrow$ Pilih **Databases** $\rightarrow$ **PostgreSQL**[cite: 41].
3. Masukkan konfigurasi basis data:
   * **Database Name:** `houseprice_db`
   * **Default Username:** `houseprice_admin`
   * **Default Password:** Buat kata sandi aman (misal: 32 karakter acak)
4. Klik **Start**. PostgreSQL akan berjalan dalam kontainer terisolasi dan menyediakan *internal connection string*[cite: 41].

### Langkah 3: Integrasi Repositori Backend & Otomasi Deploy (CI/CD)
1. Pada proyek yang sama di Coolify, klik **+ Add Resource** $\rightarrow$ **Public/Private Repository**[cite: 41].
2. Tautkan URL repositori backend organisasi:
   `https://github.com/PBL-House-Price/houseprice-backend-ml.git`[cite: 34, 41]
3. Pilih cabang rilis utama: `main`[cite: 31, 33, 41].
4. Atur tipe build: **Dockerfile** (Coolify akan mendeteksi `Dockerfile` di root repo)[cite: 41].
5. Konfigurasikan domain layanan: Masukkan subdomain Anda (misal: `https://api.houseprice.id`)[cite: 41].
   *(Coolify dan Traefik akan otomatis mendaftarkan sertifikat SSL Let's Encrypt gratis)*[cite: 41].

### Langkah 4: Pengaturan Variabel Lingkungan (Environment Variables)
Pada tab **Environment Variables** di layanan backend Coolify, isikan variabel berikut[cite: 41]:
```ini
FLASK_ENV=production
SECRET_KEY=kunci-rahasia-produksi-super-acak-minimal-32-karakter
DB_HOST=houseprice-postgres
DB_PORT=5432
DB_NAME=houseprice_db
DB_USER=houseprice_admin
DB_PASSWORD=kata_sandi_yang_dibuat_pada_langkah_2
MODEL_PATH=models/rf_house_price_v1.pkl
ALLOWED_ORIGINS=[https://admin.houseprice.id](https://admin.houseprice.id),http://localhost:3000
```
Klik **Deploy**. Backend Flask Anda sekarang aktif dan dapat diakses publik melalui HTTPS[cite: 41]!

---

## 3. Pilihan 2: Penerapan Berbasis Docker Compose (Local / Fallback Mode)

Template ini digunakan untuk menjalankan seluruh ekosistem backend dan basis data di komputer lokal pengembang atau saat demonstrasi luring tanpa koneksi internet luar[cite: 41].

### A. Berkas `Dockerfile` (Backend Flask)
Simpan berkas ini di root repositori `houseprice-backend-ml`:
```dockerfile
# Menggunakan base image resmi Python yang stabil dan ramping
FROM python:3.10-slim

# Mencegah Python membuat berkas .pyc dan memastikan output log terkirim langsung
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

# Instalasi dependensi sistem untuk kompilasi psycopg2
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libpq-dev \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Salin dan pasang paket pustaka Python
COPY requirements.txt /app/
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Salin seluruh kode sumber aplikasi
COPY . /app/

# Port operasional WSGI
EXPOSE 5000

# Eksekusi server produksi menggunakan Gunicorn (4 Worker threads)
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "--timeout", "120", "wsgi:app"]
```

### B. Berkas `docker-compose.yml`
Simpan berkas ini di root repositori `houseprice-backend-ml`:
```yaml
version: '3.8'

services:
  # 1. Basis Data PostgreSQL
  postgres:
    image: postgres:15-alpine
    container_name: houseprice-postgres
    restart: always
    environment:
      POSTGRES_DB: ${DB_NAME:-houseprice_db}
      POSTGRES_USER: ${DB_USER:-houseprice_admin}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-houseprice_secret}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./migrations/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    networks:
      - houseprice-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-houseprice_admin} -d ${DB_NAME:-houseprice_db}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # 2. Layanan Backend Flask & Model ML
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: houseprice-backend
    restart: always
    depends_on:
      postgres:
        condition: service_healthy
    env_file:
      - .env
    volumes:
      - ./models:/app/models
      - ./uploads:/app/uploads
    ports:
      - "5000:5000"
    networks:
      - houseprice-net

networks:
  houseprice-net:
    driver: bridge

volumes:
  postgres_data:
    driver: local
```

### C. Cara Menjalankan Secara Mandiri
1. Gandakan repositori dan buat berkas `.env` dari template[cite: 41]:
   ```bash
   cp .env.example .env
   ```
2. Jalankan seluruh layanan dengan satu perintah:
   ```bash
   docker compose up -d --build
   ```
3. Periksa status layanan yang berjalan:
   ```bash
   docker compose ps
   ```
4. Menghentikan layanan secara aman:
   ```bash
   docker compose down
   ```

---

## 4. Konfigurasi Klien Mobile (Flutter Connectivity)

Setelah backend berhasil aktif pada server cloud, sambungkan aplikasi Flutter dengan menyetel URL dasar API pada berkas konfigurasi lingkungan aplikasi mobile[cite: 26, 40]:

1. Buka berkas `lib/core/constants/api_constants.dart` pada repositori `houseprice-mobile-app`[cite: 31]:
   ```dart
   class ApiConstants {
     // Gunakan domain HTTPS produksi dari Coolify
     static const String baseUrl = '[https://api.houseprice.id/api/v1](https://api.houseprice.id/api/v1)';
     
     // Endpoint rute
     static const String predictEndpoint = '$baseUrl/predict';
     static const String marketStatsEndpoint = '$baseUrl/market-statistics';
     static const String locationsEndpoint = '$baseUrl/locations';
   }
   ```
2. **Catatan Pengujian Emulator Android:**
   Jika menguji menggunakan koneksi server lokal di PC pengembang (`localhost:5000`), ubah URL dasar emulator Android menjadi `http://10.0.2.2:5000/api/v1` (karena Android Emulator menggunakan alias IP `10.0.2.2` untuk merujuk ke host mesin pengembang).

---

## 5. Prosedur Verifikasi Pasca-Deployment (Smoke Testing)

Setelah proses deployment selesai, jalankan tiga pengujian cepat untuk memastikan integritas operasional[cite: 27]:

1. **Uji Jalur HTTPS:**
   ```bash
   curl -I [https://api.houseprice.id/api/v1/locations](https://api.houseprice.id/api/v1/locations)
   ```
   *Hasil:* Wajib mengembalikan kode status `HTTP/2 200` dengan sertifikat SSL valid.
2. **Uji Latensi Komputasi Inferensi:**
   Kirimkan permintaan inferensi uji coba dan pastikan respons mengembalikan nilai harga nominal serta parameter `latency_ms` bernilai $< 1.500$ ms[cite: 1, 27].
3. **Uji Integritas Basis Data:**
   Pastikan tabel `prediction_logs` pada basis data PostgreSQL berhasil mencatat 1 baris riwayat inferensi baru dari pengujian di atas[cite: 1, 23].