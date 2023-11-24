# Preparation

1. Make the Laravel project:
```bash
composer create-project laravel/laravel openbk "10.*.*"
```
in this case we're making Student Violation Tracker so name it something like that

2. Now after we cd into the project and opened it in your favourite code editor we need to prepare the database called **"openbk"**

3. Open the .env file in your project directory and configure the database connection by setting the DB_DATABASE to "openbk"
```env
DB_DATABASE = openbk
```

4. In the config/app.php file, set the timezone to your preferred timezone. For example, if you live in Jakarta, set it to "Asia/Jakarta."
```env
'timezone' => 'Asia/Jakarta',
```

5. Now we're making the migration for the tables in the database, there are 5 tables in total but one of them are a trigger table.

**Table Siswa**:
```bash
php artisan make:migration create_siswa_table
```
Schema:
```php
Schema::create('siswa', function (Blueprint $table) {
            $table->id();
            $table->char('nis', 10)->unique;
            $table->string('nama', 50);
            $table->string('kelas', 10);             
            $table->timestamps();
        });
```
**Table Petugas**:

```bash
php artisan make:migration create_petugas_table
```

Schema:
```php
Schema::create('petugas', function (Blueprint $table) {
            $table->id();
            $table->char('id_petugas', 10)->unique();
            $table->string('nama', 50);
            $table->string('username', 50)->unique();
            $table->string('password', 200);
            $table->string('telp', 15);
            $table->timestamps();
        });
```

Since we will be using a library to manage Petugas permissions and role in the database

We need to install the required library through composer by doing this command below:

```bash
 composer require spatie/laravel-permission "6.*"
```

After that if the service provider did not automatically get registered you may manually add the service provider in your config/app.php file:

```php
'providers' => [
    // ...
    Spatie\Permission\PermissionServiceProvider::class,
];
```

Now to generate the migrations for the roles and permissions tables:

```bash
php artisan vendor:publish --provider="Spatie\Permission\PermissionServiceProvider"
```

**Table Pelanggaran**:
```bash
php artisan make:migration create_pelanggaran_table
```

Schema:
```php
Schema::create('pelanggaran', function (Blueprint $table) {
            $table->id();
            $table->char('id_pelanggaran', 10)->unique();
            $table->date('tgl_pelanggaran');
            $table->char('nis', 10);
            $table->text('isi_pelanggaran');
            $table->text('foto')->nullable();
            $table->timestamps();
        
            $table->foreign('nis')->references('nis')->on('siswa')
                ->onDelete('cascade')
                ->onUpdate('cascade');
        });
```

**Table Tanggapan**:
```bash
php artisan make:migration create_tanggapan_table 
```

Schema:
```php
Schema::create('tanggapan', function (Blueprint $table) {
            $table->id();
            $table->char('id_tanggapan', 10)->unique();
            $table->char('id_pelanggaran', 10);
            $table->date('tgl_tanggapan');
            $table->text('isi_tanggapan');
            $table->char('id_petugas', 10);
            $table->timestamps();
        
            $table->foreign('id_pelanggaran')->references('id_pelanggaran')->on('pelanggaran')
                ->onDelete('cascade')
                ->onUpdate('cascade');
        
            $table->foreign('id_petugas')->references('id_petugas')->on('petugas');
        });
```

**Trigger Pelanggaran**:
```bash
php artisan make:migration create_pelanggaran_trigger
```

Schema:
```php
use Illuminate\Support\Facades\DB;

public function up(): void
    {
        DB::unprepared('
        CREATE TRIGGER `inp_plgr` AFTER INSERT ON `pelanggaran` FOR EACH ROW
        BEGIN
            INSERT INTO `inp_pelanggaran` (nis, tgl_pelanggaran, id_pelanggaran)
            VALUES (NEW.nis, NOW(), NEW.id_pelanggaran);
        END;
        ');
    }

public function down(): void
    {
        DB::unprepared('DROP TRIGGER IF EXISTS `inp_plgr`');
    }
```

