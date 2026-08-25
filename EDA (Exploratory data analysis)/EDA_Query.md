# Exploratory Data Analysis (EDA) & Business Analysis
Pada tahap ini, analisis dilakukan menggunakan kueri SQL tingkat lanjut (`JOINs`, `CTEs`, `Window Functions`, dan `Aggregations`) untuk mengekstrak *insight* dari database `olist_project`.

## EDA 1 OVERALL BUSINESS PERFORMANCE
```sql
-- 1. total orders (Seberapa besar volume bisnis Olist dalam dataset?)
SELECT
    COUNT(DISTINCT order_id) AS total_orders
FROM orders;
-- 2. total customer
SELECT
    COUNT(DISTINCT customer_unique_id) AS total_customers
FROM customers;
-- 3. Total sellers
SELECT
    COUNT(DISTINCT seller_id) AS total_sellers
FROM sellers;
-- 4. total product items sold
SELECT
    COUNT(*) AS total_items_sold
FROM order_items;
-- 5. total Total product sales
SELECT
    SUM(price) AS total_product_sales
FROM order_items;
-- 6. Total freight
SELECT
    SUM(freight_value) AS total_freight
FROM order_items;
```

