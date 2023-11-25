In the previous part, we're done with the Login and Logout. In this part we're gonna focus on how the sites would look. This modular structure makes it easy to manage different parts of your application. Make your directory to look like this:

```
\RESOURCES\VIEWS
│   welcome.blade.php
│
├───auth
│       login.blade.php
│
├───layouts
│       app.blade.php
│       footer.blade.php
│       header.blade.php
│
└───pages
    │   home.blade.php
    │   test.blade.php <!-- delete this file -->
    │
    └───petugas
            pelanggaran.blade.php
            petugas.blade.php
            siswa.blade.php
            tanggapan.blade.php
```

Next, we create controllers for different sections of the application. The following controllers are created:
```php
php artisan make:controller PetugasController
php artisan make:controller HomeController
php artisan make:controller SiswaController
php artisan make:controller PelanggaranController
php artisan make:controller TanggapanController
php artisan make:controller PelanggaranTriggerController
```

We're gonna work on HomeController and home.blade.php first. Open your HomeController make a function that return the home.blade.php:
```php
public function index() {
    return view('pages.home');
}
```

And the routes:

```php
use App\Http\Controllers\HomeController;

Route::middleware(['guest'])->group(function() {
    Route::get('/login', [LoginController::class, 'index'])->name('login');
    Route::post('/login', [LoginController::class, 'login']);
    Route::get('/', [HomeController::class, 'index']);
});
```
For the HomeController, we set up a function index that returns the home.blade.php view. This function is associated with the route '/'.

We also configure routes for the login page. The '/' route is accessible by guests, and it directs them to the HomeController@index function.

Your routes should look like this for now:
```php
Route::middleware(['guest'])->group(function() {
    Route::get('/login', [LoginController::class, 'index'])->name('login');
    Route::post('/login', [LoginController::class, 'login']);
    Route::get('/', [HomeController::class, 'index']);
});

Route::middleware(['auth', 'no-cache'])->group(function () {
    Route::get('/home', function () {
        return view('pages.petugas.siswa');
    });

    Route::post('/logout', [LoginController::class, 'logout'])->name('logout');
});
```

Now we're making the master layout, 'app.blade.php', within the 'layouts' directory. but before we do that, let's download bootstrap and jquery and put it in the public directory like this also make app.js ands tyle.css respective to their name:
```
\PUBLIC
│   .htaccess
│   favicon.ico
│   index.php
│   robots.txt
│
├───css
│       bootstrap-grid.css
│       bootstrap-grid.css.map
│       bootstrap-grid.min.css
│       bootstrap-grid.min.css.map
│       bootstrap-grid.rtl.css
│       bootstrap-grid.rtl.css.map
│       bootstrap-grid.rtl.min.css
│       bootstrap-grid.rtl.min.css.map
│       bootstrap-reboot.css
│       bootstrap-reboot.css.map
│       bootstrap-reboot.min.css
│       bootstrap-reboot.min.css.map
│       bootstrap-reboot.rtl.css
│       bootstrap-reboot.rtl.css.map
│       bootstrap-reboot.rtl.min.css
│       bootstrap-reboot.rtl.min.css.map
│       bootstrap-utilities.css
│       bootstrap-utilities.css.map
│       bootstrap-utilities.min.css
│       bootstrap-utilities.min.css.map
│       bootstrap-utilities.rtl.css
│       bootstrap-utilities.rtl.css.map
│       bootstrap-utilities.rtl.min.css
│       bootstrap-utilities.rtl.min.css.map
│       bootstrap.css
│       bootstrap.css.map
│       bootstrap.min.css
│       bootstrap.min.css.map
│       bootstrap.rtl.css
│       bootstrap.rtl.css.map
│       bootstrap.rtl.min.css
│       bootstrap.rtl.min.css.map
|       style.css
│
└───js
        app.js
        bootstrap.bundle.js
        bootstrap.bundle.js.map
        bootstrap.bundle.min.js
        bootstrap.bundle.min.js.map
        bootstrap.esm.js
        bootstrap.esm.js.map
        bootstrap.esm.min.js
        bootstrap.esm.min.js.map
        bootstrap.js
        bootstrap.js.map
        bootstrap.min.js
        bootstrap.min.js.map
        jquery-3.7.1.js
```

The master layout app.blade.php is created to provide a consistent structure for all views. It includes Bootstrap and jQuery, and has placeholders for header, content, and footer.

Go to app.blade.php and copy the code below;

