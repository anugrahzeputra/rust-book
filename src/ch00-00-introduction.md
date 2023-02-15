# Pengenalan

> Catatan: Buku ini adalah buku yg sama dengan [The Rust Programming
> Language][nsprust] yg tersedia dalam format cetak ataupun digital pada [No Starch
> Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-2nd-edition
[nsp]: https://nostarch.com/

Selamat datang pada *Bahasa Pemrograman Rust*, sebuah buku pengantar mengenai
Rust. Bahasa pemrograman Rust membantu anda membuat program lebih cepat, lebih
handal. Ergonomis tingkat-tinggi dan kontrol tingkat-rendah biasanya saling
berlawanan didalam rancangan suatu bahasa pemrograman. Rust menerima tantangan
tersebut, melalui pengalaman pengembang yg luas dan penyeimbangan kapasitas
teknik yg mumpuni. Rust menyediakan detil kontrol tingkat-rendah (seperti
penggunaan memori) tanpa kerumitan yg biasanya menyertainya.

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

* Cargo, aplikasi untuk pengelolaan dan pembuatan dependency, sehingga
  penambahan, kompilasi, dan pengelolaan dependency mudah dan konsisten lintas
  ekosistem Rust.
* Rustfmt merupakan aplikasi untuk memastikan konsistensi coding style diantara
  para pengembang.
* Rust Language Server mendukung integrasi IDE untuk pelengkapan kode dan
  penyisipan pesan error.

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
menunggu kontribusi anda pada bahasa Rust.

### Mereka yg fokus pada Kecepatan dan Stabilitas

Rust diperuntukkan bagi mereka yg menginginkan kecepatan dan stabilitas pada
suatu bahasa pemrograman. Kecepatan yg dimaksud disini adalah kecepatan kode
Rust berjalan, maupun kecepatan anda dalam menuliskan kode tersebut. Compiler
Rust memastikan stabilitas melalui penambahan fitur dan refaktorisasi. Hal ini
kontras dg betapa rapuhnya kode peninggalan sebelumnya di bahasa ini, yg tanpa
melalui pengecekan tersebut, sehingga pengembang enggan mengubah kodenya.
Dengan abstraksi tanpa-modal, fitur tingkat-tinggi yg melakukan penyusunan kode
tingkat-rendah secepat mungkin, Kode Rust menjadi lebih aman dan lebih cepat.

Bahasa Rust berharap untuk mendukung kelompok yg lain; Beberapa yg disebutkan
disini hanyalah kelompok yg paling umum. Secara keseluruhan, tujuan terbesar
Rust adalah mengeliminasi konsep timbal-balik yg disepakati oleh para
programmer selama beberapa dekade mengenai Keamanan *dan* Produktifitas,
Kecepatan *dan* Ergonomis. Silakan coba Rust dan rasakan sendiri apakah
fiturnya bermanfaat bagi anda.

## Buku ini ditujukan untuk Siapa

Buku ini mengasumsikan bahwa anda mampu menulis bahasa pemrograman yg lain,
meski tidak spesifik bahasa apa. Kita berusaha membuat bahan pembelajaran yg
mampu diterima oleh latar belakang programmer yg luas. Kita tidak fokus
berbicara mengenai *apa* itu pemrograman atau bagaimana pemrogramannya. Jika
anda benar-benar baru didalam pemrograman, akan lebih baik jika anda membaca
buku yg secara spesifik menyediakan informasi mengenai pengenalan pemrograman.

## Bagaimana cara menggunakan Buku ini

Secara umum, buku ini mengasumsikan anda membacanya secara berurutan dari depan
ke belakang. Bab selanjutnya mendasarkan konsepnya pada Bab sebelumnya, dan Bab
awal tidak akan menjelaskan secara detil mengenai topik tertentu tetapi akan
menjelaskan lebih lanjut topik tersebut pada Bab selanjutnya.

Anda akan menjumpai 2 jenis Bab di buku ini: Bab yg berisi konsep dan Bab yg
berisi Proyek. Pada Bab konsep, anda akan mempelajari suatu aspek tertentu pada
Rust. Pada Bab proyek, kita akan membuat program sederhana bersama-sama,
menerapkan apa yg telah anda pelajari sebelumnya. Bab 2, 12 dan 20 adalah Bab
Proyek; sisanya adalah Bab Konsep.

Bab 1 menjelaskan bagaimana memasang Rust, bagaimana membuat program
"Halo, dunia!", dan bagaimana cara menggunakan aplikasi Cargo, yg berfungsi
sebagai pengelola paket dan aplikasi compiler. Bab 2 mengenai menulis program
secara komprehensif di Rust, anda akan membuat permainan tebak-menebak.
Dititik ini kita membahas konsep pada tingkat yg relatif tinggi dan pada bab
selanjutnya detil tambahan akan dielaborasi.

Jika anda ingin segera masuk keproses koding, Bab 2 adalah tempat yg paling
cocok untuk itu. Bab 3 membahas fitur Rust yg mirip dg bahasa pemrograman
lainnya, dan di Bab 4 anda akan mempelajari Sistem Kepemilikan Rust. Jika anda
pelajar yg perfeksionis yg lebih memilih mempelajari setiap detilnya sebelum
berpindah ke selanjutnya, mungkin lebih baik anda lewati Bab 2 dan langsung ke
Bab 3, baru kembali ke Bab 2 jika anda ingin mengerjakan suatu proyek, untuk
menerapkan ilmu yg sudah anda pelajari sebelumnya.

Bab 5 berbicara mengenai struct dan method, dan Bab 6 membahas enum, ungkapan
`match`, dan pembuatan aliran kontrol `if let`. Anda akan menggunakan struct
dan enum untuk membuat suatu tipe tertentu di Rust.

Di Bab 7, anda akan mempelajari Sistem Modul Rust dan aturan privasi untuk
mengelola kode anda dan API publik nya yg terkait. Bab 8 berbicara mengenai
struktur data jamak yg tersedia pada library std, seperti vektor, string, dan
hash map. Bab 9 menjelajahi teknik dan filosofi Rust terhadap penanganan Error.

Bab 10 menggali lebih dalam mengenai generic, trait, dan lifetime, yg akan
memberikan anda kemudahan dalam membuat kode yg akan diterapkan pada beberapa
tipe. Bab 11 mengenai Pengujian, dimana bahkan dg jaminan keamanan dari Rust,
tetap dibutuhkan untuk menjaga logika program anda bekerja dg benar. Di Bab 12,
kita akan mencoba membuat implementasi dari subset fungsionalitas aplikasi
baris perintah `grep` yg melakukan pencarian didalam berkas. Untuk keperluan
ini, kita akan menggunakan banyak konsep yg telah dibahas pada Bab-bab
sebelumnya.

Bab 13 menjelajahi closure dan iterator: suatu fitur didalam Rust yg berasal
dari bahasa pemrograman fungsional. Di Bab 14, kita akan menelaah Cargo lebih
dalam dan berbicara mengenai cara terbaik untuk berbagi library anda dg yg
lain. Bab 15 membahas smart pointer yg disediakan oleh library std dan trait
yg mengaktifkan fungsionalitasnya.

Pada Bab 16, kita akan melalui model pemrograman konkuren yg berbeda-beda dan
berbicara mengenai bagaimana Rust membantu anda melakukan pemrograman
multithread tanpa kekhawatiran. Bab 17 membandingkan antara idiom Rust dg
prinsip pemrograman object oriented yg mungkin anda kenali sebelumnya.

Bab 18 adalah referensi mengenai pola dan pencocokan pola, yg merupakan suatu
cara yg efektif dalam mengungkapkan suatu ide tertentu pada program Rust. Bab
19 berisi topik tingkat lanjut, termasuk unsafe Rust, macro dan lebih detil
mengenai lifetime, trait, tipe, fungsi dan closure.

Pada Bab 20, kita akan menyelesaikan sebuah proyek dimana kita akan
mengimplementasikan, layanan web multithread!

Terakhir, beberapa lampiran yg berisi informasi bermanfaat mengenai bahasa ini
dalam format seperti referensi. Lampiran A meliputi kata kunci Rust, Lampiran B
meliputi operator dan simbol Rust, Lampiran C meliputi trait turunan yg
tersedia pada library std, Lampiran D meliputi beberapa aplikasi pengembang yg
bermanfaat, dan Lampiran E menjelaskan edisi Rust. Pada Lampiran F, anda akan
menemukan terjemahan dari buku ini, dan di Lampiran G kita akan membahas
bagaimana Rust dbuat dan apa itu nightly Rust.

Tidak ada cara salah dalam membaca buku ini: jika anda ingin melewati beberapa
halaman, maka lakukanlah! Anda mungkin akan lompat kebelakang pada Bab-bab awal
jika mengalami kebingungan. Lakukan apa yg menurut anda nyaman.

<span id="ferris"></span>

Sebuah bagian penting dari proses pembelajaran Rust adalah mempelajari
bagaimana membaca pesan error yg ditampilkan oleh compiler: hal ini akan
menuntun anda menuju kode yg berfungsi. Karenanya, kita akan menyediakan
banyak contoh yg tidak bisa di compile beserta pesan error dari compiler pada
tiap situasi tersebut. Pahami bahwa jika anda menjalankan contoh secara acak,
ada kemungkinan tidak bisa di compile! Pastikan anda membaca kalimat
disekitarnya apakah contoh tersebut, yg hendak anda jalankan, memang bertujuan
untuk error. Ferris juga akan membantu anda membedakan kode yg memang bertujuan
agar tidak berfungsi:

| Ferris                                                                                                           | Meaning                                          |
|------------------------------------------------------------------------------------------------------------------|--------------------------------------------------|
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | This code does not compile!                      |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | This code panics!                                |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | This code does not produce the desired behavior. |

Dalam kebanyakan situasi, kita akan arahkan anda ke versi yg benar dari kode
tersebut yg tidak bisa di compile.

## Sumber Terbuka

Berkas sumber dimana buku ini dibuat dapat ditemukan pada
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
