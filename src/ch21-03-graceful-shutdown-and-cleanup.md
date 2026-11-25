## Graceful Shutdown (Mati Secara Anggun) dan Cleanup (Bersih-bersih)

Kode di Listing 21-20 itu benar-benar udah merespons ke _requests_ secara asinkron (asynchronously) melalui 
penggunaan _thread pool_, sesuai sama apa yang kita rencanakan (intended). Tapi kita ngedapetin 
beberapa pesan peringatan (warnings) soal *fields* `workers`, `id`, dan `thread` 
yang nampaknya tidak kita pakai secara langsung, yang mana ngingetin (reminds) kita 
kalau kita ini belum ngelakuin acara bersih-bersih (cleaning up) apa pun. Pas kita 
memakai metode pencet tombol <kbd>ctrl</kbd>-<kbd>C</kbd> yang agak bar-bar (less elegant) 
buat ngehentiin (halt) eksekusi si _main thread_, semua _threads_ lain di dalamnya juga bakal 
dihentiin seketika itu juga (immediately), biarpun mereka posisinya saat itu lagi di tengah-tengah 
nugas (in the middle of) ngelayanin sebuah _request_.

Maka dari itu selanjutnya, kita bakal mengimplementasikan trait `Drop` buat manggil `join` 
pada masing-masing _threads_ di dalam _pool_ tersebut supaya mereka bisa nyelesaiin 
dulu (finish) _requests_ yang lagi mereka kerjain sebelum akhirnya bener-bener ditutup (closing). 
Baru deh setelah itu kita bakal mengimplementasikan sebuah cara buat ngasih tahu para _threads_ 
kalau mereka seharusnya berhenti nerima _requests_ baru lalu mematikan diri (shut down). 
Buat ngelihat aksi langsung dari kode ini, kita bakal memodifikasi server kita supaya dia 
cuma nerima maksimal dua buah _requests_ aja sebelum akhirnya secara anggun (gracefully) 
mematikan semua proses di _thread pool_-nya.

Satu hal yang perlu diperhatiin (to notice) seiringan kita jalan: tidak ada satu pun dari 
proses ini yang bakal berdampak (affects) sama bagian-bagian kode yang tugasnya 
mengeksekusi (executing) _closures_, jadi segala hal yang ada di sini bakal tetep 
sama persis andaikata kita ini lagi memakai sebuah _thread pool_ buat sebuah _runtime_ _async_.

### Mengimplementasikan Trait `Drop` pada `ThreadPool`

Mari kita mulai dengan mengimplementasikan `Drop` pada _thread pool_ kita. 
Pas si _pool_ tersebut di-_drop_, seharusnya semua _threads_ kita ini di-`join` (digabungin) 
buat ngebuktiin dan mastiin (make sure) kalau mereka semua udah beres (finish) ngerjain 
kerjaan mereka. Listing 21-22 nunjukin percobaan perdana (first attempt) buat bikin 
implementasi `Drop` ini; tapi kode ini masih belum bakal jalan lho ya.

<Listing number="21-22" file-name="src/lib.rs" caption="Manggil join ke masing-masing thread pas si thread pool itu keluar dari scope">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Pertama-tama kita melakukan perulangan (loop) ngelewatin masing-masing elemen di `workers` 
kepunyaan si _thread pool_. Kita memakai referensi `&mut` di sini karena `self` itu sendiri 
kan emang sebuah referensi *mutable* (bisa diubah), dan kita juga perlu bisa ngubah (mutate) 
si variabel `worker`. Buat masing-masing `worker`, kita mencetak sebuah pesan yang ngasih tahu 
kalau *instance* spesifik `Worker` ini tuh lagi bersiap dimatikan (shutting down), dan baru 
setelahnya kita manggil `join` pada nilai _thread_ yang dimiliki *instance* `Worker` 
tersebut. Kalau pemanggilan ke `join` ini ternyata gagal (fails), kita memakai `unwrap` 
buat maksa Rust jadi _panic_ dan masuk ke fase mati secara tidak anggun (ungraceful shutdown).

