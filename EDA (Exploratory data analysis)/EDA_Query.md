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
### Base Metrics Completed orders / Delivered
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

## EDA 2 SALES PERFORMANCE
### SALES TREND (MoM)
```sql
SELECT
    DATE_TRUNC('month', o.order_purchase_timestamp) AS month,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COUNT(*) AS total_items,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM(oi.freight_value), 2) AS freight,
    ROUND(SUM(oi.price + oi.freight_value), 2)AS sales_including_freight
FROM orders_clean o
JOIN order_items oi
    ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered'
GROUP BY 1
ORDER BY 1;
```
### SALES GROWTH PERCENTAGE
```sql
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', o.order_purchase_timestamp) AS month,
        SUM(oi.price + oi.freight_value) AS sales
    FROM orders_clean o
    JOIN order_items oi
        ON o.order_id = oi.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY 1)
SELECT
    month,
    ROUND(sales, 2) AS sales,
    ROUND( LAG(sales) OVER (ORDER BY month),2) AS previous_month_sales,
    ROUND((sales- LAG(sales) OVER (ORDER BY month))
        / NULLIF(LAG(sales) OVER (ORDER BY month),0 ) * 100,2) AS mom_growth_percentage
FROM monthly_sales
ORDER BY month;
```
### TOTAL TRANSACTION VALUE / SALES BY STATE
```sql
SELECT 
    c.customer_state,
    COUNT(DISTINCT o.order_id) AS total_orders,
    ROUND(SUM(oi.price + oi.freight_value)::NUMERIC, 2) AS total_transaction_value,
    ROUND(AVG(oi.freight_value)::NUMERIC, 2) AS avg_freight_cost
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered'
GROUP BY 1
ORDER BY total_transaction_value DESC;
```
### TOTAL TRANSACTION VALUE / SALES BY CATEGORY
```sql
WITH category_summary AS (
    SELECT 
        COALESCE(pc.product_category_name_english, 'Uncategorized') AS category,
        COUNT(DISTINCT o.order_id) AS total_orders,
        COUNT(oi.order_item_id) AS total_items,
        SUM(oi.price) AS product_sales,
        SUM(oi.freight_value) AS freight_value,
        SUM(oi.price + oi.freight_value) AS total_sales
    FROM products_clean p
    JOIN order_items oi ON p.product_id = oi.product_id
    left JOIN orders_clean o ON oi.order_id = o.order_id
	left JOIN product_categories_v2 pc ON pc.product_category_name = p.product_category_name
    WHERE o.order_status = 'delivered'
    GROUP BY 1)
SELECT 
    category,
    total_orders,
    total_items,
    ROUND(product_sales, 2) AS product_sales,
    ROUND(freight_value, 2) AS freight_value,
    ROUND(total_sales, 2) AS total_sales,
    ROUND((total_sales / SUM(total_sales) OVER ()) * 100, 
        2) AS sales_contribution_pct
FROM category_summary
ORDER BY total_sales DESC;
```
### TOTAL TRANSACTION VALUE / SALES BY PAYMENT METHOD
```sql
WITH payment_summary AS (
    SELECT 
        op.payment_type,
        COUNT(DISTINCT op.order_id) AS total_orders,
        COUNT(*) AS total_transactions,
        SUM(op.payment_value) AS total_revenue
    FROM order_payments_clean op
    JOIN orders_clean o ON op.order_id = o.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY op.payment_type)
SELECT 
    payment_type,
    total_orders,
    total_transactions,
    ROUND(total_revenue, 2) AS total_revenue,
    ROUND((total_revenue / SUM(total_revenue) OVER ()) * 100,2) AS revenue_contribution_pct
FROM payment_summary
ORDER BY total_revenue DESC;
```
## EDA 3 PRODUCT PERFORMANCE
### TOP 10 PRODUCT BY SALES VOLUME
```sql
SELECT 
    p.product_id,
    COALESCE(pc.product_category_name_english, 'Uncategorized') AS category_name,
    COUNT(oi.order_item_id) AS total_items_sold, -- Volume unit terjual
    COUNT(DISTINCT oi.order_id) AS total_unique_orders, -- Jumlah transaksi unik
    ROUND(SUM(oi.price)::NUMERIC, 2) AS total_product_sales
FROM order_items oi
JOIN products p ON oi.product_id = p.product_id
LEFT JOIN product_categories pc ON p.product_category_name = pc.product_category_name
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_status = 'delivered'
GROUP BY 1, 2
ORDER BY total_items_sold DESC
LIMIT 10;
```
### TOP 10 PRODUCT BY SALES VALUE
```sql
select * from order_items;
SELECT
    oi.product_id,
    COALESCE(pc.product_category_name_english, 'uncategorized') AS category,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(*) AS total_items,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM(oi.freight_value), 2) AS freight_value,
    ROUND(SUM(oi.price + oi.freight_value), 2) AS total_sales
FROM order_items oi
JOIN products_clean p
    ON oi.product_id = p.product_id
LEFT JOIN product_categories_v2 pc
    ON p.product_category_name = pc.product_category_name
LEFT JOIN orders_clean o
    ON o.order_id = oi.order_id
	WHERE o.order_status = 'delivered'
GROUP BY
    oi.product_id,
    COALESCE(pc.product_category_name_english, 'uncategorized')
ORDER BY 5 desc
LIMIT 20;
```
### top volume product in marketplace by category
```sql
SELECT 
    COALESCE(pc.product_category_name_english, 'Uncategorized') AS category_name,
    COUNT(DISTINCT p.product_id) AS total_unique_products,
    ROUND(
        (COUNT(DISTINCT p.product_id)::NUMERIC / SUM(COUNT(DISTINCT p.product_id)) OVER ()) * 100, 2
    ) AS product_share_pct
FROM products p
LEFT JOIN product_categories pc ON p.product_category_name = pc.product_category_name
GROUP BY 1
ORDER BY total_unique_products DESC
LIMIT 10;
```
## EDA 4 CUSTOMER PERFORMANCE

