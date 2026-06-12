## Menerapkan Konkurensi dengan Async

<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

Di bagian ini, kita bakal menerapkan _async_ pada beberapa tantangan 
konkurensi yang udah kita tangani pakai _threads_ di Bab 16. Karena kita udah 
membahas banyak dari ide kuncinya di sana, di bagian ini kita bakal fokus 
ke apa yang membedakan _threads_ dan _futures_.

Di banyak kasus, API buat bekerja dengan konkurensi memakai _async_ itu mirip 
banget sama API buat memakai _threads_. Di kasus lainnya, ternyata mereka itu 
cukup beda. Bahkan pas API-nya _kelihatan_ mirip antara _threads_ dan _async_, 
mereka sering kali punya perilaku yang berbeda—dan hampir selalu punya 
karakteristik performa yang berbeda juga.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Membikin Task Baru dengan `spawn_task`

Operasi pertama yang kita kerjakan di [Membikin Thread Baru dengan 
Spawn][thread-spawn] adalah ngitung naik (counting up) dari dua _threads_ yang 
berbeda. Mari kita lakuin hal yang sama memakai _async_. _Crate_ `trpl` 
menyediakan sebuah fungsi `spawn_task` yang kelihatannya mirip banget sama API 
`thread::spawn`, dan sebuah fungsi `sleep` yang merupakan versi _async_ dari API 
`thread::sleep`. Kita bisa memakai mereka berdua buat mengimplementasikan 
contoh ngitung tersebut, seperti yang ditunjukkan di Listing 17-6.

<Listing number="17-6" caption="Membikin sebuah task baru buat mencetak sesuatu sementara main task mencetak hal lain" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Sebagai langkah awal, kita nge-_setup_ fungsi `main` kita pakai `trpl::run` 
supaya fungsi tingkat teratas kita ini bisa jadi fungsi _async_.

> Catatan: Dari titik ini sampai ke akhir bab, setiap contoh bakal selalu 
> nyertain kode pembungkus dengan `trpl::run` di `main` ini, jadi kita 
> bakal sering ngelewatin bagian itu persis kayak kita ngelewatin fungsi `main` 
> sebelumnya. Jangan lupa buat menambahkannya di kode Anda ya!

Terus kita menulis dua _loops_ (perulangan) di dalam blok itu, masing-masing 
mengandung panggilan ke `trpl::sleep`, yang bakal nungguin selama setengah 
detik (500 milidetik) sebelum mengirim pesan selanjutnya. Kita naruh satu _loop_ 
di dalam _body_ dari `trpl::spawn_task` dan _loop_ yang satunya lagi ditaruh di 
sebuah _for loop_ tingkat teratas. Kita juga nambahin `await` setelah pemanggilan 
ke `sleep`.

Kode ini berperilaku mirip sama implementasi yang berbasis _thread_—termasuk fakta 
bahwa Anda mungkin bakal ngelihat pesan-pesannya muncul dengan urutan yang 
beda di terminal Anda sendiri pas Anda menjalankannya:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Versi ini berhenti sesaat setelah _for loop_ di dalam _body_ dari blok _async_ 
utama kita selesai, karena _task_ yang dibikin sama `spawn_task` bakal dimatiin 
pas fungsi `main` berakhir. Kalau Anda pengen _task_ tersebut jalan terus sampai 
kelar, Anda harus memakai sebuah _join handle_ buat nungguin _task_ pertama 
tersebut sampai selesai. Dengan _threads_, kita memakai method `join` buat 
"memblokir" sampai _thread_-nya selesai jalan. Di Listing 17-7, kita bisa 
memakai `await` buat melakukan hal yang sama, karena *task handle* itu sendiri 
adalah sebuah _future_. Tipe `Output`-nya adalah sebuah `Result`, jadi kita 
juga meng-_unwrap_-nya setelah nge-_await_-nya.

<Listing number="17-7" caption="Memakai `await` dengan sebuah *join handle* buat menjalankan sebuah task sampai selesai" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Versi yang di-update ini berjalan sampai _kedua_ loops tersebut kelar.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Sejauh ini, kelihatannya _async_ dan _threads_ ngasih kita hasil dasar yang sama 
aja, cuma beda di sintaks: memakai `await` bukannya memanggil `join` pada 
*join handle*, dan nge-_await_ pemanggilan fungsi `sleep`.

