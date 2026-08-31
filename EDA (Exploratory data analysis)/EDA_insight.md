# EDA RESULT / INSIGHT

## EDA 1 OVERALL BUSINESS PERFORMANCE KPI
### tujuan 
Mengukur kondisi bisnis secara keseluruhan melalui baseline KPI dan memahami distribusi status order untuk mengetahui kondisi transaksi Olist.
### Proses Analisis
1. Menghitung total orders, total customers, active sellers.
2. menghitung AOV
3. Menghitung total product sales / transaction value.
4. Menghitung distribusi order berdasarkan `order_status`.
5. Mengidentifikasi status order yang paling dominan.
6. Membandingkan nilai transaksi berdasarkan status order.
### finding
- Olist memiliki sekitar 99441 orders, 96096 customers, 3095 active sellers
- Average order value adalah 160.58 
- Mayoritas order berada pada status delivered.
- Status non-delivered seperti canceled, shipped, processing, dan lainnya memiliki proporsi jauh lebih kecil.
- Hal ini menunjukkan bahwa sebagian besar order berhasil mencapai tahap penyelesaian.
### Output
- Baseline KPI
- Order status distribution
- Sales / transaction value by order status
### Kesimpulan
Olist memiliki volume transaksi yang besar, yakni 99441 order dan 96096 total customers dengan mayoritas order berhasil diselesaikan yakni 96478 order. Meskipun demikian, keberadaan canceled dan non-delivered orders menunjukkan adanya operational issues yang perlu dianalisis lebih lanjut pada tahap cancellation dan delivery performance.
  
## EDA 2 SALES PERFORMANCE 
### tujuan 
Menganalisis perkembangan penjualan dari waktu ke waktu dan mengidentifikasi pola pertumbuhan maupun penurunan sales.
### Proses Analisis
1. Mengelompokkan transaksi berdasarkan bulan.
2. Menghitung monthly sales dan order volume.
3. Menghitung Month-over-Month (MoM) growth.
4. Mengidentifikasi periode dengan pertumbuhan dan penurunan terbesar.
5. Menghitung Sales by State
6. Menghitung Sales by Payment Method
### finding
- Sales mengalami fluktuasi sepanjang periode observasi.
- Terdapat periode dengan pertumbuhan maupun penurunan monthly sales.
- MoM growth membantu mengidentifikasi perubahan performa jangka pendek.
- total sales terbanyak adalah dari SP (state) dengan 40501 total orders dan 5769703.15 total transaction value
- total sales terbanyakan menggunakan payment method credit 
### Output
- Monthly Sales Trend
- Monthly Order Trend
- Monthly Sales + MoM Growth
- sales by state
- sales bu payment method
### Kesimpulan
Sales Olist menunjukkan pola yang dinamis sepanjang periode observasi. Analisis MoM memberikan gambaran mengenai perubahan performa penjualan dari bulan ke bulan dan dapat digunakan untuk mengidentifikasi periode pertumbuhan maupun penurunan.

## EDA 3 PRODUCT PERFORMANCE
### tujuan 
Mengidentifikasi kategori dan produk yang menjadi kontributor utama terhadap sales.
### Proses Analisis
1. Menghubungkan orders, order_items, dan products.
2. Menghitung sales per product category.
3. Mengurutkan kategori berdasarkan sales dan volume.
4. Mengidentifikasi top-performing categories dan product.
### finding
- Sales tidak terdistribusi secara merata antar kategori.
- Beberapa kategori/produk menjadi revenue drivers utama.
- Terdapat perbedaan antara kategori/produk dengan sales tinggi dan kategori/produk dengan volume transaksi tinggi.
### Output
- Sales by category
- Top product categories
- Top product 
- Product/category sales contribution
### Kesimpulan
Beberapa kategori memberikan kontribusi signifikan terhadap sales Olist. Kategori dengan kontribusi tinggi perlu diperhatikan sebagai revenue drivers, sementara kategori dengan volume tinggi tetapi nilai transaksi relatif rendah perlu dievaluasi dari sisi product mix dan average order value

## EDA 4 CUSTOMERS PERFORMANCE
### tujuan 
Memahami perilaku pembelian customer, tingkat repeat purchase, serta perbedaan nilai customer antara one-time dan repeat customers.
### Proses Analisis
1. Mengidentifikasi customer berdasarkan customer_unique_id.
2. Menghitung jumlah order per customer.
3. Mengelompokkan customer menjadi:
4.  * one-time
    * repeat
5. Menghitung repeat purchase rate.
6. Membandingkan average customer value.
7. Menganalisis pola repeat purchase.
### finding
- Repeat customer rate hanya 3.12%.
- 96.88% customer merupakan one-time customers.
- Average customer value:
  * One-time: 161.49
  * Repeat: 310.49
- Repeat customers memiliki nilai rata-rata sekitar dua kali customer one-time.
- 52.91% repeat customers melakukan pembelian lintas kategori.
- 47.09% melakukan pembelian dalam kategori yang sama.

### Output

### Kesimpulan
