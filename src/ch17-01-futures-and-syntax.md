## Futures dan Sintaks Async

Elemen-elemen kunci dari pemrograman _asynchronous_ di Rust adalah _futures_ dan 
keyword `async` dan `await` di Rust.

Sebuah _future_ adalah nilai yang mungkin belum siap sekarang tapi bakal jadi siap 
di suatu waktu di masa depan (future). (Konsep yang sama ini juga muncul di banyak 
bahasa lain, kadang-kadang pakai nama lain kayak _task_ atau _promise_.) Rust 
menyediakan trait `Future` sebagai blok penyusun (building block) supaya berbagai 
operasi _async_ bisa diimplementasikan pakai struktur data yang beda-beda tapi 
dengan antarmuka (interface) yang sama. Di Rust, _futures_ adalah tipe-tipe yang 
mengimplementasikan trait `Future`. Tiap _future_ menyimpan informasinya sendiri 
soal sejauh mana kemajuan yang udah dibikin dan apa makna dari "siap" ("ready").

Anda bisa menerapkan keyword `async` ke blok dan fungsi buat menentukan kalau mereka 
bisa diinterupsi dan dilanjutkan lagi (resumed). Di dalam sebuah blok _async_ atau 
fungsi _async_, Anda bisa memakai keyword `await` buat _menunggu sebuah future_ 
(_await a future_) (yaitu, nunggu dia sampai jadi siap). Titik mana pun di mana 
Anda me-_await_ sebuah _future_ di dalam sebuah blok atau fungsi _async_ adalah titik 
potensial buat blok atau fungsi _async_ itu buat nge-_pause_ dan di-_resume_. Proses 
ngecek ke sebuah _future_ buat melihat apakah nilainya udah tersedia atau belum ini 
disebut _polling_.

Beberapa bahasa pemrograman lain, kayak C# sama JavaScript, juga memakai keyword 
`async` dan `await` buat pemrograman _async_. Kalau Anda udah familier sama 
bahasa-bahasa itu, Anda mungkin bakal sadar ada beberapa perbedaan signifikan dari 
cara Rust melakukan hal tersebut, termasuk cara nanganin sintaksnya. Itu ada alasan 
bagusnya lho, kayak yang bakal kita lihat nanti!

Pas lagi nulis Rust _async_, kita bakal memakai keyword `async` dan `await` 
sebagian besar waktunya. Rust mengompilasi mereka jadi kode yang ekuivalen 
menggunakan trait `Future`, mirip kayak gimana dia mengompilasi `for` _loops_ jadi 
kode yang ekuivalen memakai trait `Iterator`. Tapi karena Rust nyediain trait 
`Future`, Anda juga bisa mengimplementasikannya buat tipe data Anda sendiri 
kalau Anda butuh. Banyak fungsi yang bakal kita lihat di sepanjang bab ini 
mengembalikan tipe yang punya implementasi `Future`-nya masing-masing. Kita bakal 
balik lagi ke definisi trait-nya di akhir bab ini dan gali lebih dalem soal 
gimana cara kerjanya, tapi detail segini udah cukup buat kita lanjut jalan dulu.

Ini semua mungkin terasa agak abstrak, jadi mari kita tulis program _async_ 
pertama kita: sebuah _web scraper_ mini. Kita bakal memasukkan dua URL dari 
_command line_, nge-_fetch_ (ngambil data dari) kedua URL itu secara konkuren, 
terus mengembalikan hasil dari URL mana pun yang selesai duluan. Contoh ini 
bakal punya lumayan banyak sintaks baru, tapi jangan khawatir—kita bakal jelasin 
semua yang perlu Anda tahu seiring kita berjalan.

## Program Async Pertama Kita

