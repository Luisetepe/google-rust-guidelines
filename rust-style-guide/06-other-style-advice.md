# Other Style Advice

> **Source:** [Rust Style Guide — Other style advice](https://doc.rust-lang.org/nightly/style-guide/advice.html) · **License:** MIT OR Apache-2.0
>
> Non-formatting conventions: expression-oriented code, naming (case conventions and reserved words), and module paths.

<a id="rs-adv-01"></a>

## RS-ADV-01 — Expressions

**Rule:** **SHOULD** use Rust's expression-oriented nature where possible.

**Example:**
```rust
// Good
let x = if y { 1 } else { 0 };

// Bad
let x;
if y {
    x = 1;
} else {
    x = 0;
}
```

<a id="rs-adv-02"></a>

## RS-ADV-02 — Names

**Rule:** Names **MUST** follow the case conventions below. When a name is forbidden because it is a reserved word, use a raw identifier or a trailing underscore; **MUST NOT** misspell the word.

**Details:**
- Types: `UpperCamelCase`.
- Enum variants: `UpperCamelCase`.
- Struct fields: `snake_case`.
- Function and method names: `snake_case`.
- Local variables: `snake_case`.
- Macro names: `snake_case`.
- Constants (`const`s and immutable `static`s): `SCREAMING_SNAKE_CASE`.
- Reserved words (such as `crate`): use a raw identifier (`r#crate`) or a trailing underscore (`crate_`); don't misspell the word (`krate`).

**Google note:** Google adds naming rules for methods (`as_`/`to_`/`into_`, getters), lifetimes, generic parameters, extension traits, initialization functions, and abbreviations; see [Google — Naming](../google/10-naming.md), e.g. [G-NAM-01](../google/10-naming.md#g-nam-01), [G-NAM-06](../google/10-naming.md#g-nam-06).

**Example:**
```rust
// Bad: misspelled reserved word
let krate = 1;

// Good
let r#crate = 1;
let crate_ = 1;
```

**See also:** [C-CASE](../api-guidelines/01-naming.md#c-case)

<a id="rs-adv-03"></a>

## RS-ADV-03 — Modules

**Rule:** **SHOULD** avoid `#[path]` annotations where possible.

**See also:** [G-ORG-03](../google/02-code-organization.md#g-org-03)
