## Reliable

---

## First code

---

## Tasks queue processing

![task queue schema](assets/img/tasks_processing_schema.jpg)

---

## Task Modelisation

```rust
struct Task {
    id: i64,
    data: i32,
}

enum Operations {
    ADD,
    MUL,
    DIV
}

==> ( Task{ id: 1, data: 1}, Operations::ADD )
```

---

## Construct tasks queue

```rust

fn retrieve_tasks() -> Vec< (Task, Operations) > {
    
    vec![
        ( Task{ id: 1, data: 1}, Operations::ADD ),
        ( Task{ id: 2, data: 2}, Operations::MUL ),
        ( Task{ id: 3, data: 2}, Operations::MUL ),
        ( Task{ id: 4, data: 30}, Operations::DIV ),
    ]
}

```

---

## Pattern matching for operations

```rust
fn compute_operation(data: i32, operation: Operations)
    -> Result<String, OperationsError> {
    
    match operation {
        Operations::ADD => add_operation(data),
        
        Operations::MUL => mul_operation(data),
        
        Operations::DIV => div_operation(data),
    }
}
```

---

## Pattern matching for operations

```rust
fn add_operation(data: i32) -> Result<String, OperationsError> {
    let compute = data + 10;
    Ok(String::from(compute.to_string()))
}


fn mul_operation(data: i32) -> Result<String, OperationsError> {
    let compute = data * 10;
    Ok(String::from(compute.to_string()))
}


fn div_operation(data: i32) -> Result<String, OperationsError> {
    Err(OperationsError::OVERFLOW)
}
```

---

## Errors modelisation

```rust
enum OperationsError {
    UNKNOW_OPERATION,
    OVERFLOW,
    PARSING,
}
```

---

## main

```rust
fn main() {
  let tasks_queue = retrieve_tasks();

  tasks_queue.into_iter()
             .map(|(t, o)|   compute_operation(t.data, o))
             .map(|res|      res.unwrap_or(String::from("Error")))
             .for_each(|res| println!("Task give {}", res));
}
```

[full code](https://github.com/loganmzz/rust-presentation-introduction/blob/master/examples/src/bin/getting_startv3.rs)

---

## Structure

```rust
struct Task {
    id: i64,
    data: Data,
}

impl Task {
    fn new(id: i64, data: Data) -> Self {
        Self { id, data }
    }

    fn data(self) -> Data {
        self.data
    }
}
```

---

## Generic

```rust
struct Task<D> {
    id: i64,
    data: D,
}

impl <D>Task<D> {
    fn new(id: i64, data: D) -> Self {
        Self { id, data }
    }

    fn data(self) -> D {
        self.data
    }
}
```

NOTE:
no constructor pattern

---

#### Traits: Defining Shared Behavior

``` rust
pub trait Summary {
    fn summarize(&self) -> String;
}

pub struct Article {
    pub author: String,
    pub content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}", self.content)
    }
}

pub struct Tweet { ... }

impl Summary for Tweet {
    fn summarize(&self) -> String {
        ...
    }
}
```

---

### Implementing same trait multiple times

```rust
enum StateMachine {
    StateA{ data: i32 },
    StateB{ id: String},
}

struct EventB{ data: i32 }
struct EventA{ id: String}

impl From<EventB> for StateMachine {
    fn from(event: EventB) -> Self {
        StateMachine::StateA{ data: event.data }
    }
}

impl From<EventA> for StateMachine {
    fn from(event: EventA) -> Self {
        StateMachine::StateB{ id: event.id }
    }
}
```

NOTE:
No erasure

---

## Closure

```rust
let plus_one = |x: i32| { x + 1 };

plus_one(1);
```

---

## Higher-order function

```rust
fn call_with_one(some_closure: &Fn(i32) -> i32) -> i32 {
    some_closure(1)
}

call_with_one(&|x: i32| x + 1);
```

---

## Multi-paradigm language