## Bekerja dengan Sembarang Jumlah Futures

Pas kita beralih dari memakai dua _futures_ jadi tiga di bagian sebelumnya, kita 
juga harus beralih dari memakai `join` jadi memakai `join3`. Bakal nyebelin 
banget kalau kita harus manggil fungsi yang berbeda-beda setiap kali kita 
mengubah jumlah _futures_ yang mau kita gabungkan (_join_). Untungnya, kita 
punya bentuk macro dari `join` yang bisa kita kasih argumen dalam jumlah 
sembarang (arbitrary number of arguments). Macro ini juga sekalian nanganin 
proses nge-_await_ _futures_-nya sendiri. Jadi, kita bisa nulis ulang kode dari 
Listing 17-13 buat memakai `join!` ketimbang `join3`, seperti yang ada di 
Listing 17-14.

<Listing number="17-14" caption="Memakai `join!` buat menunggu beberapa futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:here}}
```

</Listing>

Ini jelas merupakan sebuah peningkatan ketimbang harus gonta-ganti antara `join` 
dan `join3` dan `join4` dan seterusnya! Namun, bahkan bentuk macro ini cuma 
bekerja saat kita udah tahu jumlah _futures_-nya dari awal (ahead of time). Di 
dunia nyata pemrograman Rust, masukin _futures_ ke dalam sebuah koleksi (collection) 
lalu menunggu sebagian atau semua _futures_ tersebut buat selesai adalah pola 
yang umum banget.

Buat ngecek semua _futures_ di suatu koleksi, kita butuh iterasi ngelewatin dan 
menge-_join_ _semua_ dari mereka. Fungsi `trpl::join_all` menerima tipe apa pun 
yang mengimplementasikan trait `Iterator`, yang mana udah Anda pelajari di Bab 
13 di bagian [Trait Iterator dan Method `next`][iterator-trait], jadi fungsi ini 
kayaknya pas banget buat kebutuhan kita. Mari kita coba menaruh _futures_ kita 
di dalam sebuah vector dan mengganti `join!` dengan `join_all` seperti yang 
ditunjukkan di Listing 17-15.

<Listing  number="17-15" caption="Menyimpan futures anonim di dalam vector dan memanggil `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

Sayangnya, kode ini tidak bisa di-compile. Sebaliknya, kita dapat error ini:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo build
copy just the compiler error
-->

```text
error[E0308]: mismatched types
  --> src/main.rs:45:37
   |
10 |         let tx1_fut = async move {
   |                       ---------- the expected `async` block
...
24 |         let rx_fut = async {
   |                      ----- the found `async` block
...
45 |         let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                     ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
              found `async` block `{async block@src/main.rs:24:22: 24:27}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and casting it to a trait object
