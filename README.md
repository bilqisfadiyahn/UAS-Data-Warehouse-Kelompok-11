# 🛒 Proyek Data Warehouse: Perancangan Gudang Data dan Analisis Multidimensi Performa Bisnis E-Commerce Olist Brazil Menggunakan Star Schema dan Atoti OLAP Cube

## 📖 Deskripsi Proyek

Proyek ini merupakan implementasi arsitektur **Data Warehouse** menggunakan pendekatan **Star Schema** dengan memanfaatkan **Brazilian E-Commerce Public Dataset by Olist** sebagai sumber data.

Sistem dibangun menggunakan **Supabase (PostgreSQL)** sebagai repositori data terpusat dan **Atoti** sebagai platform analisis multidimensi (OLAP). Proyek ini mensimulasikan proses Data Warehouse mulai dari **Data Staging (ETL)**, **Periodic Loading**, **Database Optimization**, hingga **OLAP Dashboard & Business Intelligence**.

---

## 👥 Penyusun

Proyek ini disusun oleh **Kelompok 11** sebagai bagian dari tugas mata kuliah Data Warehouse.

- Ruthtatia Grace Astridia (24031554072)
- Bilqis Fadiyah Nisrina (24031554216)
- Frelin Theresia Pania (24031554220)

---

## 💾 Dataset

Dataset yang digunakan adalah **Brazilian E-Commerce Public Dataset by Olist**. Dataset ini berisi data transaksi e-commerce di Brasil selama periode tahun 2016–2018 yang mencakup informasi:

* *Customer Data* (informasi pelanggan)
* *Order Data* (informasi pesanan)
* *Order Items Data* (detail produk pada setiap pesanan)
* *Product Data* (informasi produk)
* *Seller Data* (informasi penjual)
* *Payment Data* (informasi pembayaran)
* *Review Data* (ulasan dan rating pelanggan)
* *Geolocation Data* (lokasi pelanggan dan penjual)
* *Product Category Translation Data* (terjemahan kategori produk)

