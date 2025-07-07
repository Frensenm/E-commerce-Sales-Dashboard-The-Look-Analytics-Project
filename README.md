# E-commerce Sales Dashboard - The Look Analytics Project

## 🚀 Live Dashboard

[![Lihat Dashboard](https://img.shields.io/badge/Looker%20Studio-Live%20Dashboard-blue?style=for-the-badge&logo=looker-studio)](https://lookerstudio.google.com/reporting/a212e3ac-7752-4385-8ae0-2ff248262636/page/p_iqqp9qe2td)

---

## 📖 Project Overview

Proyek ini bertujuan untuk menganalisis data penjualan dari "The Look", sebuah perusahaan e-commerce fiktif. Tujuannya adalah untuk membangun sebuah dashboard interaktif yang dapat digunakan oleh tim manajemen untuk memonitor Key Performance Indicators (KPI) dan menemukan insight strategis terkait performa produk, demografi pelanggan, dan efektivitas kanal pemasaran. Seluruh proses, mulai dari transformasi data hingga visualisasi, dilakukan menggunakan ekosistem Google Cloud Platform.

---

## 🛠️ Tech Stack

* **Data Transformation**: Google BigQuery (SQL)
* **Data Visualization**: Looker Studio
* **Dataset**: `thelook_ecommerce` (Public Dataset di BigQuery)

---

## ⚙️ Alur Kerja Proyek

Proyek ini dibagi menjadi dua tahap utama:

### 1. Data Preparation & Transformation (BigQuery)

Tahap ini berfokus pada pembersihan, penggabungan, dan rekayasa fitur (feature engineering) dari data mentah untuk membuat satu tabel utama yang siap untuk dianalisis.

* **Penggabungan Data**: Menggabungkan 5 tabel sumber (`orders`, `order_items`, `users`, `products`, `inventory_items`) menggunakan `LEFT JOIN` untuk menciptakan satu sumber data yang komprehensif.
* **Feature Engineering**: Membuat kolom-kolom kalkulasi baru untuk analisis yang lebih dalam, seperti:
    * `profit`: Menghitung keuntungan per item.
    * `processing_duration_days`, `shipping_duration_days`: Menghitung durasi operasional.
    * `age_category`: Mengelompokkan usia pelanggan ke dalam segmen.
    * `is_returned`: Flag boolean untuk item yang diretur.
* **Pembersihan Data**: Memfilter data - data yang tidak relevan (pesanan di masa depan) untuk memastikan analisis yang akurat.

<details>
<summary>Lihat Kode Final SQL di BigQuery</summary>

```sql
WITH
  ecommerce_main_data AS (
    -- Proses JOIN
    SELECT
    OI.id as order_item_id,
    OI.order_id,
    OI.user_id,
    OI.product_id,
    OI.inventory_item_id,
    OI.sale_price as product_saleprice,

    IV.cost as product_cost,
    
    O.status as status_order,
    O.created_at,O.delivered_at,O.shipped_at,O.returned_at,
    O.num_of_item,

    P.name as product_name,
    P.category as product_category,
    P.brand as product_brand,
    P.department as department,

    U.age, U.gender,
    U.state,U.country,U.city,
    U.traffic_source,
    U.created_at as user_regist,

    (OI.sale_price - IV.cost) as profit,
    TIMESTAMP_DIFF(O.shipped_at, O.created_at, DAY) as processing_duration_days,
    TIMESTAMP_DIFF(O.delivered_at, O.shipped_at, DAY) as shipping_duration_days,
    TIMESTAMP_DIFF(O.delivered_at, O.created_at, DAY) AS total_duration_days,

    CASE
      WHEN U.age BETWEEN 12 AND 17 THEN "Remaja"
      WHEN U.age BETWEEN 18 AND 30 THEN "Dewasa Muda"
      WHEN U.age BETWEEN 31 AND 50 THEN "Dewasa Menengah"
      WHEN U.age BETWEEN 51 AND 70 THEN "Lansia Awal"
      ELSE "Lainnya"
    END,

    CASE 
    WHEN O.returned_at IS NOT NULL THEN TRUE 
    ELSE FALSE 
    END AS is_returned

    FROM `bigquery-public-data.thelook_ecommerce.order_items` as OI
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.inventory_items` as IV ON OI.inventory_item_id = IV.id
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.orders` as O ON OI.order_id = O.order_id
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.products` as P ON OI.product_id = P.id
    LEFT JOIN `bigquery-public-data.thelook_ecommerce.users` as U ON OI.user_id = U.id

    WHERE 
    O.status NOT IN ('Processing') 
    AND
    O.created_at < CURRENT_TIMESTAMP()
  )

SELECT * FROM ecommerce_main_data;
```

</details>

### 2. Data Visualization & Analysis (Looker Studio)

Data yang telah diolah kemudian divisualisasikan dalam sebuah dashboard interaktif 4 halaman di Looker Studio. Setiap halaman dirancang untuk menjawab pertanyaan bisnis spesifik.

---

## 📊 Dashboard & Key Insights

### Halaman 1: Overview
Halaman ini memberikan ringkasan kinerja bisnis secara keseluruhan.

![Overview](Overview.png)

* **Pertanyaan Bisnis**: Bagaimana kinerja penjualan, siapa pelanggan kita, dan dari mana sumber pendapatan terbesar?
* **Key Insights & Recommendations**:
    * **Insight**: Kanal 'Search' menjadi pendorong revenue terbesar, namun kanal 'Email' memiliki Average Order Value (AOV) yang lebih tinggi.
    * **Rekomendasi**: Lakukan kampanye retargeting melalui email kepada pelanggan yang sudah ada dengan menawarkan produk komplementer untuk meningkatkan nilai transaksi.

### Halaman 2: Products
Halaman ini berfokus pada analisis kinerja portofolio produk.

![Products](Products.png)

* **Pertanyaan Bisnis**: Produk dan kategori mana yang paling laku dan paling menguntungkan?
* **Key Insights & Recommendations**:
    * **Insight**: Scatter plot menunjukkan bahwa banyak produk populer (revenue tinggi) memiliki profit margin yang rendah. Produk dalam kategori "Jeans" secara konsisten menunjukkan profitabilitas dan volume penjualan yang tinggi.
    * **Rekomendasi**: Lakukan analisis biaya pada produk bervolume tinggi dengan margin rendah untuk mencari peluang efisiensi. Prioritaskan stok dan promosi untuk produk "star" seperti pada kategori "Jeans".

### Halaman 3: Customers
Halaman ini menganalisis demografi dan perilaku pelanggan.

![Customers](Customers.png)

* **Pertanyaan Bisnis**: Bagaimana perbedaan profil dan perilaku belanja antara pelanggan pria dan wanita?
* **Key Insights & Recommendations**:
    * **Insight**: Komposisi pelanggan hampir seimbang 50/50 antara pria dan wanita. Namun, data menunjukkan pelanggan pria cenderung memiliki AOV sedikit lebih tinggi, terutama pada kategori "Outerwear & Coats".
    * **Rekomendasi**: Buat kampanye marketing yang ditargetkan secara spesifik untuk koleksi "Outerwear & Coats" kepada audiens pria menjelang musim tertentu.

### Halaman 4: Time
Halaman ini menganalisis tren dan performa dari waktu ke waktu.

![Time](Time.png)

* **Pertanyaan Bisnis**: Bagaimana tren pertumbuhan bisnis dan efisiensi konversi secara bulanan?
* **Key Insights & Recommendations**:
    * **Insight**: Menunjukkan tren pertumbuhan revenue yang sangat kuat dan konsisten dari bulan ke bulan, dengan akselerasi yang signifikan memasuki tahun 2025. Meskipun revenue meningkat pesat, tingkat konversi (conversion rate) cenderung stagnan dan stabil
    * **Rekomendasi**: Untuk Strategi Bisnis, mengingat momentum pertumbuhan revenue yang sudah kuat, peluang terbesar selanjutnya adalah fokus pada peningkatan Conversion Rate. 

---
