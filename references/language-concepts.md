# Language concepts referenced by the Google Rust Style Guide

> **Source:** pages linked from the [Google Rust Style Guide](https://google.github.io/styleguide/rustguide.html) (each entry lists its own URL) · **License:** as stated per entry (Rust project docs, RFCs and the `ctor` crate: MIT OR Apache-2.0; the "Little Orphan Impls" blog post states no license and is only summarized here)
>
> Condensed definitions, conditions and conventions from the external pages the Google guide links for Rust concepts, with the Google rules that rely on them.

<a id="newtype"></a>

## Newtype

**Source:** [Rust By Example — New Type Idiom](https://doc.rust-lang.org/rust-by-example/generics/new_types.html) · MIT OR Apache-2.0

- A newtype is a tuple struct wrapping one value (e.g. `struct Miles(f64);`). It gives compile-time guarantees that the right type of value is supplied: a function that takes `&Miles` **must** be given a `Miles`, not a `Kilometers`, even though both wrap `f64`.
- Get the inner value with tuple syntax (`.0`) or destructuring.

```rust
struct Miles(f64);
struct Kilometers(f64);

impl Miles {
    pub fn to_kilometers(&self) -> Kilometers {
        Kilometers(self.0 * 1.609344)
    }
}

impl Kilometers {
    pub fn to_miles(&self) -> Miles {
        Miles(self.0 / 1.609344)
    }
}

fn is_a_marathon(distance: &Miles) -> bool {
    distance.0 >= 26.2
}

fn main() {
    let distance = Miles(30.0);
    let distance_km = distance.to_kilometers();
    println!("Is a marathon? {}", is_a_marathon(&distance));
    println!("Is a marathon? {}", is_a_marathon(&distance_km.to_miles()));
    // println!("Is a marathon? {}", is_a_marathon(&distance_km)); // does not compile: expects `Miles`

    let distance_as_primitive_1: f64 = distance.0; // Tuple
    let Miles(distance_as_primitive_2) = distance; // Destructuring
}
```

**Cited by:** [G-TYP-01](../google/06-types.md#g-typ-01) (newtype vs type alias), [G-TYP-02](../google/06-types.md#g-typ-02) (newtype tuple structs are exempt from the named-fields rule).

<a id="extension-traits"></a>

## Extension traits (RFC 445)

**Source:** [RFC 445 — Extension trait conventions](https://rust-lang.github.io/rfcs/0445-extension-trait-conventions.html) · MIT OR Apache-2.0

- **Definition:** inherent methods can only be defined in the crate that defines the type. An extension trait is a trait defined to add methods to an existing type (or family of types) outside its defining crate. What makes it an *extension* trait is that it is not designed for generic use, only as a way to add methods to specific type(s). The distinction is somewhat subjective: if the trait could be used more generically, name and export it as a "normal" trait. Extension traits SHOULD be used sparingly.
- **Dyn-compatibility split:** because a trait is usable as a trait object only if all its methods are, a trait is sometimes split into a dyn-compatible core trait plus an extension trait holding the non-dyn-compatible methods, with a blanket impl. `Box<dyn Core>` implements the core trait, so all extension methods work on trait objects too.
- **Convention:**
  1. Prefer adding default methods to existing traits, or prefer generically useful traits, over extension traits whenever feasible.
  2. A true extension trait should have a clear type or trait it extends, and is named `FooExt` where `Foo` is that type or trait.
  3. Conditionally-applicable extension traits combine the type/trait name, the qualification and `Ext`: e.g. `AdditiveIterator` → `IteratorAddExt`.
  4. The earlier `Prelude` suffix convention (RFC 344) is deprecated.
- **Alternatives rejected:** `Util` / `Methods` suffixes; `Ext` is shorter and names the pattern.
- **Drawbacks:** extension traits tend to require extra imports.

```rust
extern crate foo;
use foo::Foo;

trait FooExt {
    fn bar(&self);
}

impl FooExt for Foo {
    fn bar(&self) { .. }
}
```

```rust
// The core, object-safe trait
trait Iterator<A> {
    fn next(&mut self) -> Option<A>;
}

// The extension trait offering object-unsafe methods
trait IteratorExt<A>: Iterator<A> {
    fn chain<U: Iterator<A>>(self, other: U) -> Chain<Self, U> { ... }
    fn zip<B, U: Iterator<B>>(self, other: U) -> Zip<Self, U> { ... }
    fn map<B>(self, f: |A| -> B) -> Map<'r, A, B, Self> { ... }
    ...
}

// A blanket impl
impl<A, I> IteratorExt<A> for I where I: Iterator<A> {
    ...
}
```

(Pre-1.0 syntax, kept as in the RFC.)

**Cited by:** [G-TRT-05](../google/07-traits.md#g-trt-05), [G-TRT-06](../google/07-traits.md#g-trt-06), [G-NAM-04](../google/10-naming.md#g-nam-04) (`~Ext` suffix).

<a id="dyn-compatibility"></a>

## Dyn compatibility

**Source:** [The Rust Reference — Traits: Dyn compatibility](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility) · MIT OR Apache-2.0

A dyn-compatible trait can be the base trait of a trait object (`dyn Trait`). Formerly called *object safety*. A trait is dyn compatible if all of:

- All supertraits are dyn compatible.
- `Sized` is not a supertrait (it must not require `Self: Sized`).
- It has no associated constants.
- It has no associated types with generics.
- Every associated function is either dispatchable from a trait object or explicitly non-dispatchable:
  - **Dispatchable** functions must:
    - have no type parameters (lifetime parameters are allowed);
    - be a method that does not use `Self` except in the receiver type;
    - have a receiver of type `&Self` (`&self`), `&mut Self` (`&mut self`), `Box<Self>`, `Rc<Self>`, `Arc<Self>`, or `Pin<P>` where `P` is one of these;
    - not have an opaque return type: not an `async fn` (hidden `Future` type), no return-position `impl Trait`;
    - not have a `where Self: Sized` bound (a `self` by-value receiver implies this);
    - not have a C-variadic parameter (`_: ...`).
  - **Explicitly non-dispatchable** functions require a `where Self: Sized` bound (a by-value `self` receiver implies this).
- The `AsyncFn`, `AsyncFnMut` and `AsyncFnOnce` traits are not dyn compatible.

```rust
// Dyn compatible methods.
trait TraitMethods {
    fn by_ref(self: &Self) {}
    fn by_ref_mut(self: &mut Self) {}
    fn by_box(self: Box<Self>) {}
    fn by_rc(self: Rc<Self>) {}
    fn by_arc(self: Arc<Self>) {}
    fn by_pin(self: Pin<&Self>) {}
    fn with_lifetime<'a>(self: &'a Self) {}
    fn nested_pin(self: Pin<Arc<Self>>) {}
}

// Dyn compatible, but these methods cannot be dispatched on a trait object.
trait NonDispatchable {
    fn foo() where Self: Sized {}                          // non-methods cannot be dispatched
    fn returns(&self) -> Self where Self: Sized;           // Self type isn't known until runtime
    fn param(&self, other: Self) where Self: Sized {}      // `other` may be a different concrete type
    fn typed<T>(&self, x: T) where Self: Sized {}          // generics are not compatible with vtables
}

// Dyn incompatible.
trait DynIncompatible {
    const CONST: i32 = 1;              // ERROR: associated const
    fn foo() {}                        // ERROR: associated function without Sized
    fn returns(&self) -> Self;         // ERROR: Self in return type
    fn typed<T>(&self, x: T) {}        // ERROR: generic type parameters
    fn nested(self: Rc<Box<Self>>) {}  // ERROR: nested receiver cannot be dispatched on
}

trait TraitWithSize where Self: Sized {}         // `Self: Sized` traits are dyn incompatible

trait Super<A> {}
trait WithSelf: Super<Self> where Self: Sized {} // dyn incompatible: `Self` as a type argument
```

**Cited by:** [G-TRT-04](../google/07-traits.md#g-trt-04) (rationale for preferring generics over `dyn Trait`: some traits are not dyn compatible, e.g. generic functions, associated functions without `self`, functions returning `Self`).

<a id="inherent-impls"></a>

## Inherent implementations

**Source:** [std — Keyword `impl`: Inherent Implementations](https://doc.rust-lang.org/std/keyword.impl.html#inherent-implementations) · MIT OR Apache-2.0

- An `impl` block implements functionality for a type. Inherent implementations are standalone (`impl Type { .. }`); trait implementations implement a trait for a type (`impl Trait for Type { .. }`).
- An implementation contains functions and consts. A function is called as `Type::f()`; if its first parameter is `self`, `&self` or `&mut self` it can also be called with method syntax (`value.f()`).
- Scope: an inherent implementation's functionality is in scope wherever the implementing type is; a trait implementation's functionality is brought into scope by importing the trait.
- Per the Reference glossary, an inherent implementation applies to a nominal type, not a trait-type pair. Per [RFC 445](https://rust-lang.github.io/rfcs/0445-extension-trait-conventions.html), inherent methods can only be defined in the crate that defines the type (see [extension traits](#extension-traits)).

```rust
struct Example {
    number: i32,
}

impl Example {
    fn boo() {
        println!("boo! Example::boo() was called!");
    }

    fn answer(&mut self) {
        self.number += 42;
    }

    fn get_number(&self) -> i32 {
        self.number
    }
}
```

**Cited by:** [G-TRT-05](../google/07-traits.md#g-trt-05) (use an inherent impl unless a trait is needed for generics, `impl`/`dyn Trait`, or an extension trait), [G-TRT-08](../google/07-traits.md#g-trt-08) (inherent constructor alongside an important `From` impl).

<a id="from-conversions"></a>

## When to implement `From`

**Source:** [std — `From`: When to implement `From`](https://doc.rust-lang.org/std/convert/trait.From.html#when-to-implement-from) · MIT OR Apache-2.0

There is no technical restriction, but a `From` conversion is expected to be:

- **Infallible:** if it can fail, use `TryFrom`; don't provide a `From` impl that panics.
- **Lossless:** it should not semantically lose or discard information. `i32: From<u16>` exists (recover with `u16: TryFrom<i32>`); `String: From<&str>` exists (get an equivalent back via `Deref`); there is no `u16: From<u32>`. Some wiggle room for semantically irrelevant information: `Box<[T]>: From<Vec<T>>` exists although capacity may not be preserved.
- **Value-preserving:** the conceptual kind and meaning of the value stay the same even if type and representation differ. `-1_i8 as u8` is lossless but not available via `From` (`-1` and `255` are different values); `f32: From<i16>` is available (same real number); `String: From<char>` is available (both text) but `String: From<u32>` is not (number vs text; use `Display`).
- **Obvious:** the only reasonable conversion between the two types. Otherwise use a named method or constructor (e.g. `str::as_bytes`, `u32::from_ne_bytes` / `from_le_bytes` / `from_be_bytes`). `IpAddr: From<Ipv6Addr>` exists because there is only one reasonable way to wrap it.

**Cited by:** [G-NAM-05](../google/10-naming.md#g-nam-05) (value-to-value conversions: `from_foo()` with an associated `From` impl), [G-TRT-08](../google/07-traits.md#g-trt-08).

<a id="turbofish"></a>

## Turbofish

**Source:** [The Rust Reference — Glossary: Turbofish](https://doc.rust-lang.org/reference/glossary.html#turbofish) · MIT OR Apache-2.0

Paths with generic parameters in expressions must prefix the opening `<` with `::`, giving `::<>` ("turbofish"). The prefix disambiguates generic paths from comparisons in comma-separated lists (see the [bastion of the turbofish](https://github.com/rust-lang/rust/blob/1.58.0/src/test/ui/parser/bastion-of-the-turbofish.rs)).

```rust
let ok_num = Ok::<_, ()>(5);
let vec = [1, 2, 3].iter().map(|n| n * 2).collect::<Vec<_>>();
```

**Cited by:** [G-IMPL-01](../google/13-implementation-concerns.md#g-impl-01) (prefer types on variable declarations over turbofishes; a forced turbofish may use `_`).

<a id="ctor"></a>

## Code before `main()`: the `ctor` crate

**Source:** [docs.rs — `ctor`](https://docs.rs/ctor/latest/ctor/) · MIT OR Apache-2.0 (crate license)

- `ctor` provides module initialization functions (like C/C++ `__attribute__((constructor))`) that run when a static library is loaded or an executable starts, i.e. before `main()`, on Linux, macOS, Windows, WASM, BSDs and others.
- `#[ctor]` (written `#[ctor(unsafe)]`) decorates free functions, `self`-less associated functions, and `static` items (initialized at startup; must not be accessed from other threads before constructors have run, or the initializer may panic). It is implemented with platform-specific linker sections (e.g. `.init_array`, `__DATA,__mod_init_func`, `.CRT$XCU`).
- **Upstream warning:** Rust's philosophy is that nothing happens before or after `main`, and this library explicitly subverts that. Code in `ctor`/`dtor` functions should limit itself to `libc` functions and code that does not rely on Rust's stdlib services.

```rust
use ctor::ctor;
use libc_print::*;

#[ctor(unsafe)]
fn foo() {
    libc_println!("Life before main!");
}
```

**Cited by:** [G-FEAT-12](../google/12-other-rust-features.md#g-feat-12) (do not run code before `main()`, e.g. via the `ctor` crate).

<a id="orphan-rules"></a>

## Orphan rules and trait coherence

**Source:** [Niko Matsakis — Little Orphan Impls (2015)](https://smallcultfollowing.com/babysteps/blog/2015/01/14/little-orphan-impls/) (license not stated); current rule: [The Rust Reference — Orphan rules](https://doc.rust-lang.org/reference/items/implementations.html#orphan-rules) (MIT OR Apache-2.0)

- **Coherence:** for a given trait and set of type parameters there must be exactly one applicable impl, so `MyType: Trait` identifies a unique impl.
- **Orphan rules** enforce coherence by preventing implementing *external traits for external types*. Intuitively: either the trait or the self type must be local. You cannot `impl Show for Vec<T>` in your library (both from std), but you can for `MyType`, and you can implement your own `MyTrait` for any type, including `Vec<T>`. The rules target "cousin" crates sharing a common ancestor (at least libstd) that both implement an ancestor's trait.
- **The post** (a 2015 design discussion) shows that "a local type appears somewhere" is unsound once type parameters are involved, and compares the *ordered* rule (a local type must appear before the first type parameter), the *covered* rule (impl type parameters must be covered by a local type), and hybrids ("covered self", "covered first"), leaning towards "covered first". Consequence: trait authors should put the "principal" type as `Self` and order other type parameters by importance.
- **Current rule (Reference):** given `impl<P1..=Pn> Trait<T1..=Tn> for T0`, the impl is valid only if `Trait` is a local trait, or at least one of `T0..=Tn` is a local type (call the first one `Ti`) and no uncovered type parameter `P1..=Pn` appears in `T0..Ti` (excluding `Ti`). For coherence, fundamental types are special: the `T` in `Box<T>` is not covered, and `Box<LocalType>` is local.

**Cited by:** [G-ORG-01](../google/02-code-organization.md#g-org-01) (larger crates are allowed when trait impl coherence rules force multiple types and traits into one library).

<a id="hir-grouping"></a>

## Large type groups: `rustc_hir::hir`

**Source:** [nightly-rustc — Module `rustc_hir::hir`](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_hir/hir/index.html) · MIT OR Apache-2.0

The Google guide cites this module, which contains a large number of HIR types (`Expr`, `Item`, `Block`, `Arm`, ...), as the example of a large "group" of types that may be referenced qualified (e.g. `hir::Expr`) instead of importing every type name.

**Cited by:** [G-ORG-09](../google/02-code-organization.md#g-org-09) (exception to importing type names unqualified).

<a id="cargo-layout"></a>

## Conventional Cargo package layout

**Source:** [The Cargo Book — Package Layout](https://doc.rust-lang.org/cargo/guide/project-layout.html) (linked by Google as [Cargo](https://doc.rust-lang.org/cargo/)) · MIT OR Apache-2.0

```text
.
├── Cargo.lock
├── Cargo.toml
├── src/
│   ├── lib.rs
│   ├── main.rs
│   └── bin/
│       ├── named-executable.rs
│       ├── another-executable.rs
│       └── multi-file-executable/
│           ├── main.rs
│           └── some_module.rs
├── benches/
│   ├── large-input.rs
│   └── multi-file-bench/
│       ├── main.rs
│       └── bench_module.rs
├── examples/
│   ├── simple.rs
│   └── multi-file-example/
│       ├── main.rs
│       └── ex_module.rs
└── tests/
    ├── some-integration-tests.rs
    └── multi-file-test/
        ├── main.rs
        └── test_module.rs
```

- `Cargo.toml` and `Cargo.lock` live in the package root.
- Source code goes in `src/`; default library `src/lib.rs`; default executable `src/main.rs`; other executables in `src/bin/`.
- Benchmarks in `benches/`, examples in `examples/`, integration tests in `tests/`.
- A multi-file binary, example, bench or integration test goes in a subdirectory of `src/bin`, `examples`, `benches` or `tests` with a `main.rs` plus extra modules; the executable is named after the directory.
- By convention binaries, examples, benches and integration tests use `kebab-case` (unless compatibility requires otherwise, e.g. a pre-existing binary name); modules within them use `snake_case`.

**Cited by:** [G-ORG-02](../google/02-code-organization.md#g-org-02) (Chromium's flat layout intentionally differs; `//third_party` code also built with Cargo follows this conventional layout).