app.blade.php:
```php
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>{{ $title ?? 'app-name' }}</title>

    <link rel="stylesheet" href="{{ asset('css/bootstrap.min.css') }}">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.css">
    <link rel="stylesheet" href="{{ asset('css/style.css') }}">

    <script src="{{ asset('js/bootstrap.min.js') }}"></script>
    <script src="{{ asset('js/jquery-3.7.1.js') }}"></script>
    <script src="{{ asset('js/app.js') }}"></script>
</head>

<body>
    @auth
        <div class="wrapper">
            <!-- Sidebar -->
            <aside id="sidebar" class="bg-dark">
                <div class="h-100">
                    <div class="sidebar-logo text-white">
                        @auth
                            @role('gurubk')
                                Guru
                                @elserole('admin')
                                Admin
                            @else
                                Latihan UKK
                            @endrole
                        @endauth

                        @guest
                            Latihan UKK
                        @endguest
                    </div>
                    <!-- Sidebar Navigation -->
                    <ul class="sidebar-nav text-white">
                        <li class="sidebar-header">
                            Tools & Components
                        </li>
                        <li class="sidebar-item">
                            <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse"
                                data-bs-target="#pages" aria-expanded="false" aria-controls="pages">
                                <i class="fa-regular fa-file-lines pe-2"></i>
                                Pages
                            </a>
                            <ul id="pages" class="sidebar-dropdown list-unstyled collapse" data-bs-parent="#sidebar">
                                @foreach (['Siswa', 'Petugas', 'Pelanggaran', 'Tanggapan', 'Trigger'] as $item)
                                    <li class="sidebar-item">
                                        <a href="{{ url(strtolower($item)) }}" class="sidebar-link text-white">
                                            {{ $item }}
                                        </a>
                                    </li>
                                @endforeach
                            </ul>
                        </li>
                    </ul>
            </aside>
            <!-- Main Component -->
            <div class="main">
                <nav class="navbar navbar-expand px-3 border-bottom bg-dark" data-bs-theme="dark">
                    <!-- Button for sidebar toggle -->
                    <button class="btn" type="button" data-bs-theme="dark">
                        <span class="navbar-toggler-icon"></span>
                    </button>

                    <div class="ms-auto">
                        @auth
                            @role('gurubk')
                                <form action="{{ route('logout') }}" method="POST">
                                    @csrf
                                    <button type="submit" class="btn btn-danger">Logout</button>
                                </form>
                            @endrole

                            @role('admin')
                                <form action="{{ route('logout') }}" method="POST">
                                    @csrf
                                    <button type="submit" class="btn btn-danger">Logout</button>
                                </form>
                            @endrole
                        @endauth
                        @guest
                            <li class="nav-item dropdown">
                                <a class="btn btn-primary" href="/login">Login</a>
                            </li>
                        @endguest
                    </div>
                </nav>
                <main class="content px-3 py-2">
                    <div class="container-fluid">
                        <div class="mb-3">
                            @yield('content')
                        </div>
                    </div>
                </main>
            </div>
        </div>
    @endauth

    @guest
        <!-- Main Component -->
        <div class="main">
            <nav class="navbar navbar-expand px-3 border-bottom bg-dark" data-bs-theme="dark">
                <a class="navbar-brand" href="/">
                    <b>
                        @guest
                            Latihan UKK
                        @endguest
                    </b>
                </a>

                <div class="ms-auto">
                    @guest
                        <li class="nav-item dropdown">
                            <a class="btn btn-primary" href="/login">Login</a>
                        </li>
                    @endguest
                </div>
            </nav>
            <main class="content px-3 py-2">
                <div class="container-fluid">
                    <div class="mb-3">
                        @yield('content')
                    </div>
                </div>
            </main>
        </div>
    @endguest

    @include('layouts.footer')

</body>

</html>
```
style.css:
```css
:root,
[data-bs-theme="light"] {
  --bs-blue: #0d6efd;
  --bs-indigo: #6610f2;
  --bs-purple: #6f42c1;
  --bs-pink: #d63384;
  --bs-red: #dc3545;
  --bs-orange: #fd7e14;
  --bs-yellow: #ffc107;
  --bs-green: #198754;
  --bs-teal: #20c997;
  --bs-cyan: #0dcaf0;
  --bs-black: #000;
  --bs-white: #fff;
  --bs-gray: #6c757d;
  --bs-gray-dark: #343a40;
  --bs-gray-100: #f8f9fa;
  --bs-gray-200: #e9ecef;
  --bs-gray-300: #dee2e6;
  --bs-gray-400: #ced4da;
  --bs-gray-500: #adb5bd;
  --bs-gray-600: #6c757d;
  --bs-gray-700: #495057;
  --bs-gray-800: #343a40;
  --bs-gray-900: #212529;
  --bs-primary: #0d6efd;
  --bs-secondary: #6c757d;
  --bs-success: #198754;
  --bs-info: #0dcaf0;
  --bs-warning: #ffc107;
  --bs-danger: #dc3545;
  --bs-light: #f8f9fa;
  --bs-dark: #212529;
  --bs-primary-rgb: 13, 110, 253;
  --bs-secondary-rgb: 108, 117, 125;
  --bs-success-rgb: 25, 135, 84;
  --bs-info-rgb: 13, 202, 240;
  --bs-warning-rgb: 255, 193, 7;
  --bs-danger-rgb: 220, 53, 69;
  --bs-light-rgb: 248, 249, 250;
  --bs-dark-rgb: 33, 37, 41;
  --bs-primary-text-emphasis: #052c65;
  --bs-secondary-text-emphasis: #2b2f32;
  --bs-success-text-emphasis: #0a3622;
  --bs-info-text-emphasis: #055160;
  --bs-warning-text-emphasis: #664d03;
  --bs-danger-text-emphasis: #58151c;
  --bs-light-text-emphasis: #495057;
  --bs-dark-text-emphasis: #495057;
  --bs-primary-bg-subtle: #cfe2ff;
  --bs-secondary-bg-subtle: #e2e3e5;
  --bs-success-bg-subtle: #d1e7dd;
  --bs-info-bg-subtle: #cff4fc;
  --bs-warning-bg-subtle: #fff3cd;
  --bs-danger-bg-subtle: #f8d7da;
  --bs-light-bg-subtle: #fcfcfd;
  --bs-dark-bg-subtle: #ced4da;
  --bs-primary-border-subtle: #9ec5fe;
  --bs-secondary-border-subtle: #c4c8cb;
  --bs-success-border-subtle: #a3cfbb;
  --bs-info-border-subtle: #9eeaf9;
  --bs-warning-border-subtle: #ffe69c;
  --bs-danger-border-subtle: #f1aeb5;
  --bs-light-border-subtle: #e9ecef;
  --bs-dark-border-subtle: #adb5bd;
  --bs-white-rgb: 255, 255, 255;
  --bs-black-rgb: 0, 0, 0;
  --bs-font-sans-serif: system-ui, -apple-system, "Segoe UI", Roboto,
    "Helvetica Neue", "Noto Sans", "Liberation Sans", Arial, sans-serif,
    "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji";
  --bs-font-monospace: SFMono-Regular, Menlo, Monaco, Consolas,
    "Liberation Mono", "Courier New", monospace;
  --bs-gradient: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.15),
    rgba(255, 255, 255, 0)
  );
  --bs-body-font-family: var(--bs-font-sans-serif);
  --bs-body-font-size: 1rem;
  --bs-body-font-weight: 400;
  --bs-body-line-height: 1.5;
  --bs-body-color: #212529;
  --bs-body-color-rgb: 33, 37, 41;
  --bs-body-bg: #fff;
  --bs-body-bg-rgb: 255, 255, 255;
  --bs-emphasis-color: #000;
  --bs-emphasis-color-rgb: 0, 0, 0;
  --bs-secondary-color: rgba(33, 37, 41, 0.75);
  --bs-secondary-color-rgb: 33, 37, 41;
  --bs-secondary-bg: #e9ecef;
  --bs-secondary-bg-rgb: 233, 236, 239;
  --bs-tertiary-color: rgba(33, 37, 41, 0.5);
  --bs-tertiary-color-rgb: 33, 37, 41;
  --bs-tertiary-bg: #f8f9fa;
  --bs-tertiary-bg-rgb: 248, 249, 250;
  --bs-heading-color: inherit;
  --bs-link-color: #0d6efd;
  --bs-link-color-rgb: 13, 110, 253;
  --bs-link-decoration: underline;
  --bs-link-hover-color: #0a58ca;
  --bs-link-hover-color-rgb: 10, 88, 202;
  --bs-code-color: #d63384;
  --bs-highlight-color: #212529;
  --bs-highlight-bg: #fff3cd;
  --bs-border-width: 1px;
  --bs-border-style: solid;
  --bs-border-color: #dee2e6;
  --bs-border-color-translucent: rgba(0, 0, 0, 0.175);
  --bs-border-radius: 0.375rem;
  --bs-border-radius-sm: 0.25rem;
  --bs-border-radius-lg: 0.5rem;
  --bs-border-radius-xl: 1rem;
  --bs-border-radius-xxl: 2rem;
  --bs-border-radius-2xl: var(--bs-border-radius-xxl);
  --bs-border-radius-pill: 50rem;
  --bs-box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
  --bs-box-shadow-sm: 0 0.125rem 0.25rem rgba(0, 0, 0, 0.075);
  --bs-box-shadow-lg: 0 1rem 3rem rgba(0, 0, 0, 0.175);
  --bs-box-shadow-inset: inset 0 1px 2px rgba(0, 0, 0, 0.075);
  --bs-focus-ring-width: 0.25rem;
  --bs-focus-ring-opacity: 0.25;
  --bs-focus-ring-color: rgba(13, 110, 253, 0.25);
  --bs-form-valid-color: #198754;
  --bs-form-valid-border-color: #198754;
  --bs-form-invalid-color: #dc3545;
  --bs-form-invalid-border-color: #dc3545;
}

*,
::after,
::before {
  box-sizing: border-box;
}

body {
  font-family: "Poppins", sans-serif;
  margin: 0;
}

h3 {
  font-size: 1.2375rem;
  color: #000000;
}

a {
  cursor: pointer;
  text-decoration: none;
  font-family: "Poppins", sans-serif;
}

li {
  list-style: none;
}

/* Layout skeleton */

.wrapper {
  align-items: stretch;
  display: flex;
  width: 100%;
}

#sidebar {
  max-width: 264px;
  min-width: 264px;
  transition: all 0.35s ease-in-out;
  box-shadow: 0 0 35px 0 rgba(49, 57, 66, 0.5);
  z-index: 1111;
}

#sidebar,
#sidebar a,
#sidebar a:hover,
#sidebar a:focus,
#sidebar .sidebar-header {
    color: #fff;
}

/* Sidebar collapse */

#sidebar.collapsed {
  margin-left: -264px;
}

.main {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  width: 100%;
  overflow: hidden;
  transition: all 0.35s ease-in-out;
}

.sidebar-logo {
  padding: 1.15rem 1.5rem;
}

.sidebar-logo a {
  color: #000000;
  font-size: 1.25rem;
  font-weight: 600;
}

.sidebar-nav {
  padding: 0;
}

.sidebar-header {
  color: #000000;
  font-size: 0.75rem;
  padding: 1.5rem 1.5rem 0.375rem;
}

a.sidebar-link {
  padding: 0.625rem 1.625rem;
  color: #000000;
  position: relative;
  display: block;
  font-size: 1rem;
}

.sidebar-link[data-bs-toggle="collapse"]::after {
  border: solid;
  border-width: 0 0.075rem 0.075rem 0;
  content: "";
  display: inline-block;
  padding: 2px;
  position: absolute;
  right: 1.5rem;
  top: 1.4rem;
  transform: rotate(-135deg);
  transition: all 0.2s ease-out;
}

.sidebar-link[data-bs-toggle="collapse"].collapsed::after {
  transform: rotate(45deg);
  transition: all 0.2s ease-out;
}

.content {
  flex: 1;
  max-width: 100vw;
  width: 100vw;
}

/* Responsive */

@media (min-width: 768px) {
  .content {
    width: auto;
  }
}
```

app.js:
```js
document.addEventListener("DOMContentLoaded", function () {
    const sidebar = document.getElementById('sidebar');
    const toggler = document.querySelector(".btn");

    toggler.addEventListener("click", function () {
        sidebar.classList.toggle("collapsed");
    });
});
```