Biar fokus bab ini tetap pada belajar _async_ ketimbang sibuk ngerjain bagian-
bagian ekosistem, kita udah ngebikin _crate_ `trpl` (`trpl` itu singkatan dari 
"The Rust Programming Language"). _Crate_ ini nge-_re-export_ (ekspor ulang) semua 
tipe, trait, dan fungsi yang bakal Anda butuhin, utamanya dari _crate_ 
[`futures`][futures-crate] dan [`tokio`][tokio]. _Crate_ `futures` adalah tempat 
resmi buat bereksperimen dengan kode _async_ di Rust, dan di sanalah trait `Future` 
asal-muasalnya didesain. Tokio adalah _async runtime_ yang paling banyak dipakai 
di Rust saat ini, apalagi buat aplikasi web. Ada banyak _runtimes_ keren lainnya 
di luar sana, dan mereka mungkin lebih cocok buat kebutuhan Anda. Kita memakai 
_crate_ `tokio` di balik layarnya `trpl` karena dia sudah teruji dengan baik dan 
banyak dipakai.

Di beberapa kasus, `trpl` juga me-*rename* atau ngebungkus (wraps) API aslinya 
biar Anda tetap fokus sama detail-detail yang relevan buat bab ini. Kalau Anda 
mau paham apa yang dilakukan sama _crate_ ini, silakan aja cek 
[_source code_-nya][crate-source]. Anda bakal bisa lihat dari _crate_ mana asal dari 
tiap fitur yang di-_re-export_, dan kita udah ninggalin komentar yang ekstensif yang 
ngejelasin apa aja yang dilakuin sama _crate_ tersebut.

Bikin sebuah project *binary* baru bernama `hello-async` dan tambahin _crate_ 
`trpl` sebagai *dependency*:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Sekarang kita bisa pakai berbagai potongan yang disediain sama `trpl` buat nulis 
program _async_ pertama kita. Kita bakal bikin alat _command line_ mini yang 
nge-_fetch_ dua halaman web, narik elemen `<title>` dari masing-masing halaman, 
terus nyetak judul dari halaman mana pun yang nyelesein seluruh proses tersebut 
paling duluan.

### Mendefinisikan Fungsi page_title

