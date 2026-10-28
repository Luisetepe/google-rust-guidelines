# Comprehensive Rust — Naming Conventions

> **Source:** [Comprehensive Rust — Naming Conventions](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions.html) (and its sub-pages) · **License:** CC-BY-4.0 for text (per the SPDX header of each page's source); embedded code samples Apache-2.0
>
> The common prefix/suffix components of Rust method names, with standard-library examples, as taught in the "Idiomatic Rust → Foundations of API design → Predictable API" section of Comprehensive Rust.

The Google Rust Style Guide cites this section from [G-NAM-01 — Naming methods](../google/10-naming.md#g-nam-01). A formal, consistently applied naming convention lets developers treat names like a domain-specific language and quickly understand a method's functionality and use cases; Rust's community developed its conventions early, so they are mostly consistent (e.g. in the standard library). Code blocks below are signature sketches (some simplified), not compilable code.

Sub-pages covered, in course order: [`new`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/new.html), [`is`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/is.html), [`mut`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/mut.html), [`with` constructor](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/with-constructor.html), [`with` copy-setter](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/with-copy-setter.html), [`with` closure](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/with-closure.html), [`try`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/try.html), [`from`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/from.html), [`into`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/into.html), [`to`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/to.html), [`as_`/`_ref`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/as-and-ref.html), [`by`](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/by.html), [exercise](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions/exercise.html).

<a id="cr-nam-01"></a>

## CR-NAM-01 — `new`: constructor functions

**Rule:** `new` (as a whole method name or as a prefix) **SHOULD** name the "default" constructor function for a type.

**Details:**
- Rust has no `new` keyword; values are created only by calling functions or populating them directly. `new` is purely conventional and has no special syntactic meaning.
- `new` is sometimes a prefix, and sometimes takes arguments.

**Example:**
```rust
impl<T> Vec<T> {
    fn new() -> Vec<T>;
}

impl<T> Box<T> {
    fn new(T) -> Box<T>;
}
```

**See also:** [G-NAM-05](../google/10-naming.md#g-nam-05), [C-CTOR](../api-guidelines/05-predictability.md#c-ctor)

<a id="cr-nam-02"></a>

## CR-NAM-02 — `is_[condition]`: boolean check

**Rule:** A method that checks a boolean condition about a value **SHOULD** be named `is_[condition]`; **SHOULD NOT** put `not` in the name (no `is_not_` methods) — callers write `!value.is_[condition]()` instead.

**Details:**
- There are no `is_not_` methods in the standard library.

**Example:**
```rust
impl<T> Vec<T> {
    fn is_empty(&self) -> bool;
}

impl f32 {
    fn is_nan(self) -> bool;
}

impl u32 {
    fn is_power_of_two(self) -> bool;
}
```

<a id="cr-nam-03"></a>

## CR-NAM-03 — `[method]_mut`: mutable reference access

**Rule:** An access-style method that gives access to a mutable reference **SHOULD** use the `_mut` suffix, paired with an immutable version that gets the shorter name.

**Details:**
- The `_mut` method requires mutable access to the value it is called on.
- Rust can't abstract over mutability, so one method cannot serve both mutable and immutable use; write pairs of functions instead.

**Example:**
```rust
impl<T> Vec<T> {
    fn get(&self, index: usize) -> Option<&T>;
    fn get_mut(&mut self, index: usize) -> Option<&mut T>;
}

impl<T> [T] {
    fn iter(&self) -> impl Iterator<Item = &T>;
    fn iter_mut(&mut self) -> impl Iterator<Item = &mut T>;
}
```

**See also:** [C-GETTER](../api-guidelines/01-naming.md#c-getter), [C-ITER](../api-guidelines/01-naming.md#c-iter)

<a id="cr-nam-04"></a>

## CR-NAM-04 — `with` as constructor

**Rule:** A constructor that sets one value of a type while using default values for the rest **MAY** use the `with_` prefix, read as "`<Type>` with specific setting".

**Details:**
- Most common when initializing heap memory for container types.
- Distinct from `new` constructors because it specifies a value for something API users usually don't care about.

**Rationale:**
- `Vec::with_capacity` scans well as creating a "Vec with capacity"; `Vec::new_capacity` or `Vec::from_capacity` do not communicate what is going on.

**Example:**
```rust
impl<T> Vec<T> {
    // Initializes memory for at least N elements, len is still 0.
    fn with_capacity(capacity: usize) -> Vec<T>;
}
```

<a id="cr-nam-05"></a>

## CR-NAM-05 — `with` as copy-and-set

**Rule:** A method that copies a value but changes a specific part of it **MAY** use the `with_` prefix, read as "like `<value>`, but with something different".

**Details:**
- The original value is unchanged: `with_extension` copies the data of a `&Path` into a new `PathBuf` with a different extension.

**Example:**
```rust
impl Path {
    // Simplified. "/home/me/mortgage.pdf".with_extension("mov") =>
    // "/home/me/mortgage.mov"
    fn with_extension(&self, ext: &OsStr) -> PathBuf;
}
```

<a id="cr-nam-06"></a>

## CR-NAM-06 — `with`: working with closures

**Rule:** A method that takes a function or closure to use instead of a "sensible default" computation **MAY** use the `_with` suffix, read as "do X, but with this specific way of computing things".

**Details:**
- Similar to `by` ([CR-NAM-12](#cr-nam-12)).

**Example:**
```rust
impl<T> Vec<T> {
    // Simplified. If the resize is larger than the current vec size, use the
    // closure to populate elements.
    pub fn resize_with(&mut self, new_len: usize, f: impl FnMut() -> T);
}

mod iter {
    // Create an infinite, lazy iterator using a closure.
    pub fn repeat_with<A, F: FnMut() -> A>(repeater: F) -> RepeatWith<F>;
}
```

<a id="cr-nam-07"></a>

## CR-NAM-07 — `try_[method]`: fallible methods with specific errors

**Rule:** A method that can fail and returns a `Result` **SHOULD** use the `try_` prefix.

**Details:**
- `TryFrom` is a `From`-like trait for types whose single-value constructors might fail.

**Exceptions:**
- Methods that return a reference to an existing value with only one failure mode (e.g. "index out of bounds" for `Vec::get`, "key does not exist" for `HashMap::get`) are named `get` and return `Option`, not `Result`; they are not `try_get`.

**Example:**
```rust
impl TryFrom<i32> for u32 {
    type Error = TryFromIntError;

    // Return type as written in the source; the actual std signature returns Result<u32, TryFromIntError>.
    fn try_from(value: i32) -> Result<i64, TryFromIntError>;
}

impl<T> Receiver<T> {
    fn try_recv(&self) -> Result<T, TryRecvError>;
}
```

**See also:** [G-NAM-05](../google/10-naming.md#g-nam-05)

<a id="cr-nam-08"></a>

## CR-NAM-08 — `from`: conversion constructors

**Rule:** The `from_` prefix **SHOULD** be used for constructor-style, `From`-trait-style functions that strongly imply a type conversion; `new` **SHOULD** still be preferred for most constructor-style functions.

**Details:**
- `from_` functions may take multiple arguments, but usually imply the user does more of the work than a usual constructor would.
- The implication of `from` is transformation of one data type to another.

**Example:**
```rust
impl Duration {
    fn from_days(days: u64) -> Duration;
}

impl i32 {
    fn from_ascii(src: &[u8]) -> Result<i32, ParseIntError>;
}

impl u32 {
    fn from_le_bytes(bytes: [u8; 4]) -> u32;
}
```

**See also:** [G-NAM-05](../google/10-naming.md#g-nam-05), [`From` conversions](../references/language-concepts.md#from-conversions)

<a id="cr-nam-09"></a>

## CR-NAM-09 — `into`: consuming conversion

**Rule:** A method that consumes `self` and returns an owned value of another type **SHOULD** use the `into_` prefix.

**Details:**
- Not a reinterpret cast: the data can be rearranged, reallocated or changed in any way, including losing information.
- `into_iter` consumes a collection (e.g. a vec, btreeset or hashmap) and yields an iterator over owned values, unlike `iter`/`iter_mut`, which yield references.

**Example:**
```rust
pub trait IntoIterator {
    fn into_iter(self) -> Self::IntoIter;
}

impl str {
    fn into_string(self: Box<str>) -> String;
}
```

**See also:** [C-CONV](../api-guidelines/01-naming.md#c-conv), [G-NAM-01](../google/10-naming.md#g-nam-01)

<a id="cr-nam-10"></a>

## CR-NAM-10 — `to`: non-consuming conversion

**Rule:** A method that creates a new owned value without consuming `self`, and implies a type conversion, **SHOULD** use the `to_` prefix.

**Details:**
- `to_` methods return a different type and strongly imply a non-trivial type conversion or even a data transformation (e.g. `str::to_uppercase`).
- They most commonly take `&self`, but **MAY** take `self` by value if the type implements `Copy` (the call still does not consume the original).
- Also seen in functions that convert the endianness of primitives, or copy and expose the value of a newtype.
- `to_owned` appears on reference values like `&str`; `into_owned` appears on owned values that hold borrowed references, like `Cow`, where the owned `Cow` is consumed to produce an owned value of the referenced type.

**Exceptions:**
- To take `&self` and return an owned value of the *same* type, implement [`Clone`](https://doc.rust-lang.org/stable/std/clone/trait.Clone.html) or [`ToOwned`](https://doc.rust-lang.org/stable/std/borrow/trait.ToOwned.html) instead.
- To consume the source value, use the `into_` pattern ([CR-NAM-09](#cr-nam-09)).

**Example:**
```rust
impl str {
    fn to_owned(&self) -> String;

    fn to_uppercase(&self) -> String;
}

impl u32 {
    // Take an owned self because `u32` implements `Copy`
    fn to_be(self) -> u32;
}
```

**See also:** [C-CONV](../api-guidelines/01-naming.md#c-conv), [G-NAM-01](../google/10-naming.md#g-nam-01)

<a id="cr-nam-11"></a>

## CR-NAM-11 — `as_` and `_ref`: reference conversions

**Rule:** A method that returns a borrow of a type's primary piece of contained data **SHOULD** use the `as_` prefix; to distinguish two reference getters, use the `_ref` suffix.

**Details:**
- The return value is most often a reference that borrows `self`.
- The borrow may be only logical: `as_ptr()` returns a raw pointer, whose borrowing the borrow checker does not track.
- The type should contain one primary piece of data being borrowed out; the `as` convention does not work for an aggregate of many fields with no obvious primary one.

**Example:**
```rust
impl<T> Rc<T> {
    // Very common on container types, see how it's also on Option.
    fn as_ref(&self) -> &T;

    fn as_ptr(&self) -> *const T;
}

impl<T> Option<T> {
    fn as_ref(&self) -> Option<&T>;

    fn as_slice(&self) -> &[T];
}
```

**See also:** [C-CONV](../api-guidelines/01-naming.md#c-conv), [G-NAM-01](../google/10-naming.md#g-nam-01)

<a id="cr-nam-12"></a>

## CR-NAM-12 — `by`: custom comparator or projection

**Rule:** A method that takes a custom comparison or projection function **SHOULD** use the `by` component (`_by`, `_by_key`).

**Details:**
- `sort_by` takes a comparator that replaces the normal `Ord` comparison logic.
- `sort_by_key` takes a projection function mapping each element to an alternate value used for sorting (e.g. to sort by a particular struct field).

**Exceptions:**
- Sometimes "by" is simply a preposition, e.g. [`Read::by_ref()`](https://doc.rust-lang.org/std/io/trait.Read.html#method.by_ref) and the nightly-only [`Iterator::advance_by()`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.advance_by).

**Example:**
```rust
impl<T> [T] {
    fn sort(&mut self) where T: Ord;

    fn sort_by(&mut self, compare: impl FnMut(&T, &T) -> Ordering);

    fn sort_by_key<K, F>(&mut self, f: F)
    where
        F: FnMut(&T) -> K,
        K: Ord;
}
```

## Exercise (non-normative)

The course closes the section with an exercise: (1) what do these names imply they do; (2) what should these signatures be named.

```rust
// What are the types of these methods?
Option::is_some // ?
slice::get // ?
slice::get_unchecked_mut // ?
Option::as_ref // ?
str::from_utf8_unchecked_mut // ?
Rc::get_mut // ?
Vec::dedup_by_key // ?

// What should we name methods with these types?
fn ____(String) -> Self;
fn ____(&self) -> Option<&InnerType>; // details for InnerType do not matter.
fn ____(self, String) -> Self;
fn ____(&mut self) -> Option<&mut InnerType>;
```

Answers given by the course, types:
- `Option::is_some(&self) -> bool`
- `slice::get(&self /* &[T] */, usize) -> Option<&T>`
- `slice::get_unchecked_mut(&self /* &[T] */, usize) -> &T` (unsafe and simplified; as written in the source — the actual std method takes `&mut self` and returns `&mut T`)
- `Option::as_ref(&self /* &Option<T> */) -> Option<&T>`
- `str::from_utf8_unchecked_mut(v: &mut [u8]) -> &mut str` (unsafe)
- `Rc::get_mut(&mut self /* &mut Rc<T> */) -> Option<&mut T>` (simplified)
- `Vec::dedup_by_key<K: PartialEq>(&mut self /* &mut Vec<T> */, key: impl FnMut(&mut T) -> K)` (simplified)

Answers given by the course, names:
- `fn from_string(String) -> Self`
- `fn inner(&self) -> Option<&InnerType>` or `as_ref`, depending on context
- `fn with_string(self, String) -> Self`
- `fn inner_mut(&mut self) -> Option<&mut InnerType>` or `as_ref_mut`, depending on context