After that, we can make the tables. We're gonna learn how to fetch data and display it in the views, we're gonna work on home.blade.php first and go through everything else later on.

home.blade.php:
```php
@extends('layouts.app', ['title' => 'Home'])

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <i class="fas fa-table me-1"></i>
            Data Pelanggaran Siswa
        </div>
        <div class="card-body">
            <div class="table-responsive">
                <div class="row mb-3">
                    <form class="col-12 col-lg-auto mb-2 mb-lg-0 me-lg-auto" role="search" method="get" action="/">
                        <div class="input-group">
                            <input type="text" name="search" class="form-control" id="search" placeholder="Masukkan NIS Siswa">
                            <button type="submit" class="btn btn-primary">Search</button>
                        </div>
                    </form>
                </div>
                @if ($data->isNotEmpty())
                    <table class="table table-striped table-bordered">
                        <thead>
                            <tr>
                                <th>No</th>
                                <th>NIS</th>
                                <th>Nama</th>
                                <th>Kelas</th>
                                <th>Tanggal</th>
                                <th>Pelanggaran</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php $no = 1; ?>
                            @foreach ($data as $dt)
                                <tr>
                                    <td>{{ $no++ }}</td>
                                    <td>{{ $dt->nis }}</td>
                                    <td>{{ $dt->siswa->nama }}</td>
                                    <td>{{ $dt->siswa->kelas }}</td>
                                    <td>
                                        @if ($dt->tgl_pelanggaran)
                                            {{ $dt->tgl_pelanggaran->format('d F Y') }}
                                        @else
                                            N/A
                                        @endif
                                    </td>
                                    <td>{{ $dt->isi_pelanggaran }}</td>
                                </tr>
                            @endforeach
                        </tbody>
                    </table>
                @else
                    <p>Tidak ada data</p>
                @endif
            </div>
        </div>
    </div>
</div>
@endsection

```

Now put these in the Pelanggaran model:
```php
class Pelanggaran extends Model
{
    use HasFactory;
    protected $table = 'pelanggaran';
    protected $guarded = [
        'id',
    ];
    protected $primaryKey = 'id';
    public $timestamps = false;

    protected $casts = [
        'tgl_pelanggaran' => 'date',
    ];

    protected $fillable = [
        'id_pelanggaran',
        'tgl_pelanggaran',
        'nis',
        'isi_pelanggaran',
        'foto',
    ];

    public function siswa()
    {
        return $this->belongsTo(Siswa::class, 'nis', 'nis');
    }

    public function tanggapan()
    {
        return $this->hasOne(Tanggapan::class, 'id_pelanggaran', 'id_pelanggaran');
    }
}
```

Also add these in to Siswa model since laravel assumes that the table name is the plurar form of the model name with a lowercase and a standard underscores as word separators. So since we have a model named 'Siswa', laravel assumes that the associated database table is named 'siswas'

Siswa model:
```php
class Siswa extends Model
{
    use HasFactory;

    protected $table = 'siswa';
}
```

Now update your HomeController index function to:
```php
public function index() {
        $data = Pelanggaran::all();
        return view('pages.home', compact('data'));
    }
```

Now for the search function add this function in HomeController below the index functionm:
```php
public function search(Request $request)
    {
        $keyword = request('search');
        $data = Pelanggaran::where('nis', 'like', "%" . $keyword . "%")->paginate(5);
        return view('pages.home', compact('data'));
    }
```

And this route in the guests group:
```php
Route::get('/', [HomeController::class, 'search'])->name('search');
```

Now everything in home.blade.php should display Pelanggaran if you input data into the database. we're moving on to petugas's siswa.blade.php, making the entire CRUD for this one.

First we want to display data just like home.blade.php but in siswa.blade.php instead:
```php
@extends('layouts.app', ['title' => 'Siswa'])

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <i class="fas fa-table me-1"></i>
            Data Pelanggaran Siswa
        </div>
        <div class="card-body">
            <div class="table-responsive">
                <div class="row mb-3">
                    <form class="col-12 col-lg-auto mb-2 mb-lg-0 me-lg-auto" role="search" method="get" action="/siswa">
                        <div class="input-group">
                            <input type="text" name="search" class="form-control" id="search" placeholder="Masukkan NIS Siswa">
                            <button type="submit" class="btn btn-primary">Search</button>
                        </div>
                    </form>
                    <div class="col-12 col-lg-auto">
                        <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#tambahSiswa">
                            <i class="fas fa-plus"></i> Tambah
                        </button>
                    </div>

                </div>

                @if ($data->isNotEmpty())
                <table class="table table-hover table-bordered">
                    <thead class="table-dark">
                        <tr>
                            <th>No</th>
                            <th>NIS</th>
                            <th>Nama</th>
                            <th>Kelas</th>
                            <th>Tanggal</th>
                            <th>Aksi</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php $no = 1; ?>
                        @foreach ($data as $dt)
                        <tr>
                            <td>{{ $no++ }}</td>
                            <td>{{ $dt->nis }}</td>
                            <td>{{ $dt->nama }}</td>
                            <td>{{ $dt->kelas }}</td>
                            <td>
                                @if ($dt->created_at)
                                    {{ $dt->created_at->format('d F Y') }}
                                @else
                                    N/A
                                @endif
                            </td>
                            <td>
                                <button type="button" class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#ubahSiswa{{ $dt->id }}">
                                    <i class="fas fa-edit"></i> Ubah
                                </button>
                                <a href="" class="btn btn-danger btn-sm" data-bs-toggle="modal" data-bs-target="#hapusSiswa{{ $dt->id }}">
                                    <i class="fas fa-trash-alt"></i> Hapus
                                </a>
                            </td>
                        </tr>                     
                        @endforeach
                    </tbody>
                </table>
                @else
                <p>Tidak ada data</p>
                @endif
            </div>
        </div>
    </div>
</div>
@endsection
```

In SiswaController:
```php
use App\Models\Siswa;


class SiswaController extends Controller
{
    public function index() {
        $data = Siswa::all();
        return view('pages.petugas.siswa', compact('data'));    
    }
    
    public function search(Request $request)
    {
        $keyword = request('search');
        $data = Siswa::where('nis', 'like', "%" . $keyword . "%")->paginate(5);
        return view('pages.petugas.siswa', compact('data'));
    }
}
```

Add this inside the auth routes:
```php
    use App\Http\Controllers\SiswaController;

    Route::get('/siswa', [SiswaController::class, 'index']);
    Route::get('/siswa', [SiswaController::class, 'search'])->name('search');
```

Now to add Siswa into the database we need to make modal and its button first

add this button below the search form like this:
```php
<div class="row mb-3">
        <form class="col-12 col-lg-auto mb-2 mb-lg-0 me-lg-auto" role="search" method="get" action="/siswa">
            <div class="input-group">
                <input type="text" name="search" class="form-control" id="search" placeholder="Masukkan NIS Siswa">
                <button type="submit" class="btn btn-primary">Search</button>
            </div>
        </form>
        <div class="col-12 col-lg-auto">
            <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#tambahSiswa">Tambah</button>
        </div>
</div>
```

Now add this modal form code before the @endsection
```php
{{-- Modal Tambah --}}
<div class="modal fade" id="tambahSiswa" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h1 class="modal-title fs-5" id="exampleModalLabel">Tambah Siswa</h1>
            </div>
            <form action="{{ route('siswa.store') }}" method="POST"> <!-- You should set the appropriate route name -->
                @csrf
                <div class="modal-body">
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" id="nis" name="ns" placeholder="NIS">
                                <label for="nis" class="form-floating-label">NIS</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" id="nama" name="nm" placeholder="Nama">
                                <label for="nama" class="form-floating-label">Nama</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <select for="kelas" name="kls" id="kelas" class="form-select">
                                    @for ($i = 10; $i <= 12; $i++)
                                        @foreach (['MM1', 'MM2', 'RPL', 'TKJ'] as $class)
                                            <option value="{{ $i . '-' . $class }}">{{ $i . '-' . $class }}</option>
                                        @endforeach
                                    @endfor
                                </select>
                                <label for="kelas" class="form-floating-label">Kelas</label>
                            </div>
                        </div>
                    </div>               
                </div>
                <div class="modal-footer">
                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                    <button type="submit" class="btn btn-primary">Simpan</button>
                </div>
            </form>
        </div>
    </div>
</div>
```

