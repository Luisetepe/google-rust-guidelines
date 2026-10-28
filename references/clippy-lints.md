# Clippy and rustc lints cited by the Google Rust Style Guide

> **Source:** [Clippy Lints](https://rust-lang.github.io/rust-clippy/master/index.html), [rust-clippy source](https://github.com/rust-lang/rust-clippy), [The rustc book — Allowed-by-default lints](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html) · **License:** MIT OR Apache-2.0
>
> Every Clippy and rustc lint the Google guide mentions, with group, default level, what it checks, why, examples, configuration and the Google rules that cite it.

Each entry is anchored by the exact lint name. "Group / default level" is as shown on the Clippy lint list (master). The Google guide cites these lints "for context": they back a Google rule, but the guide does not mandate enabling them.

<a id="borrowed_box"></a>

## borrowed_box

**Source:** [Clippy — `borrowed_box`](https://rust-lang.github.io/rust-clippy/master/index.html#borrowed_box) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/types/mod.rs))

**Group / default level:** `complexity` / `warn` · Added in: pre 1.29.0

**What it checks:** Usage of `&Box<T>` anywhere in the code.

**Why:** A `&Box<T>` parameter requires the caller to box `T` before calling. Using `&T` gives the parameter a concrete type and generalizes the function; a `&Box<T>` argument still auto-derefs to `&T` at the call site.

**Example:**
```rust
// Bad
fn foo(bar: &Box<T>) { ... }

// Good
fn foo(bar: &T) { ... }
```

**Cited by:** [G-FIO-01](../google/05-function-inputs-and-outputs.md#g-fio-01) (use `&T` instead of `&Box<T>`).

<a id="ptr_arg"></a>

## ptr_arg

**Source:** [Clippy — `ptr_arg`](https://rust-lang.github.io/rust-clippy/master/index.html#ptr_arg) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/ptr/mod.rs))

**Group / default level:** `style` / `warn` · Added in: pre 1.29.0

**What it checks:** Function arguments of type `&String`, `&Vec`, `&PathBuf`, and `Cow<_>`. Also suggests replacing `.clone()` calls with the appropriate `.to_owned()` / `to_string()` calls.

**Why:** Requiring the specific type makes the function less useful for no benefit; slices (`&[T]`, `&str`) usually suffice and can be obtained from other types too.

**Known problems:**
- `fn(&Vec)`-typed references pointing to your function will fail to compile after applying the suggestion; undo the change or change the reference's type.
- If the function is part of your public interface, other crates may reference it; carefully deprecate the function before applying the suggestion.

**Example:**
```rust
// Bad
fn foo(&Vec<u32>) { .. }

// Good
fn foo(&[u32]) { .. }
```

**Cited by:** [G-FIO-01](../google/05-function-inputs-and-outputs.md#g-fio-01) (use `&[T]` / `&str` instead of `&Vec<T>` / `&String`).

<a id="should_implement_trait"></a>

## should_implement_trait

**Source:** [Clippy — `should_implement_trait`](https://rust-lang.github.io/rust-clippy/master/index.html#should_implement_trait) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/methods/mod.rs); trait list as linked by Google: [`mod.rs#L3940-L3971` @ `9283497`](https://github.com/rust-lang/rust-clippy/blob/928349795f9f28f54e69bd0f0ffeed33de5e272a/clippy_lints/src/methods/mod.rs#L3940-L3971))

**Group / default level:** `style` / `warn` · Added in: pre 1.29.0

**What it checks:** Methods in an inherent implementation that should instead live in an implementation of a `std` trait.

**Why:** Implementing the traits improves ergonomics for users, often at very little cost. People seeing e.g. a `mul(...)` method may expect `*` to work too, so you need a good reason to disappoint them.

**Example:**
```rust
// Bad: inherent `add` can be confused with `std::ops::Add::add`
struct X;
impl X {
    fn add(&self, other: &X) -> X {
        // ..
    }
}
```

**How matching works** (at the commit Google links): only methods in inherent impls (not trait impls) that are exported are checked. A method is reported when all of these match a row of the table below: method name; parameter count (including `self`); output type; kind of the first parameter (self kind); function header = safe, non-`const`, non-`async` (ABI is not compared); and, for rows with "explicit lifetime: no", the method has no explicit lifetime parameters. The message is "method `X` can be confused for the standard trait method `Trait::X`" with the help "consider implementing the trait `Trait` or choosing a less ambiguous method name".

**Trait methods checked** (`TRAIT_METHODS`, 30 entries):

| Trait | Method | Params | Self kind | Output | Lints with explicit lifetimes |
| --- | --- | --- | --- | --- | --- |
| `std::ops::Add` | `add` | 2 | `self` | any | yes |
| `std::convert::AsMut` | `as_mut` | 1 | `&mut self` | reference | yes |
| `std::convert::AsRef` | `as_ref` | 1 | `&self` | reference | yes |
| `std::ops::BitAnd` | `bitand` | 2 | `self` | any | yes |
| `std::ops::BitOr` | `bitor` | 2 | `self` | any | yes |
| `std::ops::BitXor` | `bitxor` | 2 | `self` | any | yes |
| `std::borrow::Borrow` | `borrow` | 1 | `&self` | reference | yes |
| `std::borrow::BorrowMut` | `borrow_mut` | 1 | `&mut self` | reference | yes |
| `std::clone::Clone` | `clone` | 1 | `&self` | any | yes |
| `std::cmp::Ord` | `cmp` | 2 | `&self` | any | yes |
| `std::default::Default` | `default` | 0 | none | any | yes |
| `std::ops::Deref` | `deref` | 1 | `&self` | reference | yes |
| `std::ops::DerefMut` | `deref_mut` | 1 | `&mut self` | reference | yes |
| `std::ops::Div` | `div` | 2 | `self` | any | yes |
| `std::ops::Drop` | `drop` | 1 | `&mut self` | unit | yes |
| `std::cmp::PartialEq` | `eq` | 2 | `&self` | `bool` | yes |
| `std::iter::FromIterator` | `from_iter` | 1 | none | any | yes |
| `std::str::FromStr` | `from_str` | 1 | none | any | yes |
| `std::hash::Hash` | `hash` | 2 | `&self` | unit | yes |
| `std::ops::Index` | `index` | 2 | `&self` | reference | yes |
| `std::ops::IndexMut` | `index_mut` | 2 | `&mut self` | reference | yes |
| `std::iter::IntoIterator` | `into_iter` | 1 | `self` | any | yes |
| `std::ops::Mul` | `mul` | 2 | `self` | any | yes |
| `std::ops::Neg` | `neg` | 1 | `self` | any | yes |
| `std::iter::Iterator` | `next` | 1 | `&mut self` | any | no |
| `std::ops::Not` | `not` | 1 | `self` | any | yes |
| `std::ops::Rem` | `rem` | 2 | `self` | any | yes |
| `std::ops::Shl` | `shl` | 2 | `self` | any | yes |
| `std::ops::Shr` | `shr` | 2 | `self` | any | yes |
| `std::ops::Sub` | `sub` | 2 | `self` | any | yes |

"Self kind: none" means the first parameter's type must differ from `Self` (no receiver). "Lints with explicit lifetimes: no" means a method with explicit lifetime parameters is not reported, because such methods cannot implement the equivalent trait method.

**Cited by:** [G-TRT-02](../google/07-traits.md#g-trt-02) (implement the known std trait, e.g. `impl FromStr` instead of `fn from_str`; having both is acceptable for discoverability).

<a id="new_without_default"></a>

## new_without_default

**Source:** [Clippy — `new_without_default`](https://rust-lang.github.io/rust-clippy/master/index.html#new_without_default) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/new_without_default.rs))

**Group / default level:** `style` / `warn` · Added in: pre 1.29.0 · Past names: `new_without_default_derive`

**What it checks:** Public types with a `pub fn new() -> Self` method and no implementation of `Default`.

**Why:** The type can be constructed without arguments, so users may expect to be able to use `Default`.

**Example:**
```rust
// Bad
pub struct Foo(Bar);

impl Foo {
    pub fn new() -> Self {
        Foo(Bar::new())
    }
}

// Good: add a `Default` impl that delegates to `new`
pub struct Foo(Bar);

impl Default for Foo {
    fn default() -> Self {
        Foo::new()
    }
}
```

**Cited by:** [G-TRT-07](../google/07-traits.md#g-trt-07) (`Default`: implement if the type implements `new()` or otherwise has a default state); related: [G-TRT-09](../google/07-traits.md#g-trt-09).

<a id="wrong_self_convention"></a>

## wrong_self_convention

**Source:** [Clippy — `wrong_self_convention`](https://rust-lang.github.io/rust-clippy/master/index.html#wrong_self_convention) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/methods/mod.rs))

**Group / default level:** `style` / `warn` · Added in: pre 1.29.0

**What it checks:** Methods with certain name prefixes/suffixes that do not follow the standard conventions for how `self` is taken:

| Prefix | Postfix | `self` taken | `self` type |
| --- | --- | --- | --- |
| `as_` | none | `&self` or `&mut self` | any |
| `from_` | none | none | any |
| `into_` | none | `self` | any |
| `is_` | none | `&mut self` or `&self` or none | any |
| `to_` | `_mut` | `&mut self` | any |
| `to_` | not `_mut` | `self` | `Copy` |
| `to_` | not `_mut` | `&self` | not `Copy` |

- Not triggered for `to_` methods in trait definitions (Clippy cannot tell whether an implementing type is `Copy`).
- Not triggered for `to_` methods in trait implementations when `&self` is taken (the trait controls the signature, e.g. `std::string::ToString`).
- `Pin<&Self>` and `Pin<&mut Self>` are accepted where `&self` and `&mut self` are required.
- Background: [C-CONV](../api-guidelines/01-naming.md#c-conv).

**Why:** Consistency breeds readability; users following the conventions won't be surprised, e.g., by having to supply a mutable reference to an `as_..` function.

**Example:**
```rust
// Bad
impl X {
    fn as_str(self) -> &'static str {
        // ..
    }
}

// Good
impl X {
    fn as_str(&self) -> &'static str {
        // ..
    }
}
```

**Configuration:** `avoid-breaking-exported-api` (default `true`): suppress the lint whenever the suggested change would break other crates.

**Cited by:** [G-NAM-01](../google/10-naming.md#g-nam-01) (use `as_`/`to_`/`into_` prefixes with the appropriate `self` convention).

<a id="len_without_is_empty"></a>

## len_without_is_empty

**Source:** [Clippy — `len_without_is_empty`](https://rust-lang.github.io/rust-clippy/master/index.html#len_without_is_empty) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/len_without_is_empty.rs))

**Group / default level:** `style` / `warn` · Added in: pre 1.29.0

**What it checks:** Items that implement `.len()` but not `.is_empty()`.

**Why:** It is good custom to have both: for some data structures asking for the length is costly, while `.is_empty()` can usually answer in constant time. It also used to cause false positives in the `len_zero` lint (which now ignores such items).

**Example:**
```rust
// Bad: `len` without `is_empty`
impl X {
    pub fn len(&self) -> usize {
        ..
    }
}
```

**Cited by:** [G-FEAT-04](../google/12-other-rust-features.md#g-feat-04) (if you implement `len()`, also implement `is_empty()`).

<a id="exhaustive_enums"></a>

## exhaustive_enums

**Source:** [Clippy — `exhaustive_enums`](https://rust-lang.github.io/rust-clippy/master/index.html#exhaustive_enums) ([source](https://github.com/rust-lang/rust-clippy/blob/HEAD/clippy_lints/src/exhaustive_items.rs))

**Group / default level:** `restriction` / `allow` · Added in: 1.51.0

**What it checks:** Exported `enum`s that are not tagged `#[non_exhaustive]`.

**Why (restriction rationale):** Making an `enum` exhaustive is a stability commitment: adding a variant is a breaking change. A project may want no exhaustive enums, or every exhaustive enum explicitly `#[allow]`ed.

**Example:**
```rust
// Bad
enum Foo {
    Bar,
    Baz
}

// Good
#[non_exhaustive]
enum Foo {
    Bar,
    Baz
}
```

**Cited by:** [G-FEAT-08](../google/12-other-rust-features.md#g-feat-08) (mark publicly-exposed enums `#[non_exhaustive]`, with exceptions); related: [G-FEAT-09](../google/12-other-rust-features.md#g-feat-09), [G-IMPL-04](../google/13-implementation-concerns.md#g-impl-04).

<a id="missing_docs"></a>

## missing_docs

**Source:** [The rustc book — `missing-docs`](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#missing-docs)

**Kind / default level:** rustc lint (not Clippy) / `allow`

**What it checks:** Missing documentation for public items (including the crate itself).

**Why:** Ensures a library is well-documented; undocumented items can be hard for users to understand and use properly. It is allow-by-default because it can be noisy and not all projects want to enforce documenting everything.

**Example:**
```rust
#![deny(missing_docs)]
pub fn foo() {}
```

Produces:

```text
error: missing documentation for the crate
 --> lint_example.rs:1:1
  |
1 | / #![deny(missing_docs)]
2 | | fn main() {
3 | | pub fn foo() {}
4 | | }
  | |_^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_docs)]
  |         ^^^^^^^^^^^^
```

**Cited by:** [Google — When and how to write documentation](../google/11-documentation.md) (add doc comments to public APIs, potentially using `missing_docs`; it is fine to `allow` the lint where a doc comment would only restate the function name trivially).
