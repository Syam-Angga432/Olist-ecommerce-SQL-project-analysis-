# DQA (DATA QUALITY ASSESSMENT)
* DATA QUALITY ASSESSMENT (DQA) - OLIST E-COMMERCE DATASET
* Database : olist_project
* Tool     : PostgreSQL

```text
├── 1. STRUKTUR QUERY UNTUK TAHAP DQA
├── 2. IMPLEMENTASI QUERY PADA MASING-MASING TABEL OLIST
│   ├── Data Overview (Cek sampel & jumlah baris)
│   ├── Missing Values Check (Cek kolom ber-NULL)
│   ├── Duplication Check (Cek duplikasi pada Primary Key)
│   └── DATA VALIDITY CHECK CHECK (Cek logika & keabsahan nilai)
│
├── 3. QDA FINDING
```
  Tahapan DQA:
  1. Data Overview (Cek sampel & jumlah baris)
  2. Missing Values Check (Cek kolom ber-NULL)
  3. Duplication Check (Cek duplikasi pada Primary Key)
  4. DATA VALIDITY CHECK CHECK (Cek logika & keabsahan nilai)
  
## STRUKTUR QUERY YANG DIGUNAKAN UNTUK TAHAP DQA

**1. Overview**
```sql
SELECT * FROM <table_name> LIMIT 10;
SELECT COUNT(*) AS total_rows FROM <table_name>;
```
**2. Missing Values (PostgreSQL Trick)**

**version 1**
```sql
SELECT
COUNT(*) AS total_NULL
FROM <table_name>
WHERE <column_1> is null or  <column_2> IS NULL or <column_3> is null or <column_4> is null;
```
** OR** 
```sql
SELECT *
FROM <table_name>
WHERE <column_1> is null or  <column_2> IS NULL or <column_3> is null or <column_4>
```
**version 2**
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(<column_1>) AS <column_1>_null_count,
    COUNT(*) - COUNT(<column_2>) AS <column_2>_null_count
FROM <table_name>;
```
**version 3**
```sql
SELECT 
    COUNT(*) AS total_rows,
    COUNT(order_approved_at) AS filled_rows,
    SUM(CASE WHEN order_approved_at IS NULL THEN 1 ELSE 0 END) AS null_count,
    round((avg(CASE WHEN order_approved_at IS NULL THEN 1 ELSE 0 END))*100,2) AS null_percentage
FROM orders;
```
**3. Duplication Check**

**version 1**
```sql
SELECT 
    COUNT(*) - COUNT(DISTINCT <column_1>) AS <column_1>_duplicat_count,
    COUNT(*) - COUNT(DISTINCT <column_2>) AS <column_2>_duplicat_count
FROM <table_name>;
```
**version 2**
```sql
SELECT
    <column_1>,
    COUNT(*) AS duplicate_count
FROM <table_name>
GROUP BY 1
HAVING COUNT(*) > 1
ORDER BY duplicate_count DESC;
```
**4. DATA VALIDITY CHECK CHECK**
```sql
SELECT COUNT(*) FILTER (WHERE <column_numeric> < 0) AS invalid_negative_values
FROM <table_name>;
```

## IMPLEMENTASI PADA TABEL OLIST
### 1. TABEL: product_categories
```sql
-- OVERVIEW & ROW COUNT
SELECT * FROM product_categories;
SELECT COUNT(*) AS total_rows FROM orders;

-- NULL CHECK
SELECT
COUNT(*) AS NULL
FROM  product_categories
WHERE  product_category_name is null or product_category_name_english IS NULL;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT product_category_name) AS category_name_duplikat,
	COUNT(*) - COUNT(DISTINCT product_category_name_english) AS name_english_duplikat
FROM product_categories;

-- DATA VALIDITY CHECK (kelengkapan produk kategory)
select p.product_category_name
from products p 
left join product_categories pc 
on pc.product_category_name = p.product_category_name
where pc.product_category_name is null and 
		p.product_category_name is not null
order by 1;
```
### 2. TABEL: customers
```sql
-- OVERVIEW & ROW COUNT
SELECT * FROM customers;
select count (*) as row_amount from customers;

-- NULL CHECK
SELECT
COUNT(*) AS NULL
FROM customers
WHERE customer_id is null or customer_unique_id IS NULL or customer_zip_code_prefix is null or
customer_city is null or customer_state is null;