```

Ini mungkin agak mengejutkan. Toh, tidak satu pun dari blok-blok _async_ itu yang 
mengembalikan apa-apa, jadi masing-masingnya menghasilkan `Future<Output = ()>`. 
Tapi ingat ya kalau `Future` itu adalah sebuah trait, dan _compiler_ membikin 
enum yang unik buat masing-masing blok _async_. Anda tidak bisa menaruh dua struct 
tulisan tangan (hand-written) yang berbeda di dalam sebuah `Vec`, dan aturan yang 
sama juga berlaku buat enum-enum berbeda yang di-generate sama _compiler_.

Biar kode ini bisa jalan, kita perlu memakai _trait objects_, sama persis kayak 
yang kita lakukan di [“Mengembalikan Error dari Fungsi run”][dyn] di Bab 12. 
(Kita bakal ngebahas _trait objects_ secara detail di Bab 18.) Memakai _trait 
objects_ membiarkan kita memperlakukan masing-masing _futures_ anonim yang 
dihasilkan oleh tipe-tipe ini sebagai tipe yang sama, karena semuanya 
mengimplementasikan trait `Future`.

> Catatan: Di [Memakai Enum Buat Nyimpen Banyak Tipe][enum-alt] di Bab 8, kita 
> udah ngebahas cara lain buat masukin berbagai tipe ke dalam `Vec`: memakai 
> sebuah enum buat ngewakilin tiap tipe yang mungkin muncul di dalam vector-nya. 
> Tapi kita tidak bisa ngelakuin itu di sini. Pertama, kita tidak punya cara buat 
> namain tipe-tipe yang berbeda-beda itu, karena mereka sifatnya anonim. Kedua, 
> alasan kenapa kita milih pakai vector dan `join_all` dari awal itu adalah buat 
> bisa bekerja dengan sebuah koleksi _futures_ yang dinamis di mana kita cuma peduli 
> kalau mereka punya tipe output yang sama.

Kita mulai dengan membungkus tiap _future_ di `vec!` di dalam sebuah `Box::new`, 
seperti yang ditunjukkan di Listing 17-16.

<Listing number="17-16" caption="Memakai `Box::new` buat menyamakan tipe-tipe futures di dalam `Vec`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Sayangnya, kode ini masih tetap tidak bisa di-compile. Malah, kita dapat error 
dasar yang sama kayak sebelumnya buat pemanggilan `Box::new` yang kedua dan 
ketiga, ditambah error-error baru yang menyebutkan trait `Unpin`. Kita bakal 
balik lagi ke error-error `Unpin` itu sebentar lagi. Pertama, mari kita perbaiki 
error tipe (type errors) pada pemanggilan `Box::new` dengan secara eksplisit 
menganotasi tipe dari variabel `futures` (lihat Listing 17-17).

<Listing number="17-17" caption="Memperbaiki sisa error type mismatch dengan memakai deklarasi tipe eksplisit" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

Deklarasi tipe ini lumayan panjang, jadi mari kita bedah satu-satu:

1. Tipe paling dalam adalah _future_ itu sendiri. Kita nyatet secara eksplisit 
   kalau output dari _future_ tersebut adalah tipe unit `()` dengan menulis 
   `Future<Output = ()>`.
2. Terus kita menganotasi trait-nya dengan `dyn` buat menandainya sebagai dinamis.
3. Seluruh referensi trait itu dibungkus di dalam sebuah `Box`.
4. Terakhir, kita nyatain secara eksplisit kalau `futures` adalah sebuah `Vec` 
   yang berisi item-item ini.

Itu udah membikin perbedaan yang gede banget lho. Sekarang pas kita menjalankan 
_compiler_, kita cuma dapat error-error yang menyebutkan soal `Unpin`. Walaupun 
ada tiga biji, isi pesannya mirip-mirip banget.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-17
cargo build
# copy *only* the errors
# fix the paths
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
   --> src/main.rs:49:24
    |
49  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `join_all`
   --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:9
   |
49 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:33
   |
49 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `async_await` (bin "async_await") due to 3 previous errors
```

Banyak banget tuh buat dicerna, jadi mari kita bedah pelan-pelan. Bagian 
pertama dari pesannya ngasih tahu kita kalau blok _async_ pertama 
(`src/main.rs:8:23: 20:10`) tidak mengimplementasikan trait `Unpin` dan 
menyarankan pemakaian `pin!` atau `Box::pin` buat menyelesaikannya. Nanti di bab 
ini, kita bakal gali beberapa detail lebih lanjut soal `Pin` dan `Unpin`. Tapi 
buat saat ini, kita bisa ikuti aja saran dari _compiler_ buat melangkah maju. Di 
Listing 17-18, kita mulai dengan meng-import `Pin` dari `std::pin`. Terus kita 
meng-update anotasi tipe buat `futures`, dengan sebuah `Pin` membungkus setiap 
`Box`. Terakhir, kita memakai `Box::pin` buat menge-_pin_ _futures_ itu sendiri.

<Listing number="17-18" caption="Memakai `Pin` dan `Box::pin` buat membikin tipe `Vec` lulus pengecekan tipe" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Kalau kita men-compile dan menjalankan ini, kita akhirnya dapet output yang kita 
harapkan:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

Fiuh!

Ada sedikit lagi yang perlu dieksplorasi di sini. Pertama, memakai `Pin<Box<T>>` 
itu menambahkan sedikit _overhead_ akibat menaruh _futures_ ini di _heap_ memakai 
`Box`—dan kita cuma ngelakuin itu buat mensejajarkan (line up) tipe-tipenya. Toh 
sebenarnya kita *nggak* butuh alokasi _heap_ itu: _futures_ ini sifatnya lokal buat 
fungsi tertentu ini. Seperti yang udah dicatat sebelumnya, `Pin` itu sendiri adalah 
sebuah tipe pembungkus (wrapper type), jadi kita bisa dapet keuntungan dari punya 
satu tipe tunggal di dalam `Vec`—yang merupakan alasan awal kita milih pakai `Box`—
tanpa harus melakukan alokasi _heap_. Kita bisa memakai `Pin` secara langsung 
dengan tiap _future_ dengan memakai macro `std::pin::pin`.

