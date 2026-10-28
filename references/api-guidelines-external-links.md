# Rust API Guidelines — External links

> **Source:** [Rust API Guidelines — External links](https://rust-lang.github.io/api-guidelines/external-links.html) · **License:** MIT OR Apache-2.0
>
> The external documents the Rust API Guidelines list as further reading, each with a one-line summary.

- [RFC 199](https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md) — Ownership naming conventions: how to name by-value, by-reference and by-mutable-reference variants of an operation (e.g. the `_mut` suffix).
- [RFC 344](https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md) — Naming conventions: type names in method names, iterator type and method names, getters/setters, associated types, trait naming, lint naming, suffix ordering, prelude traits; lowercase compiler/std error messages.
- [RFC 430](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md) — Naming conventions: finalizes capitalization (`UpperCamelCase` for type-level, `snake_case` for value-level constructs) and the role of `unwrap`.
- [RFC 505](https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md) — Doc conventions: guidance for writing API documentation (e.g. use line comments, not block comments).
- [RFC 1574](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md) — Doc conventions: augments RFC 505 (full combined text included), e.g. the standard section headings Examples, Panics, Errors, Safety, Aborts, Undefined Behavior.
- [RFC 1687](https://github.com/rust-lang/rfcs/pull/1687) — Crate-level documentation: RFC PR "Add API documentation front page styleguide" proposing guidelines for a crate's documentation front page (closed, not merged).
- [Elegant Library APIs in Rust](https://deterministic.space/elegant-apis-in-rust.html) — Article on designing idiomatic, ergonomic Rust library APIs.
- [Rust Design Patterns](https://rust-unofficial.github.io/patterns/) — Community catalogue of Rust idioms, design patterns and anti-patterns.

The source page also defines a link target for [RFC 1105](https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md) (API evolution: which changes to stable APIs are semver-breaking) without listing it.
