# Future proofing

> **Source:** [Rust API Guidelines — Future proofing](https://rust-lang.github.io/api-guidelines/future-proofing.html) · **License:** MIT OR Apache-2.0
>
> Guidelines that leave a crate free to improve without breaking users' code: sealed traits, private fields, encapsulating newtypes, and minimal trait bounds on data structures.

<a id="c-sealed"></a>

## C-SEALED — Sealed traits protect against downstream implementations

**Rule:** A trait meant to be implemented only within its defining crate **SHOULD** use the sealed trait pattern, and its rustdoc **SHOULD** state that it is sealed and not meant to be implemented outside the current crate.

**Details:**
- Pattern: the public trait has a supertrait `private::Sealed` that lives in a private module and is implemented only for the same types as the public trait.
- Downstream crates cannot name the empty private `Sealed` supertrait, so all impls of `Sealed` (and therefore of the public trait) exist in the current crate.
- You may then, in a non-breaking release, add methods to the trait (ordinarily a breaking change for unsealed traits) and change signatures of methods that are not publicly documented (e.g. `#[doc(hidden)]` private methods).
- Removing a public method or changing the signature of a public method of a sealed trait is still a breaking change.
- Documenting the sealing avoids frustrated users trying to implement the trait.
- Precedents: [`serde_json::value::Index`](https://docs.serde.rs/serde_json/value/trait.Index.html), [`byteorder::ByteOrder`](https://docs.rs/byteorder/1.1.0/byteorder/trait.ByteOrder.html).

**Rationale:**
- Retains the ability to change the trait in a non-breaking way.

**Google note:** Google's analogous future-proofing tool for public enums and structs used by other teams is `#[non_exhaustive]` ([G-FEAT-08](../google/12-other-rust-features.md#g-feat-08), [G-FEAT-09](../google/12-other-rust-features.md#g-feat-09)). Google treats `#[doc(hidden)]` items as private and forbids accessing them unless you own the code, so callers must not use the hidden methods of another team's sealed trait ([G-FEAT-05](../google/12-other-rust-features.md#g-feat-05)).

**Example:**
```rust
/// This trait is sealed and cannot be implemented for types outside this crate.
pub trait TheTrait: private::Sealed {
    // Zero or more methods that the user is allowed to call.
    fn ...();

    // Zero or more private methods, not allowed for user to call.
    #[doc(hidden)]
    fn ...();
}

// Implement for some types.
impl TheTrait for usize {
    /* ... */
}

mod private {
    pub trait Sealed {}

    // Implement for those same types, but no others.
    impl Sealed for usize {}
}
```

**See also:** [C-HIDDEN](04-documentation.md#c-hidden)

<a id="c-struct-private"></a>

## C-STRUCT-PRIVATE — Structs have private fields

**Rule:** Struct fields **SHOULD** be private, with getter/setter methods where needed. Public fields are most appropriate for C-spirit `struct` types: compound, passive data structures.

**Rationale:**
- A public field is a strong commitment: it pins down a representation choice, *and* prevents the type from validating or maintaining invariants on the field's contents, since clients can mutate it arbitrarily.

**Google note:** Google adds: public structs with public fields used by other teams should be `#[non_exhaustive]` (with listed exceptions, e.g. to allow functional record update) ([G-FEAT-09](../google/12-other-rust-features.md#g-feat-09)); getter/setter naming is `foo()`, `foo_mut()`, `set_foo()`, `take_foo()`, `replace_foo()`, `with_foo()` ([G-FEAT-03](../google/12-other-rust-features.md#g-feat-03)); all-public-field options structs should implement `Default` ([G-TRT-09](../google/07-traits.md#g-trt-09)); types with public fields should consider implementing traits implemented by all subfields ([G-TRT-07](../google/07-traits.md#g-trt-07)).

**See also:** [C-GETTER](01-naming.md#c-getter), [C-VALIDATE](08-dependability.md#c-validate)

<a id="c-newtype-hide"></a>

## C-NEWTYPE-HIDE — Newtypes encapsulate implementation details

**Rule:** You **SHOULD** use a newtype to hide representation details (e.g. a compound iterator type) while making precise promises to the client.

**Details:**
- E.g. `my_transform` returning `Enumerate<Skip<I>>` exposes its construction; wrapping it in `MyTransformResult<I>` that implements `Iterator<Item = (usize, I::Item)>` gives the client roughly that view only.
- Besides simplifying the signature, the client does not know *how* the result is constructed or represented, so the representation can change without breaking client code.
- [`impl Trait`](https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md) (since Rust 1.26) is more concise than a newtype but limits what you can express: e.g. returning an iterator that implements `Debug` or `Clone`, or some combination of other iterator extension traits, can be problematic.
- `impl Trait` as a return type is probably great for internal APIs and **MAY** even be appropriate for public APIs, but probably not in all cases. See the Edition Guide's ["`impl Trait` for returning complex types with ease"](https://rust-lang.github.io/edition-guide/rust-2018/trait-system/impl-trait-for-returning-complex-types-with-ease.html).

**Rationale:**
- Promising less to the client keeps the implementation free to change.

**Google note:** Google prefers returning iterators (concrete types or `impl Iterator`) over `Vec<T>` where possible, with listed exceptions ([G-FIO-02](../google/05-function-inputs-and-outputs.md#g-fio-02)), and lists the flexibility to change the underlying type later as a reason for a newtype over a type alias ([G-TYP-01](../google/06-types.md#g-typ-01)).

**Example:**
```rust
// Exposes the representation
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
```

```rust
// Good: newtype hides the representation
use std::iter::{Enumerate, Skip};

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);

    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

```rust
// Alternative: impl Trait (more concise, less expressive)
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```

**See also:** [C-NEWTYPE](07-type-safety.md#c-newtype), [newtype](../references/language-concepts.md#newtype)

<a id="c-struct-bounds"></a>

## C-STRUCT-BOUNDS — Data structures do not duplicate derived trait bounds

**Rule:** Generic data structures **SHOULD NOT** use trait bounds that can be derived or that do not otherwise add semantic value.

**Details:**
- Each trait in a `derive` attribute expands into a separate `impl` block that applies only to generic arguments implementing that trait, so the bound on the struct is unnecessary.
- Adding a trait bound to a data structure is generally a breaking change (every consumer must satisfy it); deriving more std traits via `derive` is not.
- These traits **SHOULD NOT** ever be used in bounds on data structures: `Clone`, `PartialEq`, `PartialOrd`, `Debug`, `Display`, `Default`, `Error`, `Serialize`, `Deserialize`, `DeserializeOwned`.
- Grey area: other non-derivable bounds not strictly required by the definition, like `Read` or `Write`, may communicate intended behavior better but also limit future extensibility. Semantically useful bounds are still less problematic than derivable traits as bounds.
- Std precedents for the exceptions:
  - [`std::borrow::Cow`](https://doc.rust-lang.org/std/borrow/enum.Cow.html) refers to an associated type on the `Borrow` trait.
  - [`std::boxed::Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html) opts out of the implicit `Sized` bound.
  - [`std::io::BufWriter`](https://doc.rust-lang.org/std/io/struct.BufWriter.html) requires a trait bound in its `Drop` impl.

**Exceptions:**
- Trait bounds on structures are required when:
  1. The data structure refers to an associated type on the trait.
  2. The bound is `?Sized`.
  3. The data structure has a `Drop` impl that requires trait bounds (Rust currently requires all trait bounds on the `Drop` impl to also be present on the data structure).

**Rationale:**
- Duplicating derived traits as bounds is unnecessary and a backwards-compatibility hazard.

**Example:**
```rust
// Good
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Bad
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

```rust
// Deriving PartialOrd later:

// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

**See also:** [G-TRT-03](../google/07-traits.md#g-trt-03)