Namun, kita tetap harus secara eksplisit nyebutin tipe dari referensi yang di-_pin_ 
itu; kalau tidak, Rust masih tetap tidak tahu buat menerjemahkan mereka sebagai 
_dynamic trait objects_, yang mana itu adalah keharusan supaya mereka bisa masuk ke 
dalam `Vec`. Oleh karena itu, kita nambahin `pin` ke dalam daftar *imports* kita 
dari `std::pin`. Terus kita bisa nge-`pin!` tiap _future_ saat kita mendefinisikannya 
dan mendefinisikan `futures` sebagai sebuah `Vec` yang berisi referensi _mutable_ 
yang di-_pin_ ke tipe _dynamic future_ tersebut, seperti di Listing 17-19.

<Listing number="17-19" caption="Memakai `Pin` secara langsung dengan macro `pin!` buat menghindari alokasi _heap_ yang nggak perlu" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

Kita bisa melangkah sejauh ini dengan mengabaikan fakta kalau kita mungkin punya 
tipe `Output` yang berbeda-beda. Misalnya, di Listing 17-20, _future_ anonim 
buat `a` mengimplementasikan `Future<Output = u32>`, _future_ anonim buat `b` 
mengimplementasikan `Future<Output = &str>`, dan _future_ anonim buat `c` 
mengimplementasikan `Future<Output = bool>`.

<Listing number="17-20" caption="Tiga futures dengan tipe-tipe yang berbeda" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

Kita bisa memakai `trpl::join!` buat nge-_await_ mereka, karena dia memungkinkan 
kita buat memasukkan berbagai macam tipe _future_ dan menghasilkan sebuah tuple 
dari tipe-tipe tersebut. Kita *tidak bisa* memakai `trpl::join_all`, karena dia 
mewajibkan semua _futures_ yang dimasukkan ke dia buat punya tipe yang sama persis. 
Inget, error itulah yang jadi awal mula kita terjun ke petualangan bareng `Pin` ini!

Ini adalah _tradeoff_ (pertukaran) mendasar: kita bisa menangani jumlah _futures_ 
yang dinamis dengan `join_all`, asalkan mereka semua punya tipe yang sama, atau 
kita bisa menangani jumlah _futures_ yang sudah pasti (_set number_) dengan fungsi-fungsi 
`join` atau macro `join!`, bahkan jika mereka punya tipe yang berbeda. Ini adalah 
skenario yang sama kayak yang bakal kita hadapi saat bekerja dengan tipe-tipe 
lain di Rust. _Futures_ itu tidak spesial, biarpun kita punya beberapa sintaks 
bagus buat bekerja dengan mereka, dan itu adalah hal yang baik.

### Menandingkan (Racing) Futures

Pas kita "menggabungkan" (_join_) _futures_ dengan keluarga fungsi dan macro 
`join`, kita mewajibkan _semua_ _futures_ tersebut buat selesai sebelum kita 
bisa lanjut. Tapi kadang-kadang, kita cuma butuh _beberapa_ _future_ dari 
sebuah kelompok buat selesai sebelum kita lanjut—mirip kayak menandingkan 
(_racing_) satu _future_ ngelawan _future_ lainnya.

Di Listing 17-21, kita sekali lagi memakai `trpl::race` buat ngejalanin dua 
_futures_, `slow` dan `fast`, saling beradu satu sama lain.

<Listing number="17-21" caption="Memakai `race` buat ngambil hasil dari future mana pun yang kelar duluan" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

Masing-masing _future_ mencetak sebuah pesan saat mulai berjalan, ngasih jeda 
selama beberapa waktu dengan memanggil dan nge-_await_ `sleep`, dan terus 
mencetak pesan lain pas dia kelar. Terus kita ngoper baik `slow` maupun `fast` 
ke `trpl::race` dan nungguin salah satu dari mereka buat kelar. (Hasilnya di sini 
tidak terlalu mengejutkan: `fast` menang.) Tidak kayak waktu kita memakai `race` di 
[Program Async Pertama Kita][async-program], kita cuma mengabaikan aja instance 
`Either` yang dikembalikannya di sini, karena semua perilaku yang menarik terjadi 
di dalam _body_ dari blok-blok _async_ tersebut.

