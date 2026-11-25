## Lampiran B: Operator dan Simbol

Lampiran ini berisi glosarium (kamus ringkas) tentang sintaks Rust, termasuk 
operator dan simbol-simbol lain yang muncul sendirian maupun di dalam konteks 
seperti _paths_, _generics_, _trait bounds_, _macros_, atribut, komentar, *tuples*, 
dan tanda kurung.

### Operator

Tabel B-1 berisi daftar operator di Rust, contoh bagaimana operator tersebut 
muncul di dalam konteksnya, penjelasan singkat, dan apakah operator 
tersebut bisa di-overload (_overloadable_) atau tidak. Kalau sebuah operator 
bisa di-overload, trait relevan yang dipakai untuk nge-overload operator 
tersebut juga dicantumkan.

<span class="caption">Tabel B-1: Operator</span>

| Operator                  | Contoh                                                  | Penjelasan                                                            | Bisa Di-overload? |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | ----------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Ekspansi macro                                                        |                   |
| `!`                       | `!expr`                                                 | Bitwise atau logical complement (komplemen logika)                    | `Not`             |
| `!=`                      | `expr != expr`                                          | Perbandingan ketidaksamaan (nonequality)                              | `PartialEq`       |
| `%`                       | `expr % expr`                                           | Sisa hasil bagi (remainder) aritmatika                                | `Rem`             |
| `%=`                      | `var %= expr`                                           | Sisa hasil bagi aritmatika dan assignment (penugasan)                 | `RemAssign`       |
| `&`                       | `&expr`, `&mut expr`                                    | Meminjam (Borrow)                                                     |                   |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Tipe pointer pinjaman (Borrowed pointer type)                         |                   |
| `&`                       | `expr & expr`                                           | Bitwise AND                                                           | `BitAnd`          |
| `&=`                      | `var &= expr`                                           | Bitwise AND dan assignment                                            | `BitAndAssign`    |
| `&&`                      | `expr && expr`                                          | Logical AND (hubungan singkat / short-circuiting)                     |                   |
| `*`                       | `expr * expr`                                           | Perkalian aritmatika                                                  | `Mul`             |
| `*=`                      | `var *= expr`                                           | Perkalian aritmatika dan assignment                                   | `MulAssign`       |
| `*`                       | `*expr`                                                 | Dereference (Membuka rujukan)                                         | `Deref`           |
| `*`                       | `*const type`, `*mut type`                              | Raw pointer                                                           |                   |
| `+`                       | `trait + trait`, `'a + trait`                           | Batasan tipe gabungan (Compound type constraint)                      |                   |
| `+`                       | `expr + expr`                                           | Penjumlahan aritmatika                                                | `Add`             |
| `+=`                      | `var += expr`                                           | Penjumlahan aritmatika dan assignment                                 | `AddAssign`       |
| `,`                       | `expr, expr`                                            | Pemisah argumen dan elemen                                            |                   |
| `-`                       | `- expr`                                                | Negasi aritmatika                                                     | `Neg`             |
| `-`                       | `expr - expr`                                           | Pengurangan aritmatika                                                | `Sub`             |
| `-=`                      | `var -= expr`                                           | Pengurangan aritmatika dan assignment                                 | `SubAssign`       |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Tipe kembalian (return type) fungsi dan closure                       |                   |
| `.`                       | `expr.ident`                                            | Akses ke field                                                        |                   |
| `.`                       | `expr.ident(expr, ...)`                                 | Pemanggilan method                                                    |                   |
| `.`                       | `expr.0`, `expr.1`, dan seterusnya                      | Pengindeksan tuple                                                    |                   |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Literal range eksklusif-kanan                                         | `PartialOrd`      |
| `..=`                     | `..=expr`, `expr..=expr`                                | Literal range inklusif-kanan                                          | `PartialOrd`      |
| `..`                      | `..expr`                                                | Sintaks pembaruan literal struct (Struct literal update syntax)       |                   |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Pengikatan pattern “dan sisanya”                                      |                   |
| `...`                     | `expr...expr`                                           | (Deprecated, pakai `..=` sebagai gantinya) Di pattern: range inklusif |                   |
| `/`                       | `expr / expr`                                           | Pembagian aritmatika                                                  | `Div`             |
| `/=`                      | `var /= expr`                                           | Pembagian aritmatika dan assignment                                   | `DivAssign`       |
| `:`                       | `pat: type`, `ident: type`                              | Batasan tipe (Constraints)                                            |                   |
| `:`                       | `ident: expr`                                           | Penginisialisasi field struct                                         |                   |
| `:`                       | `'a: loop {...}`                                        | Label loop                                                            |                   |
| `;`                       | `expr;`                                                 | Pengakhir statement dan item                                          |                   |
| `;`                       | `[...; len]`                                            | Bagian dari sintaks array berukuran tetap                             |                   |
| `<<`                      | `expr << expr`                                          | Pergeseran bit ke kiri (Left-shift)                                   | `Shl`             |
| `<<=`                     | `var <<= expr`                                          | Left-shift dan assignment                                             | `ShlAssign`       |
| `<`                       | `expr < expr`                                           | Perbandingan lebih kecil dari                                         | `PartialOrd`      |
| `<=`                      | `expr <= expr`                                          | Perbandingan lebih kecil atau sama dengan                             | `PartialOrd`      |
| `=`                       | `var = expr`, `ident = type`                            | Assignment / ekuivalensi                                              |                   |
| `==`                      | `expr == expr`                                          | Perbandingan kesamaan (Equality)                                      | `PartialEq`       |
| `=>`                      | `pat => expr`                                           | Bagian dari sintaks arm `match`                                       |                   |
| `>`                       | `expr > expr`                                           | Perbandingan lebih besar dari                                         | `PartialOrd`      |
| `>=`                      | `expr >= expr`                                          | Perbandingan lebih besar atau sama dengan                             | `PartialOrd`      |
| `>>`                      | `expr >> expr`                                          | Pergeseran bit ke kanan (Right-shift)                                 | `Shr`             |
| `>>=`                     | `var >>= expr`                                          | Right-shift dan assignment                                            | `ShrAssign`       |
| `@`                       | `ident @ pat`                                           | Pengikatan pattern (Pattern binding)                                  |                   |
| `^`                       | `expr ^ expr`                                           | Bitwise exclusive OR                                                  | `BitXor`          |
| `^=`                      | `var ^= expr`                                           | Bitwise exclusive OR dan assignment                                   | `BitXorAssign`    |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Alternatif di _pattern_                                               |                   |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Bitwise OR                                                            | `BitOr`           |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Bitwise OR dan assignment                                             | `BitOrAssign`     |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Logical OR (hubungan singkat / short-circuiting)                      |                   |
| `?`                       | `expr?`                                                 | Propagasi error                                                       |                   |

