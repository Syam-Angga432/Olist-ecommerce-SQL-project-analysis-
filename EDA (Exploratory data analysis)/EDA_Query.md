# Exploratory Data Analysis (EDA) & Business Analysis
Pada tahap ini, analisis dilakukan menggunakan kueri SQL tingkat lanjut (`JOINs`, `CTEs`, `Window Functions`, dan `Aggregations`) untuk mengekstrak *insight* dari database `olist_project`.

## EDA 1 OVERALL BUSINESS PERFORMANCE
### Key Performance 
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
### Base KPI
``` sql
SELECT
    -- (Order & Customer)
    (SELECT COUNT(*)
     FROM orders_clean) AS total_orders,

    (SELECT COUNT(DISTINCT c.customer_unique_id)
     FROM orders_clean o
     JOIN customers c
       ON o.customer_id = c.customer_id) AS total_customers,

    -- (Order Items)
    (SELECT COUNT(DISTINCT oi.order_id)
     FROM order_items oi) AS orders_with_items,

    (SELECT COUNT(*)
     FROM order_items oi) AS total_items_sold,

    (SELECT COUNT(DISTINCT oi.product_id)
     FROM order_items oi) AS unique_products_sold,

    (SELECT COUNT(DISTINCT oi.seller_id)
     FROM order_items oi) AS active_sellers,

    -- (Sales)
    (SELECT ROUND(SUM(oi.price), 2)
     FROM order_items oi) AS total_product_sales,

    (SELECT ROUND(SUM(oi.freight_value), 2)
     FROM order_items oi) AS total_freight,

    (SELECT ROUND(SUM(oi.price + oi.freight_value), 2)
     FROM order_items oi) AS total_sales_including_freight,

    -- (Average Order Value)
    (SELECT ROUND(
        SUM(oi.price + oi.freight_value)
        / NULLIF(COUNT(DISTINCT oi.order_id), 0), 2)
     FROM order_items oi) AS average_order_value;
```
### Total Transaction value, total order, percentage by status
```sql
SELECT
    o.order_status,
    COUNT(DISTINCT o.order_id) AS total_orders,
	ROUND(COUNT(*) * 100.0/ SUM(COUNT(*)) OVER (),2) AS percentage,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM(oi.freight_value), 2) AS freight_value,
    ROUND(SUM(oi.price + oi.freight_value), 2) AS sales_including_freight
FROM orders_clean o
JOIN order_items oi
    ON o.order_id = oi.order_id
GROUP BY o.order_status
ORDER BY sales_including_freight DESC;
```
### ?
```sql
SELECT
    COUNT(DISTINCT o.order_id) AS completed_orders,
    COUNT(*) AS completed_items,
    ROUND(SUM(oi.price), 2) AS completed_product_sales,
    ROUND(SUM(oi.freight_value), 2) AS completed_freight,
    ROUND(
        SUM(oi.price + oi.freight_value),2) AS completed_sales_including_freight,
    ROUND(SUM(oi.price + oi.freight_value)
        / NULLIF(COUNT(DISTINCT o.order_id), 0),2
    ) AS completed_aov

FROM orders_clean o
JOIN order_items oi
    ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered';
```
