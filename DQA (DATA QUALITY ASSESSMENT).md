# DQA (DATA QUALITY ASSESSMENT)
* DATA QUALITY ASSESSMENT (DQA) - OLIST E-COMMERCE DATASET
* Database : olist_project
* Tool     : PostgreSQL
  
  Tahapan DQA:
  1. Data Overview (Cek sampel & jumlah baris)
  2. Missing Values Check (Cek kolom ber-NULL)
  3. Duplication Check (Cek duplikasi pada Primary Key)
  4. Data Validity Check (Cek logika & keabsahan nilai)
  
## STRUKTUR QUERY UNTUK TAHAP DQA

**1. Overview**
```
SELECT * FROM <table_name> LIMIT 10;
SELECT COUNT(*) AS total_rows FROM <table_name>;
```
**2. Missing Values (PostgreSQL Trick)**

**version 1**
```
SELECT
COUNT(*) AS total_NULL
FROM <table_name>
WHERE <column_1> is null or  <column_2> IS NULL or <column_3> is null or <column_4> is null;
```
** OR** 
```
SELECT *
FROM <table_name>
WHERE <column_1> is null or  <column_2> IS NULL or <column_3> is null or <column_4>
```
**version 2**
```
SELECT 
    COUNT(*) AS total_rows,
    COUNT(*) - COUNT(<column_1>) AS <column_1>_null_count,
    COUNT(*) - COUNT(<column_2>) AS <column_2>_null_count
FROM <table_name>;
```
**version 3**
```
SELECT 
    COUNT(*) AS total_rows,
    COUNT(order_approved_at) AS filled_rows,
    SUM(CASE WHEN order_approved_at IS NULL THEN 1 ELSE 0 END) AS null_count,
    round((avg(CASE WHEN order_approved_at IS NULL THEN 1 ELSE 0 END))*100,2) AS null_percentage
FROM orders;
```
**3. Duplication Check**

**version 1**
```
SELECT 
    COUNT(*) - COUNT(DISTINCT <column_1>) AS <column_1>_duplicat_count,
    COUNT(*) - COUNT(DISTINCT <column_2>) AS <column_2>_duplicat_count
FROM <table_name>;
```
**version 2**
```
SELECT
    <column_1>,
    COUNT(*) AS duplicate_count
FROM <table_name>
GROUP BY 1
HAVING COUNT(*) > 1
ORDER BY duplicate_count DESC;
```
**4. Data Validity Check**
```
SELECT COUNT(*) FILTER (WHERE <column_numeric> < 0) AS invalid_negative_values
FROM <table_name>;
```

## IMPLEMENTASI PADA TABEL OLIST
### 1. TABEL: product_categories
```
-- Overview & Row Count
SELECT * FROM product_categories;
SELECT COUNT(*) AS total_rows FROM orders;

-- Null Check
SELECT
COUNT(*) AS NULL
FROM  product_categories
WHERE  product_category_name is null or product_category_name_english IS NULL;

-- Duplicate Check
SELECT
	COUNT(*) - COUNT(DISTINCT product_category_name) AS category_name_duplikat,
	COUNT(*) - COUNT(DISTINCT product_category_name_english) AS name_english_duplikat
FROM product_categories;

-- Data Validity Check (kelengkapan produk kategory)
select p.product_category_name
from products p 
left join product_categories pc 
on pc.product_category_name = p.product_category_name
where pc.product_category_name is null and 
		p.product_category_name is not null
order by 1;
```
### 2. TABEL: customers
```
-- Overview & Row Count
SELECT * FROM customers;
select count (*) as row_amount from customers;

-- Null Check
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

-- Duplicate Check
SELECT
	COUNT(*) - COUNT(DISTINCT customer_id) AS id_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_unique_id) AS unique_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_zip_code_prefix) AS zip_code_duplikat
FROM customers;

SELECT
customer_unique_id,
COUNT(*) 
FROM customers
GROUP BY customer_unique_id
HAVING COUNT(*)>1
order by 2 desc;
```
### 3. TABEL: geolocation
```
-- Overview & Row Count
select * from geolocation;
select count (*) as row_amount from geolocation;

-- Null Check
SELECT
COUNT(*) AS NULL
FROM geolocation
WHERE geolocation_zip_code_prefix is null or geolocation_lat IS NULL or geolocation_lng is null or
geolocation_city is null or geolocation_state is null;
```

### 4. TABEL: sellers
```
-- Overview & Row Count
select * from sellers;
select count (*) as row_amount from sellers;

-- Null Check
SELECT
COUNT(*) AS NULL
FROM sellers
WHERE seller_id is null or seller_zip_code_prefix IS NULL or seller_city is null 
or seller_state is null;

-- Duplicate Check
SELECT
	COUNT(*) - COUNT(DISTINCT seller_id) AS id_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_zip_code_prefix) AS zip_code_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_city) AS city_duplikat,
	COUNT(*) - COUNT(DISTINCT seller_state) AS state_duplikat
FROM sellers;
```
### 5. TABEL: products
```
-- Overview & Row Count
select * from products;
select count (*) as row_amount from products;

-- Null Check (terdapat 610 product_category_name null)
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
WHERE product_id is null or product_category_name IS NULL or product_name_lenght is null or product_description_lenght is null or product_photos_qty is null or product_weight_g is null or product_length_cm is null or product_height_cm is null or product_width_cm is null;

-- Duplicate Check
SELECT
	COUNT(*) - COUNT(DISTINCT product_id) AS id_duplikat
FROM products;

-- data validity
SELECT
    COUNT(*) FILTER (WHERE product_weight_g < 0) AS negative_weight,
    COUNT(*) FILTER (WHERE product_length_cm < 0) AS negative_length,
    COUNT(*) FILTER (WHERE product_height_cm < 0) AS negative_height,
    COUNT(*) FILTER (WHERE product_width_cm < 0) AS negative_width,
    COUNT(*) FILTER (WHERE product_photos_qty < 0) AS negative_photos
FROM products;
```
### 6. TABEL: orders
```
-- Overview & Row Count
select * from orders;
select count(*) as jumlah_row from orders;

-- Null Check
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

-- Duplicate Check
SELECT
	COUNT(*) - COUNT(DISTINCT order_id) AS order_id_duplikat,
	COUNT(*) - COUNT(DISTINCT customer_id) AS customer_id_duplikat
FROM orders;

-- data validity
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
-- are the 23 include in 166
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
```
-- Overview & Row Count
-- Null Check
-- Duplicate Check
```
### 8. TABEL: order_payments
```
-- Overview & Row Count
-- Null Check
-- Duplicate Check
```
### 9. TABEL: order_review
```
-- Overview & Row Count
-- Null Check
-- Duplicate Check
```

