Now, we have to organize our routes for the specific roles. To achieve this we could group it by middleware and roles.

Guest routes are established to cater to unauthenticated users, so don't change anything for the guest routes. let it be like this:

```php
Route::middleware(['guest'])->group(function () {
    Route::get('/login', [LoginController::class, 'index'])->name('login');
    Route::post('/login', [LoginController::class, 'login']);
    Route::get('/', [HomeController::class, 'index']);
    Route::get('/', [HomeController::class, 'search'])->name('search');
});
```

Now this route group is for authenticated users with added requirement of 'no-cache'. It also has common routes like logout, Siswa (student) index and search, Pelanggaran (violation) index and search, and Tanggapan (response) index and search. The role-specific routes for administrators ('admin') and Guru users ('gurubk') are defined within their respective middleware groups.
code:
```php
Route::middleware(['auth', 'no-cache'])->group(function () {
    // Common routes for all roles
    Route::post('/logout', [LoginController::class, 'logout'])->name('logout');
    Route::get('/siswa', [SiswaController::class, 'index']);
    Route::get('/siswa', [SiswaController::class, 'search'])->name('siswa.search');
    Route::get('/pelanggaran', [PelanggaranController::class, 'index']);
    Route::get('/pelanggaran', [PelanggaranController::class, 'search'])->name('pelanggaran.search');
    Route::get('/tanggapan', [TanggapanController::class, 'index']);
    Route::get('/tanggapan', [TanggapanController::class, 'search'])->name('tanggapan.search');

    // (Admin and Guru routes will be explained later on)
});
```

And admin routes are exclusive to users with the 'admin' role. These routes empower administrators to manage student data (Siswa), trigger specific actions, handle Pelanggaran (violations), and oversee Petugas (staff) management.

So add this after the common routes:
```php
// admin routes
    Route::middleware(['role:admin'])->group(function () {
        Route::post('/siswa', [SiswaController::class, 'store'])->name('siswa.store');
        Route::put('/siswa/{id}', [SiswaController::class, 'update'])->name('siswa.update');
        Route::delete('/siswa/{id}', [SiswaController::class, 'destroy'])->name('siswa.destroy');

        Route::get('/trigger', [PelanggaranController::class, 'trigger']);

        Route::delete('/pelanggaran/{id}', [PelanggaranController::class, 'destroy'])->name('pelanggaran.destroy');

        Route::get('/petugas', [PetugasController::class, 'index']);
        Route::post('/petugas', [PetugasController::class, 'store'])->name('petugas.store');
        Route::put('/petugas/{id}', [PetugasController::class, 'update'])->name('petugas.update');
        Route::delete('/petugas/{id}', [PetugasController::class, 'destroy'])->name('petugas.destroy');

    });
```

Guru routes are designed exclusively for users with the 'gurubk' role. These routes provide Guru users with the ability to manage Pelanggaran (violations) and Tanggapan (responses). The middleware ensures that only users with the designated roles can access these specialized functionalities.

Add this after admin routes:
```php
Route::middleware(['role:gurubk'])->group(function () {
        Route::post('/pelanggaran', [PelanggaranController::class, 'store'])->name('pelanggaran.store');
        Route::put('/pelanggaran/{id}', [PelanggaranController::class, 'update'])->name('pelanggaran.update');
        Route::post('/tanggapan', [TanggapanController::class, 'store'])->name('tanggapan.store');
        Route::put('/tanggapan/{id}', [TanggapanController::class, 'update'])->name('tanggapan.update');
        Route::delete('/tanggapan/{id}', [TanggapanController::class, 'destroy'])->name('tanggapan.destroy');
    });
```

