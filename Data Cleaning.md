# DATA CLEANING - OLIST E-COMMERCE DATASET

## temuan pada tahap DQA 
### product_categories
pada product_categories terdapat produk kategori yang belum lengkap yaitu:
`pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos`
### products
terdapat null sebanyak 610 `product_category_name`begitu pula pada `product_name_lenght`,`product_description_lenght`,`product_photos_qty`.
### orders
* **terdapat null pada**

* **cek jumlah null Per-Status**

* keberadaan null pada status seperti `shipped`,`cancelled`,`unvailable` dinilai cukup wajar, misal pada `shipped` null pada `order_delivered_customer_date` wajar karena barang memang sedang dalam proses pengantaran sehingga belum sampai di tangan customer, maka dari itu `order_delivered_customer_date` tidak memiliki value/null. begitu juga dengan status lainnya
* namun pada status `delivered`dimana seharusnya tidak memiliki null/semua kolom terisi, terdapat null mulai dari `approved at` hingga `customer_date`.

* **order_delivered_carrier_date < order_purchase_timestamp(invalid_carrier_delivery)**

* **order_delivered_customer_date < order_delivered_carrier_date(invalid_delivery_sequence)**

* 23 anomali tanggal tidak termasuk kedalam 166 atau terpisah, sehingga totalnya adalah 189

### ORDER_PAYMENTS
* terdapat 2 `order_id` dengan payment_instalments < 1

