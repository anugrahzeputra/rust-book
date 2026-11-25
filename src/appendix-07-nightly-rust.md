## Lampiran G - Bagaimana Rust Dibuat dan “Nightly Rust”

Lampiran ini membahas bagaimana Rust dibuat dan bagaimana pengaruhnya bagi kita 
sebagai pengembang Rust.

### Kestabilan Tanpa Kemandekan

Sebagai sebuah bahasa, Rust sangat peduli pada kestabilan kode kita. Kami ingin 
Rust menjadi fondasi kokoh yang bisa kita jadikan pijakan untuk membangun 
program, dan jika segalanya terus berubah, hal itu akan mustahil dilakukan. Di 
sisi lain, jika kami tidak bisa bereksperimen dengan fitur baru, kami mungkin 
tidak akan menemukan kekurangan penting sebelum fitur itu dirilis dan tidak 
bisa diubah lagi.

Solusi kami terhadap masalah ini adalah apa yang kami sebut “kestabilan tanpa 
kemandekan” (_stability without stagnation_), dan prinsip panduan kami adalah: 
kita tidak perlu takut melakukan upgrade ke versi baru Rust yang stabil. Setiap 
upgrade seharusnya mudah dilakukan, tetapi juga membawa fitur-fitur baru, lebih 
sedikit bug, dan waktu kompilasi yang lebih cepat.

### Tut, Tut! Saluran Rilis dan Model Kereta

Pengembangan Rust berjalan dengan _jadwal kereta_ (_train schedule_). Artinya, 
semua pengembangan dilakukan di branch `main` pada repositori Rust. Rilis 
mengikuti model kereta rilis perangkat lunak, seperti yang digunakan oleh Cisco 
IOS dan proyek perangkat lunak lainnya. Ada tiga _saluran rilis_ (_release 
channels_) untuk Rust:

- Nightly
- Beta
- Stable

Sebagian besar pengembang Rust terutama memakai saluran stable, tetapi mereka 
yang ingin mencoba fitur-fitur baru yang masih eksperimental bisa memakai 
nightly atau beta.

Berikut contoh bagaimana proses pengembangan dan rilisnya bekerja: misalkan 
tim Rust sedang mengerjakan rilis Rust 1.5. Rilis itu terjadi pada Desember 
2015, tetapi bisa memberi kita nomor versi yang realistis. Sebuah fitur baru 
ditambahkan ke Rust: sebuah commit baru mendarat di branch `main`. Setiap 
malam, versi nightly baru dari Rust dibuat. Setiap hari adalah hari rilis, dan 
rilis ini dibuat secara otomatis oleh infrastruktur rilis kami. Jadi seiring 
waktu berjalan, rilis kita terlihat seperti ini, sekali setiap malam:

```text
nightly: * - - * - - *
```

