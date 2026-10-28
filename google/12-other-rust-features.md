# Other Rust features

> **Source:** [Google Rust Style Guide — Other Rust features](https://google.github.io/styleguide/rustguide.html#other-rust-features) · **License:** CC-BY 3.0
>
> Rules for unsafe code, unstable features, accessor naming, API completeness, `#[doc(hidden)]`, `const fn`, operator overloading, `#[non_exhaustive]`, `#[must_use]` and pre-`main()` code.

<a id="g-feat-01"></a>

## G-FEAT-01 — Unsafe code

**Rule:** **MUST NOT** write unsafe code unless there is no way to implement the same functionality with safe code.

**Details:**
- If you do have to write unsafe code:
  - Encapsulate it in a safe abstraction and isolate it from business logic as much as possible. Ideally, put unsafe code in a separate crate with few dependencies beyond the standard library.
  - Add a safety comment starting with `SAFETY:` that explains why the code is correct.
  - Add a technical justification for the unsafe code. Depending on context, it may justify an individual `unsafe` block or, more generally, cover an entire file or build target. It should answer:
    - Why is this code not written in an equivalent safe way?
    - If the reason is performance, are there benchmarks?

**Example:**
```rust
// SAFETY: `index` was bounds-checked against `self.items.len()` above.
let item = unsafe { self.items.get_unchecked(index) };
```

<a id="g-feat-02"></a>

## G-FEAT-02 — No unstable features

**Rule:** **MUST NOT** use unstable features.

**Exceptions:**
- `[android]` Embedded programming may require certain unstable features. Review is required to enable them (to help search for alternatives), but features absolutely required to build an artifact correctly will generally be approved.
- `[chrome]` Style arbiters may blanket-approve a small list of features for the entire codebase. Typically only features with a clear path to stabilization are approved.
- `[Fuchsia]` See the [Fuchsia unstable-features guidance](https://fuchsia.dev/fuchsia-src/development/languages/rust/unstable).

**See also:** [Platform notes](../references/platform-notes.md)

<a id="g-feat-03"></a>

## G-FEAT-03 — Getters and setters

**Rule:** Writing and naming getters and setters **SHOULD** follow the naming guidelines below.

**Details:**
- `foo()`: getter returning `&T`, or `T` by value if `T: Copy`.
- `foo_mut()`: getter returning `&mut T`.
- `take_foo()`: getter that "clears" the value (sets it to `None` or another default state) and returns the old value.
- `set_foo()`: setter taking `T` by value.
- `with_foo()` / `with_mut_foo()`: accessor operating via a closure, invoked as `object.with_foo(|foo| frob(foo))` (see below).
- `replace_foo()`: setter that returns the old value.
- **Accessors that accept closures** (`with_foo()`), typically invoked as `object.with_foo(|foo| do_something_with(foo))`, can be a good choice when:
  - The property is inside an interior-mutability wrapper (e.g. `Mutex`, `RefCell`). These mechanisms must constrain access to a scope, and the closure provides it. This is usually preferable to returning the RAII object implementing the scope (e.g. `MutexGuard<T>`), which would leak an implementation detail.
  - Follow-up work or validation is needed after the property changes and a `set_foo()` accessor can't be used (e.g. the property's type is expensive to clone or not cloneable at all).
- A `with_foo()` accessor often lets the closure return a value and forwards it as its own return value.
- If you need two closure-accepting accessors, one read-only and one mutable, they are commonly named `with_foo()` and `with_mut_foo()`. Otherwise, `with_foo()` is often also used for mutating accessors.

**Example:**
```rust
impl Stuff {
    fn foo(&self) -> &Foo { ... }
    fn foo_mut(&mut self) -> &mut Foo { ... }

    /// Sets "baz" property to `None` and returns the old value.
    fn take_baz(&mut self) -> Option<Baz> { ... }

    fn set_foo(&mut self, foo: Foo) { ... }

    /// Calls `f` with a borrow of the "bar" property.
    /// Returns the return value of `f`.
    fn with_bar<R>(&self, f: impl FnOnce(&Bar) -> R) -> R { ... }

    /// Calls `f` with a mutable borrow of the "bar" property.
    /// Returns the return value of `f`.
    fn with_mut_bar<R>(&self, f: impl FnOnce(&mut Bar) -> R) -> R { ... }

    /// Sets "foo" property to `foo` and returns the old value.
    fn replace_foo(&mut self, foo: Foo) -> Foo { ... }
}
```

**See also:** [C-GETTER](../api-guidelines/01-naming.md#c-getter), [G-NAM-01](../google/10-naming.md#g-nam-01)

<a id="g-feat-04"></a>

## G-FEAT-04 — Implement expected API sets together

**Rule:** If you implement `len()`, you **MUST** also implement `is_empty()`.

**Example:**
```rust
impl Queue {
    pub fn len(&self) -> usize { ... }
    pub fn is_empty(&self) -> bool { ... }
}
```

**See also:** [`clippy::len_without_is_empty`](../references/clippy-lints.md#len_without_is_empty)

<a id="g-feat-05"></a>

## G-FEAT-05 — Don't access `#[doc(hidden)]` items you don't own

**Rule:** Treat items marked `#[doc(hidden)]` as private: you **MUST NOT** access them unless you own the code.

<a id="g-feat-06"></a>

## G-FEAT-06 — Use `const` functions responsibly

**Rule:** **MUST NOT** mark a public function `const` unless you expect it to remain `const` in the future (or can commit to helping clients migrate when it changes).

**Rationale:**
- Turning a `const` function into a regular function is a breaking change.

<a id="g-feat-07"></a>

## G-FEAT-07 — Avoid overloading for non-mathematical purposes

**Rule:** Use operator overloads for their original mathematical/logical purpose; **SHOULD NOT** use them merely to make code more concise ("syntax sugar").

**See also:** [C-OVERLOAD](../api-guidelines/05-predictability.md#c-overload)

<a id="g-feat-08"></a>

## G-FEAT-08 — `#[non_exhaustive]` on publicly-exposed enums

**Rule:** If an enum is part of a public API used by other teams, you **MUST** mark it `#[non_exhaustive]`.

**Applies to:** Public APIs used by other teams.

**Exceptions:**
- The enum will never change, e.g.:
  - Enums describing an existing ontology (e.g. `enum Color { Red, Green, Blue }`).
  - Two-state enums that could be Booleans (e.g. `enum Visibility { Visible, Hidden }`).
- The enum *may* change in the future, but you are committed to fixing resulting breakages via an LSC (large-scale change).
- Enums used only a small number of times can't create a large breakage and thus may ignore this rule.

**Rationale:**
- `#[non_exhaustive]` forces users to include a wildcard arm (`_ => ...`) in `match` statements, so code keeps compiling when new variants are added later.
- Primarily important for enums used across team boundaries, where the scope of breakage could be significant.

**Example:**
```rust
// Public enum that may gain variants: mark it.
#[non_exhaustive]
pub enum Error {
    NotFound,
    PermissionDenied,
}

// Exception: existing ontology, will never change.
pub enum Color {
    Red,
    Green,
    Blue,
}

// Exception: two states that could be a Boolean.
pub enum Visibility {
    Visible,
    Hidden,
}
```

**See also:** [`clippy::exhaustive_enums`](../references/clippy-lints.md#exhaustive_enums), [G-IMPL-04](../google/13-implementation-concerns.md#g-impl-04), [G-FEAT-09](#g-feat-09)

<a id="g-feat-09"></a>

## G-FEAT-09 — `#[non_exhaustive]` on public structs with public fields

**Rule:** Structs with public fields that are part of a public API used by other teams **SHOULD** be marked `#[non_exhaustive]`.

**Applies to:** Public APIs used by other teams.

**Exceptions:**
- The fields are unlikely to change, e.g. they describe an existing ontology (such as a `Rectangle` type), or the reviewer can otherwise be confident they won't change.
- The fields *may* change in the future, but you are committed to fixing resulting breakages via an LSC (large-scale change).
- You want users to be able to use functional record update (FRU) initialization syntax (`Person { name: "foo", ..Default::default() }`).
- Structs used only a small number of times can't create a large breakage and thus may ignore this rule.

**Rationale:**
- Adding a new field (even a private one) to a struct whose fields are all public is a breaking change, because callers can no longer name all fields in a struct literal.
- A `#[non_exhaustive]` struct can't be constructed with struct literal syntax outside its own crate, so new private fields can be added without breaking users.
- Primarily important for structs used across team boundaries, where the scope of breakage could be significant.

**See also:** [C-STRUCT-PRIVATE](../api-guidelines/10-future-proofing.md#c-struct-private), [G-FEAT-08](#g-feat-08)

<a id="g-feat-10"></a>

## G-FEAT-10 — `#[must_use]` on functions

**Rule:** You **MUST** use `#[must_use]` on functions in the cases below.

**Details:**
- The caller cannot use the function correctly if they ignore the return value. This includes pure functions (no side effects), such as `HashMap::get()`.
- The return value represents an error or otherwise critical information about the function's behavior.

**Exceptions:**
- If the return type is already annotated `#[must_use]` (e.g. `Result`), the function need not be annotated.

**See also:** [G-FEAT-11](#g-feat-11)

<a id="g-feat-11"></a>

## G-FEAT-11 — `#[must_use]` on types

**Rule:** You **MUST** use `#[must_use]` on types intended to be returned by functions that would otherwise be annotated `#[must_use]` per [G-FEAT-10](#g-feat-10).

**Details:**
- Examples of such types:
  - Error types (similar to `Result` or `StatusOr`).
  - Types representing a lazily evaluated computation: the function call does nothing by itself, and the return value must be accessed to cause the desired side effect.

<a id="g-feat-12"></a>

## G-FEAT-12 — Don't run code before `main()`

**Rule:** **MUST NOT** cause code to run before `main()`, e.g. via techniques such as the [ctor crate](https://docs.rs/ctor/latest/ctor/).

**See also:** [Life before main / `ctor`](../references/language-concepts.md#ctor)
