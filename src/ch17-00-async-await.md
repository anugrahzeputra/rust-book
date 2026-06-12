# Dasar-dasar Pemrograman Asynchronous: Async, Await, Futures, dan Streams

Banyak operasi yang kita suruh komputer buat lakukan bisa memakan waktu agak lama 
buat selesai. Bakal menyenangkan banget kalau kita bisa melakukan hal lain selagi 
kita nungguin proses-proses yang panjang itu buat kelar. Komputer modern menawarkan 
dua teknik buat mengerjakan lebih dari satu operasi pada satu waktu: _parallelism_ 
(paralelisme) dan _concurrency_ (konkurensi). Tapi, begitu kita mulai nulis 
program yang ngelibatin operasi paralel atau konkuren, kita bakal cepat 
menemukan tantangan baru yang melekat pada _asynchronous programming_ (pemrograman 
asynchronous), di mana operasi mungkin tidak selesai berurutan seperti saat mereka 
dimulai. Bab ini dibangun di atas penggunaan _threads_ buat paralelisme dan 
konkurensi di Bab 16 dengan memperkenalkan pendekatan alternatif buat pemrograman 
_asynchronous_: _Futures_, _Streams_, sintaks `async` dan `await` di Rust yang 
mendukungnya, serta _tools_ buat mengelola dan mengoordinasi berbagai operasi 
_asynchronous_.

Mari kita pertimbangkan sebuah contoh. Katakanlah Anda lagi ngekspor video perayaan 
keluarga yang sudah Anda bikin, sebuah operasi yang bisa memakan waktu mulai 
dari beberapa menit sampai berjam-jam. Ekspor video bakal memakai sebanyak 
mungkin tenaga CPU dan GPU yang bisa dia dapatkan. Kalau Anda cuma punya satu 
_core_ (inti) CPU dan sistem operasi Anda tidak nge-_pause_ ekspor itu sampai ia 
selesai—yaitu, kalau ia mengeksekusi ekspornya secara _synchronous_ (sinkron)—Anda 
tidak bakal bisa ngelakuin hal lain di komputer Anda saat tugas itu berjalan. Itu 
bakal jadi pengalaman yang cukup bikin frustrasi. Untungnya, sistem operasi di 
komputer Anda bisa, dan emang ngelakuin itu, menginterupsi proses ekspor secara 
kasatmata (invisibly) cukup sering biar Anda bisa mengerjakan tugas lain di saat 
yang bersamaan.

Sekarang katakanlah Anda lagi men-download video yang di-_share_ sama orang lain, 
yang juga bisa memakan waktu lumayan lama tapi tidak terlalu menyita banyak waktu 
CPU. Di kasus ini, CPU harus menunggu data dari jaringan (network) buat tiba. 
Meskipun Anda bisa mulai membaca data tersebut begitu dia mulai berdatangan, itu 
bisa memakan waktu beberapa saat sampai semuanya muncul. Bahkan setelah semua 
datanya ada, kalau videonya gede banget, itu bisa memakan waktu setidaknya satu 
atau dua detik buat nge-_load_ semuanya. Itu mungkin tidak kedengeran terlalu 
lama, tapi itu adalah waktu yang sangat lama buat prosesor modern, yang mana 
bisa melakukan miliaran operasi setiap detik. Sekali lagi, sistem operasi Anda 
bakal menginterupsi program Anda secara kasatmata buat membiarkan CPU mengerjakan 
tugas lain sembari nunggu pemanggilan jaringan selesai.

Ekspor video adalah contoh dari operasi _CPU-bound_ atau _compute-bound_ 
(bergantung pada prosesor). Dia dibatasi oleh potensi kecepatan pemrosesan data 
komputer di dalam CPU atau GPU, dan seberapa besar dari kecepatan itu yang bisa 
didedikasikan buat operasi tersebut. Download video adalah contoh dari operasi 
_IO-bound_ (bergantung pada input-output), karena dia dibatasi oleh kecepatan 
_input dan output_ komputer; dia cuma bisa berjalan secepat data yang bisa dikirim 
lewat jaringan.