**Table inp_pelanggaran**:
```php
Schema::create('inp_pelanggaran', function (Blueprint $table) {
            $table->id();
            $table->integer('nis');
            $table->timestamp('tgl_pelanggaran');
            $table->integer('id_pelanggaran');
            $table->timestamps();
        });
```

After all that, run the migrations:
```
php artisan migrate
```

6. Make the models for each table
```bash
php artisan make:model Siswa
php artisan make:model Petugas
php artisan make:model Pelanggaran
php artisan make:model Tanggapan
```

7. Since we log in using Petugas table, we need to configure the authentication in your 'config/auth.php'

```php
'defaults' => [
        'guard' => 'web',
        'passwords' => 'petugas',
    ],
```
```php
 'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'petugas',
        ],
    ],
```
```php
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\Models\User::class,
        ],
        'petugas' => [
            'driver' => 'eloquent',
            'model' => App\Models\Petugas::class,
        ],
    ],
```

8. Change the Petugas Model into this below:

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Spatie\Permission\Traits\HasRoles;
use Laravel\Sanctum\HasApiTokens;

class Petugas extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable, HasRoles;

    protected $guard_name = 'web';
    protected $guarded = ['id'];
    protected $table = 'petugas';
    public $timestamps = false;

    protected $fillable = [
        'id_petugas',
        'nama',
        'username',
        'password',
        'telp',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $enums = [
      // we dont use level anymore since it's managed by the library roles stuff
    ];

    protected $casts = [
        'username_verified_at' => 'datetime',
        'password' => 'hashed',
    ];
}
```

Create seeder for Petugas:
```bash
php artisan make:seeder CreatePetugasSeeder
```

Open the seeder for Petugas and change the code like this below:
```php
namespace Database\Seeders;

use Illuminate\Database\Console\Seeds\WithoutModelEvents;
use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\Hash;
use App\Models\Petugas;
use Spatie\Permission\Models\Role;
use Spatie\Permission\Models\Permission;

class CreatePetugasSeeder extends Seeder
{
    /**
     * Run the database seeds.
     */
    public function run(): void
    {
        $adminRole = Role::firstOrCreate([
            'name' => 'admin',
            'guard_name' => 'web'
        ]);
        $gurubkRole = Role::firstOrCreate([
            'name' => 'gurubk', 
            'guard_name' => 'web'
        ]);

        $users = [
            [
                'id_petugas' => 'admin1',
                'nama' => 'Admin',
                'username' => 'admin',
                'password' => bcrypt('admin123'),
                'telp' => '0812345678',
            ],
            [
                'id_petugas' => 'gurubk1',
                'nama' => 'Guru BK',
                'username' => 'bk',
                'password' => bcrypt('guru123'),
                'telp' => '08571234657',
            ],
        ];

        foreach ($users as $userData) {
            $user = Petugas::create($userData);

            if ($userData['nama'] === 'Admin') {
                $user->assignRole($adminRole);
            } elseif ($userData['nama'] === 'Guru BK') {
                $user->assignRole($gurubkRole);
            }
        }
    }
}
```

Then seed the database by this command:
```bash
php artisan db:seed --class=CreatePetugasSeeder
```

Tips: Put call the CreatePetugasSeeder in DatabaseSeeder.php because by doing that you can just use --seed when you want to migrate the databse and seed it at the same time

```php
use Database\Seeders\CreatePetugasSeeder;

public function run(): void
    {
        $this->call(CreatePetugasSeeder::class);
    }
```

You can do:
```php
php artisan migrate --seed

// or

php artisan migrate:fresh --seed
```

9. Change model TriggerPelanggaran
```php
class TriggerPelanggaran extends Model
{
    use HasFactory;
    protected $table = 'inp_pelanggaran';
    public $timestamps = false;
}
```

10. Now we make the LoginController for Petugas to be able to log in to the site.

```bash
php artisan make:controller LoginController
```

Open the LoginController, then make an index function inside the class that returns the login page.

```php
use Illuminate\Support\Facades\Auth;

 public function index() {
        return view('auth.login');
    }
