# When and how to write documentation

> **Source:** [Google Rust Style Guide — When and how to write documentation](https://google.github.io/styleguide/rustguide.html#writing-doc) · **License:** CC-BY 3.0
>
> What to document with doc comments, what to put in them, and how to format doc comments versus ordinary comments.

Doc comments are `///` (documents the following item) and `//!` (documents the enclosing module or library); ordinary comments are `//`. G-DOC-01..06 cover when and what to document; G-DOC-07..12 cover how to format documentation comments.

<a id="g-doc-01"></a>

## G-DOC-01 — Document public APIs

**Rule:** Public APIs **MUST** have documentation comments; the `missing_docs` lint **MAY** be used to help enforce this.

**Exceptions:**
- Comments should say something the code does not already say. If there is nothing insightful to say, omit the comment rather than repeat the function name trivially; it is fine to `allow` the `missing_docs` lint in that case.
- Omitting documentation on individual enum variants is generally okay, especially if their names are self-explanatory.

**Example:**
```rust
// Bad: repeats the function name, says nothing new
/// Removes an element.
pub fn remove_element() -> Element {}
```

**See also:** [`missing_docs` lint](https://doc.rust-lang.org/rustc/lints/listing/allowed-by-default.html#missing-docs)

<a id="g-doc-02"></a>

## G-DOC-02 — Consider including examples

**Rule:** Documentation **SHOULD** include examples: not every function needs one, but enough functionality **SHOULD** be covered by examples *somewhere* to give an idea of how the crate works.

<a id="g-doc-03"></a>

## G-DOC-03 — Document panic cases

**Rule:** Functions that may panic due to malformed input **MUST** always document their panic cases.

<a id="g-doc-04"></a>

## G-DOC-04 — Document invariants of unsafe functions and traits

**Rule:** Unsafe functions and traits **MUST** clearly document their invariants.

**See also:** [G-FEAT-01](12-other-rust-features.md#g-feat-01)

<a id="g-doc-05"></a>

## G-DOC-05 — Use capitalization and punctuation

**Rule:** Documentation comments **MUST** use capitalization and punctuation.

<a id="g-doc-06"></a>

## G-DOC-06 — Don't explain language basics

**Rule:** Documentation **MUST NOT** explain the basics of the programming language.

**Example:**
```rust
// Bad: restates what `impl FileSystem for MockFileSystem` already says
struct MockFileSystem { ... }

/// A mock implementation of the FileSystem trait, intended to be used in unit tests.
impl FileSystem for MockFileSystem
```

<a id="g-doc-07"></a>

## G-DOC-07 — Use `///` for item documentation

**Rule:** Items following the comment, such as functions, structs, enums, traits and type aliases, **MUST** be documented with `///` comments.

<a id="g-doc-08"></a>

## G-DOC-08 — Use `//!` for module and library documentation

**Rule:** The module or library itself **MUST** be documented with `//!` comments, which **SHOULD** be placed at the top of the file.

<a id="g-doc-09"></a>

## G-DOC-09 — Follow the Rust style guide's doc-comment rules

**Rule:** Doc comments **MUST** follow the rules in the official Rust style guide on doc comments.

**See also:** [Rust Style Guide — formatting conventions (doc comments)](../rust-style-guide/01-formatting-conventions.md)

<a id="g-doc-10"></a>

## G-DOC-10 — Use Markdown syntax

**Rule:** Doc comments **MUST** use Markdown syntax.

<a id="g-doc-11"></a>

## G-DOC-11 — Use `//` for non-documentation comments

**Rule:** Non-documentation comments (implementation comments, TODOs not interesting to users, commented-out source code, etc.) **MUST** use ordinary `//` comment syntax.

**Details:**
- Mixing doc comments (`///`, `//!`) and ordinary comments (`//`) is acceptable and even encouraged: if one thing should be documented publicly and another noted as an implementation comment, use both.

**Rationale:**
- Many languages distinguish doc comments only by convention (e.g. extra `*`); Rust (like Python) uses an entirely different syntax, which brings concrete benefits:
- Doc comments can be consumed by tooling: documentation generators like `rustdoc`, IDE features, and Kythe.
- Doc comments have a standard structure and formatting rules (Markdown), allowing rich text.
- Misapplied doc comment syntax (e.g. on an expression instead of a documentable item) is linted against, catching documentation errors early; with no lint errors, doc comments are guaranteed visible to documentation tooling, removing flakiness and unpredictability.
- Doc comments are part of the AST, so they can be produced and consumed by macros, both `macro_rules!` and procedural macros.

**Example:**
```rust
// Good: public doc comment plus an implementation-only TODO
/// Finds the first `FooItem` with the given `name`, or `None` if not present.
// TODO: crbug.com/12345 - `name` should be &str, not String.
pub fn get_named(items: &[FooItem], name: String) -> Option<&FooItem> {...}
```

<a id="g-doc-12"></a>

## G-DOC-12 — Prefer direct doc comments over macro-generated docs

**Rule:** Documentation **SHOULD** be written directly with `///` and `//!` comments rather than generated from macros.

**Rationale:**
- The documentation is then directly readable when viewing or modifying the source code.