And this function in the controller to add siswa:
```php
public function store(Request $request) {
    $request->validate([
        'ns' => 'required',
        'nm' => 'required',
        'kls' => 'required',
    ]);

    $siswa = new Siswa;
    $siswa->nis = $request->ns;
    $siswa->nama = $request->nm;
    $siswa->kelas = $request->kls;
    $siswa->save();

    if ($siswa) {
        // Redirect with success message
        return redirect('/siswa')->with('success', 'Siswa berhasil ditambah');
    } else {
        // Redirect with error message
        return redirect('/siswa')->with('error', 'Gagal menambah siswa');
    }
}
```

add the route inside the auth group:
```php
 Route::post('/siswa', [SiswaController::class, 'store'])->name('siswa.store');
```

Also update your Siswa model to this:
```php
class Siswa extends Model
{
    use HasFactory;

    protected $guarded = ['id'];

    protected $table = 'siswa';
    
    public $timestamps = false;

    protected $fillable = [
        'id',
        'nis',
        'nama', 
        'kelas',
    ];

    public function pelanggaran()
    {
        return $this->hasMany(Pelanggaran::class, 'nis', 'nis');
    }
}
```

You should be able to add siswa into the database now. 

To edit, or update, and deleting data we have to make another modal including its button, We want to work on edit first, now change its button:

```php
    <button type="button" class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#ubahSiswa{{ $dt->id }}">Ubah</button>
```

And add the edit modal inside the foreach because we want each data to have its own modal based on their id, place it before the @endforeach:

```php
{{-- Modal Ubah --}}
<div class="modal fade" id="ubahSiswa{{ $dt->id }}" tabindex="-1" aria-labelledby="editSiswaLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h1 class="modal-title fs-5" id="editSiswaLabel">Edit Siswa</h1>
            </div>
            <form action="{{ route('siswa.update', ['id' => $dt->id]) }}" method="POST">
                @csrf
                @method('PUT')
                <div class="modal-body">
                    <input type="hidden" name="id" value="{{ $dt->id }}">
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" id="nis" name="nis"
                                    placeholder="nis" value="{{ $dt->nis }}">
                                <label for="nis" class="form-floating-label">NIS</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <div a class="form-floating mb-3">
                                <input type="text" class="form-control" id="nama" name="nama"
                                    placeholder="nama" value="{{ $dt->nama }}">
                                <label for="nama" class="form-floating-label">Nama</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <select for="kelas" name="kelas" id="kelas" class="form-select">
                                    @for ($i = 10; $i <= 12; $i++)
                                        @foreach (['MM1', 'MM2', 'RPL', 'TKJ'] as $class)
                                            <option value="{{ $i . '-' . $class }}">{{ $i . '-' . $class }}</option>
                                        @endforeach
                                    @endfor
                                </select>
                                <label for="kelas" class="form-floating-label">Kelas</label>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                    <button type="submit" class="btn btn-primary">Simpan</button>
                </div>
            </form>
        </div>
    </div>
</div>
```

Now add this to the SiswaController:
```php
public function update(Request $request, $id)
    {
        $request->validate([
            'nis' => 'required',
            'nama' => 'required',
            'kelas' => 'required',
        ]);

        $siswa = Siswa::find($id);

        if (!$siswa) {
            return redirect()->back()->with('error', 'Siswa tidak ditemukan');
        }

        $updated = $siswa->update([
            'nis' => $request->nis,
            'nama' => $request->nama,
            'kelas' => $request->kelas,
        ]);

        if ($updated) {
            // Redirect with success message
            return redirect('/siswa')->with('success', 'Siswa berhasil diperbarui');
        } else {
            // Redirect with error message
            return redirect('/siswa')->with('error', 'Gagal memperbarui siswa');
        }
    }
```

And this in the auth route group:
```php
Route::put('/siswa/{id}', [SiswaController::class, 'update'])->name('siswa.update');
```

You should be able to update data Siswa now, for the delete its similiar first add these changes onto the button:
```php
 <a href="" class="btn btn-danger btn-sm" data-bs-toggle="modal" data-bs-target="#hapusSiswa{{ $dt->id }}">Hapus</a>
```

And place this modal below the edit modal:
```php
{{-- Modal Hapus --}}
<div class="modal fade" id="hapusSiswa{{ $dt->id }}" tabindex="-1" aria-labelledby="exampleModalLabel"
    aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-danger">
                <h1 class="modal-title fs-5" id="exampleModalLabel">Hapus Siswa</h1>
            </div>
            <div class="modal-body">
                <div class="text-center">
                    <h4>Apakah anda ykin menghapus siswa <span>
                            <font color="red">{{ $dt->nama }}</font>
                        </span></h4>
                </div>
            </div>
            <form action="{{ route('siswa.destroy', ['id' => $dt->id]) }}" method="POST">
                @csrf
                @method('DELETE')
                <div class="modal-footer">
                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                    <button type="submit" class="btn btn-danger">Simpan</button>
                </div>
            </form>

        </div>
    </div>
</div>
```

Add this to the SiswaController:

```php
    public function destroy($id)
    {
        // Find the student by ID
        $siswa = Siswa::find($id);

        if (!$siswa) {
            return redirect('/siswa')->with('error', 'Siswa tidak ditemukan.');
        }

        // Delete the student's record
        $siswa->delete();

        return redirect('/siswa')->with('success', 'Siswa berhasil dihapus.');
    }
```

And the route inside the auth route group:
```php
Route::delete('/siswa/{id}', [SiswaController::class, 'destroy'])->name('siswa.destroy');
```

And now you need 

Now you could do CRUD on Siswa, since you haven't learned how to work with file uploads in Laravel, i should show that first beforehand.

First we need to prepare the routes for Pelanggaran table:
```php
    Route::get('/pelanggaran', [PelanggaranController::class, 'index']);
    Route::get('/pelanggaran', [PelanggaranController::class, 'search'])->name('pelanggaran.search');
    Route::post('/pelanggaran', [PelanggaranController::class, 'store'])->name('pelanggaran.store');
    Route::put('/pelanggaran/{id}', [PelanggaranController::class, 'update'])->name('pelanggaran.update');
    Route::delete('/pelanggaran/{id}', [PelanggaranController::class, 'destroy'])->name('pelanggaran.destroy');
```

And moving on to the views pelanggaran.blade.php:
```php
@extends('layouts.app', ['title' => 'Pelanggaran'])

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <i class="fas fa-table me-1"></i>
            Data Pelanggaran
        </div>
        <div class="card-body">
            <div class="table-responsive">
                @role('gurubk')
                <div class="row mb-3">
                    <form class="col-12 col-lg-auto mb-2 mb-lg-0 me-lg-auto" role="search" method="get" action="/pelanggaran">
                        <div class="input-group">
                            <input type="text" name="search" class="form-control" id="search" placeholder="Masukkan NIS Siswa">
                            <button type="submit" class="btn btn-primary">Search</button>
                        </div>
                    </form>

                    <div class="col-12 col-lg-auto">
                        <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#tambahPelanggaran">
                            <i class="fas fa-plus"></i> Tambah
                        </button>
                    </div>
                </div>


                @if ($data->isNotEmpty())
                    <table class="table table-striped table-bordered">
                        <thead class="table-dark">
                            <tr>
                                <th>No</th>
                                <th>Foto</th>
                                <th>NIS</th>
                                <th>Nama</th>
                                <th>Kelas</th>
                                <th>Tanggal</th>
                                <th>Isi Pelanggaran</th>
                                <th>Aksi</th>

                            </tr>
                        </thead>
                        <tbody>
                            <?php $no = 1; ?>
                            @foreach ($data as $dt)
                                <tr>
                                    <td>{{ $no++ }}</td>
                                    <td style="text-align: center">
                                        <img src="{{ asset('photos/' . $dt->foto) }}" width="75" height="75">
                                    </td>
                                    <td>{{ $dt->nis }}</td>
                                    <td>{{ $dt->siswa->nama }}</td>
                                    <td>{{ $dt->siswa->kelas }}</td>
                                    <td>
                                        @if ($dt->tgl_pelanggaran)
                                            {{ $dt->tgl_pelanggaran->format('d F Y') }}
                                        @else
                                            N/A
                                        @endif
                                    </td>
                                    <td>{{ $dt->isi_pelanggaran }}</td>
                                    <td>
                                        <button type="button" class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#ubahPelanggaran{{ $dt->id }}">
                                            <i class="fas fa-edit"></i> Ubah
                                        </button>
                                        <a href="" class="btn btn-danger btn-sm" data-bs-toggle="modal" data-bs-target="#hapusPelanggaran{{ $dt->id }}">
                                            <i class="fas fa-trash-alt"></i> Hapus
                                        </a>
                                    </td>
                                </tr>                      
                                @endforeach
                        </tbody>
                    </table>
                @else
                    <p>Tidak ada data</p>
                @endif
            </div>
        </div>
    </div>
</div>
@endsection
```