### Simbol Non-Operator

Tabel-tabel berikut ini berisi semua simbol yang tidak berfungsi sebagai operator; 
yang artinya, mereka tidak berperilaku seperti pemanggilan fungsi atau method.

Tabel B-2 menunjukkan simbol-simbol yang muncul sendirian dan valid dipakai di 
berbagai macam tempat.

<span class="caption">Tabel B-2: Sintaks Berdiri Sendiri (Stand-Alone Syntax)</span>

| Simbol                                                                 | Penjelasan                                                                         |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `'ident`                                                               | _Lifetime_ bernama atau label loop                                                 |
| Digit yang langsung diikuti `u8`, `i32`, `f64`, `usize`, dan seterusnya | Literal numerik bertipe spesifik                                                   |
| `"..."`                                                                | Literal string                                                                     |
| `r"..."`, `r#"..."#`, `r##"..."##`, dan seterusnya                     | Literal raw string; karakter escape tidak diproses                                 |
| `b"..."`                                                               | Literal byte string; membangun array of bytes alih-alih string                     |
| `br"..."`, `br#"..."#`, `br##"..."##`, dan seterusnya                  | Literal raw byte string; gabungan raw dan byte string literal                      |
| `'...'`                                                                | Literal karakter                                                                   |
| `b'...'`                                                               | Literal byte ASCII                                                                 |
| <code>&vert;...&vert; expr</code>                                      | Closure                                                                            |
| `!`                                                                    | Tipe bottom yang selalu kosong untuk fungsi yang tidak pernah kembali (diverging)  |
| `_`                                                                    | Pengikatan pattern yang “diabaikan”; juga dipakai biar literal integer mudah dibaca |

