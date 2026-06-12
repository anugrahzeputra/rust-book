## Melihat Lebih Dekat Traits untuk Async

<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

Di sepanjang bab ini, kita udah memakai trait `Future`, `Pin`, `Unpin`, `Stream`, 
dan `StreamExt` pakai berbagai cara. Namun sejauh ini, kita menghindari buat 
masuk terlalu jauh ke detail soal gimana mereka bekerja atau gimana mereka 
saling terkait, yang mana itu tidak masalah buat sebagian besar pekerjaan Rust 
sehari-hari Anda. Tapi kadang-kadang, Anda bakal ketemu situasi di mana Anda 
perlu memahami sedikit lebih banyak soal detail-detail ini. Di bagian ini, kita 
bakal menggali secukupnya aja buat ngebantu di skenario-skenario tersebut, 
sambil tetap nyimpen penyelaman yang _benar-benar_ mendalam buat dokumentasi lain.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### Trait `Future`

Mari mulai dengan melihat lebih dekat gimana trait `Future` bekerja. Ini adalah 
gimana Rust mendefinisikannya:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Definisi trait itu mencakup sekumpulan tipe baru dan juga beberapa sintaks yang 
belum pernah kita lihat sebelumnya, jadi mari kita bedah definisinya sepotong 
demi sepotong.

Pertama, _associated type_ (tipe terkait) `Output` dari `Future` menyatakan apa 
hasil resolusi dari _future_ tersebut. Ini analog (mirip) dengan _associated type_ 
`Item` buat trait `Iterator`. Kedua, `Future` juga punya method `poll`, yang 
menerima referensi `Pin` spesial buat parameter `self`-nya dan referensi 
_mutable_ ke tipe `Context`, dan mengembalikan sebuah `Poll<Self::Output>`. Kita 
bakal ngomongin lebih lanjut soal `Pin` dan `Context` sebentar lagi. Buat 
sekarang, mari fokus ke apa yang dikembalikan sama method-nya, yaitu tipe `Poll`:

```rust
enum Poll<T> {
    Ready(T),
    Pending,
}
```

Tipe `Poll` ini mirip dengan sebuah `Option`. Dia punya satu varian yang punya 
nilai, `Ready(T)`, dan satu yang tidak, `Pending`. Tapi `Poll` punya arti yang 
cukup beda dari `Option` lho! Varian `Pending` (tertunda) mengindikasikan kalau 
_future_ tersebut masih punya kerjaan buat dilakuin, jadi si pemanggil fungsi 
bakal perlu mengecek lagi nanti. Varian `Ready` (siap) mengindikasikan kalau 
_future_-nya udah menyelesaikan kerjaannya dan nilai `T`-nya udah tersedia.

> Catatan: Pada sebagian besar _futures_, si pemanggil fungsi tidak seharusnya 
> memanggil `poll` lagi setelah _future_-nya memulangkan `Ready`. Banyak 
> _futures_ bakal mengalami *panic* kalau di-_poll_ lagi setelah mereka menjadi 
> siap. _Futures_ yang aman buat di-_poll_ lagi bakal bilang begitu secara 
> eksplisit di dokumentasi mereka. Ini mirip sama gimana `Iterator::next` 
> berperilaku.

