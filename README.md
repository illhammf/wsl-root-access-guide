# WSL Root Access Guide

Panduan mengatasi masalah akses direktori `/root` pada Ubuntu WSL melalui Windows File Explorer menggunakan Linux Access Control List (ACL).

Repository ini dibuat untuk membantu pengguna Windows Subsystem for Linux (WSL) yang mengalami kendala ketika membuka direktori `/root` melalui Windows File Explorer, khususnya ketika menggunakan akun Linux biasa (non-root).

## Daftar Isi

- [Tentang Project](#-tentang-project)
- [Permasalahan](#-permasalahan)
- [Mengapa Hal Ini Terjadi?](#-mengapa-hal-ini-terjadi)
- [Persiapan](#-persiapan)
- [Langkah Penyelesaian](#-langkah-penyelesaian)
- [Verifikasi Akses](#-verifikasi-akses)
- [Troubleshooting](#-troubleshooting)
- [Mengembalikan Permission](#-mengembalikan-permission)
- [Catatan Keamanan](#-catatan-keamanan)
- [Referensi](#-referensi)

---

## Tentang Project

Pada sistem Linux, direktori `/root` merupakan home directory khusus untuk pengguna administrator (root).

Secara default, direktori tersebut memiliki permission:

```bash
drwx------
```

Konfigurasi tersebut berarti hanya pengguna root yang memiliki akses ke direktori tersebut.

Ketika menggunakan Ubuntu WSL melalui Windows, kondisi ini dapat menyebabkan Windows File Explorer tidak bisa membuka direktori `/root`, meskipun pengguna masih bisa mengaksesnya melalui terminal menggunakan `sudo`.

Project ini mendokumentasikan solusi menggunakan ACL agar pengguna Linux biasa dapat membaca dan melihat isi direktori `/root` tanpa harus mengubah kepemilikan direktori maupun memberikan akses tulis.

## Permasalahan

Salah satu pesan error yang dapat muncul ketika membuka direktori `/root` melalui Windows File Explorer adalah:

```text
\\wsl.localhost\Ubuntu\root is not accessible.

You might not have permission to use this
network resource.

Attempt to access invalid address.
```

Permasalahan ini dapat terjadi ketika:

- Ubuntu WSL dijalankan menggunakan akun Linux biasa.
- Direktori `/root` masih menggunakan permission default.
- Pengguna biasa tidak memiliki izin untuk mengakses direktori administrator.
- Windows File Explorer mencoba membaca direktori yang tidak dapat diakses oleh akun Linux tersebut.

## Mengapa Hal Ini Terjadi?

Linux menggunakan sistem permission untuk mengatur siapa yang dapat membaca, menulis, dan menjalankan atau mengakses file maupun direktori.

Contoh permission direktori `/root`:

```bash
ls -ld /root
```

Hasil:

```text
drwx------ 25 root root 4096 Nov 2 13:02 /root
```

Penjelasan:

| Komponen | Keterangan |
|---|---|
| `d` | Menunjukkan direktori |
| `rwx` | Root memiliki akses penuh |
| `---` | Group tidak memiliki akses |
| `---` | Pengguna lain tidak memiliki akses |
| `root` | Pemilik direktori |
| `root` | Group direktori |

Karena pengguna biasa tidak memiliki izin untuk melewati direktori `/root`, Windows File Explorer juga tidak dapat mengakses isi direktori tersebut melalui integrasi WSL.

## Persiapan

### 1. Pastikan menggunakan Ubuntu WSL

Buka terminal Ubuntu dan periksa pengguna yang sedang digunakan:

```bash
whoami
```

Contoh hasil:

```text
aditzero
```

Pastikan terminal tidak sedang berjalan sebagai root apabila ingin memberikan akses kepada akun pengguna biasa.

### 2. Periksa permission direktori `/root`

Jalankan:

```bash
ls -ld /root
```

Apabila hasilnya seperti berikut:

```text
drwx------ root root
```

Artinya direktori `/root` masih menggunakan permission default.

### 3. Periksa nama distribusi WSL

Buka PowerShell Windows:

```powershell
wsl --list --quiet
```

Contoh hasil:

```text
Ubuntu
docker-desktop
```

Nama distribusi diperlukan ketika mengakses folder WSL melalui Windows File Explorer.

### 4. Install ACL

Kembali ke terminal Ubuntu:

```bash
sudo apt update
sudo apt install acl -y
```

ACL merupakan singkatan dari Access Control List, yaitu mekanisme Linux untuk memberikan izin akses secara lebih spesifik kepada pengguna atau kelompok tertentu.

Periksa instalasi:

```bash
getfacl --version
```

## Langkah Penyelesaian

Tujuan konfigurasi ini adalah memberikan akses baca dan penelusuran direktori `/root` kepada pengguna biasa tanpa memberikan izin untuk membuat, mengubah, atau menghapus file.

### 1. Berikan akses pada direktori `/root`

Misalnya username Ubuntu adalah `aditzero`.

Jalankan:

```bash
sudo setfacl -m u:aditzero:rx /root
```

Penjelasan:

- `sudo`: menjalankan perintah dengan hak administrator.
- `setfacl`: mengatur Access Control List.
- `-m`: menambahkan atau mengubah aturan ACL.
- `u:aditzero`: menentukan pengguna yang mendapatkan izin.
- `rx`: memberikan izin membaca dan melewati direktori.

### 2. Berikan akses baca pada seluruh isi `/root`

Jalankan:

```bash
sudo setfacl -R -m u:aditzero:r-X /root
```

Penjelasan:

- `-R`: menerapkan konfigurasi secara rekursif.
- `r`: memberikan izin membaca file.
- `X`: memberikan izin eksekusi hanya pada direktori dan file yang sebelumnya sudah memiliki izin eksekusi.

Dengan demikian, pengguna dapat melihat dan membuka isi direktori tanpa mendapatkan izin menulis.

**Catatan:** Perintah ini memberikan akses baca secara luas terhadap isi `/root`, termasuk file tersembunyi dan file konfigurasi.

### 3. Verifikasi konfigurasi ACL

Jalankan:

```bash
getfacl /root
```

Contoh hasil yang diharapkan:

```text
# file: root
# owner: root
# group: root
user::rwx
user:aditzero:r-x
group::---
mask::r-x
other::---
```

Pastikan terdapat aturan:

```text
user:aditzero:r-x
```

## Verifikasi Akses

Setelah konfigurasi ACL selesai, buka Windows File Explorer.

Masukkan alamat berikut pada address bar:

```text
\\wsl.localhost\Ubuntu\root
```

Jika nama distribusi WSL berbeda, sesuaikan `Ubuntu` dengan nama distribusi yang digunakan.

Contoh:

```text
\\wsl.localhost\Ubuntu-22.04\root
```

Apabila konfigurasi berhasil, Windows File Explorer seharusnya sudah dapat menampilkan isi direktori `/root`.

## Troubleshooting

### 1. Perintah getfacl tidak ditemukan

Pesan error:

```text
Command 'getfacl' not found
```

Solusi:

```bash
sudo apt update
sudo apt install acl -y
```

### 2. Permission denied ketika mengakses `/root`

Periksa:

```bash
ls -ld /root
getfacl /root
```

Pastikan pengguna yang digunakan sudah mendapatkan izin `r-x`.

### 3. Windows File Explorer masih tidak bisa membuka direktori

Periksa nama distribusi:

```powershell
wsl --list --quiet
```

Pastikan alamat File Explorer menggunakan nama distribusi yang benar.

Jika masih bermasalah, coba tutup File Explorer, kemudian jalankan PowerShell:

```powershell
wsl --shutdown
```

Setelah itu buka kembali Ubuntu WSL dan File Explorer.

Perlu diketahui bahwa jika masalah masih muncul setelah permission benar, penyebabnya bisa berasal dari integrasi WSL atau Windows, bukan semata-mata permission Linux.

## Mengembalikan Permission

Jika ingin menghapus aturan ACL yang sebelumnya diberikan kepada pengguna `aditzero`, jalankan:

```bash
sudo setfacl -x u:aditzero /root
```

Kemudian hapus aturan ACL pengguna tersebut dari seluruh isi `/root`:

```bash
sudo setfacl -R -x u:aditzero /root
```

**Perhatian:** Jalankan langkah ini hanya jika aturan ACL tersebut memang dibuat untuk kebutuhan panduan ini. Perintah tersebut menghapus entri ACL milik pengguna yang disebutkan, bukan mengembalikan seluruh ACL lainnya ke kondisi sebelumnya.

## Catatan Keamanan

Penggunaan ACL memungkinkan administrator memberikan akses lebih spesifik tanpa harus mengubah kepemilikan file atau direktori.

Namun, ada beberapa hal yang harus diperhatikan:

1. Jangan menggunakan `chmod -R 777 /root`.
2. Jangan mengubah seluruh kepemilikan `/root` menjadi pengguna biasa.
3. Hindari memberikan izin tulis jika hanya membutuhkan akses baca.
4. Ingat bahwa `/root` dapat berisi file konfigurasi, kredensial, dan informasi sensitif.
5. Berikan akses hanya kepada pengguna yang memang dipercaya.
6. Gunakan ACL secara selektif apabila hanya beberapa direktori yang perlu diakses.

Untuk kebutuhan melihat isi `/root`, izin baca dan penelusuran (`r-x`) lebih sesuai dibandingkan izin penuh (`rwx`).

## Referensi

- [Microsoft Learn - Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/)
- [Microsoft Learn - WSL File System](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
- [Linux Manual - setfacl](https://man7.org/linux/man-pages/man1/setfacl.1.html)
- [Linux Manual - getfacl](https://man7.org/linux/man-pages/man1/getfacl.1.html)
- [Linux Manual - chmod](https://man7.org/linux/man-pages/man1/chmod.1.html)

---

## 👨‍💻 Kontributor

Dokumentasi ini dibuat sebagai panduan pembelajaran dan troubleshooting Linux, khususnya pada lingkungan Windows Subsystem for Linux (WSL).

**Repository:** `wsl-root-access-guide`

**Author:** Ilham Firmansyah

Jika panduan ini bermanfaat, silakan gunakan, pelajari, dan kembangkan sesuai kebutuhan.