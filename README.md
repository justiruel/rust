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
Garbage Collection (misalnya di Java, Go, JavaScript):
Runtime ada GC (Garbage Collector) yang memantau memory.
GC akan pause program sesekali untuk membersihkan data yang tidak direferensikan.
Overhead runtime lebih besar, ada kemungkinan stop-the-world pause.
Rust (Ownership + Borrowing):
Tidak ada GC.
Memory dikelola di compile time lewat aturan ownership.
Heap otomatis dibebaskan ketika owner keluar scope.
Tidak ada pause runtime → performa konsisten.
Analogi:
GC = ada petugas kebersihan yang patroli membersihkan sampah, kadang mengganggu aktivitas.
Rust = setiap orang wajib membersihkan sampahnya sendiri sebelum keluar ruangan.

## 12. Perbandingan Async: Node.js vs Rust
Node.js (Promise / async-await):
Single-threaded event loop.
Async dikerjakan lewat callback queue (task diparkir, dijalankan ketika I/O selesai).
Memory manajemen ditangani GC (developer tidak mengurus lifetime).
Contoh Node.js:
```rust
async function run() {
    let data = await fetch("/api");
    console.log(data);
}
```
Rust (async / Future):
async fn menghasilkan Future.
Future adalah state machine, berjalan hanya jika dijalankan executor.
Memory tetap aman karena ownership & borrow checker.

Contoh Rust:
```rust
async fn run() {
    let data = fetch_data().await;
    println!("{}", data);
}
```
Analogi:
Node.js: ada sekretaris (event loop) yang mencatat janji, memanggil balik kalau waktunya tiba.
Rust: janji (Future) adalah mesin kecil yang berhenti sementara, lalu dilanjutkan oleh eksekutor saat siap.
