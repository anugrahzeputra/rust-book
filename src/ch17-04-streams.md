## Streams: Futures Berurutan

<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

Sejauh ini di bab ini, kita sebagian besar berkutat dengan _futures_ individual. 
Satu pengecualian besarnya adalah _async channel_ yang kita pakai. Ingat 
kembali gimana kita memakai _receiver_ untuk _async channel_ kita sebelumnya di bab 
ini di bagian [“Menghitung Naik di Dua Task Memakai Message Passing”][17-02-messages]. 
Method _async_ `recv` menghasilkan serangkaian item seiring berjalannya waktu 
(over time). Ini adalah sebuah instance dari pola yang jauh lebih umum yang 
dikenal sebagai sebuah _stream_ (aliran).

Kita udah melihat serangkaian item dulu di Bab 13, waktu kita ngebahas trait 
`Iterator` di bagian [Trait Iterator dan Method `next`][iterator-trait], tapi ada 
dua perbedaan antara _iterators_ dan _async channel receiver_. Perbedaan pertama 
adalah waktu: _iterators_ itu sinkron (synchronous), sementara _channel receiver_ 
itu asinkron (asynchronous). Perbedaan kedua adalah API-nya. Pas bekerja secara 
langsung pakai `Iterator`, kita memanggil method `next`-nya yang sinkron. Buat 
_stream_ `trpl::Receiver` secara khusus, kita malah memanggil method `recv` 
yang asinkron. Selain dari itu, API-API ini terasa mirip banget, dan 
kemiripan ini bukanlah sebuah kebetulan. Sebuah _stream_ itu ibarat bentuk 
iterasi _asynchronous_. Kalau `trpl::Receiver` itu spesifik nunggu buat 
menerima pesan, API _stream_ yang bertujuan umum (general-purpose) itu jauh 
lebih luas: dia menyediakan item berikutnya (next item) seperti yang dilakukan 
sama `Iterator`, tapi secara _asynchronous_.

Kemiripan antara _iterators_ dan _streams_ di Rust berarti kita sebenarnya 
bisa membikin sebuah _stream_ dari _iterator_ mana pun. Sama kayak _iterator_, 
kita bisa bekerja pakai sebuah _stream_ dengan memanggil method `next`-nya 
lalu nge-_await_ hasil outputnya, kayak di Listing 17-30.

<Listing number="17-30" caption="Membikin sebuah stream dari sebuah iterator lalu mencetak nilai-nilainya" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-30/src/main.rs:stream}}
```

</Listing>

Kita mulai dengan sebuah *array* angka, yang mana kita konversi jadi 
sebuah _iterator_ lalu memanggil `map` padanya buat ngegandain (double) semua 
nilainya. Terus kita konversi _iterator_ itu jadi sebuah _stream_ dengan 
memakai fungsi `trpl::stream_from_iter`. Berikutnya, kita *loop* ngelewatin 
item-item di dalam _stream_ saat mereka berdatangan memakai _loop_ `while let`.

Sayangnya, pas kita nyoba jalanin kodenya, dia tidak bisa di-compile, malah 
dia ngelaporin kalau tidak ada method `next` yang tersedia:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-30
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = note: the full type name has been written to 'file:///projects/async-await/target/debug/deps/async_await-575db3dd3197d257.long-type-14490787947592691573.txt'
   = note: consider using `--verbose` to print the full type name to the console
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Seperti yang dijelasin sama output ini, alasan error _compiler_ ini terjadi 
adalah karena kita butuh trait yang tepat di dalam *scope* supaya bisa 
memakai method `next`. Berdasarkan pembahasan kita sejauh ini, Anda mungkin 
dengan masuk akal (reasonably) berharap kalau trait itu adalah `Stream`, tapi 
sebenarnya dia itu `StreamExt`. Singkatan dari _extension_ (ekstensi), `Ext` 
adalah pola yang umum di komunitas Rust buat memperluas satu trait dengan 
trait lainnya.

Kita bakal ngejelasin trait `Stream` dan `StreamExt` lebih detail sedikit 
lagi di akhir bab ini, tapi buat sekarang yang perlu Anda tahu adalah 
bahwa trait `Stream` mendefinisikan *interface* tingkat rendah (low-level) 
yang secara efektif menggabungkan trait `Iterator` dan `Future`. `StreamExt` 
menyediakan sekumpulan API tingkat lebih tinggi di atas `Stream`, termasuk 
method `next` dan method-method utilitas lainnya yang mirip sama yang 
disediakan oleh trait `Iterator`. `Stream` dan `StreamExt` belum jadi 
bagian dari _standard library_ Rust, tapi sebagian besar *crates* di 
ekosistem Rust memakai definisi yang sama.

Perbaikan buat error _compiler_ ini adalah menambahkan sebuah *statement* 
`use` buat `trpl::StreamExt`, seperti di Listing 17-31.

<Listing number="17-31" caption="Sukses memakai sebuah iterator sebagai basis buat sebuah stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-31/src/main.rs:all}}
```