Perhatikan bahwa kalau Anda menukar urutan argumen ke `race`, urutan dari pesan 
"started" (dimulai) bakal berubah, biarpun _future_ `fast` bakal selalu kelar 
duluan. Itu karena implementasi dari fungsi `race` yang ini itu tidak adil. Dia 
selalu menjalankan _futures_ yang dikasih sebagai argumen sesuai urutan mereka 
dimasukkan. Implementasi-implementasi lain _ada_ yang adil dan bakal milih secara 
acak _future_ mana yang mau di-_poll_ duluan. Tapi terlepas dari apakah implementasi 
_race_ yang lagi kita pakai ini adil atau tidak, _salah satu_ dari _futures_ 
tersebut bakal jalan sampai ke _await_ pertama di dalam _body_-nya sebelum _task_ 
lain bisa mulai jalan.

Ingat kembali dari [Program Async Pertama Kita][async-program] bahwa di tiap 
titik _await_, Rust ngasih sebuah _runtime_ kesempatan buat mem-_pause_ _task_ 
tersebut lalu beralih ke _task_ lain kalau _future_ yang lagi di-_await_ itu belum 
siap. Kebalikannya juga benar: Rust *cuma* mem-_pause_ blok _async_ dan ngembaliin 
kontrol ke _runtime_ di suatu titik _await_. Semua hal di antara titik-titik 
_await_ itu berjalan secara sinkron (synchronous).

Itu artinya kalau Anda ngelakuin setumpuk pekerjaan di dalam sebuah blok _async_ 
tanpa adanya titik _await_, _future_ itu bakal nge-blokir _futures_ lain apa pun 
dari bikin progress. Anda mungkin kadang-kadang dengar ini disebut sebagai satu 
_future_ membikin _futures_ lain "kelaparan" (_starving_ other futures). Di 
beberapa kasus, itu mungkin bukan masalah besar. Namun, kalau Anda lagi ngelakuin 
semacam _setup_ yang mahal atau kerjaan yang butuh waktu lama, atau kalau Anda 
punya sebuah _future_ yang bakal terus-terusan ngelakuin satu _task_ tertentu tanpa 
henti (indefinitely), Anda bakal perlu mikirin soal kapan dan di mana harus 
ngembaliin kontrol ke _runtime_.

Dengan alasan yang sama, kalau Anda punya operasi-operasi _blocking_ yang memakan 
waktu lama, _async_ bisa jadi alat yang berguna buat nyediain cara-cara supaya 
berbagai bagian berbeda dari program bisa saling terhubung satu sama lain.

Tapi *gimana* caranya Anda ngembaliin kontrol ke _runtime_ di kasus-kasus 
kayak gitu?

<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Menyerahkan (Yielding) Kontrol ke Runtime

Mari kita simulasikan sebuah operasi yang memakan waktu lama. Listing 17-22 
memperkenalkan sebuah fungsi `slow`.

<Listing number="17-22" caption="Memakai `thread::sleep` buat mensimulasikan operasi yang lambat" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

Kode ini memakai `std::thread::sleep` ketimbang `trpl::sleep` supaya pemanggilan ke 
`slow` bakal memblokir _thread_ saat ini selama beberapa milidetik. Kita bisa 
memakai `slow` buat dijadiin pemeran pengganti buat operasi-operasi di dunia nyata 
yang sama-sama memakan waktu lama dan nge-blokir.

Di Listing 17-23, kita memakai `slow` buat pura-pura ngelakuin jenis kerjaan 
_CPU-bound_ ini di dalam sepasang _futures_.

