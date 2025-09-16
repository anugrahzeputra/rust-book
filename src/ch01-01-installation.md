## Instalasi

Langkah pertama yang kita lakukan adalah instalasi bahasa Rust. Kita akan men-
download Rust melalui `rustup`, tool _command line_ ini digunakan untuk mengatur 
versi dari Rust dan beberapa _tool - tool_ yang ada didalamnya. untuk instalasi ini 
kita membutuhkan koneksi internet untuk mendownloadnya.

> Catatan: Jika kamu lebih memilih untuk tidak menggunakan `rustup` untuk beberapa
> alasan, kamu bisa merujuk pada [Halaman Instalasi Rust Lainnya][otherinstall]
> untuk opsi lainnya

Langkah selanjutnya memasang versi stable dari compiler Rust. Jaminan
stabilitas Rust memastikan bahwa semua contoh di buku ini yg bisa di compile
akan tetap bisa di compile dengan versi Rust yg lebih baru. Hasil Output
mungkin berbeda sedikit antarversi karena Rust sering kali memperbaiki pesan
error dan peringatan. Dengan kata lain, semua versi stable Rust yg lebih baru,
yg kita install menggunakan metode berikut, seharusnya bekerja sesuai harapan
dg isi buku ini.

> ### Notasi Baris Perintah
>
> Pada bab ini dan keseluruhan buku, kita akan tunjukkan beberapa perintah yg
> digunakan pada terminal. Baris yg seharusnya kita masukkan di terminal semua
> diawali dg `$`. kita tidak perlu mengetikkan karakter `$`; Itu adalah penanda
> baris perintah yg ditampilkan untuk menunjukkan awal dari tiap perintah.
> Baris yg tidak dimulai dg `$` biasanya menampilkan output dari perintah
> sebelumnya. Sebagai tambahan, contoh khusus PowerShell akan menggunakan `>`
> daripada `$`.

### Pemasangan `rustup` pada Linux atau macOS

Jika kita menggunakan Linux atau macOS, buka sebuah terminal dan masukkan
perintah berikut:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Perintah tersebut akan mengunduh sebuah script dan memulai pemasangan aplikasi
`rustup`, dan memasang versi stable terakhir Rust. Kita mungkin akan diminta
untuk memasukkan kata kunci. Jika pemasangan sukses, baris berikut akan muncul:

```text
Rust is installed now. Great!
```

You will also need a _linker_, which is a program that Rust uses to join its
compiled outputs into one file. It is likely you already have one. If you get
linker errors, you should install a C compiler, which will typically include a
linker. A C compiler is also useful because some common Rust packages depend on
C code and will need a C compiler.

Di macOS, kita memasang compiler C dg menjalankan:

```console
$ xcode-select --install
```

Pengguna Linux biasanya memasang GCC atau Clang, sesuai dg dokumentasi
masing-masing distribusi. Sebagai contoh, jika kita menggunakan Ubuntu, kita
bisa memasang paket `build-essential`.

### Memasang `rustup` pada Windows

On Windows, go to [https://www.rust-lang.org/tools/install][install] and follow
the instructions for installing Rust. At some point in the installation, you’ll
be prompted to install Visual Studio. This provides a linker and the native
libraries needed to compile programs. If you need more help with this step, see
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]

The rest of this book uses commands that work in both _cmd.exe_ and PowerShell.
If there are specific differences, we’ll explain which to use.

### Pemecah Masalah

Untuk mengecek apakah kita memiliki Rust yg terpasang dg baik, buka terminal
dan masukkan:

```console
$ rustc --version
```

Seharusnya keluar nomor versi, hash commit, dan tanggal commit untuk versi
stable terakhir yg telah diterbitkan, dalam format:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Jika kita mendapatkan informasi tersebut, berarti kita telah berhasil memasang
Rust! Jika kita tidak bertemu dg informasi tersebut, silakan cek apakah Rust
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
dimana kita bisa mencari bantuan. Cari tahu bagaimana berhubungan dg Rustacean
(sebutan bagi kita) lainnya di [halaman komunitas][community].

### Memperbarui dan Menghapus

Begitu Rust terpasang melalui `rustup`, memperbarui ke versi terbaru menjadi
lebih mudah. Dari terminal kita, jalankan perintah berikut:

```console
$ rustup update
```

Untuk menghapus Rust dan `rustup`, jalankan perintah berikut dari terminal
kita:

```console
$ rustup self uninstall
```

### Dokumentasi Lokal

Pemasangan Rust juga menyertakan salinan dokumentasi lokal jadi kita dapat
membacanya secara luring. Jalankan `rustup doc` untuk membuka dokumentasi lokal
di peramban kita.

Setiap kali ada sebuah tipe atau fungsi yg tersedia di library std dan kita
tidak paham mengenai apa dan bagaimana cara menggunakannya, gunakan dokumentasi
API (Application Programming Interface) untuk mencari tahu!

### Text Editors and Integrated Development Environments

This book makes no assumptions about what tools you use to author Rust code.
Just about any text editor will get the job done! However, many text editors and
integrated development environments (IDEs) have built-in support for Rust. You
can always find a fairly current list of many editors and IDEs on [the tools
page][tools] on the Rust website.

### Working Offline with This Book

In several examples, we will use Rust packages beyond the standard library. To
work through those examples, you will either need to have an internet connection
or to have downloaded those dependencies ahead of time. To download the
dependencies ahead of time, you can run the following commands. (We’ll explain
what `cargo` is and what each of these commands does in detail later.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

This will cache the downloads for these packages so you will not need to
download them later. Once you have run this command, you do not need to keep the
`get-dependencies` folder. If you have run this command, you can use the
`--offline` flag with all `cargo` commands in the rest of the book to use these
cached versions instead of attempting to use the network. 

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
