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

```
