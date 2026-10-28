# Naming

> **Source:** [Google Rust Style Guide — Naming](https://google.github.io/styleguide/rustguide.html#naming) · **License:** CC-BY 3.0
>
> Naming rules for methods, lifetimes, generic parameters, extension traits and initialization functions, plus the ban on obscure abbreviations.

<a id="g-nam-01"></a>

## G-NAM-01 — Naming methods

**Rule:** Method names **MUST** follow the Rust standard library style. Methods **MUST** use prefixes such as `as_...()`, `to_...()` or `into_...()` with the appropriate `self` convention.

**Details:**
- See also the [naming conventions](https://google.github.io/comprehensive-rust/idiomatic/foundations-api-design/predictable-api/naming-conventions.html) guidance in the Comprehensive Rust course.
- Prefix/`self` pairing is defined by the Rust API Guidelines ([C-CONV](../api-guidelines/01-naming.md#c-conv)) and checked by the Clippy lint [`wrong_self_convention`](../references/clippy-lints.md#wrong_self_convention).

**See also:** [Comprehensive Rust naming conventions](../references/comprehensive-rust-naming-conventions.md), [C-CONV](../api-guidelines/01-naming.md#c-conv), [`clippy::wrong_self_convention`](../references/clippy-lints.md#wrong_self_convention), [G-FEAT-03](12-other-rust-features.md#g-feat-03)

<a id="g-nam-02"></a>

## G-NAM-02 — Naming lifetimes

**Rule:** Lifetimes **SHOULD** have descriptive names, especially type lifetimes.

**Exceptions:**
- Type lifetimes of extremely generic abstractions like `Cow`.
- Anonymous lifetimes (`'_`) **MAY** be used wherever they work.
- Single-character lifetimes are okay when they only express the relationship between a method's input and output, or are needed for a local higher-ranked lifetime bound (`for<'a>`). Up to two such local lifetimes are fine; if more are needed, descriptive names are strongly advised.

<a id="g-nam-03"></a>

## G-NAM-03 — Naming generic parameters

**Rule:** Generic parameters **MUST** have a short name (single or multi character) related to the kind of thing they represent.

**Details:**
- Single-letter generics are okay, but the letter **SHOULD** be somewhat indicative of the object type, ideally taken from the most important trait bound.
- `T` or `U` are fine in a relatively unbound context, i.e. an API intended for use with most types, like `Vec<T>`.

<a id="g-nam-04"></a>

## G-NAM-04 — Naming extension traits

**Rule:** An [extension trait](07-traits.md#g-trt-06) name **MUST** have an `~Ext` suffix (for example, `ForeignTypeExt`).

**Example:**
```rust
// Good
trait ForeignTypeExt {
    // ...
}
```

**See also:** [G-TRT-06](07-traits.md#g-trt-06), [extension traits](../references/language-concepts.md#extension-traits)

<a id="g-nam-05"></a>

## G-NAM-05 — Naming initialization functions

**Rule:** Initialization functions **MUST** follow these naming conventions.

**Details:**
- Constructors: use `new()`, or `new_with_foo()` if there is more than one. If the constructor returns a `Result`, use `try_new()`.
- Value-to-value conversions: use `from_foo()` with an associated `From` impl. See [when to implement `From`](https://doc.rust-lang.org/std/convert/trait.From.html#when-to-implement-from) for when an operation counts as a value-to-value conversion.

**See also:** [`From` conversions](../references/language-concepts.md#from-conversions), [G-TRT-08](07-traits.md#g-trt-08)

<a id="g-nam-06"></a>

## G-NAM-06 — Avoid abbreviations

**Rule:** Names **MUST NOT** use abbreviations (especially acronyms and initialisms) that someone outside your project is unlikely to know, and **MUST NOT** abbreviate by deleting letters within a word.

**Exceptions:**
- Common abbreviations found in Wikipedia.
- Abbreviations unlikely to confuse people outside your project, for example `IpAddr`. You and your reviewer make this determination; in general the preference **SHOULD** still be not to abbreviate.

**See also:** [Google C++ Style Guide — General Naming Rules](../references/cpp-style-guide-excerpts.md) ([original](https://google.github.io/styleguide/cppguide.html#General_Naming_Rules))
