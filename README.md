# WSL Root Access Guide

**Panduan Mengatasi Direktori `/root` yang Tidak Bisa Diakses melalui Windows File Explorer pada Ubuntu WSL**

Repository ini berisi dokumentasi dan panduan teknis untuk mengatasi masalah ketika pengguna tidak dapat membuka direktori `/root` melalui Windows File Explorer, meskipun direktori tersebut dapat diakses melalui terminal Ubuntu menggunakan hak administrator.

Solusi yang dibahas menggunakan **Access Control List (ACL)** untuk memberikan izin baca dan penelusuran kepada pengguna Linux biasa tanpa harus mengubah kepemilikan direktori `/root`.

---

## 📑 Daftar Isi

1. [Tentang Project](#-tentang-project)
2. [Latar Belakang](#-latar-belakang)
3. [Permasalahan](#-permasalahan)
4. [Memahami Struktur Direktori Linux](#-memahami-struktur-direktori-linux)
5. [Memahami Linux Permission](#-memahami-linux-permission)
6. [Mengenal Access Control List (ACL)](#-mengenal-access-control-list-acl)
7. [Persiapan](#-persiapan)
8. [Langkah Penyelesaian](#-langkah-penyelesaian)
9. [Pengujian Melalui Terminal](#-pengujian-melalui-terminal)
10. [Pengujian Melalui Windows File Explorer](#-pengujian-melalui-windows-file-explorer)
11. [Troubleshooting](#-troubleshooting)
12. [Mengembalikan Permission](#-mengembalikan-permission)
13. [Keamanan dan Hal yang Perlu Diperhatikan](#-keamanan-dan-hal-yang-perlu-diperhatikan)
14. [Referensi](#-referensi)
15. [Kontributor](#-kontributor)

---

## 📖 Tentang Project

Pada sistem operasi Linux, direktori `/root` merupakan home directory khusus bagi pengguna administrator atau superuser.

Secara default, direktori tersebut hanya dapat diakses oleh pengguna `root`. Hal ini merupakan bagian dari mekanisme keamanan Linux untuk melindungi file konfigurasi, data, dan informasi penting milik administrator.

Namun, pada lingkungan Windows Subsystem for Linux (WSL), pengguna terkadang perlu membuka direktori tersebut melalui Windows File Explorer.

Contohnya, ketika pengguna sebelumnya menyimpan project di `/root`, kemudian beralih menggunakan akun Linux biasa untuk memenuhi kebutuhan pengembangan aplikasi atau keperluan perkuliahan.

Ketika mencoba membuka direktori berikut:

```text
\\wsl.localhost\Ubuntu\root
```

Windows File Explorer dapat menampilkan pesan bahwa direktori tidak dapat diakses.

Project ini membahas penyebab masalah tersebut dan salah satu pendekatan penyelesaiannya menggunakan Linux ACL.

### Tujuan Project

- Memahami penyebab direktori `/root` tidak dapat dibuka melalui Windows File Explorer.
- Memahami hubungan antara Linux permission dan akses file melalui WSL.
- Mengenal penggunaan Access Control List (ACL).
- Memberikan akses baca kepada pengguna Linux biasa.
- Mempertahankan kepemilikan direktori oleh pengguna `root`.
- Menghindari penggunaan permission yang terlalu terbuka seperti `777`.
- Menyediakan panduan troubleshooting yang dapat digunakan kembali oleh pengguna WSL lainnya.

---

## 🧩 Latar Belakang

Windows Subsystem for Linux (WSL) memungkinkan pengguna menjalankan lingkungan Linux secara langsung pada Windows.

Salah satu fitur WSL adalah kemampuan mengakses filesystem Linux melalui Windows File Explorer.

Biasanya, direktori Linux dapat diakses melalui alamat:

```text
\\wsl.localhost\<NamaDistribusi>\
```

Sebagai contoh:

```text
\\wsl.localhost\Ubuntu\
```

Dari lokasi tersebut, pengguna dapat melihat direktori Linux seperti:

```text
/
├── bin/
├── boot/
├── dev/
├── etc/
├── home/
├── root/
├── usr/
└── var/
```

Namun, tidak semua direktori dapat diakses oleh setiap pengguna.

Direktori `/root` secara default memiliki pembatasan akses yang ketat. Oleh karena itu, pengguna biasa mungkin dapat membuka direktori home miliknya sendiri, tetapi tidak dapat membuka direktori `/root`.

---

## ❗ Permasalahan

Contoh pesan error yang dapat muncul pada Windows File Explorer:

```text
\\wsl.localhost\Ubuntu\root is not accessible.

You might not have permission to use this
network resource.

Attempt to access invalid address.
```

Permasalahan ini dapat terjadi ketika:

1. Pengguna sebelumnya menjalankan aktivitas pengembangan project menggunakan akun `root`.
2. Pengguna kemudian beralih menggunakan akun Linux biasa.
3. Project atau file masih berada di dalam direktori `/root`.
4. Windows File Explorer mencoba mengakses direktori tersebut.
5. Linux menolak akses karena pengguna biasa tidak memiliki permission yang diperlukan.

### Contoh Kondisi Pengguna

Misalnya, pengguna memiliki dua akun atau lingkungan direktori:

```text
/root/
    project-lama/

/home/<username>/
    project-mobile/
```

Pengguna dapat menjalankan perintah melalui terminal dengan bantuan `sudo`, tetapi ketika mencoba membuka `/root` melalui Windows File Explorer, akses ditolak.

Hal ini bukan berarti project hilang atau filesystem rusak. Salah satu kemungkinan penyebabnya adalah pembatasan permission Linux.

---

## 📂 Memahami Struktur Direktori Linux

Sebelum melakukan perubahan permission, penting untuk memahami perbedaan antara direktori `/root` dan `/home`.

### 1. Direktori `/root`

Direktori `/root` merupakan home directory khusus administrator Linux.

Contoh:

```text
/root/
├── .bashrc
├── .profile
├── project-lama/
└── konfigurasi/
```

Direktori ini secara default hanya dapat diakses oleh root.

### 2. Direktori `/home`

Direktori `/home` merupakan tempat penyimpanan home directory pengguna biasa.

Contoh:

```text
/home/
└── <username>/
    ├── Documents/
    ├── Downloads/
    ├── Projects/
    └── project-mobile/
```

Pengguna biasa biasanya memiliki hak akses penuh terhadap home directory miliknya sendiri.

### 3. Perbedaan Keduanya

| Direktori | Fungsi |
|---|---|
| `/root` | Home directory administrator |
| `/home/<username>` | Home directory pengguna biasa |
| `/tmp` | Penyimpanan file sementara |
| `/usr` | Program dan komponen sistem |
| `/etc` | Konfigurasi sistem |

**Catatan:** Project yang digunakan untuk pengembangan sehari-hari umumnya lebih disarankan disimpan di home directory pengguna biasa. Namun, apabila project memang harus tetap berada di `/root`, aksesnya perlu diatur dengan memperhatikan keamanan.

---

## 🔐 Memahami Linux Permission

Linux memiliki sistem permission untuk menentukan hak akses pengguna terhadap file dan direktori.

Untuk memeriksa permission `/root`, jalankan:

```bash
ls -ld /root
```

Contoh hasil:

```text
drwx------ 25 root root 4096 Nov 2 13:02 /root
```

Penjelasan:

| Bagian | Arti |
|---|---|
| `d` | Menunjukkan bahwa objek merupakan direktori |
| `rwx` | Pemilik memiliki izin baca, tulis, dan eksekusi |
| `---` | Group tidak memiliki izin |
| `---` | Pengguna lainnya tidak memiliki izin |
| `root` | Pemilik direktori |
| `root` | Group direktori |

### Arti Permission

| Simbol | Nama | Fungsi |
|---|---|---|
| `r` | Read | Membaca file atau melihat daftar isi direktori |
| `w` | Write | Mengubah file atau isi direktori |
| `x` | Execute | Menjalankan file atau melewati direktori |

Khusus direktori, permission `x` sangat penting karena dibutuhkan untuk menelusuri atau memasuki direktori tersebut.

Contohnya, pengguna dapat diberikan izin `r-x` agar dapat membaca daftar isi dan melewati direktori tanpa mendapatkan izin untuk mengubahnya.

---

## 🛠️ Mengenal Access Control List (ACL)

Access Control List (ACL) merupakan mekanisme Linux yang memungkinkan administrator memberikan permission secara lebih spesifik kepada pengguna atau group tertentu.

Tanpa ACL, permission Linux standar umumnya diatur berdasarkan:

- Owner (pemilik).
- Group (kelompok).
- Others (pengguna lainnya).

Dengan ACL, administrator dapat memberikan izin tambahan kepada pengguna tertentu tanpa harus mengubah kepemilikan file atau direktori.

### Contoh Permission ACL

```text
user::rwx
user:<username>:r-x
group::---
mask::r-x
other::---
```

Artinya, pengguna dengan nama yang ditentukan memperoleh izin baca dan penelusuran, sementara pemilik tetap root.

### Perintah ACL yang Digunakan

| Perintah | Fungsi |
|---|---|
| `setfacl` | Mengatur ACL |
| `getfacl` | Melihat konfigurasi ACL |
| `-m` | Menambahkan atau mengubah aturan |
| `-x` | Menghapus aturan ACL tertentu |
| `-R` | Menerapkan perubahan secara rekursif |
| `-d` | Mengatur default ACL |
| `-b` | Menghapus seluruh entri ACL tambahan |

---

## ⚙️ Persiapan

Sebelum melakukan konfigurasi, pastikan beberapa hal berikut:

- Ubuntu WSL sudah terpasang.
- Pengguna dapat membuka terminal Ubuntu.
- Pengguna mengetahui username Linux yang digunakan.
- Pengguna memiliki akses administrator melalui `sudo`.
- Windows File Explorer dapat mengakses filesystem WSL lainnya.

### 1. Periksa Username Linux

Jalankan:

```bash
whoami
```

Contoh hasil:

```text
ilhamfirmansyah
```

Username yang ditampilkan akan digunakan pada konfigurasi ACL.

Dalam panduan ini, `<username>` merupakan placeholder. Ganti dengan username Linux milik masing-masing pengguna.

Contoh:

```bash
sudo setfacl -m u:ilhamfirmansyah:rx /root
```

Jangan mengetik tanda `<` dan `>` secara literal ketika menjalankan perintah.

### 2. Periksa Direktori `/root`

Jalankan:

```bash
ls -ld /root
```

Periksa apakah permission masih menunjukkan:

```text
drwx------
```

### 3. Periksa Nama Distribusi WSL

Buka PowerShell Windows:

```powershell
wsl --list --quiet
```

Contoh hasil:

```text
Ubuntu
```

Nama distribusi tersebut akan digunakan ketika membuka filesystem melalui Windows File Explorer.

### 4. Install ACL

Buka terminal Ubuntu dan jalankan:

```bash
sudo apt update
sudo apt install acl -y
```

Periksa apakah instalasi berhasil:

```bash
getfacl --version
```

Apabila versi ACL berhasil ditampilkan, berarti utilitas sudah tersedia.

---

## 🚀 Langkah Penyelesaian

Pada bagian ini, kita akan memberikan akses baca dan penelusuran kepada pengguna biasa agar dapat membuka direktori `/root` melalui Windows File Explorer.

**Target konfigurasi:**

- Owner `/root` tetap `root`.
- Pengguna biasa dapat membaca isi direktori.
- Pengguna biasa dapat menelusuri direktori.
- Pengguna biasa tidak mendapatkan izin menulis.
- Pengguna biasa tidak mendapatkan izin menghapus file.

### 1. Berikan Permission pada Direktori `/root`

Jalankan:

```bash
sudo setfacl -m u:<username>:rx /root
```

Contoh:

```bash
sudo setfacl -m u:ilhamfirmansyah:rx /root
```

Penjelasan:

- `sudo`: menjalankan perintah sebagai administrator.
- `setfacl`: mengatur Access Control List.
- `-m`: menambahkan atau mengubah aturan ACL.
- `u:<username>`: menentukan pengguna yang diberikan akses.
- `rx`: memberikan izin membaca dan menelusuri direktori.

Dengan konfigurasi ini, pengguna dapat melewati direktori `/root` dan membaca daftar isinya.

### 2. Berikan Permission pada Isi Direktori `/root`

Selanjutnya, jalankan:

```bash
sudo setfacl -R -m u:<username>:r-X /root
```

Contoh:

```bash
sudo setfacl -R -m u:ilhamfirmansyah:r-X /root
```

Penjelasan:

- `-R`: menerapkan aturan secara rekursif.
- `r`: memberikan izin membaca.
- `X`: memberikan izin eksekusi pada direktori dan file yang sebelumnya sudah memiliki izin eksekusi.

Dengan demikian, pengguna dapat membaca file dan menelusuri direktori di dalam `/root`.

**Penting:** Perintah ini memberikan akses baca secara luas ke seluruh isi `/root`, termasuk file tersembunyi dan file konfigurasi. Pastikan memang ingin membuka seluruh isinya sebelum menjalankan perintah ini.

### 3. Periksa Hasil Konfigurasi

Jalankan:

```bash
getfacl /root
```

Contoh hasil:

```text
# file: root
# owner: root
# group: root
user::rwx
user:ilhamfirmansyah:r-x
group::---
mask::r-x
other::---
```

Pastikan terdapat baris:

```text
user:<username>:r-x
```

Periksa juga salah satu direktori di dalam `/root`:

```bash
getfacl /root/<nama-direktori>
```

Pastikan pengguna memperoleh izin yang sesuai.

---

## 🧪 Pengujian Melalui Terminal

Setelah konfigurasi ACL selesai, lakukan beberapa pengujian.

### 1. Uji Akses sebagai Pengguna Biasa

Jalankan:

```bash
sudo -u <username> ls -la /root
```

Contoh:

```bash
sudo -u ilhamfirmansyah ls -la /root
```

Apabila berhasil, daftar isi direktori `/root` akan ditampilkan.

### 2. Uji Akses ke Subdirektori

Misalnya terdapat direktori bernama `perkuliahan`:

```bash
sudo -u <username> ls -la /root/perkuliahan
```

Pengguna seharusnya dapat melihat daftar project di dalamnya.

### 3. Uji Bahwa Akses Tulis Tidak Diberikan

Untuk memastikan pengguna tidak dapat membuat file baru di `/root`, lakukan pengujian:

```bash
sudo -u <username> touch /root/permission-test
```

Dengan konfigurasi baca saja, perintah tersebut seharusnya menghasilkan:

```text
Permission denied
```

Pengujian ini membuktikan bahwa pengguna memperoleh akses baca, bukan akses penuh.

---

## 🖥️ Pengujian Melalui Windows File Explorer

Setelah konfigurasi permission selesai, buka Windows File Explorer.

### 1. Buka File Explorer

Tekan:

```text
Win + E
```

### 2. Masukkan Alamat WSL

Pada address bar, masukkan:

```text
\\wsl.localhost\Ubuntu\root
```

Apabila nama distribusi WSL berbeda, sesuaikan bagian `Ubuntu`.

Contoh:

```text
\\wsl.localhost\Ubuntu-22.04\root
```

### 3. Periksa Isi Direktori

Jika konfigurasi berhasil, Windows File Explorer seharusnya dapat menampilkan isi direktori `/root`.

Pengguna dapat melihat folder project dan file lainnya sesuai permission yang diberikan.

---

## 🔧 Troubleshooting

Bagian ini membahas beberapa masalah yang mungkin muncul selama proses konfigurasi.

### 1. Perintah `getfacl` Tidak Ditemukan

Pesan error:

```text
Command 'getfacl' not found
```

Penyebab:

Utilitas ACL belum terpasang.

Solusi:

```bash
sudo apt update
sudo apt install acl -y
```

### 2. Permission Denied Ketika Membuka `/root`

Periksa permission:

```bash
ls -ld /root
```

Kemudian periksa ACL:

```bash
getfacl /root
```

Pastikan username sudah memperoleh izin `r-x`.

Jika belum, jalankan kembali:

```bash
sudo setfacl -m u:<username>:rx /root
```

### 3. Pengguna Bisa Membuka `/root`, tetapi Tidak Bisa Membuka Subdirektori

Penyebab:

Direktori di dalam `/root` mungkin memiliki permission yang membatasi akses.

Periksa:

```bash
getfacl /root/<nama-direktori>
```

Jika memang diperlukan, berikan izin baca dan penelusuran pada direktori tersebut secara selektif.

Contoh:

```bash
sudo setfacl -m u:<username>:r-x /root/<nama-direktori>
```

Perlu diingat bahwa setiap direktori induk dalam jalur akses juga harus mengizinkan penelusuran.

### 4. Windows File Explorer Masih Menampilkan Error

Pastikan nama distribusi WSL benar:

```powershell
wsl --list --quiet
```

Periksa kembali alamat yang digunakan:

```text
\\wsl.localhost\<NamaDistribusi>\root
```

Jika permission Linux sudah benar tetapi File Explorer masih gagal, coba tutup File Explorer dan jalankan PowerShell:

```powershell
wsl --shutdown
```

Setelah itu, buka kembali Ubuntu WSL dan coba akses melalui File Explorer.

Perlu diperhatikan bahwa `wsl --shutdown` akan menghentikan sementara seluruh distribusi WSL yang sedang berjalan.

### 5. ACL Sudah Benar, tetapi Akses Tetap Ditolak

Periksa seluruh komponen jalur direktori:

```bash
namei -l /root
```

Kemudian periksa ACL:

```bash
getfacl /root
```

Periksa juga permission dan ACL pada subdirektori yang ingin dibuka.

Perlu diingat bahwa ACL memiliki permission mask. Meskipun suatu entri menunjukkan `rwx`, hak akses efektifnya dapat dibatasi oleh mask.

---

## 🔄 Mengembalikan Permission

Apabila suatu saat pengguna tidak lagi membutuhkan akses ke `/root`, aturan ACL yang sebelumnya dibuat dapat dihapus.

### 1. Hapus ACL dari Direktori `/root`

```bash
sudo setfacl -x u:<username> /root
```

### 2. Hapus ACL dari Isi Direktori `/root`

```bash
sudo setfacl -R -x u:<username> /root
```

### 3. Verifikasi

```bash
getfacl /root
```

Pastikan entri pengguna yang sebelumnya ditambahkan sudah tidak ada.

**Catatan:** Perintah tersebut menghapus entri ACL untuk pengguna yang disebutkan, bukan mengembalikan seluruh konfigurasi ACL ke kondisi awal. Jika sebelumnya sudah terdapat aturan ACL milik pengguna tersebut, catat konfigurasi awal sebelum melakukan perubahan.

---

## 🔒 Keamanan dan Hal yang Perlu Diperhatikan

Direktori `/root` merupakan lokasi penting dalam sistem Linux.

Di dalamnya dapat terdapat:

- File konfigurasi administrator.
- SSH private key.
- Credential aplikasi.
- Environment variable.
- File konfigurasi database.
- Source code dan project pribadi.
- File tersembunyi lainnya.

Oleh karena itu, pemberian akses ke direktori `/root` harus dilakukan dengan hati-hati.

### Rekomendasi Keamanan

1. Gunakan ACL untuk mengatur akses pengguna secara spesifik.
2. Hindari menggunakan `chmod -R 777 /root`.
3. Jangan mengubah kepemilikan seluruh direktori `/root` menjadi pengguna biasa.
4. Jangan memberikan akses tulis jika hanya membutuhkan akses baca.
5. Hindari membuka file yang berisi credential atau informasi rahasia.
6. Jika hanya membutuhkan satu project, berikan akses ke direktori project tersebut saja.
7. Gunakan home directory pengguna biasa untuk project development sehari-hari jika tidak ada kebutuhan khusus untuk menyimpannya di `/root`.

### Perbedaan Akses

| Permission | Fungsi |
|---|---|
| `r-x` | Membaca dan menelusuri direktori |
| `rwX` | Membaca, menulis, dan menelusuri direktori serta file sesuai ketentuan `X` |
| `rwx` | Akses penuh pada objek yang diberikan permission tersebut |

Untuk kebutuhan melihat isi `/root` melalui File Explorer, akses baca dan penelusuran lebih sesuai dibandingkan memberikan akses penuh.

---

## 📚 Referensi

Dokumentasi resmi dan manual yang digunakan sebagai referensi:

1. [Microsoft Learn - Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/)
2. [Microsoft Learn - Working Across Windows and Linux File Systems](https://learn.microsoft.com/en-us/windows/wsl/filesystems)
3. [Linux Manual - setfacl](https://man7.org/linux/man-pages/man1/setfacl.1.html)
4. [Linux Manual - getfacl](https://man7.org/linux/man-pages/man1/getfacl.1.html)
5. [Linux Manual - chmod](https://man7.org/linux/man-pages/man1/chmod.1.html)

---

## 👨‍💻 Kontributor

**Ilham Firmansyah**

Repository ini dibuat sebagai dokumentasi pembelajaran dan troubleshooting Linux, khususnya mengenai pengelolaan permission pada Windows Subsystem for Linux (WSL).

Tujuannya adalah membantu pengguna memahami mekanisme permission Linux dan menyelesaikan masalah akses direktori `/root` secara lebih aman.

Jika dokumentasi ini bermanfaat, silakan gunakan, pelajari, dan kembangkan sesuai kebutuhan.

---

**Made for learning, troubleshooting, and sharing knowledge.**