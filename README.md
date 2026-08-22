# Olist E-Commerce Sales & Logistics Performance Analysis
## Executive Summary
Proyek ini menganalisis dataset publik dari Olist—platform marketplace e-commerce terbesar di Brazil—yang mencakup lebih dari 100.000 transaksi dari periode 2016 hingga 2018. Mengingat e-commerce di Brazil menghadapi tantangan geografis dan logistik yang kompleks, analisis ini bertujuan untuk mengevaluasi dinamika penjualan, distribusi pelanggan, performa pengiriman, serta dampaknya terhadap kepuasan pelanggan (review score).

Menggunakan PostgreSQL, analisis dilakukan dengan menggabungkan dan mengagregasi 8 tabel data terelasi untuk mengekstrak business insight yang actionable. Hasil analisis ini memberikan gambaran menyeluruh tentang faktor-faktor penentu pertumbuhan revenue dan efisiensi rantai pasok Olist guna mendukung pengambilan keputusan berbasis data.

## Key Question
### Overall Business & Sales Dynamics:
* Bagaimana kinerja bisnis Olist secara keseluruhan, dan bagaimana tren pertumbuhan penjualan berubah dari waktu ke waktu (MoM & YoY)?
* Kategori produk mana yang memberikan kontribusi revenue terbesar serta mendominasi volume penjualan?
### Customer Retention & Value:
* Seberapa kuat tingkat retensi dan pembelian berulang (repeat purchase) pelanggan di platform Olist?
* Apakah pelanggan berulang (repeat customers) memiliki nilai transaksi (customer lifetime value) dan pola pembelian yang berbeda dibandingkan one-time customers?
### Seller Concentration & Operational Health:
* Seberapa terkonsentrasi revenue pada segmen seller tertentu, dan seller mana yang memiliki indikasi masalah operasional?
### Order Cancellations & Revenue Impact:
* Di mana pembatalan pesanan (cancellations) paling banyak terjadi, dan seberapa besar dampaknya terhadap potensi kerugian bisnis?
### Logistics & Customer Satisfaction:
* Seberapa baik performa Olist dalam mengirimkan pesanan tepat waktu, dan bagaimana keterlambatan pengiriman mempengaruhi tingkat kepuasan pelanggan (review score)?

## 3. Dataset & Database Schema

Analisis ini menggunakan dataset publik dari Olist yang disimpan dalam database PostgreSQL bernama **`olist_project`**. Database ini terdiri dari 9 tabel terelasi yang mencakup seluruh rantai transaksi e-commerce, mulai dari pemesanan, produk, pembayaran, hingga logistik dan ulasan pelanggan.

### Tabel Database (`olist_project`)

* **`orders`**: Menyimpan data transaksi utama (ID pesanan, ID pelanggan, status pesanan, dan *timestamp* proses pengiriman).
* **`order_items`**: Detail item di setiap pesanan (ID produk, ID penjual, harga item, dan biaya ongkir/*freight*).
* **`customers`**: Informasi pelanggan (ID unik pelanggan, lokasi kota, dan *state*).
* **`products`**: Spesifikasi produk (ID produk, kategori, ukuran, dan berat).
* **`sellers`**: Informasi penjual/mitra Olist (ID penjual, lokasi kota, dan *state*).
* **`order_reviews`**: Ulasan dan skor rating (1–5) yang diberikan oleh pelanggan untuk setiap pesanan.
* **`order_payments`**: Metode pembayaran (kartu kredit, *boleto*, voucher) dan jumlah cicilan (*installments*).
* **`geolocation`**: Data koordinat geospasial (kode pos, latitude, dan longitude) di wilayah Brazil.
* **`product_categories`**: Tabel translasi nama kategori produk dari bahasa Portugis ke bahasa Inggris.

### Entity Relationship Diagram (ERD)
<img width="2201" height="1049" alt="ERD" src="https://github.com/user-attachments/assets/ba3c0f30-ccdf-4162-85de-19a558ff006b" />
