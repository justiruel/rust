# 📘 Ringkasan Konsep Rust (Ownership, Borrowing, Stack & Heap)

## 1. Ownership
- Setiap value di Rust punya **satu owner** (pemilik).
- Saat owner keluar scope, value akan di-`drop` (heap dibersihkan).

```rust
fn main() {
    {
        let s = String::from("halo"); // s adalah owner
        println!("{}", s);
    } // s keluar scope, heap otomatis dibebaskan
}
```

---

## 2. Ownership Transfer (Move)
- Kalau value di-assign ke variable lain, **ownership pindah**.
- String tidak implementasi `Copy`, sehingga `a` tidak bisa lagi digunakan setelah dipindah.

```rust
fn main() {
    let a = String::from("halo");
    let b = a; // ownership pindah ke b
    // println!("{}", a); // ❌ error: a sudah tidak valid
    println!("{}", b);
}
```

---

## 3. Copy
- Tipe sederhana (integer, bool, dll) disimpan di stack dan otomatis `Copy`.

```rust
fn main() {
    let x = 5;
    let y = x; // x dicopy, keduanya valid
    println!("x = {}, y = {}", x, y);
}
```

---

## 4. Clone
- Jika ingin menggandakan data di heap, gunakan `.clone()`.
- Ini membuat **alokasi heap baru**.

```rust
fn main() {
    let a = String::from("halo");
    let b = a.clone(); // duplikasi isi heap
    println!("a = {}, b = {}", a, b);
}
```

---

## 5. Stack vs Heap
- **Stack**: data ukuran tetap, cepat diakses (contoh: integer).
- **Heap**: data ukuran dinamis (contoh: `String`, `Vec`), stack hanya menyimpan pointer + length + capacity.

```rust
fn main() {
    let x = 10; // langsung di stack
    let s = String::from("halo"); // pointer di stack, isi "halo" di heap
}
```
- untuk tipe data yang kecil dan jelas ukurannya seperti i32, bool, f64 maka value akan langsung disimpan di stack
- sedangkan tipe data yang besar atau ukurannya belum pasti misalkan String, Vec<T> maka nilai akan disimpan di heap, sedangkan di stack yg disimpan hanya pointer (alamat heap), len, dan capacity 

---

## 6. Borrowing (Pinjam Referensi)
- Immutable borrow `&T` → bisa banyak, hanya baca.
- Mutable borrow `&mut T` → hanya satu dalam satu waktu.

```rust
fn main() {
    let mut s = String::from("halo");

    let r1 = &s;
    let r2 = &s;
    println!("{} {}", r1, r2); // ✅ banyak immutable

    let r3 = &mut s;
    r3.push_str(" dunia");
    println!("{}", r3); // ✅ hanya satu mutable
}
```

- di mutable borrow, satu chain aktif (walau panjang) → aman.
Contoh:
```rust
let mut s = String::from("halo");
let a = &mut s;       // pinjam s
let b = &mut a;       // pinjam a
let c = &mut b;       // pinjam b
c.push_str(" dunia"); // tetap cuma 1 jalur → oke
```
- ❌ Dua jalur aktif ke data yang sama → error.
Contoh:
```rust
let mut s = String::from("halo");
let a = &mut s;
let b = &mut s; 
```
---

## 7. Aturan Borrowing (Aliasing XOR Mutability)
- Tidak boleh ada immutable dan mutable borrow bersamaan.

```rust
fn main() {
    let mut s = String::from("halo");

    let r1 = &s;
    let r2 = &mut s; // ❌ error: tidak boleh baca & tulis bersamaan
    println!("{}", r1);
}
```

Analogi:  
- Banyak pembaca boleh baca buku bersama (immutable).  
- Tapi kalau ada yang pakai pulpen (mutable), cuma boleh satu orang.

---

## 8. Mutable Borrow Chain
- Referensi bisa di-*chain* selama tetap hanya ada satu jalur aktif.

```rust
fn main() {
    let mut s = String::from("halo");
    let r1 = &mut s;
    let r2 = &mut r1;
    r2.push_str(" dunia"); // ✅ aman, masih satu jalur borrow
    println!("{}", r2);
}
```

---

## 9. Length vs Capacity
- `len` → jumlah elemen saat ini.
- `capacity` → ruang yang sudah dialokasikan di heap.

```rust
fn main() {
    let mut v = Vec::new();
    v.push(1);
    println!("len = {}, capacity = {}", v.len(), v.capacity());
}
```
---