Saat Anda melihat kode yang memakai `await`, di balik layar Rust 
mengompilasinya menjadi kode yang memanggil `poll`. Kalau Anda melihat kembali 
ke Listing 17-4, di mana kita mencetak judul halaman buat satu URL tunggal 
setelah dia di-resolve, Rust mengompilasinya menjadi sesuatu yang kira-kira (tapi 
tidak sama persis) kayak gini:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // Terus apa yang ditaruh di sini?
    }
}
```

Apa yang harus kita lakukan saat _future_-nya masih `Pending`? Kita butuh cara 
buat nyoba lagi, dan lagi, dan lagi, sampai _future_-nya akhirnya siap. Dengan 
kata lain, kita butuh sebuah _loop_:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

Tapi kalau Rust mengompilasinya persis kayak kode itu, setiap `await` bakal 
nge-blokir—yang mana itu kebalikan dari apa yang pengen kita capai! Sebaliknya, 
Rust memastikan kalau _loop_ tersebut bisa menyerahkan (hand off) kontrol ke 
sesuatu yang bisa mem-_pause_ kerjaan di _future_ ini buat ngerjain _futures_ 
lain lalu balik mengecek _future_ ini lagi nanti. Kayak yang udah kita lihat, 
"sesuatu" itu adalah sebuah _async runtime_, dan penjadwalan serta koordinasi ini 
adalah salah satu tugas utamanya.

Di awal bab ini, kita mendeskripsikan proses nge-_await_ `rx.recv`. Panggilan 
`recv` mengembalikan sebuah _future_, dan nge-_await_ _future_ tersebut bakal 
menge-_poll_-nya. Kita nyatat kalau _runtime_ bakal mem-_pause_ _future_-nya 
sampai dia siap ngeluarin entah `Some(message)` atau `None` pas _channel_-nya 
tutup. Dengan pemahaman kita yang lebih dalam soal trait `Future`, dan 
spesifiknya `Future::poll`, kita bisa melihat gimana itu bekerja. _Runtime_ 
tahu kalau _future_-nya belum siap pas dia mengembalikan `Poll::Pending`. 
Sebaliknya, _runtime_ tahu kalau _future_-nya *sudah* siap dan memajukan 
(_advances_) dia pas `poll` mengembalikan `Poll::Ready(Some(message))` atau 
`Poll::Ready(None)`.

Detail persisnya soal gimana sebuah _runtime_ melakukan hal itu ada di luar 
cakupan buku ini, tapi kuncinya adalah melihat mekanisme dasar dari _futures_: 
sebuah _runtime_ me-_poll_ (mengecek) setiap _future_ yang jadi tanggung 
jawabnya, lalu menidurkan (putting back to sleep) _future_ tersebut kalau dia 
belum siap.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>

### Trait `Pin` dan `Unpin`

Pas kita ngenalin ide tentang _pinning_ di Listing 17-16, kita ketemu sama pesan 
error yang ruwet (gnarly) banget. Ini bagian yang relevan dari pesannya lagi:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

Pesan error ini ngasih tahu kita tidak cuma kalau kita perlu nge-_pin_ nilai-
nilainya tapi juga kenapa _pinning_ itu dibutuhkan. Fungsi `trpl::join_all` 
mengembalikan struct bernama `JoinAll`. Struct itu bersifat generik terhadap tipe 
`F`, yang dibatasi (constrained) harus mengimplementasikan trait `Future`. Nge-
_await_ sebuah _future_ secara langsung pakai `await` otomatis nge-_pin_ _future_ 
tersebut secara implisit. Itulah kenapa kita tidak perlu memakai `pin!` di mana-
mana tiap kita mau nge-_await_ _futures_.

Namun, di sini kita tidak nge-_await_ sebuah _future_ secara langsung. Sebaliknya, 
kita membangun sebuah _future_ baru, `JoinAll`, dengan memasukkan sekumpulan 
(collection) _futures_ ke fungsi `join_all`. _Signature_ buat `join_all` 
mewajibkan agar tipe dari item-item di dalam koleksinya itu semuanya 
mengimplementasikan trait `Future`, dan `Box<T>` baru mengimplementasikan 
`Future` *kalau* `T` yang dia bungkus itu adalah sebuah _future_ yang 
mengimplementasikan trait `Unpin`.

Wah, banyak banget tuh yang harus dicerna! Buat bener-bener paham, mari kita gali 
sedikit lebih jauh lagi soal gimana trait `Future` sebenarnya bekerja, khususnya di 
seputar _pinning_ (pematrian).

Lihat lagi definisi dari trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Parameter `cx` dan tipe `Context`-nya adalah kunci gimana sebuah _runtime_ 
sebenarnya bisa tahu kapan harus mengecek _future_ apa pun padahal dia tetap 
bisa bersifat "malas" (lazy). Sekali lagi, detail gimana itu bekerja di luar 
cakupan bab ini, dan Anda umumnya cuma perlu mikirin ini kalau lagi nulis 
implementasi `Future` kustom. Sebaliknya, kita bakal fokus ke tipe buat `self`, 
karena ini pertama kalinya kita lihat sebuah method di mana `self` punya anotasi 
tipe. Anotasi tipe buat `self` bekerja kayak anotasi tipe buat parameter fungsi 
lainnya, tapi dengan dua perbedaan kunci:

- Dia ngasih tahu Rust `self` itu harus bertipe apa supaya method ini bisa 
  dipanggil.

- Dia tidak bisa berupa sembarang tipe. Dia dibatasi (restricted) cuma boleh 
  tipe yang diimplementasikan sama method tersebut, referensi atau _smart pointer_ 
  ke tipe tersebut, atau sebuah `Pin` yang ngebungkus referensi ke tipe tersebut.

Kita bakal lihat sintaks ini lebih banyak di [Bab 18][ch-18]. Buat sekarang, cukup 
tahu aja kalau kita mau menge-_poll_ (mengecek) sebuah _future_ buat melihat 
apakah dia `Pending` atau `Ready(Output)`, kita butuh sebuah referensi _mutable_ 
ke tipenya yang dibungkus di dalam `Pin`.

`Pin` adalah tipe pembungkus buat tipe-tipe mirip _pointer_ kayak `&`, `&mut`, 
`Box`, dan `Rc`. (Secara teknis, `Pin` bekerja buat tipe-tipe yang 
mengimplementasikan trait `Deref` atau `DerefMut`, tapi ini secara efektif sama 
aja dengan bilang bekerja cuma pakai _pointers_.) `Pin` bukanlah sebuah _pointer_ 
itu sendiri dan tidak punya perilaku (behavior) sendiri layaknya `Rc` dan `Arc` 
dengan _reference counting_-nya; dia murni adalah sebuah alat yang bisa dipakai 
sama _compiler_ buat menegakkan batasan-batasan (constraints) di penggunaan 
_pointer_.

Mengingat kembali bahwa `await` diimplementasikan dengan melakukan panggilan ke 
`poll` ini mulai ngebantu ngejelasin pesan error yang kita lihat sebelumnya, tapi 
waktu itu error-nya tentang `Unpin`, bukan `Pin`. Jadi gimana pastinya hubungan 
`Pin` sama `Unpin`, dan kenapa `Future` butuh `self` buat ada di dalam tipe `Pin` 
supaya bisa manggil `poll`?

Ingat kembali dari bagian awal bab ini kalau serangkaian titik _await_ di dalam 
sebuah _future_ di-compile menjadi sebuah _state machine_, dan _compiler_ 
memastikan _state machine_ itu ngikutin semua aturan normal Rust soal keamanan, 
termasuk _borrowing_ dan kepemilikan (ownership). Buat bikin itu bekerja, Rust 
melihat data apa aja yang dibutuhin antara satu titik _await_ dan entah titik 
_await_ berikutnya atau akhir dari blok _async_ tersebut. Terus dia bikin sebuah 
varian (variant) yang koresponden di dalam _state machine_ hasil kompilasinya. 
Setiap varian mendapat hak akses yang dia butuhkan ke data yang bakal dipakai 
di bagian *source code* (kode sumber) tersebut, baik dengan mengambil kepemilikan 
dari data itu atau dengan mendapatkan referensi _mutable_ atau _immutable_ 
kepadanya.

Sejauh ini semuanya bagus-bagus aja: kalau kita salah nulis sesuatu soal 
kepemilikan atau referensi di sebuah blok _async_ tertentu, si _borrow checker_ 
bakal ngasih tahu kita. Tapi pas kita mau mindahin (move around) _future_ yang 
berkorespondensi sama blok itu—seperti mindahin dia ke dalam sebuah `Vec` buat 
dioper ke `join_all`—urusannya jadi lebih rumit.

Pas kita memindahkan sebuah _future_—baik dengan masukin (_pushing_) dia ke 
sebuah struktur data buat dipakai sebagai _iterator_ dengan `join_all` atau dengan 
ngembaliin dia dari sebuah fungsi—itu sebenarnya berarti kita memindahkan _state 
machine_ yang dibikin Rust buat kita. Dan tidak kayak mayoritas tipe lainnya di 
Rust, _futures_ yang dibikin Rust buat blok _async_ bisa aja berakhir punya 
referensi ke diri mereka sendiri di _fields_ (bidang) dari varian apa pun, 
seperti yang ditunjukin dalam ilustrasi disederhanakan di Gambar 17-4.

<figure>

<img alt="Tabel tiga baris dengan satu kolom yang merepresentasikan sebuah future, fut1, yang punya nilai data 0 dan 1 di dua baris pertama dan sebuah panah yang menunjuk dari baris ketiga kembali ke baris kedua, yang merepresentasikan referensi internal di dalam future tersebut." src="img/trpl17-04.svg" class="center" />

<figcaption>Gambar 17-4: Sebuah tipe data yang merujuk pada dirinya sendiri (self-referential).</figcaption>

</figure>

Secara default, objek apa pun yang punya referensi ke dirinya sendiri itu 
tidak aman buat dipindahin (moved), karena referensi itu selalu menunjuk ke 
alamat memori aktual dari apa pun yang dia rujuk (lihat Gambar 17-5). Kalau 
Anda memindahkan struktur data itu sendiri, referensi-referensi internal tersebut 
bakal ditinggal dengan menunjuk ke lokasi yang lama. Namun, lokasi memori itu 
sekarang udah jadi tidak valid. Pertama, nilainya tidak bakal di-update pas 
Anda bikin perubahan ke struktur datanya. Kedua—yang lebih penting—komputer 
sekarang bebas buat memakai ulang memori itu buat tujuan lain! Anda bisa aja 
ngebaca data yang sama sekali tidak berhubungan nantinya.

<figure>

<img alt="Dua tabel, yang menggambarkan dua futures, fut1 dan fut2, di mana masing-masing punya satu kolom dan tiga baris, yang merepresentasikan hasil dari memindahkan sebuah future keluar dari fut1 ke dalam fut2. Yang pertama, fut1, diwarnai abu-abu, dengan tanda tanya di setiap indeksnya, merepresentasikan memori yang tidak diketahui. Yang kedua, fut2, punya 0 dan 1 di baris pertama dan kedua serta panah yang menunjuk dari baris ketiganya kembali ke baris kedua dari fut1, yang merepresentasikan pointer yang merujuk pada lokasi memori lama dari future tersebut sebelum dipindahkan." src="img/trpl17-05.svg" class="center" />

<figcaption>Gambar 17-5: Hasil yang tidak aman dari memindahkan tipe data self-referential.</figcaption>

</figure>

Secara teori, _compiler_ Rust bisa aja mencoba buat meng-update setiap referensi 
ke sebuah objek setiap kali dia dipindahkan, tapi itu bisa nambahin banyak beban 
performa (_performance overhead_), apalagi kalau ada seluruh jaring referensi yang 
perlu di-update. Kalau sebaliknya kita bisa mastiin kalau struktur data tersebut 
*tidak berpindah di memori*, kita tidak perlu meng-update referensi apa pun. Ini 
persisnya yang diwajibkan oleh _borrow checker_ Rust: di _safe code_ (kode aman), 
dia mencegah Anda dari memindahkan (moving) item apa pun yang punya referensi aktif 
kepadanya.

`Pin` dibangun di atas itu buat ngasih kita jaminan persis yang kita butuhin. Pas 
kita nge-_pin_ sebuah nilai dengan membungkus pointer ke nilai itu di dalam `Pin`, 
dia tidak bisa lagi dipindahkan. Oleh karena itu, kalau Anda punya `Pin<Box<SomeType>>`, 
Anda sebenarnya nge-_pin_ si nilai `SomeType`-nya, *bukan* pointer `Box`-nya. 
Gambar 17-6 mengilustrasikan proses ini.

<figure>

<img alt="Tiga kotak dijejer berdampingan. Yang pertama dilabeli “Pin”, yang kedua “b1”, dan yang ketiga “pinned”. Di dalam “pinned” ada sebuah tabel yang dilabeli “fut”, dengan satu kolom; tabel ini merepresentasikan sebuah future dengan sel buat tiap bagian dari struktur datanya. Sel pertamanya punya nilai “0”, sel keduanya punya panah yang keluar darinya dan menunjuk ke sel keempat dan terakhir, yang punya nilai “1” di dalamnya, dan sel ketiga punya garis putus-putus dan elipsis (...) buat menandakan mungkin ada bagian lain di struktur data tersebut. Secara keseluruhan, tabel “fut” ini merepresentasikan sebuah future yang self-referential. Sebuah panah keluar dari kotak berlabel “Pin”, ngelewatin kotak berlabel “b1” dan berujung di dalam kotak “pinned” di tabel “fut”." src="img/trpl17-06.svg" class="center" />

<figcaption>Gambar 17-6: Nge-pin sebuah `Box` yang menunjuk ke tipe future self-referential.</figcaption>

</figure>

Kenyataannya, pointer `Box` tersebut masih bisa berpindah-pindah dengan bebas. 
Inget: kita cuma peduli sama mastiin data yang pada akhirnya dirujuk itu tetap 
di tempatnya. Kalau sebuah pointer berpindah, *tapi data yang dia tunjuk ada di 
tempat yang sama*, seperti di Gambar 17-7, tidak ada potensi masalah. (Sebagai 
latihan independen, lihat dokumentasi buat tipe-tipe ini serta modul `std::pin` 
dan cobalah cari tahu gimana Anda bakal ngelakuin ini pakai `Pin` yang ngebungkus 
sebuah `Box`.) Kuncinya adalah tipe _self-referential_ (merujuk pada diri sendiri) 
itu sendiri yang tidak bisa berpindah, karena dia masih di-_pin_.

<figure>

<img alt="Empat kotak yang disusun di tiga kolom kasar, identik sama diagram sebelumnya dengan satu perubahan di kolom kedua. Sekarang ada dua kotak di kolom kedua, dilabeli “b1” dan “b2”, “b1” diwarnai abu-abu, dan panah dari “Pin” ngelewatin “b2” bukannya “b1”, mengindikasikan kalau pointernya udah pindah dari “b1” ke “b2”, tapi data di “pinned” belum pindah." src="img/trpl17-07.svg" class="center" />

<figcaption>Gambar 17-7: Memindahkan (moving) sebuah `Box` yang menunjuk ke tipe future self-referential.</figcaption>

</figure>

Namun, sebagian besar tipe itu sangat aman buat dipindah-pindah, biarpun mereka 
kebetulan berada di belakang pembungkus `Pin`. Kita cuma butuh mikirin soal 
_pinning_ (pematrian) pas item-item punya referensi internal. Nilai-nilai primitif 
kayak angka dan Boolean itu aman karena mereka udah jelas tidak punya referensi 
internal apa pun. Mayoritas tipe yang biasa Anda pakai di Rust juga gitu. Anda bisa 
mindah-mindahin sebuah `Vec`, misalnya, tanpa perlu khawatir. Asalkan kita cuma 
tahu apa yang udah kita lihat sejauh ini, kalau Anda punya sebuah `Pin<Vec<String>>`, 
Anda harus ngelakuin semuanya lewat API aman yang disediakan oleh `Pin` biarpun dia 
sangat membatasi (restrictive), padahal sebuah `Vec<String>` itu sebenarnya selalu 
aman buat dipindahin kalau tidak ada referensi lain ke dia. Kita butuh cara buat 
ngasih tahu _compiler_ kalau tidak apa-apa buat mindah-mindahin item di 
kasus-kasus kayak gini—nah, di sinilah `Unpin` mulai berperan.

`Unpin` adalah sebuah _marker trait_ (trait penanda), mirip kayak trait `Send` 
dan `Sync` yang kita lihat di Bab 16, dan oleh karena itu dia tidak punya 
fungsionalitas apa pun sendiri. _Marker traits_ cuma ada buat ngasih tahu 
_compiler_ kalau aman buat memakai tipe yang mengimplementasikan trait tersebut di 
dalam konteks tertentu. `Unpin` menginformasikan ke _compiler_ kalau suatu tipe 
tertentu *tidak* perlu menjunjung tinggi jaminan (guarantees) apa pun terkait apakah 
nilai tersebut bisa dipindahkan dengan aman.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

Sama halnya dengan `Send` dan `Sync`, _compiler_ mengimplementasikan `Unpin` 
secara otomatis buat semua tipe di mana dia bisa membuktikan kalau itu aman. Ada 
satu kasus spesial, sekali lagi mirip sama `Send` dan `Sync`, yaitu di mana 
`Unpin` *tidak* diimplementasikan buat sebuah tipe. Notasi buat ini adalah 
<code>impl !Unpin for <em>SomeType</em></code>, di mana 
<code><em>SomeType</em></code> adalah nama dari tipe yang *memang* perlu 
menjunjung tinggi jaminan-jaminan tersebut biar tetap aman kapan pun sebuah pointer 
ke tipe itu dipakai di dalam sebuah `Pin`.

Dengan kata lain, ada dua hal yang perlu diingat soal hubungan antara `Pin` dan 
`Unpin`. Pertama, `Unpin` itu adalah kasus "normal", dan `!Unpin` itu adalah kasus 
spesial. Kedua, apakah sebuah tipe mengimplementasikan `Unpin` atau `!Unpin` itu 
*cuma* penting pas Anda lagi memakai pointer yang di-_pin_ ke tipe tersebut seperti 
<code>Pin<&mut <em>SomeType</em>></code>.

Biar lebih jelas (concrete), mari pikirkan tentang sebuah `String`: dia punya 
sebuah _length_ (panjang) dan karakter-karakter Unicode yang membentuknya. Kita bisa 
ngebungkus sebuah `String` di dalam `Pin`, seperti yang terlihat di Gambar 17-8. 
Namun, `String` secara otomatis mengimplementasikan `Unpin`, sama kayak sebagian 
besar tipe lainnya di Rust.

<figure>

<img alt="Alur kerja konkuren" src="img/trpl17-08.svg" class="center" />

<figcaption>Gambar 17-8: Nge-pin sebuah `String`; garis putus-putus mengindikasikan kalau `String` mengimplementasikan trait `Unpin`, dan oleh karenanya tidak di-pin.</figcaption>

</figure>

Sebagai hasilnya, kita bisa melakukan hal-hal yang bakal jadi ilegal (dilarang) kalau 
seandainya `String` malah mengimplementasikan `!Unpin`, seperti mengganti satu 
string dengan string lainnya di lokasi memori yang sama persis seperti di Gambar 
17-9. Ini tidak melanggar kontrak `Pin`, karena `String` tidak punya referensi 
internal yang membikinnya tidak aman buat dipindah-pindah! Inilah persisnya kenapa dia 
mengimplementasikan `Unpin` dan bukannya `!Unpin`.

<figure>

<img alt="Alur kerja konkuren" src="img/trpl17-09.svg" class="center" />

<figcaption>Gambar 17-9: Mengganti `String` dengan `String` lain yang sepenuhnya beda di memori.</figcaption>

</figure>

Sekarang kita udah tahu cukup banyak buat memahami error-error yang dilaporin 
buat panggilan `join_all` dari balik lagi di Listing 17-17. Waktu itu kita nyoba 
mindahin _futures_ yang dihasilin sama blok _async_ ke dalam sebuah 
`Vec<Box<dyn Future<Output = ()>>>`, tapi kayak yang udah kita lihat, _futures_ 
itu bisa aja punya referensi internal, jadi mereka tidak mengimplementasikan 
`Unpin`. Mereka perlu di-_pin_, dan terus baru kita bisa masukin tipe `Pin`-nya 
ke dalam `Vec`, dengan yakin kalau data dasar (underlying data) di _futures_ 
tersebut *tidak* bakal dipindahin (moved).

`Pin` dan `Unpin` sebagian besar penting buat ngebangun _libraries_ tingkat 
lebih rendah (lower-level), atau pas Anda ngebangun sebuah _runtime_ itu sendiri, 
bukannya buat kode Rust sehari-hari. Namun, pas Anda melihat traits ini di pesan 
error, sekarang Anda bakal punya gambaran yang lebih baik soal gimana cara 
membenarkan kode Anda!

> Catatan: Kombinasi dari `Pin` dan `Unpin` ini membikin jadi mungkin buat dengan 
> aman mengimplementasikan keseluruhan kelas dari tipe-tipe kompleks di Rust yang 
> kalau tidak bakal susah banget (challenging) karena mereka itu self-referential 
> (merujuk ke dirinya sendiri). Tipe-tipe yang mewajibkan `Pin` paling sering 
> muncul di Rust _async_ hari ini, tapi sesekali, Anda mungkin bisa juga melihat 
> mereka di konteks yang lain.
>
> Penjelasan spesifik tentang gimana `Pin` dan `Unpin` bekerja, dan aturan-aturan 
> yang wajib mereka patuhi (uphold), dibahas secara mendalam di dokumentasi API 
> buat `std::pin`, jadi kalau Anda tertarik buat belajar lebih lanjut, itu adalah 
> tempat yang pas banget buat mulai.
>
> Kalau Anda mau paham gimana hal-hal bekerja di balik layar dengan lebih mendetail 
> lagi, lihat Bab [2][under-the-hood] dan [4][pinning] dari [_Asynchronous
> Programming in Rust_][async-book] (Pemrograman Asynchronous di Rust).

### Trait `Stream`

Sekarang setelah Anda punya pemahaman yang lebih dalam soal trait `Future`, `Pin`, 
dan `Unpin`, kita bisa mengalihkan perhatian kita ke trait `Stream`. Seperti 
yang Anda pelajari di awal bab, _streams_ itu mirip sama _iterators_ yang 
asynchronous. Namun, tidak seperti `Iterator` dan `Future`, `Stream` tidak 
punya definisi di _standard library_ sampai saat tulisan ini dibikin, tapi *ada* 
definisi yang sangat umum (common) dari _crate_ `futures` yang dipakai di 
seluruh ekosistem.

Mari kita bahas ulang (review) definisi dari trait `Iterator` dan `Future` 
sebelum melihat gimana trait `Stream` mungkin menggabungkan keduanya. Dari 
`Iterator`, kita punya ide tentang sebuah urutan (sequence): method `next`-nya 
menyediakan sebuah `Option<Self::Item>`. Dari `Future`, kita punya ide tentang 
kesiapan (readiness) seiring berjalannya waktu: method `poll`-nya menyediakan 
sebuah `Poll<Self::Output>`. Buat merepresentasikan urutan dari item-item yang 
jadi siap seiring berjalannya waktu, kita mendefinisikan trait `Stream` yang 
menyatukan fitur-fitur tersebut:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

Trait `Stream` mendefinisikan sebuah _associated type_ bernama `Item` buat tipe 
dari item-item yang dihasilkan oleh _stream_ tersebut. Ini mirip sama 
`Iterator`, di mana bisa ada nol atau banyak item, dan tidak kayak `Future`, 
di mana selalu cuma ada satu `Output` tunggal, meskipun itu cuma tipe unit `()`.

`Stream` juga mendefinisikan sebuah method buat ngambil item-item tersebut. 
Kita menamainya `poll_next`, buat ngejelasin kalau dia menge-_poll_ dengan cara 
yang sama seperti yang dilakuin sama `Future::poll` dan ngasilin urutan item 
dengan cara yang sama seperti yang dilakuin sama `Iterator::next`. Tipe 
kembaliannya ngegabungin `Poll` dengan `Option`. Tipe terluarnya (outer type) 
adalah `Poll`, karena dia harus dicek kesiapannya (readiness), persis kayak 
sebuah _future_. Tipe terdalamnya (inner type) adalah `Option`, karena dia 
perlu ngasih sinyal apakah ada lebih banyak pesan atau tidak, persis kayak 
sebuah _iterator_.

Sesuatu yang mirip banget kayak definisi ini kemungkinan besar bakal berujung jadi 
bagian dari _standard library_ Rust. Sementara itu, dia adalah bagian dari alat 
(toolkit) kebanyakan _runtimes_, jadi Anda bisa ngandelin dia, dan semua hal yang 
bakal kita bahas selanjutnya seharusnya secara umum tetap berlaku!

Namun, di contoh yang kita lihat di bagian soal streaming tadi, kita tidak 
memakai `poll_next` *atau* `Stream`, tapi sebaliknya memakai `next` dan 
`StreamExt`. Kita *bisa aja* bekerja secara langsung memakai API `poll_next` 
dengan menulis sendiri (_hand-writing_) _state machines_ buat `Stream` kita, 
sama kayak kalau kita *bisa aja* bekerja dengan _futures_ secara langsung via 
method `poll` mereka. Tapi memakai `await` itu jauh lebih enak lho, dan trait 
`StreamExt` menyediakan method `next` supaya kita bisa melakukan persis hal itu:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Catatan: Definisi aslinya yang kita pakai lebih awal di bab ini kelihatannya 
> agak sedikit beda dari ini, karena dia mendukung versi-versi Rust yang belum 
> mendukung pemakaian fungsi _async_ di dalam _traits_. Sebagai hasilnya, 
> kelihatannya kayak gini:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Tipe `Next` itu adalah sebuah `struct` yang mengimplementasikan `Future` dan 
> memungkinkan kita buat menamai _lifetime_ dari referensi ke `self` dengan 
> `Next<'_, Self>`, supaya `await` bisa bekerja sama method ini.