```

Now the go to resources/views/ and make auth/login.blade.php and put this code:
```php
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <!-- CSS only -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.0/dist/css/bootstrap.min.css" rel="stylesheet" 
    integrity="sha384-gH2yIJqKdNHPEq0n4Mqa/HGKIhSkIHeL5AyhkYV8i59U5AR6csBvApHHNl/vI1Bx" crossorigin="anonymous">
    <title>Login</title>
</head>
<body>
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card my-5">
                <div class="card-body">
                    <h1 class="text-center mb-4 text-primary">Tentang Aplikasi</h1>
                    <p class="text-center text-muted">
                        Aplikasi ini dibuat untuk membantu guru BP/BK dalam mendata dan mendokumentasikan pelanggaran tata tertib di sekolah.
                    </p>
                    <hr class="my-4">
                    <h2 class="text-center mb-4 text-primary">Login</h2>
                    <form action="" method="POST">
                        @csrf
                        <div class="mb-3">
                            <label for="username" class="form-label">Username</label>
                            <input type="text" name="username" value="{{ old('username') }}" class="form-control" id="username" required>
                        </div>
                        <div class="mb-3">
                            <label for="password" class="form-label">Password</label>
                            <input type="password" name="password" class="form-control" id="password" required>
                        </div>
                        <button type="submit" class="btn btn-primary btn-block">Sign In</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```
Go to routes/web.php and add this line of code:
```php
use App\Http\Controllers\LoginController;

Route::get('/', [LoginController::class, 'index'])->name('login');
Route::post('/', [LoginController::class, 'login']);
```

Now make the login function
```php
public function login(Request $request)
    {
        $credentials = $request->validate([
            'username' => 'required',
            'password' => 'required',
        ]);

        if (Auth::attempt($credentials)) {
            return redirect()->intended('/test');
        }

        return back()->withErrors(['error' => 'Invalid username or password']);
    }
```

Since we already make the route we don't need to do that, now make another folder and files in views views/pages/test.blade.php

```php
<form action="{{ route('logout') }}" method="POST">
    @csrf
    <button type="submit">Logout</button>
</form>
```
The route:
```php
Route::get('/test', function () {
        return view('pages.test');
    });
```

You should be able to log in now. Now let's apply our middleware so when Petugas logged out, they can't go back with the account still being logged in

Make the middleware:
```php
php artisan make:middleware NoCacheMiddleware
```

Go to App\Http\Middleware\NoCacheMiddleware.php and change the code to this below:
```php
public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        // Prevent caching for authenticated users
        if (auth()->check()) {
            $response->headers->set('Cache-Control', 'no-cache, no-store, must-revalidate');
            $response->headers->set('Pragma', 'no-cache');
            $response->headers->set('Expires', '0');
        }

        return $response;
    }
```

Now we go to App\Http\Kernel.php and add both middlewareGroups and aliases like this:

```php
protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\NoCacheMiddleware::class,
            ],
        ];
```

```php
protected $middlewareAliases = [
        'no-cache' => \App\Http\Middleware\NoCacheMiddleware::class,
    ];
```

In the routes change it like this:
```php
Route::middleware(['guest'])->group(function() {
    Route::get('/', [LoginController::class, 'index'])->name('login');
    Route::post('/', [LoginController::class, 'login']);
});

Route::middleware(['auth', 'no-cache'])->group(function () {
    Route::get('/test', function () {
        return view('pages.test');
    });

    Route::post('/logout', [LoginController::class, 'logout'])->name('logout');
});
```

Now the middleware should work correctly.

Now we gotta make the logout function since we already have the routes and the button we just have to add this line of code in our LoginController:

```php
public function logout() {
        Auth::logout();

        return redirect()->route('login');
    }
```

You should now be able to log in and log out with proper middleware to prevent back button access after logout.