</Listing>

Dengan semua kepingan itu digabungin, kode ini berjalan sesuai yang kita mau! 
Bahkan, karena kita udah punya `StreamExt` di dalam *scope*, kita bisa 
memakai semua method utilitasnya, persis kayak sama _iterators_. Misalnya, 
di Listing 17-32, kita memakai method `filter` buat menyaring (filter out) 
semua angka kecuali yang merupakan kelipatan tiga dan lima.

<Listing number="17-32" caption="Menyaring stream dengan method `StreamExt::filter`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-32/src/main.rs:all}}
```

</Listing>

Tentu saja, ini tidak terlalu menarik, karena kita bisa ngelakuin hal yang sama 
pakai _iterators_ biasa tanpa perlu _async_ sama sekali. Mari kita lihat 
apa yang bisa kita lakuin yang *unik* cuma buat _streams_.

### Menggabungkan (Composing) Streams

Banyak konsep secara natural (naturally) direpresentasikan sebagai _streams_: 
item-item yang jadi tersedia di dalam antrean (queue), bongkahan-bongkahan data 
(chunks of data) yang ditarik sedikit-sedikit dari sistem file saat dataset 
utuhnya terlalu besar buat muat di memori komputer, atau data yang berdatangan 
lewat jaringan seiring berjalannya waktu. Karena _streams_ adalah _futures_, kita 
bisa memakai mereka bareng _future_ jenis apa pun lainnya dan ngegabungin mereka 
pakai cara-cara yang menarik. Misalnya, kita bisa menge-kelompokkan (batch up) 
beberapa *events* (kejadian) buat menghindari memicu terlalu banyak panggilan 
jaringan, nge-set *timeouts* pada rentetan operasi yang lambat, atau mengerem 
(throttle) *events* dari *user interface* (antarmuka pengguna) buat menghindari 
ngelakuin kerjaan yang tidak perlu.

Mari mulai dengan ngebangun sebuah _stream_ pesan kecil-kecilan sebagai 
pengganti _stream_ data yang mungkin kita lihat dari sebuah WebSocket atau 
protokol komunikasi *real-time* lainnya, seperti yang ditunjukin di Listing 17-33.

<Listing number="17-33" caption="Memakai _receiver_ `rx` sebagai `ReceiverStream`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-33/src/main.rs:all}}
```

</Listing>

Pertama, kita membikin sebuah fungsi bernama `get_messages` yang mengembalikan 
`impl Stream<Item = String>`. Untuk implementasinya, kita membikin sebuah 
_async channel_, *loop* ngelewatin 10 huruf pertama alfabet Inggris, dan ngirim 
mereka lewat _channel_ tersebut.

Kita juga memakai sebuah tipe baru: `ReceiverStream`, yang mengonversi _receiver_ 
`rx` dari `trpl::channel` menjadi sebuah `Stream` yang punya method `next`. 
Balik lagi di `main`, kita memakai _loop_ `while let` buat mencetak semua pesan 
dari _stream_ tersebut.