In the end, your routes should look like this:
```php
use Illuminate\Support\Facades\Route;

use App\Http\Controllers\LoginController;
use App\Http\Controllers\HomeController;
use App\Http\Controllers\SiswaController;
use App\Http\Controllers\PelanggaranController;
use App\Http\Controllers\PetugasController;
use App\Http\Controllers\TanggapanController;
use App\Models\Pelanggaran;

Route::middleware(['guest'])->group(function () {
    Route::get('/login', [LoginController::class, 'index'])->name('login');
    Route::post('/login', [LoginController::class, 'login']);
    Route::get('/', [HomeController::class, 'index']);
    Route::get('/', [HomeController::class, 'search'])->name('search');
});

Route::middleware(['auth', 'no-cache'])->group(function () {
    // common routes for all roles
    Route::post('/logout', [LoginController::class, 'logout'])->name('logout');
    Route::get('/siswa', [SiswaController::class, 'index']);
    Route::get('/siswa', [SiswaController::class, 'search'])->name('siswa.search');
    Route::get('/pelanggaran', [PelanggaranController::class, 'index']);
    Route::get('/pelanggaran', [PelanggaranController::class, 'search'])->name('pelanggaran.search');
    Route::get('/tanggapan', [TanggapanController::class, 'index']);
    Route::get('/tanggapan', [TanggapanController::class, 'search'])->name('tanggapan.search');

    // admin routes
    Route::middleware(['role:admin'])->group(function () {
        Route::post('/siswa', [SiswaController::class, 'store'])->name('siswa.store');
        Route::put('/siswa/{id}', [SiswaController::class, 'update'])->name('siswa.update');
        Route::delete('/siswa/{id}', [SiswaController::class, 'destroy'])->name('siswa.destroy');

        Route::get('/trigger', [PelanggaranController::class, 'trigger']);

        Route::delete('/pelanggaran/{id}', [PelanggaranController::class, 'destroy'])->name('pelanggaran.destroy');

        Route::get('/petugas', [PetugasController::class, 'index']);
        Route::post('/petugas', [PetugasController::class, 'store'])->name('petugas.store');
        Route::put('/petugas/{id}', [PetugasController::class, 'update'])->name('petugas.update');
        Route::delete('/petugas/{id}', [PetugasController::class, 'destroy'])->name('petugas.destroy');

    });

    // Guru routes
    Route::middleware(['role:gurubk'])->group(function () {
        Route::post('/pelanggaran', [PelanggaranController::class, 'store'])->name('pelanggaran.store');
        Route::put('/pelanggaran/{id}', [PelanggaranController::class, 'update'])->name('pelanggaran.update');
        Route::post('/tanggapan', [TanggapanController::class, 'store'])->name('tanggapan.store');
        Route::put('/tanggapan/{id}', [TanggapanController::class, 'update'])->name('tanggapan.update');
        Route::delete('/tanggapan/{id}', [TanggapanController::class, 'destroy'])->name('tanggapan.destroy');
    });
});
```


Now we wan't to separate the pages just like how we did on the routes.

the ```@if (auth()->check())``` is used to check the if the user is authenticated or not for the html to be displayed, so the guest routes are not affected.

```php
@if (auth()->check())
                            
@endif
```

The section isde ```@role('gurubk')``` is for users with the 'gurubk' role,
It adds additional links for Siswa, Pelanggaran, and Tanggapan.

```php
@if (auth()->check())
    @role('gurubk')
        <li class="sidebar-item">
            <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse" data-bs-target="#pages"
                aria-expanded="false" aria-controls="pages">
                <i class="fa-regular fa-file-lines pe-2"></i>
                Pages
            </a>
            <ul id="pages" class="sidebar-dropdown list-unstyled collapse" data-bs-parent="#sidebar">
                @foreach (['Siswa', 'Pelanggaran', 'Tanggapan'] as $item)
                    <li class="sidebar-item">
                        <a href="{{ url(strtolower($item)) }}" class="sidebar-link text-white">
                            {{ $item }}
                        </a>
                    </li>
                @endforeach
            </ul>
        </li>
    @else
        @endrole
    @endrole
@endif
```

Now for the ```role('admin')``` which reflect the users with 'admin' role,
It adds additional links for Petugas and Trigger.

```php
@if (auth()->check())
    @role('gurubk')
        <li class="sidebar-item">
            <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse" data-bs-target="#pages"
                aria-expanded="false" aria-controls="pages">
                <i class="fa-regular fa-file-lines pe-2"></i>
                Pages
            </a>
            <ul id="pages" class="sidebar-dropdown list-unstyled collapse" data-bs-parent="#sidebar">
                @foreach (['Siswa', 'Pelanggaran', 'Tanggapan'] as $item)
                    <li class="sidebar-item">
                        <a href="{{ url(strtolower($item)) }}" class="sidebar-link text-white">
                            {{ $item }}
                        </a>
                    </li>
                @endforeach
            </ul>
        </li>
    @else
        @role('admin')
            <li class="sidebar-item">
                <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse" data-bs-target="#pages"
                    aria-expanded="false" aria-controls="pages">
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
        @endrole
    @endrole
@endif
```
Now, we're all done with the routes and roles. time to move to the next thing which is preview images.

Go to app.js in public directory and place this code:
```js
$(document).ready(function () {
    function handleImagePreview(input, previewElement) {
        $(input).change(function () {
            let reader = new FileReader();
            reader.onload = function (e) {
                $(previewElement).attr('src', e.target.result);
            };
            reader.readAsDataURL(this.files[0]);
        });
    }

    handleImagePreview('#image', '#prevFoto');

    handleImagePreview('#ubahImg', '#prevImg');
});
```

Now go to part 4, we're all done here.