Just like before add this function into the PelanggaranController:
```php
public function index() {
        $data = Pelanggaran::all();
        $dataSiswa = Siswa::all();
        return view('pages.petugas.pelanggaran', compact('data', 'dataSiswa'));
}

public function search(Request $request) {
    $keyword = $request->search;
    $data = Pelanggaran::where('nis', 'like', "%" . $keyword . "%")->paginate(5);
    $dataSiswa = Siswa::all();
    return view('pages.petugas.pelanggaran', compact(['data', 'dataSiswa']));
}
```

Just like before add this modal before the @endsection line:
```php
{{-- Modal Tambah --}}
<div class="modal fade" id="tambahPelanggaran" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h5 class="modal-title" id="exampleModalLabel">Tambah Data Pelanggaran</h5>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
                <form id="create-pelanggaran-form" action="{{ route('pelanggaran.store') }}" method="POST" enctype="multipart/form-data">
                    @csrf
                    <div class="row g-2">
                        <div class="col-md">
                            Foto Preview:<br>
                            <img id="prevFoto" src="{{ asset('avatar.png') }}" class="rounded" style="width: 150px">
                            <input type="file" class="form-control" name="photo" id="image">
                        </div>
                        <div class="col-md">
                            <div class="form-floating">
                                <div class="row g-1">
                                    <div class="col-md">
                                        <div class="form-floating">
                                            <input type="text" class="form-control" name="idpel" placeholder="Id Pelanggaran">
                                            <label for="floatingInputGrid">Id Pelanggaran:</label>
                                        </div>
                                    </div>
                                </div>
                                <br>
                                <div class="row g-2">
                                    <div class="col-md">
                                        <div class="form-floating">
                                            <select class="form-select" name="nis">
                                                <option> -- Pilih NIS --</option>
                                                @foreach ($dataSiswa as $nis)
                                                    <option value="{{ $nis->nis }}" name="id">
                                                        {{ $nis->nis }} => {{ $nis->nama }} {{ $nis->kelas }}
                                                    </option>
                                                @endforeach
                                            </select>
                                            <label for="floatingInputGrid">NIS:</label>
                                        </div>
                                    </div>
                                </div>
                                <br>
                                <div class="row g-1">
                                    <div class="col-md">
                                        <div class="form-floating">
                                            <input min="2021-01-01" type="date" class="form-control" name="tgl"
                                                placeholder="tanggal pelanggaran">
                                            <label for="floatingInputGrid">Tanggal Pelanggaran:</label>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        <div class="row g-1">
                            <div class="col-md">
                                <label>Isi Pelanggaran:</label>
                                <textarea class="form-control scrollable" rows="5" name="isi"></textarea>
                            </div>
                        </div>
                        <br>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-bs- dismiss="modal">Tutup</button>
                            <button type="submit" class="btn btn-primary">Simpan</button>
                        </div>
                </form>
            </div>
            </div>
        </div>
    </div>
</div>
```

Now add this function into your PelanggaranController:
```php
    public function store(Request $request)
    {
        // this block of code validate the inputted data
        $this->validate($request, [
            'photo' => 'mimes:jpg,jpeg,png|max:2048',
            'idpel' => 'required',
            'nis' => 'required',
            'tgl' => 'required',
            'isi' => 'required',
        ]);

        // process the uploaded photo
        $photoPath = 'avatar.png'; // default value if theres no photo uploaded

        if ($request->hasFile('photo')) {
            $photo = $request->file('photo');
            $photoPath = $photo->getClientOriginalName();
            $photo->move(public_path('photos'), $photoPath);
        }

        // create a new pelanggaran record
        Pelanggaran::create([
            'foto' => $photoPath,
            'id_pelanggaran' => $request->idpel,
            'nis' => $request->nis,
            'tgl_pelanggaran' => $request->tgl,
            'isi_pelanggaran' => $request->isi,
        ]);

        return redirect()->back()->with('success', 'Data Pelanggaran berhasil disimpan.');
    }
```

You should be able to input data to pelanggaran only if you have input a Siswa data.

For the Edit/Update just like before put this inside the foreach before the endforeach:
```php
{{-- Modal Ubah --}}
<div class="modal fade" id="ubahPelanggaran{{ $dt->id }}" tabindex="-1" aria-labelledby="editPelanggaranLabel"
    aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h1 class="modal-title fs-5" id="editPelanggaranLabel">Edit Pelanggaran</h1>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form action="{{ route('pelanggaran.update', ['id' => $dt->id]) }}" method="POST"
                enctype="multipart/form-data">
                @csrf
                @method('PUT')
                <div class="modal-body">
                    <div class="row g-2">
                        <div class="col-md">
                            Foto Preview:<br>
                            <img id="prevImg" src="{{ asset('photos/' . $dt->foto) }}" class="rounded"
                                style="width: 150px">
                            <input type="file" class="form-control" name="photo" id="ubahImg">
                        </div>
                        <div class="col-md">
                            <div class="form-floating">
                                <input type="text" class="form-control" name="idpel"
                                    value="{{ $dt->id_pelanggaran }}">
                                <label for="idpel">Id Pelanggaran:</label>
                            </div>
                            <br>
                            <div class="form-floating">
                                <select class="form-control" name="nis">
                                    <option value="{{ $dt->nis }}">
                                        {{ $dt->nis }} {{ $dt->siswa->nama }} {{ $dt->siswa->kelas }}
                                    </option>
                                    @foreach ($dataSiswa as $nis)
                                        <option value="{{ $nis->nis }}">
                                            {{ $nis->nis }} => {{ $nis->nama }} {{ $nis->kelas }}
                                        </option>
                                    @endforeach
                                </select>
                                <label for="nis">Nis:</label>
                            </div>
                            <br>
                            <div class="form-floating">
                                <input type="date" class="form-control" name="tgl"
                                    value="{{ $dt->tgl_pelanggaran->format('Y-m-d') }}">
                                <label for="tgl">Tanggal Pelanggaran:</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <label>Isi Pelanggaran:</label>
                            @isset($dt)
                                <textarea class="form-control scrollable" name="isi" rows="5" required>{{ $dt->isi_pelanggaran }}</textarea>
                            @else
                                Created by Siswanto @2023 49
                                <textarea class="form-control scrollable" name="isi" rows="5" required></textarea>
                                @endIf
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Tutup</button>
                            <button type="submit" class="btn btn-success">Ubah</button>
                        </div>
                    </div>
                </form>
            </div>
        </div>
    </div>
```

In the controller:
```php
public function update(Request $request, $id)
    {
        $this->validate($request, [
            'idpel' => 'required',
            'nis' => 'required',
            'tgl' => 'required',
            'isi' => 'required',
            'photo' => 'mimes:jpg,jpeg,png|max:2048',
        ]);

        $upd = Pelanggaran::find($id);

        if ($request->hasFile('photo')) {
            $image = $request->file('photo');
            $fotoPath = $image->getClientOriginalName();
            $image->move(public_path('photos'), $fotoPath);
            $upd->foto = $fotoPath;
        }

        $upd->id_pelanggaran = $request->idpel;
        $upd->nis = $request->nis;
        $upd->tgl_pelanggaran = $request->tgl;
        $upd->isi_pelanggaran = $request->isi;

        if ($upd->save()) {
            // Redirect with success message
            return redirect()->back()->with(['success' => 'Data pelanggaran berhasil diubah']);
        } else {
            // Redirect with error message
            return redirect()->back()->with(['error' => 'Data pelanggaran gagal diubah']);
        }
    }
```

