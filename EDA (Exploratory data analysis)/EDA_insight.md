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
2. Menghitung monthly sales.
3. Menghitung monthly order volume.
4. Menghitung Month-over-Month (MoM) growth.
5. Mengidentifikasi periode dengan pertumbuhan dan penurunan terbesar.
### finding
- Sales mengalami fluktuasi sepanjang periode observasi.
- Terdapat periode dengan pertumbuhan maupun penurunan monthly sales.
- MoM growth membantu mengidentifikasi perubahan performa jangka pendek.
### Output
- Monthly Sales Trend
- Monthly Order Trend
- Monthly Sales + MoM Growth
### Kesimpulan
Sales Olist menunjukkan pola yang dinamis sepanjang periode observasi. Analisis MoM memberikan gambaran mengenai perubahan performa penjualan dari bulan ke bulan dan dapat digunakan untuk mengidentifikasi periode pertumbuhan maupun penurunan.

## EDA 3 PRODUCT PERFORMANCE
### tujuan 

### Proses Analisis

### finding

### Output

### Kesimpulan