Ini dia error yang bakal kita dapat pas kita mencoba buat men-compile kode ini:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Pesan error ini ngasih tahu kita kalau kita itu tidak bisa manggil `join` gara-gara kita cuma 
punya sebuah pinjaman _mutable_ (mutable borrow) dari masing-masing `worker` sedangkan si 
`join` ini mengambil hak kepemilikan (ownership) dari argumennya. Buat memecahkan 
(solve) isu ini, kita perlu buat mindahin (_move_) si nilai _thread_ tersebut keluar dari 
*instance* `Worker` yang tadinya nge-hak miliki (_owns_) si `thread` itu supaya si 
`join` bisa narik memakan (consume) _thread_ tersebut. Salah satu cara buat melakukan ini 
adalah dengan ngambil pendekatan (_approach_) yang sama kayak yang pernah kita lakuin di 
Listing 18-15. Kalau si `Worker` ini tadinya nampung `Option<thread::JoinHandle<()>>`, 
kita bisa aja manggil method `take` pada nilai `Option` tersebut buat mindahin (move) nilai 
aslinya dari varian `Some` lalu ninggalin sebuah varian `None` buat menempati posisinya 
(in its place). Dengan kata lain, sebuah `Worker` yang posisinya lagi jalan (running) bakal 
punya sebuah varian `Some` di dalam `thread`, dan pas kita pengen ngebersihin (clean up) 
sebuah `Worker`, kita bakal ngeganti si `Some` jadi `None` sehingga si `Worker` ini 
tidak bakal punya _thread_ buat dijalanin.

Namun masalahnya, momen di mana proses ini itu dibutuhkan (come up) _satu-satunya_ cuma 
terjadi saat kita lagi nge-_drop_ si `Worker` aja kan. Konsekuensinya sebagai ganti rugi 
(in exchange), kita jadinya mesti terus-terusan berurusan (deal with) sama tipe 
`Option<thread::JoinHandle<()>>` di mana pun kita lagi nyoba ngakses `worker.thread`. 
Penulisan bahasa Rust yang idiomatik itu emang cukup sering sekali (quite a bit) memakai 
`Option`, tapi pas kita mulai nemuin diri kita lagi asik ngebungkusin (wrapping) sesuatu yang mana 
padahal kita udah tahu (_know_) nilai itu tuh _selalu_ ada dan eksis (present) ke dalam 
sebuah `Option` cuma murni dijadiin sekadar jalan pintas buat ngakalin (workaround) hal macem 
gini, maka ini merupakan tanda ide yang bagus buat mulai nyari pendekatan alternatif 
(alternative approaches) supaya bisa ngebikin kode kita lebih rapi (cleaner) dan tidak 
gampang rawan error (less error-prone).

Di kasus yang ini, sebenernya ada jalan alternatif yang lebih oke (better alternative): 
yaitu method `Vec::drain`. Method ini menerima sebuah parameter jarak (range parameter) buat 
menentukan _items_ mana aja yang mau dihilangkan dari dalam _vector_ tersebut lalu dia bakal 
mengembalikan (returns) sebuah iterator dari item-item yang ditarik itu. Dengan memberikan 
sintaks *range* `..` kita bakal mengosongkan dan menghilangkan *semua* (*every*) nilai dari _vector_ tersebut.

