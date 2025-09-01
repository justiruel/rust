
# 📘 Belajar Konsep Rust: Ownership, Borrowing, Stack/Heap, Async, dan Lainnya

Dokumen ini adalah rangkuman lengkap dari diskusi panjang mengenai konsep fundamental di Rust,
dibandingkan dengan bahasa lain seperti Node.js dan PHP.

---

## 1. Ownership & Borrowing

### Ownership
- Setiap nilai di Rust hanya boleh dimiliki **satu owner** pada satu waktu.
- Jika ownership berpindah (move), owner lama tidak bisa lagi mengakses datanya.

```rust
fn main() {
    let a = String::from("halo");
    let b = a; // ownership berpindah ke b

    // println!("{}", a); // ❌ error: a sudah tidak valid
    println!("{}", b); // ✅ b masih valid
}
```

### Borrowing
- Alih-alih memindahkan ownership, kita bisa **meminjam** data.
- Ada 2 jenis borrow:
  - **Immutable borrow** (`&T`) → bisa banyak sekaligus.
  - **Mutable borrow** (`&mut T`) → hanya boleh satu pada satu scope.

```rust
fn main() {
    let mut s = String::from("halo");

    let r1 = &s; // immutable borrow
    let r2 = &s; // immutable borrow lain
    println!("{} {}", r1, r2); // ✅ boleh banyak immutable

    let r3 = &mut s; // mutable borrow
    r3.push_str(" dunia");
    println!("{}", r3);
}
```

---

## 2. Stack vs Heap

- **Stack**: cepat, ukuran tetap, LIFO (last-in, first-out).
- **Heap**: fleksibel, bisa berubah ukuran, lebih lambat karena alokasi dinamis.

### Contoh
```rust
fn main() {
    let x = 5; // stack (ukuran fixed)
    let y = x; // copy, nilai di-duplicate di stack
    println!("x={}, y={}", x, y);

    let s = String::from("hello"); // pointer (stack) + data ("hello") (heap)
    println!("{}", s);
}
```

📌 Untuk `String` atau `Vec`:
- Stack menyimpan: pointer ke heap, `len`, `capacity`.
- Heap menyimpan: data aktual.

---

## 3. Copy, Clone, dan Move

- **Copy**: untuk tipe kecil (integer, bool, char) → duplikasi langsung di stack.
- **Clone**: membuat salinan baru di heap.
- **Move**: ownership berpindah, data lama tidak bisa diakses lagi.

```rust
fn main() {
    let x = 10; // Copy berlaku
    let y = x; 
    println!("x={}, y={}", x, y); // ✅

    let a = String::from("halo");
    let b = a; // Move berlaku
    // println!("{}", a); // ❌ tidak bisa
    println!("{}", b);

    let c = String::from("halo");
    let d = c.clone(); // Clone (heap baru dialokasikan)
    println!("c={}, d={}", c, d); // ✅
}
```

---

## 4. Borrowing: Mutable vs Immutable

Aturan:
- Banyak immutable borrow = ✅
- Satu mutable borrow = ✅
- Mutable + immutable di scope yang sama = ❌ (kecuali borrow lama sudah tidak dipakai lagi).

### Contoh khusus
```rust
fn main() {
    let mut s = String::from("halo");

    let r1 = &mut s;
    println!("{}", r1);
    r1.push_str("!!!");

    let r2 = &s; // ✅ boleh karena r1 tidak dipakai lagi setelah atas
    println!("{}", r2);
}
```

---

## 5. Garbage Collection vs Rust Ownership

### Garbage Collection (GC)
- Ada di Java, Go, PHP, JavaScript.
- Runtime memantau referensi objek, otomatis membebaskan memory yang tidak dipakai.
- Kelemahan: overhead & pause time (GC stop-the-world).

### Rust Ownership
- Memory otomatis dibebaskan saat owner keluar dari scope.
- Tidak ada GC, sehingga **lebih prediktif dan efisien**.

```rust
fn main() {
    {
        let s = String::from("halo");
        println!("{}", s);
    } // s keluar scope → heap dibebaskan otomatis
    println!("heap sudah dibersihkan");
}
```

---

## 6. Async di Node.js vs Rust

### Node.js
- Single-threaded event loop.
- Menggunakan `Promise`, `async/await`.
- Cocok untuk IO-bound tasks.

```javascript
async function fetchData() {
    return new Promise(resolve => setTimeout(() => resolve("done"), 1000));
}

async function main() {
    const result = await fetchData();
    console.log(result);
}

main();
```

### Rust
- Menggunakan `Future` + async runtime (Tokio, async-std).
- Perlu `.await` untuk mengeksekusi.
- Tidak ada GC → lebih efisien tapi lebih kompleks.

```rust
use tokio::time::{sleep, Duration};

async fn fetch_data() -> String {
    sleep(Duration::from_secs(1)).await;
    "done".to_string()
}

#[tokio::main]
async fn main() {
    let result = fetch_data().await;
    println!("{}", result);
}
```

---

## 7. Promise.all vs join!

### Node.js
```javascript
async function task1() { return "A"; }
async function task2() { return "B"; }

async function main() {
    const results = await Promise.all([task1(), task2()]);
    console.log(results); // ["A", "B"]
}
main();
```

### Rust
```rust
use tokio::join;

async fn task1() -> &'static str { "A" }
async fn task2() -> &'static str { "B" }

#[tokio::main]
async fn main() {
    let (a, b) = join!(task1(), task2());
    println!("{}, {}", a, b);
}
```

---

## 8. Unsafe Rust

- Memungkinkan bypass aturan borrow checker.
- Berguna untuk optimisasi tingkat rendah (akses pointer, interoperabilitas dengan C).

```rust
fn main() {
    let x: i32 = 42;
    let r: *const i32 = &x;

    unsafe {
        println!("Nilai x: {}", *r); // raw pointer dereference
    }
}
```

---

## 9. Kesimpulan

- **Ownership** → Rust tidak butuh GC, memory aman otomatis.
- **Borrowing** → memungkinkan peminjaman data tanpa cloning, dengan aturan ketat.
- **Stack vs Heap** → data kecil disimpan di stack, data besar di heap (pointer disimpan di stack).
- **Copy vs Clone vs Move** → Copy untuk tipe kecil, Clone untuk duplikasi heap, Move untuk transfer ownership.
- **Async** → Node.js lebih sederhana (Promise), Rust lebih efisien (Future) tapi lebih kompleks.
- **Unsafe** → dipakai hanya jika sangat perlu (misalnya optimisasi low-level).
