The first thing we want to work at is the notification when we do the CRUD, so install this package first with the command:
```bash
composer require yoeunes/toastr
```

The usage is very simple and straightforward, you just have to use ```toastr()``` helper function inside your controller to set a toast notification.

```php
// Display an error toast with no title
toastr()->error('Oops! Something went wrong!');
``` 
I took this explaination from the their github page, check them out!

Now we want to modify every single controller, well not everyone of them but most. The first controller is SiswaController:

```php
class SiswaController extends Controller
{
    public function index()
    {
        $data = Siswa::all();
        return view('pages.petugas.siswa', compact('data'));
    }

    public function search(Request $request)
    {
        $keyword = request('search');
        $data = Siswa::where('nis', 'like', "%" . $keyword . "%")->paginate(5);
        return view('pages.petugas.siswa', compact('data'));
    }

    public function store(Request $request)
    {
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
            toastr()->success('Siswa berhasil ditambah');
        } else {
            toastr()->error('Gagal menambah siswa');
        }

        return redirect('/siswa');
    }

    public function update(Request $request, $id)
    {
        $request->validate([
            'nis' => 'required',
            'nama' => 'required',
            'kelas' => 'required',
        ]);

        $siswa = Siswa::find($id);

        if (!$siswa) {
            toastr()->error('Siswa tidak ditemukan');
            return redirect()->back();
        }

        $updated = $siswa->update([
            'nis' => $request->nis,
            'nama' => $request->nama,
            'kelas' => $request->kelas,
        ]);

        if ($updated) {
            toastr()->success('Siswa berhasil diperbarui');
        } else {
            toastr()->error('Gagal memperbarui siswa');
        }

        return redirect('/siswa');
    }

    public function destroy($id)
    {
        $siswa = Siswa::find($id);

        if (!$siswa) {
            toastr()->error('Siswa tidak ditemukan.');
        } else {
            $siswa->delete();
            toastr()->success('Siswa berhasil dihapus.');
        }

        return redirect('/siswa');
    }
}
```

Now, when a certain action is done a toastr notification will be displayed. We don't have to use ```with()``` function anymore because we are relying solely on ```toastr()```.

The PetugasController:
```php
class PetugasController extends Controller
{
    public function index()
    {
        $data = Petugas::all();
        $roles = Role::all();
        return view('pages.petugas.petugas', compact('data', 'roles'));
    }

    public function store(Request $request)
    {
        $request->validate([
            'id_pet' => 'required',
            'nm' => 'required',
            'role' => 'required|exists:roles,id', // Ensure the selected role exists
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

        toastr()->success('Petugas berhasil ditambahkan');

        return redirect()->back();
    }

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

        toastr()->success('Petugas berhasil diperbarui');

        return redirect()->back();
    }

    public function destroy($id)
    {
        $petugas = Petugas::find($id);

        if (!$petugas) {
            toastr()->error('Petugas tidak ditemukan');
            return redirect()->back();
        }

        $petugas->delete();

        toastr()->success('Petugas berhasil dihapus');

        return redirect()->back();
    }
}
```

WIth the PelanggaranController:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Pelanggaran;
use App\Models\Siswa;
use App\Models\TriggerPelanggaran;

class PelanggaranController extends Controller
{
    public function index()
    {
        $data = Pelanggaran::all();
        $dataSiswa = Siswa::all();
        return view('pages.petugas.pelanggaran', compact('data', 'dataSiswa'));
    }

    public function store(Request $request)
    {
        $this->validate($request, [
            'photo' => 'mimes:jpg,jpeg,png|max:2048',
            'idpel' => 'required',
            'nis' => 'required',
            'tgl' => 'required',
            'isi' => 'required',
        ]);

        $photoPath = 'avatar.png';

        if ($request->hasFile('photo')) {
            $photo = $request->file('photo');
            $photoPath = $photo->getClientOriginalName();
            $photo->move(public_path('photos'), $photoPath);
        }

        Pelanggaran::create([
            'foto' => $photoPath,
            'id_pelanggaran' => $request->idpel,
            'nis' => $request->nis,
            'tgl_pelanggaran' => $request->tgl,
            'isi_pelanggaran' => $request->isi,
        ]);

        toastr()->success('Data Pelanggaran berhasil disimpan.');

        return redirect()->back();
    }

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
            toastr()->success('Data pelanggaran berhasil diubah');
        } else {
            toastr()->error('Data pelanggaran gagal diubah');
        }

        return redirect()->back();
    }

    public function destroy($id)
    {
        $del = Pelanggaran::find($id);

        if (!$del) {
            toastr()->error('Data pelanggaran tidak ditemukan.');
            return redirect()->back();
        }

        $del->delete();

        toastr()->success('Data pelanggaran berhasil dihapus');

        return redirect()->back();
    }

    public function search(Request $request)
    {
        $keyword = $request->search;
        $data = Pelanggaran::where('nis', 'like', "%" . $keyword . "%")->paginate(5);
        $dataSiswa = Siswa::all();
        return view('pages.petugas.pelanggaran', compact(['data', 'dataSiswa']));
    }

    public function trigger()
    {
        $data = TriggerPelanggaran::paginate(10);
        return view('pages.petugas.pelanggaran_trigger', compact('data'));
    }
}
```

The TanggapanController:
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Tanggapan;
use App\Models\Petugas;

class TanggapanController extends Controller
{
    public function index()
    {
        $data = Tanggapan::all();
        $petugasList = Petugas::all();
        return view('pages.petugas.tanggapan', compact('data', 'petugasList'));
    }

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

        toastr()->success('Tanggapan berhasil ditambahkan');

        return redirect()->back();
    }

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
            toastr()->error('Tanggapan not found');
            return redirect()->back();
        }

        $tanggapan->id_tanggapan = $request->id_tanggapan;
        $tanggapan->id_pelanggaran = $request->id_pelanggaran;
        $tanggapan->tgl_tanggapan = $request->tgl;
        $tanggapan->id_petugas = $request->id_petugas;
        $tanggapan->isi_tanggapan = $request->isi;
        $tanggapan->save();

        toastr()->success('Tanggapan updated successfully');

        return redirect()->back();
    }

    public function destroy($id)
    {
        $tanggapan = Tanggapan::find($id);

        if (!$tanggapan) {
            toastr()->error('Tanggapan tidak ditemukan');
            return redirect()->back();
        }

        $tanggapan->delete();

        toastr()->success('Tanggapan berhasil dihapus');

        return redirect()->back();
    }

    public function search(Request $request)
    {
        $keyword = $request->search;
        $data = Tanggapan::where('id_tanggapan', 'like', "%" . $keyword . "%")->paginate(5);
        $petugasList = Petugas::all();
        return view('pages.petugas.tanggapan', compact('data', 'petugasList'));
    }
}
```

Notifications is all done, now we're moving to how to extract the table as pdf. First, install the package through composer. We will be using mPDF:

