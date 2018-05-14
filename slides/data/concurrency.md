## Fearless Concurrency with Rust

> Rust doesn’t like data-races

---

## How do you make concurrency _painless_ ?

---

### Same tools that make Rust safe !

In Rust, threads are _isolated_ from each other automatically, due to _ownership_.

NOTE:
Writes can only happen when the thread has mutable access, either by owning the data, or by having a mutable borrow of it.
Either way, the thread is guaranteed to be the only one with access at the time. To see how this plays out, let’s look at locks.

---

Example from [Data Race Detector](https://golang.org/doc/articles/race_detector.html)

``` go
func main() {
    c := make(chan bool)
    m := make(map[string]string)

    go func() {
       m["1"] = "a" // First conflicting access.
       c <- true
    }()

    m["2"] = "b" // Second conflicting access.
    <-c

    for k, v := range m {
       fmt.Println(k, v)
    }
}
```

NOTE:
Here is an example of a data race that can lead to crashes and memory corruption:

---

> Go will happily compile and run the code above

---

## Rust version

```rust
fn main() {
    let (tx, rx) = channel();
    let mut m = HashMap::new();

    thread::spawn(move || {
        m.insert("1", "a");
        tx.send(true);
    });

    m.insert("2", "b");
    rx.recv();

    for (k, v) in &m {
        println!("{}, {}", k, v);
    }
}
```

NOTE:
We can force our closure to take ownership of its environment with the move keyword:

---

> Rustc will refuse to compile this shit 🎉

<pre><code data-trim data-noescape class=""> 
error[E0382]: use of moved value: \`m\`
  --> src/main.rs:16:5
   |
11 |     thread::spawn(move || {
   |                   ------- <span class="fragment highlight-mark">value moved (into closure) here</span>
...
16 |     m.insert("2", "b");
   |     ^ <span class="fragment highlight-mark">value used here after move</span>
   |
   = note: move occurs because `m` has type `std::collections::HashMap<&'static str, &'static str>`, which does not implement the `Copy` trait
```
</code></pre>

> Disaster averted.

---

> Every _data type_ knows whether it can safely be sent between or accessed by multiple threads

- A type is _Send_ if it is safe to _send_ it to another thread.
- A type is _Sync_ if it is safe to _share_ between threads (_&T_ is _Send_).

---

## Message passing

> Do not communicate by sharing memory; instead, share memory by communicating.

> \- Effective Go

---

## Channel API

```rust
fn send<T: Send>(chan: &Channel<T>, t: T);
        ^^^^^^^                    ^^^^^^^


fn recv<T: Send>(chan: &Channel<T>) -> T;
        ^^^^^^^
```

> _T_ must be considered _safe_ to send between threads

NOTE:
Take ownership + Send
Actor

---

## [Channel](https://doc.rust-lang.org/std/sync/mpsc/fn.channel.html): MPSC

```rust
let (tx1, rx) = mpsc::channel();
let tx2 = tx1.clone();

thread::spawn(move || tx1.send(vec![1, 2]));

thread::spawn(move || tx2.send(vec![3, 4]));

for scores in rx {
    println!("Received : {:?}", scores);
}
```

[examples_channel.rs](https://github.com/loganmzz/rust-presentation-introduction/blob/master/examples/src/bin/examples_channel.rs)

![go_die](assets/img/gopher_ahah.png)
<!-- .element class="fragment fade-up" -->

<!-- .element style="margin-top: 30px" -->

---

### Ownership messaging (safety)

<pre><code data-trim data-noescape class="rust">
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let mut scores = vec![2, 4];
    tx.send(scores);

    <span class="fragment highlight-mark">scores.push(125);</span>
});
</code></pre>

<pre><code data-trim data-noescape class="rust"> 
error[E0382]: use of moved value: `scores`
  --> main.rs:13:3
12 | 		tx.send(scores);
   | 		        <span class="fragment highlight-mark">- value moved here</span>
13 | 		scores.push(125);
   | 		<span class="fragment highlight-mark">^ value used here after move</span>
</code></pre>
<!-- .element class="fragment" -->

> Disaster averted <!-- .element class="fragment" -->

---

## Shared memory in Rust is opt-in

> "Shared-state concurrency is nevertheless a fundamental programming style, needed for systems code, for maximal performance, and for implementing other styles of concurrency."

---

## Lock on Data

- `Mutex`: A mutual exclusion primitive for protecting shared data

- `Arc`: Atomic Reference Count (Arc is `Send`)

- `RefCell`, `Cell`, ...

---

### Rust’s approach to concurrency:

- Memory safety without garbage collection.

- Concurrency without data races.

- The _compiler_ prevents all data races.