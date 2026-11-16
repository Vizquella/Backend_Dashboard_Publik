# ⚙️ Backend API Dashboard Surveilans Kesehatan

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115%2B-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Pandas](https://img.shields.io/badge/Pandas-2.2%2B-150458?logo=pandas&logoColor=white)](https://pandas.pydata.org/)
[![License](https://img.shields.io/badge/Security-Data_Sanitized-emerald)](#keamanan-data--privasi)

Layanan backend REST API untuk sistem analitik surveilans epidemiologi puskesmas. Bertugas mengolah data mentah rekam medis multi-sheet Excel, melakukan sanitasi privasi otomatis, memetakan kode klasifikasi penyakit internasional (**ICD-10**), dan menyajikan data agregat berkinerja tinggi untuk frontend.

---

## 🔒 Keamanan Data & Kepatuhan Privasi (UU PDP)

Sistem ini menerapkan prinsip **Privacy by Design**:
* **Pembersihan Kolom Sensitif (*Sanitization*):** Saat file Excel rekam medis diunggah melalui endpoint `/api/upload` atau `/api/auto_sync`, sistem hanya mengambil kolom yang dibutuhkan untuk analitik epidemiologi (`Tanggal`, `Jenis Kelamin`, `Umur`, `Diagnosa`, `ICD`).
* **Pencegahan Kebocoran:** Kolom identitas langsung seperti NIK, Nama Lengkap Pasien, Nomor Rekam Medis, dan Alamat Rumah **secara otomatis dibuang** sebelum data disimpan ke disk server.
* **Isolasi Git:** File spreadsheet rekam medis (`*.xlsx`, `*.csv`) dan file data lokal (`dashboard_data.json`) secara ketat diblokir oleh `.gitignore` sehingga tidak akan pernah terunggah ke repositori publik.

---

## 🚀 Fitur Backend

1. **Pre-Calculated Multi-Puskesmas Aggregation:**
   * Menghitung metrik global seluruh kabupaten/kota serta metrik spesifik masing-masing puskesmas secara *in-memory* dalam waktu < 0.5 detik saat proses upload.
   * Menyimpan cache agregat ke dalam `dashboard_data.json`.
2. **Ultra-Fast Endpoint (`GET /api/dashboard`):**
   * Tidak membaca ulang file Excel per permintaan HTTP.
   * Merespon dalam waktu **< 5 milidetik**, menghemat CPU dan memori server secara signifikan.
3. **Konfigurasi Diagnosa & ICD-10 Dinamis:**
   * Mendukung pemetaan kode penyakit fleksibel (berbasis ekspresi reguler / regex).
   * Mendukung penyimpanan logo instansi kustom (Data URL) dan preferensi visual admin.

---

## 📡 Dokumentasi Endpoint REST API

| Method | Endpoint | Deskripsi |
| :--- | :--- | :--- |
| `GET` | `/api/dashboard` | Mengambil seluruh data agregat surveilans (metadata, diagnosa, tren harian, demografi usia, dan per-puskesmas). Mendukung parameter query opsional `?puskesmas=<nama>`. |
| `POST` | `/api/upload` | Mengunggah file Excel rekam medis baru, membersihkan data, dan memperbarui konfigurasi diagnosa/tampilan. |
| `POST` | `/api/auto_sync` | Endpoint otomatisasi untuk sinkronisasi data langsung dari ekstensi peramban atau SIMPUS eksternal. |
| `GET` | `/docs` | Dokumentasi interaktif Swagger UI bawaan FastAPI. |

---

## 💻 Panduan Instalasi & Menjalankan

### 1. Prasyarat
* Python 3.10 atau versi yang lebih baru.

### 2. Pemasangan Dependensi
Disarankan menggunakan lingkungan virtual (*virtual environment*):
```bash
# Membuat virtual environment
python -m venv venv

# Mengaktifkan virtual environment
# Windows:
venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

# Memasang pustaka dependensi
pip install -r requirements.txt
```

### 3. Menjalankan Server
```bash
python -m uvicorn main:app --host 127.0.0.1 --port 8000 --reload
```
Aplikasi akan aktif di **`http://127.0.0.1:8000`** dan dokumentasi interaktif di **`http://127.0.0.1:8000/docs`**.

---

## 📋 Struktur Input File Excel (.xlsx)

File Excel yang diunggah harus memiliki struktur:
* **Multi-Sheet:** Setiap sheet dinamai sesuai nama Puskesmas (contoh: *Sangasanga*, *Handil Baru*, *Loa Janan*).
* **Kolom Wajib:** `Tanggal`, `Jenis Kelamin`, `Umur Tahun`, `Diagnosa 1`, `ICD-X 1`.