You should be able to Edit/Update the data now. For the delete you just put it below the Update modal:
```php
{{-- Modal Hapus --}}
<div class="modal fade" id="hapusPelanggaran{{ $dt->id }}" tabindex="-1" aria-labelledby="exampleModalLabel"
    aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
            <div class="modal-header bg-danger text-white">
                <h1 class="modal-title fs-5" id="exampleModalLabel">Hapus Pelanggaran</h1>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <div class="modal-body">
                <h4 class="text-center">Apakah anda yakin menghapus pelanggaran
                    <span>
                        <font color="blue">{{ $dt->siswa->nama }}: {{ $dt->isi_pelanggaran }}</font>
                    </span>
                </h4>
            </div>
            <form action="{{ route('pelanggaran.destroy', ['id' => $dt->id]) }}" method="POST">
                @csrf
                @method('DELETE')
                <div class="modal-footer">
                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                    <button type="submit" class="btn btn-danger">Simpan</button>
                </div>
            </form>
        </div>
    </div>
</div>
```

In PelanggaranController:
```php
public function destroy($id)
    {
        $del = Pelanggaran::find($id);

        if (!$del) {
            // Handle the case if the id wasn't found
            return redirect()->back()->with(['error' => 'Data pelanggaran not found.']);
        }

        // Delete the record
        $del->delete();

        if ($del) {
            // Redirect with a success message
            return redirect()->back()->with(['success' => 'Data pelanggaran has been deleted.']);
        } else {
            // Redirect with an error message
            return redirect()->back()->with(['error' => 'Failed to delete data pelanggaran.']);
        }
    }
```

You should be able to do CRUD in Pelanggaran table now.

For Petugas we need to do it a bit differently since we are using spatie/laravel-permission afterall.

First we need to prepare the routes:
```php
    Route::get('/petugas', [PetugasController::class, 'index']);
    Route::post('/petugas', [PetugasController::class, 'store'])->name('petugas.store');
    Route::put('/petugas/{id}', [PetugasController::class, 'update'])->name('petugas.update');
    Route::delete('/petugas/{id}', [PetugasController::class, 'destroy'])->name('petugas.destroy');
```

Now we want to just display the data, and its the same as others:
```php
use App\Models\Pelanggaran;
use App\Models\Siswa;

public function index() {
        $data = Pelanggaran::all();
        $dataSiswa = Siswa::all();
        return view('pages.petugas.pelanggaran', compact('data', 'dataSiswa'));
    }
```


For the view:
```php
@extends('layouts.app', ['title' => 'Petugas'])

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <i class="fas fa-table me-1"></i>
            Data Petugas
        </div>
        <div class="card-body">
            <div class="table-responsive">
                <div class="row mb-3">
                    <div class="col-12 col-lg-auto">
                        <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#tambahPetugas">
                            <i class="fas fa-plus"></i> Tambah
                        </button>
                    </div>
                </div>

                @if ($data->isNotEmpty())
                    <table class="table table-striped table-bordered">
                        <thead class="table-dark">
                            <tr>
                                <th>No</th>
                                <th>ID Petugas</th>
                                <th>Nama</th>
                                <th>Username</th>
                                <th>Telp</th>
                                <th>Level</th>
                                <th>Aksi</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php $no = 1; ?>
                            @foreach ($data as $dt)
                                <tr>
                                    <td>{{ $no++ }}</td>
                                    <td>{{ $dt->id_petugas }}</td>
                                    <td>{{ $dt->nama }}</td>
                                    <td>{{ $dt->username }}</td>
                                    <td>{{ $dt->telp }}</td>
                                    <td>
                                        @foreach ($dt->roles as $role)
                                            {{ $role->name }}
                                            @if (!$loop->last)
                                                ,
                                            @endif
                                        @endforeach
                                    </td>
                                    <td>
                                        <button type="button" class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#ubahPetugas{{ $dt->id }}">
                                            <i class="fas fa-edit"></i> Ubah
                                        </button>
                                        <a class="btn btn-danger btn-sm" data-bs-toggle="modal" data-bs-target="#hapusPetugas{{ $dt->id }}">
                                            <i class="fas fa-trash-alt"></i> Hapus
                                        </a>
                                    </td>
                                </tr>                          
                                @endforeach
                        </tbody>
                    </table>
                @else
                    <p>Tidak ada data</p>
                @endif
            </div>
        </div>
    </div>
</div>


@endsection
```

Now we want to be able to store the data, add this modal before the @endsection:
```php
{{-- Modal Tambah --}}
<div class="modal fade" id="tambahPetugas" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h1 class="modal-title fs-5" id="exampleModalLabel">Tambah Petugas</h1>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form action="{{ route('petugas.store') }}" method="POST">
                @csrf
                <div class="modal-body">
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" name="id_pet" id="id_pet" placeholder="ID Petugas">
                                <label for="id_pet">ID Petugas</label>
                            </div>
                        </div>
                    
                    <div class="row g-2">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" name="nm" id="nm" placeholder="Nama">
                                <label for="nm">Nama</label>
                            </div>
                        </div>
                        <div class="col-md">
                            <div class="form-floating">
                                <select class="form-select" name="role" id="role">
                                    @foreach ($roles as $role)
                                        <option value="{{ $role->id }}">{{ $role->name }}</option>
                                    @endforeach
                                </select>
                                <label for="role">Role</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-2">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" name="usernm" id="usernm" placeholder="Username">
                                <label for="usernm">Username</label>
                            </div>                    
                        </div>
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="password" class="form-control" name="passwd" id="passwd" placeholder="Password">
                                <label for="passwd">Password</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-1">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" name="tlp" id="tlp" placeholder="No. Telp/HP">
                                <label for="tlp">No Telp/HP</label>
                            </div>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Tutup</button>
                        <button type="submit" class="btn btn-primary">Simpan</button>
                    </div>                    
                </div>
            </form>
        </div>
    </div>
</div>
```

And the function for it:
```php
public function store(Request $request)
    {
        $request->validate([
            'id_pet' => 'required',
            'nm' => 'required',
            'role' => 'required|exists:roles,id',
            'usernm' => 'required|unique:petugas,username',
            'passwd' => 'required',
            'tlp' => 'required',
        ]);

        $petugas = new Petugas();
        $petugas->id_petugas = $request->id_pet;
        $petugas->nama = $request->nm;
        $petugas->username = $request->usernm;
        $petugas->password = bcrypt($request->passwd);
        $petugas->telp = $request->tlp;
        $petugas->save();

        $role = Role::find($request->role);
        $petugas->assignRole($role);

        return redirect()->back()->with('success', 'Petugas berhasil ditambahkan');
    }
```


To be able to edit the data first we need to put this modal just before the @endforeach:
```php
                {{-- Modal Ubah --}}
                <div class="modal fade" id="ubahPetugas{{ $dt->id }}" tabindex="-1"
                    aria-labelledby="editPetugasLabel" aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header bg-warning">
                                <h1 class="modal-title fs-5" id="editPetugasLabel">Edit Petugas</h1>
                                <button type="button" class="btn-close" data-bs-dismiss="modal"
                                    aria-label="Close"></button>
                            </div>
                            <form action="{{ route('petugas.update', ['id' => $dt->id]) }}" method="POST">
                                @csrf
                                @method('PUT')
                                <div class="modal-body">
                                    <input type="hidden" name="id" value="{{ $dt->id }}">
                                    <div class="row g-1">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" name="id_petugas"
                                                    id="id_petugas" value="{{ $dt->id_petugas }}" autocomplete="off">
                                                <label for="id_petugas">ID Petugas</label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row g-2">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" name="nama" id="nama"
                                                    value="{{ $dt->nama }}" autocomplete="off">
                                                <label for="nama">Nama</label>
                                            </div>
                                        </div>
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <select class="form-select" name="role" id="role"
                                                    autocomplete="off">
                                                    @foreach ($roles as $role)
                                                        <option value="{{ $role->id }}"
                                                            @if ($dt->hasRole($role->name)) selected @endif>
                                                            {{ $role->name }}
                                                            @if ($dt->hasRole($role->name))
                                                                (Assigned)
                                                            @endif
                                                        </option>
                                                    @endforeach
                                                </select>
                                                <label for="role">Role</label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row g-2">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" name="username"
                                                    id="username" value="{{ $dt->username }}" autocomplete="off">
                                                <label for="username">Username</label>
                                            </div>
                                        </div>
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="password" class="form-control" name="password"
                                                    id="password" placeholder="password" autocomplete="off">
                                                <label for="password">Password</label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row g-1">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" name="telp" id="telp"
                                                    value="{{ $dt->telp }}" autocomplete="off">
                                                <label for="telp">No Telp/HP</label>
                                            </div>
                                        </div>
                                    </div>
                                </div>
                                <div class="modal-footer">
                                    <button class="btn btn-secondary" type="button"
                                        data-bs-dismiss="modal">Tutup</button>
                                    <button type="submit" class="btn btn-warning">Simpan</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
```