Saat kita menjalankan kode ini, kita dapet hasil persis kayak yang kita harapin:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

Sekali lagi, kita bisa ngerjain ini pakai API `Receiver` biasa atau bahkan API 
`Iterator` biasa, jadi mari kita nambahin fitur yang emang butuh _streams_: 
nambahin *timeout* yang berlaku buat setiap item di dalam _stream_ tersebut, 
dan sebuah _delay_ (jeda) buat item-item yang kita keluarin (emit), seperti 
yang ditunjukin di Listing 17-34.

<Listing number="17-34" caption="Memakai method `StreamExt::timeout` buat ngeset batas waktu buat item-item di dalam sebuah stream" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-34/src/main.rs:timeout}}
```

</Listing>

Kita mulai dengan nambahin *timeout* ke _stream_ dengan memakai method `timeout`, 
yang mana berasal dari trait `StreamExt`. Terus kita meng-update _body_ dari 
_loop_ `while let`, karena _stream_-nya sekarang mengembalikan sebuah `Result`. 
Varian `Ok` mengindikasikan ada pesan yang nyampai tepat waktu; varian `Err` 
mengindikasikan kalau *timeout*-nya keburu abis sebelum ada pesan yang nyampai. 
Kita ngelakuin `match` pada hasil tersebut dan antara nyetak pesannya saat kita 
berhasil nerimanya atau nyetak pemberitahuan soal *timeout* itu. Terakhir, 
perhatikan kalau kita nge-_pin_ pesan-pesan itu setelah nerapin *timeout* 
kepadanya, karena fungsi pembantu *timeout* ngasilin sebuah _stream_ yang 
perlu di-_pin_ buat bisa di-_poll_.

Namun, karena tidak ada jeda (delays) di antara pesan-pesan tersebut, *timeout* ini 
tidak mengubah perilaku dari programnya. Mari kita nambahin jeda yang bervariasi 
ke pesan-pesan yang kita kirim, kayak yang ditunjukin di Listing 17-35.

<Listing number="17-35" caption="Mengirim pesan-pesan lewat `tx` dengan jeda async tanpa membikin `get_messages` jadi sebuah fungsi async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-35/src/main.rs:messages}}
```

</Listing>

Di `get_messages`, kita memakai method _iterator_ `enumerate` dengan *array* 
`messages` supaya kita bisa dapet indeks dari tiap item yang lagi kita kirim 
beserta item itu sendiri. Terus kita nerapin jeda 100 milidetik ke item dengan 
indeks genap (even-index items) dan jeda 300 milidetik ke item dengan indeks 
ganjil (odd-index items) buat mensimulasikan jeda yang beda-beda yang mungkin 
kita jumpai dari sebuah _stream_ pesan di dunia nyata. Karena *timeout* kita 
diset buat 200 milidetik, ini seharusnya memengaruhi separuh dari pesan-pesan 
tersebut.

Buat ngasih jeda (`sleep`) di antara setiap pesan di dalam fungsi `get_messages` 
tanpa memblokir (blocking), kita harus pakai _async_. Namun, kita tidak bisa 
ngebikin `get_messages` itu sendiri jadi fungsi _async_, karena kalau gitu 
kita bakal memulangkan (return) sebuah `Future<Output = Stream<Item = String>>` 
ketimbang `Stream<Item = String>>`. Si pemanggilnya (caller) bakal harus nge-_await_ 
`get_messages` itu sendiri buat dapet akses ke _stream_-nya. Tapi inget: semua hal 
di dalam sebuah _future_ tertentu itu terjadi secara linear; konkurensi terjadi 
_di antara_ (between) _futures_. Nge-_await_ `get_messages` bakal mewajibkan dia 
buat ngirim semua pesannya, termasuk nge-_sleep_ jeda di antara tiap pesannya, 
sebelum dia bisa memulangkan _receiver stream_-nya. Hasilnya, *timeout*-nya bakal 
jadi tidak berguna. Tidak bakal ada jeda di dalam _stream_ itu sendiri; semuanya 
udah kelar duluan sebelum _stream_-nya bahkan tersedia.

