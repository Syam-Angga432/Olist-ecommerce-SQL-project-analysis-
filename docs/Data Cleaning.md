# DATA CLEANING - OLIST E-COMMERCE DATASET

## PURPOSE
Proses ini bertujuan menghasilkan dataset yang andal dan siap dianalisis dengan cara menindaklanjuti hasil data profiling. Setiap tindakan—baik perbaikan data, pelabelan, maupun penyaringan—didokumentasikan secara rapi untuk menjaga validitas data tanpa menghilangkan informasi penting di dalamnya.

## ANALYSIS PROCESS
```text
├── 1. product_categories 
│         └── issue : data tidak lengkap | penanganan : imputasi/menambah data dengan referensi dari tabel lain.
│                             
├── 2. products
│         └── issue : data null          | penanganan : imputasi/mengubah/melabeli data null/flagging.
│                
├── 3. orders
│         ├── issue : data null          | penanganan : imputasi/mengubah/melabeli data null/flagging.
│         └── issue : invalid data       | penanganan : imputasi/data correction/ transform into null/dropping/flagging
│
└── 4. order_payments
          └── issue : invalid data       | penanganan : imputasi/data correction/ transform into null/dropping/flagging
```

## FINDING ISSUE 
### 1. product_categories
pada product_categories terdapat produk kategori yang belum lengkap yaitu:
`pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos` (`pc_gamer`,`Kitchen_Appliances_&_Food_Prep`)
### 2. products
terdapat null sebanyak 610 `product_category_name`begitu pula pada `product_name_lenght`,`product_description_lenght`,`product_photos_qty`.
### 3. orders
* **terdapat null pada** <img width="365" height="52" alt="image" src="https://github.com/user-attachments/assets/cbd83df1-9c3d-493f-a70d-4aef44ee1b63" />

* **cek jumlah null Per-Status** <img width="995" height="104" alt="image" src="https://github.com/user-attachments/assets/bf4a06a3-baf1-42dc-8f51-e1d7c83e663a" />
  
* status `delivered`dimana seharusnya tidak memiliki null/semua kolom terisi, terdapat null mulai dari `approved at` hingga `customer_date`.

* order_delivered_carrier_date < order_purchase_timestamp(invalid_carrier_delivery) sebanyak 166

* order_delivered_customer_date < order_delivered_carrier_date(invalid_delivery_sequence) sebanyak 23

* anomali tanggal 23 dan 166 bukan merupakan overlap atau terpisah, sehingga totalnya adalah 189
### 4. ORDER_PAYMENTS
* terdapat 2 `order_id` dengan payment_instalments < 1

## Data Cleaning Execution
### 1. product_categories
untuk menambahkan `pc_gamer` dan `portateis_cozinha_e..`dapat ke kolom menggunakan `insert into`, namun saya akan membuat tabel baru, yaitu `product_categories_v2` sekaligus menambahkan `pc_gamer` dan `portateis_cozinha_e..` didalamnya. tujuan nya adalah agar tidak tidak mengubah data RAW, sehingga untuk analisis kedepannya menggunakan `product_categories_v2`

**hasil:**
* RAW data `product_categories`<img width="419" height="56" alt="image" src="https://github.com/user-attachments/assets/461f0fd8-d6ed-4753-bfe4-4b38906dd9cc" />

* `product_categories_v2`<img width="419" height="95" alt="image" src="https://github.com/user-attachments/assets/b7e7346c-5fdf-4a81-8634-937053cf7abb" />

### 2. products
610 NULL pada kolom `product_category_name` akan sulit diidentfikasi dengan sumber yang terbatas, misal butuh image product sehingga bisa mengetahui masuk kategori apa atau data terbaru dari tabel yang berbeda. namun karena sumber tersebut tidak tersedia salah satu cara paling aman adalah dengan mengubah null menjadi `unknown` atau `uncategorized` agar terdeteksi pada proses analisis kedepannya.

sama seperti sebelumnya akan dibuat tabel baru, yaitu `products_clean` agar tidak mengubah RAW data.

**hasil:**      
<img width="239" height="53" alt="image" src="https://github.com/user-attachments/assets/66441000-cdd8-4230-b97e-cef1844bbd33" />

### 3. orders
* null `order_approved_at`, `order_delivered_carrier_date`,`order_delivered_customer_date` lebih baik dibirkan saja, karena Menjaga Integritas Analisis SLA / Durasi Logistik serta membuat sistem sengaja mengabaikannya (exclude) saat perhitungan durasi pengiriman. namun bila ingin tetap melakukan cleaning pada timestamp yang paling mungkin adalah pada `order_approved_at` karena selisih waktunya cukup tipis dengan order_purchase_timestamp.
* untuk `invalid_carrier_delivery` dan `invalid_delivery_sequence` dengan total invalid timestamp 188, disebabkan urutan timestamp yang salah akan di null-kan karena tidak bisa secara sembarangan menentukan durasi masing-masing order dan tetap Menjaga Integritas Analisis SLA. sehingga pada tahap delivery performance order dengan timestamp null akan di abaikan(exclude).

**hasil:**
* <img width="310" height="60" alt="image" src="https://github.com/user-attachments/assets/53cad3b8-7df0-411a-ae26-10326c0184f4" />
* <img width="137" height="53" alt="image" src="https://github.com/user-attachments/assets/cccc1cf1-c190-4a91-91f9-a0873a4cb08b" />
* <img width="161" height="51" alt="image" src="https://github.com/user-attachments/assets/54df56a8-885e-4a95-b35a-30a564dfd52d" />

### 4. ORDER_PAYMENTS
credit_card, payment_installments = 0 di ubah menjadi 1, Nilai 0 pada pembayaran kartu kredit merupakan anomali logika (data entry anomaly)

**hasil:**
<img width="232" height="50" alt="image" src="https://github.com/user-attachments/assets/bedb34b7-9a05-42b2-83de-23216bcb59af" />

## OUTPUT
1. product_categories _v2 : menambahkan `pc_gamer` dan `portateis_cozinha_e_preparadores_de_alimentos` (`pc_gamer`,`Kitchen_Appliances_&_Food_Prep`) pada  `product_category_name`dan `product_category_name_english`
2. products_clean         : mengubah  `product_category_name` null menjadi `uncategorized`
3. orders_clean           : imputasi `order_approved_at`, mengubah `invalid_carrier_delivery`dan `invalid_delivery_sequence` menjadi null
4. order_payments_clean   : imputasi  `payment_installments`

## ANALISIS TABEL LOGIC
```text
├── 1. RAW DATA (QDA)
│   ├─ product_categories
│   ├─ products
│   ├─ orders
│   └── order_payments
└── 2. CLEAN DATA (EDA / ANALYSIS)
    ├── product_categories _v2
    ├── products_clean
    ├── orders_clean
    └── order_payments_clean
```
## SUMMARY
seluruh issue yang ditemukan pada tahap DQA telah di tindak lanjuti, ada kemungkin masih ada beberapa ketidaksesuaian data yang tidak di cari tau lebih lanjut atau tidak ditindak lanjuti namun sifatnya tidak crucial atau berpengaruh signifikan terhadap tahapan analisis selanjutnya. pada eksekusi data cleaning kali ini telah dilakukan imputasi,menambahkan data,transform into null, dan ada beberapa data null yang dibiarkan apa adanya/tidak ditindak lanjuti karena bukan merupakan anomali yang mengurangi kualitas data.
