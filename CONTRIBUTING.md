# Pedoman Kontribusi Tim (Contributing Guidelines) - HousePrice

Dokumen ini memuat standar kolaborasi dan alur kerja pengembangan perangkat lunak bagi seluruh anggota tim pengembang proyek HousePrice pada GitHub Organization. Seluruh kontributor wajib mematuhi panduan ini untuk menjaga stabilitas basis kode dan konsistensi riwayat repositori.

---

## 1. Prinsip Kolaborasi Tim

1. **Satu Jalur Integrasi:** Cabang `develop` berfungsi sebagai pusat integrasi harian. Tidak ada anggota yang diizinkan melakukan *commit* atau *push* langsung ke cabang `main`[cite: 31, 33, 34].
2. **Review Sebelum Penggabungan:** Setiap penambahan fitur atau skrip uji wajib melalui proses *Pull Request* (PR) dan ditinjau oleh minimal satu rekan tim serta disetujui oleh *Project Manager*[cite: 33, 35].
3. **Pemisahan Peran & Tanggung Jawab:** Setiap anggota bertanggung jawab penuh atas cabang fitur yang menjadi tugas utamanya lintas mata kuliah terintegrasi (Mobile, ML, dan PMPL)[cite: 19, 28, 29].

---

## 2. Struktur Cabang Repositori (Branching Model)

Struktur cabang mengacu pada *Git Flow Terstruktur*[cite: 31, 33]:

* `main` (Protected): Cabang produksi yang hanya memuat versi rilis stabil untuk demonstrasi *milestone* dan evaluasi dosen[cite: 31, 33, 36].
* `develop`: Cabang integrasi aktif tempat bertemunya seluruh fitur yang telah diverifikasi[cite: 31, 33].
* `feat/<nama-fitur>`: Cabang pengembangan fitur baru[cite: 31, 33, 34].
* `test/<nama-fitur>`: Cabang khusus pembuatan skrip pengujian mutu perangkat lunak[cite: 31, 33, 36].
* `release/<versi>`: Cabang stabilisasi menjelang pengujian akhir atau rilis demonstrasi dosen[cite: 31, 36].
* `hotfix/<nama-perbaikan>`: Cabang perbaikan darurat jika terjadi kesalahan fatal pada cabang `main` saat demonstrasi langsung[cite: 37].

---

## 3. Alur Kerja Harian Pengembang (Daily Workflow)

### Langkah 1: Sinkronisasi Cabang Dasar
Sebelum membuat cabang baru atau memulai pengerjaan harian, selalu perbarui cabang `develop` lokal dari repositori pusat[cite: 34]:

```bash
git checkout develop
git pull origin develop
```

### Langkah 2: Pembuatan Cabang Fitur Baru
Buat cabang kerja baru dengan format penamaan yang telah ditentukan[cite: 34]:

* **Mobile App (`houseprice-mobile-app`):**
  * `feat/ui-design-system-home` (Jasmine)[cite: 31]
  * `feat/property-form-input` (Hanif)[cite: 31]
  * `feat/core-networking-state` (Nuril)[cite: 31]
  * `feat/valuation-kpr-feature` (Naufal)[cite: 31]
  * `feat/result-local-history` (Nurfinka)[cite: 31]
  * `test/widget-form-bva` (Jasmine)[cite: 31]

* **Backend & ML (`houseprice-backend-ml` & `ml-model-testing`):**
  * `feat/data-eda-visualization` (Hanif)[cite: 33]
  * `feat/data-clean-duplicates` (Jasmine)[cite: 33]
  * `feat/data-clean-missing` (Nurfinka)[cite: 33]
  * `feat/data-outlier-encoding` (Naufal)[cite: 33]
  * `feat/model-train-tuning` (Nuril)[cite: 33]
  * `test/api-integration-perf` (Hanif)[cite: 33]

Contoh perintah pembuatan cabang[cite: 34]:
```bash
git checkout -b feat/property-form-input
```

### Langkah 3: Melakukan Perubahan dan Commit
Kelompokkan perubahan secara logis dan gunakan format *Conventional Commits*[cite: 35, 37]:

```bash
git status
git add .
git commit -m "feat: implement form input fields and local validation"
```

Struktur pesan *commit* yang diizinkan[cite: 37, 38]:
* `feat`: Penambahan fitur fungsional baru[cite: 37].
* `fix`: Perbaikan *bug* atau galat logika[cite: 37, 38].
* `test`: Penambahan atau penyesuaian skrip uji[cite: 38].
* `refactor`: Perapian struktur kode tanpa mengubah fungsi[cite: 38].
* `chore`: Penyesuaian dependensi, pustaka, atau berkas konfigurasi[cite: 38].
* `docs`: Pembaruan dokumentasi Markdown[cite: 38].

### Langkah 4: Menjaga Keterkinian Kode (Rebase / Merge Sync)
Jika cabang `develop` pada remote telah diperbarui oleh anggota tim lain, sinkronkan cabang fitur aktif sebelum mengajukan PR[cite: 35]:

```bash
git checkout develop
git pull origin develop
git checkout feat/property-form-input
git merge develop
```

### Langkah 5: Mengunggah Cabang dan Membuka Pull Request
Unggah cabang kerja ke repositori organisasi[cite: 35]:

```bash
git push -u origin feat/property-form-input
```

Setelah itu, buka antarmuka GitHub Organization dan buat *Pull Request* dengan tujuan penggabungan (*base*) ke cabang `develop`[cite: 35].

---

## 4. Standar Pull Request (PR) & Kriteria Penggabungan

Setiap *Pull Request* wajib memenuhi kriteria berikut agar dapat disetujui (*merge criteria*):

1. **Deskripsi Jelas:** Mengisi template PR yang menjelaskan ringkasan perubahan, modul yang terdampak, serta hasil pengujian lokal.
2. **Tidak Ada Konflik:** Bebas dari konflik percabangan (*no merge conflicts*) terhadap target cabang `develop`.
3. **Lolos Pengujian Mandiri:** Melampirkan tangkapan layar bukti eksekusi pengujian fungsional atau pengujian unit.
4. **Persetujuan Minimal:** Memperoleh persetujuan (*approval*) dari anggota tim terkait sesuai pembagian modul pada `CODEOWNERS` dan persetujuan akhir dari *Project Manager*.

---

## 5. Prosedur Rilis Demonstrasi (Milestone Release)

Menjelang evaluasi *Milestone* tengah semester (Minggu 8), *Milestone* fungsional (Minggu 12), atau *Expo* akhir (Minggu 16), rilis dikelola secara terpusat oleh *Project Manager*[cite: 8, 28, 36]:

1. Buat cabang rilis dari `develop`:
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b release/v1.0-milestone1
   ```
2. Lakukan penyesuaian versi aplikasi dan konfigurasi lingkungan produksi[cite: 36].
3. Gabungkan cabang rilis ke `main` melalui PR khusus[cite: 36].
4. Berikan penanda rilis resmi (*Git Tag*)[cite: 36]:
   ```bash
   git checkout main
   git pull origin main
   git tag -a v1.0.0-milestone1 -m "Release Milestone 1: Core Models & Form UI"
   git push origin main --tags
   ```
5. Gabungkan kembali penyesuaian versi ke cabang `develop`[cite: 36, 37].