Tabel B-3 menunjukkan simbol-simbol yang muncul di dalam konteks penulisan 
jalur (path) melewati hierarki modul menuju sebuah item.

<span class="caption">Tabel B-3: Sintaks Terkait Path</span>

| Simbol                                  | Penjelasan                                                                                                    |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `ident::ident`                          | Path namespace                                                                                                |
| `::path`                                | Path relatif terhadap akar crate (yaitu path absolut eksplisit)                                               |
| `self::path`                            | Path relatif terhadap modul saat ini (yaitu path relatif eksplisit)                                           |
| `super::path`                           | Path relatif terhadap modul induk dari modul saat ini                                                         |
| `type::ident`, `<type as trait>::ident` | Konstanta, fungsi, dan tipe terkait (associated)                                                              |
| `<type>::...`                           | Item terkait untuk tipe yang tidak bisa dinamai langsung (misalnya, `<&T>::...`, `<[T]>::...`, dan seterusnya) |
| `trait::method(...)`                    | Menghilangkan ambiguitas pemanggilan method dengan menyebut nama trait yang mendefinisikannya                  |
| `type::method(...)`                     | Menghilangkan ambiguitas pemanggilan method dengan menyebut nama tipe tempat method itu didefinisikan         |
| `<type as trait>::method(...)`          | Menghilangkan ambiguitas pemanggilan method dengan menyebut nama trait dan tipenya                             |

Tabel B-4 menunjukkan simbol-simbol yang muncul di konteks pemakaian 
parameter tipe _generic_.

<span class="caption">Tabel B-4: Generics</span>

| Simbol                         | Penjelasan                                                                                                                                             |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `path<...>`                    | Menentukan parameter untuk tipe generik di dalam sebuah tipe (misalnya, `Vec<u8>`)                                                                     |
| `path::<...>`, `method::<...>` | Menentukan parameter untuk tipe, fungsi, atau method generik di dalam ekspresi; sering disebut _turbofish_ (misalnya, `"42".parse::<i32>()`)         |
| `fn ident<...> ...`            | Mendefinisikan fungsi generik                                                                                                                          |
| `struct ident<...> ...`        | Mendefinisikan struct generik                                                                                                                          |
| `enum ident<...> ...`          | Mendefinisikan enum generik                                                                                                                            |
| `impl<...> ...`                | Mendefinisikan implementasi generik                                                                                                                    |
| `for<...> type`                | Batasan lifetime tingkat lanjut (Higher-ranked lifetime bounds)                                                                                        |
| `type<ident=type>`             | Tipe generik di mana satu atau lebih _associated types_ diberi nilai spesifik (misalnya, `Iterator<Item=T>`)                                           |

Tabel B-5 menunjukkan simbol-simbol yang muncul di dalam konteks untuk 
membatasi (constraining) parameter tipe generik dengan _trait bounds_ 
(batasan trait).

<span class="caption">Tabel B-5: Batasan Trait Bound</span>

| Simbol                        | Penjelasan                                                                                                                                                    |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                        | Parameter generik `T` dibatasi pada tipe-tipe yang mengimplementasikan `U`                                                                                    |
| `T: 'a`                       | Tipe generik `T` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a` (artinya tipe tersebut tidak boleh secara transitif menampung referensi berumur lebih pendek dari `'a`) |
| `T: 'static`                  | Tipe generik `T` tidak punya referensi pinjaman (borrowed references) selain referensi yang bersifat `'static`                                                |
| `'b: 'a`                      | _Lifetime_ generik `'b` wajib berumur lebih panjang (outlive) dari _lifetime_ `'a`                                                                            |
| `T: ?Sized`                   | Mengizinkan parameter tipe generik untuk bisa berupa tipe yang berukuran dinamis (dynamically sized type)                                                     |
| `'a + trait`, `trait + trait` | Batasan tipe gabungan (Compound type constraint)                                                                                                              |