Perbedaan yang lebih besar adalah kita tidak perlu ngebikin (_spawn_) sebuah 
_thread_ sistem operasi lain buat ngelakuin ini. Faktanya, kita bahkan tidak perlu 
ngebikin _task_ di sini. Karena blok _async_ di-compile jadi _futures_ anonim, 
kita bisa menaruh tiap _loop_ di dalam sebuah blok _async_ lalu nyuruh 
_runtime_-nya buat ngejalanin kedua blok itu sampai selesai memakai fungsi 
`trpl::join`.

Di bagian [Menunggu Semua Threads buat Selesai Memakai `join` Handles][join-handles], 
kita udah nunjukin gimana cara pakai method `join` pada tipe `JoinHandle` yang 
dibalikin pas Anda memanggil `std::thread::spawn`. Fungsi `trpl::join` ini mirip, 
tapi buat _futures_. Pas Anda ngasih dua _futures_ ke dia, dia bakal ngasilin satu 
_future_ baru tunggal yang mana outputnya adalah sebuah tuple yang isinya output 
dari setiap _future_ yang Anda masukin ke dia sesaat setelah mereka _keduanya_ 
selesai. Jadi, di Listing 17-8, kita memakai `trpl::join` buat nungguin `fut1` 
dan `fut2` sampai kelar. Kita _tidak_ nge-_await_ `fut1` dan `fut2`, tapi kita 
malah nge-_await_ _future_ baru yang dihasilin sama `trpl::join`. Kita ngabaiin 
output-nya, karena isinya cuma tuple yang nampung dua nilai unit `()`.

<Listing number="17-8" caption="Memakai `trpl::join` buat nge-await dua futures anonim" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Pas kita jalanin kode ini, kita melihat kedua _futures_ jalan sampai kelar:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Nah, sekarang Anda bakal ngelihat urutan yang sama persis di setiap jalan (run), 
yang mana ini beda banget sama apa yang kita lihat di _threads_. Ini karena 
fungsi `trpl::join` itu sifatnya _adil_ (fair), yang artinya dia ngecek setiap 
_future_ sama seringnya, ganti-gantian di antara mereka, dan tidak pernah 
ngebiarin satu _future_ ngacir jalan duluan (race ahead) kalau _future_ yang 
satunya lagi siap jalan juga. Dengan _threads_, sistem operasi yang mutusin 
_thread_ mana yang mau dicek dan berapa lama _thread_ itu dibiarin jalan. 
Dengan Rust _async_, _runtime_ yang mutusin _task_ mana yang mau dicek. 
(Di praktiknya, detailnya bisa jadi ribet karena sebuah _async runtime_ mungkin 
aja pakai _threads_ sistem operasi di balik layarnya sebagai bagian dari cara 
dia mengelola konkurensi, jadi buat ngejamin keadilan bisa jadi makan usaha 
lebih buat sebuah _runtime_—tapi itu tetap mungkin dilakuin!) _Runtimes_ tidak 
diwajibkan menjamin keadilan buat operasi apa pun yang dikasih ke dia, dan 
mereka sering kali nawarin berbagai macam API buat ngasih Anda pilihan apakah 
Anda mau fungsionalitas yang adil atau tidak.

Cobain beberapa variasi ini dalam nge-_await_ _futures_ tersebut dan lihat 
apa yang terjadi:

- Hapus blok _async_ dari sekitar salah satu atau kedua _loops_.
- Nge-_await_ setiap blok _async_ secara langsung persis setelah mendefinisikannya.
- Bungkus _loop_ pertama doang di dalam sebuah blok _async_, terus nge-_await_ 
  _future_ hasilnya setelah _body_ dari _loop_ kedua.

Sebagai tantangan ekstra, lihat apakah Anda bisa nyari tahu bakal jadi apa 
outputnya di setiap kasus _sebelum_ Anda beneran ngejalanin kodenya!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>

### Menghitung Naik di Dua Task Memakai Message Passing

Berbagi data (sharing data) di antara _futures_ juga bakal terasa familier: kita 
bakal pakai _message passing_ (pengiriman pesan) lagi, tapi kali ini pakai tipe 
dan fungsi versi _async_. Kita bakal ambil rute yang agak beda dari apa yang 
kita lakuin di [Memakai Message Passing buat Mentransfer Data Antar 
Threads][message-passing-threads] buat ngilustrasiin beberapa perbedaan 
mendasar antara konkurensi berbasis _thread_ dan konkurensi berbasis _futures_. Di 
Listing 17-9, kita bakal mulai dengan cuma satu blok _async_—_bukan_ membikin 
_task_ terpisah kayak yang kita lakuin pas membikin _thread_ terpisah dulu.