<Listing number="17-23" caption="Memakai `thread::sleep` buat mensimulasikan operasi yang lambat" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:slow-futures}}
```

</Listing>

Sebagai awalan, setiap _future_ cuma ngembaliin kontrol ke _runtime_ *setelah* 
nyelesein sekumpulan operasi yang lambat. Kalau Anda jalanin kode ini, Anda bakal 
melihat output ini:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Kayak di contoh kita yang lebih awal, `race` tetap kelar sesaat setelah `a` kelar. 
Tapi tidak ada persilangan (interleaving) antara kedua _futures_ tersebut. _Future_ 
`a` ngelakuin semua kerjaannya sampai panggilan ke `trpl::sleep` di-_await_, 
baru habis itu _future_ `b` ngelakuin semua kerjaannya sampai panggilan 
`trpl::sleep`-nya sendiri di-_await_, lalu akhirnya _future_ `a` kelar. Biar kedua 
_futures_ bisa bikin progress di antara _tasks_ mereka yang lambat itu, kita butuh 
_await points_ (titik tunggu) supaya kita bisa ngembaliin kontrol ke _runtime_. 
Itu artinya kita butuh sesuatu yang bisa kita _await_!

Kita udah bisa melihat jenis serah-terima kontrol kayak gini terjadi di Listing 
17-23: kalau kita nghapus `trpl::sleep` di akhir dari _future_ `a`, dia bakal 
kelar tanpa _future_ `b` jalan *sama sekali*. Mari kita coba pakai fungsi `sleep` 
sebagai titik awal buat ngebiarin operasi-operasi ini ganti-gantian (switch off) 
bikin progress, kayak yang ditunjukin di Listing 17-24.

<Listing number="17-24" caption="Memakai `sleep` buat ngebiarin operasi ganti-gantian bikin progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

Di Listing 17-24, kita nambahin panggilan ke `trpl::sleep` dengan _await points_ 
di antara setiap panggilan ke `slow`. Sekarang kerjaan dari kedua _futures_ 
tersebut jadi bersilangan (interleaved):

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-24
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

_Future_ `a` masih jalan sebentar sebelum menyerahkan kontrol ke `b`, karena dia 
manggil `slow` dulu sebelum pernah manggil `trpl::sleep`, tapi setelah itu kedua 
_futures_ bertukar maju mundur setiap kali salah satu dari mereka kena _await point_. 
Di kasus ini, kita ngelakuin itu setelah setiap panggilan ke `slow`, tapi kita 
bisa juga ngepecah kerjaannya dengan cara apa pun yang paling masuk akal buat kita.

Tapi sebenarnya kita tidak benar-benar mau nge-_sleep_ di sini: kita cuma mau 
bikin progress secepat yang kita bisa. Kita cuma butuh ngembaliin kontrol ke 
_runtime_. Kita bisa ngelakuin itu secara langsung, memakai fungsi `yield_now`. 
Di Listing 17-25, kita ngegantiin semua panggilan ke `sleep` tadi sama `yield_now`.

<Listing number="17-25" caption="Memakai `yield_now` buat ngebiarin operasi ganti-gantian bikin progress" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-25/src/main.rs:yields}}
```

</Listing>

Kode ini selain lebih jelas maksud aslinya, dia juga bisa jauh lebih cepat 
ketimbang memakai `sleep`, karena *timers* (pengukur waktu) kayak yang dipakai 
sama `sleep` sering kali punya batas soal seberapa granular mereka bisa ngukur 
waktu. Versi `sleep` yang lagi kita pakai ini, contohnya, bakal selalu nge-_sleep_ 
minimal satu milidetik, bahkan kalau kita ngasih ke dia sebuah `Duration` bernilai 
satu nanodetik. Inget lho, komputer modern itu _cepat banget_: mereka bisa 
ngelakuin banyak hal dalam waktu satu milidetik!

Anda bisa lihat ini sendiri dengan nyiapin _benchmark_ (tes performa) kecil-kecilan, 
kayak yang ada di Listing 17-26. (Ini emang bukan cara yang paling teliti buat 
ngelakuin _performance testing_, tapi udah cukup buat nunjukin perbedaannya di sini.)

