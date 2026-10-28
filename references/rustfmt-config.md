# rustfmt configuration cited by the Google Rust Style Guide

> **Source:** [rustfmt — Configuring Rustfmt (`Configurations.md`)](https://raw.githubusercontent.com/rust-lang/rustfmt/master/Configurations.md), [rustfmt README](https://github.com/rust-lang/rustfmt/blob/master/README.md), [rustfmt CHANGELOG](https://github.com/rust-lang/rustfmt/blob/master/CHANGELOG.md) · **License:** MIT OR Apache-2.0
>
> Every rustfmt option (and the `#[rustfmt::skip]` attribute) the Google guide mentions: default, possible values, stability, Google's value and examples, plus a Google-aligned `rustfmt.toml`.

rustfmt reads `rustfmt.toml` or `.rustfmt.toml` from the project or any parent directory (falling back to the home directory and the `rustfmt` directory in the global config directory, e.g. `.config/rustfmt/`). Each option is stable or unstable: stable options can always be used; unstable options are only available on a nightly toolchain and must be opted into (`unstable_features = true` in `rustfmt.toml`, or `--unstable-features`). The Google guide links option docs at rustfmt `v1.6.0` (`https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=#<option>`); stability below is from rustfmt master, with differences from v1.6.0 noted.

Per [G-FMT-01](../google/03-formatting.md#g-fmt-01), a codebase **MAY** choose not to apply `max_width`, `hex_literal_case` and `format_macro_matchers` if it cares about building on stable and the option is unavailable on its stable Rust.

<a id="max_width"></a>

## max_width

**Source:** [`max_width`](https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=#max_width)

Maximum width of each line.

- **Default:** `100`
- **Possible values:** any nonnegative integer
- **Stable:** Yes (also in v1.6.0)
- **Google value:** `max_width = 100` (maximum of 100 columns per line); identical to the default.
- See also upstream option `error_on_line_overflow`. Other width options (e.g. `array_width`) are by default derived as a percentage of `max_width` via `use_small_heuristics`.

**Cited by:** [G-FMT-01](../google/03-formatting.md#g-fmt-01).

<a id="hex_literal_case"></a>

## hex_literal_case

**Source:** [`hex_literal_case`](https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=#hex_literal_case)

Controls the case of the letters in hexadecimal literal values.

- **Default:** `Preserve`
- **Possible values:** `Preserve`, `Upper`, `Lower`
- **Stable:** Yes on rustfmt master (stabilized in [#6935](https://github.com/rust-lang/rustfmt/pull/6935), listed under rustfmt 1.10.0 in the CHANGELOG). No in v1.6.0 (tracking issue [#5081](https://github.com/rust-lang/rustfmt/issues/5081)): older toolchains need nightly rustfmt.
- **Google value:** `hex_literal_case = "Lower"` (lowercase letters in hex literals: `0xab123cdef`, not `0xAB123CDEF`).

**Example** (with `Lower`):
```rust
// Before
let x = 0xAB123CDEF;

// After
let x = 0xab123cdef;
```

**Cited by:** [G-FMT-01](../google/03-formatting.md#g-fmt-01).

<a id="format_macro_matchers"></a>

## format_macro_matchers

**Source:** [`format_macro_matchers`](https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=#format_macro_matchers)

Formats the metavariable matching patterns in macros.

- **Default:** `false`
- **Possible values:** `true`, `false`
- **Stable:** No (tracking issue [#3354](https://github.com/rust-lang/rustfmt/issues/3354)); nightly only, also in v1.6.0.
- **Google value:** `format_macro_matchers = true`.
- See also upstream option `format_macro_bodies`.

**Example:**
```rust
// `false` (default)
macro_rules! foo {
    ($a: ident : $b: ty) => {
        $a(42): $b;
    };
    ($a: ident $b: ident $c: ident) => {
        $a = $b + $c;
    };
}

// `true` (Google)
macro_rules! foo {
    ($a:ident : $b:ty) => {
        $a(42): $b;
    };
    ($a:ident $b:ident $c:ident) => {
        $a = $b + $c;
    };
}
```

**Cited by:** [G-FMT-01](../google/03-formatting.md#g-fmt-01).

<a id="imports_granularity"></a>

## imports_granularity

**Source:** [`imports_granularity`](https://rust-lang.github.io/rustfmt/?version=v1.6.0&search=#imports_granularity)

Controls how imports are structured in `use` statements; imports are merged or split to the configured granularity.

- **Default:** `Preserve`
- **Possible values:** `Preserve`, `Crate`, `Module`, `Item`, `One`
- **Stable:** No (tracking issue [#4991](https://github.com/rust-lang/rustfmt/issues/4991)); nightly only.
- **Google value:** `imports_granularity = "Module"` (a `use` statement imports items from one module; no `::` inside the `{..}` block).
- Operates within the bounds of user-defined import groups (see upstream option `group_imports`).
- rustfmt will not change the granularity of imports containing comments if that could lose or misplace the comments.

**Values:**
- `Preserve` (default): do not change granularity; keep the structure the developer wrote.
- `Crate`: merge imports from the same crate into one `use`; split imports from different crates.
- `Module`: merge imports from the same module into one `use`; split imports from different modules. Does not merge top-level modules.
- `Item`: flatten so each import has its own `use`.
- `One`: merge all imports into a single `use` as long as they have the same visibility.

**Example:**
```rust
// Preserve (input kept as written)
use foo::b;
use foo::b::{f, g};
use foo::{a, c, d::e};
use qux::{h, i};

// Crate
use foo::{
    a, b,
    b::{f, g},
    c,
    d::e,
};
use qux::{h, i};

// Module (Google)
use foo::b::{f, g};
use foo::d::e;
use foo::{a, b, c};
use qux::{h, i};

// Item
use foo::a;
use foo::b;
use foo::b::f;
use foo::b::g;
use foo::c;
use foo::d::e;
use qux::h;
use qux::i;

// One
pub use foo::{x, y};
use {
    bar::{
        a,
        b::{self, f, g},
        c,
        d::e,
    },
    qux::{h, i},
};
```

**Cited by:** [G-ORG-06](../google/02-code-organization.md#g-org-06) ("This rule corresponds to the rustfmt setting `imports_granularity = Module`").

<a id="rustfmt-skip"></a>

## `#[rustfmt::skip]` (attribute)

**Source:** [rustfmt README — Tips](https://github.com/rust-lang/rustfmt/blob/master/README.md#tips)

Not a `rustfmt.toml` option: an attribute that tells rustfmt not to format the annotated item ("for things you do not want rustfmt to mangle"). To skip only a given macro or attribute, use `#[rustfmt::skip::macros(target_macro_name)]` or `#[rustfmt::skip::attributes(target_attribute_name)]`.

```rust
#![rustfmt::skip::attributes(custom_attribute)]

#[custom_attribute(formatting , here , should , be , Skipped)]
#[rustfmt::skip::macros(html)]
fn main() {
    let macro_result1 = html! { <div>
Hello</div>
    }.to_string();
}
```

**Google restriction:** [G-FMT-02](../google/03-formatting.md#g-fmt-02) allows `#[rustfmt::skip]` only for manual alignment (situation-specific reason, e.g. aligned similar lines or a matrix-constructing function) and generated code (ideally use a genrule instead); it should not be skipped for personal style choices.

## Escapes for invisible characters (no rustfmt option)

[G-FMT-01](../google/03-formatting.md#g-fmt-01) also prescribes escapes for invisible characters in literals (e.g. a Zero-Width Joiner written as `\u{200D}`). rustfmt's `Configurations.md` has no option for this; it is a manual convention.

## Google-aligned rustfmt.toml

Exactly the settings the Google guide prescribes ([G-FMT-01](../google/03-formatting.md#g-fmt-01), [G-ORG-06](../google/02-code-organization.md#g-org-06)):

```toml
# Stable.
max_width = 100

# Stable on recent rustfmt (1.10.0+ per CHANGELOG); nightly-only on older
# toolchains (e.g. v1.6.0, the version the Google guide links).
hex_literal_case = "Lower"

# Unstable: requires nightly rustfmt.
format_macro_matchers = true

# Unstable: requires nightly rustfmt.
imports_granularity = "Module"
```

- Unstable options take effect only with nightly rustfmt (e.g. `cargo +nightly fmt`); per `Configurations.md` they must also be opted into via `unstable_features = true` or `--unstable-features`.
- A codebase that must build on stable **MAY** omit `max_width`, `hex_literal_case` and `format_macro_matchers` where unavailable ([G-FMT-01](../google/03-formatting.md#g-fmt-01)). The guide gives no such stable-toolchain allowance for `imports_granularity`; it states the rule in [G-ORG-06](../google/02-code-organization.md#g-org-06) independently of the tool.