Now for the function:
```php
public function update(Request $request, $id)
    {
        $request->validate([
            'id_petugas' => 'required',
            'nama' => 'required',
            'role' => 'required|exists:roles,id',
            'username' => 'required|unique:petugas,username,' . $id,
            'telp' => 'required',
        ]);

        $petugas = Petugas::findOrFail($id);

        $petugas->id_petugas = $request->id_petugas;
        $petugas->nama = $request->nama;
        $petugas->username = $request->username;
        $petugas->telp = $request->telp;

        if (!empty($request->password)) {
            $petugas->password = bcrypt($request->password);
        }

        $petugas->save();

        $role = Role::find($request->role);
        $petugas->syncRoles([$role->name]);

        return redirect()->back()->with('success', 'Petugas berhasil diperbarui');
    }
```

You should be able to edit the petugas as you please now, for the delete its the same as others just put it below the update modal:
```php
                {{-- Modal Hapus --}}
                <div class="modal fade" id="hapusPetugas{{ $dt->id }}" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header bg-danger text-white">
                                <h1 class="modal-title fs-5" id="exampleModalLabel">Hapus Petugas</h1>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <h4 class="text-center">Apakah anda yakin menghapus petugas
                                   <span><font color="blue">{{$dt->nama}} </font></span>
                                </h4>
                            </div>
                            <form action="{{ route('petugas.destroy', ['id' => $dt->id]) }}" method="POST">
                                @csrf
                                @method('DELETE')
                                <div class="modal-footer">
                                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                                    <button type="submit" class="btn btn-danger">Simpan</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
```

In PetugasController:
```php
public function destroy($id)
    {
        $petugas = Petugas::find($id);

        if (!$petugas) {
            return redirect()->back()->with('error', 'Petugas tidak ditemukan');
        }

        if ($petugas->delete()) {
            return redirect()->back()->with('success', 'Petugas berhasil dihapus');
        }
    }
```

Now you can do CRUD on there. Now for the tanggapan, its the same as others, really.

First we need to go to web.php and prepare the routes
```php
    Route::get('/tanggapan', [TanggapanController::class, 'index']);
    Route::get('/tanggapan', [TanggapanController::class, 'search'])->name('tanggapan.search');
    Route::post('/tanggapan', [TanggapanController::class, 'store'])->name('tanggapan.store');
    Route::put('/tanggapan/{id}', [TanggapanController::class, 'update'])->name('tanggapan.update');
    Route::delete('/tanggapan/{id}', [TanggapanController::class, 'destroy'])->name('tanggapan.destroy');
```

And its model:
```php
class Tanggapan extends Model
{
    use HasFactory;

    protected $table = 'tanggapan';

    public $timestamps = false;

    protected $casts = [
        'tgl_tanggapan' => 'date',
    ];

    protected $fillable = [
        'id_tanggapan', 
        'id_pelanggaran', 
        'tgl_tanggapan', 
        'isi_tanggapan', 
        'id_petugas',
    ];

    public function pelanggaran()
    {
        return $this->belongsTo(Pelanggaran::class, 'id_pelanggaran', 'id_pelanggaran');
    }

    public function petugas()
    {
        return $this->belongsTo(Petugas::class, 'id_petugas', 'id_petugas');
    }
}
```

For the view:
```php
@@extends('layouts.app', ['title' => 'Tanggapan'])

@section('content')
<div class="container">
    <div class="card mb-4">
        <div class="card-header">
            <i class="fas fa-table me-1"></i>
            Data Tanggapan
        </div>
        <div class="card-body">
            <div class="table-responsive">
                <div class="row mb-3">
                    <form class="col-12 col-lg-auto mb-2 mb-lg-0 me-lg-auto" role="search" method="get" action="/tanggapan">
                        <div class="input-group">
                            <input type="text" name="search" class="form-control" id="search" placeholder="Masukkan ID Tanggapan">
                            <button type="submit" class="btn btn-primary">Search</button>
                        </div>
                    </form>
                    <div class="col-12 col-lg-auto">
                        <button type="button" class="btn btn-primary btn-sm" data-bs-toggle="modal" data-bs-target="#tambahSiswa">
                            <i class="fas fa-plus"></i> Tambah
                        </button>
                    </div>
                </div>
                @if ($data->isNotEmpty())
                    <table class="table table-striped table-bordered">
                        <thead class="table-dark">
                            <tr>
                                <th>No</th>
                                <th>ID Tanggapan</th>
                                <th>ID Pelanggaran</th>
                                <th>Tanggal Tanggapan</th>
                                <th>Isi Tanggapan</th>
                                <th>ID Petugas</th>
                                <th>Aksi</th>
                            </tr>
                        </thead>
                        <tbody>
                            <?php $no = 1; ?>
                            @foreach ($data as $dt)
                                <tr>
                                    <td>{{ $no++ }}</td>
                                    <td>{{ $dt->id_tanggapan }}</td>
                                    <td>{{ $dt->id_pelanggaran }}</td>
                                    <td>
                                        @if ($dt->tgl_tanggapan)
                                            {{ $dt->tgl_tanggapan->format('d F Y') }}
                                        @else
                                            N/A
                                        @endif
                                    </td>
                                    <td>{{ $dt->isi_tanggapan }}</td>
                                    <td>{{ $dt->id_petugas }}</td>
                                    <td>
                                        <button type="button" class="btn btn-warning btn-sm" data-bs-toggle="modal" data-bs-target="#ubahTanggapan{{ $dt->id }}">
                                            <i class="fas fa-edit"></i> Ubah
                                        </button>
                                        <a class="btn btn-danger btn-sm" data-bs-toggle="modal" data-bs-target="#hapusTanggapan{{ $dt->id }}">
                                            <i class="fas fa-trash-alt"></i> Hapus
                                        </a>
                                    </td>
                                </tr>                          
                                @endforeach
                        </tbody>
                    </table>
                @else
                    <p>Tidak ada data</p>
                @endif
            </div>
        </div>
    </div>
</div>
@endsection
```

Now the function for it:
```php
use App\Models\Tanggapan;
use APp\Models\Petugas;

public function index() {
    $data = Tanggapan::all();
    $petugasList = Petugas::all();
    return view('pages.petugas.tanggapan', compact('data', 'petugasList'));
}

public function search(Request $request) {
    $keyword = request('search');
    $data = Tanggapan::where('id_tanggapan', 'like', "%" . $keyword . "%")->paginate(5);
    $petugasList = Petugas::all();
    return view('pages.petugas.tanggapan', compact('data', 'petugasList'));
}
```

Now just like before add this modal before the @endsection:
```php
{{-- Modal Tambah --}}
<div class="modal fade" id="tambahTanggapan" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog">
        <div class="modal-content">
            <div class="modal-header bg-primary text-white">
                <h1 class="modal-title fs-5" id="exampleModalLabel">Tambah Tanggapan</h1>
                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
            </div>
            <form action="{{ route('tanggapan.store') }}" method="POST">
                @csrf
                <div class="modal-body">
                    <div class="row g-2">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" name="id_tgpn" id="id_tgpn" placeholder="ID Tanggapan">
                                <label for="id_tgpn">ID Tanggapan</label>
                            </div>
                        </div>
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="text" class="form-control" id="id_pelanggaran" name="id_plngrn" placeholder="NIS">
                                <label for="id_pelanggaran" class="form-floating-label">ID Pelanggaran</label>
                            </div>
                        </div>
                    </div>
                    <div class="row g-2">
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <input type="date" class="form-control" id="tgl" name="tgl" value="">
                                <label for="tgl" class="">Tanggal Tanggapan:</label>
                            </div>
                        </div>
                        <div class="col-md">
                            <div class="form-floating mb-3">
                                <select class="form-select" name="id_pet" id="id_pet" placeholder="ID Petugas">
                                    @foreach($petugasList as $petugas)
                                        <option value="{{ $petugas->id_petugas }}">
                                            {{ $petugas->nama }} (ID: {{ $petugas->id_petugas }})
                                        </option>
                                    @endforeach
                                </select>
                                <label for="id_pet">Pilih Petugas</label>
                            </div>         
                        </div>
                    </div>
                    <div class="row g-1">
                        <label for="isi">Isi Tanggapan:</label>
                        <textarea class="form-control scrollable" rows="5" name="isi" id="isi"></textarea>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Tutup</button>
                        <button type="submit" class="btn btn-primary">Simpan</button>
                    </div>                    
                </div>
            </form>            
        </div>
    </div>
</div>
```

