## Lampiran B: Operator dan Simbol

Lampiran ini berisi glosarium (kamus ringkas) tentang sintaks Rust, termasuk 
operator dan simbol-simbol lain yang muncul sendirian maupun di dalam konteks 
seperti _paths_, _generics_, _trait bounds_, _macros_, atribut, komentar, *tuples*, 
dan tanda kurung.

### Operator

Tabel B-1 berisi daftar operator di Rust, contoh gimana operator tersebut 
muncul di dalam konteksnya, penjelasan singkat, dan apakah operator 
tersebut bisa di-overload (overloadable) atau tidak. Kalau sebuah operator 
bisa di-overload, trait relevan yang dipakai buat nge-overload operator 
tersebut juga dicantumkan.

<span class="caption">Tabel B-1: Operator</span>

| Operator                  | Contoh                                                  | Penjelasan                                                            | Bisa Di-overload?  |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Ekspansi macro                                                       |                |
| `!`                       | `!expr`                                                 | Bitwise atau logical complement (komplemen logika)                                        | `Not`          |
| `!=`                      | `expr != expr`                                          | Perbandingan ketidaksamaan (nonequality)                                                | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Sisa hasil bagi (remainder) aritmatika                                                  | `Rem`          |
| `%=`                      | `var %= expr`                                           | Sisa hasil bagi aritmatika dan assignment (penugasan)                                   | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Meminjam (Borrow)                                                                |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Tipe pointer pinjaman (Borrowed pointer type)                                                 |                |
| `&`                       | `expr & expr`                                           | Bitwise AND                                                           | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | Bitwise AND dan assignment                                            | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | Logical AND (hubungan singkat/short-circuiting)                                          |                |
| `*`                       | `expr * expr`                                           | Perkalian aritmatika                                             | `Mul`          |
| `*=`                      | `var *= expr`                                           | Perkalian aritmatika dan assignment                              | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference (Membuka rujukan)                                                           | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Raw pointer                                                           |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Batasan tipe gabungan (Compound type constraint)                                              |                |
| `+`                       | `expr + expr`                                           | Penjumlahan aritmatika                                                   | `Add`          |
| `+=`                      | `var += expr`                                           | Penjumlahan aritmatika dan assignment                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Pemisah argumen dan elemen                                        |                |
| `-`                       | `- expr`                                                | Negasi aritmatika                                                   | `Neg`          |
| `-`                       | `expr - expr`                                           | Pengurangan aritmatika                                                | `Sub`          |
| `-=`                      | `var -= expr`                                           | Pengurangan aritmatika dan assignment                                 | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipe balasan (return type) fungsi dan closure                                      |                |
| `.`                       | `expr.ident`                                            | Akses ke field                                                          |                |
| `.`                       | `expr.ident(expr, ...)`                                 | Pemanggilan method                                                           |                |
| `.`                       | `expr.0`, `expr.1`, dst.                                | Indeks pada tuple                                                        |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Literal rentang kanan eksklusif (Right-exclusive range)                                         | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | Literal rentang kanan inklusif (Right-inclusive range)                                         | `PartialOrd`   |
| `..`                      | `..expr`                                                | Sintaks pembaruan (update) literal struct                                          |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Pengikatan _pattern_ "Dan sisanya" (“And the rest”)                                        |                |
| `...`                     | `expr...expr`                                           | (Usang/Deprecated, pakai `..=` sebagai gantinya) Di dalam _pattern_: pola rentang inklusif |                |
| `/`                       | `expr / expr`                                           | Pembagian aritmatika                                                   | `Div`          |
| `/=`                      | `var /= expr`                                           | Pembagian aritmatika dan assignment                                    | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | Batasan-batasan (Constraints)                                                           |                |
| `:`                       | `ident: expr`                                           | Inisialisasi field struct                                              |                |
| `:`                       | `'a: loop {...}`                                        | Label perulangan (Loop label)                                                            |                |
| `;`                       | `expr;`                                                 | Terminator untuk statement dan item                                         |                |
| `;`                       | `[...; len]`                                            | Bagian dari sintaks array ukuran-tetap (fixed-size array)                                       |                |
| `<<`                      | `expr << expr`                                          | Left-shift (Geser kiri)                                                            | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | Left-shift dan assignment                                             | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | Perbandingan lebih kecil dari                                                  | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | Perbandingan lebih kecil dari atau sama dengan                                      | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | Assignment (Penugasan) / ekuivalensi                                                |                |
| `==`                      | `expr == expr`                                          | Perbandingan kesamaan                                                   | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | Bagian dari sintaks arm (lengan) match                                              |                |
| `>`                       | `expr > expr`                                           | Perbandingan lebih besar dari                                               | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | Perbandingan lebih besar dari atau sama dengan                                   | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | Right-shift (Geser kanan)                                                           | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | Right-shift dan assignment                                            | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | Pengikatan _pattern_ (Pattern binding)                                                       |                |
| `^`                       | `expr ^ expr`                                           | Bitwise exclusive OR                                                  | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | Bitwise exclusive OR dan assignment                                   | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Alternatif di _pattern_                                                  |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Bitwise OR                                                            | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Bitwise OR dan assignment                                             | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Logical OR (hubungan singkat/short-circuiting)                                           |                |
| `?`                       | `expr?`                                                 | Propagasi error                                                     |                |

