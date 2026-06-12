## Lampiran C: Derivable Traits (Trait yang Bisa Di-derive)

Di berbagai tempat di buku ini, kita udah ngebahas atribut `derive`, 
yang mana bisa Anda sematkan ke definisi _struct_ atau _enum_. Atribut `derive` 
ini bakal menghasilkan kode yang mengimplementasikan (implement) sebuah trait 
lengkap dengan implementasi _default_ (bawaan)-nya pada tipe yang udah Anda 
anotasi (annotated) memakai sintaks `derive` tersebut.

Di lampiran ini, kita menyediakan referensi dari semua traits di _standard library_ 
yang mana bisa Anda pakai dengan atribut `derive`. Masing-masing bagian mencakup:

- Operator dan method apa saja yang bakal difungsikan (_enable_) dengan nge-_derive_ trait ini
- Apa saja yang dilakukan sama implementasi trait yang disediain oleh `derive` tersebut
- Apa makna (signifies) dari mengimplementasikan trait tersebut bagi tipe Anda
- Persyaratan dan kondisi di mana Anda dibolehkan atau tidak dibolehkan buat 
  mengimplementasikan trait ini
- Contoh-contoh operasi yang mewajibkan adanya trait ini

Kalau Anda pengen mendapatkan perilaku yang berbeda dari yang disediain sama atribut 
`derive`, silakan cek dokumentasi [standard library](../std/index.html) buat setiap trait 
demi mendapatkan detail soal gimana caranya buat mengimplementasikan mereka secara manual.

Trait-trait yang terdaftar di sini adalah satu-satunya trait yang didefinisikan sama 
_standard library_ yang bisa diimplementasikan ke tipe Anda memakai `derive`. Trait 
lain yang didefinisikan di _standard library_ itu tidak punya perilaku bawaan (_default_) 
yang masuk akal (sensible), jadinya itu semua tergantung Anda buat mengimplementasikannya 
pakai cara yang paling masuk akal sejalan dengan apa yang mau Anda capai.

Contoh dari sebuah trait yang tidak bisa di-_derive_ adalah `Display`, yang mana 
bertugas menangani format teks buat para pengguna akhir (_end users_). Anda harus 
selalu mempertimbangkan cara yang paling pantas buat menampilkan sebuah tipe ke 
_end user_. Bagian mana aja dari tipe tersebut yang boleh dilihat sama _end user_? 
Bagian mana aja yang bakal mereka anggap relevan? Format data kayak gimana yang 
bakal paling gampang dimengerti sama mereka? _Compiler_ Rust tidak punya wawasan 
(_insight_) kayak gini, jadinya dia tidak bisa menyediakan perilaku _default_ 
yang pantas buat Anda.

Daftar _derivable traits_ (trait yang bisa di-_derive_) yang disediain di 
lampiran ini itu tidaklah komprehensif: _libraries_ lain bisa aja ngimplementasiin 
`derive` buat trait mereka sendiri, ngebikin daftar trait yang bisa Anda pakai 
dengan `derive` itu bener-bener jadi tanpa batas (open-ended). Mengimplementasikan 
`derive` ini melibatkan pemakaian _procedural macro_, yang mana udah dibahas di 
bagian [“Macros”][macros] di Bab 20.

### `Debug` Buat Output Programmer

Trait `Debug` memfungsikan (_enables_) pemformatan _debug_ di dalem format _strings_, 
yang mana bisa Anda indikasikan dengan nambahin sisipan `:?` ke dalem kurung 
kurawal `{}` (*placeholders*).

Trait `Debug` ngasih Anda kebebasan buat mencetak *instances* dari sebuah tipe buat 
tujuan *debugging* (pemeriksaan error), supaya Anda dan programmer lainnya yang 
lagi makek tipe Anda tersebut bisa menginspeksi *instance* tersebut pas ada di 
satu titik tertentu di program pas lagi dieksekusi.

