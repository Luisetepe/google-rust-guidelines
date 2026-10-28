# Rust Style Editions

> **Source:** [Rust Style Guide — Rust style editions](https://doc.rust-lang.org/nightly/style-guide/editions.html) · **License:** MIT OR Apache-2.0
>
> How the default Rust style evolves through style editions, and what changed in each.

## How style editions work

- The default Rust style evolves over time, but changes to it only appear in *style editions*, to avoid breaking established code style and CI jobs that check it.
- Code written in a given [Rust edition](https://doc.rust-lang.org/edition-guide/) uses the corresponding style edition by default. Formatting tools such as `rustfmt` allow updating the style edition separately from the Rust edition, so code style can be migrated separately from semantic edition changes (see [rustfmt configuration](../references/rustfmt-config.md)).
- The current style guide describes the latest style edition; each distinct past style has an archived version of the guide.
- Archived guides do not document formatting for Rust constructs newer than the archive. Each style edition still formats all constructs valid in its Rust edition; the style of newer constructs comes from the first subsequent style edition that provides formatting rules for that construct (without any of that edition's systematic/global changes).
- Not every Rust edition changes the style: Rust 2015, 2018 and 2021 all use the same style edition.

<a id="rs-ed-01"></a>

## RS-ED-01 — Rust next style edition

**Rule:** **MUST NOT** break within a nullary function call `func()` or a unit literal `()`.

**Applies to:** the next (post-2024) style edition.

**Example:**
```rust
// Bad
func(
)

// Good
func()
```

<a id="rs-ed-02"></a>

## RS-ED-02 — Rust 2024 style edition

**Rule:** This style guide describes the Rust 2024 style edition, which (per the source) is currently nightly-only and may change before the release of Rust 2024.

**Applies to:** Rust 2024 style edition.

**Details:**
- Full change history: the git history of the style guide. Notable changes:
  - Miscellaneous `rustfmt` bugfixes.
  - Use version-sort (sort `x8`, `x16`, `x32`, `x64`, `x128` in that order); see [Formatting conventions](01-formatting-conventions.md).
  - Change "ASCIIbetical" sort to Unicode-aware "non-lowercase before lowercase".

**Example:**
```rust
// Rust 2024 style (version sort)
use x::{x8, x16, x32, x64, x128};
```

<a id="rs-ed-03"></a>

## RS-ED-03 — Rust 2015/2018/2021 style edition

**Rule:** The style edition shared by Rust 2015, Rust 2018 and Rust 2021 is described by the [archived style guide](https://github.com/rust-lang/rust/tree/37343f4a4d4ed7ad0891cb79e8eb25acf43fb821/src/doc/style-guide/src).

**Applies to:** Rust 2015, 2018 and 2021 style edition.
