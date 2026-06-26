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