Trait `Debug` diwajibkan (required), misalnya, saat kita pengen makai macro `assert_eq!`. 
Macro ini bakal mencetak nilai-nilai dari *instances* yang diberikan kepadanya 
sebagai argumen kalau penegasan kesamaannya (equality assertion) gagal supaya 
para programmer bisa lihat dengan jelas alasan kenapa kedua instance itu tidak 
sama (weren't equal).

### `PartialEq` dan `Eq` Buat Perbandingan Kesamaan (Equality Comparisons)

Trait `PartialEq` ngasih Anda kemungkinan buat ngebandingin *instances* dari sebuah tipe 
buat mengecek apakah mereka itu sama atau tidak, dan juga memfungsikan pemakaian 
operator `==` dan `!=`.

Nge-_derive_ `PartialEq` bakal mengimplementasikan method `eq`. Pas `PartialEq` di-_derive_ 
pada _structs_, dua instances dianggap sama hanya jika _semua_ bidang (_fields_)-nya sama, 
dan kedua instances itu bakal dianggap tidak sama (not equal) kalau ada bidang apa pun 
yang tidak sama. Saat di-_derive_ pada _enums_, masing-masing varian bakal dianggap sama 
dengan varian dirinya sendiri dan dianggap tidak sama dengan varian-varian yang lainnya.

Trait `PartialEq` diwajibkan, misalnya, pas lagi memakai macro `assert_eq!`, 
yang mana dia perlu bisa ngebandingin dua buah instances dari suatu tipe buat 
ngelihat apakah mereka sama (equality).

Trait `Eq` tidak punya method apa-apa. Tujuannya adalah sekadar buat ngasih tanda 
(signal) bahwa untuk setiap nilai dari tipe yang dianotasi, nilai itu dijamin pasti 
sama dengan dirinya sendiri. Trait `Eq` ini cuma bisa diterapin (applied) ke tipe-tipe 
yang juga mengimplementasikan `PartialEq`, walaupun tidak semua tipe yang 
mengimplementasikan `PartialEq` itu otomatis bisa mengimplementasikan `Eq`. Salah satu 
contohnya adalah pada tipe *floating point number*: implementasi perbandingan buat angka *floating 
point* menyatakan kalau dua instances dari nilai *not-a-number* (`NaN`) itu tidaklah sama 
(not equal) dengan satu sama lain.

Salah satu contoh kasus di mana `Eq` diwajibkan adalah untuk *keys* (kunci) di dalem 
sebuah `HashMap<K, V>` supaya si `HashMap<K, V>` ini bisa ngebedain apakah dua *keys* 
yang ada itu beneran sama (same) atau tidak.

### `PartialOrd` dan `Ord` Buat Perbandingan Pengurutan (Ordering Comparisons)

Trait `PartialOrd` memungkinkan Anda buat membandingkan instances dari suatu tipe buat 
keperluan *sorting* (pengurutan). Sebuah tipe yang mengimplementasikan `PartialOrd` bisa 
dipakai dengan operator `<`, `>`, `<=`, dan `>=`. Anda cuma bisa memakai atribut trait 
`PartialOrd` ini ke tipe-tipe yang juga udah mengimplementasikan `PartialEq`.

Nge-_derive_ `PartialOrd` bakal mengimplementasikan method `partial_cmp`, yang bakal 
mengembalikan sebuah `Option<Ordering>` yang mana isinya bakal berupa `None` kalau 
nilai-nilai yang dikasih itu ternyata gagal memproduksi sebuah urutan (ordering). 
Contoh dari sebuah nilai yang gagal memproduksi urutan, sekalipun sebagian besar nilai di 
tipe tersebut aslinya bisa dibandingin, adalah nilai *not-a-number* (`NaN`) pada *floating 
point*. Manggil `partial_cmp` memakai angka floating-point mana pun dicampur dengan nilai 
`NaN` pasti bakal mengembalikan `None`.

Saat di-_derive_ pada _structs_, `PartialOrd` ngebandingin dua buah instances dengan 
cara ngebandingin setiap nilai di dalam tiap *fields*-nya berdasarkan dengan urutan kemunculan 
*fields* tersebut di saat struct-nya didefinisikan (struct definition). Saat 
di-_derive_ pada _enums_, varian-varian enum yang dideklarasikan (muncul) lebih awal di dalam 
definisi enum bakal dianggap lebih kecil (_less than_) ketimbang varian-varian yang muncul belakangan.

Trait `PartialOrd` diwajibkan, misalnya, buat pemakaian method `gen_range` dari _crate_ 
`rand` yang tugasnya menghasilkan nilai acak di dalem jangkauan (_range_) yang 
udah dispesifikasikan pakai ekspresi _range_.

Trait `Ord` ngasih tahu Anda kalau, untuk sembarang dua nilai apa pun dari tipe yang udah 
dianotasi, pastilah selalu ada sebuah sistem pengurutan yang valid yang bakal eksis (exist). 
Trait `Ord` mengimplementasikan method `cmp`, yang mengembalikan sebuah tipe `Ordering` dan bukan 
`Option<Ordering>` karena sebuah pengurutan yang valid itu bakal selalu dijamin selalu mungkin 
buat terjadi. Anda cuma boleh naruh atribut trait `Ord` ke tipe yang mana juga udah 
mengimplementasikan `PartialOrd` sekaligus `Eq` (dan perlu diingat kalau `Eq` juga mewajibkan adanya 
`PartialEq`). Saat di-_derive_ pada _structs_ dan _enums_, method `cmp` bakal beroperasi (behaves) 
pakai cara yang sama persis kayak apa yang dilakuin sama implementasi yang di-_derive_ untuk method 
`partial_cmp` yang ada di dalam `PartialOrd`.

Salah satu contoh pas `Ord` diwajibkan (required) adalah saat kita mau nyimpen nilai-nilai 
ke dalam sebuah `BTreeSet<T>`, yaitu struktur data yang nyimpen data berdasarkan urutan *sort* (pengurutan) 
dari nilai-nilai tersebut.

### `Clone` dan `Copy` Buat Menduplikasi Nilai (Duplicating Values)

Trait `Clone` membiarkan Anda secara eksplisit ngebikin _deep copy_ (salinan mendalam) dari 
sebuah nilai, dan di mana proses duplikasi ini juga bisa jadi ngelibatin pengeksekusian 
(running) kode-kode tambahan dan penyalinan data *heap*. Silakan lihat [“Variabel dan 
Data Berinteraksi dengan Clone”][variables-and-data-interacting-with-clone] di Bab 4 
buat informasi lebih jauh soal `Clone`.

Nge-_derive_ `Clone` bakal mengimplementasikan method `clone`, yang mana saat diimplementasikan 
buat keseluruhan tipe, dia bakal memanggil `clone` juga secara beruntun pada masing-masing 
komponen dari tipe tersebut. Artinya semua fields atau bagian nilai yang ada di tipe tersebut 
harus mutlak udah mengimplementasikan `Clone` juga supaya tipe utamanya bisa nge-_derive_ 
`Clone`.

Sebuah contoh kapan `Clone` diwajibkan adalah pas kita lagi manggil method `to_vec` pada 
sebuah _slice_. Si _slice_ ini kan emang tidak ngantongin (doesn't own) instances dari tipe 
yang disimpennya, tapi vector yang dikembalikan dari panggilan `to_vec` itu jelas butuh 
dan wajib punya hak milik (own) buat instances yang ada di dalemnya, makanya si `to_vec` 
ini bakal manggil method `clone` buat setiap _item_ yang ada. Dengan demikian, tipe yang 
disimpan di dalam _slice_ tersebut wajib mengimplementasikan `Clone`.

Trait `Copy` membiarkan Anda buat menduplikasi sebuah nilai cuma dengan cara meng-copy *bits* 
yang ada di dalam memori _stack_ aja; jadinya tidak perlu ada pengeksekusian kode 
tambahan apa-apa di sini. Lihat [“Stack-Only Data: Copy”][stack-only-data-copy] 
di Bab 4 buat informasi lebih jauh soal `Copy`.

Trait `Copy` ini sama sekali tidak mendefinisikan method apa pun karena tujuannya itu buat 
mencegah (prevent) para programmer dari nge-_overload_ method tersebut dan ngelanggar (_violating_) 
asumsi mutlak bahwa tidak ada eksekusi kode acak apa pun yang berjalan selama proses duplikasi ini. 
Dengan cara kayak gini, semua programmer bisa berasumsi dengan aman (assume) kalau aksi 
meng-*copy* sebuah nilai yang punya trait ini bakal kerasa cepet banget (very fast).

Anda bisa nge-_derive_ `Copy` pada tipe apa pun yang mana semua elemen komponennya 
itu udah mengimplementasikan `Copy`. Tipe yang udah mengimplementasikan `Copy` 
juga wajib mutlak mengimplementasikan `Clone`, karena sebuah tipe yang mengimplementasikan 
`Copy` otomatis udah punya implementasi buat `Clone` yang sepele (trivial implementation) yang mana dia 
menunaikan tugas yang sama persis kayak si `Copy` tersebut.

Trait `Copy` itu jarang banget diwajibkan secara eksplisit; tapi tipe-tipe yang 
mengimplementasikan `Copy` punya privilese ketersediaan buat dioptimasi (optimizations 
available), yang artinya Anda jadinya tidak perlu rajin-rajin ngetik manggil `clone`, yang mana 
ngebikin kodenya jadi lebih padat (concise) dan ringkas.

Segala apa pun yang bisa dicapai (possible) pakai `Copy` tentu juga bisa Anda capai (accomplish) 
dengan memakai `Clone`, tapi ya kodenya itu mungkin bisa jadi agak lebih lelet (slower) atau 
menuntut keharusan buat manggil `clone` di mana-mana (in places).

### `Hash` Buat Memetakan Nilai ke Nilai Lain Berukuran Tetap (Fixed Size)

Trait `Hash` ngasih Anda kapabilitas buat ngambil *instance* dari sebuah tipe dengan ukuran 
yang sembarang (arbitrary size) lalu memetakan (_map_) instance tersebut menjadi sebuah nilai yang 
punya ukuran tetap (_fixed size_) menggunakan sebuah fungsi _hash_. Nge-_derive_ `Hash` bakal 
mengimplementasikan method `hash`. Implementasi _derived_ dari method `hash` ini bakal 
ngegabungin (combines) seluruh hasil dari pemanggilan method `hash` ke setiap komponen (_parts_) dari 
tipe tersebut, yang artinya bahwa _semua_ *fields* atau nilai yang ada di dalem tipe ini wajib juga udah 
mengimplementasikan `Hash` supaya si tipe utamanya ini bisa di-_derive_ sama `Hash`.

Contoh dari kasus di mana `Hash` diwajibkan (required) adalah pas kita mau nyimpen 
*keys* (kunci) di dalam koleksi `HashMap<K, V>` buat tujuan menyimpen data secara 
lebih efisien (efficiently).

### `Default` Buat Pembuatan Nilai Bawaan (Default Values)

Trait `Default` ngebolehin Anda buat ngebikin nilai bawaan (default value) buat sebuah tipe. 
Nge-_derive_ `Default` bakal mengimplementasikan fungsi `default`. Implementasi turunan 
(_derived implementation_) dari fungsi `default` ini bakal ngerjain tugasnya dengan cara 
ikut-ikutan manggil fungsi `default` buat setiap komponen/elemen dari tipe tersebut, yang mana 
tentu artinya semua fields atau bagian-bagian nilai di dalam tipe tersebut juga diwajibkan 
(must also) buat udah mengimplementasikan `Default` biar tipe utamanya bisa nge-_derive_ 
trait `Default`.

Fungsi `Default::default` ini umumnya sering dipakai digabungin barengan sama sintaks pembaruan struct 
(_struct update syntax_) yang pernah kita obrolin di [“Ngebikin Instances dari Instances Lain dengan 
Struct Update Syntax”][creating-instances-from-other-instances-with-struct-update-syntax] 
di Bab 5. Anda bisa mengkustomisasi beberapa *fields* tertentu aja dari sebuah struct 
dan sesudahnya itu mengatur (set) lalu memakai nilai _default_ bawaannya buat ngisi 
sisa bidang-bidang (rest of the fields) yang belum diisi dengan cara memakai kode 
timbuhan `..Default::default()`.

Trait `Default` diwajibkan saat Anda memakai method `unwrap_or_default` pada *instances* dari 
tipe `Option<T>`, contohnya. Kalau nilai `Option<T>`-nya ternyata adalah `None`, method 
`unwrap_or_default` ini nantinya bakal nge-return (ngembaliin) hasil tebakan tebasan yang asalnya dari panggilan 
`Default::default` buat si tipe `T` yang lagi disimpen di dalam `Option<T>` tersebut.

[creating-instances-from-other-instances-with-struct-update-syntax]: ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[macros]: ch20-05-macros.html#macros