SELECT
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(customer_id) AS customer_id_null,
    COUNT(*) - COUNT(customer_unique_id) AS unique_id_null,
    COUNT(*) - COUNT(customer_zip_code_prefix) AS zip_code_null,
    COUNT(*) - COUNT(customer_city) AS customer_city_null,
    COUNT(*) - COUNT(customer_state) AS customer_state
from  customers;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT customer_id) AS id_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_unique_id) AS unique_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_zip_code_prefix) AS zip_code_duplikat
FROM customers;
--		|
--		v
SELECT
customer_unique_id,
COUNT(*) 
FROM customers
GROUP BY customer_unique_id
HAVING COUNT(*)>1
order by 2 desc;
```
### 3. TABEL: geolocation
```sql
-- OVERVIEW & ROW COUNT
select * from geolocation;
select count (*) as row_amount from geolocation;

-- NULL CHECK
SELECT
COUNT(*) AS NULL
FROM geolocation
WHERE geolocation_zip_code_prefix is null or geolocation_lat IS NULL or geolocation_lng is null or
geolocation_city is null or geolocation_state is null;
```

### 4. TABEL: sellers
```sql
-- OVERVIEW & ROW COUNT
select * from sellers;
select count (*) as row_amount from sellers;

-- NULL CHECK
SELECT
COUNT(*) AS NULL
FROM sellers
WHERE seller_id is null or seller_zip_code_prefix IS NULL or seller_city is null 
or seller_state is null;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT seller_id) AS id_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_zip_code_prefix) AS zip_code_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_city) AS city_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_state) AS state_duplikat
FROM sellers;
```
### 5. TABEL: products
```sql
-- OVERVIEW & ROW COUNT
select * from products;
select count (*) as row_amount from products;

-- NULL CHECK (terdapat 610 product_category_name null)
SELECT
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(product_id) AS id_null,
    COUNT(*) - COUNT(product_category_name) AS category_null,
    COUNT(*) - COUNT(product_name_lenght) AS name_lenght_null,
    COUNT(*) - COUNT(product_description_lenght) AS description_lenght_null,
    COUNT(*) - COUNT(product_photos_qty) AS photos_qty_null,
	COUNT(*) - COUNT(product_weight_g) AS weight_g_null,
	COUNT(*) - COUNT(product_length_cm) AS length_cm_null,
	COUNT(*) - COUNT(product_height_cm) AS height_cm_null,
	COUNT(*) - COUNT(product_width_cm) AS width_cm_null
from  products;
--     |
--     v
SELECT *
FROM products
WHERE product_id is null or product_category_name IS NULL or product_name_lenght is null or product_description_lenght is null or
product_photos_qty is null or product_weight_g is null or product_length_cm is null or product_height_cm is null or product_width_cm
is null;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT product_id) AS id_duplikat
FROM products;

-- DATA VALIDITY CHECK
SELECT
    COUNT(*) FILTER (WHERE product_weight_g < 0) AS negative_weight,
    COUNT(*) FILTER (WHERE product_length_cm < 0) AS negative_length,
    COUNT(*) FILTER (WHERE product_height_cm < 0) AS negative_height,
    COUNT(*) FILTER (WHERE product_width_cm < 0) AS negative_width,
    COUNT(*) FILTER (WHERE product_photos_qty < 0) AS negative_photos
FROM products;
```
### 6. TABEL: orders
```sql
-- OVERVIEW & ROW COUNT
select * from orders;
select count(*) as jumlah_row from orders;

-- NULL CHECK
SELECT
    COUNT(*) AS total_rows,

    COUNT(*) - COUNT(order_id) AS order_id_null,
    COUNT(*) - COUNT(customer_id) AS customer_id_null,
    COUNT(*) - COUNT(order_status) AS _status_null,
    COUNT(*) - COUNT(order_purchase_timestamp) AS purchase_timestamp_null,
    COUNT(*) - COUNT(order_approved_at) AS approved_at_null,
	COUNT(*) - COUNT(order_delivered_carrier_date) AS delivered_carrier_date_null,
	COUNT(*) - COUNT(order_delivered_customer_date) AS delivered_customer_date_null,
	COUNT(*) - COUNT(order_estimated_delivery_date) AS estimated_null
