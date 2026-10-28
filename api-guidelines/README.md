# Rust API Guidelines

> **Source:** [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
>
> Library API design guidelines from the Rust library team. IDs are the upstream `C-XXX` IDs; this table doubles as the checklist.

## Chapters

- [Naming](01-naming.md) — 7 rules
- [Interoperability](02-interoperability.md) — 8 rules
- [Macros](03-macros.md) — 5 rules
- [Documentation](04-documentation.md) — 8 rules
- [Predictability](05-predictability.md) — 7 rules
- [Flexibility](06-flexibility.md) — 4 rules
- [Type safety](07-type-safety.md) — 4 rules
- [Dependability](08-dependability.md) — 3 rules
- [Debuggability](09-debuggability.md) — 2 rules
- [Future proofing](10-future-proofing.md) — 4 rules
- [Necessities](11-necessities.md) — 2 rules

## [Naming](01-naming.md)

| Entry | Title |
| --- | --- |
| [C-CASE](01-naming.md#c-case) | Casing conforms to RFC 430 |
| [C-CONV](01-naming.md#c-conv) | Ad-hoc conversions follow `as_`, `to_`, `into_` conventions |
| [C-GETTER](01-naming.md#c-getter) | Getter names follow Rust convention |
| [C-ITER](01-naming.md#c-iter) | Methods on collections that produce iterators follow `iter`, `iter_mut`, `into_iter` |
| [C-ITER-TY](01-naming.md#c-iter-ty) | Iterator type names match the methods that produce them |
| [C-FEATURE](01-naming.md#c-feature) | Feature names are free of placeholder words |
| [C-WORD-ORDER](01-naming.md#c-word-order) | Names use a consistent word order |

## [Interoperability](02-interoperability.md)

| Entry | Title |
| --- | --- |
| [C-COMMON-TRAITS](02-interoperability.md#c-common-traits) | Types eagerly implement common traits |
| [C-CONV-TRAITS](02-interoperability.md#c-conv-traits) | Conversions use the standard traits `From`, `AsRef`, `AsMut` |
| [C-COLLECT](02-interoperability.md#c-collect) | Collections implement `FromIterator` and `Extend` |
| [C-SERDE](02-interoperability.md#c-serde) | Data structures implement Serde's `Serialize`, `Deserialize` |
| [C-SEND-SYNC](02-interoperability.md#c-send-sync) | Types are `Send` and `Sync` where possible |
| [C-GOOD-ERR](02-interoperability.md#c-good-err) | Error types are meaningful and well-behaved |
| [C-NUM-FMT](02-interoperability.md#c-num-fmt) | Binary number types provide `Hex`, `Octal`, `Binary` formatting |
| [C-RW-VALUE](02-interoperability.md#c-rw-value) | Generic reader/writer functions take `R: Read` and `W: Write` by value |

## [Macros](03-macros.md)

| Entry | Title |
| --- | --- |
| [C-EVOCATIVE](03-macros.md#c-evocative) | Input syntax is evocative of the output |
| [C-MACRO-ATTR](03-macros.md#c-macro-attr) | Item macros compose well with attributes |
| [C-ANYWHERE](03-macros.md#c-anywhere) | Item macros work anywhere that items are allowed |
| [C-MACRO-VIS](03-macros.md#c-macro-vis) | Item macros support visibility specifiers |
| [C-MACRO-TY](03-macros.md#c-macro-ty) | Type fragments are flexible |

## [Documentation](04-documentation.md)

| Entry | Title |
| --- | --- |
| [C-CRATE-DOC](04-documentation.md#c-crate-doc) | Crate level docs are thorough and include examples |
| [C-EXAMPLE](04-documentation.md#c-example) | All items have a rustdoc example |
| [C-QUESTION-MARK](04-documentation.md#c-question-mark) | Examples use `?`, not `try!`, not `unwrap` |
| [C-FAILURE](04-documentation.md#c-failure) | Function docs include error, panic, and safety considerations |
| [C-LINK](04-documentation.md#c-link) | Prose contains hyperlinks to relevant things |
| [C-METADATA](04-documentation.md#c-metadata) | Cargo.toml includes all common metadata |
| [C-RELNOTES](04-documentation.md#c-relnotes) | Release notes document all significant changes |
| [C-HIDDEN](04-documentation.md#c-hidden) | Rustdoc does not show unhelpful implementation details |

## [Predictability](05-predictability.md)

| Entry | Title |
| --- | --- |
| [C-SMART-PTR](05-predictability.md#c-smart-ptr) | Smart pointers do not add inherent methods |
| [C-CONV-SPECIFIC](05-predictability.md#c-conv-specific) | Conversions live on the most specific type involved |
| [C-METHOD](05-predictability.md#c-method) | Functions with a clear receiver are methods |
| [C-NO-OUT](05-predictability.md#c-no-out) | Functions do not take out-parameters |
| [C-OVERLOAD](05-predictability.md#c-overload) | Operator overloads are unsurprising |
| [C-DEREF](05-predictability.md#c-deref) | Only smart pointers implement `Deref` and `DerefMut` |
| [C-CTOR](05-predictability.md#c-ctor) | Constructors are static, inherent methods |

## [Flexibility](06-flexibility.md)

| Entry | Title |
| --- | --- |
| [C-INTERMEDIATE](06-flexibility.md#c-intermediate) | Functions expose intermediate results to avoid duplicate work |
| [C-CALLER-CONTROL](06-flexibility.md#c-caller-control) | Caller decides where to copy and place data |
| [C-GENERIC](06-flexibility.md#c-generic) | Functions minimize assumptions about parameters by using generics |
| [C-OBJECT](06-flexibility.md#c-object) | Traits are object-safe if they may be useful as a trait object |

## [Type safety](07-type-safety.md)

| Entry | Title |
| --- | --- |
| [C-NEWTYPE](07-type-safety.md#c-newtype) | Newtypes provide static distinctions |
| [C-CUSTOM-TYPE](07-type-safety.md#c-custom-type) | Arguments convey meaning through types, not `bool` or `Option` |
| [C-BITFLAG](07-type-safety.md#c-bitflag) | Types for a set of flags are `bitflags`, not enums |
| [C-BUILDER](07-type-safety.md#c-builder) | Builders enable construction of complex values |

## [Dependability](08-dependability.md)

| Entry | Title |
| --- | --- |
| [C-VALIDATE](08-dependability.md#c-validate) | Functions validate their arguments |
| [C-DTOR-FAIL](08-dependability.md#c-dtor-fail) | Destructors never fail |
| [C-DTOR-BLOCK](08-dependability.md#c-dtor-block) | Destructors that may block have alternatives |

## [Debuggability](09-debuggability.md)

| Entry | Title |
| --- | --- |
| [C-DEBUG](09-debuggability.md#c-debug) | All public types implement `Debug` |
| [C-DEBUG-NONEMPTY](09-debuggability.md#c-debug-nonempty) | `Debug` representation is never empty |

## [Future proofing](10-future-proofing.md)

| Entry | Title |
| --- | --- |
| [C-SEALED](10-future-proofing.md#c-sealed) | Sealed traits protect against downstream implementations |
| [C-STRUCT-PRIVATE](10-future-proofing.md#c-struct-private) | Structs have private fields |
| [C-NEWTYPE-HIDE](10-future-proofing.md#c-newtype-hide) | Newtypes encapsulate implementation details |
| [C-STRUCT-BOUNDS](10-future-proofing.md#c-struct-bounds) | Data structures do not duplicate derived trait bounds |

## [Necessities](11-necessities.md)

| Entry | Title |
| --- | --- |
| [C-STABLE](11-necessities.md#c-stable) | Public dependencies of a stable crate are stable |
| [C-PERMISSIVE](11-necessities.md#c-permissive) | Crate and its dependencies have a permissive license |
