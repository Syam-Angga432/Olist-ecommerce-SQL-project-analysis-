# DATA CLEANING - OLIST E-COMMERCE DATASET

## temuan masalah pada tahap DQA 
### product_categories
pada product_categories terdapat produk kategori yang belum lengkap yaitu:
`pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos` (`pc_gamer`,`Kitchen_Appliances_&_Food_Prep`)
### products
terdapat null sebanyak 610 `product_category_name`begitu pula pada `product_name_lenght`,`product_description_lenght`,`product_photos_qty`.
### orders
* **terdapat null pada** <img width="365" height="52" alt="image" src="https://github.com/user-attachments/assets/cbd83df1-9c3d-493f-a70d-4aef44ee1b63" />

* **cek jumlah null Per-Status** <img width="995" height="104" alt="image" src="https://github.com/user-attachments/assets/bf4a06a3-baf1-42dc-8f51-e1d7c83e663a" />
  
* status `delivered`dimana seharusnya tidak memiliki null/semua kolom terisi, terdapat null mulai dari `approved at` hingga `customer_date`.

* order_delivered_carrier_date < order_purchase_timestamp(invalid_carrier_delivery) sebanyak 166

* order_delivered_customer_date < order_delivered_carrier_date(invalid_delivery_sequence) sebanyak 23

* anomali tanggal 23 dan 166 bukan merupakan overlap atau terpisah, sehingga totalnya adalah 189
### ORDER_PAYMENTS
* terdapat 2 `order_id` dengan payment_instalments < 1

## Data Cleaning Execution
### product_categories
untuk menambahkan `pc_gamer` dan `portateis_cozinha_e..`dapat ke kolom menggunakan `insert into`, namun saya akan membuat tabel baru, yaitu `product_categories_v2` sekaligus menambahkan `pc_gamer` dan `portateis_cozinha_e..` didalamnya. tujuan nya adalah agar tidak tidak mengubah data RAW, sehingga untuk analisis kedepannya menggunakan `product_categories_v2`

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
**hasil:**
* RAW data `product_categories`<img width="419" height="56" alt="image" src="https://github.com/user-attachments/assets/461f0fd8-d6ed-4753-bfe4-4b38906dd9cc" />

* `product_categories_v2`<img width="419" height="95" alt="image" src="https://github.com/user-attachments/assets/b7e7346c-5fdf-4a81-8634-937053cf7abb" />

### products
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
**hasil:**      
<img width="239" height="53" alt="image" src="https://github.com/user-attachments/assets/66441000-cdd8-4230-b97e-cef1844bbd33" />

### orders