Sebagai gantinya, kita membiarkan `get_messages` jadi fungsi biasa yang 
mengembalikan sebuah _stream_, dan kita membikin (`spawn`) sebuah _task_ buat 
nangani pemanggilan `sleep` yang _async_ tersebut.

> Catatan: Manggil `spawn_task` pakai cara ini bisa jalan karena kita udah 
> nge-_setup_ _runtime_ kita sebelumnya; seandainya belum, ini bakal menyebabkan 
> *panic*. Implementasi-implementasi lain milih *tradeoffs* (pertukaran) yang beda: 
> mereka mungkin membikin _runtime_ baru dan menghindari *panic* tapi malah kena 
> sedikit _overhead_ tambahan, atau mereka bisa jadi tidak nyediain cara mandiri 
> (standalone) buat ngebikin _tasks_ tanpa merujuk ke sebuah _runtime_ sama sekali. 
> Pastikan Anda tahu *tradeoff* apa yang dipilih sama _runtime_ Anda dan tulis kode 
> Anda sesuai dengan hal itu!

Sekarang kode kita punya hasil yang jauh lebih menarik. Di sela-sela setiap 
pasang pesan, muncul error `Problem: Elapsed(())`.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

*Timeout* tersebut tidak mencegah pesan-pesannya buat nyampai di akhir. Kita 
tetep dapet semua pesan aslinya, karena _channel_ kita itu sifatnya tidak terbatas 
(unbounded): dia bisa nampung pesan sebanyak apa pun yang muat di memori kita. 
Kalau pesannya tidak nyampai sebelum *timeout* abis, _stream handler_ kita 
bakal nyatet itu, tapi pas dia nge-_poll_ _stream_-nya lagi, pesannya mungkin 
sekarang udah nyampai.

Anda bisa dapet perilaku yang beda kalau butuh dengan memakai jenis _channels_ 
lain atau jenis _streams_ lain pada umumnya. Mari kita lihat salah satunya beraksi 
dengan ngegabungin sebuah _stream_ yang ngeluarin interval waktu bareng _stream_ 
pesan ini.

### Menggabungkan (Merging) Streams

Pertama, mari kita bikin _stream_ lain, yang bakal ngeluarin item setiap 
milidetik kalau kita biarin dia jalan sendiri. Biar gampang, kita bisa pakai 
fungsi `sleep` buat ngirim pesan setelah _delay_ tertentu dan menggabungkannya 
pakai pendekatan yang sama kayak yang kita pakai di `get_messages` yaitu dengan 
bikin sebuah _stream_ dari sebuah _channel_. Bedanya adalah kali ini kita bakal 
ngirim balik jumlah interval yang udah terlewat, jadi tipe kembaliannya bakal 
berupa `impl Stream<Item = u32>`, dan kita bisa menamain fungsinya `get_intervals` 
(lihat Listing 17-36).

<Listing number="17-36" caption="Membikin sebuah stream yang isinya counter yang bakal dikeluarin sekali setiap milidetik" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-36/src/main.rs:intervals}}
```

</Listing>

Kita mulai dengan mendefinisikan sebuah `count` di dalam _task_ tersebut. (Kita 
bisa juga sih mendefinisikannya di luar _task_, tapi itu lebih jelas kalau kita 
mbatasin *scope* dari sebuah variabel.) Terus kita bikin sebuah *infinite loop* 
(perulangan tanpa henti). Setiap iterasi dari _loop_ tersebut bakal secara 
_asynchronous_ nge-_sleep_ selama satu milidetik, nambahin jumlah di `count`, dan 
lalu ngirim jumlah itu lewat _channel_-nya. Karena ini semua dibungkus di dalam 
_task_ yang dibikin sama `spawn_task`, semuanya—termasuk *infinite loop* tersebut—
bakal dibersihin (cleaned up) berbarengan sama _runtime_-nya.

Jenis *infinite loop* yang cuma berhenti pas seluruh _runtime_-nya dibongkar (torn 
down) ini lumayan umum di Rust _async_: banyak program yang emang butuh buat 
jalan terus tanpa henti (indefinitely). Dengan _async_, ini tidak bakal nge-blokir 
hal lain, asalkan ada minimal satu titik _await_ di tiap iterasi yang ngelewatin 
_loop_ tersebut.

Sekarang, kembali ke dalam blok _async_ di fungsi `main` kita, kita bisa mencoba 
buat menggabungkan (merge) _stream_ `messages` dan `intervals`, seperti yang 
ditunjukin di Listing 17-37.

<Listing number="17-37" caption="Mencoba buat ngegabungin stream `messages` dan `intervals`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-37/src/main.rs:main}}
```

