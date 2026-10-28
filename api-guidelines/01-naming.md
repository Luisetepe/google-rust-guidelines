# Naming

> **Source:** [Rust API Guidelines — Naming](https://rust-lang.github.io/api-guidelines/naming.html) · **License:** MIT OR Apache-2.0
>
> How a crate aligns with Rust naming conventions: casing, conversion and getter method names, iterator methods and types, Cargo feature names, and word order.

<a id="c-case"></a>

## C-CASE — Casing conforms to RFC 430

**Rule:** Names **SHOULD** follow the basic Rust naming conventions of [RFC 430](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md): `UpperCamelCase` for "type-level" constructs (types and traits), `snake_case` for "value-level" constructs.

**Details:**

| Item | Convention |
| ---- | ---------- |
| Crates | [unclear](https://github.com/rust-lang/api-guidelines/issues/29) |
| Modules | `snake_case` |
| Types | `UpperCamelCase` |
| Traits | `UpperCamelCase` |
| Enum variants | `UpperCamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Macros | `snake_case!` |
| Local variables | `snake_case` |
| Statics | `SCREAMING_SNAKE_CASE` |
| Constants | `SCREAMING_SNAKE_CASE` |
| Type parameters | concise `UpperCamelCase`, usually single uppercase letter: `T` |
| Lifetimes | short `lowercase`, usually a single letter: `'a`, `'de`, `'src` |
| Features | [unclear](https://github.com/rust-lang/api-guidelines/issues/101) but see [C-FEATURE](#c-feature) |

- In `UpperCamelCase`, acronyms and contractions of compound words count as one word: `Uuid` not `UUID`, `Usize` not `USize`, `Stdin` not `StdIn`.
- In `snake_case`, acronyms and contractions are lower-cased: `is_xid_start`.
- In `snake_case` or `SCREAMING_SNAKE_CASE`, a "word" **SHOULD NOT** consist of a single letter unless it is the last "word": `btree_map` not `b_tree_map`, but `PI_2` not `PI2`.
- Crate names **SHOULD NOT** use `-rs` or `-rust` as a suffix or prefix. Every crate is Rust; reminding users of it serves no purpose.
- Std examples: the whole standard library.

**Google note:** Google narrows several rows: lifetimes should have descriptive names ([G-NAM-02](../google/10-naming.md#g-nam-02)), generic parameters may be multi-character and should be indicative of the bound ([G-NAM-03](../google/10-naming.md#g-nam-03)), constructors are `new()` / `new_with_foo()` / `try_new()` and conversions `from_foo()` ([G-NAM-05](../google/10-naming.md#g-nam-05)), and obscure abbreviations are banned ([G-NAM-06](../google/10-naming.md#g-nam-06)).

**Example:**
```rust
struct Uuid;              // not UUID
fn is_xid_start() {}      // acronym lower-cased
mod btree_map {}          // not b_tree_map
const PI_2: f64 = 0.0;    // single-letter word only as the last word; not PI2
```

**See also:** [Rust Style Guide — other style advice (names)](../rust-style-guide/06-other-style-advice.md), [Comprehensive Rust naming conventions](../references/comprehensive-rust-naming-conventions.md)

<a id="c-conv"></a>

## C-CONV — Ad-hoc conversions follow `as_`, `to_`, `into_` conventions

**Rule:** Conversions **SHOULD** be provided as methods whose name prefix reflects cost and ownership:

| Prefix | Cost | Ownership |
| ------ | ---- | --------- |
| `as_` | Free | borrowed -> borrowed |
| `to_` | Expensive | borrowed -> borrowed<br>borrowed -> owned (non-Copy types)<br>owned -> owned (Copy types) |
| `into_` | Variable | owned -> owned (non-Copy types) |

**Details:**
- [`str::as_bytes()`](https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes): free view of a `str` as UTF-8 bytes; borrowed `&str` -> borrowed `&[u8]`.
- [`Path::to_str`](https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str): expensive UTF-8 check on an OS path; borrowed -> borrowed. Calling it `as_str` would be wrong because it has nontrivial runtime cost.
- [`str::to_lowercase()`](https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase): Unicode-correct lowercase; iterates characters and may allocate; borrowed `&str` -> owned `String`.
- [`f64::to_radians()`](https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians): degrees to radians; takes `f64` (a `&f64` is unwarranted since `f64` is cheap to copy). `into_radians` would mislead because the input is not consumed.
- [`String::into_bytes()`](https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes): free extraction of the underlying `Vec<u8>`; owned `String` -> owned `Vec<u8>`.
- [`BufReader::into_inner()`](https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner): takes ownership and extracts the underlying reader for free; buffered data is discarded.
- [`BufWriter::into_inner()`](https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner): takes ownership and extracts the underlying writer, requiring a potentially expensive flush of buffered data.
- `as_` and `into_` typically _decrease abstraction_: `as` exposes a view into the underlying representation, `into` deconstructs data into it. `to_` typically stays at the same abstraction level but does work to change representation.
- When a type wraps a single value to give it higher-level semantics, access to the wrapped value **SHOULD** be provided by an `into_inner()` method. Applies to buffering wrappers ([`BufReader`](https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner)), encoding/decoding ([`GzDecoder`](https://docs.rs/flate2/0.2.19/flate2/read/struct.GzDecoder.html#method.into_inner)), atomic access ([`AtomicBool`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner)), or similar semantics.
- If `mut` in a conversion name is part of the return type, it **SHOULD** appear where it appears in the type: [`Vec::as_mut_slice`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice) (returns a mut slice) is preferred over `as_slice_mut`.
- More std examples: [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref), [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr), [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec), [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter).

**Google note:** Google makes these prefixes with the matching `self` convention mandatory for method naming ([G-NAM-01](../google/10-naming.md#g-nam-01)) and names value-to-value conversion constructors `from_foo()` backed by a `From` impl ([G-NAM-05](../google/10-naming.md#g-nam-05)).

**Example:**
```rust
// Return type is a mut slice.
fn as_mut_slice(&mut self) -> &mut [T];
```

**See also:** [C-GETTER](#c-getter), [C-CONV-TRAITS](02-interoperability.md#c-conv-traits), [`clippy::wrong_self_convention`](../references/clippy-lints.md#wrong_self_convention)

<a id="c-getter"></a>

## C-GETTER — Getter names follow Rust convention

**Rule:** With a few exceptions, getters **SHOULD NOT** use the `get_` prefix.

**Details:**
- A getter for field `first` is `first()`, not `get_first()`; its mutable variant is `first_mut()`, not `get_first_mut`, `get_mut_first` or `mut_first`.
- For getters that do runtime validation such as bounds checking, consider (**SHOULD**) adding unsafe `_unchecked` variants, typically with the signatures shown below.
- The line between getters and conversions ([C-CONV](#c-conv)) can be subtle and is not always clear-cut. E.g. [`TempDir::path`](https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.path) is a getter for the temporary directory's filesystem path, while [`TempDir::into_path`](https://docs.rs/tempdir/0.3.5/tempdir/struct.TempDir.html#method.into_path) is a conversion that transfers responsibility for deleting the directory to the caller. Since `path` is a getter, `get_path` or `as_path` would be wrong.

**Exceptions:**
- `get` is used only when there is a single and obvious thing that could reasonably be gotten, e.g. [`Cell::get`](https://doc.rust-lang.org/std/cell/struct.Cell.html#method.get) accesses the content of a `Cell`.
- Std examples: [`std::io::Cursor::get_mut`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut), [`std::pin::Pin::get_mut`](https://doc.rust-lang.org/std/pin/struct.Pin.html#method.get_mut), [`std::sync::PoisonError::get_mut`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut), [`std::sync::atomic::AtomicBool::get_mut`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut), [`std::collections::hash_map::OccupiedEntry::get_mut`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut), [`<[T]>::get_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked).

**Google note:** Google adds a full accessor naming set: `foo()`, `foo_mut()`, `take_foo()`, `set_foo()`, `with_foo()` / `with_mut_foo()` (closure accessors) and `replace_foo()` ([G-FEAT-03](../google/12-other-rust-features.md#g-feat-03)).

**Example:**
```rust
pub struct S {
    first: First,
    second: Second,
}

impl S {
    // Not get_first.
    pub fn first(&self) -> &First {
        &self.first
    }

    // Not get_first_mut, get_mut_first, or mut_first.
    pub fn first_mut(&mut self) -> &mut First {
        &mut self.first
    }
}
```

```rust
// Checked getters plus unsafe unchecked variants.
fn get(&self, index: K) -> Option<&V>;
fn get_mut(&mut self, index: K) -> Option<&mut V>;
unsafe fn get_unchecked(&self, index: K) -> &V;
unsafe fn get_unchecked_mut(&mut self, index: K) -> &mut V;
```

**See also:** [C-CONV](#c-conv)

<a id="c-iter"></a>

## C-ITER — Methods on collections that produce iterators follow `iter`, `iter_mut`, `into_iter`

**Rule:** Per [RFC 199](https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md), for a container with elements of type `U`, iterator methods **SHOULD** be named `iter` (yields `&U`), `iter_mut` (yields `&mut U`) and `into_iter` (yields `U`).

**Details:**
- Applies to data structures that are conceptually homogeneous collections.
- Applies to methods only, not functions.
- Std examples: [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter), [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut), [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter), [`BTreeMap::iter`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter), [`BTreeMap::iter_mut`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter_mut).

**Exceptions:**
- Not conceptually homogeneous collections: `str` is a slice of bytes guaranteed to be valid UTF-8, which is more nuanced, so instead of `iter`/`iter_mut`/`into_iter` it provides [`str::bytes`](https://doc.rust-lang.org/std/primitive.str.html#method.bytes) (iterate as bytes) and [`str::chars`](https://doc.rust-lang.org/std/primitive.str.html#method.chars) (iterate as chars).
- Functions: [`percent_encode`](https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html) from the `url` crate returns an iterator over percent-encoded string fragments; an `iter`/`iter_mut`/`into_iter` convention would add no clarity.

**Example:**
```rust
fn iter(&self) -> Iter             // Iter implements Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut implements Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter implements Iterator<Item = U>
```

**See also:** [C-ITER-TY](#c-iter-ty)

<a id="c-iter-ty"></a>

## C-ITER-TY — Iterator type names match the methods that produce them

**Rule:** A method called `into_iter()` **SHOULD** return a type called `IntoIter`, and similarly for all other methods that return iterators.

**Details:**
- Applies chiefly to methods, but often makes sense for functions too: the `url` crate's [`percent_encode`](https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html) function returns an iterator type called [`PercentEncode`](https://docs.rs/url/1.4.0/url/percent_encoding/struct.PercentEncode.html).
- These type names make the most sense prefixed with their owning module, e.g. [`vec::IntoIter`](https://doc.rust-lang.org/std/vec/struct.IntoIter.html).
- Std examples:
  - [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter) returns [`Iter`](https://doc.rust-lang.org/std/slice/struct.Iter.html)
  - [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut) returns [`IterMut`](https://doc.rust-lang.org/std/slice/struct.IterMut.html)
  - [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter) returns [`IntoIter`](https://doc.rust-lang.org/std/vec/struct.IntoIter.html)
  - [`BTreeMap::keys`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys) returns [`Keys`](https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html)
  - [`BTreeMap::values`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.values) returns [`Values`](https://doc.rust-lang.org/std/collections/btree_map/struct.Values.html)

**See also:** [C-ITER](#c-iter)

<a id="c-feature"></a>

## C-FEATURE — Feature names are free of placeholder words

**Rule:** A [Cargo feature](http://doc.crates.io/manifest.html#the-features-section) name **MUST NOT** include words that convey zero meaning, as in `use-abc` or `with-abc`; name the feature `abc` directly.

**Details:**
- Most common case: crates with an optional dependency on the Rust standard library. The feature **MUST** be called `std`, not `use-std`, `with-std` or any other creative name.
- This aligns with the implicit features Cargo infers for optional dependencies: an optional `serde` dependency is enabled with `features = ["serde"]` (not `use-serde` / `with-serde`), so explicit features like `std` should behave the same way.
- Cargo requires features to be additive, so a negatively named feature like `no-abc` is practically never correct.

**Example:**
```toml
# In Cargo.toml (canonical optional-std pattern)

[features]
default = ["std"]
std = []
```

```rust
// In lib.rs
#![no_std]

#[cfg(feature = "std")]
extern crate std;
```

```toml
# Crate `x` with optional dependencies on Serde and on the Rust standard library.
# Depend with features = ["serde"] and/or features = ["std"].
[package]
name = "x"
version = "0.1.0"

[features]
std = ["serde/std"]

[dependencies]
serde = { version = "1.0", optional = true }
```

**See also:** [C-SERDE](02-interoperability.md#c-serde), [Rust Style Guide — Cargo.toml](../rust-style-guide/07-cargo-toml.md)

<a id="c-word-order"></a>

## C-WORD-ORDER — Names use a consistent word order

**Rule:** Names **SHOULD** use a word order consistent within the crate and with similar functionality in the standard library; the particular order chosen is not important.

**Details:**
- Std error types use verb-object-error order: [`JoinPathsError`](https://doc.rust-lang.org/std/env/struct.JoinPathsError.html), [`ParseBoolError`](https://doc.rust-lang.org/std/str/struct.ParseBoolError.html), [`ParseCharError`](https://doc.rust-lang.org/std/char/struct.ParseCharError.html), [`ParseFloatError`](https://doc.rust-lang.org/std/num/struct.ParseFloatError.html), [`ParseIntError`](https://doc.rust-lang.org/std/num/struct.ParseIntError.html), [`RecvTimeoutError`](https://doc.rust-lang.org/std/sync/mpsc/enum.RecvTimeoutError.html), [`StripPrefixError`](https://doc.rust-lang.org/std/path/struct.StripPrefixError.html).
- So an error for an address failing to parse would be `ParseAddrError`, not `AddrParseError`.

**Example:**
```rust
// Bad: breaks the std verb-object-error order
struct AddrParseError;

// Good
struct ParseAddrError;
```