from orders;
--    | 
--    v
SELECT * 
FROM orders
WHERE order_id is null or customer_id IS NULL or order_status is null or
order_purchase_timestamp is null or order_approved_at is null or order_delivered_carrier_date 
is null or order_delivered_customer_date is null or order_estimated_delivery_date is null;
--    |
--    v
SELECT
    order_status,
    COUNT(*) AS total_orders,
    COUNT(*) - COUNT(order_approved_at) AS approved_at_null,
    COUNT(*) - COUNT(order_delivered_carrier_date) AS carrier_date_null,
    COUNT(*) - COUNT(order_delivered_customer_date) AS customer_date_null
FROM orders
GROUP BY order_status
ORDER BY total_orders DESC;
--    |
--    v
SELECT * 
FROM orders
WHERE  order_status = 'delivered' and (order_approved_at is null or order_delivered_carrier_date 
is null or order_delivered_customer_date is null) 
order by order_approved_at,order_delivered_carrier_date, order_delivered_customer_date;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT order_id) AS order_id_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_id) AS customer_id_duplikat
FROM orders;

-- DATA VALIDITY CHECK

-- order_approved_at < order_purchase_timestamp
SELECT COUNT(*) AS invalid_purchase_approval
FROM orders
WHERE order_approved_at IS NOT NULL
  AND order_approved_at < order_purchase_timestamp;

-- order_delivered_carrier_date < order_purchase_timestamp
SELECT COUNT(*) AS invalid_carrier_delivery
FROM orders
WHERE order_delivered_carrier_date IS NOT NULL
  AND (order_delivered_carrier_date < order_purchase_timestamp);

-- order_delivered_customer_date < order_purchase_timestamp
SELECT COUNT(*) AS invalid_customer_delivery
FROM orders
WHERE order_delivered_customer_date IS NOT NULL
  AND order_delivered_customer_date < order_purchase_timestamp;

-- order_delivered_customer_date < order_delivered_carrier_date
SELECT COUNT(*) AS invalid_delivery_sequence
FROM orders
WHERE order_delivered_customer_date IS NOT NULL
  AND order_delivered_carrier_date IS NOT NULL
  AND order_delivered_customer_date < order_delivered_carrier_date;

-- mengecek 166 invalid carrier delivery
SELECT
    order_id,
    order_status,
    order_purchase_timestamp,
    order_approved_at,
    order_delivered_carrier_date,
    order_delivered_customer_date,
    order_estimated_delivery_date
FROM orders
WHERE order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date < order_purchase_timestamp
ORDER BY order_purchase_timestamp;

-- ukur
SELECT
    COUNT(*) AS invalid_records,
    MIN(
        order_delivered_carrier_date - order_purchase_timestamp
    ) AS smallest_difference,
    MAX(
        order_delivered_carrier_date - order_purchase_timestamp
    ) AS largest_difference
FROM orders
WHERE order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date < order_purchase_timestamp;

-- order_id that have largerst diference limit 10
SELECT 
    order_id,
    order_status,
    order_purchase_timestamp,
    order_delivered_carrier_date,
    (order_delivered_carrier_date - order_purchase_timestamp) AS difference_interval
FROM orders
WHERE order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date < order_purchase_timestamp
ORDER BY difference_interval ASC
LIMIT 10;

-- -- order_id that have smallest diference limit 10
SELECT 
    order_id,
    order_status,
    order_purchase_timestamp,
    order_delivered_carrier_date,
    (order_delivered_carrier_date - order_purchase_timestamp) AS difference_interval
FROM orders
WHERE order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date < order_purchase_timestamp
ORDER BY difference_interval DESC
LIMIT 10;

--checking 23 invalid_delivery_sequence
SELECT
    order_id,
    order_status,
    order_purchase_timestamp,
    order_approved_at,
    order_delivered_carrier_date,
    order_delivered_customer_date,
    order_estimated_delivery_date
FROM orders
WHERE order_delivered_customer_date IS NOT NULL
  AND order_delivered_carrier_date IS NOT NULL
  AND order_delivered_customer_date < order_delivered_carrier_date
ORDER BY order_purchase_timestamp;

