## Deskripsi Perubahan
<!-- Jelaskan secara ringkas perubahan apa yang dibuat, alasan perubahan tersebut, dan konteks latar belakangnya. -->

## Tipe Perubahan
Pilih salah satu atau beberapa tipe perubahan berikut:
- [ ] `feat`: Fitur baru untuk pengguna atau sistem
- [ ] `fix`: Perbaikan bug atau cacat kode
- [ ] `refactor`: Refaktor struktur kode tanpa mengubah fungsi bisnis
- [ ] `docs`: Pembaruan atau penambahan dokumentasi
- [ ] `test`: Penambahan atau pembaruan skrip pengujian (PMPL)
- [ ] `chore`: Pemeliharaan dependensi, konfigurasi build, atau CI/CD

## Tautan Task / Issue
* Mengacu ke Task: `MOB-xx` / `ML-xx` / `QA-xx`
* Menutup Issue: Closes #

## Domain Mata Kuliah Terintegrasi
Tandai lingkup mata kuliah yang terdampak oleh Pull Request ini:
- [ ] **Pemrograman Mobile** (Flutter / Layered Architecture / UI / SQLite)
- [ ] **Pembelajaran Mesin** (Scikit-Learn / Preprocessing / Model / Flask API)
- [ ] **Penjaminan Mutu Perangkat Lunak** (BVA, EP, Usability, Defect Log, STR)

---

## Verifikasi & Bukti Pengujian Mutu (PMPL Checklist)
Setiap pengajuan PR wajib melampirkan bukti pengujian fungsional dan penjaminan mutu:

- [ ] **Lolos Pengujian Statis & Linter:**
  - Mobile: `flutter analyze` bebas dari *warning* dan *error*.
  - Backend: `flake8 .` mematuhi standar PEP8.
- [ ] **Pengujian Fungsional / Unit Test:**
  - Seluruh pengujian lokal (`flutter test` / `pytest`) lolos 100%.
  - Matriks pengujian batas (BVA / EP) telah dipenuhi untuk modul form masukan.
- [ ] **Tangkapan Layar / Bukti Eksekusi (Opsional jika relevan):**
  <!-- Tempelkan tangkapan layar antarmuka aplikasi, cuplikan terminal hasil test runner, atau respons Postman di bawah ini -->
  > *[Tangkapan Layar / Log Output]*

---

## Checklist Pengusul (Author Checklist)
Pastikan hal-hal berikut telah diverifikasi sebelum meminta peninjauan (*review*):
- [ ] Kode telah mengikuti konvensi penamaan dan panduan arsitektur tim.
- [ ] Tidak menyertakan kredensial rahasia, kunci API, atau file `.env` ke dalam commit.
- [ ] Komentar kode ditambahkan pada bagian yang memerlukan penjelasan matematis/algoritma rumit.
- [ ] Dokumentasi terkait (README / API docs / Kamus Data) telah disesuaikan jika ada perubahan skema atau kontrak fungsi.
- [ ] Telah menetapkan minimal 1 peninjau (*reviewer*) sesuai pemetaan `CODEOWNERS`.