<Listing number="17-26" caption="Bandingin performa dari `sleep` dan `yield_now`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-26/src/main.rs:here}}
```

</Listing>

Di sini, kita ngabaiin (skip) semua proses pencetakan status, ngoper `Duration` 
satu nanodetik ke `trpl::sleep`, lalu membiarkan setiap _future_ jalan sendiri, 
tanpa ada pergantian (switching) antar _futures_. Terus kita jalanin selama 
1.000 iterasi lalu melihat berapa lama waktu yang dihabisin sama _future_ yang 
memakai `trpl::sleep` dibandingin sama _future_ yang memakai `trpl::yield_now`.

Versi yang pakai `yield_now` itu _jauuuh_ lebih cepat!

Ini artinya _async_ bisa berguna bahkan buat tugas-tugas _compute-bound_, 
tergantung dari apa lagi yang lagi dilakuin sama program Anda, karena dia nyediain 
alat yang berguna buat menstrukturkan hubungan antara berbagai bagian program 
yang berbeda. Ini adalah sebuah bentuk _cooperative multitasking_ (multitasking 
koperatif), di mana setiap _future_ punya wewenang (power) buat nentuin kapan dia 
nyerahin (hands over) kontrol via _await points_. Oleh karena itu, tiap _future_ 
juga punya tanggung jawab buat menghindari nge-blokir terlalu lama. Di beberapa 
sistem operasi _embedded_ (tertanam) berbasis Rust, ini adalah _satu-satunya_ 
jenis multitasking yang ada!

Di kode dunia nyata, tentu saja Anda tidak bakal sering-sering menyelingi 
panggilan fungsi sama _await points_ di setiap baris berturut-turut. Walaupun 
menyerahkan (yielding) kontrol dengan cara kayak gini itu lumayan tidak memakan 
banyak _resource_ (inexpensive), dia tidak gratis. Di banyak kasus, nyoba buat 
ngepecah tugas _compute-bound_ malah bisa bikin dia jadi lambat banget secara 
signifikan, jadi kadang-kadang bakal lebih baik buat performa _keseluruhan_ (overall) 
kalau ngebiarin sebuah operasi nge-blokir sebentar. Selalu lakukan pengukuran 
(measure) buat melihat di mana sebenarnya letak hambatan performa (performance 
bottlenecks) dari kode Anda. Sifat mendasar (underlying dynamic) yang satu ini 
tetap penting buat diingat, ya, apalagi kalau Anda _ngelihat_ ada banyak banget 
kerjaan yang terjadi secara sekuensial (serial) yang padahal Anda harapin bisa 
terjadi secara konkuren!

### Membikin Abstraksi Async Kita Sendiri

Kita juga bisa menggabungkan (compose) _futures_ barengan buat membikin pola-pola 
baru. Misalnya, kita bisa bikin sebuah fungsi `timeout` dengan blok-blok penyusun 
_async_ yang udah kita punya. Pas kita kelar, hasilnya bakal jadi sebuah blok 
penyusun lain yang bisa kita pakai buat bikin lebih banyak lagi abstraksi _async_.

Listing 17-27 nunjukin gimana kita ngarepin `timeout` ini bekerja bareng sebuah 
_future_ yang lambat.

<Listing number="17-27" caption="Memakai `timeout` imajinasi kita buat ngejalanin operasi yang lambat dengan batas waktu" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-27/src/main.rs:here}}
```

</Listing>

Mari kita implementasikan ini! Sebagai awalan, mari kita pikirin soal API buat 
`timeout`:

- Dia harus berupa fungsi _async_ itu sendiri supaya kita bisa nge-_await_ dia.
- Parameter pertamanya seharusnya adalah sebuah _future_ yang mau dijalankan. Kita 
  bisa membikinnya jadi generik biar dia bisa bekerja sama _future_ apa pun.
- Parameter keduanya bakal jadi waktu maksimal buat menunggu. Kalau kita memakai 
  sebuah `Duration`, itu bakal gampang dioper ke `trpl::sleep`.
- Dia seharusnya mengembalikan sebuah `Result`. Kalau _future_-nya kelar dengan 
  sukses, `Result`-nya bakal bernilai `Ok` dengan nilai yang dihasilin sama 
  _future_ tersebut. Kalau *timeout*-nya habis duluan, `Result`-nya bakal bernilai 
  `Err` dengan durasi (duration) waktu dia tadi nungguin _timeout_ tersebut.

