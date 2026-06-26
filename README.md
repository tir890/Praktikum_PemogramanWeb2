# Lab 7: Web Programming - Modul 5: Pagination dan Pencarian

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. [cite_start]Modul ini berfokus pada implementasi fitur pembatasan halaman (Pagination) dan pencarian data artikel pada halaman admin[cite: 3, 9, 11, 70].

## 📌 Tujuan Praktikum
1. [cite_start]Memahami konsep dasar dan fungsi **Pagination** untuk membatasi tampilan data yang banyak[cite: 4, 12].
2. [cite_start]Memahami konsep dasar fitur **Pencarian** data[cite: 5, 71].
3. [cite_start]Mampu mengimplementasikan Paging dan Pencarian secara dinamis menggunakan library bawaan **CodeIgniter 4**[cite: 6, 14].

---

## 💻 Langkah-Langkah Praktikum

### 1. Membuat Pagination
[cite_start]Pagination digunakan untuk memecah tampilan data artikel yang banyak menjadi beberapa halaman (dalam praktikum ini dibatasi maksimal 10 record per halaman)[cite: 12, 13, 22].

* [cite_start]**Modifikasi Controller:** Buka file Controller `Artikel.php`, lalu sesuaikan method `admin_index()` menjadi seperti berikut[cite: 15, 16, 74]:

```php
public function admin_index()
{
    $title = 'Daftar Artikel';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'artikel' => $model->paginate(10), // Membatasi 10 record per halaman
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

* 
**Modifikasi View:** Buka file `views/artikel/admin_index.php` dan tambahkan kode pemanggil links pager tepat di bawah deklarasi tabel data:



```php
<?= $pager->links(); ?>

```

#### 📸 Hasil Pagination

Berikut adalah tampilan halaman admin setelah library pagination diterapkan (silakan tambahkan data artikel baru untuk menguji navigasi halaman):

---

### 2. Membuat Fitur Pencarian Data

Fitur pencarian berfungsi untuk menyaring atau memfilter data berdasarkan kata kunci (keyword) judul artikel yang diinputkan oleh pengguna.

* 
**Modifikasi Kembali Controller:** Ubah method `admin_index()` pada Controller `Artikel.php` agar dapat menangkap parameter pencarian `q` menggunakan query string `getVar()`:



```php
public function admin_index()
{
    $title = 'Daftar Artikel';
    $q = $this->request->getVar('q') ?? '';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'q'       => $q,
        'artikel' => $model->like('judul', $q)->paginate(10), // Memfilter data berdasarkan input 'q'
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

* 
**Menambahkan Form Pencarian pada View:** Buka kembali file `views/artikel/admin_index.php`, letakkan form pencarian ini tepat sebelum deklarasi tabel:



```html
<form method="get" class="form-search">
    <input type="text" name="q" value="<?= $q; ?>" placeholder="Cari data">
    <input type="submit" value="Cari" class="btn btn-primary">
</form>

```

* 
**Modifikasi Link Pager:** Ubah kode pemanggil links pager sebelumnya agar tetap mempertahankan parameter pencarian `q` saat berpindah halaman:



```php
<?= $pager->only(['q'])->links(); ?>

```

#### 📸 Hasil Pencarian Data

Berikut adalah tampilan halaman admin ketika melakukan pencarian data artikel tertentu:

---

## 📝 Kesimpulan

Dengan memanfaatkan library bawaan CodeIgniter 4, pembuatan fitur pagination dan pencarian menjadi jauh lebih efisien. Penggunaan method `$model->paginate()` secara otomatis menangani *offset* data, sementara `$pager->only(['q'])->links()` memastikan query pencarian tidak hilang saat user melakukan navigasi antar halaman (*paging*).

```
