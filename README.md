
# Monthly Report of The Look E-Commerce Data

Page ini bertujuan untuk mendokumentasikan cara memperoleh monthly report berdasarkan **produk** dengan jumlah item terjual terbanyak tiap bulannya dari The Look E-Commerce data dari BigQuery public data menggunakan SQL BigQuery.

Query SQL tersebut akan menghasilkan temporary table dengan sample sebagai berikut.

![](https://github.com/auliaaaz/reportTheLookEcommerce/blob/main/result.jpg)

### Query
Query SQL ini digunakan untuk mengambil data bulanan produk dari dataset ecommerce dengan mengurutkan produk berdasarkan jumlah item terjual terbanyak setiap bulan

```sql
SELECT month, year, product_id, product_name, total_item, total_revenue, number_of_order, number_of_customer
FROM
    (SELECT a.*, ROW_NUMBER() OVER(PARTITION BY a.year, a.month order by a.total_item desc) allmonth
    FROM
        (SELECT EXTRACT(MONTH FROM o.created_at) AS month,
                EXTRACT(YEAR FROM o.created_at) AS year,
                p.id AS product_id,
                p.name AS product_name,
                SUM(o.num_of_item) AS total_item,
                ROUND(SUM(o.num_of_item * oi.sale_price), 2) AS total_revenue,
                COUNT(DISTINCT o.order_id) number_of_order,
                COUNT(DISTINCT o.user_id) number_of_customer
        FROM bigquery-public-data.thelook_ecommerce.order_items oi
        JOIN bigquery-public-data.thelook_ecommerce.orders o
        ON oi.order_id = o.order_id
        JOIN bigquery-public-data.thelook_ecommerce.products p
        ON oi.product_id = p.id
        WHERE o.status NOT IN ('Cancelled', 'Returned')
        GROUP BY year, month, product_id, product_name) a
    ) b
WHERE allmonth=1
ORDER BY year DESC, month DESC
```

### Penjelasan
Akan dijelaskan pada bagian paling pertama SQL query ini dijalankan yaitu pada **inner query `a`**
```sql
(SELECT EXTRACT(MONTH FROM o.created_at) AS month,
                EXTRACT(YEAR FROM o.created_at) AS year,
                p.id AS product_id,
                p.name AS product_name,
                SUM(o.num_of_item) AS total_item,
                ROUND(SUM(o.num_of_item * oi.sale_price), 2) AS total_revenue,
                COUNT(DISTINCT o.order_id) number_of_order,
                COUNT(DISTINCT o.user_id) number_of_customer
        FROM bigquery-public-data.thelook_ecommerce.order_items oi
        JOIN bigquery-public-data.thelook_ecommerce.orders o
        ON oi.order_id = o.order_id
        JOIN bigquery-public-data.thelook_ecommerce.products p
        ON oi.product_id = p.id
        WHERE o.status NOT IN ('Cancelled', 'Returned')
        GROUP BY year, month, product_id, product_name) a
```
1. Query `SELECT` memilih:

* month dengan query `EXTRACT(MONTH FROM o.created_at)` untuk memisahkan dan mengambil data bulan dari kolom created_at yang memiliki format datetime.
* year dengan query `EXTRACT(YEAR FROM o.created_at)` untuk memisahkan dan mengambil data tahun dari kolom created_at yang memiliki format datetime.
* product_id
* product_name
* total_item dengan menjumlahkan (`SUM`) semua num_of_item.
* total_revenue dengan menjumlahkan (`SUM`) perkalian dari num_of_item dan sale_price, kemudian membulatkannya (`ROUND`) dengan mengambil 2 angka di belakang koma.
* number_of_order dengan menghitung banyaknya (`COUNT`) pemesanan (order_id) yang unik (artinya, jika terdapat nilai pada order_id yang sama maka hanya salah satunya yang akan diambil)
* number_of_customer dengan menghitung banyaknya (`COUNT`) orang yang melakukan pesanan (user_id) yang unik.

2. Menggunakan query `FROM` sebagai referensi tabel yang digunakan dalam `SELECT`, yaitu `bigquery-public-data.thelook_ecommerce.order_items` alias `oi`.

3. Menggunakan query `JOIN` untuk menggabungkan tabel yang diperlukan:
* `JOIN` dengan tabel orders alias `o` menggunakan kunci order_id.
* `JOIN` dengan tabel products alias `p` menggunakan kunci product_id di tabel `oi` dan id di tabel `p`.

4. Memfilter hasil dengan `WHERE` untuk hanya menampilkan status yang bukan `Cancelled` atau `Returned`.

5. Mengelompokkan data (`GROUP BY`) berdasarkan: year, kemudian month, product_id, dan product_name.

Jadi, hasil dari blok inner query a ini menghasilkan suatu hasil sementara yang digunakan pada blok diluarnya, begitupun dengan hasil dari middle query b.


**Middle query `b`**
```sql
SELECT a.*, ROW_NUMBER() OVER(PARTITION BY a.year, a.month order by a.total_item desc) allmonth
    FROM
        ...
    ) b
```
Blok b memilih (`SELECT`) semua kolom dari blok `a`, kemudian menggunakan `ROW_NUMBER()` untuk memberikan nomor urut (`allmonth`) berdasarkan total_item secara descending dalam setiap partisi year dan month.

**Query terluar** 
```sql
SELECT month, year, product_id, product_name, total_item, total_revenue, number_of_order, number_of_customer
FROM
    ...
WHERE allmonth=1
ORDER BY year DESC, month DESC
```
1. Memilih (`SELECT`) kolom month, year, product_id, product_name, total_item, total_revenue, number_of_order, dan number_of_customer dari b dengan memfilter (`WHERE`) hanya baris di mana allmonth bernilai 1. Ini memastikan hanya baris teratas (dengan total_item terbanyak) untuk setiap kombinasi year dan month yang akan ditampilkan.

2. Mengurutkan (`ORDER BY`) hasil berdasarkan year dan month secara descending.
   

Note:

AS: Alias 

o.created_at: penggunaan `o` untuk merujuk pada table sumber kolom created_at berada