Mari mulai dengan nulis fungsi yang menerima satu URL halaman sebagai parameternya, 
melakukan _request_ ke sana, dan mengembalikan teks dari elemen judulnya (lihat 
Listing 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Mendefinisikan fungsi async buat dapet elemen title dari halaman HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Pertama, kita mendefinisikan sebuah fungsi bernama `page_title` dan menandainya 
pakai keyword `async`. Terus kita pakai fungsi `trpl::get` buat nge-_fetch_ 
URL apa pun yang dimasukkan dan nambahin keyword `await` buat nungguin (await) 
respons-nya. Buat dapet teks dari respons tersebut, kita memanggil method `text`-nya, 
dan sekali lagi menunggunya dengan keyword `await`. Kedua langkah ini sifatnya 
_asynchronous_. Buat fungsi `get`, kita harus nunggu server buat ngirim balik 
bagian pertama dari responsnya, yang mana bakal berisi HTTP _headers_, _cookies_, 
dan lain-lain, dan bisa aja dikirim secara terpisah dari _body_ (isi utama) 
responsnya. Apalagi kalau _body_-nya itu besar banget, itu bisa memakan waktu yang 
lumayan lama buat semuanya sampai (arrive). Karena kita harus nunggu buat 
_keseluruhan_ responsnya sampai, method `text` itu juga _async_.

Kita harus secara eksplisit nge-_await_ kedua _futures_ ini, karena _futures_ 
di Rust itu _lazy_ (malas): mereka tidak bakal ngelakuin apa-apa sampai Anda 
menyuruh mereka dengan keyword `await`. (Faktanya, Rust bakal ngeluarin peringatan 
_compiler_ kalau Anda tidak nge-_await_ sebuah _future_.) Ini mungkin ngingetin 
Anda soal pembahasan _iterators_ di Bab 13 di bagian [Memproses Serangkaian Item 
dengan Iterators][iterators-lazy]. _Iterators_ tidak bakal ngelakuin apa-apa 
kecuali kalau Anda memanggil method `next`-nya—baik itu secara langsung atau dengan 
memakai `for` _loops_ atau method-method kayak `map` yang memakai `next` di 
balik layarnya. Demikian juga, _futures_ tidak bakal ngelakuin apa-apa kecuali 
Anda menyuruh mereka secara eksplisit. Sifat _lazy_ ini ngebolehin Rust menghindari 
menjalankan kode _async_ sampai dia bener-bener dibutuhkan.

> Catatan: Ini berbeda dari perilaku yang kita lihat di bab sebelumnya saat 
> memakai `thread::spawn` di [Membikin Thread Baru dengan spawn][thread-spawn], 
> di mana _closure_ yang kita kasih ke _thread_ lain itu langsung mulai berjalan. 
> Ini juga beda dari cara banyak bahasa lain melakukan pendekatan ke _async_. Tapi 
> penting buat Rust buat bisa ngasih jaminan performanya, sama halnya dengan 
> _iterators_.

Begitu kita dapet `response_text`, kita bisa nge-_parse_ nilainya jadi sebuah instance 
dari tipe `Html` menggunakan `Html::parse`. Daripada pakai string mentah, sekarang 
kita punya sebuah tipe data yang bisa kita pakai buat ngolah HTML tersebut sebagai 
struktur data yang lebih kaya (richer). Secara spesifik, kita bisa pakai method 
`select_first` buat nemuin instance pertama dari CSS _selector_ yang kita kasih. 
Dengan memasukkan string `"title"`, kita bakal dapet elemen `<title>` pertama di 
dokumen tersebut, kalau emang ada. Karena mungkin aja nggak ada elemen yang cocok, 
`select_first` mengembalikan `Option<ElementRef>`. Terakhir, kita memakai 
method `Option::map`, yang ngebolehin kita beroperasi pada item di dalam `Option` 
kalau itemnya ada, dan tidak ngelakuin apa-apa kalau itemnya nggak ada. (Kita juga 
bisa aja pakai ekspresi `match` di sini, tapi `map` itu lebih idiomatik.) Di dalam 
_body_ fungsi yang kita berikan ke `map`, kita memanggil `inner_html` pada `title` 
buat mendapatkan kontennya, yang mana adalah sebuah `String`. Pada akhirnya, 
kita punya sebuah `Option<String>`.

Perhatikan bahwa keyword `await` di Rust ditaruh _setelah_ ekspresi yang lagi Anda 
tungguin, bukan di sebelumnya. Yakni, ia adalah sebuah keyword _postfix_ (akhiran). 
Ini mungkin beda dari apa yang biasa Anda jumpai kalau Anda pernah memakai `async` 
di bahasa lain, tapi di Rust ini bikin _chaining_ (rentetan) pemanggilan method 
jadi jauh lebih enak buat dibikin. Hasilnya, kita bisa ngubah _body_ dari 
`page_title` buat nge-_chain_ pemanggilan fungsi `trpl::get` dan `text` sekaligus 
pake `await` di antara mereka, kayak yang ditunjukin di Listing 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Chaining (menyambung) dengan keyword `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Selesai deh, kita udah berhasil nulis fungsi _async_ pertama kita! Sebelum kita nambahin 
beberapa kode di `main` buat manggil dia, mari kita ngomongin lebih lanjut soal 
apa yang udah kita tulis ini dan apa maknanya.

Pas Rust ngelihat ada blok yang ditandain pake keyword `async`, dia mengompilasinya 
jadi tipe data anonim unik yang mengimplementasikan trait `Future`. Pas Rust ngelihat 
fungsi ditandain pake `async`, dia mengompilasinya jadi fungsi non-async yang 
_body_-nya merupakan sebuah blok _async_. Tipe kembalian fungsi _async_ adalah 
tipe dari tipe data anonim yang dibikin sama _compiler_ buat blok _async_ tersebut.

Jadi, nulis `async fn` itu ekuivalen (sama aja) kayak nulis fungsi yang mengembalikan 
sebuah _future_ dari tipe kembaliannya. Bagi _compiler_, definisi fungsi kayak 
`async fn page_title` di Listing 17-1 itu sama aja kayak fungsi non-async yang 
didefinisiin kayak gini:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Mari kita telusuri bagian demi bagian dari versi yang udah diubah (transformed) ini:

- Dia memakai sintaks `impl Trait` yang udah kita bahas dulu di Bab 10 di 
  bagian ["Traits sebagai Parameter"][impl-trait].
- Trait yang dikembalikan adalah sebuah `Future` dengan _associated type_ `Output`. 
  Perhatikan bahwa tipe `Output`-nya adalah `Option<String>`, yang mana sama dengan 
  tipe kembalian asli dari versi `async fn` si `page_title`.
- Semua kode yang dipanggil di dalam _body_ dari fungsi aslinya dibungkus di dalam 
  sebuah blok `async move`. Ingat kembali kalau blok itu adalah ekspresi. Keseluruhan 
  blok ini adalah ekspresi yang dikembalikan dari fungsinya.
- Blok _async_ ini ngasilin nilai bertipe `Option<String>`, seperti yang baru aja 
  dijelasin. Nilai tersebut cocok sama tipe `Output` di tipe kembaliannya. Ini sama 
  persis kayak blok-blok lain yang pernah Anda lihat.
- _Body_ fungsi baru tersebut adalah sebuah blok `async move` gara-gara gimana 
  dia memakai parameter `url`. (Kita bakal ngebahas lebih banyak lagi soal `async` 
  versus `async move` nanti di bab ini.)

Sekarang kita bisa memanggil `page_title` di `main`.

## Menentukan Judul Satu Halaman Tunggal

Sebagai awalan, kita cuma bakal ngambil judul buat satu halaman aja. Di Listing 17-3, 
kita mengikuti pola yang sama yang kita pakai di Bab 12 buat dapet argumen 
_command line_ di bagian [Menerima Argumen Command Line][cli-args]. Terus kita 
mengoper URL pertamanya ke `page_title` dan nge-_await_ hasilnya. Karena nilai yang 
dihasilin oleh _future_ tersebut adalah sebuah `Option<String>`, kita memakai ekspresi 
`match` buat mencetak pesan yang beda-beda dengan memperhitungkan (account for) 
apakah halamannya punya `<title>` atau tidak.

<Listing number="17-3" file-name="src/main.rs" caption="Manggil fungsi `page_title` dari `main` memakai argumen yang dikasih sama user">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Sayangnya, kode ini tidak bisa di-compile. Satu-satunya tempat di mana kita bisa 
memakai keyword `await` adalah di dalam fungsi atau blok _async_, dan Rust tidak bakal 
ngebiarin kita nandain fungsi spesial `main` sebagai `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

Alasan `main` tidak bisa ditandain `async` adalah karena kode _async_ itu butuh 
sebuah _runtime_: sebuah _crate_ Rust yang mengelola detail pengeksekusian kode 
_asynchronous_. Fungsi `main` di sebuah program bisa _menginisialisasi_ (initialize) 
sebuah _runtime_, tapi fungsi `main` itu bukanlah sebuah _runtime_ itu sendiri. 
(Kita bakal lihat lebih lanjut soal kenapa ini terjadi sebentar lagi.) Setiap program 
Rust yang mengeksekusi kode _async_ punya minimal satu tempat di mana dia nge-_setup_ 
_runtime_ lalu mengeksekusi _futures_-nya.

Kebanyakan bahasa yang mendukung _async_ sudah membundel (bundle) sebuah _runtime_ 
bawaan, tapi Rust tidak begitu. Sebaliknya, ada banyak _async runtimes_ berbeda 
yang tersedia, di mana masing-masing membikin _tradeoffs_ (pertukaran) yang cocok 
buat _use case_ (kasus penggunaan) yang jadi targetnya. Misalnya, _web server_ 
yang _high-throughput_ (kemampuan transmisi besar) dengan banyak _core_ CPU dan 
RAM dalam jumlah besar punya kebutuhan yang sangat berbeda dari mikrokontroler 
dengan _single core_, jumlah RAM yang kecil, dan tidak punya kemampuan alokasi 
_heap_ sama sekali. Crate yang nyediain _runtimes_ ini juga sering kali ngasih 
versi _async_ dari fungsionalitas umum kayak I/O file atau jaringan.

Di sini, dan di sepanjang sisa bab ini, kita bakal memakai fungsi `run` dari _crate_ 
`trpl`, yang mana nerima sebuah _future_ sebagai argumen lalu menjalankannya 
sampai selesai (completion). Di balik layar, manggil `run` bakal nge-_setup_ sebuah 
_runtime_ yang dipakai buat ngejalanin _future_ yang dikasih. Setelah _future_ 
tersebut selesai, `run` bakal mengembalikan nilai apa pun yang dihasilin oleh 
_future_ itu.

Kita bisa aja meneruskan _future_ yang dikembalikan sama `page_title` langsung 
ke `run`, dan begitu selesai, kita bisa melakukan `match` pada `Option<String>` yang 
dihasilkannya, kayak yang udah kita coba lakuin di Listing 17-3. Tapi, buat 
mayoritas contoh di bab ini (dan mayoritas kode _async_ di dunia nyata), kita 
bakal ngelakuin lebih dari sekadar satu pemanggilan fungsi _async_ aja, jadi 
alih-alih begitu kita bakal meneruskan sebuah blok `async` lalu secara eksplisit 
nge-_await_ hasil dari panggilan `page_title`, seperti di Listing 17-4.

<Listing number="17-4" caption="Nge-await sebuah blok async dengan `trpl::run`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Pas kita jalanin kode ini, kita dapet perilaku kayak yang kita harapin di awal:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run https://www.rust-lang.org
# copy the output here
-->

```console
$ cargo run -- https://www.rust-lang.org
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Fiuh—akhirnya kita punya kode _async_ yang jalan! Tapi sebelum kita nambahin 
kode buat nandingin (race) kedua situs web itu satu sama lain, mari kita alihkan 
sejenak perhatian kita kembali ke gimana _futures_ itu bekerja.

Setiap _await point_ (titik penantian)—yakni, di mana aja kodenya memakai keyword 
`await`—merepresentasikan sebuah tempat di mana kontrol dikembalikan (_handed back_) 
ke _runtime_. Biar itu bisa terjadi, Rust perlu melacak (keep track of) _state_ 
(keadaan) yang terlibat di dalam blok _async_ tersebut sehingga _runtime_ bisa 
memulai beberapa pekerjaan lain lalu balik lagi nanti kalau dia udah siap buat 
mencoba melanjutkan (_advancing_) pekerjaan pertama tadi. Ini adalah sebuah 
_state machine_ (mesin keadaan) kasatmata, seolah-olah Anda menulis sebuah enum 
kayak gini buat menyimpan _state_ saat ini di tiap titik _await_:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Nulis kode buat bertransisi di antara setiap _state_ ini secara manual (by hand) 
bakal makan waktu dan gampang rawan error, apalagi kalau nanti Anda harus 
nambahin fungsionalitas dan lebih banyak _states_ lagi ke kode tersebut. Untungnya, 
_compiler_ Rust otomatis membikin dan mengelola struktur data _state machine_ buat 
kode _async_. Aturan-aturan _borrowing_ dan _ownership_ normal seputar struktur data 
itu tetap berlaku semua, dan syukurnya, _compiler_ juga nanganin pengecekan itu 
buat kita dan menyediakan pesan error yang berguna. Kita bakal membedah beberapa 
kasus kayak gitu nanti di bab ini.

Pada akhirnya, sesuatu harus mengeksekusi _state machine_ ini, dan "sesuatu" itu 
adalah sebuah _runtime_. (Inilah kenapa Anda mungkin pernah ketemu istilah _executors_ 
(pengeksekusi) pas lagi nyari tahu soal _runtimes_: sebuah _executor_ adalah bagian 
dari sebuah _runtime_ yang bertugas mengeksekusi kode _async_ tersebut.)

Sekarang Anda bisa tahu kenapa _compiler_ ngelarang kita membikin `main` itu 
sendiri jadi fungsi _async_ balik di Listing 17-3 tadi. Kalau `main` itu fungsi 
_async_, sesuatu yang lain bakal harus mengelola _state machine_ buat _future_ 
apa pun yang dikembaliin sama `main`, tapi padahal `main` adalah titik awal 
(starting point) buat programnya! Sebaliknya, kita memanggil fungsi `trpl::run` di 
`main` buat nge-_setup_ sebuah _runtime_ lalu ngejalanin _future_ yang dikembalikan 
sama blok `async` sampai dia selesai.

> Catatan: Beberapa _runtimes_ menyediakan macros sehingga Anda _bisa_ menulis 
> fungsi `main` yang _async_. Macros itu nulis ulang `async fn main() { ... }` 
> jadi `fn main` normal, yang melakukan persis hal yang sama kayak yang kita lakuin 
> secara manual di Listing 17-4: memanggil fungsi yang mengeksekusi sebuah _future_ 
> sampai selesai kayak yang dilakuin sama `trpl::run`.

Sekarang mari kita gabungin bagian-bagian ini dan lihat gimana kita bisa nulis 
kode konkuren.

### Menandingkan (Racing) Dua URL Kita Satu Sama Lain

Di Listing 17-5, kita memanggil `page_title` dengan dua URL berbeda yang dimasukkan 
dari _command line_ lalu menandingkan (_race_) mereka berdua.

<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Kita mulai dengan manggil `page_title` buat tiap URL yang dikasih sama *user*. Kita 
simpan _futures_ hasilnya sebagai `title_fut_1` dan `title_fut_2`. Inget, mereka 
ini belum ngelakuin apa-apa, karena _futures_ itu sifatnya _lazy_ dan kita belum 
nge-_await_ mereka. Terus kita ngoper _futures_ tersebut ke `trpl::race`, yang 
mengembalikan sebuah nilai buat ngindikasikan _future_ mana yang kelar duluan 
di antara yang dioper kepadanya.

> Catatan: Di balik layar, `race` dibangun di atas fungsi yang lebih umum, yaitu 
> `select`, yang bakal Anda temui lebih sering di kode Rust di dunia nyata. 
> Fungsi `select` bisa ngelakuin banyak hal yang fungsi `trpl::race` tidak bisa, 
> tapi dia juga punya kerumitan ekstra yang bisa kita lewatin dulu buat sekarang.

Dua-duanya bisa sah-sah aja (legitimately) "menang," jadi nggak masuk akal kalau 
kita memulangkan `Result`. Alih-alih begitu, `race` memulangkan sebuah tipe yang 
belum pernah kita lihat sebelumnya, yaitu `trpl::Either`. Tipe `Either` itu 
agak mirip sama `Result` dalam hal dia punya dua kasus. Bedanya sama `Result`, 
tidak ada konsep "sukses" atau "gagal" yang tertanam (baked) di dalam `Either`. 
Alih-alih begitu, dia memakai `Left` (kiri) dan `Right` (kanan) buat ngindikasikan 
"yang satu atau yang lainnya":

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Fungsi `race` mengembalikan `Left` yang berisi output dari argumen _future_ pertama 
kalau dia selesai duluan, atau `Right` yang berisi output argumen _future_ kedua 
kalau dia yang selesai duluan. Ini cocok dengan urutan munculnya argumen-argumen 
tersebut saat manggil fungsinya: argumen pertama ada di *kiri* argumen kedua.

Kita juga meng-update `page_title` buat mengembalikan URL yang sama dengan yang 
dimasukkan. Dengan begitu, kalau halaman yang kembali duluan tidak punya `<title>` 
yang bisa kita urai (resolve), kita masih bisa nyetak pesan yang bermakna. Dengan 
informasi yang udah tersedia itu, kita bungkus (wrap up) ini semua dengan ngubah 
output `println!` kita buat mengindikasikan baik URL mana yang kelar duluan, dan 
apa, kalau memang ada, `<title>` buat halaman web di URL tersebut.

Tadaa, Anda udah ngebikin _web scraper_ mini yang bisa jalan! Silakan pilih beberapa 
URL lalu jalanin alat _command line_ Anda. Anda mungkin mendapati kalau beberapa situs 
secara konsisten emang lebih kencang dibanding yang lain, sementara di kasus lain 
situs yang kencang itu berubah-ubah di tiap jalan (run). Yang lebih penting, Anda udah 
belajar dasar-dasar dari bekerja dengan _futures_, jadi sekarang kita bisa gali 
lebih dalem soal apa yang bisa kita lakuin dengan _async_.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
