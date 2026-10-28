# Google Rust Style Guide

> **Source:** [Google Rust Style Guide](https://google.github.io/styleguide/rustguide.html)
>
> Google's own Rust rules. Binding for library code; strong recommendation for application code. Overrides the Rust Style Guide and API Guidelines on conflict.

## Chapters

- [Background and Scope](01-background-and-scope.md) — 2 rules
- [Code Organization](02-code-organization.md) — 10 rules
- [Code Formatting and Aesthetics](03-formatting.md) — 2 rules
- [Ownership and borrowing](04-ownership-and-borrowing.md) — 2 rules
- [Function inputs and outputs](05-function-inputs-and-outputs.md) — 6 rules
- [Types](06-types.md) — 2 rules
- [Traits](07-traits.md) — 9 rules
- [Concurrency](08-concurrency.md) — 4 rules
- [Macros](09-macros.md) — 2 rules
- [Naming](10-naming.md) — 6 rules
- [When and how to write documentation](11-documentation.md) — 12 rules
- [Other Rust features](12-other-rust-features.md) — 12 rules
- [Implementation concerns](13-implementation-concerns.md) — 8 rules
- [Testing](14-testing.md) — 1 rule

## [Background and Scope](01-background-and-scope.md)

| Entry | Title |
| --- | --- |
| [G-BG-01](01-background-and-scope.md#g-bg-01) | Follow public Rust style rules |
| [G-BG-02](01-background-and-scope.md#g-bg-02) | Apply API design rules in context |

## [Code Organization](02-code-organization.md)

| Entry | Title |
| --- | --- |
| [G-ORG-01](02-code-organization.md#g-org-01) | Prefer small crates `[chrome]` |
| [G-ORG-02](02-code-organization.md#g-org-02) | Use flat file layout for `rust_library` sources `[chrome]` |
| [G-ORG-03](02-code-organization.md#g-org-03) | Name source files and leaf modules after the primary user-defined item |
| [G-ORG-04](02-code-organization.md#g-org-04) | Don't implement APIs in non-leaf modules |
| [G-ORG-05](02-code-organization.md#g-org-05) | Don't use glob imports |
| [G-ORG-06](02-code-organization.md#g-org-06) | A `use` statement should import items from one module |
| [G-ORG-07](02-code-organization.md#g-org-07) | Prefer locally-scoped trait imports |
| [G-ORG-08](02-code-organization.md#g-org-08) | Qualify calls to top-level functions with a module name |
| [G-ORG-09](02-code-organization.md#g-org-09) | Type names should be imported and used unqualified |
| [G-ORG-10](02-code-organization.md#g-org-10) | Use the appropriate import mechanism `[chrome]` |

## [Code Formatting and Aesthetics](03-formatting.md)

| Entry | Title |
| --- | --- |
| [G-FMT-01](03-formatting.md#g-fmt-01) | Use the appropriate formatting tool |
| [G-FMT-02](03-formatting.md#g-fmt-02) | Opting out of automatic formatting |

## [Ownership and borrowing](04-ownership-and-borrowing.md)

| Entry | Title |
| --- | --- |
| [G-OWN-01](04-ownership-and-borrowing.md#g-own-01) | Avoid `Rc` and `Arc` |
| [G-OWN-02](04-ownership-and-borrowing.md#g-own-02) | Prefer borrows over deep copies |

## [Function inputs and outputs](05-function-inputs-and-outputs.md)

| Entry | Title |
| --- | --- |
| [G-FIO-01](05-function-inputs-and-outputs.md#g-fio-01) | Accept flexible references for immutable data |
| [G-FIO-02](05-function-inputs-and-outputs.md#g-fio-02) | Iterators vs. owned collections in return types |
| [G-FIO-03](05-function-inputs-and-outputs.md#g-fio-03) | Iterators vs. slices in function parameters |
| [G-FIO-04](05-function-inputs-and-outputs.md#g-fio-04) | Accepting `Into` |
| [G-FIO-05](05-function-inputs-and-outputs.md#g-fio-05) | Prefer `enum` over `bool` in APIs |
| [G-FIO-06](05-function-inputs-and-outputs.md#g-fio-06) | Reporting errors |

## [Types](06-types.md)

| Entry | Title |
| --- | --- |
| [G-TYP-01](06-types.md#g-typ-01) | Newtypes versus type aliases |
| [G-TYP-02](06-types.md#g-typ-02) | Prefer named fields over tuples and non-newtype tuple structs |

## [Traits](07-traits.md)

| Entry | Title |
| --- | --- |
| [G-TRT-01](07-traits.md#g-trt-01) | Add trait implementations when they become necessary |
| [G-TRT-02](07-traits.md#g-trt-02) | Implement known standard library traits instead of custom methods |
| [G-TRT-03](07-traits.md#g-trt-03) | Use `#[derive()]` if possible |
| [G-TRT-04](07-traits.md#g-trt-04) | Generics versus `dyn Trait` |
| [G-TRT-05](07-traits.md#g-trt-05) | When to define a trait versus an inherent implementation |
| [G-TRT-06](07-traits.md#g-trt-06) | Extension traits |
| [G-TRT-07](07-traits.md#g-trt-07) | Specific guidance for important traits |
| [G-TRT-08](07-traits.md#g-trt-08) | Pair important `From` impls with explicit constructor methods |
| [G-TRT-09](07-traits.md#g-trt-09) | Options structs should implement Default |

## [Concurrency](08-concurrency.md)

| Entry | Title |
| --- | --- |
| [G-CON-01](08-concurrency.md#g-con-01) | Handling poison |
| [G-CON-02](08-concurrency.md#g-con-02) | Don't use `async` / `.await` `[chrome]` |
| [G-CON-03](08-concurrency.md#g-con-03) | Don't start new threads through the Rust standard library `[chrome]` |
| [G-CON-04](08-concurrency.md#g-con-04) | Use `fuchsia_sync::Mutex` `[Fuchsia]` |

## [Macros](09-macros.md)

| Entry | Title |
| --- | --- |
| [G-MAC-01](09-macros.md#g-mac-01) | Prefer non-macro facilities, in order |
| [G-MAC-02](09-macros.md#g-mac-02) | Weigh the costs of macros and code generation |

## [Naming](10-naming.md)

| Entry | Title |
| --- | --- |
| [G-NAM-01](10-naming.md#g-nam-01) | Naming methods |
| [G-NAM-02](10-naming.md#g-nam-02) | Naming lifetimes |
| [G-NAM-03](10-naming.md#g-nam-03) | Naming generic parameters |
| [G-NAM-04](10-naming.md#g-nam-04) | Naming extension traits |
| [G-NAM-05](10-naming.md#g-nam-05) | Naming initialization functions |
| [G-NAM-06](10-naming.md#g-nam-06) | Avoid abbreviations |

## [When and how to write documentation](11-documentation.md)

| Entry | Title |
| --- | --- |
| [G-DOC-01](11-documentation.md#g-doc-01) | Document public APIs |
| [G-DOC-02](11-documentation.md#g-doc-02) | Consider including examples |
| [G-DOC-03](11-documentation.md#g-doc-03) | Document panic cases |
| [G-DOC-04](11-documentation.md#g-doc-04) | Document invariants of unsafe functions and traits |
| [G-DOC-05](11-documentation.md#g-doc-05) | Use capitalization and punctuation |
| [G-DOC-06](11-documentation.md#g-doc-06) | Don't explain language basics |
| [G-DOC-07](11-documentation.md#g-doc-07) | Use `///` for item documentation |
| [G-DOC-08](11-documentation.md#g-doc-08) | Use `//!` for module and library documentation |
| [G-DOC-09](11-documentation.md#g-doc-09) | Follow the Rust style guide's doc-comment rules |
| [G-DOC-10](11-documentation.md#g-doc-10) | Use Markdown syntax |
| [G-DOC-11](11-documentation.md#g-doc-11) | Use `//` for non-documentation comments |
| [G-DOC-12](11-documentation.md#g-doc-12) | Prefer direct doc comments over macro-generated docs |

## [Other Rust features](12-other-rust-features.md)

| Entry | Title |
| --- | --- |
| [G-FEAT-01](12-other-rust-features.md#g-feat-01) | Unsafe code |
| [G-FEAT-02](12-other-rust-features.md#g-feat-02) | No unstable features |
| [G-FEAT-03](12-other-rust-features.md#g-feat-03) | Getters and setters |
| [G-FEAT-04](12-other-rust-features.md#g-feat-04) | Implement expected API sets together |
| [G-FEAT-05](12-other-rust-features.md#g-feat-05) | Don't access `#[doc(hidden)]` items you don't own |
| [G-FEAT-06](12-other-rust-features.md#g-feat-06) | Use `const` functions responsibly |
| [G-FEAT-07](12-other-rust-features.md#g-feat-07) | Avoid overloading for non-mathematical purposes |
| [G-FEAT-08](12-other-rust-features.md#g-feat-08) | `#[non_exhaustive]` on publicly-exposed enums |
| [G-FEAT-09](12-other-rust-features.md#g-feat-09) | `#[non_exhaustive]` on public structs with public fields |
| [G-FEAT-10](12-other-rust-features.md#g-feat-10) | `#[must_use]` on functions |
| [G-FEAT-11](12-other-rust-features.md#g-feat-11) | `#[must_use]` on types |
| [G-FEAT-12](12-other-rust-features.md#g-feat-12) | Don't run code before `main()` |

## [Implementation concerns](13-implementation-concerns.md)

| Entry | Title |
| --- | --- |
| [G-IMPL-01](13-implementation-concerns.md#g-impl-01) | Don't overuse type inference |
| [G-IMPL-02](13-implementation-concerns.md#g-impl-02) | Prefer pattern matching over check-then-unwrap |
| [G-IMPL-03](13-implementation-concerns.md#g-impl-03) | Prefer `expect` over `unwrap` |
| [G-IMPL-04](13-implementation-concerns.md#g-impl-04) | Don't omit patterns on `non_exhaustive` enums when matching |
| [G-IMPL-05](13-implementation-concerns.md#g-impl-05) | Use context strings to clarify assertions |
| [G-IMPL-06](13-implementation-concerns.md#g-impl-06) | Don't use iterator combinator chains when direct iteration is clearer |
| [G-IMPL-07](13-implementation-concerns.md#g-impl-07) | Don't call `panic::catch_unwind` |
| [G-IMPL-08](13-implementation-concerns.md#g-impl-08) | Prefer `format!` / `write!` for concatenation and substitution |

## [Testing](14-testing.md)

| Entry | Title |
| --- | --- |
| [G-TEST-01](14-testing.md#g-test-01) | Use the right per-codebase test framework |
