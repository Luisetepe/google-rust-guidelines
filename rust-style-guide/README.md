# The Rust Style Guide

> **Source:** [The Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/)
>
> The default Rust style, enforced by `rustfmt`. Applies unless the Google guide overrides it (see **Google note** entries).

## Chapters

- [Formatting Conventions](01-formatting-conventions.md) — 10 rules
- [Items](02-items.md) — 24 rules
- [Statements](03-statements.md) — 4 rules
- [Expressions](04-expressions.md) — 29 rules
- [Types and Bounds](05-types-and-bounds.md) — 3 rules
- [Other Style Advice](06-other-style-advice.md) — 3 rules
- [Cargo.toml Conventions](07-cargo-toml.md) — 2 rules
- [Guiding Principles and Rationale](08-guiding-principles.md) — 4 rules
- [Rust Style Editions](09-style-editions.md) — 3 rules
- [Nightly-only Syntax](10-nightly-only-syntax.md) — 1 rule

## [Formatting Conventions](01-formatting-conventions.md)

| Entry | Title |
| --- | --- |
| [RS-FMT-01](01-formatting-conventions.md#rs-fmt-01) | Indentation and line width |
| [RS-FMT-02](01-formatting-conventions.md#rs-fmt-02) | Block indent |
| [RS-FMT-03](01-formatting-conventions.md#rs-fmt-03) | Trailing commas |
| [RS-FMT-04](01-formatting-conventions.md#rs-fmt-04) | Blank lines |
| [RS-FMT-05](01-formatting-conventions.md#rs-fmt-05) | Trailing whitespace |
| [RS-FMT-06](01-formatting-conventions.md#rs-fmt-06) | Sorting (version sorting) |
| [RS-FMT-07](01-formatting-conventions.md#rs-fmt-07) | Comments |
| [RS-FMT-08](01-formatting-conventions.md#rs-fmt-08) | Doc comments |
| [RS-FMT-09](01-formatting-conventions.md#rs-fmt-09) | Attributes |
| [RS-FMT-10](01-formatting-conventions.md#rs-fmt-10) | *Small* items |

## [Items](02-items.md)

| Entry | Title |
| --- | --- |
| [RS-ITEM-01](02-items.md#rs-item-01) | Item ordering |
| [RS-ITEM-02](02-items.md#rs-item-02) | Function definitions |
| [RS-ITEM-03](02-items.md#rs-item-03) | Tuples and tuple structs |
| [RS-ITEM-04](02-items.md#rs-item-04) | Tuples and tuple structs: single-line |
| [RS-ITEM-05](02-items.md#rs-item-05) | Enums |
| [RS-ITEM-06](02-items.md#rs-item-06) | Structs and unions |
| [RS-ITEM-07](02-items.md#rs-item-07) | Tuple structs |
| [RS-ITEM-08](02-items.md#rs-item-08) | Traits |
| [RS-ITEM-09](02-items.md#rs-item-09) | Impls |
| [RS-ITEM-10](02-items.md#rs-item-10) | Extern crate |
| [RS-ITEM-11](02-items.md#rs-item-11) | Modules |
| [RS-ITEM-12](02-items.md#rs-item-12) | `macro_rules!` |
| [RS-ITEM-13](02-items.md#rs-item-13) | Generics |
| [RS-ITEM-14](02-items.md#rs-item-14) | `where` clauses |
| [RS-ITEM-15](02-items.md#rs-item-15) | Type aliases |
| [RS-ITEM-16](02-items.md#rs-item-16) | Associated types |
| [RS-ITEM-17](02-items.md#rs-item-17) | Extern items |
| [RS-ITEM-18](02-items.md#rs-item-18) | Imports (`use` statements) |
| [RS-ITEM-19](02-items.md#rs-item-19) | Large list imports |
| [RS-ITEM-20](02-items.md#rs-item-20) | Ordering of imports |
| [RS-ITEM-21](02-items.md#rs-item-21) | Ordering list import |
| [RS-ITEM-22](02-items.md#rs-item-22) | Normalisation |
| [RS-ITEM-23](02-items.md#rs-item-23) | Nested imports |
| [RS-ITEM-24](02-items.md#rs-item-24) | Merging/un-merging imports |

## [Statements](03-statements.md)

| Entry | Title |
| --- | --- |
| [RS-STMT-01](03-statements.md#rs-stmt-01) | Let statements |
| [RS-STMT-02](03-statements.md#rs-stmt-02) | else blocks (let-else statements) |
| [RS-STMT-03](03-statements.md#rs-stmt-03) | Macros in statement position |
| [RS-STMT-04](03-statements.md#rs-stmt-04) | Expressions in statement position |

## [Expressions](04-expressions.md)

| Entry | Title |
| --- | --- |
| [RS-EXPR-01](04-expressions.md#rs-expr-01) | Blocks |
| [RS-EXPR-02](04-expressions.md#rs-expr-02) | Closures |
| [RS-EXPR-03](04-expressions.md#rs-expr-03) | Struct literals |
| [RS-EXPR-04](04-expressions.md#rs-expr-04) | Unit literals |
| [RS-EXPR-05](04-expressions.md#rs-expr-05) | Tuple literals |
| [RS-EXPR-06](04-expressions.md#rs-expr-06) | Tuple struct literals |
| [RS-EXPR-07](04-expressions.md#rs-expr-07) | Enum literals |
| [RS-EXPR-08](04-expressions.md#rs-expr-08) | Array literals |
| [RS-EXPR-09](04-expressions.md#rs-expr-09) | Array accesses, indexing, and slicing |
| [RS-EXPR-10](04-expressions.md#rs-expr-10) | Unary operations |
| [RS-EXPR-11](04-expressions.md#rs-expr-11) | Binary operations |
| [RS-EXPR-12](04-expressions.md#rs-expr-12) | Casts (`as`) |
| [RS-EXPR-13](04-expressions.md#rs-expr-13) | Control flow |
| [RS-EXPR-14](04-expressions.md#rs-expr-14) | Function calls |
| [RS-EXPR-15](04-expressions.md#rs-expr-15) | Single-line calls |
| [RS-EXPR-16](04-expressions.md#rs-expr-16) | Multi-line calls |
| [RS-EXPR-17](04-expressions.md#rs-expr-17) | Method calls |
| [RS-EXPR-18](04-expressions.md#rs-expr-18) | Macro uses |
| [RS-EXPR-19](04-expressions.md#rs-expr-19) | Format string macros |
| [RS-EXPR-20](04-expressions.md#rs-expr-20) | Chains of fields and method calls |
| [RS-EXPR-21](04-expressions.md#rs-expr-21) | Multi-line chain elements |
| [RS-EXPR-22](04-expressions.md#rs-expr-22) | Control flow expressions |
| [RS-EXPR-23](04-expressions.md#rs-expr-23) | Single line `if else` |
| [RS-EXPR-24](04-expressions.md#rs-expr-24) | Match |
| [RS-EXPR-25](04-expressions.md#rs-expr-25) | Match arm line-breaking |
| [RS-EXPR-26](04-expressions.md#rs-expr-26) | Combinable expressions |
| [RS-EXPR-27](04-expressions.md#rs-expr-27) | Ranges |
| [RS-EXPR-28](04-expressions.md#rs-expr-28) | Hexadecimal literals |
| [RS-EXPR-29](04-expressions.md#rs-expr-29) | Patterns |

## [Types and Bounds](05-types-and-bounds.md)

| Entry | Title |
| --- | --- |
| [RS-TYPE-01](05-types-and-bounds.md#rs-type-01) | Single line formatting |
| [RS-TYPE-02](05-types-and-bounds.md#rs-type-02) | Line breaks |
| [RS-TYPE-03](05-types-and-bounds.md#rs-type-03) | Precise capturing bounds |

## [Other Style Advice](06-other-style-advice.md)

| Entry | Title |
| --- | --- |
| [RS-ADV-01](06-other-style-advice.md#rs-adv-01) | Expressions |
| [RS-ADV-02](06-other-style-advice.md#rs-adv-02) | Names |
| [RS-ADV-03](06-other-style-advice.md#rs-adv-03) | Modules |

## [Cargo.toml Conventions](07-cargo-toml.md)

| Entry | Title |
| --- | --- |
| [RS-CARGO-01](07-cargo-toml.md#rs-cargo-01) | Formatting conventions |
| [RS-CARGO-02](07-cargo-toml.md#rs-cargo-02) | Metadata conventions |

## [Guiding Principles and Rationale](08-guiding-principles.md)

| Entry | Title |
| --- | --- |
| [RS-PRIN-01](08-guiding-principles.md#rs-prin-01) | Readability |
| [RS-PRIN-02](08-guiding-principles.md#rs-prin-02) | Aesthetics |
| [RS-PRIN-03](08-guiding-principles.md#rs-prin-03) | Specifics |
| [RS-PRIN-04](08-guiding-principles.md#rs-prin-04) | Application |

## [Rust Style Editions](09-style-editions.md)

| Entry | Title |
| --- | --- |
| [RS-ED-01](09-style-editions.md#rs-ed-01) | Rust next style edition |
| [RS-ED-02](09-style-editions.md#rs-ed-02) | Rust 2024 style edition |
| [RS-ED-03](09-style-editions.md#rs-ed-03) | Rust 2015/2018/2021 style edition |

## [Nightly-only Syntax](10-nightly-only-syntax.md)

| Entry | Title |
| --- | --- |
| [RS-NIGHT-01](10-nightly-only-syntax.md#rs-night-01) | Frontmatter |
