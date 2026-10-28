# Code Formatting and Aesthetics

> **Source:** [Google Rust Style Guide — Code formatting and aesthetics](https://google.github.io/styleguide/rustguide.html#code-formatting) · **License:** CC-BY 3.0
>
> Which formatting tool and rustfmt configuration to use, and when automatic formatting may be skipped.

Formatting details not covered here follow the [Rust Style Guide](../rust-style-guide/01-formatting-conventions.md) ([G-BG-01](01-background-and-scope.md#g-bg-01)), most of which rustfmt enforces automatically.

<a id="g-fmt-01"></a>

## G-FMT-01 — Use the appropriate formatting tool

**Rule:** Code **MUST** be formatted with the codebase's appropriate formatting tool (e.g., `git cl format` in Chromium).

**Details:**
- Configuration (for style guide authors) — the `rustfmt` options Google applies:
  - Maximum of 100 columns per line: [`max_width = 100`](../references/rustfmt-config.md#max_width).
  - Lowercase letters in hex literals (`0xab123cdef`, not `0xAB123CDEF`): [`hex_literal_case = Lower`](../references/rustfmt-config.md#hex_literal_case).
  - Macro matcher formatting: [`format_macro_matchers = true`](../references/rustfmt-config.md#format_macro_matchers).
  - Escapes for invisible characters in literals: e.g., text containing a Zero-Width Joiner uses `\u{200D}`.

**Exceptions:**
- A codebase **MAY** choose not to apply any of these `rustfmt` options if it cares about building on stable and the option is unavailable on its stable Rust.

**Example:**
```toml
# rustfmt.toml
max_width = 100
hex_literal_case = "Lower"
format_macro_matchers = true
```

```rust
// Bad: uppercase hex; Zero-Width Joiner typed literally (invisible in source)
const MASK: u64 = 0xAB123CDEF;
// const JOINED: &str = "a<literal U+200D here>b";

// Good
const MASK: u64 = 0xab123cdef;
const JOINED: &str = "a\u{200D}b";
```

**See also:** [rustfmt option docs (v1.6.0)](https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=), [Rust Style Guide — formatting conventions](../rust-style-guide/01-formatting-conventions.md), [G-FMT-02](#g-fmt-02)

<a id="g-fmt-02"></a>

## G-FMT-02 — Opting out of automatic formatting

**Rule:** You **MAY** opt out of automatic formatting with `#[rustfmt::skip]`, but only in the situations listed below. Formatting **SHOULD NOT** be skipped for personal style choices.

**Details:**
- **Manual alignment:** a situation-specific reason for a snippet to line up in a particular way (e.g., multiple similar lines you want to stay aligned for readability, or a matrix-constructing function).
- **Generated code** (ideally, use a genrule instead).

**Example:**
```rust
// OK: manual alignment of a matrix
#[rustfmt::skip]
fn identity() -> [[f32; 3]; 3] {
    [
        [1.0, 0.0, 0.0],
        [0.0, 1.0, 0.0],
        [0.0, 0.0, 1.0],
    ]
}
```

**See also:** [G-FMT-01](#g-fmt-01)