Trait `StreamExt` juga adalah rumah bagi semua method-method menarik yang tersedia 
buat dipakai bareng _streams_. `StreamExt` secara otomatis diimplementasikan buat 
setiap tipe yang mengimplementasikan `Stream`, tapi trait-trait ini didefinisikan 
secara terpisah buat memungkinkan komunitas beriterasi bikin API yang lebih nyaman 
tanpa memengaruhi trait fundamental aslinya (foundational trait).

Di versi `StreamExt` yang dipakai di _crate_ `trpl`, trait tersebut tidak cuma 
mendefinisikan method `next` tapi juga menyediakan implementasi default buat 
`next` yang menangani secara benar detail-detail panggilan ke `Stream::poll_next`. 
Ini artinya bahkan pas Anda perlu nulis tipe data streaming Anda sendiri, Anda 
*cuma* perlu mengimplementasikan `Stream`, dan kemudian siapa pun yang memakai 
tipe data Anda bisa memakai `StreamExt` beserta method-methodnya dengan tipe Anda 
secara otomatis.

Cukup sekian yang bakal kita bahas buat detail-detail tingkat rendah (lower-level) 
soal trait-trait ini. Sebagai penutup, mari pertimbangkan gimana _futures_ 
(termasuk _streams_), _tasks_ (tugas), dan _threads_ (utas) semuanya bekerja 
barengan jadi satu kesatuan!

[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