Di dua contoh ini, interupsi kasatmata dari sistem operasi ngasih suatu bentuk 
konkurensi. Namun, konkurensi itu cuma terjadi di tingkat keseluruhan program: 
sistem operasi menginterupsi satu program buat membiarkan program lain 
mengerjakan tugasnya. Di banyak kasus, karena kita paham program kita di tingkat 
yang jauh lebih spesifik (granular) ketimbang sistem operasi, kita bisa menemukan 
peluang-peluang buat konkurensi yang tidak bisa dilihat sama sistem operasi.

Misalnya, kalau kita lagi bikin _tool_ buat mengelola download file, kita seharusnya 
bisa menulis program kita sedemikian rupa sehingga pas mulai nge-download satu 
file UI-nya tidak _freeze_ (macet), dan _user_ seharusnya bisa memulai banyak 
download di saat yang bersamaan. Namun, banyak API sistem operasi buat berinteraksi 
dengan jaringan itu sifatnya _blocking_ (memblokir); yakni, mereka memblokir 
progress program sampai data yang mereka proses itu benar-benar siap seutuhnya.

> Catatan: Ini adalah cara kerja _kebanyakan_ pemanggilan fungsi, kalau dipikir-pikir. 
> Namun, istilah _blocking_ biasanya dikhususkan buat pemanggilan fungsi yang 
> berinteraksi dengan file, jaringan, atau *resources* lain di komputer, karena 
> itu adalah kasus-kasus di mana suatu program individu bakal dapat keuntungan 
> kalau operasi tersebut _non_-blocking.

Kita bisa menghindari memblokir _main thread_ kita dengan membikin sebuah _spawned 
thread_ yang didedikasikan buat men-download tiap file. Namun, _overhead_ (beban) 
dari _threads_ itu pada akhirnya bakal jadi masalah. Bakal lebih oke kalau pemanggilan 
fungsinya dari awal emang tidak memblokir. Bakal lebih baik lagi kalau kita bisa nulis 
kodenya pake gaya langsung yang sama dengan yang kita pakai di kode _blocking_, 
kayak gini contohnya:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

Nah, itulah persisnya yang dikasih sama abstraksi _async_ (singkatan dari _asynchronous_) 
di Rust buat kita. Di bab ini, Anda bakal belajar semua hal tentang _async_ 
selagi kita membahas topik-topik berikut:

- Gimana cara memakai sintaks `async` dan `await` di Rust
- Gimana cara memakai model _async_ buat nyelesein beberapa tantangan yang sama 
  seperti yang udah kita lihat di Bab 16
- Gimana _multithreading_ dan _async_ menyediakan solusi yang saling melengkapi, 
  yang bisa Anda gabungkan di banyak kasus

Tapi, sebelum kita lihat gimana cara kerja _async_ di praktiknya, kita perlu sedikit 
belok sebentar buat ngebahas perbedaan antara paralelisme dan konkurensi.

### Paralelisme dan Konkurensi

Sejauh ini kita memperlakukan paralelisme dan konkurensi seolah-olah maknanya bisa 
dituker-tukar (interchangeable). Sekarang kita harus bisa membedakannya dengan 
lebih presisi, karena perbedaannya bakal kerasa pas kita udah mulai ngerjain kode.

Bayangin aja cara-cara beda yang bisa dipakai sebuah tim buat ngebagi kerjaan 
di suatu proyek _software_. Anda bisa menugaskan satu orang beberapa tugas, 
menugaskan tiap anggota satu tugas, atau memakai campuran dari kedua pendekatan 
tersebut.

