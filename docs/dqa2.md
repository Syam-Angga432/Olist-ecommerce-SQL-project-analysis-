# DQA (DATA QUALITY ASSESSMENT)
* DATA QUALITY ASSESSMENT (DQA) - OLIST E-COMMERCE DATASET
* Database : olist_project
* Tool     : PostgreSQL

## TABEL ROWS
| No | tabel            | total_rows |
|----|------------------|------------|
| 1  | product_category | 71         |
| 2  | customers        | 99441      |
| 3  | geolocation      | 1000163    |
| 4  | sellers          | 3095       |
| 5  | products         | 32951      |
| 6  | orders           | 99441      |
| 7  | order_items      | 112650     |
| 8  | order_payment    | 103886     |
| 9  | order_reviews    | 99224      |

## FINDINGS
### 1. product_categories
pada product_categories terdapat produk kategori yang belum lengkap yaitu:
`pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos`
### 2. customers
duplikasi pada kolom `customer_unique_id` adalah hal yang wajar, `customer_id` dan `customer_unique_id` merupakan dua kolom dengan fungsi yang berbeda. terdapat perbedaan jumlah dimana `customer_id` (99441) dan `customer_unique_id` (96.096). (3345) di indikasikan sebagai repeat order.
### 3. products
terdapat null sebanyak 610 `product_category_name`begitu pula pada `product_name_lenght`,`product_description_lenght`,`product_photos_qty`.
### 4. orders
* pada `tabel orders` (96478) order memiliki `order_status` "delivered" atau 97,02% dari total
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

### 5. ORDER_PAYMENTS
* terdapat 2 `order_id` dengan payment_instalments < 1

<img width="235" height="50" alt="image" src="https://github.com/user-attachments/assets/16d68237-2080-435d-94d4-4cec8c52f157" />

<img width="1159" height="139" alt="image" src="https://github.com/user-attachments/assets/f01cbc1d-ab1f-48ae-bcdf-ece7f72565dc" />
