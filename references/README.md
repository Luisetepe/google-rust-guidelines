# References

> Condensed notes on pages the Google guide links to: tooling, language concepts, and related guides.

## Chapters

- [Rust API Guidelines — External links](api-guidelines-external-links.md)
- [Clippy and rustc lints cited by the Google Rust Style Guide](clippy-lints.md)
- [Comprehensive Rust — Naming Conventions](comprehensive-rust-naming-conventions.md) — 12 rules
- [Google C++ Style Guide — Excerpts Cited by the Rust Guide](cpp-style-guide-excerpts.md) — 13 rules
- [Language concepts referenced by the Google Rust Style Guide](language-concepts.md)
- [Platform Notes — Fuchsia, Chromium, google3](platform-notes.md)
- [rustfmt configuration cited by the Google Rust Style Guide](rustfmt-config.md)

## [Clippy and rustc lints cited by the Google Rust Style Guide](clippy-lints.md)

| Entry | Title |
| --- | --- |
| [`#borrowed_box`](clippy-lints.md#borrowed_box) | borrowed_box |
| [`#ptr_arg`](clippy-lints.md#ptr_arg) | ptr_arg |
| [`#should_implement_trait`](clippy-lints.md#should_implement_trait) | should_implement_trait |
| [`#new_without_default`](clippy-lints.md#new_without_default) | new_without_default |
| [`#wrong_self_convention`](clippy-lints.md#wrong_self_convention) | wrong_self_convention |
| [`#len_without_is_empty`](clippy-lints.md#len_without_is_empty) | len_without_is_empty |
| [`#exhaustive_enums`](clippy-lints.md#exhaustive_enums) | exhaustive_enums |
| [`#missing_docs`](clippy-lints.md#missing_docs) | missing_docs |

## [Comprehensive Rust — Naming Conventions](comprehensive-rust-naming-conventions.md)

| Entry | Title |
| --- | --- |
| [CR-NAM-01](comprehensive-rust-naming-conventions.md#cr-nam-01) | `new`: constructor functions |
| [CR-NAM-02](comprehensive-rust-naming-conventions.md#cr-nam-02) | `is_[condition]`: boolean check |
| [CR-NAM-03](comprehensive-rust-naming-conventions.md#cr-nam-03) | `[method]_mut`: mutable reference access |
| [CR-NAM-04](comprehensive-rust-naming-conventions.md#cr-nam-04) | `with` as constructor |
| [CR-NAM-05](comprehensive-rust-naming-conventions.md#cr-nam-05) | `with` as copy-and-set |
| [CR-NAM-06](comprehensive-rust-naming-conventions.md#cr-nam-06) | `with`: working with closures |
| [CR-NAM-07](comprehensive-rust-naming-conventions.md#cr-nam-07) | `try_[method]`: fallible methods with specific errors |
| [CR-NAM-08](comprehensive-rust-naming-conventions.md#cr-nam-08) | `from`: conversion constructors |
| [CR-NAM-09](comprehensive-rust-naming-conventions.md#cr-nam-09) | `into`: consuming conversion |
| [CR-NAM-10](comprehensive-rust-naming-conventions.md#cr-nam-10) | `to`: non-consuming conversion |
| [CR-NAM-11](comprehensive-rust-naming-conventions.md#cr-nam-11) | `as_` and `_ref`: reference conversions |
| [CR-NAM-12](comprehensive-rust-naming-conventions.md#cr-nam-12) | `by`: custom comparator or projection |

## [Google C++ Style Guide — Excerpts Cited by the Rust Guide](cpp-style-guide-excerpts.md)

| Entry | Title |
| --- | --- |
| [CPP-GOAL-01](cpp-style-guide-excerpts.md#cpp-goal-01) | Style rules should pull their weight |
| [CPP-GOAL-02](cpp-style-guide-excerpts.md#cpp-goal-02) | Optimize for the reader, not the writer |
| [CPP-GOAL-03](cpp-style-guide-excerpts.md#cpp-goal-03) | Be consistent with existing code |
| [CPP-GOAL-04](cpp-style-guide-excerpts.md#cpp-goal-04) | Be consistent with the broader C++ community when appropriate |
| [CPP-GOAL-05](cpp-style-guide-excerpts.md#cpp-goal-05) | Avoid surprising or dangerous constructs |
| [CPP-GOAL-06](cpp-style-guide-excerpts.md#cpp-goal-06) | Avoid constructs that the average programmer would find tricky or hard to maintain |
| [CPP-GOAL-07](cpp-style-guide-excerpts.md#cpp-goal-07) | Be mindful of our scale |
| [CPP-GOAL-08](cpp-style-guide-excerpts.md#cpp-goal-08) | Concede to optimization when necessary |
| [CPP-GOAL-09](cpp-style-guide-excerpts.md#cpp-goal-09) | Common sense and good taste prevail |
| [CPP-NAM-01](cpp-style-guide-excerpts.md#cpp-nam-01) | Make the purpose understandable to a new reader |
| [CPP-NAM-02](cpp-style-guide-excerpts.md#cpp-nam-02) | Scale descriptiveness with scope and context |
| [CPP-NAM-03](cpp-style-guide-excerpts.md#cpp-nam-03) | Minimize unfamiliar abbreviations |
| [CPP-NAM-04](cpp-style-guide-excerpts.md#cpp-nam-04) | Vocabulary names |

## [Language concepts referenced by the Google Rust Style Guide](language-concepts.md)

| Entry | Title |
| --- | --- |
| [`#newtype`](language-concepts.md#newtype) | Newtype |
| [`#extension-traits`](language-concepts.md#extension-traits) | Extension traits (RFC 445) |
| [`#dyn-compatibility`](language-concepts.md#dyn-compatibility) | Dyn compatibility |
| [`#inherent-impls`](language-concepts.md#inherent-impls) | Inherent implementations |
| [`#from-conversions`](language-concepts.md#from-conversions) | When to implement `From` |
| [`#turbofish`](language-concepts.md#turbofish) | Turbofish |
| [`#ctor`](language-concepts.md#ctor) | Code before `main()`: the `ctor` crate |
| [`#orphan-rules`](language-concepts.md#orphan-rules) | Orphan rules and trait coherence |
| [`#hir-grouping`](language-concepts.md#hir-grouping) | Large type groups: `rustc_hir::hir` |
| [`#cargo-layout`](language-concepts.md#cargo-layout) | Conventional Cargo package layout |

## [Platform Notes — Fuchsia, Chromium, google3](platform-notes.md)

| Entry | Title |
| --- | --- |
| [`#fuchsia-unstable`](platform-notes.md#fuchsia-unstable) | Fuchsia — Unstable Rust features |
| [`#fuchsia-testing`](platform-notes.md#fuchsia-testing) | Fuchsia — Testing Rust code |
| [`#chromium-rust-gtest-interop`](platform-notes.md#chromium-rust-gtest-interop) | Chromium — `rust_gtest_interop` |
| [`#google3`](platform-notes.md#google3) | google3 |

## [rustfmt configuration cited by the Google Rust Style Guide](rustfmt-config.md)

| Entry | Title |
| --- | --- |
| [`#max_width`](rustfmt-config.md#max_width) | max_width |
| [`#hex_literal_case`](rustfmt-config.md#hex_literal_case) | hex_literal_case |
| [`#format_macro_matchers`](rustfmt-config.md#format_macro_matchers) | format_macro_matchers |
| [`#imports_granularity`](rustfmt-config.md#imports_granularity) | imports_granularity |
| [`#rustfmt-skip`](rustfmt-config.md#rustfmt-skip) | `#[rustfmt::skip]` (attribute) |
