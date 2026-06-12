## Menyusun Semuanya: Futures, Tasks, dan Threads

Kayak yang kita lihat di [Bab 16][ch16], _threads_ nyediain satu pendekatan 
buat konkurensi. Kita udah melihat pendekatan lainnya di bab ini: memakai 
_async_ dengan _futures_ dan _streams_. Kalau Anda mikir kapan harus milih 
salah satu metode ketimbang yang lain, jawabannya adalah: ya tergantung! Dan 
di banyak kasus, pilihannya bukanlah _threads_ **atau** _async_, melainkan 
_threads_ **dan** _async_.

Banyak sistem operasi udah nyediain model konkurensi berbasis _thread_ selama 
puluhan tahun sekarang, dan banyak bahasa pemrograman ngedukung mereka sebagai 
hasilnya. Namun, model-model ini bukan tanpa kekurangannya (*tradeoffs*). Di 
banyak sistem operasi, mereka memakai lumayan banyak memori buat tiap _thread_, 
dan mereka datang sama sedikit _overhead_ (beban) pas buat _startup_ (mulai) 
dan _shutdown_ (matiin). _Threads_ juga cuma jadi pilihan pas sistem operasi dan 
_hardware_ Anda ngedukung mereka. Tidak kayak komputer desktop dan _mobile_ pada 
umumnya, beberapa sistem _embedded_ (tertanam) bahkan tidak punya OS sama 
sekali, jadi mereka juga tidak punya _threads_.

Model _async_ nyediain sekelompok _tradeoffs_ yang beda—dan pada akhirnya saling 
melengkapi (complementary). Di model _async_, operasi konkuren tidak perlu punya 
_threads_ mereka sendiri. Alih-alih begitu, mereka bisa jalan di _tasks_ (tugas), 
kayak pas kita memakai `trpl::spawn_task` buat memulai kerjaan dari sebuah fungsi 
sinkron di bagian _streams_. Sebuah _task_ itu mirip sama sebuah _thread_, tapi 
alih-alih dikelola sama sistem operasi, dia dikelola sama kode di level _library_: 
yaitu _runtime_-nya.

Di bagian sebelumnya, kita melihat kalau kita bisa ngebangun sebuah _stream_ 
dengan memakai _async channel_ dan membikin (_spawning_) sebuah _async task_ 
yang bisa kita panggil dari kode sinkron. Kita bisa ngelakuin hal yang persis sama 
pakai sebuah _thread_. Di Listing 17-40, kita memakai `trpl::spawn_task` dan 
`trpl::sleep`. Di Listing 17-41, kita ngegantiin mereka sama API `thread::spawn` 
dan `thread::sleep` dari _standard library_ di dalam fungsi `get_intervals`.

