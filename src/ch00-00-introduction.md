# Pengenalan

> Catatan: Buku ini adalah buku yg sama dengan [The Rust Programming
> Language][nsprust] yg tersedia dalam format cetak ataupun digital pada [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

Selamat datang di _The Rust Programming Language_, sebuah buku pengantar tentang Rust.
Bahasa pemrograman Rust membantu kita menulis perangkat lunak yang lebih cepat dan andal.
Ergonomi tingkat tinggi dan kontrol tingkat rendah sering kali dianggap berlawanan
dalam desain bahasa pemrograman; Rust menantang konflik tersebut. Dengan menyeimbangkan
kapasitas teknis yang kuat dan pengalaman pengembang yang menyenangkan, Rust memberi kita
opsi untuk mengendalikan detail tingkat rendah (seperti penggunaan memori) tanpa semua
kerumitan yang biasanya terkait dengan kontrol semacam itu.

## Rust untuk siapa

Secara ideal Rust ditujukan untuk seluruh kelompok. Mari kita lihat pada beberapa
kelompok besar.

### Tim Pengembang

Rust menyediakan peralatan produktif untuk kolaborasi tim pengembang dg tingkat
pemahaman yg bervariasi. Penggunaan kode tingkat-rendah menjadikan resiko mudah
terpapar dg berbagai jenis bug, dimana pada bahasa lain terdeteksi setelah
melalui uji ekstensif dan kode yg diaudit oleh pengembang yg berpengalaman.
Pada Rust, compiler berperan sebagai penjaga gerbang yg akan menolak memproses
kode dg bug, termasuk bug concurrency. Dengan cara ini, tim bisa fokus
menggunakan waktunya untuk logika pemrograman daripada mencari bug.

Rust juga menyediakan peralatan pengembang secara sistemik:

- Cargo, the included dependency manager and build tool, makes adding,
  compiling, and managing dependencies painless and consistent across the Rust
  ecosystem.
- The Rustfmt formatting tool ensures a consistent coding style across
  developers.
- The rust-analyzer powers Integrated Development Environment (IDE)
  integration for code completion and inline error messages.

Dengan menggunakan ini dan aplikasi lainnya di ekosistem Rust, pengembang dapat
lebih produktif ketika menulis kode tingkat-sistem.

### Pelajar

Rust sangat cocok untuk pelajar dan mereka yg tertarik dalam mempelajari konsep
sistem. Melalui Rust, banyak orang yg belajar mengenai topik pengembangan OS.
Komunitas menyambut baik pertanyaan dari para pelajar. Melalui buku ini, Tim
Rust berharap agar konsep sistem dapat diakses lebih banyak orang, terutama
mereka yg masih baru didalam pemrograman.

### Perusahaan

Ratusan perusahaan, dari kecil hingga besar, menggunakan Rust untuk berbagai
macam keperluan, termasuk Aplikasi Terminal, Layanan Web, Aplikasi Pengembang,
Piranti Embedded, Konversi Analisa Video Audio, Mata Uang Kripto,
Bioinformatika, Mesin Pencari, Aplikasi Iot, Mesin Pintar, dan beberapa bagian
dari aplikasi peramban Firefox.

### Pengembang Sumber Terbuka

Rust diperuntukkan bagi mereka yg ingin secara manual membuat bahasa
pemrograman Rust, komunitas, aplikasi pengembang, dan modul-modul. Kita
menunggu kontribusi kita pada bahasa Rust.

### Mereka yg fokus pada Kecepatan dan Stabilitas

Rust is for people who crave speed and stability in a language. By speed, we
mean both how quickly Rust code can run and the speed at which Rust lets you
write programs. The Rust compiler’s checks ensure stability through feature
additions and refactoring. This is in contrast to the brittle legacy code in
languages without these checks, which developers are often afraid to modify. By
striving for zero-cost abstractions—higher-level features that compile to
lower-level code as fast as code written manually—Rust endeavors to make safe
code be fast code as well.

The Rust language hopes to support many other users as well; those mentioned
here are merely some of the biggest stakeholders. Overall, Rust’s greatest
ambition is to eliminate the trade-offs that programmers have accepted for
decades by providing safety _and_ productivity, speed _and_ ergonomics. Give
Rust a try and see if its choices work for you.

## Buku ini ditujukan untuk Siapa

This book assumes that you’ve written code in another programming language but
doesn’t make any assumptions about which one. We’ve tried to make the material
broadly accessible to those from a wide variety of programming backgrounds. We
don’t spend a lot of time talking about what programming _is_ or how to think
about it. If you’re entirely new to programming, you would be better served by
reading a book that specifically provides an introduction to programming.

## Bagaimana cara menggunakan Buku ini

Secara umum, buku ini mengasumsikan kita membacanya secara berurutan dari depan
ke belakang. Bab selanjutnya mendasarkan konsepnya pada Bab sebelumnya, dan Bab
awal tidak akan menjelaskan secara detil mengenai topik tertentu tetapi akan
menjelaskan lebih lanjut topik tersebut pada Bab selanjutnya.

You’ll find two kinds of chapters in this book: concept chapters and project
chapters. In concept chapters, you’ll learn about an aspect of Rust. In project
chapters, we’ll build small programs together, applying what you’ve learned so
far. Chapters 2, 12, and 21 are project chapters; the rest are concept chapters.

Bab 1 menjelaskan bagaimana memasang Rust, bagaimana membuat program
"Halo, dunia!", dan bagaimana cara menggunakan aplikasi Cargo, yg berfungsi
sebagai pengelola paket dan aplikasi compiler. Bab 2 mengenai menulis program
secara komprehensif di Rust, kita akan membuat permainan tebak-menebak.
Dititik ini kita membahas konsep pada tingkat yg relatif tinggi dan pada bab
selanjutnya detil tambahan akan dielaborasi.

Jika kita ingin segera masuk keproses koding, Bab 2 adalah tempat yg paling
cocok untuk itu. Bab 3 membahas fitur Rust yg mirip dg bahasa pemrograman
lainnya, dan di Bab 4 kita akan mempelajari Sistem Kepemilikan Rust. Jika kita
pelajar yg perfeksionis yg lebih memilih mempelajari setiap detilnya sebelum
berpindah ke selanjutnya, mungkin lebih baik kita lewati Bab 2 dan langsung ke
Bab 3, baru kembali ke Bab 2 jika kita ingin mengerjakan suatu proyek, untuk
menerapkan ilmu yg sudah kita pelajari sebelumnya.

Bab 5 berbicara mengenai struct dan method, dan Bab 6 membahas enum, ungkapan
`match`, dan pembuatan aliran kontrol `if let`. Kita akan menggunakan struct
dan enum untuk membuat suatu tipe tertentu di Rust.

Di Bab 7, kita akan mempelajari Sistem Modul Rust dan aturan privasi untuk
mengelola kode kita dan API publik nya yg terkait. Bab 8 berbicara mengenai
struktur data jamak yg tersedia pada library std, seperti vektor, string, dan
hash map. Bab 9 menjelajahi teknik dan filosofi Rust terhadap penanganan Error.

Bab 10 menggali lebih dalam mengenai generic, trait, dan lifetime, yg akan
memberikan kita kemudahan dalam membuat kode yg akan diterapkan pada beberapa
tipe. Bab 11 mengenai Pengujian, dimana bahkan dg jaminan keamanan dari Rust,
tetap dibutuhkan untuk menjaga logika program kita bekerja dg benar. Di Bab 12,
kita akan mencoba membuat implementasi dari subset fungsionalitas aplikasi
baris perintah `grep` yg melakukan pencarian didalam berkas. Untuk keperluan
ini, kita akan menggunakan banyak konsep yg telah dibahas pada Bab-bab
sebelumnya.

In Chapter 16, we’ll walk through different models of concurrent programming and
talk about how Rust helps you to program in multiple threads fearlessly. In
Chapter 17, we build on that by exploring Rust’s async and await syntax, along
with tasks, futures, and streams, and the lightweight concurrency model they
enable.

Chapter 18 looks at how Rust idioms compare to object-oriented programming
principles you might be familiar with. Chapter 19 is a reference on patterns and
pattern matching, which are powerful ways of expressing ideas throughout Rust
programs. Chapter 20 contains a smorgasbord of advanced topics of interest,
including unsafe Rust, macros, and more about lifetimes, traits, types,
functions, and closures.

In Chapter 21, we’ll complete a project in which we’ll implement a low-level
multithreaded web server!

Finally, some appendixes contain useful information about the language in a more
reference-like format. **Appendix A** covers Rust’s keywords, **Appendix B**
covers Rust’s operators and symbols, **Appendix C** covers derivable traits
provided by the standard library, **Appendix D** covers some useful development
tools, and **Appendix E** explains Rust editions. In **Appendix F**, you can
find translations of the book, and in **Appendix G** we’ll cover how Rust is
made and what nightly Rust is.

Terakhir, beberapa lampiran yg berisi informasi bermanfaat mengenai bahasa ini
dalam format seperti referensi. Lampiran A meliputi kata kunci Rust, Lampiran B
meliputi operator dan simbol Rust, Lampiran C meliputi trait turunan yg
tersedia pada library std, Lampiran D meliputi beberapa aplikasi pengembang yg
bermanfaat, dan Lampiran E menjelaskan edisi Rust. Pada Lampiran F, kita akan
menemukan terjemahan dari buku ini, dan di Lampiran G kita akan membahas
bagaimana Rust dbuat dan apa itu nightly Rust.

Tidak ada cara salah dalam membaca buku ini: jika kita ingin melewati beberapa
halaman, maka lakukanlah! Kita mungkin akan lompat kebelakang pada Bab-bab awal
jika mengalami kebingungan. Lakukan apa yg menurut kita nyaman.

<span id="ferris"></span>

Sebuah bagian penting dari proses pembelajaran Rust adalah mempelajari
bagaimana membaca pesan error yg ditampilkan oleh compiler: hal ini akan
menuntun kita menuju kode yg berfungsi. Karenanya, kita akan menyediakan
banyak contoh yg tidak bisa di compile beserta pesan error dari compiler pada
tiap situasi tersebut. Pahami bahwa jika kita menjalankan contoh secara acak,
ada kemungkinan tidak bisa di compile! Pastikan kita membaca kalimat
disekitarnya apakah contoh tersebut, yg hendak kita jalankan, memang bertujuan
untuk error. Ferris juga akan membantu kita membedakan kode yg memang bertujuan
agar tidak berfungsi:

| Ferris                                                                                                           | Meaning                                          |
| ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | This code does not compile!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | This code panics!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | This code does not produce the desired behavior. |

Dalam kebanyakan situasi, kita akan arahkan kita ke versi yg benar dari kode
tersebut yg tidak bisa di compile.

## Sumber Terbuka

Berkas sumber dimana buku ini dibuat dapat ditemukan pada
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
