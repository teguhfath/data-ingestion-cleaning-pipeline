# End-to-End Data Engineering Pipeline: Web Scraping, Data Cleaning & SQL Transformation (Session 9)

Repository ini berisi hasil pembelajaran dan implementasi praktis dari **Sesi 9 — SQL, Web Scraping & Data Cleaning** pada program **AI Engineer Bootcamp by DigitalSkola**. 

Proyek ini mendemonstrasikan dua pendekatan arsitektur *data pipeline* modern secara end-to-end menggunakan Python dan PostgreSQL:
1. **Path A — ETL (Extract, Transform, Load):** Melakukan *web scraping* multi-sumber (RSS Feeds & HTML static pages), membersihkan serta menstandardisasi data menggunakan `pandas` sebelum dimuat ke database.
2. **Path B — ELT (Extract, Load, Transform):** Memuat dataset mentah (`employee_attrition.csv`) langsung ke PostgreSQL dan melakukan transformasi serta analisis agregasi menggunakan SQL tingkat lanjut.

---

## Struktur Proyek & Analisis Komponen

```text
.
├── scraper.py               # Modul Extract: mengambil data dari RSS feeds & situs web
├── cleaner.py               # Modul Transform (ETL): pembersihan teks, HTML stripping, & deduplikasi
├── db.py                    # Modul Load & DB Management: koneksi SQLAlchemy, DDL, & upsert logic
├── pipeline.py              # Orchestrator utama yang menjalankan alur ETL & ELT
├── employee_attrition.csv   # Dataset mentah IBM HR Analytics Employee Attrition
├── transform_attrition.sql  # Kumpulan query SQL analitik & window functions untuk Path ELT
├── requirements.txt         # Daftar pustaka Python yang dibutuhkan
└── .env                     # Konfigurasi environment (Database URL & Student Name)
```

### Rincian Fungsi Setiap File

1. **`scraper.py` (Data Extraction)**
   * Mengambil data artikel berita dari berbagai RSS feed (`BBC`, `NYTimes`, dll.) menggunakan `feedparser` dan `requests`.
   * Melakukan *scraping* struktur HTML statis (seperti `books.toscrape.com` dan `quotes.toscrape.com`) menggunakan `BeautifulSoup4` dengan pengaturan *User-Agent* dan *rate limiting* (`time.sleep`) yang aman.

2. **`cleaner.py` (Data Cleaning & Standardization)**
   * Menghapus sisa tag HTML dari konten teks, membersihkan *whitespace* berlebih, dan menormalisasi format tanggal publikasi (`published_at`) ke format standar `datetime`.
   * Menangani nilai kosong (*missing values*) dan melakukan deduplikasi berdasarkan URL unik untuk menjaga integritas data sebelum masuk ke database.

3. **`db.py` (Database Layer & Idempotent Loading)**
   * Mengelola koneksi ke PostgreSQL menggunakan `SQLAlchemy` dan konfigurasi `pydantic-settings` dari file `.env`.
   * Membuat tabel target `ai_engineer3.articles_<student_name>` beserta *index* pada kolom `source` dan `published_at`.
   * Menerapkan mekanisme *safe insert* (`INSERT ... ON CONFLICT (url) DO NOTHING`) sehingga *pipeline* bersifat *idempotent* (aman dijalankan berulang kali tanpa duplikasi).

4. **`pipeline.py` (Pipeline Orchestrator)**
   * Mengoordinasikan eksekusi kedua jalur data:
     * Menjalankan alur **ETL**: `scrape` $\rightarrow$ `clean` $\rightarrow$ `save_articles()`.
     * Menjalankan alur **ELT**: membaca `employee_attrition.csv` dan memuatnya secara utuh (*raw load*) ke tabel `ai_engineer3.raw_attrition_<student_name>` menggunakan `pandas.to_sql()`.

5. **`transform_attrition.sql` (SQL Analytics & Transformation)**
   * Berisi rangkaian *query* analitik untuk mengolah tabel mentah HR Attrition di PostgreSQL, meliputi:
     * Tingkat *attrition* dan rata-rata pendapatan per `Department`.
     * Segmentasi masa kerja karyawan (`YearsAtCompany`) menggunakan ekspresi kondisional `CASE WHEN`.
     * Analisis dampak jam lembur (`OverTime`) terhadap tingkat *attrition*.
     * Pemeringkatan karyawan berpendapatan tertinggi di setiap departemen menggunakan *window function* `RANK() OVER (PARTITION BY "Department" ORDER BY "MonthlyIncome" DESC)`.

---

## Panduan Instalasi & Menjalankan Pipeline

### 1. Prasyarat
* Python 3.10 atau versi lebih baru
* Akses ke database PostgreSQL

### 2. Clone Repository & Install Dependensi
```bash
git clone <url-repository-anda>
cd <nama-folder-repository>

python -m venv venv
source venv/bin/activate  # Untuk Linux/Mac
# venv\Scripts\activate   # Untuk Windows

pip install -r requirements.txt
```

### 3. Konfigurasi Environment (`.env`)
Buat file `.env` pada direktori utama proyek dan isi kredensial database Anda:
```env
DATABASE_URL=postgresql://username:password@host:5432/database_name
STUDENT_NAME=nama_anda
```

### 4. Menjalankan End-to-End Pipeline
Jalankan skrip orchestrator untuk mengeksekusi proses *scraping*, *cleaning*, dan pemuatan data ke PostgreSQL:
```bash
python pipeline.py
```

### 5. Menjalankan Query Transformasi ELT
Setelah tabel `raw_attrition_<student_name>` berhasil dimuat ke database, jalankan *query* yang terdapat pada `transform_attrition.sql` melalui SQL client (DBeaver, pgAdmin, atau `psql`) untuk melihat hasil analisis HR Attrition.

---

## Teknologi yang Digunakan
* **Bahasa Pemrograman:** Python 3
* **Web Scraping:** `requests`, `beautifulsoup4`, `feedparser`
* **Data Manipulation:** `pandas`
* **Database & ORM:** PostgreSQL, `SQLAlchemy`, `psycopg2-binary`
* **Configuration Management:** `pydantic-settings`, `python-dotenv`