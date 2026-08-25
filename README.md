# Olist E-Commerce Sales & Logistics Performance Analysis
## 1. Executive Summary
Proyek ini menganalisis dataset publik dari Olist—platform marketplace e-commerce terbesar di Brazil—yang mencakup lebih dari 100.000 transaksi dari periode 2016 hingga 2018. Mengingat e-commerce di Brazil menghadapi tantangan geografis dan logistik yang kompleks, analisis ini bertujuan untuk mengevaluasi dinamika penjualan, distribusi pelanggan, performa pengiriman, serta dampaknya terhadap kepuasan pelanggan (review score).

Menggunakan PostgreSQL, analisis dilakukan dengan menggabungkan dan mengagregasi 8 tabel data terelasi untuk mengekstrak business insight yang actionable. Hasil analisis ini memberikan gambaran menyeluruh tentang faktor-faktor penentu pertumbuhan revenue dan efisiensi rantai pasok Olist guna mendukung pengambilan keputusan berbasis data.

---

## 📊 Key Performance Metrics

> **Catatan Metrik:** Untuk mencerminkan *realized revenue* dan performa operasional yang bersih, seluruh metrik transaksi dihitung berdasarkan pesanan dengan status **`delivered`**. Angka total ekosistem keseluruhan (*all order statuses*) dicantumkan sebagai referensi skop data.

| metrics kategory | Key Metric | value (Delivered Only/completed order) | overall (All Statuses) | description |
| :--- | :--- | :---: | :---: | :--- |
| **Financial** | **total transaction value** | **R$ 15.419.773,75** | R$ 15.843.553,24 | Total nilai transaksi terkirim (Produk + biaya pengiriman) |
| | **Product Sales** | **R$ 13.221.498,11** | R$ 13.591.643,70 | Nilai total produk  |
| | **Average Order Value** | **R$ 159,83** | R$ 160,58 | Rata-rata nilai per transaksi terkirim |
| | **Freight Ratio** | **14,26%** | 14,21% | Porsi biaya pengiriman dari total transaksi |
| **Marketplace Scale**| **Total Orders** | **96.478** | 99.441 | Total transaksi |
| | **Total Customers** | **93.099** | 96.096 | Jumlah pelanggan unik (*customer_unique_id*) |
| | **Active Sellers** | **2.970** | 3.095 | Penjual yang berhasil menyelesaikan pesanan |
| | **Items Sold** | **110.197** | 112.650 | Total unit barang terkirim |
| **Quality & Ops** | **Customer Repeat Rate**| **3,12%** | - | Hanya 3,12% pelanggan yang belanja >1 kali |
| | **Late Delivery Rate** | **8,11%** | - | Persentase pengiriman melewati estimasi |
| | **Avg. Delivery Days** | **12,56 Hari** | - | Rata-rata durasi kirim (vs. estimasi 23,74 hari) |
| | **Overall Review Score**| **4,15 / 5.00** | - | Skor kepuasan pelanggan rata-rata |

---

## 2. Key Question
### Overall Business & Sales Dynamics:
* Bagaimana kinerja bisnis Olist secara keseluruhan, dan bagaimana tren pertumbuhan penjualan berubah dari waktu ke waktu (MoM)?
* Kategori produk mana yang memberikan kontribusi revenue terbesar serta mendominasi volume penjualan?
### Customer Retention & Value:
* Seberapa kuat tingkat retensi dan pembelian berulang (repeat purchase) pelanggan di platform Olist?
* Apakah pelanggan berulang (repeat customers) memiliki nilai transaksi dan pola pembelian yang berbeda dibandingkan one-time customers?
### Seller Concentration & Operational Health:
* Seberapa terkonsentrasi revenue pada segmen seller tertentu, dan seller mana yang memiliki indikasi masalah operasional?
### Order Cancellations & Revenue Impact:
* Di mana pembatalan pesanan (cancellations) paling banyak terjadi, dan seberapa besar dampaknya terhadap potensi kerugian bisnis?
### Logistics & Customer Satisfaction:
* Seberapa baik performa Olist dalam mengirimkan pesanan tepat waktu, dan bagaimana keterlambatan pengiriman mempengaruhi tingkat kepuasan pelanggan (review score)?

## 3. Dataset & Database Schema