</Listing>

Kita mulai dengan memanggil `get_intervals`. Terus kita gabungin _stream_ 
`messages` dan `intervals` pakai method `merge`, yang ngegabungin beberapa 
_stream_ jadi satu _stream_ yang ngeluarin item-item dari salah satu _stream_ 
sumbernya sesaat setelah item-item itu tersedia, tanpa maksain urutan (ordering) 
apa pun secara spesifik. Terakhir, kita *loop* ngelewatin _stream_ gabungan 
(combined stream) tersebut ketimbang cuma ngelewatin `messages`.

Pada titik ini, baik `messages` maupun `intervals` tidak perlu di-_pin_ atau 
_mutable_, karena keduanya bakal digabung ke dalam satu _stream_ tunggal `merged`. 
Namun, pemanggilan ke `merge` ini belum bisa di-compile! (Pemanggilan `next` 
di dalam _loop_ `while let` juga belum, tapi kita bakal balik lagi ke situ.) 
Ini karena kedua _stream_ itu punya tipe yang berbeda. _Stream_ `messages` 
punya tipe `Timeout<impl Stream<Item = String>>`, di mana `Timeout` adalah 
tipe yang mengimplementasikan `Stream` buat panggilan `timeout`. _Stream_ 
`intervals` punya tipe `impl Stream<Item = u32>`. Buat ngegabungin kedua 
_streams_ ini, kita perlu mentransformasi (transform) salah satu dari mereka 
biar cocok sama yang satunya lagi. Kita bakal merombak (rework) _stream_ intervals 
itu, karena messages udah ada di format dasar yang kita mau dan dia harus nanganin 
error *timeout* (lihat Listing 17-38).

<!-- We cannot directly test this one, because it never stops. -->

<Listing number="17-38" caption="Menyamakan tipe dari stream `intervals` dengan tipe stream `messages`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-38/src/main.rs:main}}
```

</Listing>

Pertama, kita bisa memakai method pembantu `map` buat mentransformasi `intervals` 
jadi sebuah string. Kedua, kita perlu mencocokkannya sama `Timeout` dari `messages`. 
Karena kita sebenarnya *tidak mau* *timeout* buat `intervals`, kita bisa 
aja bikin *timeout* yang durasinya lebih lama daripada durasi-durasi lain yang lagi 
kita pakai. Di sini, kita bikin *timeout* 10 detik pakai `Duration::from_secs(10)`. 
Terakhir, kita perlu membikin `stream`-nya jadi _mutable_, supaya pemanggilan 
`next` dari _loop_ `while let` bisa beriterasi melewati _stream_ tersebut, dan 
nge-_pin_ dia supaya aman buat ngelakuin itu. Kode ini ngebawa kita _hampir_ ke 
tempat yang kita butuhkan. Semua tipenya udah bener. Tapi kalau Anda menjalankan ini, 
bakal ada dua masalah. Pertama, dia tidak bakal pernah berhenti! Anda harus 
menghentikannya pakai <span class="keystroke">ctrl-c</span>. Kedua, pesan-pesan dari 
abjad bahasa Inggris bakal tenggelam (buried) di tengah-tengah semua pesan *counter* 
interval:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the tasks running differently rather than
changes in the compiler -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

Listing 17-39 nunjukin salah satu cara buat nyelesein dua masalah terakhir ini.

<Listing number="17-39" caption="Memakai `throttle` dan `take` buat nanganin streams yang digabungkan" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-39/src/main.rs:throttle}}
```

