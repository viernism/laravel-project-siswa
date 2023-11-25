```php
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

and

```php
@if (auth()->check())
                            @role('gurubk')
                                <li class="sidebar-item">
                                    <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse"
                                        data-bs-target="#pages" aria-expanded="false" aria-controls="pages">
                                        <i class="fa-regular fa-file-lines pe-2"></i>
                                        Pages
                                    </a>
                                    <ul id="pages" class="sidebar-dropdown list-unstyled collapse"
                                        data-bs-parent="#sidebar">
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
                                        <a href="#" class="sidebar-link collapsed" data-bs-toggle="collapse"
                                            data-bs-target="#pages" aria-expanded="false" aria-controls="pages">
                                            <i class="fa-regular fa-file-lines pe-2"></i>
                                            Pages
                                        </a>
                                        <ul id="pages" class="sidebar-dropdown list-unstyled collapse"
                                            data-bs-parent="#sidebar">
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