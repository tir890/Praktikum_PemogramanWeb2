# Lab 7: Web Programming - Modul 5: Pagination dan Pencarian

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada implementasi fitur pembatasan halaman (Pagination) dan pencarian data artikel pada halaman admin.

## 📌 Tujuan Praktikum
1. Memahami konsep dasar dan fungsi **Pagination** untuk membatasi tampilan data yang banyak.
2. Memahami konsep dasar fitur **Pencarian** data.
3. Mampu membuat Paging dan Pencarian secara dinamis menggunakan library bawaan **CodeIgniter 4**.

---

## 💻 Langkah-Langkah Praktikum

### 1. Membuat Pagination
Pagination merupakan proses yang digunakan untuk membatasi tampilan data yang panjang dari database pada sebuah website. Pada praktikum ini, data artikel dibatasi maksimal 10 record per halaman.

* **Modifikasi Controller:** Buka file Controller `Artikel.php`, lalu sesuaikan method `admin_index()` menjadi seperti berikut:

```php
public function admin_index()
{
    $title = 'Daftar Artikel';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'artikel' => $model->paginate(10), // data dibatasi 10 record per halaman
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

* **Modifikasi View:** Buka file `views/artikel/admin_index.php` dan tambahkan kode pemanggil links pager tepat di bawah deklarasi tabel data:

```php
<?= $pager->links(); ?>

