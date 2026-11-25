# Berkontribusi (Contributing)

Kami bakal sangat senang dapat bantuan kita! Makasih ya udah peduli sama buku ini.

## Di Mana Harus Mengedit

Semua editan wajib dilakukan di dalam direktori `src`.

Direktori `nostarch` mengandung *snapshots* yang gunanya buat dikirim ke pihak penerbit 
untuk keperluan versi cetak buku (print version). File-file snapshot ini mencerminkan 
apa aja yang udah dikirim dan belum dikirim, makanya file ini cuma bakal di-*update* saat 
kami ngirim hasil editan (edits) ke pihak No Starch. **Tolong jangan submit *pull requests* 
yang ngerubah file-file di dalem direktori `nostarch` ya, PR semacam itu bakal langsung 
kami tutup (closed).**

Kita menggunakan [`rustfmt`][rustfmt] untuk menerapkan gaya pemformatan standar (standard formatting) pada kode Rust di dalam repo ini, dan kita juga menggunakan [`dprint`][dprint] untuk menerapkan pemformatan serupa pada source Markdown sekaligus kode non-Rust lainnya di proyek ini.

[rustfmt]: https://github.com/rust-lang/rustfmt
[dprint]: https://dprint.dev

Kita pada umumnya otomatis punya `rustfmt` yang sudah terpasang (_installed_) andaikata 
kita udah memasang komponen _toolchain_ Rust; tapi kalau misal kita ternyata belum 
punya salinan aplikasi `rustfmt`, kita bisa menambahkan perkakas ini dengan menjalankan 
perintah berikut:

```sh
rustup component add rustfmt
```

Buat menginstal `dprint`, kita bisa menjalankan perintah ini:

```sh
cargo install dprint
```

Atau silakan ikuti [instruksi-instruksi][install-dprint] yang terpajang di situs 
web `dprint`.

[install-dprint]: https://dprint.dev/install/

Buat memformat kode Rust, kita bisa menjalankan perintah `rustfmt <path to file>` 
(jalur ke file), dan untuk memformat file-file lainnya, kita bisa mengoper 
perintah `dprint fmt <path to file>`. Mayoritas teks _editors_ pada umumnya juga udah 
punya dukungan bawaan (*native*) maupun fitur ekstensi tambahan untuk `rustfmt` dan 
sekaligus `dprint`.

## Mengecek Perbaikan yang Sudah Ada (Checking for Fixes)

Buku ini mengikuti jadwal rilis (_release trains_) Rust. Karena itu, jika kita 
menemukan masalah di https://doc.rust-lang.org/stable/book, masalah tersebut 
mungkin sudah diperbaiki di *branch* `main` pada repo ini, hanya saja 
perbaikannya belum melewati jalur nightly -> beta -> stable. Silakan cek 
*branch* `main` di repo ini sebelum melaporkan *issue* baru.

Melihat riwayat (_history_) untuk berkas tertentu juga bisa memberi informasi 
lebih lanjut tentang apakah suatu masalah sudah diperbaiki atau belum jika kita 
sedang mencoba menelusurinya.

Mohon cari juga di *issues* (yang terbuka maupun tertutup) dan *PR* (yang 
terbuka maupun tertutup) sebelum melaporkan *issue* baru atau membuka *PR* baru.

## Lisensi (Licensing)

Repositori ini berada di bawah lisensi yang sama dengan Rust sendiri, yaitu 
MIT/Apache2. Kita bisa menemukan teks lengkap masing-masing lisensi di dalam 
berkas `LICENSE-*` di repositori ini.

## Pedoman Perilaku (Code of Conduct)

Proyek Rust memiliki [pedoman perilaku](http://rust-lang.org/policies/code-of-conduct) 
yang berlaku untuk semua sub-proyek, termasuk repositori ini. Mohon hormati 
pedoman tersebut!

## Ekspektasi (Expectations)

Karena buku ini [dicetak][nostarch], dan karena kami ingin menjaga versi 
_online_ buku ini sedekat mungkin dengan versi cetaknya jika memungkinkan, 
proses penanganan *issue* atau *pull request* kita mungkin memakan waktu lebih 
lama dari biasanya.

[nostarch]: https://nostarch.com/rust-programming-language-2nd-edition

Sejauh ini, kami melakukan revisi besar bersamaan dengan peluncuran 
[Rust Editions](https://doc.rust-lang.org/edition-guide/). Di sela-sela revisi 
besar tersebut, kami hanya memperbaiki kesalahan (error). Jika *issue* atau 
*pull request* kita bukan murni perbaikan error, kemungkinan besar baru akan 
ditangani saat kami mengerjakan revisi besar berikutnya: perkiraan waktunya 
bisa berbulan-bulan atau bertahun-tahun. Terima kasih atas kesabaran kita!

## Butuh Bantuan Tambahan (Help wanted)

Jika kita mencari cara untuk membantu yang tidak melibatkan banyak membaca atau 
menulis, lihat [*open issues* dengan label E-help-wanted][help-wanted]. Ini bisa 
berupa perbaikan kecil pada teks, kode Rust, kode *frontend*, atau *shell 
scripts* yang membantu kami bekerja lebih efisien atau meningkatkan kualitas 
buku ini dengan cara tertentu!

[help-wanted]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3AE-help-wanted

## Terjemahan-terjemahan (Translations)

Kami sangat senang menerima bantuan kita dalam menerjemahkan buku ini! Lihat 
label [Translations] untuk bergabung dengan upaya terjemahan yang sedang 
berjalan. Buka *issue* baru untuk mulai mengerjakan bahasa baru! Kami sedang 
menunggu [dukungan mdbook][mdbook support] untuk multibahasa sebelum 
menggabungkannya, tetapi silakan mulai saja!

[Translations]: https://github.com/rust-lang/book/issues?q=is%3Aopen+is%3Aissue+label%3ATranslations
[mdbook support]: https://github.com/rust-lang/mdBook/issues/5