-- apakah 23 termasuk kedalam 166
SELECT COUNT(*) AS overlap_anomalies
FROM orders
WHERE order_delivered_carrier_date < order_purchase_timestamp
  AND order_delivered_customer_date < order_delivered_carrier_date;
-- timestamps anomalies status  
SELECT
    order_status,
    COUNT(*) AS total_anomalies
FROM orders
WHERE
    (order_delivered_carrier_date IS NOT NULL
        AND order_delivered_carrier_date < order_purchase_timestamp)
    OR
    ( order_delivered_customer_date IS NOT NULL
        AND order_delivered_carrier_date IS NOT NULL
        AND order_delivered_customer_date < order_delivered_carrier_date)
GROUP BY order_status
ORDER BY total_anomalies DESC;
```
### 7. TABEL: order_item
```sql
-- OVERVIEW & ROW COUNT
select * from order_items;
select count(*) as jumlah_row from order_items;

-- NULL CHECK
SELECT
count(*)
FROM order_items
WHERE order_id is null or order_item_id IS NULL or product_id is null or
seller_id is null or shipping_limit_date is null or price is null or freight_value is null;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT order_id) AS order_id_duplikat,
	COUNT(*) - COUNT(DISTINCT order_item_id) AS order_item_id_duplikat,
	COUNT(*) - COUNT(DISTINCT product_id) AS product_id_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_id) AS seller_id_duplikat,
	COUNT(*) - COUNT(DISTINCT shipping_limit_date) AS shipping_limit_date_duplikat,
	COUNT(*) - COUNT(DISTINCT price) AS price_duplikat,
	COUNT(*) - COUNT(DISTINCT freight_value) AS freight_value_duplikat
FROM order_items;

SELECT
order_id,
COUNT(*) 
FROM order_items
GROUP BY order_id
HAVING COUNT(*)>1
order by 2 desc;

-- data validation
SELECT
    COUNT(*) FILTER (WHERE price < 0) AS negative_price,
    COUNT(*) FILTER (WHERE freight_value < 0) AS negative_freight
FROM order_items;
```
### 8. TABEL: order_payments
```sql
-- OVERVIEW & ROW COUNT
select * from order_payments
order by order_id;
select count(*) as jumlah_row from order_payments;
-- NULL CHECK
SELECT
count(*)
FROM order_payments
WHERE order_id is null or payment_sequential IS NULL or payment_type is null or
payment_installments is null or payment_value is null;

-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT order_id) AS order_id_duplikat,
	COUNT(*) - COUNT(DISTINCT payment_sequential) AS payment_sequential_duplikat,
	COUNT(*) - COUNT(DISTINCT payment_type) AS payment_type_duplikat,
	COUNT(*) - COUNT(DISTINCT payment_installments) AS payment_installments_duplikat,
	COUNT(*) - COUNT(DISTINCT payment_value) AS payment_value_duplikat
FROM order_payments;

SELECT
order_id,
COUNT(*) 
FROM order_payments
GROUP BY order_id
HAVING COUNT(*)>1
order by 2 desc;

-- DATA VALIDITY CHECK
SELECT
    COUNT(*) FILTER (WHERE payment_value < 0) AS negative_payment,
    COUNT(*) FILTER (WHERE payment_installments < 1) AS invalid_installments
FROM order_payments;
--		|
--		v
SELECT
    order_id,
    payment_sequential,
    payment_type,
    payment_installments,
    payment_value
FROM order_payments
WHERE payment_installments < 1;
--		|
--		v
SELECT
    payment_type,
    payment_installments,
    COUNT(*) AS total_records
FROM order_payments
WHERE payment_installments < 1
GROUP BY payment_type, payment_installments
ORDER BY total_records DESC;
```
### 9. TABEL: order_review
```sql
-- OVERVIEW & ROW COUNT
select * from order_reviews;
select count(*) as jumlah_row from order_reviews;

-- NULL CHECK
SELECT
count(*)
FROM order_reviews
WHERE review_id is null or order_id IS NULL or review_score is null or
review_comment_title is null or review_comment_message is null or review_creation_date
is null or review_answer_timestamp is null ;

