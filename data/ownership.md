
<!-- .slide: data-background="assets/img/safety.gif" -->

---

<p><span>Mutable data</span><!-- .element: class="fragment" -->  +  <span>Many ref on it</span><!-- .element: class="fragment" --> = <span> No safety</span><!-- .element: class="fragment" --></p> 

---

## JVM

_Can't prevent_ data races, iterator invalidation, ...

---

## Java _Runtime errors_
```
Exception in thread "main" java.lang.NullPointerException

... 10ms after

Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException
```

## C/C++ _Runtime errors_

```
$ ./client_demo

Segmentation fault
```

---

<!-- .slide: data-background="assets/img/debuger_rescue.gif" -->


---

<!-- .slide: data-background="assets/img/dont_want.gif" -->

---

## Rust solution

Rust _avoids_ the need for _GC_ through:

* ownership 
* borrowing

**Check at compile time.**
<!-- .element: class="fragment" --> 

---

## Ownership

Ensure **only one** active _binding_ to _allocated memory at a time_

_✔️ eliminates double frees, use after free..._  <!-- .element: class="fragment" -->

---

## Ownership

<pre><code data-trim data-noescape class="rust"> 
fn generate_events() {
    let <span class="fragment highlight-mark">events</span> = vec![...];
    transform_events(events); //take events ownership
}

fn transform_events(<span class="fragment highlight-mark">events</span>: Vec<Event>) {
    // Ownership of events transfered in
    // transform_event scope 
} <span class="fragment strong">◀️ free memory allocation events</span>
</code></pre> 

---

## Ownership

<pre><code data-trim data-noescape class="rust"> 
fn generate_events() {
    let events = vec![...];

    transform_events(events);
    <span class="fragment highlight-mark">transform_events(events);</span> 
}

fn transform_events(events: Vec<Event>) { /* ... */ }
</code></pre>

<pre><code data-trim data-noescape class="rust"> 
error[E0382]: use of moved value: `events`
 --> src/move_example.rs:4:19
  |
3 |   transform_events(events);
  |                   <span class="fragment highlight-mark">------ value moved here</span>
4 |   transform_events(events);
  |                   <span class="fragment highlight-mark">^^^^^^ value used here after move</span>
</code></pre> 
<!-- .element class="fragment" -->


---

## Borrowing

> What if, we **borrow** the resource instead?

---

## Borrowing *&T*

Many reader, no writers

<pre><code data-trim data-noescape class="rust">
fn read_events() {
    let treasures = get_treasures(...); // take ownership

    get_value_treasures(<span class="fragment highlight-mark">&treasures</span>); //reader
    get_date_treasures(<span class="fragment highlight-mark">&treasures</span>); //reader
}

fn get_value_treasures(treasures: <span class="fragment highlight-mark">&</span>Vec<Treasure>) {
    // Can't modify the treasures here
}
</code></pre>

---

## Borrowing &mut T

Exactly _one mutable_ reference

<pre><code data-trim data-noescape class="rust">
let mut treasures = get_treasures(...); // take ownership

let t1 = <span class="fragment highlight-mark">&mut</span> treasures;
modify_values_treasures(t1);

let t2 = <span class="fragment highlight-mark">&mut</span> treasures;
modify_owner_treasures(t2);

fn get_value_treasures(treasures: <span class="fragment highlight-mark">&mut</span> Vec<Treasures>) {
    // Can mutate treasures
}
</code></pre>

---

## Borrowing &mut T

<pre><code data-trim data-noescape class="rust">
error[E0499]: cannot borrow `treasures` as mutable more than 
once at a time
 --> src/main.rs:7:19
  |
4 |     let t1 = &mut treasures;
  |                  <span class="fragment highlight-mark">--------- first mutable borrow occurs here</span> 
  |                              
...
7 |     let t2 = &mut treasures; 
  |                   <span class="fragment highlight-mark">^^^^^^^^^ second mutable borrow occurs here</span> 
  |                             
8 |     get_value_treasures(t2);
9 | }
  | - first borrow ends here
</code></pre>

---

## Borrowing look like

> Compile time _read-write lock_ on data (not the code)

---

<p>It's more <span style="color:orange;"> safe </span>at runtime !! </p><!-- .element class="big" -->

---

## Everything is done by the Borrow Checker

[src/librustc_borrowck/borrowck](https://github.com/rust-lang/rust/tree/master/src/librustc_borrowck/borrowck)

1. Set *formal rules*
2. `gather_loans`
3. `check_loans`

---

### Formal rules

```
PREDICATE(X, Y, Z)
  Condition 1
  Condition 2
  Condition 3
```

---

Example: Checking mutability of variables

``` prolog
MUTABILITY(X, MQ)                   // M-Var-Mut
  DECL(X) = mut

MUTABILITY(X, imm)                  // M-Var-Imm
  DECL(X) = imm

MUTABILITY(P.f, MQ)                 // M-Field
  MUTABILITY(P, MQ)
```

---

```rust
fn check_mutability<'a, 'tcx>(bccx: &BorrowckCtxt<'a, 'tcx>,
                              borrow_span: Span,
                              cause: AliasableViolationKind,
                              cmt: &mc::cmt_<'tcx>,
                              req_kind: ty::BorrowKind)
                              -> Result<(),()> {
     match req_kind {
        ty::UniqueImmBorrow | ty::ImmBorrow => {
            match cmt.mutbl {
                 mc::McImmutable | mc::McDeclared | mc::McInherited => {
                     Ok(())
                }
            }
        }

        ty::MutBorrow => {
            // Only mutable data can be lent as mutable.
            if !cmt.mutbl.is_mutable() {
                Err(bccx.report(BckError { span: borrow_span,
                                           cause,
                                           cmt,
                                           code: err_mutbl }))
            } else {
                Ok(())
            }
        }
    }
}
```

[check_mutability](https://github.com/rust-lang/rust/blob/e471c206cf472b54acee83a231560e16c439ab63/src/librustc_borrowck/borrowck/gather_loans/mod.rs#L206)