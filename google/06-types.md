# Types

> **Source:** [Google Rust Style Guide — Types](https://google.github.io/styleguide/rustguide.html#types) · **License:** CC-BY 3.0
>
> When to define a newtype instead of a type alias, and when to prefer named fields over tuples.

<a id="g-typ-01"></a>

## G-TYP-01 — Newtypes versus type aliases

**Rule:** You **MUST** use a [newtype](https://doc.rust-lang.org/rust-by-example/generics/new_types.html) instead of a type alias in the cases listed below.

**Details:**
- The type you are defining has notable semantic differences.
- The type warrants an API of its own.
- You want the flexibility to change the underlying type later, especially if you don't control all usages of your type.
- For type safety, if it's easy for the types to get mixed up. E.g. a `Date` type may use newtypes to distinguish `Day`, `Month`, and `Year` numbers in its internal calculations.

**Example:**
```rust
// Bad: aliases are interchangeable; day and month are easily swapped.
type Day = u8;
type Month = u8;

// Good: distinct types; mixing them up is a compile error.
struct Day(u8);
struct Month(u8);
struct Year(u16);
```

**See also:** [Newtype](../references/language-concepts.md#newtype), [C-NEWTYPE](../api-guidelines/07-type-safety.md#c-newtype), [C-NEWTYPE-HIDE](../api-guidelines/10-future-proofing.md#c-newtype-hide), [G-TYP-02](#g-typ-02)

<a id="g-typ-02"></a>

## G-TYP-02 — Prefer named fields over tuples and non-newtype tuple structs

**Rule:** You **SHOULD** prefer types with named fields over tuples, as tuple indexes (`.0`, `.1`, ...) quickly become confusing.

**Exceptions:**
- Newtype tuple structs (tuple structs with only one field).
- Very local contexts where there is unlikely to be any confusion.

**Example:**
```rust
// Bad: what are .0 and .1?
fn dimensions() -> (u32, u32) { ... }
struct Size(u32, u32);

// Good
struct Size {
    width: u32,
    height: u32,
}
fn dimensions() -> Size { ... }
```

**See also:** [G-TYP-01](#g-typ-01)