### Simbol Non-Operator

Daftar berikut ini berisi semua simbol yang tidak berfungsi sebagai operator; 
yang artinya, mereka tidak berperilaku seperti pemanggilan fungsi atau method.

Tabel B-2 menunjukkan simbol-simbol yang muncul sendirian dan valid dipakai di 
berbagai macam tempat.

<span class="caption">Tabel B-2: Sintaks Berdiri Sendiri (Stand-Alone Syntax)</span>

| Simbol                                                                 | Penjelasan                                                            |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident`                                                               | Nama _lifetime_ atau label perulangan                                           |
| Angka yang langsung diikuti dengan `u8`, `i32`,  `f64`, `usize`, dsb. | Literal numerik dengan tipe spesifik                                       |
| `"..."`                                                                | Literal string                                                         |
| `r"..."`, `r#"..."#`, `r##"..."##`, dsb.                               | Literal _raw string_ (string mentah), karakter escape di dalamnya tidak diproses                    |
| `b"..."`                                                               | Literal _byte string_; bikin array dari bytes ketimbang string  |
| `br"..."`, `br#"..."#`, `br##"..."##`, dsb.                            | Literal _raw byte string_, gabungan dari _raw string_ dan _byte string_    |
| `'...'`                                                                | Literal karakter                                                      |
| `b'...'`                                                               | Literal ASCII byte                                                     |
| <code>&vert;...&vert; expr</code>                                      | Closure                                                                |
| `!`                                                                    | Tipe dasar (bottom type) yang selalu kosong untuk fungsi-fungsi divergen                       |
| `_`                                                                    | Pengikatan _pattern_ "Abaikan" (Ignored); juga dipakai buat ngebikin literal integer jadi gampang dibaca |

Tabel B-3 nunjukin simbol-simbol yang muncul di dalam konteks penulisan 
jalur (path) melewati hierarki modul menuju sebuah item.

<span class="caption">Tabel B-3: Sintaks Terkait Path</span>

| Simbol                                  | Penjelasan                                                                                                                     |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `ident::ident`                          | Path ke namespace                                                                                                                  |
| `::path`                                | Path relatif ke arah _extern prelude_, di mana semua _crates_ lain berakar (contohnya, jalur absolut eksplisit yang menyertakan nama _crate_) |
| `self::path`                            | Path yang relatif terhadap modul saat ini (yakni, jalur relatif eksplisit).                                                        |
| `super::path`                           | Path yang relatif terhadap _parent_ (induk) dari modul saat ini                                                                               |
| `type::ident`, `<type as trait>::ident` | Konstanta, fungsi, dan tipe-tipe _associated_ (terkait)                                                                                      |
| `<type>::...`                           | Item _associated_ buat tipe yang mana tidak bisa dinamai secara langsung (contoh, `<&T>::...`, `<[T]>::...`, dsb.)                                |
| `trait::method(...)`                    | Menghilangkan ambiguitas pemanggilan method dengan menyebutkan nama trait yang mendefinisikannya                                                                |
| `type::method(...)`                     | Menghilangkan ambiguitas pemanggilan method dengan menyebutkan nama tipe di mana method itu didefinisikan                                                          |
| `<type as trait>::method(...)`          | Menghilangkan ambiguitas pemanggilan method dengan menyebutkan nama trait dan juga tipenya                                                                       |

Tabel B-4 nunjukin simbol-simbol yang muncul di konteks pemakaian 
parameter tipe _generic_.

<span class="caption">Tabel B-4: Generics</span>