```

#### 📸 Hasil Pagination

Berikut adalah tampilan halaman admin setelah fungsi pagination diterapkan (tambahkan data baru untuk melihat navigasi halaman):

---

### 2. Membuat Fitur Pencarian Data

Pencarian data digunakan untuk menyaring atau memfilter data artikel berdasarkan kata kunci tertentu yang diinputkan oleh user.

* **Modifikasi Kembali Controller:** Ubah method `admin_index()` pada Controller `Artikel.php` agar dapat menangkap query pencarian `q` dan menyaring data menggunakan method `like()`:

```php
public function admin_index()
{
    $title = 'Daftar Artikel';
    $q = $this->request->getVar('q') ?? '';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'q'       => $q,
        'artikel' => $model->like('judul', $q)->paginate(10), // data dibatasi 10 record per halaman
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

* **Menambahkan Form Pencarian pada View:** Buka kembali file `views/artikel/admin_index.php`, lalu tambahkan form pencarian berikut tepat sebelum deklarasi tabel:

```html
<form method="get" class="form-search">
    <input type="text" name="q" value="<?= $q; ?>" placeholder="Cari data">
    <input type="submit" value="Cari" class="btn btn-primary">
</form>

```

* **Modifikasi Link Pager:** Ubah kode pemanggil links pager sebelumnya agar tetap mempertahankan parameter pencarian `q` saat berpindah halaman:

```php
<?= $pager->only(['q'])->links(); ?>

```

#### 📸 Hasil Pencarian Data

Berikut adalah tampilan halaman admin ketika melakukan pencarian data artikel berdasarkan kata kunci:

---

## 📝 Kesimpulan

Dengan memanfaatkan library bawaan CodeIgniter 4, pembuatan fitur pagination dan pencarian menjadi jauh lebih efisien. Penggunaan method `$model->paginate()` secara otomatis mengontrol *offset* data, sementara `$pager->only(['q'])->links()` memastikan query pencarian tetap aktif dan tidak hilang saat user melakukan navigasi antar halaman (*paging*).

```

```

# Lab 7: Web Programming - Modul 6: Login dan Autentikasi

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada pembuatan fitur autentikasi pengguna (Login) guna membatasi hak akses pada halaman administrasi.

## 📌 Tujuan Praktikum
1. Memahami konsep dasar **Autentikasi** dan manajemen session pada web aplikasi.
2. Mampu membuat halaman login serta memproses data autentikasi user dari database.
3. Mampu membatasi akses halaman admin menggunakan mekanisme pengecekan session.

---

## 💻 Langkah-Langkah Praktikum

### 1. Membuat Tabel User
Fitur autentikasi membutuhkan tabel pada database untuk menyimpan data akun pengguna (username dan password).

* Buat tabel baru bernama `user` menggunakan query SQL berikut pada database kamu:

```sql
CREATE TABLE user (
    id INT(11) NOT NULL AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    password VARCHAR(250) NOT NULL,
    PRIMARY KEY (id)
);

```

* Tambahkan satu data user uji coba ke dalam tabel tersebut. Pastikan password dienkripsi menggunakan fungsi `password_hash()` pada PHP (atau menggunakan MD5/SHA jika disesuaikan dengan instruksi praktikum).

---

### 2. Membuat Model User

Model ini berfungsi untuk melakukan query ke tabel `user` guna mencocokkan data login yang diinput oleh pengguna.

* Buat file baru bernama `UserModel.php` di dalam folder `app/Models/` dan masukkan kode berikut:

```php
<?php
namespace App\Models;
use CodeIgniter\Model;

class UserModel extends Model
{
    protected $table = 'user';
    protected $primaryKey = 'id';
    protected $allowedFields = ['username', 'password'];
}

```

---

### 3. Membuat Controller User

Controller ini akan menangani proses penampilan form login, validasi input, pencocokan data akun ke database, serta penghapusan session saat user logout.

* Buat file baru bernama `User.php` di dalam folder `app/Controllers/` dan tambahkan kode berikut:

```php
<?php
namespace App\Controllers;
use App\Models\UserModel;

class User extends BaseController
{
    public function login()
    {
        $session = session();
        $model = new UserModel();
        $username = $this->request->getVar('username');
        $password = $this->request->getVar('password');
        
        if ($username) {
            $data = $model->where('username', $username)->first();
            if ($data) {
                $pass = $data['password'];
                // Sesuaikan enkripsi password (contoh: password_verify atau md5)
                $verify_pass = ($password === $pass); 
                
                if ($verify_pass) {
                    $ses_data = [
                        'id'       => $data['id'],
                        'username' => $data['username'],
                        'logged_in'=> TRUE
                    ];
                    $session->set($ses_data);
                    return redirect()->to('/admin/artikel');
                } else {
                    $session->setFlashdata('msg', 'Password Salah');
                    return redirect()->to('/user/login');
                }
            } else {
                $session->setFlashdata('msg', 'Username Tidak Ditemukan');
                return redirect()->to('/user/login');
            }
        }
        return view('user/login');
    }

    public function logout()
    {
        $session = session();
        $session->destroy();
        return redirect()->to('/user/login');
    }
}

```

---

### 4. Membuat View Login

View ini berfungsi untuk menampilkan form input bagi user yang ingin masuk ke dalam sistem admin.

* Buat folder baru bernama `user` di dalam `app/Views/`, lalu buat file `login.php` di dalamnya:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login Portal Berita</title>
    <link rel="stylesheet" href="/css/style.css"> </head>
<body>
    <div id="login-wrapper">
        <h1>Sign In</h1>
        <?php if(session()->getFlashdata('msg')):?>
            <div class="alert alert-danger"><?= session()->getFlashdata('msg') ?></div>
        <?php endif;?>
        <form action="/user/login" method="post">
            <div class="mb-3">
                <label for="InputForEmail" class="form-label">Username</label>
                <input type="text" name="username" class="form-control" id="InputForEmail" value="<?= set_value('username') ?>">
            </div>
            <div class="mb-3">
                <label for="InputForPassword" class="form-label">Password</label>
                <input type="password" name="password" class="form-control" id="InputForPassword">
            </div>
            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</body>
</html>

```

#### 📸 Hasil Tampilan Halaman Login

Berikut adalah penampakan halaman login setelah dikonfigurasi:

---

### 5. Membatasi Akses Halaman Admin

Agar halaman admin tidak bisa diakses langsung via URL tanpa login terlebih dahulu, kita harus menambahkan pengecekan session di awal method-method admin pada Controller `Artikel.php`.

* Buka kembali `app/Controllers/Artikel.php`, kemudian tambahkan validasi session pada method `admin_index()` (atau constructor jika ingin memproteksi seluruh isi class):

```php
public function admin_index()
{
    // Cek apakah user sudah login atau belum
    if (!session()->get('logged_in')) {
        return redirect()->to('/user/login');
    }

    $title = 'Daftar Artikel';
    $q = $this->request->getVar('q') ?? '';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'q'       => $q,
        'artikel' => $model->like('judul', $q)->paginate(10),
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

---

## 📝 Kesimpulan

Proses autentikasi pada CodeIgniter 4 dapat dikelola dengan mudah menggunakan **Session Library**. Dengan menyimpan status `logged_in => TRUE` pada session saat pencocokan data database sukses, kita dapat menyaring hak akses halaman krusial (seperti dashboard admin) dan me-redirect user anonim kembali ke form login demi menjaga keamanan data aplikasi.

```

---

### 💡 Langkah Selanjutnya:
1. Simpan kode di atas ke file `README.md` pada repositorimu (bisa digabungkan atau diletakkan di dalam subfolder Modul 6).
2. Ambil *screenshot* halaman loginmu, simpan dengan nama `login_page.png` di dalam folder `screenshot`.
3. Jika sudah siap, silakan unggah berkas **Modul 7**!

```

# Lab 7: Web Programming - Modul 7: Kelanjutan Fitur Login dan Autentikasi

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada penyempurnaan fitur login, penanganan validasi form login, serta manajemen session untuk proteksi halaman admin.

## 📌 Tujuan Praktikum
1. Memahami alur pengamanan halaman web menggunakan session secara mendalam.
2. Mampu membuat sistem validasi masukan (input validation) pada form login.
3. Mengimplementasikan fungsi pembatasan hak akses (hak administrator) dan fitur logout.

---

## 💻 Langkah-Langkah Praktikum

### 1. Membuat Halaman Login (Views)
Halaman login dibuat untuk memfasilitasi pengguna dalam memasukkan *username* dan *password* mereka.

* **Membuat File View:** Buat atau pastikan file `login.php` berada di dalam folder `app/Views/user/` dengan kode sebagai berikut:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Login Portal Berita</title>
    <link rel="stylesheet" href="/css/style.css">
</head>
<body>
    <div id="login-wrapper">
        <h1>Sign In</h1>
        
        <?php if(session()->getFlashdata('msg')):?>
            <div class="alert alert-danger">
                <?= session()->getFlashdata('msg') ?>
            </div>
        <?php endif;?>
        
        <form action="/user/login" method="post">
            <div class="mb-3">
                <label for="InputForEmail" class="form-label">Username</label>
                <input type="text" name="username" class="form-control" id="InputForEmail" value="<?= set_value('username') ?>">
            </div>
            <div class="mb-3">
                <label for="InputForPassword" class="form-label">Password</label>
                <input type="password" name="password" class="form-control" id="InputForPassword">
            </div>
            <button type="submit" class="btn btn-primary">Login</button>
        </form>
    </div>
</body>
</html>

```

---

### 2. Membuat Controller User & Proses Autentikasi

Controller ini berfungsi memproses data kiriman dari form login, memeriksa kecocokan data ke model, dan mengeset data session jika autentikasi berhasil.

* **Membuat/Modifikasi Controller:** Pastikan file `User.php` pada folder `app/Controllers/` berisi logika pengecekan password dan pengaturan session berikut:

```php
<?php
namespace App\Controllers;
use App\Models\UserModel;

class User extends BaseController
{
    public function login()
    {
        $session = session();
        $model = new UserModel();
        $username = $this->request->getVar('username');
        $password = $this->request->getVar('password');
        
        if ($username) {
            $data = $model->where('username', $username)->first();
            if ($data) {
                $pass = $data['password'];
                
                // Melakukan verifikasi password (disesuaikan dengan skema enkripsi database)
                $verify_pass = ($password === $pass);
                
                if ($verify_pass) {
                    $ses_data = [
                        'id'       => $data['id'],
                        'username' => $data['username'],
                        'logged_in'=> TRUE
                    ];
                    $session->set($ses_data);
                    return redirect()->to('/admin/artikel');
                } else {
                    $session->setFlashdata('msg', 'Password Salah');
                    return redirect()->to('/user/login');
                }
            } else {
                $session->setFlashdata('msg', 'Username Tidak Ditemukan');
                return redirect()->to('/user/login');
            }
        }
        return view('user/login');
    }

    // Fungsi untuk menghapus session saat user keluar sistem
    public function logout()
    {
        $session = session();
        $session->destroy();
        return redirect()->to('/user/login');
    }
}

```

#### 📸 Hasil Tampilan Login & Validasi Error

Berikut adalah tampilan antarmuka form login serta penanganan kondisi saat data login yang dimasukkan tidak valid:

---

### 3. Mengamankan Controller Admin (Artikel)

Untuk mencegah akses ilegal langsung melalui pengetikan URL di browser, tambahkan pemeriksaan status `logged_in` pada method admin di Controller `Artikel.php`.

* **Implementasi Proteksi Session:**

```php
public function admin_index()
{
    // Memeriksa status login user
    if (!session()->get('logged_in')) {
        return redirect()->to('/user/login');
    }

    $title = 'Daftar Artikel';
    $q = $this->request->getVar('q') ?? '';
    $model = new ArtikelModel();
    
    $data = [
        'title'   => $title,
        'q'       => $q,
        'artikel' => $model->like('judul', $q)->paginate(10),
        'pager'   => $model->pager,
    ];
    
    return view('artikel/admin_index', $data);
}

```

---

## 📝 Kesimpulan

Melalui praktikum di Modul 7 ini, sistem autentikasi diperkuat dengan pemanfaatan **Flashdata Session** untuk mengirimkan feedback error secara langsung kepada pengguna jika proses login gagal. Validasi status session di sisi Controller memastikan bahwa modul-modul administrasi data artikel tetap aman dari akses pengguna luar yang belum terautentikasi.

---

# Lab 7: Web Programming - Modul 8: Asynchronous JavaScript and XML (AJAX)

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada implementasi teknologi AJAX untuk memuat dan memanipulasi data artikel secara dinamis tanpa melakukan reload halaman keseluruhan.

## 📌 Tujuan Praktikum
1. Memahami konsep dasar AJAX (Asynchronous JavaScript and XML) dan mekanisme cara kerjanya.
2. Mampu mengimplementasikan pemanggilan data dari server menggunakan library jQuery AJAX pada CodeIgniter 4.
3. Mengembangkan fungsi manipulasi data (seperti hapus data) secara asynchronous dan menangani respons berformat JSON.

---

## 💻 Langkah-Langkah Praktikum

### 1. Persiapan & Menambahkan Pustaka jQuery
Praktikum ini menggunakan library **jQuery** untuk mempermudah penulisan sintaks penanganan AJAX.

* Unduh pustaka jQuery versi terbaru melalui situs resminya ([jquery.com](https://jquery.com)).
* Letakkan atau salin berkas `jquery-3.6.0.min.js` (atau versi yang kamu gunakan) ke dalam direktori project kamu pada folder:
  `public/assets/js/jquery-3.6.0.min.js`

---

### 2. Membuat AJAX Controller
Controller baru dibuat khusus untuk menangani request asynchronous dan mengembalikan data dalam bentuk format JSON, bukan file view HTML utuh.

* Buat file baru bernama `AjaxController.php` di dalam folder `app/Controllers/` dan masukkan kode berikut:

```php
<?php 
namespace App\Controllers;

use CodeIgniter\Controller;
use CodeIgniter\HTTP\Request;
use CodeIgniter\HTTP\Response;
use App\Models\ArtikelModel;

class AjaxController extends Controller 
{
    public function index() 
    {
        return view('ajax/index');
    }

    // Mengambil semua data artikel dan mengembalikannya dalam format JSON
    public function getData() 
    {
        $model = new ArtikelModel();
        $data = $model->findAll();
        return $this->response->setJSON($data);
    }

    // Menghapus data artikel berdasarkan ID melalui request AJAX
    public function delete($id) 
    {
        $model = new ArtikelModel();
        $model->delete($id);
        
        $data = [
            'status' => 'OK'
        ];
        return $this->response->setJSON($data);
    }
}

```

---

### 3. Membuat View AJAX (Menampilkan & Menghapus Data)

Halaman view ini memuat tabel kosong yang kemudian diisi secara dinamis oleh JavaScript/jQuery setelah data berhasil ditarik dari server melalui *endpoint* `getData()`.

* Buat folder baru bernama `ajax` di dalam `app/Views/`, lalu buat file `index.php` di dalamnya:

```html
<?= $this->include('template/header'); ?>

<h1>Data Artikel (Fitur AJAX)</h1>
<table class="table-data" id="artikelTable">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        </tbody>
</table>

<script src="<?= base_url('assets/js/jquery-3.6.0.min.js') ?>"></script>

<script>
$(document).ready(function() {
    
    // Fungsi untuk memunculkan pesan loading selama proses fetch data berlangsung
    function showLoadingMessage() {
        $('#artikelTable tbody').html('<tr><td colspan="4">Loading data...</td></tr>');
    }

    // Fungsi utama untuk memuat data artikel secara asynchronous
    function loadData() {
        showLoadingMessage();
        
        $.ajax({
            url: "<?= base_url('ajax/getData') ?>",
            method: "GET",
            dataType: "json",
            success: function(data) {
                var tableBody = "";
                for (var i = 0; i < data.length; i++) {
                    var row = data[i];
                    tableBody += '<tr>';
                    tableBody += '<td>' + row.id + '</td>';
                    tableBody += '<td>' + row.judul + '</td>';
                    tableBody += '<td><span class="status">---</span></td>'; // Placeholder status
                    tableBody += '<td>';
                    tableBody += '<a href="<?= base_url('artikel/edit/') ?>' + row.id + '" class="btn btn-primary">Edit</a> ';
                    tableBody += '<a href="#" class="btn btn-danger btn-delete" data-id="' + row.id + '">Delete</a>';
                    tableBody += '</td>';
                    tableBody += '</tr>';
                }
                $('#artikelTable tbody').html(tableBody);
            }
        });
    }

    // Panggil fungsi loadData saat pertama kali halaman dibuka
    loadData();

    // Logika event trigger untuk menghapus artikel secara asynchronous
    $(document).on('click', '.btn-delete', function(e) {
        e.preventDefault();
        var id = $(this).data('id');
        
        if (confirm('Apakah Anda yakin ingin menghapus artikel ini?')) {
            $.ajax({
                url: "<?= base_url('ajax/delete/') ?>" + id,
                method: "DELETE",
                success: function(data) {
                    // Muat ulang data tabel tanpa harus reload halaman browser
                    loadData(); 
                },
                error: function (jqXHR, textStatus, errorThrown) {
                    alert('Error ketika menghapus artikel: ' + textStatus + ' - ' + errorThrown);
                }
            });
        }
    });
});
</script>

<?= $this->include('template/footer'); ?>

```

#### 📸 Hasil Pengujian Fitur AJAX

Berikut adalah tampilan halaman artikel ketika dimuat menggunakan mekanisme penanganan data berbasis AJAX:

---

## 📝 Kesimpulan

Penerapan **AJAX (Asynchronous JavaScript and XML)** di CodeIgniter 4 secara signifikan mampu meningkatkan *User Experience* (UX) karena pembaruan konten tabel dan aksi hapus data terjadi di latar belakang (*background*) tanpa memaksa browser memuat ulang (*reload*) keseluruhan halaman. Penanganan data dari server dijembatani menggunakan representasi objek data terstruktur berformat **JSON** melalui method `$this->response->setJSON()`.


# Lab 7: Web Programming - Modul 9: Implementasi AJAX Pagination dan Search

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada implementasi AJAX untuk menangani fitur perpindahan halaman (Pagination) dan pencarian data (Search) artikel yang terintegrasi dengan filter kategori secara dinamis tanpa muat ulang (*reload*) halaman.

## 📌 Tujuan Praktikum
1. Memahami konsep dasar manipulasi komponen Pagination dan Search menggunakan Request AJAX.
2. Mampu mengonstruksi payload respons berformat JSON yang membawa data relasional, tautan paging, dan parameter pencarian pada CodeIgniter 4.
3. Meningkatkan performa aplikasi serta kualitas *User Experience* (UX) melalui pemrosesan data asinkron.

---

## 💻 Langkah-Langkah Praktikum

### 1. Modifikasi Controller Artikel
Kita mengubah method `admin_index()` pada Controller `Artikel.php` agar mampu mendeteksi jenis request. Jika request dikirim via AJAX, server akan merespons dengan data JSON yang berisi daftar artikel hasil filter/pencarian beserta konfigurasi tautan paginasinya.

* Buka file `app/Controllers/Artikel.php` dan sesuaikan method `admin_index()` menjadi seperti berikut:

```php
public function admin_index()
{
    $title = 'Daftar Artikel (Admin)';
    $model = new ArtikelModel();
    
    // Menangkap parameter input query string
    $q = $this->request->getVar('q') ?? '';
    $kategori_id = $this->request->getVar('kategori_id') ?? '';
    $page = $this->request->getVar('page') ?? 1;
    
    // Membangun query builder dengan join tabel kategori
    $builder = $model->table('artikel')
                     ->select('artikel.*, kategori.nama_kategori')
                     ->join('kategori', 'kategori.id_kategori = artikel.id_kategori');
    
    // Filter pencarian berdasarkan kata kunci judul jika diinputkan
    if ($q != '') {
        $builder->like('artikel.judul', $q);
    }
    
    // Filter berdasarkan kategori jika dipilih
    if ($kategori_id != '') {
        $builder->where('artikel.id_kategori', $kategori_id);
    }
    
    // Menangani kembalian data berformat JSON khusus untuk request AJAX
    if ($this->request->isAJAX()) {
        $data = [
            'artikel' => $builder->paginate(5, 'default', $page),
            'pager'   => $model->pager->links('default', 'bootstrap_full') // Menggunakan template link bootstrap
        ];
        return $this->response->setJSON($data);
    }
    
    // Mengambil data kategori untuk dikirim ke view pertama kali
    $kategoriModel = new \App\Models\KategoriModel(); 
    $data = [
        'title'    => $title,
        'kategori' => $kategoriModel->findAll()
    ];
    
    return view('artikel/admin_index', $data);
}

```

---

### 2. Modifikasi View Halaman Admin

Ubah struktur tabel pada halaman view agar isi body tabel (`<tbody>`) dan wadah navigasi halaman (*pagination container*) di-render secara dinamis menggunakan jQuery setelah menerima data JSON dari server.

* Buka file `app/Views/artikel/admin_index.php` dan sesuaikan strukturnya menjadi seperti berikut:

```html
<?= $this->include('template/admin_header'); ?>

<h1><?= $title; ?></h1>

<form id="searchForm" class="form-inline mb-3">
    <input type="text" id="searchBox" class="form-control mr-2" placeholder="Cari judul artikel...">
    
    <select id="categoryFilter" class="form-control mr-2">
        <option value="">-- Semua Kategori --</option>
        <?php foreach($kategori as $k): ?>
            <option value="<?= $k['id_kategori']; ?>"><?= $k['nama_kategori']; ?></option>
        <?php endforeach; ?>
    </select>
    
    <button type="submit" class="btn btn-primary">Cari</button>
</form>

<table class="table-data" id="artikelTable">
    <thead>
        <tr>
            <th>ID</th>
            <th>Judul</th>
            <th>Kategori</th>
            <th>Status</th>
            <th>Aksi</th>
        </tr>
    </thead>
    <tbody>
        </tbody>
</table>

<div id="paginationContainer" class="mt-3"></div>

<script src="<?= base_url('assets/js/jquery-3.6.0.min.js') ?>"></script>
<script>
$(document).ready(function() {
    const tableBody = $('#artikelTable tbody');
    const paginationContainer = $('#paginationContainer');
    const searchForm = $('#searchForm');
    const searchBox = $('#searchBox');
    const categoryFilter = $('#categoryFilter');

    // Fungsi utama untuk melakukan fetch data dari server
    function fetchData(url) {
        tableBody.html('<tr><td colspan="5">Sedang memuat data...</td></tr>');
        
        $.ajax({
            url: url,
            method: 'GET',
            dataType: 'json',
            success: function(response) {
                // 1. Render data tabel artikel
                let tableHtml = '';
                if(response.artikel.length > 0) {
                    response.artikel.forEach(row => {
                        tableHtml += '<tr>';
                        tableHtml += '<td>' + row.id + '</td>';
                        tableHtml += '<td>' + row.judul + '</td>';
                        tableHtml += '<td>' + (row.nama_kategori ? row.nama_kategori : 'Tanpa Kategori') + '</td>';
                        tableHtml += '<td>' + row.status + '</td>';
                        tableHtml += '<td>';
                        tableHtml += '<a href="<?= base_url('admin/artikel/edit/') ?>' + row.id + '" class="btn btn-primary">Ubah</a> ';
                        tableHtml += '<a href="<?= base_url('admin/artikel/delete/') ?>' + row.id + '" class="btn btn-danger btn-delete" data-id="'+row.id+'">Hapus</a>';
                        tableHtml += '</td>';
                        tableHtml += '</tr>';
                    });
                } else {
                    tableHtml = '<tr><td colspan="5">Data tidak ditemukan.</td></tr>';
                }
                tableBody.html(tableHtml);

                // 2. Render komponen links pagination dinamis
                paginationContainer.html(response.pager);
            },
            error: function() {
                tableBody.html('<tr><td colspan="5">Gagal memuat data dari server.</td></tr>');
            }
        });
    }

    // Intersepsi submit form pencarian
    searchForm.on('submit', function(e) {
        e.preventDefault();
        const q = searchBox.val();
        const kategori_id = categoryFilter.val();
        fetchData(`<?= base_url('admin/artikel') ?>?q=${q}&kategori_id=${kategori_id}`);
    });

    // Otomatis melakukan pencarian saat filter kategori diubah
    categoryFilter.on('change', function() {
        searchForm.trigger('submit');
    });

    // Intersepsi klik pada tautan pagination agar link berpindah secara asynchronous
    $(document).on('click', '#paginationContainer a', function(e) {
        e.preventDefault();
        const url = $(this).attr('href');
        if (url && url !== '#') {
            fetchData(url);
        }
    });

    // Initial Load: Memuat data pertama kali saat halaman dibuka
    fetchData('<?= base_url('admin/artikel') ?>');
});
</script>

<?= $this->include('template/admin_footer'); ?>

```

#### 📸 Hasil Pengujian AJAX Pagination & Search

Berikut adalah tampilan halaman administrasi artikel yang mengintegrasikan pencarian kata kunci, penyaringan kategori, dan navigasi halaman berbasis teknologi AJAX:

---

## 📝 Kesimpulan

Melalui praktikum Modul 9 ini, fitur pencarian dan pagination ditingkatkan fungsionalitasnya dengan menggabungkan query relasional (`JOIN`) antara tabel `artikel` dan `kategori`. Dengan memanfaatkan penanganan event klik tautan (`event.preventDefault()`) dan manipulasi DOM menggunakan jQuery, transisi perpindahan halaman serta penyaringan data dapat berjalan mulus tanpa interupsi reload browser penuh.


# Lab 7: Web Programming - Modul 10: Pembuatan RESTful API

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework CodeIgniter 4** pada folder `lab7_php_ci`. Modul ini berfokus pada implementasi RESTful API untuk menyediakan *resource* data artikel yang dapat dikonsumsi oleh aplikasi atau platform lain.

## 📌 Tujuan Praktikum
1. Memahami konsep dasar Application Programming Interface (API) dan arsitektur RESTful.
2. Mampu membuat REST Server untuk melayani HTTP request (GET, POST, PUT, DELETE) menggunakan CodeIgniter 4.
3. Mampu melakukan pengujian *endpoint* API menggunakan aplikasi REST Client (seperti Postman).

---

## 💻 Langkah-Langkah Praktikum

### 1. Membuat Controller REST API
CodeIgniter 4 menyediakan `ResponseTrait` untuk mempermudah pengembalian respons berformat JSON/XML dengan kode status HTTP yang sesuai.

* Buat file baru bernama `Post.php` di dalam folder `app/Controllers/` dan masukkan kode berikut:

```php
<?php
namespace App\Controllers;

use CodeIgniter\RESTful\ResourceController;
use CodeIgniter\API\ResponseTrait;
use App\Models\ArtikelModel;

class Post extends ResourceController
{
    use ResponseTrait;

    // GET: Menampilkan semua data artikel
    public function index()
    {
        $model = new ArtikelModel();
        $data = $model->findAll();
        return $this->respond($data, 200);
    }

    // GET: Menampilkan detail artikel berdasarkan ID
    public function show($id = null)
    {
        $model = new ArtikelModel();
        $data = $model->getWhere(['id' => $id])->getResult();
        
        if ($data) {
            return $this->respond($data, 200);
        } else {
            return $this->failNotFound('Data artikel tidak ditemukan untuk ID: ' . $id);
        }
    }

    // POST: Menambah data artikel baru
    public function create()
    {
        $model = new ArtikelModel();
        $data = [
            'judul' => $this->request->getVar('judul'),
            'isi'   => $this->request->getVar('isi'),
            'slug'  => url_title($this->request->getVar('judul'), '-', true)
        ];
        
        $model->insert($data);
        $response = [
            'status'   => 201,
            'error'    => null,
            'messages' => [
                'success' => 'Data artikel berhasil ditambahkan.'
            ]
        ];
        return $this->respondCreated($response);
    }

    // PUT/PATCH: Memperbarui data artikel berdasarkan ID
    public function update($id = null)
    {
        $model = new ArtikelModel();
        $json = $this->request->getJSON();
        
        if ($json) {
            $data = [
                'judul' => $json->judul,
                'isi'   => $json->isi,
                'slug'  => url_title($json->judul, '-', true)
            ];
        } else {
            $data = [
                'judul' => $this->request->getRawInput()['judul'],
                'isi'   => $this->request->getRawInput()['isi'],
                'slug'  => url_title($this->request->getRawInput()['judul'], '-', true)
            ];
        }

        $model->update($id, $data);
        $response = [
            'status'   => 200,
            'error'    => null,
            'messages' => [
                'success' => 'Data artikel berhasil diperbarui.'
            ]
        ];
        return $this->respond($response);
    }

    // DELETE: Menghapus data artikel berdasarkan ID
    public function delete($id = null)
    {
        $model = new ArtikelModel();
        $data = $model->find($id);
        
        if ($data) {
            $model->delete($id);
            $response = [
                'status'   => 200,
                'error'    => null,
                'messages' => [
                    'success' => 'Data artikel berhasil dihapus.'
                ]
            ];
            return $this->respondDeleted($response);
        } else {
            return $this->failNotFound('Data tidak ditemukan untuk ID: ' . $id);
        }
    }
}

```

---

### 2. Konfigurasi Routing RESTful API

Untuk memetakan request HTTP secara otomatis ke dalam method CRUD di controller `Post.php`, kita daftarkan *resource* routes pada konfigurasi aplikasi.

* Buka file `app/Config/Routes.php` dan tambahkan baris kode berikut:

```php
$routes->resource('post');

```

*Dengan sintaks ini, URL `http://localhost:8080/post` otomatis mendukung method GET, POST, PUT, dan DELETE.*

---

### 3. Pengujian RESTful API menggunakan Postman

Pengujian dilakukan untuk mensimulasikan permintaan dari client aplikasi lain:

* **Mendapatkan Semua Data (GET):** Akses URL `http://localhost:8080/post` dengan method `GET`. Server akan mengembalikan seluruh data dalam struktur JSON dengan status `200 OK`.
* **Menambahkan Data Baru (POST):** Akses URL `http://localhost:8080/post` dengan method `POST`. Isikan parameter `judul` dan `isi` pada bagian body request. Server akan merespons dengan status `201 Created`.
* **Mengubah Data (PUT):** Akses URL data spesifik (misal ID 4): `http://localhost:8080/post/4` dengan method `PUT`. Kirimkan data baru melalui raw JSON atau x-www-form-urlencoded untuk memperbarui data artikel.
* **Menghapus Data (DELETE):** Akses URL data spesifik (misal ID 4): `http://localhost:8080/post/4` menggunakan method `DELETE`. Jika berhasil, server mengembalikan pesan sukses dan status `200 OK`.

#### 📸 Hasil Pengujian API

Berikut adalah contoh tangkapan layar pengujian penghapusan resource data melalui client API:

---

## 📝 Kesimpulan

Penyediaan arsitektur **RESTful API** pada CodeIgniter 4 dapat diimplementasikan secara instan berkat tersedianya class `ResourceController` dan `ResponseTrait`. Format keluaran berupa data terstruktur **JSON** dengan kode status HTTP (*HTTP Status Codes*) standar memastikan bahwa sistem backend ini siap berinteraksi dan berintegrasi secara aman dengan berbagai macam platform frontend independent maupun aplikasi mobile.


# Lab 8: Web Programming - Modul 11: Frontend API menggunakan VueJS 3

Repository ini merupakan bagian dari praktikum pemrograman web pada folder `lab8_vuejs`. Modul ini berfokus pada pembuatan aplikasi sisi depan (*Frontend*) berbasis komponen menggunakan **Framework VueJS 3** untuk mengonsumsi, menampilkan, dan memanipulasi data artikel dari RESTful API backend (CodeIgniter 4).

## 📌 Tujuan Praktikum
1. Memahami konsep dasar arsitektur Single Page Application (SPA) dan Framework VueJS.
2. Mampu melakukan integrasi data dari REST Server ke dalam View-Layer secara reaktif.
3. Mampu membuat komponen antarmuka interaktif seperti penanganan Form Input dan Modal Pop-up menggunakan VueJS.

---

## 💻 Langkah-Langkah Praktikum

### 1. Struktur Proyek Frontend
Praktikum ini dibuat secara terpisah dari folder backend di dalam folder khusus bernama `lab8_vuejs`. Di dalamnya, fungsionalitas diintegrasikan langsung menggunakan CDN VueJS untuk kesederhanaan implementasi.

---

### 2. Membuat Halaman Utama dan Integrasi VueJS
Halaman ini bertindak sebagai kerangka utama tempat aplikasi Vue di-mount (`#app`) serta memuat pustaka JavaScript yang diperlukan.

* Buat file bernama `index.html` pada folder proyek `lab8_vuejs` dan tambahkan struktur dasar berikut:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Dashboard Artikel - VueJS Frontend</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <header>
            <h1>Daftar Artikel</h1>
            <button @click="showModal = true" class="btn btn-primary">Tambah Data</button>
        </header>

        <table class="table-data">
            <thead>
                <tr>
                    <th>ID</th>
                    <th>Judul</th>
                    <th>Status</th>
                    <th>Aksi</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="row in artikel" :key="row.id">
                    <td>{{ row.id }}</td>
                    <td>{{ row.judul }}</td>
                    <td><span class="status-badge" :class="row.status">{{ row.status }}</span></td>
                    <td>
                        <button class="btn btn-danger" @click="deleteArtikel(row.id)">Hapus</button>
                    </td>
                </tr>
                <tr v-if="artikel.length === 0">
                    <td colspan="4">Tidak ada data artikel atau server mati.</td>
                </tr>
            </tbody>
        </table>

        <div v-if="showModal" class="modal">
            <div class="modal-content">
                <span class="close" @click="closeModal">&times;</span>
                <h2>Tambah Data Artikel</h2>
                <form @submit.prevent="saveArtikel">
                    <p>
                        <label>Judul</label>
                        <input type="text" v-model="form.judul" required placeholder="Masukkan judul artikel">
                    </p>
                    <p>
                        <label>Isi Artikel</label>
                        <textarea v-model="form.isi" required placeholder="Masukkan isi konten"></textarea>
                    </p>
                    <p>
                        <button type="submit" class="btn btn-success">Simpan</button>
                        <button type="button" class="btn btn-secondary" @click="closeModal">Batal</button>
                    </p>
                </form>
            </div>
        </div>
    </div>

    <script src="[https://unpkg.com/vue@3/dist/vue.global.js](https://unpkg.com/vue@3/dist/vue.global.js)"></script>
    <script>
        const { createApp } = Vue;

        createApp({
            data() {
                return {
                    artikel: [],
                    showModal: false,
                    // Endpoint URL API dari backend CodeIgniter 4
                    apiUrl: 'http://localhost:8080/post', 
                    form: {
                        judul: '',
                        isi: ''
                    }
                }
            },
            methods: {
                // Method untuk mengambil data dari REST API
                fetchData() {
                    fetch(this.apiUrl)
                        .then(response => response.json())
                        .then(data => {
                            this.artikel = data;
                        })
                        .catch(error => {
                            console.error("Gagal memuat API:", error);
                        });
                },
                // Method untuk menyimpan data baru via POST request
                saveArtikel() {
                    fetch(this.apiUrl, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/x-www-form-urlencoded'
                        },
                        body: new URLSearchParams(this.form)
                    })
                    .then(response => response.json())
                    .then(data => {
                        this.fetchData(); // Refresh list data
                        this.closeModal(); // Tutup modal
                    })
                    .catch(error => console.error("Gagal menyimpan data:", error));
                },
                // Method untuk menghapus data via DELETE request
                deleteArtikel(id) {
                    if (confirm("Apakah Anda yakin ingin menghapus data ini?")) {
                        fetch(`${this.apiUrl}/${id}`, {
                            method: 'DELETE'
                        })
                        .then(response => response.json())
                        .then(() => {
                            this.fetchData(); // Refresh list data
                        })
                        .catch(error => console.error("Gagal menghapus data:", error));
                    }
                },
                closeModal() {
                    this.showModal = false;
                    this.form.judul = '';
                    this.form.isi = '';
                }
            },
            mounted() {
                // Otomatis jalankan fetch data saat aplikasi siap
                this.fetchData(); 
            }
        }).mount('#app');
    </script>