<Listing number="17-9" caption="Membikin sebuah _channel_ async dan me-assign kedua paruhnya ke `tx` dan `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Di sini, kita pakai `trpl::channel`, versi _async_ dari API _multiple-producer, 
single-consumer channel_ yang kita pakai bareng _threads_ di Bab 16 dulu. Versi 
_async_ dari API ini cuma beda sedikit dari versi yang berbasis _thread_: dia 
memakai _receiver_ `rx` yang _mutable_ ketimbang yang _immutable_, dan method 
`recv`-nya ngasilin sebuah _future_ yang harus kita `await` ketimbang langsung 
ngasilin nilainya. Sekarang kita bisa ngirim pesan dari pengirim ke penerimanya. 
Perhatiin kalau kita tidak perlu membikin sebuah _thread_ terpisah atau bahkan 
membikin sebuah _task_ terpisah; kita cuma butuh nge-_await_ pemanggilan ke 
`rx.recv`.

Method sinkron `Receiver::recv` di `std::mpsc::channel` itu bakal nge-blok 
(block) sampai dia menerima sebuah pesan. Method `trpl::Receiver::recv` tidak 
begitu, karena dia itu _async_. Alih-alih nge-blok, dia mengembalikan kontrol 
ke _runtime_ sampai entah ada pesan yang diterima atau sisi pengirim di _channel_ 
tersebut ditutup. Sebagai perbandingan, kita tidak nge-_await_ pemanggilan ke 
`send` karena `send` tidak memblokir. Dan dia tidak perlu begitu, karena 
_channel_ yang lagi kita pakai buat ngirim ini ukurannya tidak terbatas 
(unbounded).

> Catatan: Karena semua kode _async_ ini berjalan di dalam sebuah blok _async_ 
> di dalam pemanggilan ke `trpl::run`, semua hal di dalamnya bisa terhindar dari 
> pemblokiran (blocking). Namun, kode _di luar_ blok tersebut bakal terblokir 
> menunggu kembalian (return) dari fungsi `run`. Itulah intinya dari fungsi 
> `trpl::run`: dia ngasih Anda _pilihan_ di mana Anda mau memblokir eksekusi 
> di sekitar serangkaian kode _async_, yang mana berarti di titik itulah terjadi 
> transisi (peralihan) antara kode _sync_ dan _async_. Di mayoritas _async 
> runtimes_, `run` itu nyatanya dinamain `block_on` dengan alasan ini persis.

Perhatiin dua hal tentang contoh ini. Pertama, pesannya bakal sampai saat itu 
juga. Kedua, walaupun kita pakai _future_ di sini, belum ada konkurensi yang 
terjadi. Semua hal di listing ini terjadi berurutan, sama persis kayak kalau 
tidak ada _futures_ yang terlibat sama sekali.

Mari kita beresin masalah bagian pertama dengan ngirim serangkaian pesan 
dan ngasih _sleep_ (jeda) di antara pesan-pesan tersebut, seperti yang 
ditunjukin di Listing 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Mengirim dan menerima banyak pesan lewat _channel_ async dan ngasih jeda pake `await` di antara setiap pesannya" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Selain mengirim pesan-pesannya, kita juga perlu menerima mereka. Di kasus ini, 
karena kita tahu berapa banyak pesan yang bakal datang, kita bisa ngelakuin itu 
secara manual dengan manggil `rx.recv().await` empat kali. Tapi di dunia nyata, 
kita umumnya bakal menunggu sejumlah pesan yang *nggak diketahui* banyaknya, jadi 
kita perlu terus menunggu sampai kita nentuin kalau tidak ada pesan yang datang 
lagi.

Di Listing 16-10, kita memakai `for` _loop_ buat memproses semua item yang diterima 
dari sebuah _channel_ sinkron. Rust tapi belum punya cara buat nulis `for` 
_loop_ yang berjalan di sebuah rangkaian item yang _asynchronous_, jadi kita perlu 
memakai sebuah _loop_ yang belum pernah kita lihat sebelumnya: _conditional loop_ 
(perulangan bersyarat) `while let`. Ini adalah versi perulangan dari konstruk 
`if let` yang udah kita lihat di bagian [Control Flow yang Ringkas pake `if 
let` sama `let else`][if-let]. _Loop_ ini bakal terus jalan selama pola 
(pattern) yang ditentukannya masih cocok sama nilainya.

