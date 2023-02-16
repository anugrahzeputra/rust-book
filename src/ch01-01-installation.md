## Pemasangan

Langkah pertama adalah memasang Rust. Kita akan mengunduh Rust menggunakan
`rustup`, sebuah aplikasi baris perintah yg digunakan untuk mengelola versi Rust
dan aplikasi terkait lainnya. Anda membutuhkan koneksi internet untuk
mengunduhnya.

> Catatan: Jika anda memilih untuk tidak menggunakan `rustup` karena suatu
> alasan tertentu, silakan baca
> [halaman Metode Pemasangan Rust Lainnya][otherinstall] untuk pilihan lebih
> lanjut.

Langkah selanjutnya memasang versi stable dari compiler Rust. Jaminan
stabilitas Rust memastikan bahwa semua contoh di buku ini yg bisa di compile
akan tetap bisa di compile dengan versi Rust yg lebih baru. Hasil Output
mungkin berbeda sedikit antarversi karena Rust sering kali memperbaiki pesan
error dan peringatan. Dengan kata lain, semua versi stable Rust yg lebih baru,
yg anda install menggunakan metode berikut, seharusnya bekerja sesuai harapan
dg isi buku ini.

> ### Notasi Baris Perintah
>
> Pada bab ini dan keseluruhan buku, kita akan tunjukkan beberapa perintah yg
> digunakan pada terminal. Baris yg seharusnya anda masukkan di terminal semua
> diawali dg `$`. Anda tidak perlu mengetikkan karakter `$`; Itu adalah penanda
> baris perintah yg ditampilkan untuk menunjukkan awal dari tiap perintah.
> Baris yg tidak dimulai dg `$` biasanya menampilkan output dari perintah
> sebelumnya. Sebagai tambahan, contoh khusus PowerShell akan menggunakan `>`
> daripada `$`.

### Pemasangan `rustup` pada Linux atau macOS

Jika anda menggunakan Linux atau macOS, buka sebuah terminal dan masukkan
perintah berikut:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Perintah tersebut akan mengunduh sebuah script dan memulai pemasangan aplikasi
`rustup`, dan memasang versi stable terakhir Rust. Anda mungkin akan diminta
untuk memasukkan kata kunci. Jika pemasangan sukses, baris berikut akan muncul:

```text
Rust is installed now. Great!
```

Anda juga akan membutuhkan sebuah *linker*, suatu program yg digunakan Rust
untuk menggabungkan hasil kompilasi program menjadi satu berkas. Kemungkinan
besar anda sudah memasangnya. Jika anda mendapat error linker, sebaiknya anda
pasang sebuah compiler C, biasanya itu sudah termasuk dg linker. Compiler C
juga berguna karena beberapa paket Rust bergantung pada kode C dan akan
membutuhkan compiler C.

Di macOS, anda memasang compiler C dg menjalankan:

```console
$ xcode-select --install
```

Pengguna Linux biasanya memasang GCC atau Clang, sesuai dg dokumentasi
masing-masing distribusi. Sebagai contoh, jika anda menggunakan Ubuntu, anda
bisa memasang paket `build-essential`.

### Memasang `rustup` pada Windows

Untuk windows, silakan baca [https://www.rust-lang.org/tools/install][install]
dan ikuti instruksi untuk memasang Rust. Pada satu titik di pemasangannya, anda
akan mendapat sebuah pesan yg menjelaskan bahwa anda juga memerlukan
MSVC build tools untuk Visual Studio 2013 dan sesudahnya.

Untuk mendapatkan build tools tersebut, anda perlu memasang [Visual Studio
2022][visualstudio]. Ketika ditanya workloads yg hendak dipasang, masukkan:

* “Desktop Development with C++”
* The Windows 10 or 11 SDK
* The English language pack component, beserta paket bahasa lain yg anda pilih

Untuk selanjutnya buku ini menggunakan perintah yg bekerja baik di *cmd.exe*
maupun PowerShell. Jika ada perbedaan spesifik, kita akan menjelaskan mana yg
seharusnya digunakan.

### Pemecah Masalah

Untuk mengecek apakah anda memiliki Rust yg terpasang dg baik, buka terminal
dan masukkan:

```console
$ rustc --version
```

Seharusnya keluar nomor versi, hash commit, dan tanggal commit untuk versi
stable terakhir yg telah diterbitkan, dalam format:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Jika anda mendapatkan informasi tersebut, berarti anda telah berhasil memasang
Rust! Jika anda tidak bertemu dg informasi tersebut, silakan cek apakah Rust
berada di variabel sistem `%PATH%` sebagai berikut:

Di Windows CMD, gunakan:

```console
> echo %PATH%
```

Di PowerShell, gunakan:

```powershell
> echo $env:Path
```

Di Linux dan macOS, gunakan:

```console
$ echo $PATH
```

Jika semuanya sudah benar tetapi Rust masih belum bekerja, ada beberapa tempat
dimana anda bisa mencari bantuan. Cari tahu bagaimana berhubungan dg Rustacean
(sebutan bagi kita) lainnya di [halaman komunitas][community].

### Memperbarui dan Menghapus

Begitu Rust terpasang melalui `rustup`, memperbarui ke versi terbaru menjadi
lebih mudah. Dari terminal anda, jalankan perintah berikut:

```console
$ rustup update
```

Untuk menghapus Rust dan `rustup`, jalankan perintah berikut dari terminal
anda:

```console
$ rustup self uninstall
```

### Dokumentasi Lokal

Pemasangan Rust juga menyertakan salinan dokumentasi lokal jadi anda dapat
membacanya secara luring. Jalankan `rustup doc` untuk membuka dokumentasi lokal
di peramban anda.

Setiap kali ada sebuah tipe atau fungsi yg tersedia di library std dan anda
tidak paham mengenai apa dan bagaimana cara menggunakannya, gunakan dokumentasi
API (Application Programming Interface) untuk mencari tahu!

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[visualstudio]: https://visualstudio.microsoft.com/downloads/
[community]: https://www.rust-lang.org/community
