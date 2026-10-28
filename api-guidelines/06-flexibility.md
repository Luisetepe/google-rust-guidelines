# Flexibility

> **Source:** [Rust API Guidelines — Flexibility](https://rust-lang.github.io/api-guidelines/flexibility.html) · **License:** MIT OR Apache-2.0
>
> Guidelines that let a crate support diverse real-world use cases: exposing intermediate results, leaving data placement to the caller, generic parameters, and object-safe traits.

<a id="c-intermediate"></a>

## C-INTERMEDIATE — Functions expose intermediate results to avoid duplicate work

**Rule:** When a function that answers a question also computes related data that is potentially of interest to the client, you **SHOULD** consider exposing that data in the API.

**Details:**
- Std precedents:
  - [`Vec::binary_search`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.binary_search) returns neither a `bool` nor an `Option<usize>`; it returns the index if found, and otherwise the index at which the value would need to be inserted.
  - [`String::from_utf8`](https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8) can fail on non-UTF-8 input; its error exposes the byte offset up to which the input was valid UTF-8 and hands back ownership of the input bytes.
  - [`HashMap::insert`](https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.insert) returns an `Option<T>` with the preexisting value for the key, if any, saving users who want that value a second hash table lookup.

**Rationale:**
- Avoids duplicate work by the client.

<a id="c-caller-control"></a>

## C-CALLER-CONTROL — Caller decides where to copy and place data

**Rule:** A function that requires ownership of an argument **SHOULD** take it by value rather than borrowing and cloning it. A function that does not require ownership **SHOULD** take a shared or exclusive borrow rather than taking ownership and dropping the argument.

**Details:**
- The `Copy` trait **SHOULD** be used as a bound only when absolutely needed, not as a way of signaling that copies should be cheap to make.

**Google note:** Google adds: use `.clone()` only for genuine deep copies, not as a substitute for borrowing ([G-OWN-02](../google/04-ownership-and-borrowing.md#g-own-02)); borrowed immutable parameters use the most flexible reference type (`&T`, `&[T]`, `&str`, `Option<&T>`) ([G-FIO-01](../google/05-function-inputs-and-outputs.md#g-fio-01)).

**Example:**
```rust
// Good: requires ownership, takes it
fn foo(b: Bar) {
    /* use b as owned, directly */
}

// Bad: borrows, then clones
fn foo(b: &Bar) {
    let b = b.clone();
    /* use b as owned after cloning */
}
```

```rust
// Good: does not require ownership, borrows
fn foo(b: &Bar) {
    /* use b as borrowed */
}

// Bad: takes ownership it doesn't need
fn foo(b: Bar) {
    /* use b as borrowed, it is implicitly dropped before function returns */
}
```

<a id="c-generic"></a>

## C-GENERIC — Functions minimize assumptions about parameters by using generics

**Rule:** Functions **SHOULD** use generics to pinpoint exactly the assumptions they need about their arguments; e.g. a function that only iterates over data should take `I: IntoIterator<Item = T>` rather than `&[T]`, `&Vec<T>`, or a specific collection.

**Details:**
- The fewer assumptions a function makes about its inputs, the more widely usable it is.
- Advantages of generics:
  - *Reusability*: applies to an open-ended set of types, with a clear contract for the functionality they must provide.
  - *Static dispatch and optimization*: each use is specialized ("monomorphized"), so (1) trait method calls are static, direct calls and (2) the compiler can inline and otherwise optimize them.
  - *Inline layout*: a `struct`/`enum` generic over `T` lays out values of `T` inline, without indirection.
  - *Inference*: type parameters can usually be inferred, cutting verbosity where explicit conversions or other method calls would otherwise be needed.
  - *Precise types*: generics name the specific implementing type, so you can be precise where that exact type is required or produced; `fn binary<T: Trait>(x: T, y: T) -> T` consumes and produces exactly the same `T` and cannot be called with two different types that both implement `Trait`.
- Disadvantages of generics:
  - *Code size*: specialization duplicates the function body; weigh this against the performance benefits of static dispatch.
  - *Homogeneous types*: `T` stands for a *single* concrete type (e.g. a `Vec<T>` holds elements of one type, laid out inline). When heterogeneous collections are needed, see trait objects ([C-OBJECT](#c-object)).
  - *Signature verbosity*: heavy use of generics can make signatures harder to read and understand.
- Std precedent: [`std::fs::File::open`](https://doc.rust-lang.org/std/fs/struct.File.html#method.open) takes a generic `AsRef<Path>`, so files can be opened from a string literal `"f.txt"`, a [`Path`](https://doc.rust-lang.org/std/path/struct.Path.html), an [`OsString`](https://doc.rust-lang.org/std/ffi/struct.OsString.html), and a few other types.

**Google note:** Google scopes this by code type: highly reusable library code should accept iterators/`impl IntoIterator`, while application code may accept `&[T]` when simpler ([G-FIO-03](../google/05-function-inputs-and-outputs.md#g-fio-03)); `impl Into<T>` only when call-site ergonomics are paramount and almost never in application code ([G-FIO-04](../google/05-function-inputs-and-outputs.md#g-fio-04)); libraries prefer generics over `dyn Trait` except where code size matters ([G-TRT-04](../google/07-traits.md#g-trt-04)).

**Example:**
```rust
// Good: the function only needs to iterate
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }

// Bad: assumes a specific representation
fn foo(c: &[i64]) { /* ... */ }
fn foo(c: &Vec<i64>) { /* ... */ }
fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }
```

```rust
// Precise types: x, y and the result are all exactly the same T
fn binary<T: Trait>(x: T, y: T) -> T
```

**See also:** [G-FIO-01](../google/05-function-inputs-and-outputs.md#g-fio-01)

<a id="c-object"></a>

## C-OBJECT — Traits are object-safe if they may be useful as a trait object

**Rule:** When designing a trait, you **MUST** decide early whether it will be used as an object or as a generic bound. A trait meant to be used as an object **SHOULD** have methods that take and return trait objects rather than use generics.

**Details:**
- Trait objects have significant limitations: methods invoked through a trait object cannot use generics and cannot use `Self` except in receiver position.
- A `where Self: Sized` clause **MAY** be used to exclude specific methods from the trait object; adding it to a generic method makes an otherwise non-object-safe trait object-safe.
- Advantages of trait objects:
  - *Heterogeneity*: when you need it, you really need it.
  - *Code size*: no specialized (monomorphized) copies of code, which can greatly reduce code size.
- Disadvantages of trait objects:
  - *No generic methods*: trait objects cannot currently provide generic methods.
  - *Dynamic dispatch and fat pointers*: inherent indirection and vtable dispatch, which can carry a performance penalty.
  - *No `Self`*: except for the receiver argument, methods on trait objects cannot use `Self`.
- Std precedents:
  - [`io::Read`](https://doc.rust-lang.org/std/io/trait.Read.html) and [`io::Write`](https://doc.rust-lang.org/std/io/trait.Write.html) are often used as objects.
  - [`Iterator`](https://doc.rust-lang.org/std/iter/trait.Iterator.html) marks several generic methods `where Self: Sized` to remain usable as an object.

**Google note:** Google (which calls object safety "`dyn` compatibility") tells libraries to prefer generics over `dyn Trait` parameters except where code size matters ([G-TRT-04](../google/07-traits.md#g-trt-04)), and to define traits only for generics, `impl Trait`/`dyn Trait`, or extension traits ([G-TRT-05](../google/07-traits.md#g-trt-05)).

**Example:**
```rust
// Not object-safe, due to the generic method
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T);
}

// Object-safe: `Self: Sized` excludes the generic method from the trait object
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T) where Self: Sized;
}
```

**See also:** [C-GENERIC](#c-generic), [`dyn` compatibility](../references/language-concepts.md#dyn-compatibility)