Pemanggilan ke `rx.recv` ngasilin sebuah _future_, yang terus kita `await`. 
_Runtime_ bakal mem-pause _future_ ini sampai dia siap. Begitu ada pesan yang 
datang, _future_ tersebut bakal di-resolve (terselesaikan) jadi `Some(message)` 
sebanyak pesan yang datang. Pas _channel_-nya ditutup, terlepas dari apakah ada 
pesan yang datang atau tidak, _future_ ini malah bakal nge-resolve jadi `None` 
buat mengindikasikan kalau tidak ada nilai lagi yang datang dan makanya kita harus 
berhenti nge-poll (polling)—yang berarti, berhenti nge-_await_.

_Loop_ `while let` mengumpulkan semua logika ini jadi satu. Kalau hasil dari 
memanggil `rx.recv().await` adalah `Some(message)`, kita dapet akses ke pesannya 
dan kita bisa makainya di dalam _body_ dari _loop_, persis kayak pas kita 
pakai `if let`. Kalau hasilnya `None`, _loop_-nya berakhir. Setiap kali _loop_-nya 
selesai, dia bakal ketemu lagi sama _await point_, jadi _runtime_ bakal nge-_pause_ 
dia lagi sampai ada pesan lain yang datang.

Kodenya sekarang udah berhasil mengirim dan menerima semua pesan. Sayangnya, masih 
ada beberapa masalah nih. Pertama, pesan-pesannya tidak berdatangan dalam 
interval setengah detik. Mereka datang sekaligus seketika, 2 detik (2.000 
milidetik) setelah kita memulai programnya. Kedua, program ini tidak pernah 
berakhir! Dia malah bakal terus nungguin pesan-pesan baru selamanya. Anda bakal 
perlu mematikannya memakai <span class="keystroke">ctrl-c</span>.

Mari kita mulai dengan meneliti kenapa pesan-pesannya berdatangan sekaligus setelah 
seluruh jedanya selesai, bukannya datang dengan jeda-jeda di antara mereka. Di dalam 
sebuah blok _async_, urutan dari keyword `await` yang muncul di kodenya juga 
adalah urutan eksekusi mereka pas programnya jalan.

Cuma ada satu blok _async_ di Listing 17-10, jadi semua hal di dalamnya jalan 
secara linear. Masih belum ada konkurensi. Semua pemanggilan `tx.send` 
terjadi, diselingi dengan semua pemanggilan `trpl::sleep` beserta _await 
points_-nya. Baru setelah itu _loop_ `while let` dapat giliran buat ngelewatin 
_await points_ di pemanggilan `recv`.

Biar dapet perilaku yang kita inginkan, di mana ada _delay_ (jeda) di antara 
setiap pesan, kita harus menaruh operasi `tx` dan `rx` di dalam blok _async_ 
mereka sendiri, seperti yang ditunjukin di Listing 17-11. Kemudian _runtime_ bisa 
mengeksekusi masing-masing secara terpisah memakai `trpl::join`, sama kayak 
di contoh penghitungan (counting) sebelumnya. Sekali lagi, kita nge-_await_ 
hasil dari panggilan `trpl::join`, bukan nge-_await_ _futures_ individu itu 
sendiri. Kalau kita nge-_await_ _futures_ itu secara berurutan, kita malah 
bakal berujung kembali ke _flow_ (aliran) sekuensial (berurutan) lagi—yang mana 
itu hal yang *tidak* pengen kita lakuin.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Misahin `send` dan `recv` ke blok `async` mereka sendiri-sendiri dan nge-_await_ _futures_ buat blok-blok tersebut" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Dengan kode yang di-update di Listing 17-11, pesan-pesannya kini dicetak 
dalam interval 500 milidetik, bukannya datang borongan (all in a rush) setelah 
2 detik.

Tapi programnya masih belum mau berakhir nih, karena cara _loop_ `while let` 
berinteraksi dengan `trpl::join`:

- _Future_ yang dikembalikan dari `trpl::join` itu cuma kelar kalau *kedua* 
  _futures_ yang dimasukkan ke dia udah kelar dua-duanya.
- _Future_ dari `tx` itu kelar setelah dia kelar nge-_sleep_ setelah mengirimkan 
  pesan terakhir dari `vals`.
- _Future_ dari `rx` tidak bakal kelar sampai _loop_ `while let` itu berakhir.
- _Loop_ `while let` itu tidak bakal berakhir sampai proses nge-_await_ `rx.recv` 
  menghasilkan `None`.
- Proses nge-_await_ `rx.recv` cuma bakal ngembaliin `None` pas sisi sebelah sana 
  dari _channel_-nya itu ditutup (closed).
- _Channel_ tersebut cuma bakal ditutup kalau kita memanggil `rx.close` atau 
  pas sisi pengirimnya (sender side), `tx`, di-_drop_.