Storing data function:
```php
public function store(Request $request)
    {
        $request->validate([
            'id_tgpn' => 'required',
            'id_plngrn' => 'required',
            'tgl' => 'required|date',
            'id_pet' => 'required|exists:petugas,id_petugas',
            'isi' => 'required',
        ]);

        $tanggapan = new Tanggapan();
        $tanggapan->id_tanggapan = $request->id_tgpn;
        $tanggapan->id_pelanggaran = $request->id_plngrn;
        $tanggapan->tgl_tanggapan = $request->tgl;
        $tanggapan->id_petugas = $request->id_pet;
        $tanggapan->isi_tanggapan = $request->isi;
        $tanggapan->save();

        return redirect()->back()->with('success', 'Tanggapan berhasil ditambahkan');
    }
```

You should try inserting some data now. For the edit just like the others, put this right before the @endforeach:
```php
                {{-- Modal Ubah --}}
                <div class="modal fade" id="ubahTanggapan{{ $dt->id }}" tabindex="-1" aria-labelledby="editTanggapanLabel" aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header bg-warning">
                                <h1 class="modal-title fs-5" id="editTanggapanLabel">Edit Tanggapan</h1>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <form action="{{ route('tanggapan.update', ['id' => $dt->id]) }}" method="POST">
                                @csrf
                                @method('PUT')
                                <div class="modal-body">
                                    <input type="hidden" name="id" value="{{ $dt->id }}">
                                    <div class="row g-2">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" name="id_tanggapan" id="id_tanggapan" placeholder="ID Tanggapan" value="{{ $dt->id_tanggapan }}">
                                                <label for="id_tanggapan">ID Tanggapan</label>
                                            </div>
                                        </div>
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="text" class="form-control" id="id_pelanggaran" name="id_pelanggaran" placeholder="id_pelanggaran" value="{{ $dt->id_pelanggaran }}">
                                                <label for="id_pelanggaran" class="form-floating-label">ID Pelanggaran</label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="row g-2">
                                        <div class="col-md">
                                            <div class="form-floating mb-3">
                                                <input type="date" class="form-control" id="tgl" name="tgl" value="{{ $dt->tgl_tanggapan->format('Y-m-d') }}">
                                                <label for="tgl" class="">Tanggal Tanggapan</label>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-md">
                                        <div class="form-floating mb-3">
                                            <select class="form-select" name="id_petugas" id="id_petugas" placeholder="ID Petugas">
                                                @foreach($petugasList as $petugas)
                                                    <option value="{{ $petugas->id_petugas }}" @if($dt->id_petugas === $petugas->id_petugas) selected @endif>
                                                        {{ $petugas->nama }} (ID: {{ $petugas->id_petugas }})
                                                    </option>
                                                @endforeach
                                            </select>
                                            <label for="id_petugas">Pilih Petugas</label>
                                        </div>         
                                    </div>
                                    <div class="row g-1">
                                        <label for="isi">Isi Tanggapan:</label>
                                        <textarea class="form-control scrollable" rows="5" name="isi" id="isi">{{ $dt->isi_tanggapan }}</textarea>
                                    </div>
                                </div>
                                <div class="modal-footer">
                                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                                    <button type="submit" class="btn btn-warning">Simpan</button>
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
```

For the function:
```php
public function update(Request $request, $id)
    {
        $request->validate([
            'id_tanggapan' => 'required',
            'id_pelanggaran' => 'required',
            'tgl' => 'required|date',
            'id_petugas' => 'required',
            'isi' => 'required',
        ]);

        $tanggapan = Tanggapan::find($id);

        if (!$tanggapan) {
            return redirect()->back()->with('error', 'Tanggapan not found');
        }

        $tanggapan->id_tanggapan = $request->id_tanggapan;
        $tanggapan->id_pelanggaran = $request->id_pelanggaran;
        $tanggapan->tgl_tanggapan = $request->tgl;
        $tanggapan->id_petugas = $request->id_petugas;
        $tanggapan->isi_tanggapan = $request->isi;
        $tanggapan->save();

        return redirect()->back()->with('success', 'Tanggapan updated successfully');
    }
```

For deleting the data you should do the modal first right after the edit modal:
```php
                {{-- Modal Hapus --}}
                <div class="modal fade" id="hapusTanggapan{{ $dt->id }}" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
                    <div class="modal-dialog">
                        <div class="modal-content">
                            <div class="modal-header bg-danger text-white">
                                <h1 class="modal-title fs-5" id="exampleModalLabel">Hapus Tanggapan</h1>
                                <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                            </div>
                            <div class="modal-body">
                                <div class="text-center">
                                    <h4>Apakah anda yakin menghapus tanggapan ini? <span><font color="red">{{ $dt->nama }}</font></span></h4>
                                </div>
                            </div>
                            <form action="{{ route('tanggapan.destroy', ['id' => $dt->id]) }}" method="POST">
                                @csrf
                                @method('DELETE')
                                <div class="modal-footer">
                                    <button class="btn btn-secondary" type="button" data-bs-dismiss="modal">Tutup</button>
                                    <button type="submit" class="btn btn-danger">Simpan</button>
                                </div>
                            </form>
                            
                        </div>
                    </div>
                </div>
```

Now the function:
```php
public function destroy($id) {
        $tanggapan = Tanggapan::find($id);

        if (!$tanggapan) {
            return redirect()->back()->with('error', 'Tanggapan tidak ditemukan');
        }

        $tanggapan->delete();

        return redirect()->back()->with('success', 'Tanggapan berhasil dihapus'); 
    }
```

Now you could do CRUD in tanggapan. at this point, your routes should look like this:
```php
use App\Http\Controllers\LoginController;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\SiswaController;
use App\Http\Controllers\PelanggaranController;
use App\Http\Controllers\PetugasController;
use App\Http\Controllers\TanggapanController;

Route::middleware(['guest'])->group(function() {
    Route::get('/login', [LoginController::class, 'index'])->name('login');
    Route::post('/login', [LoginController::class, 'login']);
    Route::get('/', [HomeController::class, 'index']);
    Route::get('/', [HomeController::class, 'search'])->name('search');
});

Route::middleware(['auth', 'no-cache'])->group(function () {
    Route::get('/siswa', [SiswaController::class, 'index']);
    Route::get('/siswa', [SiswaController::class, 'search'])->name('search');
    Route::post('/siswa', [SiswaController::class, 'store'])->name('siswa.store');
    Route::put('/siswa/{id}', [SiswaController::class, 'update'])->name('siswa.update');
    Route::delete('/siswa/{id}', [SiswaController::class, 'destroy'])->name('siswa.destroy');

    Route::get('/pelanggaran', [PelanggaranController::class, 'index']);
    Route::get('/pelanggaran', [PelanggaranController::class, 'search'])->name('pelanggaran.search');
    Route::post('/pelanggaran', [PelanggaranController::class, 'store'])->name('pelanggaran.store');
    Route::put('/pelanggaran/{id}', [PelanggaranController::class, 'update'])->name('pelanggaran.update');
    Route::delete('/pelanggaran/{id}', [PelanggaranController::class, 'destroy'])->name('pelanggaran.destroy');

    Route::get('/petugas', [PetugasController::class, 'index']);
    Route::post('/petugas', [PetugasController::class, 'store'])->name('petugas.store');
    Route::put('/petugas/{id}', [PetugasController::class, 'update'])->name('petugas.update');
    Route::delete('/petugas/{id}', [PetugasController::class, 'destroy'])->name('petugas.destroy');

    Route::get('/tanggapan', [TanggapanController::class, 'index']);
    Route::get('/tanggapan', [TanggapanController::class, 'search'])->name('tanggapan.search');
    Route::post('/tanggapan', [TanggapanController::class, 'store'])->name('tanggapan.store');
    Route::put('/tanggapan/{id}', [TanggapanController::class, 'update'])->name('tanggapan.update');
    Route::delete('/tanggapan/{id}', [TanggapanController::class, 'destroy'])->name('tanggapan.destroy');

    Route::post('/logout', [LoginController::class, 'logout'])->name('logout');
});
```