Listing 17-28 nunjukin deklarasi ini.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-28" caption="Mendefinisikan *signature* dari `timeout`" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-28/src/main.rs:declaration}}
```

</Listing>

Itu menuhin tujuan-tujuan tipe kita. Sekarang mari kita pikirin soal _perilaku_ 
(behavior) yang kita butuhin: kita mau menandingkan (_race_) _future_ yang 
dimasukin ke dia ngelawan durasi waktunya. Kita bisa memakai `trpl::sleep` buat 
bikin _future_ _timer_ (pengukur waktu) dari durasi waktu tersebut, lalu memakai 
`trpl::race` buat ngejalanin _timer_ itu bareng _future_ yang dimasukkan sama 
si pemanggil fungsi.

Kita juga tahu kalau `race` itu tidak adil (not fair), dan mem-_poll_ argumen-
argumennya sesuai urutan mereka dimasukkan. Maka dari itu, kita mengoper 
`future_to_try` ke `race` duluan biar dia dapat kesempatan buat kelar bahkan kalau 
`max_time`-nya itu durasi yang sangat singkat. Kalau `future_to_try` kelar duluan, 
`race` bakal ngembaliin `Left` dengan output dari `future_to_try`. Kalau `timer` 
yang kelar duluan, `race` bakal ngembaliin `Right` dengan output `()` dari _timer_ 
tersebut.

Di Listing 17-29, kita melakukan `match` pada hasil dari nge-_await_ `trpl::race`.

<Listing number="17-29" caption="Mendefinisikan `timeout` pake `race` dan `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-29/src/main.rs:implementation}}
```

</Listing>

Kalau `future_to_try` sukses dan kita dapat `Left(output)`, kita ngembaliin 
`Ok(output)`. Kalau _timer_ dari *sleep* habis duluan dan kita dapat `Right(())`, 
kita ngabaiin `()` tersebut pake `_` lalu ngembaliin `Err(max_time)` sebagai 
gantinya.

Selesai deh, kita udah punya `timeout` yang bisa jalan yang dibangun dari dua fungsi 
pembantu (helpers) _async_ lainnya. Kalau kita jalanin kode kita, dia bakal 
mencetak mode kegagalannya setelah *timeout* habis:

```text
Failed after 2 seconds
```

Karena _futures_ itu digabungkan (compose) sama _futures_ lain, Anda bisa 
ngebangun alat-alat yang sangat _powerful_ memakai blok-blok penyusun _async_ 
yang lebih kecil. Misalnya, Anda bisa memakai pendekatan yang sama persis kayak 
ini buat ngegabungin *timeouts* dengan fitur coba lagi (retries), dan lalu memakai 
kombinasi itu bersama operasi-operasi seperti panggilan jaringan (salah satu contoh 
dari awal bab ini).

Di praktiknya, Anda biasanya bakal bekerja secara langsung pakai `async` dan 
`await`, dan kemudian dengan fungsi-fungsi dan *macros* seperti `join`, `join_all`, 
`race`, dan lain-lain. Anda cuma bakal butuh beralih buat pakai `pin` sesekali aja 
buat memakai _futures_ bersama API-API tersebut.

Kita udah melihat berbagai cara buat bekerja bareng banyak _futures_ di waktu yang 
bersamaan (at the same time). Habis ini, kita bakal melihat gimana kita bisa bekerja 
bareng banyak _futures_ secara berurutan seiring berjalannya waktu (over time) memakai 
_streams_. Tapi sebelum itu, ada beberapa hal lagi yang mungkin mau Anda coba pikirin 
dulu:

- Kita memakai sebuah `Vec` bareng `join_all` buat nungguin *semua* _futures_ di suatu 
  kelompok buat kelar. Gimana caranya Anda bisa memakai sebuah `Vec` buat memproses 
  sekelompok _futures_ secara berurutan sebagai gantinya? Apa aja _tradeoffs_ 
  (kelebihan/kekurangan) dari ngelakuin hal itu?

- Coba lihat-lihat tipe `futures::stream::FuturesUnordered` dari _crate_ `futures`. 
  Gimana perbedaannya kalau memakai itu ketimbang memakai `Vec`? (Jangan terlalu 
  musingin fakta kalau tipe itu berasal dari bagian `stream` di dalam _crate_ 
  tersebut; dia bisa bekerja dengan baik kok sama sembarang koleksi _futures_.)

[dyn]: ch12-03-improving-error-handling-and-modularity.html
[enum-alt]: ch08-01-vectors.html#using-an-enum-to-store-multiple-types
[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
