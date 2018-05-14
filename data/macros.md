## Macros

<table class="no-border center-mid">
<tr class="fragment"> <td>**functions**</td> <td>abstract over</td> <td>_variables_</td> </tr>
<tr class="fragment"> <td>**generics**</td> <td>abstract over</td> <td>_types_</td>
<tr class="fragment"> <td>**macros**</td> <td>abstract over</td> <td>_syntax tree_</td>
</table>

---

## **A**bstract **S**yntax **T**ree

```rust
macro_rules! new_macro {

    ($e: expr) => (println!("res = {}", $e));

}

new_macro!(2 + 2)
```

Note:
* AST-based ⇒ both call and definition must be parsable 

---

## Access AST 

| | |
| --- | --- |
| `block` | |
| `expr` | ⇒ expressions
| `ident` | ⇒ identifiers (variable/function names)
| `item` | ⇒ component of a crate (i.e. global definitions)
| `pat` | ⇒ pattern
| `path` | ⇒ (e.g. `::std::fmt`)
| `stmt` | ⇒ statement
| `tt` | ⇒ token tree
| `ty` | ⇒ type
| `meta` | ⇒ attribute content (i.e. `#[...]`)
<!-- .element class="headless compact" -->

---

## _vec!_ macro

```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

====> vec![1, 2 , 3]
```