<Listing number="17-41" caption="Memakai API `std::thread` ketimbang API _async_ `trpl` buat fungsi `get_intervals`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-41/src/main.rs:threads}}
```

</Listing>

Kalau Anda ngejalanin kode ini, outputnya identik sama yang di Listing 17-40. Dan 
perhatiin betapa sedikitnya perubahan di sini dari perspektif kode pemanggilnya 
(calling code). Selain itu, biarpun salah satu dari fungsi kita membikin sebuah 
_async task_ di _runtime_ dan yang satunya lagi membikin sebuah _thread_ OS, 
_streams_ yang dihasilkannya tidak terpengaruh sama sekali oleh perbedaan-perbedaan 
itu.

Terlepas dari kemiripan-kemiripan ini, kedua pendekatan ini berperilaku sangat 
beda, meskipun kita mungkin bakal kesusahan buat ngukurnya di contoh yang sangat 
sederhana ini. Kita bisa membikin (_spawn_) jutaan _async tasks_ di komputer 
pribadi modern mana pun. Kalau kita nyoba ngelakuin itu pakai _threads_, kita 
bener-bener bakal kehabisan memori!

Namun, ada alasannya kenapa API-API ini sangat mirip. _Threads_ bertindak sebagai 
batas (boundary) buat sekumpulan operasi sinkron; konkurensi itu mungkin terjadi 
*di antara* (between) _threads_. _Tasks_ bertindak sebagai batas buat sekumpulan 
operasi *asynchronous* (asinkron); konkurensi itu mungkin terjadi baik *di antara* 
maupun *di dalam* (within) _tasks_, karena sebuah _task_ bisa beralih (switch) di 
antara _futures_ di dalam isi kodenya (body). Terakhir, _futures_ adalah unit 
konkurensi paling mendetail (granular) di Rust, dan setiap _future_ bisa 
merepresentasikan sebuah pohon (tree) yang berisi _futures_ lainnya. _Runtime_—
secara spesifik, bagian _executor_-nya—mengelola _tasks_, dan _tasks_ mengelola 
_futures_. Dalam hal ini, _tasks_ itu mirip sama _threads_ ringan (lightweight) 
yang dikelola oleh _runtime_ dengan berbagai kapabilitas tambahan yang datang dari 
fakta kalau dia dikelola oleh sebuah _runtime_ bukannya oleh sistem operasi.

Ini tidak berarti kalau _async tasks_ itu selalu lebih baik dari _threads_ (atau 
sebaliknya). Konkurensi dengan _threads_ dalam beberapa hal adalah model pemrograman 
yang lebih simpel ketimbang konkurensi dengan `async`. Itu bisa jadi sebuah kelebihan 
atau sebuah kelemahan. _Threads_ itu sifatnya agak seperti "jalankan dan lupakan" 
(fire and forget); mereka tidak punya padanan (equivalent) bawaan (native) buat 
sebuah _future_, jadi mereka cuma jalan aja sampai selesai tanpa diinterupsi 
kecuali oleh sistem operasi itu sendiri. Yakni, mereka tidak punya dukungan bawaan 
buat *intratask concurrency* (konkurensi di dalam task) kayak yang dipunyai oleh 
_futures_. _Threads_ di Rust juga tidak punya mekanisme buat pembatalan 
(cancellation)—sebuah topik yang belum kita bahas secara eksplisit di bab ini tapi 
udah diimplikasikan oleh fakta bahwa kapan pun kita mengakhiri (ended) sebuah 
_future_, keadaannya (state) bakal dibersihkan (cleaned up) dengan bener.

Keterbatasan-keterbatasan ini juga membikin _threads_ lebih susah buat digabungkan 
(composed) ketimbang _futures_. Jauh lebih susah, misalnya, buat memakai _threads_ 
buat ngebangun fungsi pembantu (helpers) kayak method `timeout` dan `throttle` 
yang kita bangun di awal bab ini. Fakta bahwa _futures_ adalah struktur data yang 
lebih kaya (richer) berarti mereka bisa digabung-gabungin dengan lebih natural 
(alami), kayak yang udah kita lihat.

Maka, _Tasks_ ngasih kita kontrol *tambahan* atas _futures_, yang memungkinkan 
kita buat milih di mana dan gimana cara ngelompokkin mereka. Dan ternyata _threads_ 
sama _tasks_ itu sering banget bekerja dengan sangat baik kalau digabungin barengan, 
karena _tasks_ itu (setidaknya di beberapa _runtimes_) bisa dipindah-pindahin 
(moved around) di antara _threads_. Faktanya, di balik layar, _runtime_ yang dari 
tadi kita pakai—termasuk fungsi `spawn_blocking` dan `spawn_task`—secara default 
itu _multithreaded_! Banyak _runtimes_ memakai pendekatan yang disebut _work stealing_ 
(mencuri pekerjaan) buat secara transparan mindah-mindahin _tasks_ di antara 
_threads_, berdasarkan seberapa banyak _threads_ itu lagi dipakai saat ini, buat 
ningkatin performa keseluruhan (overall) dari sistemnya. Pendekatan itu sebenarnya 
butuh _threads_ *dan* _tasks_, dan makanya butuh _futures_.

Pas lagi mikirin metode mana yang harus dipakai kapan, pertimbangkan aturan praktis 
(rules of thumb) berikut ini:

- Kalau kerjaannya _sangat bisa diparalelkan_ (very parallelizable), kayak memproses 
  setumpuk data di mana tiap bagian bisa diproses secara terpisah, _threads_ adalah 
  pilihan yang lebih oke.
- Kalau kerjaannya _sangat konkuren_ (very concurrent), kayak nangani pesan-pesan 
  dari setumpuk sumber yang berbeda-beda yang mungkin berdatangan di berbagai 
  interval waktu yang berbeda atau dengan kecepatan yang berbeda, _async_ adalah 
  pilihan yang lebih oke.

Dan kalau Anda butuh paralelisme dan konkurensi dua-duanya, Anda tidak perlu harus 
milih antara _threads_ dan _async_. Anda bisa memakai mereka barengan dengan bebas, 
ngebiarin masing-masing memainkan bagian yang paling dia kuasai. Misalnya, Listing 
17-42 nunjukin contoh yang lumayan umum dari campuran kayak gini di kode Rust dunia 
nyata.

<Listing number="17-42" caption="Mengirim pesan-pesan dengan kode yang _blocking_ di dalam sebuah _thread_ dan nge-_await_ pesan-pesannya di dalam sebuah blok _async_" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-42/src/main.rs:all}}
```

