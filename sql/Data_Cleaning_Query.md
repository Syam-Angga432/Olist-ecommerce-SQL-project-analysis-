# DATA CLEANING - OLIST E-COMMERCE DATASET
## Data Cleaning Execution
### 1. product_categories
untuk menambahkan `pc_gamer` dan `portateis_cozinha_e..`ke dapat kolom menggunakan `insert into`, namun pada data cleaning kali ini akan dibuat tabel baru, yaitu `product_categories_v2` sekaligus menambahkan `pc_gamer` dan `portateis_cozinha_e..` didalamnya. tujuan nya adalah agar tidak tidak mengubah data RAW, sehingga untuk proses analisis kedepannya menggunakan `product_categories_v2`

```sql
CREATE TABLE product_categories_v2 AS
-- 1. Ambil seluruh data kategori asli yang sudah ada
SELECT 
    product_category_name, 
    product_category_name_english
FROM product_categories

UNION ALL

-- 2. Tambahkan 2 kategori baru
SELECT 'pc_gamer' AS product_category_name, 'pc_gamer' AS product_category_name_english
UNION ALL
SELECT 'portateis_cozinha_e_preparadores_de_alimentos', 'Kitchen_Appliances_&_Food_Prep'
;

-- 3. tetapkan constraint kembali
ALTER TABLE product_categories_v2 
ADD CONSTRAINT pk_product_categories_v2 PRIMARY KEY (product_category_name)
;
```
#### 1.1 validation
```sql
SELECT count (*) FROM product_categories_v2;

select * from product_categories_v2
where product_category_name_english in ('pc_gamer','Kitchen_Appliances_&_Food_Prep');
```
### 2. products
610 NULL pada kolom `product_category_name` akan sulit diidentfikasi dengan sumber yang terbatas, misal butuh image product sehingga bisa mengetahui masuk kategori apa atau data terbaru dari tabel yang berbeda. namun karena sumber tersebut tidak tersedia salah satu cara paling aman adalah dengan mengubah null menjadi `unknown` atau `uncategorized` agar terdeteksi pada proses analisis kedepannya.

sama seperti sebelumnya akan dibuat tabel baru, yaitu `products_clean` agar tidak mengubah RAW data.
```sql
CREATE TABLE products_clean AS
SELECT
    product_id,
    COALESCE(
        product_category_name,
        'Uncategorized'
    ) AS product_category_name,
    product_name_lenght,
    product_description_lenght,
    product_photos_qty,
    product_weight_g,
    product_length_cm,
    product_height_cm,
    product_width_cm
FROM products;
```
#### 2.1 validation
```sql
select * from products_clean 
where product_category_name is null;

select 
product_category_name,
count (product_id) as jumlah_produk_
from products_clean 
where product_category_name = 'Uncategorized'
group by 1
;
```

### 3. orders
* null `order_approved_at`, `order_delivered_carrier_date`,`order_delivered_customer_date` lebih baik dibirkan saja, karena Menjaga Integritas Analisis SLA / Durasi Logistik serta membuat sistem sengaja mengabaikannya (exclude) saat perhitungan durasi pengiriman. namun bila ingin tetap melakukan cleaning pada timestamp yang paling mungkin adalah pada `order_approved_at` karena selisih waktunya cukup tipis dengan order_purchase_timestamp.
* `invalid_carrier_delivery` dan `invalid_delivery_sequence` dikarenakan urutan timestamp yang salah akan di null-kan dikarenakan tidak bisa secara sembarangan menentukan durasi masing-masing order dan tetap Menjaga Integritas Analisis SLA. sehingga pada tahap delivery performance order dengan timestamp null akan di abaikan(exclude)

```sql
CREATE TABLE orders_clean AS
WITH avg_approval AS (
    -- Menghitung rata-rata durasi approval untuk pesanan 'delivered'
    SELECT AVG(order_approved_at - order_purchase_timestamp) AS avg_duration
    FROM orders
    WHERE order_status = 'delivered'
      AND order_approved_at IS NOT NULL
      AND order_purchase_timestamp IS NOT NULL),
step1_impute_and_clean_carrier AS (
    -- Tahap 1: Imputasi order_approved_at + Pembersihan order_delivered_carrier_date
    SELECT 
        order_id,
        customer_id,
        order_status,
        order_purchase_timestamp,
        -- Imputasi approval date untuk status 'delivered'
        CASE 
            WHEN order_status = 'delivered' AND order_approved_at IS NULL 
            THEN order_purchase_timestamp + (SELECT avg_duration FROM avg_approval)
            ELSE order_approved_at
        END AS order_approved_at,
        -- Pembersihan carrier_date (harus >= purchase_timestamp)
        CASE 
            WHEN order_delivered_carrier_date >= order_purchase_timestamp
            THEN order_delivered_carrier_date
            ELSE NULL
        END AS order_delivered_carrier_date,
        order_delivered_customer_date,
        order_estimated_delivery_date
    FROM orders)
-- Tahap 2: Pembersihan customer_date menggunakan carrier_date yang SUDAH BERSIH dari Tahap 1
SELECT 
    order_id,
    customer_id,
    order_status,
    order_purchase_timestamp,
    order_approved_at,
    order_delivered_carrier_date,
    -- Pembersihan customer_date
    CASE 
        WHEN order_delivered_customer_date >= order_purchase_timestamp
        AND (order_delivered_carrier_date IS NULL 
        OR order_delivered_customer_date >= order_delivered_carrier_date)
        THEN order_delivered_customer_date
        ELSE NULL
        END AS order_delivered_customer_date,
        order_estimated_delivery_date
FROM step1_impute_and_clean_carrier;
```
#### 3.1 validation
a. target-->approved_at_null = 0
```sql
SELECT
    order_status,
    COUNT(*) AS total_orders,
    COUNT(*) - COUNT(order_approved_at) AS approved_at_null
from orders_clean 
where order_status = 'delivered'
group by 1;
```
b. target-->invalid_carrier_date = 0
```sql
SELECT
    COUNT(*) AS invalid_carrier_date
FROM orders_clean
WHERE order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date < order_purchase_timestamp;
```
c.  target-->invalid_delivery_sequence = 0
```sql
SELECT
    COUNT(*) AS invalid_delivery_sequence
FROM orders_clean
WHERE order_delivered_customer_date IS NOT NULL
  AND order_delivered_carrier_date IS NOT NULL
  AND order_delivered_customer_date < order_delivered_carrier_date;
```

### 4. ORDER_PAYMENTS
credit_card, payment_installments = 0 di ubah menjadi 1, Nilai 0 pada pembayaran kartu kredit merupakan anomali logika (data entry anomaly)
```sql
CREATE OR REPLACE VIEW order_payments_clean AS
SELECT
    order_id,
    payment_sequential,
    payment_type,
    CASE
        WHEN payment_type = 'credit_card'
             AND payment_installments < 1
        THEN NULL
        ELSE payment_installments
    END AS payment_installments,
    payment_value
FROM order_payments;
```
#### 4.1 validation
```sql
SELECT
    COUNT(*) FILTER (WHERE payment_value < 0) AS negative_payment,
    COUNT(*) FILTER (WHERE payment_installments < 1) AS invalid_installments
FROM order_payments_clean;
```