SELECT
    COUNT(*) AS total_rows,

    COUNT(*) - COUNT(review_id) AS review_id_null,
    COUNT(*) - COUNT(order_id) AS order_id_null,
    COUNT(*) - COUNT(review_score) AS review_score_null,
    COUNT(*) - COUNT(review_comment_title) AS review_comment_title_null,
    COUNT(*) - COUNT(review_comment_message) AS review_comment_message_null,
	COUNT(*) - COUNT(review_creation_date) AS review_creation_date_null,
	COUNT(*) - COUNT(review_answer_timestamp) AS review_answer_timestamp_null
from order_reviews;
-- DUPLICATE CHECK
SELECT
	COUNT(*) - COUNT(DISTINCT review_id) AS review_id_duplikat,
    COUNT(*) - COUNT(DISTINCT order_id) AS order_id_duplikat,
    COUNT(*) - COUNT(DISTINCT review_score) AS review_score_duplikat,
    COUNT(*) - COUNT(DISTINCT review_comment_title) AS review_comment_title_duplikat,
    COUNT(*) - COUNT(DISTINCT review_comment_message) AS review_comment_message_duplikat,
	COUNT(*) - COUNT(DISTINCT review_creation_date) AS review_creation_date_duplikat,
	COUNT(*) - COUNT(DISTINCT review_answer_timestamp) AS review_answer_timestamp_duplikat
FROM order_reviews;

-- DATA VALIDITY CHECK 
SELECT
    COUNT(*) FILTER (WHERE review_score < 1) AS score_below_1,
    COUNT(*) FILTER (WHERE review_score > 5) AS score_above_5
FROM order_reviews;
```

## QDA FINDING
### product_categories
pada product_categories terdapat produk kategori yang belum lengkap yaitu:
`pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos`
### customers
duplikasi pada kolom `customer_unique_id` adalah hal yang wajar, bahkan mengindikasikan repeat order oleh customer.
### products
terdapat null sebanyak 610 `product_category_name`begitu pula pada `product_name_lenght`,`product_description_lenght`,`product_photos_qty`.
### orders
* **terdapat null pada**
<img width="365" height="52" alt="image" src="https://github.com/user-attachments/assets/cbd83df1-9c3d-493f-a70d-4aef44ee1b63" />

* **cek jumlah null Per-Status**
<img width="995" height="367" alt="image" src="https://github.com/user-attachments/assets/48ea3ece-aedf-4325-aefb-2aa6b03c523b" />

* keberadaan null pada status seperti `shipped`,`cancelled`,`unvailable` dinilai cukup wajar, misal pada `shipped` null pada `order_delivered_customer_date` wajar karena barang memang sedang dalam proses pengantaran sehingga belum sampai di tangan customer, maka dari itu `order_delivered_customer_date` tidak memiliki value/null. begitu juga dengan status lainnya
* namun pada status `delivered`dimana seharusnya tidak memiliki null/semua kolom terisi, terdapat null mulai dari `approved at` hingga `customer_date`.

* **order_delivered_carrier_date < order_purchase_timestamp(invalid_carrier_delivery)**
<img width="1308" height="282" alt="image" src="https://github.com/user-attachments/assets/b731b88c-43a4-4d64-b5cc-4c8d92906a64" />
<img width="241" height="101" alt="image" src="https://github.com/user-attachments/assets/d69fa010-42b5-4eac-8395-3b898c9b8fa6" />

* **order_delivered_customer_date < order_delivered_carrier_date(invalid_delivery_sequence)**
<img width="1308" height="281" alt="image" src="https://github.com/user-attachments/assets/61842c48-65b4-40a1-a0c7-95f2a1fa26b7" />
<img width="260" height="103" alt="image" src="https://github.com/user-attachments/assets/2cb3e1b7-2cad-4324-ba13-f9f245e6584b" />

* 23 anomali tanggal tidak termasuk kedalam 166 atau terpisah, sehingga totalnya adalah 189
<img width="402" height="141" alt="image" src="https://github.com/user-attachments/assets/a97a3785-e959-4da1-afbb-1ba366a0b298" />

### ORDER_PAYMENTS
* terdapat 2 `order_id` dengan payment_instalments < 1

<img width="235" height="50" alt="image" src="https://github.com/user-attachments/assets/16d68237-2080-435d-94d4-4cec8c52f157" />

<img width="1159" height="139" alt="image" src="https://github.com/user-attachments/assets/f01cbc1d-ab1f-48ae-bcdf-ece7f72565dc" />