Tabel B-6 menunjukkan simbol-simbol yang muncul di konteks untuk 
memanggil atau mendefinisikan _macros_ dan menentukan atribut untuk 
sebuah item.

<span class="caption">Tabel B-6: Macros dan Atribut</span>

| Simbol                                      | Penjelasan                         |
| ------------------------------------------- | ---------------------------------- |
| `#[meta]`                                   | Atribut luar (Outer attribute)     |
| `#![meta]`                                  | Atribut dalam (Inner attribute)    |
| `$ident`                                    | Substitusi (penggantian) macro     |
| `$ident:kind`                               | Metavariabel macro                 |
| `$(...)...`                                 | Pengulangan macro (Macro repetition) |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Pemanggilan (invocation) macro     |

Tabel B-7 menunjukkan simbol-simbol yang berfungsi untuk membuat 
komentar.

<span class="caption">Tabel B-7: Komentar</span>

| Simbol     | Penjelasan                                                       |
| ---------- | ---------------------------------------------------------------- |
| `//`       | Komentar baris                                                   |
| `//!`      | Komentar dokumentasi baris bagian dalam (Inner line doc comment) |
| `///`      | Komentar dokumentasi baris bagian luar (Outer line doc comment)  |
| `/*...*/`  | Komentar blok (Block comment)                                    |
| `/*!...*/` | Komentar dokumentasi blok bagian dalam (Inner block doc comment) |
| `/**...*/` | Komentar dokumentasi blok bagian luar (Outer block doc comment)  |

Tabel B-8 menunjukkan konteks di mana tanda kurung biasa (parentheses) 
dipakai.

<span class="caption">Tabel B-8: Tanda Kurung Biasa (Parentheses)</span>

| Simbol            | Penjelasan                                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------------------------- |
| `()`              | Tuple kosong (dikenal juga dengan sebutan unit), baik literal maupun tipenya                               |
| `(expr)`          | Ekspresi yang dibungkus tanda kurung                                                                       |
| `(expr,)`         | Ekspresi tuple dengan satu elemen tunggal                                                                  |
| `(type,)`         | Tipe tuple dengan satu elemen tunggal                                                                      |
| `(expr, ...)`     | Ekspresi tuple                                                                                             |
| `(type, ...)`     | Tipe tuple                                                                                                 |
| `expr(expr, ...)` | Ekspresi pemanggilan fungsi; juga dipakai buat menginisialisasi _tuple structs_ dan varian _tuple enum_    |

Tabel B-9 menunjukkan konteks di mana kurung kurawal (curly brackets) dipakai.

<span class="caption">Tabel B-9: Kurung Kurawal (Curly Brackets)</span>

| Konteks      | Penjelasan     |
| ------------ | -------------- |
| `{...}`      | Ekspresi blok  |
| `Type {...}` | Literal struct |

Tabel B-10 menunjukkan konteks di mana kurung siku (square brackets) 
dipakai.

<span class="caption">Tabel B-10: Kurung Siku (Square Brackets)</span>

| Konteks                                            | Penjelasan                                                                                                                        |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Literal array                                                                                                                     |
| `[expr; len]`                                      | Literal array yang berisi `len` salinan dari `expr`                                                                               |
| `[type; len]`                                      | Tipe array yang berisi `len` elemen bertipe `type`                                                                                |
| `expr[expr]`                                       | Pengindeksan koleksi; bisa di-overload (`Index`, `IndexMut`)                                                                      |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Pengindeksan koleksi yang berperilaku seperti slicing koleksi, memakai `Range`, `RangeFrom`, `RangeTo`, atau `RangeFull` sebagai “indeks” |