</body>
</html>

```

---

### 3. Styling Modal dan Komponen (CSS)

Untuk membuat tampilan tabel rapi dan efek transparan *overlay* pada modal pop-up, tambahkan kode berikut ke dalam file `style.css`.

* Buat file bernama `style.css` pada folder yang sama:

```css
body {
    font-family: Arial, sans-serif;
    margin: 20px;
    background-color: #f9f9f9;
}
header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}
.table-data {
    width: 100%;
    border-collapse: collapse;
    background: #fff;
}
.table-data th, .table-data td {
    border: 1px solid #ddd;
    padding: 12px;
    text-align: left;
}
.table-data th {
    background-color: #f2f2f2;
}
.btn {
    padding: 8px 12px;
    border: none;
    cursor: pointer;
    border-radius: 4px;
}
.btn-primary { background: #007bff; color: white; }
.btn-success { background: #28a745; color: white; }
.btn-danger { background: #dc3545; color: white; }
.btn-secondary { background: #6c757d; color: white; }

/* Styling Komponen Modal */
.modal {
    display: block; 
    position: fixed;
    z-index: 1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.4);
}
.modal-content {
    background-color: #fefefe;
    margin: 15% auto;
    padding: 20px;
    border: 1px solid #888;
    width: 500px;
    border-radius: 5px;
}
.close {
    color: #aaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
    cursor: pointer;
}
.close:hover { color: black; }
input[type="text"], textarea {
    width: 100%;
    padding: 8px;
    margin-top: 5px;
    box-sizing: border-box;
}

```

#### 📸 Hasil Integrasi Frontend VueJS & Modal Tambah Data

Berikut adalah tampilan halaman depan dashboard yang dikelola secara penuh menggunakan VueJS ketika komponen form tambah data diaktifkan dalam bentuk modal pop-up:

---

## 📝 Kesimpulan

Melalui praktikum Modul 11 ini, pembuatan antarmuka aplikasi menjadi sangat interaktif berkat penerapan **VueJS 3**. Fitur *reactive data binding* (`v-model`) mempermudah sinkronisasi inputan form secara berkala, sementara direktif struktural seperti `v-for` dan `v-if` mengotomatisasi rendering list tabel data dan penataan kondisi modal *pop-up* secara instan langsung dari data JSON hasil request fungsi `fetch()`.

# Lab 8: Web Programming - Modul 12: VueJS Komponen dan Routing (Single Page Application)

Repository ini merupakan kelanjutan dari praktikum pemrograman web menggunakan **Framework VueJS 3** pada folder `lab8_vuejs`. Modul ini berfokus pada pemecahan kode antarmuka menjadi komponen modular yang dapat digunakan kembali (*reusable*) serta implementasi *Client-Side Routing* menggunakan **Vue Router** untuk membangun *Single Page Application* (SPA).

## 📌 Tujuan Praktikum
1. Memahami konsep komponen modular pada Framework VueJS.
2. Memahami konsep dasar *Client-Side Routing* untuk menghindari *hard-reload* halaman pada browser.
3. Mampu mengimplementasikan integrasi Vue Router berbasis CDN pada aplikasi Frontend API.

---

## 💻 Langkah-Langkah Praktikum

### 1. Struktur Komponen Modular (JavaScript)
Untuk menjaga kebersihan kode, tampilan antarmuka dipecah menjadi berkas komponen terisolasi di dalam folder proyek `lab8_vuejs`.

* **Komponen Beranda / Home (`Home.js`):**
  Buat file baru bernama `Home.js` untuk mengelola tampilan awal aplikasi:
```javascript
export default {
    template: `
        <div class="home-container">
            <h2>Selamat Datang di Portal Berita</h2>
            <p>Ini adalah halaman beranda aplikasi Frontend berbasis Single Page Application (SPA) menggunakan VueJS 3 dan Vue Router.</p>
        </div>
    `
};

```

* **Komponen Kelola Artikel (`Artikel.php` / `Artikel.js`):**
Pindahkan logika tabel data, modal tambah data, dan fungsi fetch API dari praktikum sebelumnya ke dalam file komponen khusus bernama `Artikel.js`:

```javascript
export default {
    template: `
        <div>
            <header style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px;">
                <h2>Kelola Data Artikel</h2>
                <button @click="showModal = true" class="btn btn-primary">Tambah Data</button>
            </header>

            <table class="table-data">
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Judul</th>
                        <th>Status</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="row in artikel" :key="row.id">
                        <td>{{ row.id }}</td>
                        <td>{{ row.judul }}</td>
                        <td><span class="status-badge" :class="row.status">{{ row.status }}</span></td>
                        <td>
                            <button class="btn btn-danger" @click="deleteArtikel(row.id)">Hapus</button>
                        </td>
                    </tr>
                    <tr v-if="artikel.length === 0">
                        <td colspan="4">Tidak ada data artikel.</td>
                    </tr>
                </tbody>
            </table>

            <div v-if="showModal" class="modal">
                <div class="modal-content">
                    <span class="close" @click="closeModal">&times;</span>
                    <h3>Tambah Data Artikel</h3>
                    <form @submit.prevent="saveArtikel">
                        <p>
                            <label>Judul</label>
                            <input type="text" v-model="form.judul" required>
                        </p>
                        <p>
                            <label>Isi Artikel</label>
                            <textarea v-model="form.isi" required></textarea>
                        </p>
                        <p>
                            <button type="submit" class="btn btn-success">Simpan</button>
                            <button type="button" class="btn btn-secondary" @click="closeModal">Batal</button>
                        </p>
                    </form>
                </div>
            </div>
        </div>
    `,
    data() {
        return {
            artikel: [],
            showModal: false,
            apiUrl: 'http://localhost:8080/post',
            form: { judul: '', isi: '' }
        }
    },
    methods: {
        fetchData() {
            fetch(this.apiUrl)
                .then(res => res.json())
                .then(data => this.artikel = data)
                .catch(err => console.error("Error API:", err));
        },
        saveArtikel() {
            fetch(this.apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
                body: new URLSearchParams(this.form)
            })
            .then(res => res.json())
            .then(() => {
                this.fetchData();
                this.closeModal();
            });
        },
        deleteArtikel(id) {
            if (confirm("Hapus data ini?")) {
                fetch(`${this.apiUrl}/${id}`, { method: 'DELETE' })
                    .then(res => res.json())
                    .then(() => this.fetchData());
            }
        },
        closeModal() {
            this.showModal = false;
            this.form.judul = '';
            this.form.isi = '';
        }
    },
    mounted() {
        this.fetchData();
    }
};

```

---

### 2. Mengintegrasikan Vue Router pada Halaman Utama

Ubah file `index.html` utama untuk memuat script Vue Router, mendaftarkan pemetaan rute URL, serta menyediakan tag `<router-link>` untuk navigasi dan `<router-view>` sebagai tempat render komponen dinamis.

* Buka file `index.html` dan sesuaikan kodenya menjadi seperti berikut:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>SPA Portal Berita - Vue Router</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <nav class="navbar">
            <router-link to="/" class="nav-item">Beranda</router-link>
            <router-link to="/artikel" class="nav-item">Kelola Artikel</router-link>
        </nav>

        <main class="content-wrapper">
            <router-view></router-view>
        </main>
    </div>

    <script src="[https://unpkg.com/vue@3/dist/vue.global.js](https://unpkg.com/vue@3/dist/vue.global.js)"></script>
    <script src="[https://unpkg.com/vue-router@4/dist/vue-router.global.js](https://unpkg.com/vue-router@4/dist/vue-router.global.js)"></script>

    <script type="module">
        // Mengimpor file komponen modular
        import Home from './Home.js';
        import Artikel from './Artikel.js';

        // 1. Definisikan pemetaan rute (Routes Mapping)
        const routes = [
            { path: '/', component: Home },
            { path: '/artikel', component: Artikel }
        ];

        // 2. Inisialisasi instance Vue Router dengan mode Hash History
        const router = VueRouter.createRouter({
            history: VueRouter.createWebHashHistory(),
            routes
        });

        // 3. Mount aplikasi Vue dan daftarkan routernya
        const app = Vue.createApp({});
        app.use(router);
        app.mount('#app');
    </script>
</body>
</html>

```

---

### 3. Pembaruan Desain CSS Navigasi (style.css)

Tambahkan style berikut ke dalam berkas `style.css` untuk mempercantik bilah navigasi menu serta memberikan penanda warna otomatis pada menu yang sedang aktif diklik (`.router-link-exact-active`).

```css
/* Style Tambahan untuk Navbar SPA */
.navbar {
    background-color: #2c3e50;
    padding: 10px 20px;
    display: flex;
    gap: 15px;
    margin-bottom: 20px;
}
.nav-item {
    color: #ecf0f1;
    text-decoration: none;
    font-weight: bold;
    padding: 5px 12px;
    transition: background 0.3s;
}
.nav-item:hover {
    background-color: #34495e;
    border-radius: 4px;
}

/* Style otomatis dari Vue Router saat route sedang aktif dibuka */
.router-link-exact-active {
    background-color: #3152d6 !important;
    color: #ffffff !important;
    border-radius: 4px;
}

.home-container {
    padding: 20px;
    border: 1px solid #eff1ff;
    background: #fafafa;
    border-radius: 5px;
}

```

#### 📸 Hasil Pengujian Sistem SPA (Single Page Application)

Berikut adalah visualisasi perpindahan antar halaman menu yang berjalan secara instan di sisi klien tanpa memicu reload halaman pada browser:

---

## 📝 Kesimpulan

Melalui praktikum Modul 12, aplikasi frontend telah sukses bertransformasi menjadi **Single Page Application (SPA)** menggunakan **Vue Router**. Dengan membagi antarmuka menjadi komponen terpisah (`Home.js` & `Artikel.js`), kode program menjadi lebih terstruktur. Pemanfaatan komponen penampung `<router-view>` memastikan bahwa proses transisi dan pergantian konten halaman web terasa sangat cepat, responsif, dan menghemat bandwidth server karena tidak memerlukan pemuatan ulang aset HTML eksternal.

```

---

```
