## Lampiran G - Gimana Rust Dibuat dan “Nightly Rust”

Lampiran ini membahas tentang gimana caranya Rust itu dibuat dan gimana hal 
tersebut bakal berdampak pada Anda sebagai seorang *developer* Rust.

### Kestabilan Tanpa Kemandekan (Stability Without Stagnation)

Sebagai sebuah bahasa pemrograman, Rust sangat amat peduli terhadap kestabilan 
dari kode Anda. Kami ingin supaya Rust ini bisa menjadi pondasi sekokoh batu 
karang yang bisa Anda andalkan buat membangun program, dan andaikata segala sesuatunya 
terus-terusan berubah, hal tersebut pasti bakal mustahil. Pada saat yang sama, kalau 
kami tidak bisa bereksperimen dengan fitur-fitur baru, kami barangkali tidak bakal 
bisa menemukan kecacatan-kecacatan fatal sampai waktunya fitur tersebut dirilis, 
di mana kami sudah tidak punya lagi kuasa buat merubah hal-hal tersebut.

Solusi kami dalam menghadapi masalah ini adalah apa yang kami sebut dengan 
"kestabilan tanpa kemandekan" (_stability without stagnation_), dan prinsip 
panduan kami adalah: Anda tidak semestinya merasa was-was dan takut buat melakukan 
_upgrade_ ke versi stabil Rust yang lebih baru. Masing-masing proses _upgrade_ 
tersebut semestinya berjalan mulus tanpa masalah, tapi juga semestinya turut serta 
membawa fitur-fitur baru, lebih sedikit _bugs_, dan waktu kompilasi yang lebih cepat.

### Choo, Choo! Saluran Rilis dan Menumpang Kereta Perilisan

Proses pengembangan Rust itu beroperasi menggunakan _jadwal kereta_ (_train schedule_). 
Maksudnya adalah, semua proses pengembangan itu murni dikerjakan di dalam _branch_ 
(cabang) `master` dari repositori Rust. Proses perilisannya ini mengikuti 
model kereta rilis perangkat lunak (software release train model), yang mana 
sudah pernah dipakai oleh Cisco IOS dan proyek perangkat lunak lainnya. 
Ada tiga buah _release channels_ (saluran rilis) khusus untuk Rust:

- Nightly (Harian Malam)
- Beta (Uji Coba Beta)
- Stable (Baku Stabil)

Mayoritas dari orang-orang pemakai Rust (_Rust developers_) rata-rata kebanyakan lebih 
memilih memakai saluran _stable_, tapi buat mereka-mereka yang kebetulan ingin 
mencoba fitur-fitur eksperimental baru bisa memakai jalur _nightly_ ataupun _beta_.

Berikut ini adalah sebuah contoh ilustrasi soal gimana proses pengembangan dan 
perilisan tersebut beroperasi: mari kita asumsikan bahwa tim Rust ini posisinya lagi 
sibuk mengerjakan proyek rilisan Rust versi 1.5. Rilisan aslinya emang mendarat di 
bulan Desember 2015, tapi ini akan memberi kita angka versi yang realistis untuk 
contoh. Sebuah fitur baru telah disematkan ke dalam Rust: sebuah *commit* (kode) baru 
sudah mendarat masuk ke dalam _branch_ `master`. Setiap malam, sebuah versi _nightly_ 
baru dari Rust akan diproduksi. Setiap hari adalah hari rilis, dan perilisan ini 
diciptakan oleh infrastruktur rilis kita secara otomatis. Jadi seiring berjalannya waktu, 
rangkaian perilisan kita kelihatannya bakal seperti ini, sehari satu kali pada malam hari:

```text
nightly: * - - * - - *
```

