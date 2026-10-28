# Debuggability

> **Source:** [Rust API Guidelines — Debuggability](https://rust-lang.github.io/api-guidelines/debuggability.html) · **License:** MIT OR Apache-2.0
>
> Guidelines that make a crate conducive to easy debugging through `Debug` implementations.

<a id="c-debug"></a>

## C-DEBUG — All public types implement `Debug`

**Rule:** All public types **SHOULD** implement `Debug`.

**Exceptions:**
- Exceptions, if any, are rare.

**Google note:** Google scopes proactive trait impls to low-level library code; application code adds impls as needed ([G-TRT-01](../google/07-traits.md#g-trt-01)). Prefer `#[derive(Debug)]` over a manual impl when possible ([G-TRT-03](../google/07-traits.md#g-trt-03)); per-trait guidance is in [G-TRT-07](../google/07-traits.md#g-trt-07).

**See also:** [C-COMMON-TRAITS](02-interoperability.md#c-common-traits), [C-DEBUG-NONEMPTY](#c-debug-nonempty)

<a id="c-debug-nonempty"></a>

## C-DEBUG-NONEMPTY — `Debug` representation is never empty

**Rule:** The `Debug` representation **SHOULD NOT** ever be empty, even for conceptually empty values.

**Example:**
```rust
let empty_str = "";
assert_eq!(format!("{:?}", empty_str), "\"\"");

let empty_vec = Vec::<bool>::new();
assert_eq!(format!("{:?}", empty_vec), "[]");
```

**See also:** [C-DEBUG](#c-debug)