</Listing>

Kita mulai dengan membikin sebuah _async channel_, terus ngebikin (spawn) sebuah 
_thread_ yang ngambil kepemilikan dari sisi pengirim _channel_-nya. Di dalam 
_thread_ tersebut, kita ngirim angka 1 sampai 10, sambil nge-_sleep_ selama satu 
detik di sela-selanya. Terakhir, kita ngejalanin sebuah _future_ yang dibikin 
dengan sebuah blok _async_ yang dimasukkan ke `trpl::run` sama persis kayak yang 
udah kita lakuin di sepanjang bab ini. Di _future_ itu, kita nge-_await_ 
pesan-pesan tersebut, persis kayak di contoh _message-passing_ lain yang udah kita 
lihat.

Buat balik lagi ke skenario yang kita pakai di awal bab ini, bayangin Anda lagi 
menjalankan serangkaian tugas *encoding* video memakai sebuah _thread_ yang khusus 
didedikasikan buat itu (dedicated thread) karena *encoding* video itu sifatnya 
_compute-bound_ (ngabisin CPU), tapi Anda ngasih notifikasi ke UI kalau 
operasi-operasi itu udah kelar memakai sebuah _async channel_. Ada banyak banget 
contoh dari jenis-jenis kombinasi kayak gini di kasus penggunaan (use cases) 
dunia nyata.

## Ringkasan

Ini bukanlah terakhir kalinya Anda bakal melihat konkurensi di buku ini. Project di 
[Bab 21][ch21] bakal nerapin konsep-konsep ini di situasi yang lebih realistis 
ketimbang contoh-contoh simpel yang dibahas di sini dan bakal membandingkan 
pemecahan masalah memakai _threading_ lawan _tasks_ secara lebih langsung.

Tidak peduli mana dari pendekatan ini yang Anda pilih, Rust ngasih Anda alat-alat 
yang Anda butuhin buat nulis kode konkuren yang aman dan cepat—entah itu buat 
_web server_ yang _high-throughput_ (kemampuan transmisi besar) atau sebuah sistem 
operasi _embedded_.

Berikutnya, kita bakal ngomongin soal cara-cara idiomatik buat memodelkan masalah 
dan menstrukturkan solusi seiring program Rust Anda jadi makin gede. Selain itu, 
kita bakal membahas gimana idiom-idiom Rust berkaitan dengan idiom yang mungkin 
udah Anda ketahui dari pemrograman berorientasi objek (object-oriented programming).

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]: ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