| Simbol                         | Penjelasan                                                                                                                              |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Menentukan parameter untuk tipe generik di dalam sebuah tipe (contoh, `Vec<u8>`)                                                                         |
| `path::<...>`, `method::<...>` | Menentukan parameter untuk tipe generik, fungsi, atau method di dalam sebuah ekspresi; sering disebut dengan _turbofish_ (contoh, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Mendefinisikan fungsi generik                                                                                                                  |
| `struct ident<...> ...`        | Mendefinisikan struct generik                                                                                                                 |
| `enum ident<...> ...`          | Mendefinisikan enumerasi generik                                                                                                               |
| `impl<...> ...`                | Mendefinisikan implementasi generik                                                                                                            |
| `for<...> type`                | Batasan _higher-ranked lifetime_                                                                                                            |
| `type<ident=type>`             | Sebuah tipe generik di mana satu atau lebih _associated types_ punya pengisian (assignments) yang spesifik (contoh, `Iterator<Item=T>`)                                   |

Tabel B-5 menunjukkan simbol-simbol yang muncul di dalam konteks untuk 
membatasi (constraining) parameter tipe generik dengan _trait bounds_ 
(batasan trait).

<span class="caption">Tabel B-5: Batasan Trait Bound</span>

| Simbol                        | Penjelasan                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U`                        | Parameter generik `T` dibatasi pada tipe-tipe yang mengimplementasikan `U`                                                                              |
| `T: 'a`                       | Tipe generik `T` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a` (artinya tipe tersebut tidak boleh secara transitif menampung referensi yang punya _lifetime_ lebih pendek dari `'a`) |
| `T: 'static`                  | Tipe generik `T` tidak punya referensi pinjaman (borrowed references) selain referensi yang bersifat `'static`                                                                 |
| `'b: 'a`                      | _Lifetime_ generik `'b` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a`                                                                                           |
| `T: ?Sized`                   | Mengizinkan parameter tipe generik untuk bisa berupa tipe yang berukuran dinamis (dynamically sized type)                                                                                |
| `'a + trait`, `trait + trait` | Batasan tipe gabungan (Compound type constraint)                                                                                                                   |

Tabel B-6 menunjukkan simbol-simbol yang muncul di konteks untuk 
memanggil atau mendefinisikan _macros_ dan nentuin atribut buat 
sebuah item.

<span class="caption">Tabel B-6: Macros dan Atribut</span>

| Simbol                                      | Penjelasan        |
| ------------------------------------------- | ------------------ |
| `#[meta]`                                   | Atribut luar (Outer attribute)    |
| `#![meta]`                                  | Atribut dalam (Inner attribute)    |
| `$ident`                                    | Substitusi (penggantian) macro |
| `$ident:kind`                               | Metavariabel macro |
| `$(...)...`                                 | Pengulangan macro (Macro repetition)   |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Pemanggilan (invocation) macro   |

Tabel B-7 menunjukkan simbol-simbol yang berfungsi buat ngebikin 
komentar.

<span class="caption">Tabel B-7: Komentar</span>

| Simbol     | Penjelasan             |
| ---------- | ----------------------- |
| `//`       | Komentar baris            |
| `//!`      | Komentar dokumentasi baris bagian dalam (Inner line doc comment)  |
| `///`      | Komentar dokumentasi baris bagian luar (Outer line doc comment)  |
| `/*...*/`  | Komentar blok (Block comment)           |
| `/*!...*/` | Komentar dokumentasi blok bagian dalam (Inner block doc comment) |
| `/**...*/` | Komentar dokumentasi blok bagian luar (Outer block doc comment) |

Tabel B-8 nunjukin konteks di mana tanda kurung biasa (parentheses) 
dipakai.

<span class="caption">Tabel B-8: Tanda Kurung Biasa (Parentheses)</span>

| Simbol                   | Penjelasan                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Tuple kosong (dikenal juga dengan sebutan unit), baik literal maupun tipenya                                               |
| `(expr)`                 | Ekspresi yang dibungkus tanda kurung                                                                    |
| `(expr,)`                | Ekspresi tuple dengan satu elemen tunggal                                                             |
| `(type,)`                | Tipe tuple dengan satu elemen tunggal                                                                   |
| `(expr, ...)`            | Ekspresi tuple                                                                            |
| `(type, ...)`            | Tipe tuple                                                                                  |
| `expr(expr, ...)`        | Ekspresi pemanggilan fungsi; juga dipakai buat menginisialisasi _tuple structs_ dan varian dari _tuple enum_ |

Tabel B-9 nunjukin konteks di mana kurung kurawal (curly braces) 
dipakai.

<span class="caption">Tabel B-9: Kurung Kurawal (Curly Brackets)</span>

| Konteks      | Penjelasan      |
| ------------ | ---------------- |
| `{...}`      | Ekspresi blok |
| `Type {...}` | Literal struct   |

Tabel B-10 nunjukin konteks di mana kurung siku (square brackets) 
dipakai.

<span class="caption">Tabel B-10: Kurung Siku (Square Brackets)</span>

| Konteks                                            | Penjelasan                                                                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Literal array                                                                                                                 |
| `[expr; len]`                                      | Literal array yang berisi `len` jumlah salinan (copies) dari `expr`                                                                               |
| `[type; len]`                                      | Tipe array yang berisi `len` *instances* dari `type`                                                                               |
| `expr[expr]`                                       | Indexing koleksi (Collection indexing). Bisa di-overload (`Index`, `IndexMut`)                                                                       |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Indexing koleksi yang berpura-pura menjadi teknik memotong (slicing) koleksi, dengan memakai `Range`, `RangeFrom`, `RangeTo`, atau `RangeFull` sebagai "indeks"-nya |