Setiap enam minggu, tiba waktunya menyiapkan rilis baru! Branch `beta` pada 
repositori Rust bercabang dari branch `main` yang dipakai oleh nightly. 
Sekarang, ada dua rilis:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Kebanyakan pengguna Rust tidak memakai rilis beta secara aktif, tetapi menguji 
beta di sistem CI mereka untuk membantu Rust menemukan kemungkinan regresi. 
Sementara itu, tetap ada rilis nightly setiap malam:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Katakanlah ditemukan sebuah regresi. Untungnya kita punya waktu untuk menguji 
rilis beta sebelum regresi tersebut lolos ke rilis stable! Perbaikannya 
diterapkan ke branch `main` sehingga nightly diperbaiki, lalu perbaikannya 
di-backport ke branch `beta`, dan rilis beta baru pun dibuat:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Enam minggu setelah beta pertama dibuat, tiba waktunya untuk rilis stable! 
Branch `stable` dibuat dari branch `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Hore! Rust 1.5 selesai! Namun, kita lupa satu hal: karena enam minggu sudah 
berlalu, kita juga membutuhkan beta baru untuk versi Rust _berikutnya_, 1.6. 
Jadi setelah `stable` bercabang dari `beta`, versi `beta` berikutnya bercabang 
lagi dari `nightly`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Ini disebut “model kereta” karena setiap enam minggu, sebuah rilis “berangkat 
dari stasiun”, tetapi tetap harus menempuh perjalanan melalui saluran beta 
sebelum tiba sebagai rilis stable.

Rust dirilis setiap enam minggu, tepat waktu seperti jam. Jika kita tahu 
tanggal satu rilis Rust, kita bisa tahu tanggal rilis berikutnya: enam minggu 
kemudian. Sisi baik dari jadwal rilis setiap enam minggu adalah kereta 
berikutnya akan segera tiba. Jika suatu fitur kebetulan tertinggal dari satu 
rilis tertentu, tidak perlu khawatir: rilis berikutnya akan datang dalam waktu 
singkat! Ini membantu mengurangi tekanan untuk menyelipkan fitur yang mungkin 
belum matang menjelang tenggat rilis.

Berkat proses ini, kita selalu bisa mencoba build Rust berikutnya dan 
memastikan sendiri bahwa upgrade itu mudah dilakukan: jika rilis beta tidak 
bekerja sesuai harapan, kita bisa melaporkannya ke tim dan memperbaikinya 
sebelum rilis stable berikutnya terjadi! Kerusakan di rilis beta relatif jarang 
terjadi, tetapi `rustc` tetaplah perangkat lunak, dan bug memang ada.

### Waktu Pemeliharaan

Proyek Rust mendukung versi stable terbaru. Ketika versi stable baru dirilis, 
versi lama mencapai akhir masa pakainya (EOL). Artinya, setiap versi didukung 
selama enam minggu.

<a id="unstable-features"></a>

### Fitur Tidak Stabil

Ada satu hal lagi dalam model rilis ini: fitur tidak stabil (_unstable 
features_). Rust menggunakan teknik yang disebut “feature flags” untuk 
menentukan fitur apa saja yang diaktifkan pada suatu rilis tertentu. Jika fitur 
baru sedang dikembangkan secara aktif, fitur itu mendarat di branch `main`, 
dan karenanya di nightly, tetapi di balik sebuah _feature flag_. Jika kita, 
sebagai pengguna, ingin mencoba fitur yang masih dikerjakan ini, kita bisa 
melakukannya, tetapi kita harus memakai rilis Rust nightly dan menambahkan 
anotasi flag yang sesuai di kode sumber kita untuk mengaktifkannya.

Jika kita memakai rilis Rust beta atau stable, kita tidak bisa memakai feature 
flag apa pun. Inilah kunci yang memungkinkan kami mendapatkan pengalaman 
penggunaan praktis atas fitur baru sebelum menyatakannya stabil selamanya. 
Mereka yang ingin mencoba fitur paling mutakhir bisa melakukannya, dan mereka 
yang menginginkan pengalaman yang kokoh bisa tetap di stable dan yakin bahwa 
kode mereka tidak akan rusak. Kestabilan tanpa kemandekan.

Buku ini hanya berisi informasi tentang fitur stable, karena fitur yang sedang 
dikembangkan masih terus berubah, dan hampir pasti akan berbeda antara saat 
buku ini ditulis dan saat fitur tersebut diaktifkan di build stable. Kita bisa 
menemukan dokumentasi untuk fitur khusus nightly secara online.

### Rustup dan Peran Rust Nightly

Rustup memudahkan kita berpindah antar saluran rilis Rust yang berbeda, baik 
secara global maupun per proyek. Secara default, kita sudah memasang Rust 
stable. Untuk memasang nightly, misalnya:

```console
$ rustup toolchain install nightly
```

Kita juga bisa melihat semua _toolchain_ (rilis Rust beserta komponen 
terkaitnya) yang sudah terpasang menggunakan `rustup`. Berikut contohnya pada 
komputer Windows milik salah satu penulis buku ini:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Seperti yang bisa kita lihat, toolchain stable adalah default-nya. Kebanyakan 
pengguna Rust memakai stable hampir sepanjang waktu. Kita mungkin ingin tetap 
memakai stable secara umum, tetapi memakai nightly pada proyek tertentu karena 
kita membutuhkan fitur yang masih sangat baru. Untuk melakukannya, kita bisa 
memakai `rustup override` di direktori proyek tersebut untuk mengatur toolchain 
nightly sebagai toolchain yang harus dipakai `rustup` saat kita berada di 
direktori itu:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Sekarang, setiap kali kita memanggil `rustc` atau `cargo` di dalam 
_~/projects/needs-nightly_, `rustup` akan memastikan kita memakai Rust nightly, 
bukan default Rust stable kita. Ini sangat berguna kalau kita punya banyak 
proyek Rust!

### Proses RFC dan Tim

Lalu bagaimana cara kita mengetahui fitur-fitur baru ini? Model pengembangan 
Rust mengikuti _proses Request For Comments (RFC)_. Jika kita menginginkan 
peningkatan pada Rust, kita bisa menulis proposal yang disebut RFC.

Siapa pun bisa menulis RFC untuk meningkatkan Rust, dan proposal tersebut akan 
ditinjau serta didiskusikan oleh tim Rust, yang terdiri dari banyak subtim 
berdasarkan topik. Daftar lengkap tim tersedia [di situs web Rust](https://www.rust-lang.org/governance), yang 
mencakup tim untuk setiap bidang proyek: desain bahasa, implementasi compiler, 
infrastruktur, dokumentasi, dan lainnya. Tim yang relevan membaca proposal dan 
komentarnya, menulis komentar mereka sendiri, dan pada akhirnya tercapai 
konsensus untuk menerima atau menolak fitur tersebut.

Jika fitur itu diterima, sebuah issue dibuka di repositori Rust, dan seseorang 
bisa mengimplementasikannya. Orang yang mengimplementasikannya bisa saja bukan 
orang yang pertama kali mengusulkan fitur tersebut! Ketika implementasinya 
sudah siap, ia mendarat di branch `main` di balik sebuah feature gate, seperti 
yang kita bahas di bagian [“Fitur Tidak Stabil”](#unstable-features)<!-- ignore -->.

Setelah beberapa waktu, setelah pengembang Rust yang memakai rilis nightly 
sempat mencoba fitur baru tersebut, anggota tim akan mendiskusikan fitur itu, 
bagaimana hasilnya di nightly, dan memutuskan apakah fitur tersebut layak masuk 
ke Rust stable atau tidak. Jika keputusannya adalah melanjutkan, feature gate 
akan dihapus, dan fitur tersebut kini dianggap stabil! Fitur itu pun naik 
kereta menuju rilis stable baru dari Rust.