- Kita tidak memanggil `rx.close` di mana pun, dan `tx` tidak bakal di-_drop_ 
  sampai blok _async_ paling luar yang kita masukin ke `trpl::run` itu berakhir.
- Blok tersebut tidak bisa berakhir karena dia diblokir menunggu `trpl::join` 
  buat kelar, yang mana bawa kita kembali lagi ke awal dari daftar (list) ini.

Kita bisa aja menutup `rx` secara manual dengan manggil `rx.close` di suatu 
tempat, tapi itu kurang masuk akal. Berhenti setelah nanganin pesan sebanyak 
jumlah sembarang yang kita tentuin emang bakal bikin programnya bisa berakhir, 
tapi kita berisiko buat kelewatan beberapa pesan. Kita butuh cara lain buat mastiin 
kalau `tx` itu di-_drop_ *sebelum* akhir dari fungsinya.

Saat ini, blok _async_ tempat kita ngirim pesan cuma meminjam (borrows) `tx` 
karena ngirim pesan itu tidak butuh kepemilikan (ownership), tapi kalau kita 
bisa memindahkan (move) `tx` ke dalam blok _async_ itu, dia bakal di-_drop_ pas 
blok tersebut berakhir. Di Bab 13, bagian [Menangkap Referensi atau Memindahkan 
Kepemilikan][capture-or-move], Anda udah belajar gimana cara pake keyword `move` 
bareng _closures_, dan, seperti yang dibahas di Bab 16, bagian [Memakai Closures 
`move` bersama Threads][move-threads], kita juga sering kali perlu mindahin data 
ke dalam _closures_ pas lagi bekerja dengan _threads_. Dinamika dasar yang sama 
juga berlaku di blok _async_, jadi keyword `move` itu bekerja buat blok _async_ 
sama kayak yang dia lakuin buat _closures_.

Di Listing 17-12, kita ngubah blok yang dipake buat ngirim pesan-pesan dari 
`async` jadi `async move`. Pas kita jalanin _versi_ kode yang ini, dia bakal 
berakhir dengan damai (gracefully) setelah pesan terakhirnya dikirim dan diterima.

<Listing number="17-12" caption="Revisi kode dari Listing 17-11 yang bisa shut down dengan bener pas udah kelar" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

_Channel_ _async_ ini juga merupakan _channel multiple-producer_ (banyak 
pengirim), jadi kita bisa manggil `clone` pada `tx` kalau kita pengen ngirim 
pesan dari berbagai _futures_ yang berbeda, kayak yang ditunjukin di Listing 
17-13.

<Listing number="17-13" caption="Memakai banyak *producers* dengan blok async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Pertama, kita nge-_clone_ `tx`, membikin `tx1` di luar blok _async_ pertama. 
Kita mindahin `tx1` ke dalam blok itu persis kayak yang kita lakuin sebelumnya 
sama `tx`. Terus, belakangan, kita mindahin `tx` aslinya ke dalam sebuah blok 
_async_ yang *baru*, di mana kita ngirim pesan-pesan tambahan dengan _delay_ yang 
sedikit lebih lambat. Kita kebetulan menaruh blok _async_ baru ini setelah 
blok _async_ buat nerima pesan-pesan, tapi blok ini juga sah-sah aja ditaruh 
sebelum blok penerimanya. Kuncinya ada di urutan gimana _futures_ itu di-_await_, 
bukan urutan kapan mereka dibikin (created).

Kedua blok _async_ yang buat mengirim pesan ini haruslah berupa blok `async move` 
sehingga `tx` dan `tx1` dua-duanya bakal di-_drop_ pas blok-blok tersebut selesai. 
Kalau tidak, kita bakal berujung pada _infinite loop_ (perulangan tanpa henti) 
yang sama kayak yang kita mulai tadi. Terakhir, kita beralih dari `trpl::join` ke 
`trpl::join3` buat menangani tambahan _future_ yang ada.

Sekarang kita melihat semua pesan dari kedua _futures_ pengirim, dan karena 
_futures_ pengirim memakai jeda yang sedikit berbeda setelah pengiriman, pesan-
pesannya juga diterima di dalam interval waktu yang berbeda-beda.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Ini adalah awal yang bagus, tapi ini ngebatesin kita cuma buat beberapa 
_futures_ aja: dua dengan `join`, atau tiga dengan `join3`. Mari kita lihat 
gimana kita mungkin bisa bekerja pakai _futures_ yang jumlahnya lebih banyak 
lagi.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