## 10. Heap Management
- Heap dibebaskan otomatis ketika owner keluar scope.
- Jika heap penuh, Rust (via OS allocator) akan mencoba memperbesar, kalau gagal → panic/abort.
---

## 11. Perbandingan: Garbage Collection vs Ownership
### Garbage Collection (misalnya di Java, Go, JavaScript):
- Runtime ada GC (Garbage Collector) yang memantau memory.
- GC akan pause program sesekali untuk membersihkan data yang tidak direferensikan.
- Overhead runtime lebih besar, ada kemungkinan stop-the-world pause.
### Rust (Ownership + Borrowing):
- Tidak ada GC.
- Memory dikelola di compile time lewat aturan ownership.
- Heap otomatis dibebaskan ketika owner keluar scope.
- Tidak ada pause runtime → performa konsisten.
### Analogi:
- GC = ada petugas kebersihan yang patroli membersihkan sampah, kadang mengganggu aktivitas.
- Rust = setiap orang wajib membersihkan sampahnya sendiri sebelum keluar ruangan.

## 12. Perbandingan Async: Node.js vs Rust
### Node.js (Promise / async-await):
- Single-threaded event loop.
- Async dikerjakan lewat callback queue (task diparkir, dijalankan ketika I/O selesai).
- Memory manajemen ditangani GC (developer tidak mengurus lifetime).
Contoh Node.js:
```rust
async function run() {
    let data = await fetch("/api");
    console.log(data);
}
```
### Rust (async / Future):
- async fn menghasilkan Future.
- Future adalah state machine, berjalan hanya jika dijalankan executor.
- Memory tetap aman karena ownership & borrow checker.

Contoh Rust:
```rust
async fn run() {
    let data = fetch_data().await;
    println!("{}", data);
}
```
### Analogi:
- Node.js: ada sekretaris (event loop) yang mencatat janji, memanggil balik kalau waktunya tiba.
- Rust: janji (Future) adalah mesin kecil yang berhenti sementara, lalu dilanjutkan oleh eksekutor saat siap.

## 13. Async: Node.js vs Rust
### Node.js:
- Menggunakan event loop + Promise.
- Semua async task non-blocking, cocok untuk I/O bound.
- Memory dikelola oleh GC.
```
async function run() {
  const a = Promise.resolve("A selesai");
  const b = Promise.resolve("B selesai");
  const results = await Promise.all([a, b]);
  console.log(results);
}
run();
```

### Rust:
- Menggunakan async/await + executor (Tokio/async-std).
- Memory tetap dikelola dengan ownership, tanpa GC.
- Cocok untuk high-performance async system.
```
use futures::future;
#[tokio::main]
async fn main() {
    let f1 = async { "A selesai" };
    let f2 = async { "B selesai" };
    let results = future::join_all(vec![f1, f2]).await;
    println!("{:?}", results);
}
```

### 📌 Perbedaan inti:
- Node.js: semua berbasis GC, aman tapi ada overhead.
- Rust: tetap tanpa GC, async tetap mengikuti aturan ownership.

# 📘 Catatan Borrowing & Non-Lexical Lifetimes (NLL) di Rust

## 1. Kasus Studi

```rust
let mut mutated_owner = String::from("halo");

let borrow_mutated_owner = &mut mutated_owner;
println!("borrow_mutated_owner : {}", borrow_mutated_owner);
borrow_mutated_owner.push_str("!!!");

let borrow_immutated_owner = &mutated_owner;
println!("borrow_immutated_owner : {}", borrow_immutated_owner);
```

### 🔑 Analisis

* Borrow mutable `borrow_mutated_owner` **hanya berlaku sampai terakhir kali digunakan** (`push_str` dipanggil).
* Setelah itu, Rust menganggap borrow sudah "mati".
* Baru kemudian `borrow_immutated_owner` bisa dibuat tanpa melanggar aturan borrow.

---

## 2. Aturan Borrowing di Rust

* **Immutable borrow (`&T`)**: bisa banyak peminjam, read-only.
* **Mutable borrow (`&mut T`)**: hanya boleh ada satu peminjam dalam satu waktu.
* **Non-Lexical Lifetimes (NLL)**: borrow dianggap selesai **setelah terakhir kali digunakan**, tidak harus menunggu akhir scope.

---

## 3. Ownership vs Borrow

| Konsep    | Penjelasan                                                                    | Contoh                            |
| --------- | ----------------------------------------------------------------------------- | --------------------------------- |
| Ownership | Pemilik data, bertanggung jawab atas alokasi + dealokasi heap                 | `let s = String::from("halo");`   |
| Borrow    | Pinjaman data, bisa immutable atau mutable, tidak mempengaruhi lifetime owner | `let r = &s;` / `let r = &mut s;` |