Setiap enam minggu, tibalah waktunya buat menyiapkan sebuah rilisan baru! 
Cabang `beta` dari repositori Rust akan memisahkan diri (_branches off_) dari dahan 
`master` yang dipakai oleh `nightly`. Nah sekarang, bakal ada dua buah rilisan:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Mayoritas pengguna Rust tidak secara aktif memakai rilisan versi beta, melainkan 
mereka murni sekadar memakai beta buat menguji kode mereka di dalam sistem CI 
milik mereka demi ngebantu Rust mendeteksi kemungkinan adanya regresi (penurunan kinerja 
atau _error_). Sementara itu, perilisan `nightly` bakal tetap ada dan terus berjalan 
saban malam:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Katakanlah tiba-tiba sebuah regresi (masalah) ditemukan. Syukurlah kita punya waktu buat 
mengetes rilisan beta ini sebelum masalah regresi tersebut menyusup (_snuck into_) masuk 
ke dalam sebuah rilisan yang stabil (_stable release_)! Obat perbaikannya (_the fix_) akan 
diterapkan ke `master`, sedemikian rupa sehingga versi `nightly` berhasil dibenahi, dan 
kemudian perbaikan ini akan didorong mundur (_backported_) masuk ke cabang `beta`, 
dan sebuah rilisan `beta` baru bakal otomatis diproduksi:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Enam minggu setelah versi beta yang pertama diciptakan, inilah waktunya buat sebuah 
rilisan stabil (_stable release_)! Cabang `stable` lantas diproduksi dari 
cabang `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hore! Rust 1.5 sudah rampung! Namun, kita melupakan satu hal: gara-gara periode 
waktu enam minggu itu sudah berlalu, kita ini otomatis juga butuh sebuah edisi beta yang 
baru buat versi Rust yang _berikutnya_, yaitu 1.6. Makanya setelah cabang `stable` 
itu memisahkan diri dari cabang `beta`, versi dari `beta` yang selanjutnya bakal lantas 
kembali memisahkan dirinya dari `nightly` lagi:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Pola ini dijuluki “train model” (model kereta) karena setiap selang enam minggu sekali, 
sebuah rilisan “berangkat meninggalkan stasiun”, tapi dia wajib menempuh perjalanan 
melalui saluran beta terlebih dahulu sebelum akhirnya dia tiba dan mendarat 
sebagai sebuah rilisan yang stabil.

Rust merilis versi baru setiap enam minggu sekali, sama konstannya dengan putaran 
jarum jam. Kalau Anda tahu tanggal dari salah satu rilisan Rust, Anda pasti bisa menebak 
tanggal peluncuran versi berikutnya: sudah pasti enam minggu kemudian. Salah satu aspek 
menarik dari penjadwalan perilisan setiap enam mingguan ini adalah kenyataan bahwa 
kereta selanjutnya dijamin akan tiba tak lama lagi. Andaikata ada sebuah fitur 
yang kelewatan untuk masuk ke dalam satu rilisan spesifik, tidak perlu khawatir sama 
sekali: rilisan yang selanjutnya bakal segera tiba dalam waktu dekat! Hal ini 
amat membantu meredam tekanan buat buru-buru menyusupkan sebuah fitur yang barangkali 
belum dipoles secara matang (_unpolished_) mendekati tenggat waktu rilisan (_deadline_).

Berkat proses yang berjalan ini, Anda selalu bisa memantau (_check out_) hasil rakitan 
Rust yang berikutnya dan membuktikan sendiri bahwa melakukan _upgrade_ itu ternyata 
mudah: kalau sebuah rilisan beta tidak bekerja sebagaimana mestinya, Anda sanggup 
melaporkannya ke tim kami dan membuatnya dibereskan sebelum perilisan versi 
stabil tiba! Kerusakan pada rilisan versi beta itu sifatnya relatif sangat langka, tapi 
`rustc` pada akhirnya adalah sebuah perangkat lunak, dan kecacatan (_bugs_) itu 
nyatanya emang tetap ada.

### Waktu Pemeliharaan (Maintenance time)

Proyek Rust menyokong (_supports_) versi stabil yang paling mutakhir (baru). 
Ketika sebuah versi stabil yang baru dirilis, versi yang lama bakal otomatis mencapai batas 
umurnya (_end of life / EOL_). Hal ini berarti masing-masing versi cuma bakal disokong 
selama rentang enam minggu saja.

### Unstable Features (Fitur-Fitur yang Belum Stabil)

Ada satu lagi rintangan yang perlu diperhatikan dengan pola model perilisan ini: yakni 
_unstable features_ (fitur-fitur yang tidak stabil). Rust memakai sebuah teknik 
bernama "feature flags" (bendera fitur) buat menentukan fitur-fitur mana saja yang 
diaktifkan (enabled) di dalam sebuah rilisan tertentu. Kalau sebuah fitur anyar ini 
lagi ada di dalam tahap pengembangan aktif, dia akan mendarat di `master`, dan 
oleh karenanya ada di versi _nightly_, tapi fitur tersebut tetap dikunci di 
balik _feature flag_. Kalau Anda sebagai pengguna, berharap ingin bisa mencicipi 
fitur yang tengah dalam masa pengerjaan ini, Anda bisa, tapi Anda harus memakai versi 
rilisan `nightly` dari Rust dan secara sadar memberi anotasi pada kode Anda dengan bendera 
(_flag_) yang pantas tersebut supaya Anda bisa setuju mengaktifkannya (_opt in_).

Kalau Anda ini memakai versi rilisan `beta` atau `stable` dari Rust, Anda tidak 
bisa memakai _feature flags_ apa pun. Inilah kunci utamanya yang membiarkan kita 
mendapatkan pengalaman memakai fitur-fitur yang baru di praktik yang sungguhan sebelum kita 
bener-bener berani mendeklarasikan fitur-fitur tersebut aman dan stabil untuk 
selamanya. Bagi mereka yang berniat terjun ke perbatasan versi yang paling canggih 
tapi berbahaya (_bleeding edge_) diperbolehkan buat masuk, dan di saat yang bersamaan, 
mereka-mereka yang mendambakan pengalaman ber-koding yang murni aman dan stabil bagai karang 
sanggup bertahan nyaman memakai versi _stable_ dan bisa yakin bahwa kode rakitan 
mereka dijamin tidak bakal hancur. Kestabilan tanpa kemandekan.

Buku ini secara mutlak cuma memuat penjelasan informasi seputar fitur-fitur stabil saja, 
karena fitur-fitur yang emang masih di tahap uji coba pengerjaan ini pastinya sifatnya 
masih suka berubah-ubah, dan pastilah bakal ada perbedaan dari bentuk mereka pas saat buku ini ditulis 
dibanding saat mereka nantinya resmi diaktifkan dan dirilis dalam balutan edisi stabil. 
Anda bisa mencari letak dokumentasi terkait fitur-fitur yang murni eksklusif cuma 
ada di _nightly_ ini secara *online*.

### Rustup dan Peran “Nightly Rust”

Alat bantu Rustup amat mempermudah Anda buat bergonta-ganti (_change_) di 
antara berbagai tipe saluran rilisan Rust yang beda-beda, entah itu secara 
global buat seluruh perangkat sistem Anda atau di ranah basis khusus per-proyek. 
Secara _default_, Anda ini dijamin sudah punya _stable_ Rust terinstal di mesin Anda. 
Buat menancapkan versi _nightly_, sebagai contohnya:

```console
$ rustup toolchain install nightly
```

Anda juga sanggup melihat seluruh _toolchains_ (rilisan dari Rust dan 
komponen-komponen terkaitnya) yang udah Anda instal pakai utilitas `rustup` ini. 
Berikut ini adalah ilustrasi di salah satu mesin perangkat komputer Windows punya 
salah satu penulis:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Sebagaimana yang bisa Anda amati sendiri, *toolchain* stabil itu statusnya adalah 
bawaan standar (_default_). Mayoritas orang yang memakai Rust emang memanfaatkan 
versi stabil dalam sebagian besar rutinitasnya. Anda barangkali juga lebih memilih buat 
rutin memakai versi stabil hampir setiap waktu, tapi ada kalanya ingin 
mengganti khusus memakai _nightly_ untuk satu _project_ tertentu secara spesifik, 
gara-gara Anda emang butuh buat menjajal suatu fitur tingkat paling canggih yang 
baru mau dirilis (_cutting-edge feature_). Buat memuluskan hal tersebut, Anda 
bisa memakai perintah `rustup override` dari dalam bilik direktori kepunyaan 
_project_ tersebut lalu menata (_set_) bahwasanya _toolchain_ `nightly` ini lah 
yang emang semestinya mesti ditarik dan dipake sama sang alat `rustup` 
andaikata Anda lagi ada di dalem wilayah bilik direktori tersebut:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Mulai dari saat ini, setiap saat Anda memanggil _compiler_ `rustc` atau `cargo` 
saat lagi berdiam di dalam *folder* _~/projects/needs-nightly_, `rustup` bakal 
memastikan dengan disiplin kalau Anda ini emang murni lagi ditarik supaya murni memakai 
edisi _nightly_ Rust, ketimbang dikasih jatah rilisan _stable_ Rust yang bawaan aslinya. 
Fitur ajaib macam ini bakal kerasa amat ngebantu (_comes in handy_) banget saat 
jumlah rentetan _projects_ Rust punya asuhan Anda ini emang lagi seabrek banyaknya!

### Proses RFC dan Timnya (Teams)

Lantas bagaimana dong caranya Anda bisa menimba wawasan soal fitur-fitur baru 
(new features) tsb? Bentuk model pengembangan bahasa Rust emang setia berjalan mengikuti 
satu alur _Request For Comments (RFC) process_ (proses pengajuan saran umpan balik). 
Andaikata Anda mendambakan adanya perombakan ke arah yang lebih oke (_improvement_) 
di dalam tubuh Rust, Anda ini diizinkan buat mengajukan sebuah draf proposal 
secara tertulis, yang mana lantas draf ini dijuluki sebagai sebuah RFC.

Siapa saja pada hakikatnya bebas dan sah untuk menyusun (_write_) lembar proposal RFC buat 
menyempurnakan performa Rust, dan aneka rupa bentuk draf usulan proposal ini lantas bakal 
diulas (_reviewed_) dan dikupas didiskusikan oleh kawan-kawan dari tim Rust, 
yang mana jajaran wujud organisasinya ini tersusun dari perpaduan sekian porsi tim-tim 
bawahan yang menaungi tiap spesialis sub-topik. Terdapat paparan daftar utuh (_full list_) 
tentang komplotan tim ini [di website milik Rust](https://www.rust-lang.org/governance), 
yang mana di dalamnya menjabarkan keberadaan _teams_ yang memfasilitasi setiap bidang ranah 
proyek tsb: perihal _language design_ (tatanan desain bahasa pemrograman), _compiler 
implementation_ (pengembangan perangkat _compiler_), _infrastructure_ (pengaturan sarana), 
_documentation_ (dokumentasi pedoman panduan), dan masih banyak lagi yang lainnya. Pihak 
_team_ yang berkaitan dengan usulan tsb bakal membacanya beserta dengan barisan umpan balik 
saran komentar tsb, dan di saat yang sama mereka juga ikutan menggoreskan lemparan 
tanggapan ulasan pandangan mereka, dan baru deh pada detik ujung puncaknya (_eventually_), 
sebuah kata sepakat mufakat konsensus bersama bakal diraih buat meresmikan mutusin apakah 
hendak mengadopsi menyetujui (_accept_) atau justru menepis (_reject_) menolak 
penerapan usulan fitur tsb.

Andaikata usulan atas wujud suatu fitur tersebut benar disetujui untuk diadopsi 
(_accepted_), otomatis akan ada sebuah _issue_ resmi yang akan dibikin dibuka 
menghiasi beranda laman pelataran repo (Rust repository) dan siapapun pun dibolehin 
untuk turun tangan sudi membedah menggarap menukangi mengimplementasikannya. 
Bisa sangat mungkin juga figur perwujudan seseorang sosok (_The person_) perajin tangan yang 
sukses kelar nuntasin ngerajut ngejait (implements) ngimplementasiin hasil fitur tsb ternyata 
malahan jatuh kepada mereka-mereka yang di titik paling awal mulanya tidak ikutan ngeproposalkan 
nyusun mengutarakan usulan tersebut! Manakala waktu (_When_) hasil pekerjaan garapan 
wujud asuhan penyematan atas implementasinya ini telah genap siap tempur (_ready_), ia 
lantas bakal mulus segera menyandar dan mengendap masuk ke _branch_ `master` namun dengan 
posisi tertahan di belakang segel kunci sebuah portal gembok gerbang batas batas gerbang fitur (_feature gate_), 
persis sebangun sejalan seperti yang pernah kita ulas di dalam segmen bagian 
[“Unstable Features (Fitur-Fitur yang Belum Stabil)”](#unstable-features) ini sebelumnya.

Nunggu berlalu beberapa jangka masa jeda (_After some time_), yaitu pas setelah porsi 
rombongan deretan kawanan _Rust developers_ (programer pemakai penganut bahasa Rust) yang udah 
sehari-harinya kebiasa ngandelin narik asupan rilisan rute jajaran mutasi dari wujud *nightly 
releases* ini pada sukses kebagian unjuk kesempatan ngejajal dan sudi nimbrung main mencoba 
iseng main ngetes (_try out_) mengujicobakan fungsi hasil wujud si rupa _new feature_ (fitur baru) 
tersebut, kawanan delegasi *team members* inilah yang gantian lantas memikul wewenang bertugas 
membahas membedah menguliti (_discuss_) kebolehan si usulan terobosan ini, lalu mengukur mengupas seputar gimana aja rekam jejak sepakterjang rekap aksi pencapaiannya 
beraksi berlaga eksis berfungsi di dalam sarang lingkup _nightly_ tersebut, terus dari sisa ulasan ini 
barulah dimufakatkan buat mutusin nentuin bareng mutusin (_decide_) apakah emang sepatutnya 
pantes (should) fitur anyar gres anyar kinyis-kinyis tsb diinagurasikan diangkat derajat dinobatkan dibiarkan bebas 
masuk nyusup melebur mulus mbrojol nembus diluncurkan dipromosi disahkan diangkat (_make it_) mbrojol masuk menyebrang nembus menyatu masuk (_into_) nyempil numpang di porsi jajaran kasta _stable Rust_ (Rust yang sifatnya udah resmi stabil) ataupun 
enggak usah sama sekali. Kalau aja seumpamanya draf perumusan hasil putusan keputusannya ini (_the decision_) mengarah menunjuk kuat berani ngambil laju inisiatif ke arah memajukan (_move forward_) nerusin nyorong meloloskan lanjut 
ke depan ngedorong memberanikan diri ngasih lampu hijau menaikkan tingkat, rupa bentuk belenggu si _feature gate_ portal palang gembok fiturnya 
otomatis dicabut dibuka ditarik diberangus dihancurkan (_removed_), yang ngebuat _the feature_ 
kepingan fiturnya ini lantas mulai saat itu detik ini pun sah ditasbihkan diresmikan dinyatakan mutlak dianggap 
menyandang status stabil (`stable`) secara paripurna tuntas seutuhnya! Selanjutnya wujud ini bakal otomatis sigap numpang nebeng nangkring 
masuk ke rangkaian iringan (_rides the trains_) di kereta rilisan ditarik melenggang terbang mengudara menuju ke 
dalam pusaran gelombang (_into_) peluncuran bongkahan wujud asuhan keluaran stabil asuhan Rust yang bener-bener asri gres (_a new stable release of Rust_).
