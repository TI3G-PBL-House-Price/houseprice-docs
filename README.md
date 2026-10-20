# HousePrice: Sistem Prediksi Harga Rumah Wilayah DKI Jakarta Berbasis Machine Learning & Mobile

[![PBL Semester 5](https://img.shields.io/badge/PBL-D4%20Teknik%20Informatika%20Polinema-blue.svg)](https://www.polinema.ac.id/)
[![Flutter](https://img.shields.io/badge/Mobile-Flutter%203.x-02569B?logo=flutter)](https://flutter.dev)
[![Flask](https://img.shields.io/badge/Backend-Flask%20Python-000000?logo=flask)](https://flask.palletsprojects.com/)
[![PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL%2015+-4169E1?logo=postgresql)](https://www.postgresql.org/)
[![License](https://img.shields.io/badge/License-Academic%20PBL-green.svg)]()

> Repositori Pusat Dokumentasi & Standar Arsitektur Sistem untuk Proyek Terintegrasi Product-Based Learning (PBL) Semester 5, Jurusan Teknologi Informasi, Politeknik Negeri Malang (T.A. 2026/2027).

---

## 📌 Ringkasan Eksekutif
Masyarakat, khususnya keluarga muda dan pendatang dari luar daerah, sering mengalami asimetri informasi saat membeli atau menjual rumah di DKI Jakarta. Sulit menilai apakah harga penawaran tergolong murah, wajar, atau terlalu mahal. 

**HousePrice** hadir sebagai solusi komprehensif terintegrasi:
1. **Aplikasi Mobile (Flutter):** Antarmuka pengguna untuk cek estimasi harga properti secara instan, mengevaluasi kewajaran penawaran harga penjual, simulasi kalkulator KPR, dan riwayat kalkulasi offline.
2. **RESTful Inference API (Flask):** Mesin inferensi cloud berlatensi rendah (< 1,5 detik) berbasis model Machine Learning Ensemble (Random Forest Regressor) yang dilatih menggunakan 10.000 data historis properti Jakarta.
3. **Web Dashboard Admin (Flask & Jinja/Bootstrap):** Portal khusus administrator teknis untuk pemantauan performa model ($R^2$, MAE, RMSE), unggah dataset baru, dan eksekusi retraining model terprogram.
4. **Basis Data Terpusat (PostgreSQL):** Penyimpanan relasional untuk master wilayah, log audit sistem, riwayat inferensi, dan metadata registrasi model.

---

## 👥 Tim Pengembang (Kelompok 2 - TI 2G)

| Nama Mahasiswa | NIM | Peran Utama & Fokus MK |
| :--- | :--- | :--- |
| **Abdul Rahman Hanif Darmawan** | 244107020232 | **Project Manager**, ML EDA & Visualisasi, Mobile Form & Validasi, Integration & Performance QA |
| **Muhammad Nuril Huda** | 244107020004 | ML Model Engineering & Tuning, Mobile Core Prediction & Networking, Usability QA |
| **Nurfinka Lailasari** | 244107020211 | ML Data Cleaning (Missing), Mobile Result UI & Local DB, QA Lead & Defect Tracking |
| **Naufal Abid Aurizky** | 244107020212 | ML Outlier Handling & Preprocessing, Mobile Business Logic (Kewajaran & KPR), Logic QA |
| **Jasmine Nasywa Nabilah** | 244107020119 | ML Duplicate Cleaning, Mobile Home UI & Design System, Blackbox BVA/EP QA |

### 👨‍🏫 Dosen Pembimbing Mata Kuliah Terintegrasi
* **Pemrograman Mobile:** Habibie Ed Dien, S.Kom., M.T.
* **Pembelajaran Mesin:** Usman Nurhasan, S.Kom., M.T.
* **Penjaminan Mutu Perangkat Lunak:** M. Hasyim Ratsanjani, S.Kom., M.Kom.

---

## 🗺️ Struktur Repositori Organisasi

Proyek ini dikelola dalam organisasi GitHub resmi menggunakan struktur multi-repo:

* [**`houseprice-docs`**](https://github.com/PBL-House-Price/houseprice-docs) (Repositori Ini): Blueprint arsitektur, PRD, standar QA, dan pedoman engineering.
* [**`houseprice-mobile-app`**](https://github.com/PBL-House-Price/houseprice-mobile-app): Kode sumber aplikasi Android Flutter (BLoC/Provider, SQLite/Shared Preferences).
* [**`houseprice-backend-ml`**](https://github.com/PBL-House-Price/houseprice-backend-ml): Layanan RESTful API Flask, Web Admin, skrip migrasi PostgreSQL, dan model final `.pkl`.
* [**`ml-model-testing`**](https://github.com/PBL-House-Price/ml-model-testing): Notebook riset eksperimen ML, eksplorasi EDA, hyperparameter tuning, dan benchmark model regresi.

---

## 🚀 Quick Start & Deployment Overview

Backend dan database dideploy menggunakan platform open-source **Coolify** dengan dukungan **Docker Compose**.

```bash
# Menjalankan backend dan database di server lokal / staging
git clone https://github.com/PBL-House-Price/houseprice-backend-ml.git
cd houseprice-backend-ml
cp .env.example .env
docker compose up -d --build
```