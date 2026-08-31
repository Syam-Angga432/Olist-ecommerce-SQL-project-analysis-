# Exploratory Data Analysis (EDA) & Business Analysis
Pada tahap ini, analisis dilakukan menggunakan kueri SQL tingkat lanjut (`JOINs`, `CTEs`, `Window Functions`, dan `Aggregations`) untuk mengekstrak *insight* dari database `olist_project`.

## EDA 1 OVERALL BUSINESS PERFORMANCE  
### Key Performance (1.1)
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
### Base KPI (1.2)
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
### Base Metrics Completed orders / Delivered (1.3)
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
### SALES TREND (MoM) (2.1)
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
### SALES GROWTH PERCENTAGE (2.2)
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
### TOTAL TRANSACTION VALUE / SALES BY STATE (2.3)
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
### TOTAL TRANSACTION VALUE / SALES BY PAYMENT METHOD (2.4)
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
### Top Product Category by value Sales
```sql
SELECT
    COALESCE(pc.product_category_name_english, 'Unknown') AS category,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(*) AS total_items,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM(oi.freight_value), 2) AS freight_value,
    ROUND(SUM(oi.price + oi.freight_value), 2) AS total_sales
FROM order_items oi
JOIN products_clean p
    ON oi.product_id = p.product_id
LEFT JOIN product_categories pc
    ON p.product_category_name = pc.product_category_name
GROUP BY COALESCE(pc.product_category_name_english, 'Unknown')
ORDER BY 6 DESC;
```
### Top Product Category by volume sales
```sql
SELECT
    COALESCE(pc.product_category_name_english, 'Unknown') AS category,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(*) AS total_items,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM(oi.freight_value), 2) AS freight_value,
    ROUND(SUM(oi.price + oi.freight_value), 2) AS total_sales
FROM order_items oi
JOIN products_clean p
    ON oi.product_id = p.product_id
LEFT JOIN product_categories pc
    ON p.product_category_name = pc.product_category_name
GROUP BY COALESCE(pc.product_category_name_english, 'Unknown')
ORDER BY 3 DESC;
```
### TOP 10 PRODUCT BY SALES VOLUME (3.1)
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
### TOP 10 PRODUCT BY SALES VALUE (3.2)
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
### top volume product in marketplace by category (3.3)
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
### customer purchase frequency(4.1)
```sql
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        COUNT(DISTINCT o.order_id) AS total_orders
    FROM orders o
    JOIN customers c
        ON o.customer_id = c.customer_id
    GROUP BY c.customer_unique_id)
SELECT
    total_orders,
    COUNT(*) AS total_customers,
    ROUND(
        COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (),3) AS customer_percentage
FROM customer_orders
GROUP BY total_orders
ORDER BY total_orders;
```
### repeat customer rate (4.2)
```sql
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        COUNT(DISTINCT o.order_id) AS total_orders
    FROM orders o
    JOIN customers c
        ON o.customer_id = c.customer_id
    GROUP BY c.customer_unique_id)
SELECT
    COUNT(*) AS total_customers,
    COUNT(*) FILTER ( WHERE total_orders = 1) AS one_time_customers,
    COUNT(*) FILTER (WHERE total_orders > 1) AS repeat_customers,
    ROUND( COUNT(*) FILTER (WHERE total_orders > 1) * 100.0 / COUNT(*),2) AS repeat_customer_rate,
	ROUND(COUNT(*) FILTER (WHERE total_orders = 1) * 100.0 / COUNT(*),2) AS one_time_customer_rate
FROM customer_orders;
```
### customer revenue/customer VALUE (4.3)
```sql
WITH customer_sales AS (
    SELECT
    c.customer_unique_id,
    COUNT(DISTINCT o.order_id) AS total_orders,
    COALESCE(SUM(oi.price + oi.freight_value), 0) AS total_sales
FROM orders_clean o
JOIN customers c
    ON o.customer_id = c.customer_id
LEFT JOIN order_items oi
    ON o.order_id = oi.order_id
GROUP BY c.customer_unique_id)
SELECT
    CASE
        WHEN total_orders = 1 THEN 'One-time'
        ELSE 'Repeat'
    END AS customer_type,
    COUNT(*) AS total_customers,
    ROUND(SUM(total_sales), 2) AS total_sales,
    ROUND(
        SUM(total_sales) * 100.0
        / SUM(SUM(total_sales)) OVER (),2) AS sales_contribution_pct,
    ROUND(AVG(total_sales),2) AS average_customer_value
FROM customer_sales
GROUP BY
    CASE
        WHEN total_orders = 1 THEN 'One-time'
        ELSE 'Repeat'
    END
ORDER BY total_sales DESC;
```
### customer repeat vs order status (4.4)
```sql
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        o.order_id,
        o.order_status
    FROM orders_clean o
    JOIN customers c
        ON o.customer_id = c.customer_id),
customer_type AS (
    SELECT
        customer_unique_id,
        COUNT(DISTINCT order_id) AS total_orders
    FROM customer_orders
    GROUP BY customer_unique_id)
SELECT
    CASE
        WHEN ct.total_orders = 1 THEN 'One-time'
        ELSE 'Repeat'
    END AS customer_type,
    co.order_status,
    COUNT(DISTINCT co.order_id) AS total_orders,
    ROUND( COUNT(DISTINCT co.order_id) * 100.0
        / SUM(COUNT(DISTINCT co.order_id))
          OVER (PARTITION BY
              CASE
                  WHEN ct.total_orders = 1 THEN 'One-time' ELSE 'Repeat'
              END),2) AS status_percentage
FROM customer_orders co
JOIN customer_type ct
    ON co.customer_unique_id = ct.customer_unique_id
GROUP BY
    CASE
        WHEN ct.total_orders = 1 THEN 'One-time' ELSE 'Repeat'
    END,
    co.order_status
ORDER BY
    customer_type,
    total_orders DESC;
```
### Same Category vs Cross Category Repeat (4.5)
```sql
WITH customer_orders AS (
    SELECT
        c.customer_unique_id,
        o.order_id,
        o.order_purchase_timestamp,
        ARRAY_AGG(DISTINCT p.product_category_name)
            FILTER (WHERE p.product_category_name IS NOT NULL) AS categories
    FROM orders_clean o
    JOIN customers c
        ON o.customer_id = c.customer_id
    JOIN order_items oi
        ON o.order_id = oi.order_id
    JOIN products_clean p
        ON oi.product_id = p.product_id
    WHERE o.order_status = 'delivered'
    GROUP BY
        c.customer_unique_id,
        o.order_id,
        o.order_purchase_timestamp),
  ordered_customers AS (
    SELECT
        *,
        ROW_NUMBER() OVER ( PARTITION BY customer_unique_id ORDER BY order_purchase_timestamp) AS order_number
    FROM customer_orders),
repeat_customers AS (
    SELECT
        customer_unique_id
    FROM ordered_customers
    GROUP BY customer_unique_id
    HAVING COUNT(*) > 1),
first_orders AS (
    SELECT
        customer_unique_id,
        categories AS first_categories
    FROM ordered_customers
    WHERE order_number = 1),
later_orders AS (
    SELECT
        o.customer_unique_id,
        o.categories
    FROM ordered_customers o
    JOIN repeat_customers r
        ON o.customer_unique_id = r.customer_unique_id
    WHERE o.order_number > 1),
customer_repeat_behavior AS (
    SELECT
        f.customer_unique_id,
        CASE
            WHEN EXISTS (SELECT 1 
						 FROM later_orders l 
						 WHERE l.customer_unique_id = f.customer_unique_id AND l.categories && f.first_categories)
            THEN 'Same Category'
            ELSE 'Cross Category'
        END AS repeat_behavior
    FROM first_orders f
    JOIN repeat_customers r
        ON f.customer_unique_id = r.customer_unique_id)
SELECT
    repeat_behavior,
    COUNT(*) AS repeat_customers,
    ROUND( COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (),2 ) AS percentage
FROM customer_repeat_behavior
GROUP BY repeat_behavior
ORDER BY repeat_customers DESC;
```
### Customers Distribution
```sql
WITH customer_geography AS (
    SELECT 
        c.customer_state,
        COUNT(DISTINCT c.customer_unique_id) AS total_customers,
        COUNT(DISTINCT o.order_id) AS total_orders,
        SUM(oi.price + oi.freight_value) AS total_sales
    FROM customers c
    JOIN orders_clean o 
        ON c.customer_id = o.customer_id
    JOIN order_items oi 
        ON o.order_id = oi.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY c.customer_state
)
SELECT 
    customer_state,
    total_customers,
    total_orders,
    ROUND(total_sales, 2) AS total_sales,
    
    -- % Konsentrasi Basis Pelanggan
    ROUND(
        (total_customers * 100.0 / SUM(total_customers) OVER ()), 
        2
    ) AS customer_share_pct,
    -- % Kontribusi Penjualan (Revenue)
    ROUND(
        (total_sales * 100.0 / SUM(total_sales) OVER ()), 
        2
    ) AS revenue_share_pct
FROM customer_geography
ORDER BY total_customers DESC;
```
## EDA 5 SELLERS PERFORMANCE 
### 20 Sellers by sales (5.1)
```sql
SELECT
    s.seller_id,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    COUNT(*) AS total_items,
    ROUND(SUM(oi.price), 2) AS product_sales,
    ROUND(SUM (oi.price)
        / COUNT(DISTINCT oi.order_id),2) AS average_order_sales
FROM sellers s
JOIN order_items oi
    ON s.seller_id = oi.seller_id
JOIN orders_clean o
    ON oi.order_id = o.order_id
GROUP BY s.seller_id
ORDER BY product_sales DESC
LIMIT 20;
```
### sellers detail (5.2)
```sql
SELECT 
    s.seller_id,
    s.seller_state,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    ROUND(SUM(oi.price), 2) AS total_revenue,
    ROUND(AVG(orw.review_score), 2) AS avg_review_score,
    
    -- Distribusi Rating
    COUNT(CASE WHEN orw.review_score = 5 THEN 1 END) AS count_5_star,
    COUNT(CASE WHEN orw.review_score <= 2 THEN 1 END) AS count_low_star
FROM sellers s
JOIN order_items oi ON s.seller_id = oi.seller_id
JOIN orders_clean o ON oi.order_id = o.order_id
JOIN order_reviews orw ON o.order_id = orw.order_id
WHERE o.order_status = 'delivered'
GROUP BY s.seller_id, s.seller_state
HAVING COUNT(DISTINCT oi.order_id) >= 20
ORDER BY total_revenue DESC;
```
### segmentation sellers (5.3)
```sql
WITH seller_items AS (
    SELECT 
        s.seller_id,
        COUNT(oi.order_item_id) AS total_items_sold,
        COALESCE(SUM(oi.price), 0) AS total_sales
    FROM sellers s
    LEFT JOIN order_items oi ON s.seller_id = oi.seller_id
    LEFT JOIN orders_clean o ON oi.order_id = o.order_id AND o.order_status = 'delivered'
    GROUP BY s.seller_id),
segmented_sellers AS (
    SELECT 
        seller_id,
        total_items_sold,
        total_sales,
        CASE 
            WHEN total_items_sold >= 100 THEN '1. High Volume (>= 100 items)'
            WHEN total_items_sold BETWEEN 20 AND 99 THEN '2. Medium Volume (20-99 items)'
            WHEN total_items_sold BETWEEN 1 AND 19 THEN '3. Low Volume (1-19 items)'
            ELSE '4. Inactive (0 items sold)'
        END AS seller_segment
    FROM seller_items)
SELECT 
    seller_segment,
    COUNT(seller_id) AS total_sellers,
    ROUND(COUNT(seller_id) * 100.0 / SUM(COUNT(seller_id)) OVER (), 2) AS seller_share_pct,
    ROUND(SUM(total_sales), 2) AS total_segment_sales
FROM segmented_sellers
GROUP BY seller_segment
ORDER BY seller_segment ASC;
```
### sellers performance by cancelation rate (5.4)
```sql
WITH seller_orders AS (
    SELECT 
        oi.seller_id,
        COUNT(DISTINCT o.order_id) AS total_orders_handled,
        COUNT(DISTINCT CASE WHEN o.order_status = 'canceled' THEN o.order_id END) AS canceled_orders
    FROM order_items oi
    JOIN orders_clean o ON oi.order_id = o.order_id
    GROUP BY oi.seller_id)
SELECT 
    seller_id,
    total_orders_handled,
    canceled_orders,
    ROUND((canceled_orders * 100.0 / total_orders_handled), 2) AS cancellation_rate_pct
FROM seller_orders
WHERE total_orders_handled >= 10
ORDER BY cancellation_rate_pct DESC
limit 20;
```
### 10 sellers by rating stars (5.5)
```sql
SELECT 
    s.seller_id,
    s.seller_state,
    COUNT(DISTINCT oi.order_id) AS total_orders,
    ROUND(SUM(oi.price), 2) AS total_revenue,
    ROUND(AVG(orw.review_score), 2) AS avg_review_score,
    
    -- Distribusi Rating
    COUNT(CASE WHEN orw.review_score = 5 THEN 1 END) AS count_5_star,
    COUNT(CASE WHEN orw.review_score <= 2 THEN 1 END) AS count_low_star
FROM sellers s
JOIN order_items oi ON s.seller_id = oi.seller_id
JOIN orders_clean o ON oi.order_id = o.order_id
JOIN order_reviews orw ON o.order_id = orw.order_id
WHERE o.order_status = 'delivered'
GROUP BY s.seller_id, s.seller_state
HAVING COUNT(DISTINCT oi.order_id) >= 10
ORDER BY total_revenue DESC;
```
### konsentrasi sellers per wilayah
```sql
WITH seller_state_sales AS (
    SELECT 
        s.seller_state,
        COUNT(DISTINCT s.seller_id) AS total_active_sellers,
        COUNT(DISTINCT oi.order_id) AS total_orders,
        SUM(oi.price) AS product_sales,
        SUM(oi.freight_value) AS freight_value,
        SUM(oi.price + oi.freight_value) AS total_sales
    FROM sellers s
    JOIN order_items oi 
        ON s.seller_id = oi.seller_id
    JOIN orders_clean o 
        ON oi.order_id = o.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY s.seller_state
)
SELECT 
    seller_state,
    total_active_sellers,
    total_orders,
    ROUND(product_sales, 2) AS product_sales,
    ROUND(freight_value, 2) AS freight_value,
    ROUND(total_sales, 2) AS total_sales,
    ROUND(
        (total_sales * 100.0 / SUM(total_sales) OVER ()), 
        2
    ) AS sales_contribution_pct
FROM seller_state_sales
ORDER BY total_sales DESC;
```
## EDA 6 DELIVERY PERFORMANCE 
### 1. Tingkat Keterlambatan Pengiriman (On-Time vs Late Delivery Rate) (6.1)
```sql
SELECT 
    CASE 
        WHEN DATE(order_delivered_customer_date) <= DATE(order_estimated_delivery_date) THEN 'On-Time / Early'
        ELSE 'Late'
    END AS delivery_status,
    COUNT(order_id) AS total_orders,
    ROUND(
        COUNT(order_id) * 100.0 / SUM(COUNT(order_id)) OVER (), 2
    ) AS percentage
FROM orders_clean
WHERE order_status = 'delivered'
  AND order_delivered_customer_date IS NOT NULL
  AND order_estimated_delivery_date IS NOT NULL
GROUP BY 1;
```
### Efisiensi Pemrosesan Penjual (6.2)
```sql
SELECT 
    ROUND(
        AVG(EXTRACT(EPOCH FROM (order_delivered_carrier_date - order_approved_at)) / 86400)::numeric, 2
    ) AS avg_seller_processing_days,
    ROUND(
        AVG(EXTRACT(EPOCH FROM (order_delivered_carrier_date - order_approved_at)) / 3600)::numeric, 2
    ) AS avg_seller_processing_hours
FROM orders_clean
WHERE order_status = 'delivered'
  AND order_approved_at IS NOT NULL
  AND order_delivered_carrier_date IS NOT NULL
  AND order_delivered_carrier_date >= order_approved_at;
```
### Durasi Transit Kurir (6.3) 
```sql
SELECT 
    ROUND(
        AVG(EXTRACT(EPOCH FROM (order_delivered_customer_date - order_delivered_carrier_date)) / 86400)::numeric, 2
    ) AS avg_carrier_transit_days,
    
    -- Total Waktu Pengiriman Keseluruhan (Purchase to Customer Delivery)
    ROUND(
        AVG(EXTRACT(EPOCH FROM (order_delivered_customer_date - order_purchase_timestamp)) / 86400)::numeric, 2
    ) AS avg_total_delivery_days
FROM orders_clean
WHERE order_status = 'delivered'
  AND order_delivered_carrier_date IS NOT NULL
  AND order_delivered_customer_date IS NOT NULL
  AND order_delivered_customer_date >= order_delivered_carrier_date;
```
### Dampak Keterlambatan Terhadap Review Score (6.4)
```sql
SELECT 
    CASE 
        WHEN DATE(o.order_delivered_customer_date) <= DATE(o.order_estimated_delivery_date) THEN 'On-Time / Early'
        ELSE 'Late'
    END AS delivery_performance,
    COUNT(DISTINCT o.order_id) AS total_orders,
    ROUND(AVG(r.review_score), 2) AS avg_review_score,
    COUNT(CASE WHEN r.review_score = 5 THEN 1 END) AS count_5_star,
    COUNT(CASE WHEN r.review_score = 1 THEN 1 END) AS count_1_star
FROM orders_clean o
JOIN order_reviews r ON o.order_id = r.order_id
WHERE o.order_status = 'delivered'
  AND o.order_delivered_customer_date IS NOT NULL
  AND o.order_estimated_delivery_date IS NOT NULL
GROUP BY 1;
```
### Disparitas Wilayah & Ongkos kirim (6.5)
```sql
SELECT 
    c.customer_state,
    COUNT(o.order_id) AS total_orders,
    ROUND(
        (COUNT(CASE WHEN o.order_delivered_customer_date <= o.order_estimated_delivery_date THEN 1 END)::NUMERIC / COUNT(o.order_id)) * 100, 2
    ) AS on_time_rate_pct,
    ROUND(AVG(DATE_PART('day', o.order_delivered_customer_date - o.order_purchase_timestamp))::NUMERIC, 1) AS avg_delivery_days,
    ROUND(AVG(oi.freight_value)::NUMERIC, 2) AS avg_freight_value
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN order_items oi ON o.order_id = oi.order_id
WHERE o.order_status = 'delivered'
  AND o.order_delivered_customer_date IS NOT NULL
GROUP BY c.customer_state
ORDER BY avg_delivery_days DESC;
```