Saat satu orang individu mengerjakan beberapa tugas yang berbeda sebelum satupun 
dari tugas-tugas itu selesai, ini dinamakan _konkurensi_. Mungkin Anda punya dua 
project berbeda yang lagi jalan di komputer Anda, dan pas Anda bosan atau mentok 
di satu project, Anda beralih ke project yang satu lagi. Anda cuma satu orang, jadi 
Anda tidak bisa bikin progress di kedua tugas pada waktu yang sama persis, tapi 
Anda bisa _multi-task_, bikin progress pada satu tugas secara bergantian dengan 
beralih di antara mereka (lihat Gambar 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="Sebuah diagram dengan kotak-kotak berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Ada panah yang menunjuk dari A1 ke B1, B1 ke A2, A2 ke B2, B2 ke A3, A3 ke A4, dan A4 ke B3. Panah di antara subtugas-subtugas ini menyilang di antara kotak untuk Tugas A dan Tugas B." />

<figcaption>Gambar 17-1: Sebuah alur kerja konkuren, beralih di antara Tugas A dan Tugas B</figcaption>

</figure>

Saat sebuah tim membagi sekelompok tugas dengan nyuruh setiap anggota mengambil satu 
tugas lalu mengerjakannya sendirian, ini dinamakan _paralelisme_. Setiap orang di tim 
tersebut bisa membikin progress pada waktu yang sama persis (lihat Gambar 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="Sebuah diagram dengan kotak-kotak berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Ada panah yang menunjuk dari A1 ke A2, A2 ke A3, A3 ke A4, B1 ke B2, dan B2 ke B3. Tidak ada panah yang menyilang antara kotak untuk Tugas A dan Tugas B." />

<figcaption>Gambar 17-2: Sebuah alur kerja paralel, di mana pekerjaan terjadi pada Tugas A dan Tugas B secara independen</figcaption>

</figure>

Di kedua alur kerja (workflows) ini, Anda mungkin harus mengoordinasikan antara 
tugas-tugas yang berbeda. Mungkin Anda _pikir_ tugas yang diberikan ke satu orang 
itu benar-benar independen dari pekerjaan orang lain, tapi ternyata dia butuh orang 
lain di tim itu buat nyelesein tugas mereka lebih dulu. Beberapa pekerjaan bisa 
dilakukan secara paralel, tapi sebagian darinya itu sebenarnya _serial_: dia 
cuma bisa terjadi dalam satu urutan, satu tugas setelah tugas lainnya, kayak di 
Gambar 17-3.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="Sebuah diagram dengan kotak-kotak berlabel Tugas A dan Tugas B, dengan belah ketupat di dalamnya yang melambangkan subtugas. Ada panah yang menunjuk dari A1 ke A2, A2 ke sepasang garis vertikal tebal seperti simbol “pause”, dari simbol itu ke A3, B1 ke B2, B2 ke B3, yang berada di bawah simbol itu, B3 ke A3, dan B3 ke B4." />

<figcaption>Gambar 17-3: Sebuah alur kerja yang sebagiannya paralel, di mana pekerjaan terjadi pada Tugas A dan Tugas B secara independen sampai Tugas A3 terhambat (blocked) menunggu hasil dari Tugas B3.</figcaption>

</figure>

Sama halnya, Anda mungkin sadar kalau salah satu tugas Anda sendiri itu bergantung 
sama tugas Anda yang lainnya. Sekarang pekerjaan konkuren Anda juga udah jadi serial.

Paralelisme dan konkurensi juga bisa bersinggungan (intersect) satu sama lain. 
Kalau Anda tahu kalau rekan kerja Anda lagi mentok sampai Anda nyelesein salah 
satu tugas Anda, Anda mungkin bakal memusatkan seluruh usaha Anda pada tugas itu 
buat "membuka jalan" (unblock) rekan kerja Anda tersebut. Anda dan rekan kerja Anda 
tidak lagi bisa bekerja secara paralel, dan Anda juga tidak lagi bisa bekerja 
secara konkuren di tugas Anda masing-masing.

Dinamika dasar yang sama mulai berlaku pada perangkat lunak dan perangkat keras 
(_software_ dan _hardware_). Di mesin yang punya satu _core_ CPU, CPU cuma bisa 
melakukan satu operasi pada satu waktu, tapi dia tetap bisa bekerja secara konkuren. 
Memakai alat seperti _threads_, proses, dan _async_, komputer bisa mem-_pause_ satu 
aktivitas lalu beralih ke aktivitas lain sebelum akhirnya berputar kembali ke aktivitas 
pertama tadi. Di mesin dengan banyak _core_ CPU, dia juga bisa mengerjakan tugas 
secara paralel. Satu _core_ bisa ngerjain satu tugas sementara _core_ lain ngerjain 
tugas yang sama sekali tidak ada hubungannya, dan operasi-operasi itu benar-benar 
terjadi pada waktu yang bersamaan.

Saat berurusan dengan _async_ di Rust, kita bakal selalu berurusan dengan konkurensi. 
Tergantung pada perangkat keras, sistem operasi, dan *async runtime* yang lagi kita 
pakai (*async runtimes* bakal dibahas sebentar lagi), konkurensi itu mungkin juga 
memakai paralelisme di balik layar.

Sekarang, mari kita selami gimana cara kerja _async programming_ di Rust sebenarnya.
