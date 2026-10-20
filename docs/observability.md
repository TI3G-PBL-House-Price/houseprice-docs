# Observabilitas & Pemantauan Sistem (Observability & Monitoring) - HousePrice

Dokumen ini mendefinisikan strategi observabilitas, arsitektur pencatatan log (*logging*), telemetri inferensi model machine learning, metrik kesehatan aplikasi, serta audit jejak administratif (*audit trail*) pada sistem HousePrice.

---

## 1. Pilar Observabilitas Sistem

Observabilitas sistem HousePrice dirancang untuk memberikan transparansi penuh terhadap kinerja komputasi *cloud*, integritas model ML, dan operasional backend melalui tiga pilar utama:

```
                       ARSITEKTUR OBSERVABILITAS
                       
  [ Klien Mobile ] ----> POST /api/v1/predict ----> [ Inference Engine ]
         |                                                   |
         | (Response Time Telemetry)                         | (Prediction Telemetry)
         v                                                   v
  +------------------+                              +------------------+
  | Application Logs |                              | prediction_logs  |
  |  (Stdout/JSON)   |                              |   (PostgreSQL)   |
  +------------------+                              +------------------+
         |                                                   |
         +-------------------+-------------------------------+
                             |
                             v
                 +-----------------------+
                 | Web Dashboard Admin   |
                 | - Latensi Inferensi   |
                 | - Metrik Model (R²)   |
                 | - Volume Permintaan   |
                 | - Log Audit Sistem    |
                 +-----------------------+
```

1. **Log Inferensi & Telemetri Real-time:** Setiap prediksi harga yang diminta oleh aplikasi mobile dicatat secara otomatis ke basis data relasional untuk memantau performa inferensi dan sebaran data masukan pengguna[cite: 1, 23].
2. **Pemantauan Performa Model (*Model Performance Tracking*):** Pelacakan metrik akurasi ($R^2$, MAE, RMSE) dari model yang sedang aktif maupun riwayat pelatihan model baru[cite: 1, 27].
3. **Audit Jejak Keamanan (*Security Audit Trail*):** Pencatatan menyeluruh terhadap setiap aksi administratif yang terjadi pada Web Admin guna menjaga akuntabilitas sistem[cite: 1, 23].

---

## 2. Telemetri Inferensi (Inference Logging)

Setiap panggilan ke endpoint `POST /api/v1/predict` direkam secara terstruktur ke tabel `prediction_logs` pada basis data PostgreSQL[cite: 1, 27].

### A. Struktur Data Log Inferensi
Atribut yang terekam pada tabel `prediction_logs` meliputi[cite: 1]:
* `id` (UUID): Identitas unik untuk setiap transaksi inferensi[cite: 1].
* `city` & `district` (VARCHAR): Parameter wilayah administrasi properti yang diminta pengguna[cite: 1].
* `surface_area` & `building_area` (NUMERIC): Parameter ukuran fisik tanah dan bangunan ($m^2$)[cite: 1].
* `bed`, `bath`, `carport` (INTEGER): Parameter fasilitas ruangan properti[cite: 1].
* `predicted_price` (NUMERIC): Nilai estimasi nominal harga hasil inferensi model Machine Learning[cite: 1].
* `model_version` (VARCHAR): Versi model regresi yang melayani inferensi (misal: `v1.0.0`)[cite: 1].
* `latency_ms` (INTEGER): Durasi pemrosesan inferensi server dalam satuan milidetik[cite: 1].
* `client_ip` (VARCHAR): Alamat IP perangkat pemohon untuk mendeteksi anomali trafik atau serangan DoS[cite: 1].
* `created_at` (TIMESTAMP): Waktu stempel pencatatan transaksi inferensi[cite: 1].

### B. Indikator Kinerja Inferensi (SLI / SLO)
* **Target Latensi (Service Level Objective):**
  * $P_{50}$ (Median): $< 200$ ms
  * $P_{95}$: $< 800$ ms
  * $P_{99}$ (Batas Ambang Kritis): $< 1.500$ ms (1,5 detik)[cite: 27, 40]
* **Tindakan Ambang Batas:** Jika latensi melebihi 1.500 ms secara konsisten pada 5 permintaan berturut-turut, sistem backend mencatat peringatan tingkat `WARNING` pada log aplikasi untuk diinvestigasi oleh penanggung jawab API[cite: 27, 28].

---

## 3. Pemantauan Metrik & Siklus Hidup Model ML

Pemantauan artefak model machine learning dilakukan melalui tabel `ml_models` dan disajikan secara visual pada antarmuka Web Dashboard Admin[cite: 1, 23]:

```
+--------------------------------------------------------------------------+
|                        DASHBOARD METRIK MODEL ML                         |
+--------------------+--------------------+--------------------------------+
| Versi Aktif:       | Koefisien R²:      | MAE: Rp 1.063.034.875          |
| v1.0.0 (RF Reg)    | 0.8272 (Target: 0.8) | RMSE: Rp 1.808.105.786        |
+--------------------+--------------------+--------------------------------+
| Status: AKTIF      | Algoritma: Ensemble Random Forest Regressor         |
+--------------------------------------------------------------------------+
```