🔗 **Link Sumber Dataset:** [Kaggle Olist Dataset](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

> ⚠️ **Catatan Penting:** Folder `Raw Data` (data mentah asli dari Kaggle) tidak di-upload ke dalam repository GitHub ini karena ukuran filenya yang terlalu besar. Jika ingin menjalankan ulang proses dari awal, silakan unduh dataset melalui link Kaggle di atas dan masukkan ke dalam folder lokal bernama `Raw Data/`.

---

## 📂 Struktur Folder Repository

* `Code/` : Berisi script Python / Jupyter Notebook untuk proses ETL dan pembuatan Atoti OLAP Cube.
* `Dataset/` : Berisi file data bersih hasil staging yang digunakan untuk pemodelan.
* `periodic_batches/` : Script atau file batch data untuk simulasi pemuatan berkala.
* `supabase_sql/` : Kumpulan query SQL untuk pembuatan tabel, partisi, indeks, dan materialized view di Supabase.

---

## 🏗️ Arsitektur Data Warehouse

```text
Raw Dataset
     │
     ▼
Data Staging (ETL)
     │
     ▼
Star Schema
     │
     ▼
Periodic Load
     │
     ▼
Supabase PostgreSQL
     │
 ┌───┼─────────┐
 ▼   ▼         ▼
Partitioning  Indexing  Materialized View
     │
     ▼
Atoti OLAP Cube
     │
     ▼
Dashboard & Business Insight
```

---

# ⚙️ Data Pipeline

## 1. Data Staging (ETL)

Tahap ETL dilakukan menggunakan Python dan Pandas untuk menyiapkan data sebelum dimuat ke dalam Data Warehouse.

### 🔹 Data Profiling

Pemeriksaan dilakukan terhadap:

- Jumlah baris dan kolom
- Missing value
- Data duplikat
- Tipe data atribut
- Integritas relasi antar tabel
- Sparsity dataset
- Imbalance ratio
- Sebaran outlier

### 🔹 Hasil Validasi

- Tidak ditemukan duplikasi pada `customer_id`
- Tidak ditemukan duplikasi pada `order_id`
- Tidak ditemukan order tanpa customer
- Relasi antar tabel valid untuk proses integrasi

### 🔹 Data Cleaning

#### Penanganan Missing Value

**Tabel Products**

- Missing value numerik diisi menggunakan median
- `product_category_name` diisi dengan `"Unknown Category"`

**Tabel Reviews**

- `review_comment_title` diisi dengan `"No Title"`
- `review_comment_message` diisi dengan `"No Review"`

### 🔹 Transformasi Datetime

Kolom tanggal pada tabel Orders dan Reviews dikonversi ke format datetime untuk mendukung analisis berbasis waktu.

### 🔹 Feature Engineering

Dibuat atribut waktu tambahan:

- year
- month
- day
- quarter
- date

Atribut ini digunakan untuk mendukung analisis hierarki waktu.

### 🔹 Integrasi Data

Seluruh tabel digabungkan menggunakan proses join untuk membentuk tabel staging terintegrasi.

| Metrik | Nilai |
|---------|---------|
| Jumlah Baris | 119.143 |
| Jumlah Kolom | 44 |

---

## 2. Pembentukan Star Schema

### Fact Table

### `fact_sales`

Berisi:

- order_id
- customer_id
- product_id
- seller_id
- payment_value
- review_score
- year
- month
- quarter
- date

### Dimension Tables

### `dim_customer`

- customer_id
- customer_city
- customer_state

### `dim_product`

Berisi informasi produk.

### `dim_seller`

Berisi informasi penjual.

### `dim_time`

- date
- year
- month
- day
- quarter

---

## 3. Persiapan Data Analitik Seller

Untuk mendukung dashboard analitik, dibuat dataset tambahan berbasis seller melalui proses agregasi data.

Dataset ini digunakan untuk analisis:

### 📌 Seller Performa Terbaik

Menghasilkan informasi seperti:

- seller_id
- total transaksi
- total pendapatan
- rata-rata rating pelanggan

### 📌 Seller Berdasarkan Wilayah

Menghasilkan informasi seperti:

- seller_state
- seller_city
- jumlah seller
- total pendapatan wilayah

Dataset hasil agregasi ini disimpan dalam format CSV untuk digunakan pada tahap visualisasi dan OLAP.

---

## 4. Simulasi Periodic Load

Untuk mensimulasikan proses periodic loading pada lingkungan Data Warehouse, data transaksi dibagi menjadi beberapa batch berdasarkan periode waktu.

| Batch | Periode | Jumlah Data |
|---------|----------------------------|-------------|
| Batch 1 | Hingga Tahun 2017 | 49.536 |
| Batch 2 | Januari – April 2018 | 30.234 |
| Batch 3 | Mei 2018 – Akhir Dataset | 27.677 |

Pembagian ini mensimulasikan proses pemuatan data secara berkala yang umum digunakan pada implementasi Data Warehouse.

---

## 5. Database Implementation & Performance Optimization (Supabase)

Setelah proses ETL selesai dilakukan, data hasil staging diekspor ke format CSV dan dimuat ke dalam Supabase.

### Loading Data

**Fact Table**

- fact_sales

**Dimension Tables**

- dim_customer
- dim_product
- dim_seller
- dim_time

### Optimasi Database

- Table Partitioning
- Materialized View
- B-Tree Indexing
- Query Optimization

### Benchmark

Pengujian menggunakan:

```sql
EXPLAIN ANALYZE
```

Hasil benchmark menunjukkan waktu eksekusi query rata-rata:

| Kondisi | Waktu Eksekusi |
|----------|----------------|
| Setelah Optimasi | 0.05 – 0.06 ms |

---

## 6. Data Presentation & OLAP (Atoti)

Tahap akhir dilakukan menggunakan Atoti untuk membangun lingkungan OLAP dan dashboard analitik.

### Yang dilakukan:

- Pembangunan Kubus OLAP
- Pembuatan Custom Measures
- Slice and Dice Analysis
- Drill Down Analysis
- Dashboard Interaktif
- Penyusunan Insight Bisnis

Dashboard yang dihasilkan mencakup analisis seperti:

- Tren penjualan
- Seller performa terbaik
- Seller berdasarkan wilayah
- Pendapatan
- Kepuasan pelanggan

---

# 📚 Konsep yang Diimplementasikan

- Data Warehouse
- ETL (Extract, Transform, Load)
- Star Schema
- Periodic Load
- Table Partitioning
- Materialized View
- B-Tree Indexing
- Query Optimization
- OLAP Cube
- Business Intelligence Dashboard