---

## 4. Contoh Praktis dengan NLL

```rust
fn main() {
    let mut s = String::from("halo");

    let borrow_mut = &mut s;
    borrow_mut.push_str(" dunia"); // terakhir kali dipakai

    // borrow_mut sudah "mati" di sini
    let borrow_immut = &s;
    println!("{}", borrow_immut); // aman
}
```

---

## 5. Analogi

* **Owner** = pemilik rumah (selama owner hidup, rumah ada; keluar scope → rumah dirobohkan).
* **Borrow** = pinjam kunci rumah (boleh dipakai sampai terakhir digunakan; setelah itu dianggap kunci dikembalikan).

### ✅ Kesimpulan
* Kode tidak error karena Rust tahu borrow mutable sudah selesai digunakan sebelum borrow berikutnya dibuat.
---

# 📘 Rust Attributes (`#[...]`)

## Apa itu Attribute?

Di Rust, tanda `#[...]` disebut **attribute**.  
Attribute adalah *metadata* yang ditambahkan pada item (struct, enum, function, module, crate, dll) untuk memberi instruksi tambahan kepada compiler, linter, atau macro.  

Ada dua bentuk utama:

- `#[...]` → attribute untuk item tertentu (function, struct, enum, dll).
- `#![...]` → attribute level **crate** atau **module** (biasanya di paling atas file `lib.rs` atau `main.rs`).

Attribute mirip dengan **annotation** di Java (`@Override`) atau **decorator** di Python (`@dataclass`).

## Jenis-Jenis Attribute yang Umum

### 1. Derive
Menghasilkan implementasi otomatis untuk trait standar.
```rust
#[derive(Debug, Clone, Copy, PartialEq, Eq, Hash)]
struct Point { x: i32, y: i32 }
```

### 2. Debugging dan Testing
```rust
#[test]
fn it_works() { assert_eq!(2 + 2, 4); }

#[should_panic]
fn test_panic() { panic!("harus panic"); }

#[ignore]
fn test_skip() { assert_eq!(1, 2); }

#[cfg(test)]
mod tests { /* hanya dikompilasi saat `cargo test` */ }
```

### 3. Conditional Compilation (cfg)
```rust
#[cfg(target_os = "linux")]
fn only_on_linux() {}

#[cfg(feature = "experimental")]
fn only_if_feature_enabled() {}

fn main() {
    if cfg!(debug_assertions) {
        println!("Running in debug mode");
    }
}
```

### 4. Allow / Deny / Warn / Forbid

Mengatur linter (rustc + clippy).
```rust
#[allow(dead_code)]
fn tidak_dipakai() {}

#[warn(unused_variables)]
fn peringatan() {}

#[deny(missing_docs)]
fn wajib_dok() {}
```

### 5. Inline dan Performance Hints
```rust
#[inline]
fn tambah(a: i32, b: i32) -> i32 { a + b }

#[inline(always)]
fn wajib_inline() -> i32 { 42 }

#[cold]
fn error_handler() {}

#[must_use]
fn hitung() -> i32 { 5 }
```

### 6. Documentation
```rust
/// Ini adalah doc comment untuk fungsi
#[doc = "Komentar dokumentasi alternatif"]
fn fungsi() {}
```

### 7. Macro Related
````rust
#[macro_export]
macro_rules! my_macro { () => { println!("Hi"); }; }

#[proc_macro]
pub fn my_proc_macro(input: TokenStream) -> TokenStream { ... }
````

### 8. Representation (repr)
Mengontrol layout struct/enum di memory.
```rust
#[repr(C)]
struct MyStruct { a: i32, b: u8 }

#[repr(u8)]
enum Status { Ok = 1, Err = 2 }
```

### 9. Panic, Safety, dan Lain-lain
```rust
#[track_caller]
fn log_error() {}

#[panic_handler]
fn my_panic(info: &PanicInfo) -> ! { loop {} }

#[no_mangle]
pub extern "C" fn exported() {}
```

### 10. Crate-Level Attributes
Ditulis dengan #![...], biasanya di lib.rs atau main.rs.
```rust
#![allow(unused_imports)]
#![deny(missing_docs)]
#![no_std]   // untuk embedded (tanpa std lib)
```

* NLL membuat borrow lebih fleksibel dan mencegah error borrow yang sebenarnya sudah tidak aktif.
* Memahami konsep ini penting untuk menulis Rust yang aman dan efisien.