Analisis ini menggunakan dataset publik dari Olist yang disimpan dalam database PostgreSQL bernama **`olist_project`**. Database ini terdiri dari 9 tabel terelasi yang mencakup seluruh rantai transaksi e-commerce, mulai dari pemesanan, produk, pembayaran, hingga logistik dan ulasan pelanggan.

### Tabel Database (`olist_project`)

* **`orders`**: Menyimpan data transaksi utama (ID pesanan, ID pelanggan, status pesanan, dan *timestamp* proses pengiriman).
* **`order_items`**: Detail item di setiap pesanan (ID produk, ID penjual, harga item, dan biaya ongkir/*freight*).
* **`customers`**: Informasi pelanggan (ID unik pelanggan, lokasi kota, dan *state*).
* **`products`**: Spesifikasi produk (ID produk, kategori, ukuran, dan berat).
* **`sellers`**: Informasi penjual/mitra Olist (ID penjual, lokasi kota, dan *state*).
* **`order_reviews`**: Ulasan dan skor rating (1–5) yang diberikan oleh pelanggan untuk setiap pesanan.
* **`order_payments`**: Metode pembayaran (kartu kredit, *boleto*, voucher) dan jumlah cicilan (*installments*).
* **`geolocation`**: Data koordinat geospasial (kode pos, latitude, dan longitude) di wilayah Brazil.
* **`product_categories`**: Tabel translasi nama kategori produk dari bahasa Portugis ke bahasa Inggris.

### Entity Relationship Diagram (ERD)
<img width="2201" height="1049" alt="ERD" src="https://github.com/user-attachments/assets/ba3c0f30-ccdf-4162-85de-19a558ff006b" />

## 4. Analysis Workflow & Analytical approach

Proses analisis dilakukan secara terstruktur menggunakan PostgreSQL melalui 4 tahapan utama:

### 1. Data Preparation
* **Creating Database & Tables:** Membuat database `olist_project` dan mendefinisikan skema 9 tabel sesuai dengan tipe data yang tepat.
* **Constraints Setup:** Menetapkannya *Primary Key* (PK) dan *Foreign Key* (FK) untuk menjaga integritas relasional antar-tabel.

### 2. Data Quality Assessment (DQA)
* **Null Value Check:** Identifikasi kolom dengan nilai yang hilang (*missing values*), terutama pada tanggal pengiriman dan skor ulasan.
* **Duplicate Check:** Pemeriksaan duplikasi pada ID unik (seperti `order_id` dan `customer_id`).
* **Data Validation:** Memastikan inkonsistensi data, seperti tanggal pengiriman yang mendahului tanggal pemesanan atau format teks kategori.

### 3. Data Cleaning
* **Handling Nulls:** Penanganan nilai *null* secara spesifik (misal: imputasi, *flagging*, atau mengeklusi transaksi yang dibatalkan dari perhitungan durasi pengiriman).
* **Handling Duplicates:** Pembersihan data ganda untuk memastikan agregasi *revenue* dan jumlah pesanan akurat.

### 4. Exploratory Data Analysis (EDA) & Business Analysis
* **Data Understanding:** Eksplorasi awal untuk memahami distribusi statistik, rentang periode data (2016–2018), dan tren umum transaksi.
* **Answering Key Business Questions:** Mengeksekusi *query* SQL tingkat lanjut (`JOINs`, `CTEs`, `Window Functions`) untuk menjawab 5 pilar pertanyaan bisnis yang telah ditetapkan

## 5. Key Finding

### 5.1 Performa Bisnis Secara Keseluruhan
Olist berhasil mencatatkan sekitar **96 ribu transaksi** dari **96 ribu pembeli unik**, dengan total nilai transaksi mencapai **15,8 juta**.
* **Penjualan Naik-Turun:** Tren penjualan tidak tumbuh mulus, melainkan mengalami naik-turun dari bulan ke bulan (MoM).
* **Bukan Cuma dari Satu Segmen:** Pendapatan Olist tersebar di banyak kategori produk dan volume pesanan, bukan hanya bergantung pada satu jenis barang saja.

> **Implikasi Bisnis:**
> Olist sudah punya modal besar dalam menarik pembeli baru. Namun, tantangan terbesarnya adalah menjaga tren penjualan agar tetap stabil. Kuncinya ada pada dua hal: menjaga ketersediaan produk terlaris dan membuat pembeli mau datang kembali.

---

### 5.2 Performa Kategori & Produk
Penjualan terbanyak didominasi oleh beberapa kategori favorit, meskipun variasi produk yang dijual di platform sebenarnya sangat luas.
* **Kategori Unggulan Jadi Kunci:** Produk yang menyumbang pendapatan terbesar perlu terus dipantau stok dan ketersediaannya. terdapat 5 produk kategory yang memberikan _total sales_ terbanyak (health_beauty,watches_gifts,bed_bath_table,sports_leisure,computers_accessories)
* **Ada Data Kategori yang Hilang:** Terdapat **610 produk** yang tidak memiliki label kategori (berdampak pada 1.451 pesanan).
* **Dampaknya Masih Tergolong Kecil:** Isu data hilang ini hanya mencakup sekitar **1,31% dari total penjualan**, jadi tidak mengganggu gambaran besar analisis.
* 

> **Implikasi Bisnis:**
> Secara keseluruhan data kategori Olist sangat bisa diandalkan. Untuk produk yang kategorinya hilang, kita cukup mengelompokkannya ke dalam label *Uncategorized* di laporan tanpa perlu membuang datanya.

---

### 5.3 Retensi & Nilai Pelanggan (Temuan Paling Krusial)
Di bagian inilah masalah terbesar sekaligus peluang terbesar Olist terungkap:

* **Hampir Semua Pembeli Cuma Belanja Sekali:** **96,88% pelanggan** tidak pernah kembali lagi setelah pembelian pertama.
* **Hanya 3,12% yang Jadi Pembeli Setia (*Repeat Customer*).**
* **Pembeli Setia Jauh Lebih Loyal & Beli Lebih Banyak:**
  * Pembeli 1x (*One-time customer*): Rata-rata belanja **161,49**
  * Pembeli Setia (*Repeat customer*): Rata-rata belanja **310,49** (hampir 2x lipat!)
* **Kontribusi Penjualan Masih Kecil:** Karena jumlahnya sangat sedikit, pembeli setia baru menyumbang **5,71% dari total omzet**.
* **Perilaku Pembeli Setia:**
  * **52,91%** mencoba membeli produk dari kategori yang berbeda.
  * **47,09%** memilih membeli barang di kategori yang sama.

> **Implikasi Bisnis:**
> Masalah utama Olist bukan kurang promosi untuk cari pembeli baru, melainkan **gagal menahan pembeli agar mau belanja lagi**. Padahal, satu pembeli setia nilainya jauh lebih besar dibanding pembeli baru. Menarik pembeli lama untuk transaksi kedua jauh lebih menguntungkan.

---

### 5.4 Kondisi Mitra Penjual (*Seller*)
Kabar baiknya, pendapatan Olist tidak menumpuk di sedikit penjual saja.
* **Pendapatan Tersebar Merata:** 10 *seller* teratas hanya menyumbang **12,80% penjualan**, dan Top 20 *seller* menyumbang **20,85%**. Olist tidak bergantung pada segelintir *seller* raksasa.
* **Isu Pembatalan Pesanan (*Cancellation*):** Ada beberapa *seller* yang memiliki persentase pembatalan pesanan cukup tinggi.
* **Harus Cermat Melihat Data:** Persentase pembatalan tinggi sering kali terjadi pada *seller* yang jumlah pesanannya cuma sedikit (misal: batal 1 dari 2 pesanan = 50% *cancellation rate*). Jadi, ini tidak selalu berarti masalah besar.

> **Implikasi Bisnis:**
> Olist relatif aman dari risiko ketergantungan pada *seller* tertentu. Risiko yang ada lebih ke arah ketidakkonsistenan kualitas layanan dari *seller-seller* tertentu.

---

### 5.5 Pembatalan Pesanan & Impact pada Omzet
Tingkat pembatalan pesanan tidak merata di seluruh platform.
* Beberapa kategori produk dan *seller* memang punya tingkat pembatalan yang lebih tinggi dari rata-rata.
* Namun, secara keseluruhan, pembatalan pesanan bukanlah masalah utama yang merusak bisnis Olist jika dibandingkan dengan masalah pengiriman dan retensi pembeli.

> **Implikasi Bisnis:**
> Olist tidak perlu membuat aturan baru yang memberatkan semua *seller*. Cukup fokus menegur atau memantau *seller-seller* tertentu yang tingkat pembatalannya memang sangat parah dan merugikan omzet.

---

### 5.6 Pengiriman Barang & Kepuasan Pembeli (Temuan Terkuat)
Pengiriman adalah faktor nomor satu yang menentukan senang atau kecewanya pembeli Olist:

* **8,11% Pesanan Mengalami Keterlambatan.**
* Rata-rata waktu kirim aktual adalah **12,5 hari** (jauh lebih cepat dibanding janji estimasi awal yang rata-rata 23,7 hari).
* Meski mayoritas paket sampai tepat waktu, **paket yang terlambat langsung menghancurkan nilai ulasan (*rating*)**.

| Status Pengiriman | Rata-Rata Rating | Persentase Rating Buruk (Bintang 1–2) |
| :--- | :---: | :---: |
| **Tepat Waktu (*On Time*)** | **4,29** | **9,26%** |
| **Terlambat (*Late*)** | **2,57** | **54,12%** |

Paket yang terlambat memiliki peluang **5,8 kali lebih besar** untuk mendapat ulasan buruk dari pembeli.

> **Implikasi Bisnis:**
> Ada hubungan yang sangat kuat antara keterlambatan pengiriman dengan kekecewaan pembeli. Memperbaiki masalah logistik adalah cara tercepat dan paling ampuh bagi Olist untuk meningkatkan kepuasan pelanggan.

---

### 5.7 Ringkasan Prioritas Bisnis

Berdasarkan seluruh hasil analisis, berikut adalah urutan prioritas yang harus ditangani Olist:

| Prioritas | Isu Bisnis | Bukti Data | Tingkat Dampak |
| :---: | :--- | :--- | :--- |
| 🔴 **1** | **Keandalan Pengiriman** | 8,11% paket terlambat; 54,12% dapat rating buruk | **Sangat Tinggi** (merusak reputasi) |
| 🔴 **2** | **Retensi Pembeli** | Cuma 3,12% yang belanja lagi | **Sangat Tinggi** (potensi omzet terbuang) |
| 🟠 **3** | **Isu Operasional Seller** | Ada *seller* dengan angka pembatalan ekstrem | **Sedang** (perlu pemantauan khusus) |
| 🟢 **4** | **Konsentrasi Seller** | Top 20 seller cuma menguasai 20,85% omzet | **Rendah** (ekosistem sudah sehat) |

---

## 6. Rekomendasi Langkah Konkret

### 6.1 Perbaiki Keandalan Pengiriman Barang
* **Temuan:** Paket yang terlambat membuat rating merosot drastis (rating rata-rata turun dari 4,29 ke 2,57).
* **Rekomendasi:**
  * Buat sistem peringatan dini untuk pesanan yang mendekati batas waktu estimasi kirim.
  * Pantau performa *seller* dan jasa ekspedisi yang paling sering terlambat.
* **KPI yang Dipantau:** *Late Delivery Rate*, *Average Delivery Days*, *Low-score Review Rate*.
* **Hasil yang Diharapkan:** Menekan angka keterlambatan kirim akan langsung mengurangi ulasan bintang 1 dan 2 secara signifikan.

### 6.2 Buat Pembeli Baru Mau Belanja Kembali
* **Temuan:** Baru 3,12% pembeli yang mau transaksi lagi, padahal pembeli setia belanja hingga 310,49 (dibanding 161,49 pada pembeli baru).
* **Rekomendasi:**
  * Kirimkan voucher diskon atau penawaran khusus beberapa hari setelah barang pertama diterima.
  * Berikan rekomendasi produk yang relevan berdasarkan barang yang baru saja mereka beli.
* **KPI yang Dipantau:** *Repeat Customer Rate*, *Second-Purchase Conversion Rate*, *Revenue per Customer*.
* **Hasil yang Diharapkan:** Menaikkan persentase pembeli kedua akan meningkatkan pendapatan Olist tanpa perlu keluar biaya iklan besar untuk cari pembeli baru.

### 6.3 Dorong Pembelian Lintas Kategori
* **Temuan:** Lebih dari separuh pembeli setia (52,91%) mencoba membeli barang dari kategori yang berbeda pada transaksi selanjutnya.
* **Rekomendasi:**
  * Manfaatkan fitur rekomendasi *cross-selling* di aplikasi/website (misal: Pembeli HP diajak membeli aksesori atau elektronik lain).
* **KPI yang Dipantau:** *Cross-category Repeat Rate*, *Average Categories Purchased per Customer*.
* **Hasil yang Diharapkan:** Nilai keranjang belanja (*basket size*) pembeli akan meningkat karena mereka membeli lebih banyak variasi barang.

### 6.4 Tegur *Seller* yang Bermasalah
* **Temuan:** Ada beberapa *seller* dengan angka pembatalan yang sangat tinggi dan merugikan pembeli.
* **Rekomendasi:**
  * Buat sistem penilaian *seller* yang menggabungkan: **Volume Pesanan + Tingkat Pembatalan + Keterlambatan Kirim**.
  * Berikan sanksi atau pendampingan khusus hanya kepada *seller* yang bervolume besar tapi layanannya buruk.
* **KPI yang Dipantau:** *Seller Cancellation Rate*, *Canceled Orders*, *Late Delivery Rate*.
* **Hasil yang Diharapkan:** Pelayanan platform meningkat tanpa perlu memberatkan *seller-seller* kecil yang sebenarnya tidak bermasalah.

### 6.5 Pertahankan Keberagaman *Seller*
* **Temuan:** Penjualan Olist sudah sangat aman dan tidak didominasi oleh sedikit *seller* besar (Top 20 *seller* hanya menguasai ~20% omzet).
* **Rekomendasi:**
  * Pertahankan ekosistem yang sehat ini dan terus dorong *seller* baru untuk bergabung.
* **KPI yang Dipantau:** *Top 10 & Top 20 Seller Sales Contribution*.
* **Hasil yang Diharapkan:** Bisnis Olist tetap stabil dan tidak rentan jika ada satu atau dua *seller* besar yang hengkang dari platform.

---

## Kesimpulan Akhir

Dari seluruh analisis ini, masalah terbesar Olist ternyata bukan pada akuisisi *seller* atau penumpukan pendapatan pada pihak tertentu. Peluang terbesar Olist ada pada dua hal: **menjaga pengiriman agar tidak terlambat** dan **membuat pembeli baru mau bertransaksi kembali**. Saat ini, baru 3,12% pembeli yang kembali belanja, padahal pembeli setia menyumbang nilai transaksi dua kali lipat lebih besar. Di sisi lain, paket yang terlambat terbukti merusak kepuasan pelanggan secara drastis (54,12% menghasilkan ulasan buruk). Dengan memfokuskan perbaikan pada keandalan pengiriman serta strategi retensi pelanggan, Olist dapat meningkatkan pendapatan dan reputasi platform secara berkelanjutan.

---

## Keterbatasan (Limitations)

- **Profitabilitas:** Dataset tidak memuat COGS (Harga Pokok Penjualan), komisi, biaya operasional, pajak, atau komponen biaya lainnya yang diperlukan untuk menghitung laba bersih (*true profit/net profit*).

- **Definisi Pendapatan (*Revenue*):** Angka penjualan di laporan ini dihitung dari nilai total transaksi (GMV), bukan murni pendapatan komisi Olist (karena data skema komisi Olist tidak tersedia).

- **Data Produk yang Hilang:** Terdapat 610 produk yang tidak memiliki informasi kategori. Data ini dikategorikan sebagai `uncategorized` pada lapisan analitis (*analytical layer*) dan tidak bisa didentifikasi secara secara pasti dengan data yang ada.

- **Atribut Produk:** Beberapa detail informasi fisik produk memiliki *missing values* dan tidak diimputasi secara manual karena nilai aslinya tidak dapat diperkirakan secara andal serta tidak bersifat krusial untuk analisis utama.

- **Kualitas Data Operasional:** Ditemukan beberapa ketidaksesuaian stempel waktu (*timestamp*) terkait pengiriman yang membuat inkonsistensi durasi.
  
- **Retensi Pelanggan:** Analisis pembelian berulang (*repeat purchase*) hanya mencerminkan aktivitas pelanggan selama periode dataset yang tersedia dan mungkin tidak merepresentasikan retensi jangka panjang.

- **Kausalitas:** Analisis ini hanya menunjukkan pola hubungan antar data (asosiasi), bukan hubungan sebab-akibat langsung. terdapat kemungkinan ada faktor luar lain yang mempengaruhinya.

- **Analisis Geografis & Biaya Pengiriman:** Analisis wilayah utamanya dilakukan pada tingkat *state* (provinsi), sementara data mengenai jarak pengiriman pasti, rute logistik, dan biaya pengiriman aktual tidak tersedia.

## Data Source

https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce  