</Listing>

Pertama, kita pakai method `throttle` pada _stream_ `intervals` supaya dia tidak 
"menenggelamkan" (overwhelm) _stream_ `messages`. _Throttling_ (mengerem) adalah cara 
buat membatasi kecepatan (rate) seberapa sering sebuah fungsi bakal dipanggil—atau, 
di kasus ini, seberapa sering _stream_-nya bakal di-_poll_ (dicek). Sekali tiap 100 
milidetik harusnya udah cukup, karena itu kurang lebih sama dengan seberapa 
sering pesan kita berdatangan.

Buat ngebatesin jumlah item yang bakal kita terima dari sebuah _stream_, kita 
menerapkan method `take` pada _stream_ `merged` tersebut, karena kita pengen 
ngebatesin hasil akhir secara keseluruhan, bukan cuma satu _stream_ atau _stream_ 
yang satunya lagi.

Sekarang pas kita menjalankan programnya, dia berhenti setelah narik 20 item dari 
_stream_, dan interval-interval itu tidak lagi menenggelamkan pesan-pesannya. 
Kita juga tidak dapat `Interval: 100` atau `Interval: 200` atau semacamnya, tapi 
sebaliknya dapet `Interval: 1`, `Interval: 2`, dan seterusnya—biarpun kita punya 
_stream_ sumber yang *bisa* ngeluarin sebuah _event_ setiap milidetik. Itu karena 
panggilan `throttle` ngasilin sebuah _stream_ baru yang ngebungkus _stream_ aslinya 
sehingga _stream_ aslinya cuma di-_poll_ sesuai kecepatan *throttle*-nya, bukan 
kecepatan "aslinya" dia. Kita tidak membiarkan setumpuk pesan interval yang tidak 
terhandle menumpuk terus sengaja kita abaikan. Malah, dari awal kita bener-bener 
tidak pernah ngasilin pesan-pesan interval itu! Inilah bukti sifat dasar "malas" 
(laziness) dari _futures_ di Rust lagi beraksi, yang memungkinkan kita buat milih 
gimana kita mau ngatur karakteristik performa kita.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

Ada satu hal terakhir yang perlu kita tangani: error! Dengan kedua _streams_ 
berbasis _channel_ ini, pemanggilan `send` bisa gagal pas sisi sebelah sana dari 
_channel_-nya ditutup—dan itu cuma bergantung sama gimana _runtime_ mengeksekusi 
_futures_ yang ngebentuk _stream_ tersebut. Sampai sekarang, kita mengabaikan 
kemungkinan itu dengan memanggil `unwrap`, tapi di sebuah aplikasi yang berkelakuan 
baik (well-behaved app), kita seharusnya menangani error tersebut secara 
eksplisit, paling tidak dengan menghentikan _loop_-nya sehingga kita tidak perlu 
nyoba mengirim pesan lebih banyak lagi. Listing 17-40 nunjukin strategi penanganan 
error yang simpel: cetak masalahnya lalu nge-`break` keluar dari _loops_-nya.

<Listing number="17-40" caption="Nanganin error dan nge-shut down loops">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-40/src/main.rs:errors}}
```

</Listing>

Seperti biasa, cara yang bener buat nanganin error pas ngirim pesan itu 
bisa beda-beda; pastikan aja Anda punya suatu strategi buat nanganin itu.

Sekarang karena kita udah melihat serangkaian cara pakai _async_ di dunia nyata, 
mari kita mundur selangkah dan gali beberapa detail tentang gimana `Future`, 
`Stream`, dan trait kunci lainnya yang dipakai sama Rust buat ngebikin _async_ 
bisa bekerja.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