### A. Metrik Kualitas yang Dipantau
1. **Koefisien Determinasi ($R^2$ Score):** Menilai ketepatan variansi harga properti. Nilai wajib berada di atas ambang batas $0,80$[cite: 1, 27].
2. **Mean Absolute Error (MAE):** Menilai rata-rata simpangan kesalahan nominal estimasi terhadap harga aktual[cite: 1, 27].
3. **Root Mean Squared Error (RMSE):** Menilai sensitivitas terhadap kesalahan prediksi berskala besar[cite: 1, 27].

### B. Prosedur Evaluasi Model Baru (Model Retraining Telemetry)
Saat admin mengeksekusi *retrain* melalui Web Admin[cite: 23]:
1. Sistem mencatat metadata eksekusi ke tabel `ml_models` dengan status `is_active = FALSE`[cite: 1].
2. Dashboard menampilkan perbandingan metrik *side-by-side* antara model baru vs model aktif saat ini[cite: 23].
3. Pergantian model aktif (`is_active = TRUE`) hanya dapat disetujui jika model baru menunjukkan nilai $R^2 \ge 0,80$ dan penurunan nilai MAE/RMSE[cite: 1, 27].

---

## 4. Pencatatan Jejak Audit Sistem (System Audit Logging)

Untuk menjamin kepatuhan dan keamanan operasional, seluruh aktivitas perubahan kritis yang dilakukan administrator dicatat ke tabel `system_audit_logs`[cite: 1]:

| Atribut Audit | Tipe Data | Deskripsi & Tujuan |
| :--- | :--- | :--- |
| `id` | UUID | Identitas unik entri audit[cite: 1]. |
| `admin_id` | UUID (FK) | Relasi ke akun administrator yang mengeksekusi tindakan (tabel `admins`)[cite: 1]. |
| `action_type` | VARCHAR(50) | Kategori operasi: `UPLOAD_DATASET`, `RETRAIN_MODEL`, `SWITCH_MODEL`, `DELETE`[cite: 1]. |
| `module_name` | VARCHAR(50) | Modul internal yang mengalami perubahan (misal: `ML_REGISTRY`, `DATASET_MGR`)[cite: 1]. |
| `description` | TEXT | Rincian deskripsi tindakan yang dijalankan (misal: *Mengaktifkan model RF v1.0.0*)[cite: 1]. |
| `ip_address` | VARCHAR(45) | Alamat IP jaringan perangkat admin saat melakukan perubahan[cite: 1]. |
| `user_agent` | TEXT | Informasi browser dan sistem operasi yang digunakan admin[cite: 1]. |
| `created_at` | TIMESTAMP | Waktu stempel presisi pencatatan aktivitas[cite: 1]. |

---

## 5. Standar Pencatatan Log Aplikasi (Application Logging Format)

Layanan Flask dikonfigurasi untuk mengeluarkan log berformat terstruktur (*structured logging*) ke `stdout` container Docker:

```json
{
  "timestamp": "2026-10-20T17:50:00.123Z",
  "level": "INFO",
  "service": "houseprice-backend",
  "endpoint": "/api/v1/predict",
  "method": "POST",
  "status_code": 200,
  "latency_ms": 118,
  "client_ip": "180.252.164.21",
  "message": "Inference executed successfully with model v1.0.0"
}
```

### Kategori Tingkat Log (Log Levels):
* **`DEBUG`:** Informasi diagnostik mendalam untuk penelusuran galat lokal (hanya aktif pada mode pengembangan)[cite: 40].
* **`INFO`:** Kejadian normal sistem operasional (permintaan inferensi sukses, pemuatan rute, migrasi selesai)[cite: 27, 40].
* **`WARNING`:** Anomali yang tidak menghentikan layanan (latensi inferensi $> 1.500$ ms, nilai input mendekati batas ekstrem)[cite: 27, 40].
* **`ERROR`:** Kegagalan fungsi yang memicu kode status HTTP 4xx atau 5xx (format JSON salah, kegagalan koneksi database)[cite: 27, 40].
* **`CRITICAL`:** Kegagalan fatal sistem (berkas model `.pkl` hilang atau korup, pool koneksi basis data habis)[cite: 27, 40].

---

## 6. Pemantauan Beban Kontainer Docker (Container Resource Monitoring)

Pada platform **Coolify**, metrik infrastruktur server dipantau secara otomatis melalui dashboard grafis bawaan:
* **Penggunaan Memori (RAM Usage):** Alokasi memori layanan Flask dipantau agar tidak melebihi kapasitas RAM yang tersedia saat memuat model scikit-learn ke memori.
* **Beban Prosesor (CPU Utilization):** Memantau beban CPU saat proses inferensi konkuren dan saat proses *retraining* dataset dijalankan di latar belakang.
* **Volume Disk Database:** Memantau pertumbuhan ukuran penyimpanan basis data PostgreSQL untuk memastikan tabel `prediction_logs` tidak membebani ruang penyimpanan server.