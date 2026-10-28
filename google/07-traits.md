# Traits

> **Source:** [Google Rust Style Guide — Traits](https://google.github.io/styleguide/rustguide.html#traits) · **License:** CC-BY 3.0
>
> When to implement and derive traits, generics versus `dyn Trait`, traits versus inherent impls, extension traits, and guidance for important standard traits.

<a id="g-trt-01"></a>

## G-TRT-01 — Add trait implementations when they become necessary

**Rule:** As a general rule, low-level library code **SHOULD** anticipate users' needs proactively, while application code **SHOULD** add trait impls as needed.

**See also:** [G-BG-02](01-background-and-scope.md#g-bg-02), [G-TRT-07](#g-trt-07)

<a id="g-trt-02"></a>

## G-TRT-02 — Implement known standard library traits instead of custom methods

**Rule:** If a method matches the purpose and behavior of a standard library trait, you **SHOULD** implement that trait (e.g. `impl FromStr for Foo` instead of `fn from_str(&str) -> Self` on `Foo`).

**Exceptions:**
- Having both the method and the trait impl is acceptable for discoverability reasons.

**Example:**
```rust
// Bad: custom method duplicating a std trait
impl Foo {
    fn from_str(s: &str) -> Self { ... }
}

// Good
impl FromStr for Foo { ... }
```

**See also:** [`clippy::should_implement_trait`](../references/clippy-lints.md#should_implement_trait) ([list of traits](https://github.com/rust-lang/rust-clippy/blob/928349795f9f28f54e69bd0f0ffeed33de5e272a/clippy_lints/src/methods/mod.rs#L3940-L3971))

<a id="g-trt-03"></a>

## G-TRT-03 — Use `#[derive()]` if possible

**Rule:** Manual trait implementations **SHOULD NOT** be written if they can be expressed as a `#[derive()]`.

<a id="g-trt-04"></a>

## G-TRT-04 — Generics versus `dyn Trait`

**Rule:** In libraries, function parameters **SHOULD** accept generics (`T: Trait` or `impl Trait`) rather than `dyn Trait`.

**Applies to:** library code

**Exceptions:**
- Cases where code size matters.

**Rationale:**
- Efficiency:
  - Additional indirection: creating a `dyn Trait` requires setting up a fat pointer, and every call of a trait function requires a vtable lookup.
  - With a `dyn Trait` parameter the compiler can't perform certain optimizations it could for a generic function, such as inlining calls to the trait's functions.
- Some traits are not [`dyn` compatible](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility): e.g. a trait is not `dyn` compatible if it has generic functions, associated functions without a `self` parameter, or functions that return `Self`.

**See also:** [dyn compatibility](../references/language-concepts.md#dyn-compatibility), [G-TRT-05](#g-trt-05)

<a id="g-trt-05"></a>

## G-TRT-05 — When to define a trait versus an inherent implementation

**Rule:** You **MUST** define a trait only for the purposes listed below; in all other cases where you want to implement functionality for a type, use an [inherent implementation](https://doc.rust-lang.org/std/keyword.impl.html#inherent-implementations). You **MUST NOT** use traits merely to organize code.

**Details:**
- Permitted purposes for defining a trait:
  - To enable generic programming (using `T: Trait` in constraints of generic functions and types).
  - To accept or return an `impl Trait` or `dyn Trait`.
  - To define an [extension trait](#g-trt-06) (subject to the caveats in G-TRT-06).

**See also:** [inherent impls](../references/language-concepts.md#inherent-impls), [G-TRT-04](#g-trt-04), [G-TRT-06](#g-trt-06)

<a id="g-trt-06"></a>

## G-TRT-06 — Extension traits

**Rule:** An extension trait's name **SHOULD** carry the suffix `~Ext` (e.g. `ForeignTypeExt`). In general, the team that owns the original type **SHOULD** also own the extension trait.

**Details:**
- An [extension trait](https://rust-lang.github.io/rfcs/0445-extension-trait-conventions.html) is a trait that adds methods to a type from a different crate.
- Its sole purpose is to add methods to the type; it is not intended for use in generics (`T: Trait`, `impl Trait`) or dynamic dispatch (`dyn Trait`). The `~Ext` suffix signals this.
- An extension trait can also be defined for every type implementing some foreign trait; this is effectively equivalent to adding default methods to the foreign trait (second example).
- Note: this rule is specifically about extension traits. Defining a trait as an input trait (e.g. `DateLike`) and implementing it on many foreign types is relatively common and not covered.

**Exceptions:**
- Extension methods expected to become well known and provide significant value, e.g. in foundational libraries / frameworks (ownership by a different team is acceptable).

**Rationale:**
- When the same team owns the extension trait and the extended type, method name conflicts are avoided, both between the base type and the extension trait and between two extension traits.

**Example:**
```rust
// Extension trait for a foreign type
extern crate foreign_crate;
use foreign_crate::ForeignType;

trait ForeignTypeExt {
  fn additional_method(&self);
}

impl ForeignTypeExt for ForeignType {
  fn additional_method(&self) { ... }
}
```

```rust
// Extension trait for every type implementing a foreign trait
extern crate foreign_crate;
use foreign_crate::ForeignTrait;

trait ForeignTraitExt {
  fn additional_method(&self);
}

impl<T: ForeignTrait> ForeignTraitExt for T {
  fn additional_method(&self) { ... }
}
```

**See also:** [extension traits](../references/language-concepts.md#extension-traits), [G-NAM-04](10-naming.md#g-nam-04), [G-TRT-05](#g-trt-05)

<a id="g-trt-07"></a>

## G-TRT-07 — Specific guidance for important traits

**Rule:** Important standard traits are implemented according to the per-trait guidance below; the requirement strength is marked on each item.

**Details:**
- **Types with public fields:** you **SHOULD** consider implementing the traits implemented by all subfields.
- **Error types:** you **MUST** implement `Copy` whenever possible; `Copy` types are simpler to manage and propagate because they can be passed by value without moving ownership. Not possible e.g. when the error must contain non-`Copy` fields like a `String`.
- **`Copy`:** you **MUST** implement it whenever it is reasonably clear the type will continue to be `Copy` in the future.
- **`Clone`:** most types **SHOULD** implement `Clone` unless they represent some external resource.
- **`Default`:** you **MUST** implement it if the type implements `new()` or otherwise has a default state (see [`clippy::new_without_default`](../references/clippy-lints.md#new_without_default)).
- **`PartialEq` / `Eq`:** if you want users to use the `==` operator, you **SHOULD** prefer implementing `PartialEq` over `Eq`. When you do implement `Eq`, be aware of the additionally required reflexivity ([`Eq` docs](https://doc.rust-lang.org/std/cmp/trait.Eq.html)).
- **`PartialOrd` / `Ord`:** if you want users to use the `<` or `>` operators, you **SHOULD** prefer implementing `PartialOrd` over `Ord`. When you do implement `Ord`, be aware that it requires total ordering ([`Ord` docs](https://doc.rust-lang.org/std/cmp/trait.Ord.html)).
- **Map keys:**
  - If a type is likely to be used as a map key or otherwise cached, you **MUST** implement `Hash` and `Eq`.
  - If, in addition, the type has a clear total ordering, you **MUST** implement `Ord` (and any traits it needs).

**See also:** [G-TRT-01](#g-trt-01), [G-TRT-03](#g-trt-03), [G-TRT-09](#g-trt-09)

<a id="g-trt-08"></a>

## G-TRT-08 — Pair important `From` impls with explicit constructor methods

**Rule:** If a type has an important `From` impl expected to be one of the primary ways of constructing it, you **MUST** also add an inherent `::new()`, `::new_from_foo()`, or `::from_foo()` method.

**See also:** [`From` conversions](../references/language-concepts.md#from-conversions), [G-NAM-05](10-naming.md#g-nam-05)

<a id="g-trt-09"></a>

## G-TRT-09 — Options structs should implement Default

**Rule:** Structs with all-public fields that are primarily used as a "bag of options" for a function **SHOULD** implement `Default`.

**Rationale:**
- Enables convenient use of struct update syntax (`Struct { a: 0, ..Default::default() }`).

**See also:** [G-TRT-07](#g-trt-07)