Jadi kita perlu meng-_update_ implementasi `drop` untuk `ThreadPool` supaya jadi kayak gini:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Kode ini ngeberesin dan nyelesein pesan error _compiler_ tadi tanpa perlu adanya (require) 
perubahan lain lagi ke dalem kode kita. Perhatikan kalau, karena `drop` bisa aja dipanggil pas 
lagi masa-masa terjadi kepanikan (panicking), si `unwrap` ini juga bisa aja tiba-tiba ikutan _panic_ 
yang mana akhirnya nyebabin kepanikan tumpuk ganda (double panic), yang mana bakal langsung 
membikin programnya _crash_ (rusak) dan secara terpaksa ngakhirin semua proses pembersihan 
(cleanup) yang lagi berjalan. Buat ukuran program contoh kayak gini, tindakan _panic_ ini sah-sah aja (fine), 
tapi hal kayak gini ya tidak disarankan (isn't recommended) lho buat kode di level produksi.

### Mengirim Sinyal ke Threads Supaya Berhenti Mendengarkan Jobs (Tugas) Baru

Dengan serangkaian perubahan yang udah kita bikin barusan, kode kita sekarang udah bisa 
sukses di-compile tanpa ngeluarin peringatan apa-apa. Namun, kabar buruknya adalah 
kalau kode ini itu masih belum (doesn't function) berjalan pakai cara yang kita harepin. Kunci permasalahannya ada di 
logika yang ada di dalem _closures_ yang lagi dijalanin (run by) sama _threads_ kepunyaan 
*instances* si `Worker`: saat ini, kita emang udah manggil `join`, tapi hal itu tidak 
bakal bisa nutup mematikan (shut down) si _threads_ ini lho, soalnya mereka itu pada asyik 
berputar (_loop_) nyari kerjaan (_looking for jobs_) buat selama-lamanya (forever). Kalau 
kita coba buat nge-_drop_ si `ThreadPool` kita ini pakai implementasi `drop` kita yang 
sekarang, _main thread_-nya bakal malah ikutan mandek keblokir (block forever), diam selamanya nungguin _thread_ 
yang pertama itu kelar (finish) yang mana tidak bakal bisa kelar.

Buat mbetulin (fix) masalah ini, kita perlu ngebikin satu ubahan (change) di dalam implementasi `drop` buat `ThreadPool` 
dan abis itu juga butuh satu ubahan di dalem perulangan (loop) si `Worker`.

Pertama-tama kita bakal ngubah implementasi `drop` pada `ThreadPool` supaya dia secara eksplisit nge-_drop_ 
si `sender` (pengirim) ini _sebelum_ dia nungguin para _threads_-nya pada kelar jalan. Listing 21-23 
nunjukin rupa ubahan-ubahan ke `ThreadPool` tersebut buat secara eksplisit nge-_drop_ `sender`. Tidak kayak 
yang terjadi di `thread`, di sini kita emang *benar-benar butuh* (_do_ need) memakai 
`Option` biar kita bisa memindahkan (_move_) variabel `sender` keluar dari 
`ThreadPool` dengan memanggil `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Secara eksplisit nge-drop `sender` sebelum nge-join para threads `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Tindakan men-_drop_ (dropping) `sender` ini otomatis bakal nutup (closes) *channel*-nya, yang mana mengindikasikan kalau 
tidak bakal ada lagi pesan (messages) baru yang bakal dikirimin. Saat momen itu benar-benar terjadi, 
semua pemanggilan (_calls_) ke `recv` yang lagi dikerjain sama *instances* si `Worker` di dalam _infinite 
loop_ (perulangan tiada henti)-nya bakal otomatis nge-return sebuah pesan error. Di Listing 21-24, 
kita ngubah bagian perulangan `Worker` supaya dia mau keluar (exit the loop) dengan anggun (gracefully) 
di dalam skenario kayak gitu (in that case), yang berarti _threads_-nya ini akhirnya benar-benar 
bisa kelar (finish) pas implementasi `drop` milik `ThreadPool` manggil `join` 
ke mereka-mereka semua.

<Listing number="21-24" file-name="src/lib.rs" caption="Secara eksplisit mecah kabur (breaking out) dari loop pas fungsi `recv` nge-return sebuah error">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Buat bisa ngelihat sendiri aksi kode (code in action) ini benar-benar berjalan, mari kita modifikasi (modify) `main` 
supaya dia ini cuma nerima batas (accept only) dua *requests* aja sebelum akhirnya dia nutup 
(shutting down) si server-nya dengan anggun (_gracefully_), kayak yang ditunjukin di Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Menutup server setelah ngelayanin dua requests aja dengan cara ngelewatin/keluar dari loop-nya">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Kita pastinya sangat tidak mau ngarep kalau sebuah web server betulan di dunia 
nyata tiba-tiba mati nutup sendiri sesudah cuma ngelayanin dua buah _requests_ doang. 
Tapi kan kode ini itu cuma dibikin buat nunjukin kalau kemampuan matikan proses 
secara anggun (_graceful shutdown_) dan bersih-bersih (_cleanup_) emang udah bisa 
jalan dengan bener.

Method `take` didefinisiin di dalam trait `Iterator` dan ngebatasin iterasi 
paling banyak cuma dua item pertama aja. Terus, si `ThreadPool` bakal keluar 
dari _scope_ di akhir fungsi `main`, dan implementasi `drop` bakal jalan.

Silakan nyalain servernya pakai `cargo run` dan bikin tiga _requests_. _Request_ 
yang ketiga harusnya error, dan di terminal kita harusnya ngelihat output yang 
mirip kayak gini:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Kita mungkin bakal ngelihat urutan ID `Worker` dan pesan yang dicetak agak beda. 
Kita bisa lihat cara kerja kode ini dari pesan-pesannya: instance `Worker` 0 dan 
3 kebagian dua _requests_ pertama. Server berhenti nerima koneksi setelah 
koneksi kedua, dan implementasi `Drop` pada `ThreadPool` mulai jalan bahkan 
sebelum `Worker 3` mulai ngejalanin pekerjaannya. Nge-_drop_ `sender` langsung 
memutus semua instance `Worker` dan nyuruh mereka buat mati. Masing-masing 
instance `Worker` nyetak pesan pas koneksi mereka terputus, lalu _thread pool_ 
manggil `join` buat nungguin tiap _thread_ `Worker` selesai.

Perhatiin satu aspek menarik dari eksekusi kali ini: `ThreadPool` udah nge-_drop_ 
`sender`, dan sebelum ada `Worker` yang nerima error, kita udah nyoba nge-`join` 
`Worker 0`. `Worker 0` belum dapet error dari `recv`, jadi _main thread_ terblokir, 
nungguin `Worker 0` kelar. Sementara itu, `Worker 3` nerima satu pekerjaan lalu 
semua _threads_ nerima error. Pas `Worker 0` udah kelar, _main thread_ nungguin 
sisa instance `Worker` buat selesai. Di titik itu, semuanya udah keluar dari 
_loop_ mereka dan berhenti.

Selamat! Kita udah berhasil nyelesaiin project ini; kita udah punya web server 
dasar yang memakai _thread pool_ buat merespons secara asinkron. Kita bisa 
ngelakuin _graceful shutdown_ ke server, yang ngebersihin semua _threads_ di 
dalam _pool_.

Ini kode lengkapnya buat referensi:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Kita masih bisa ngembangin ini lebih jauh lagi! Kalau kita mau terus ningkatin 
project ini, berikut beberapa ide yang bisa dicoba:

- Tambahin lebih banyak dokumentasi ke `ThreadPool` dan method-method publiknya.
- Tambahin unit _tests_ buat menguji fungsionalitas library-nya.
- Ubah pemanggilan `unwrap` menjadi penanganan error (_error handling_) yang lebih tangguh.
- Pakai `ThreadPool` buat ngerjain tugas lain selain ngelayanin web _requests_.
- Cari crate _thread pool_ di [crates.io](https://crates.io/) dan implementasiin web server yang serupa memakai crate tersebut. Lalu, bandingin API dan ketangguhannya sama _thread pool_ yang kita buat sendiri.

## Ringkasan

Mantap! Kita udah nyampe di akhir buku ini! Kami berterima kasih karena kita 
udah ikut menelusuri perjalanan mengenal Rust ini bareng-bareng. Sekarang kita 
udah siap buat mengimplementasiin project Rust kita sendiri dan ikut ngebantu 
di project-project orang lain. Inget aja kalau ada komunitas sesama _Rustaceans_ 
yang ramah dan bakal senang hati ngebantu kita ngadepin tantangan apa pun yang 
kita temuin di sepanjang perjalanan Rust kita.
