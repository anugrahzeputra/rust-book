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

- Cargo, manajer dependensi dan alat build bawaan, membuat proses menambah,  
  mengompilasi, dan mengelola dependensi menjadi mudah dan konsisten di seluruh ekosistem Rust.
- Alat pemformatan Rustfmt memastikan gaya penulisan kode yang konsisten di antara para pengembang.
- Rust-analyzer mendukung integrasi Integrated Development Environment (IDE)  
  untuk melengkapi kode (code completion) dan menampilkan pesan error secara langsung.

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

Rust ditujukan untuk orang-orang yang mendambakan kecepatan dan stabilitas dalam sebuah bahasa.
Dengan kecepatan, maksudnya adalah seberapa cepat kode Rust bisa dijalankan dan seberapa cepat
Rust memungkinkan kita menulis program. Pemeriksaan yang dilakukan oleh compiler Rust memastikan
stabilitas bahkan ketika ada penambahan fitur atau refactoring. Ini berbeda dengan kode warisan
(legacy code) yang rapuh pada bahasa tanpa pemeriksaan seperti ini, yang sering membuat pengembang
takut untuk memodifikasinya. Dengan berupaya mencapai zero-cost abstractions—fitur tingkat tinggi
yang dikompilasi menjadi kode tingkat rendah secepat kode yang ditulis manual—Rust berusaha agar
kode yang aman juga bisa menjadi kode yang cepat.

Bahasa Rust juga berharap bisa mendukung banyak pengguna lainnya; yang disebutkan di sini hanyalah
beberapa pemangku kepentingan terbesar. Secara keseluruhan, ambisi terbesar Rust adalah menghapus
kompromi yang telah diterima para programmer selama puluhan tahun dengan menyediakan keamanan _dan_
produktivitas, kecepatan _dan_ ergonomi. Cobalah Rust dan lihat apakah pilihan-pilihannya cocok untuk kita.

## Buku ini ditujukan untuk Siapa

Buku ini mengasumsikan bahwa kita sudah pernah menulis kode dalam bahasa pemrograman lain,
tetapi tidak mengasumsikan bahasa mana yang kita gunakan. Kami berusaha membuat materi ini
dapat diakses secara luas oleh siapa pun dengan berbagai latar belakang pemrograman.
Kami tidak banyak membahas tentang apa itu pemrograman atau bagaimana cara berpikir tentangnya.
Jika kita benar-benar baru dalam pemrograman, akan lebih baik membaca buku yang secara khusus
memberikan pengantar tentang pemrograman.

## Bagaimana cara menggunakan Buku ini

Secara umum, buku ini mengasumsikan kita membacanya secara berurutan dari depan
ke belakang. Bab selanjutnya mendasarkan konsepnya pada Bab sebelumnya, dan Bab
awal tidak akan menjelaskan secara detil mengenai topik tertentu tetapi akan
menjelaskan lebih lanjut topik tersebut pada Bab selanjutnya.

Kita akan menemukan dua jenis bab dalam buku ini: bab konsep dan bab proyek.
Dalam bab konsep, kita akan mempelajari satu aspek dari Rust. Dalam bab proyek,
kita akan membangun program kecil bersama-sama, menerapkan apa yang sudah kita pelajari sejauh ini.
Bab 2, 12, dan 21 adalah bab proyek; sisanya adalah bab konsep.

Bab 1 menjelaskan bagaimana memasang Rust, bagaimana membuat program
"Hello, World!", dan bagaimana cara menggunakan aplikasi Cargo, yg berfungsi
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

Pada Bab 16, kita akan membahas berbagai model pemrograman konkuren dan
mempelajari bagaimana Rust membantu kita menulis program multithread tanpa rasa takut.
Pada Bab 17, kita melanjutkannya dengan mengeksplorasi sintaks async dan await di Rust,
beserta tasks, futures, dan streams, serta model konkuren ringan yang mereka sediakan.

Bab 18 melihat bagaimana idiom Rust dibandingkan dengan prinsip pemrograman
berorientasi objek yang mungkin sudah kita kenal. Bab 19 adalah referensi tentang pola
dan pattern matching, yang merupakan cara kuat untuk mengekspresikan ide di seluruh
program Rust. Bab 20 berisi beragam topik lanjutan yang menarik, termasuk unsafe Rust,
macros, serta pembahasan lebih dalam tentang lifetimes, traits, types, functions, dan closures.

Pada Bab 21, kita akan menyelesaikan sebuah proyek dengan mengimplementasikan
web server multithread tingkat rendah!

Akhirnya, beberapa lampiran berisi informasi berguna tentang bahasa Rust dalam format yang
lebih mirip referensi. **Lampiran A** membahas kata kunci Rust, **Lampiran B** membahas
operator dan simbol Rust, **Lampiran C** membahas traits turunan (derivable traits) yang
disediakan pustaka standar, **Lampiran D** membahas beberapa tools pengembangan yang berguna,
dan **Lampiran E** menjelaskan edisi Rust. Pada **Lampiran F**, kita bisa menemukan terjemahan
dari buku ini, dan di **Lampiran G** kita akan membahas bagaimana Rust dikembangkan serta apa itu
Rust nightly.

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
| ---------------------------------------------------------------------------------------------------------------- |--------------------------------------------------|
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris with a question mark"/>            | Kode ini tidak bisa di compile!                  |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris throwing up their hands"/>                   | Kode ini menghasilkan panic!                     |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris with one claw up, shrugging"/> | Kode ini tidak menghasilkan cara yang diinginkan |

Dalam kebanyakan situasi, kita akan arahkan kita ke versi yg benar dari kode
tersebut yg tidak bisa di compile.

## Sumber Terbuka

Berkas sumber dimana buku ini dibuat dapat ditemukan pada
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/main/src
