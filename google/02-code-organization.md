# Code Organization

> **Source:** [Google Rust Style Guide — Code organization](https://google.github.io/styleguide/rustguide.html#code-organization) · **License:** CC-BY 3.0
>
> How to split code into crates, files and modules, and how to write `use` imports and qualified paths.

## Terminology

- **Crate:** a unit of compilation. A crate always contains a nameless Rust module, the **root module**.
- **Module:** a unit of code organization within a crate. Modules control visibility of names, can be nested, and can split code across files and directories.
- **Leaf module:** a module without submodules of its own; by definition it may still contain `mod test` and cross-language binding modules such as `#[cxx::bridge]` (see [G-ORG-04](#g-org-04)). A **non-leaf module** contains submodules.

In Chromium it is preferable **not** to use every possibility Rust offers for splitting modules across files (see [G-ORG-01](#g-org-01), [G-ORG-02](#g-org-02)).

<a id="g-org-01"></a>

## G-ORG-01 — Prefer small crates `[chrome]`

**Rule:** Crates **SHOULD** be small; ideally, but not necessarily, a single file focusing on one abstraction. If code can be split into small crates, it **SHOULD** be.

**Applies to:** `[chrome]` Chromium only.

**Details:**
- A crate corresponds to a single `rust_static_library` (chrome) rule, which may contain one or more `.rs` files.
- The root module **MAY** re-export public APIs from leaf submodules, making the submodules an implementation detail of the crate (users don't need to know about them).
- The rule applies equally to crates with and without submodules: first evaluate whether the APIs need to be in the same crate, and only then consider whether submodules could help organize them.
- If you use submodules, follow the expected file structure ([G-ORG-02](#g-org-02)).

**Exceptions:**
- Larger crates are allowed when [trait impl coherence rules](https://smallcultfollowing.com/babysteps/blog/2015/01/14/little-orphan-impls/) (see [orphan rules](../references/language-concepts.md#orphan-rules)) force multiple types and traits into one library.
- Larger crates are allowed when building a large foundational library.

**Rationale:**
- Fine-grained library targets are recommended in Chromium because of the massive codebase size, the non-local effects of adding dependencies to the build graph, and to improve parallel compilation and build caching performance.

**See also:** [Platform notes](../references/platform-notes.md)

<a id="g-org-02"></a>

## G-ORG-02 — Use flat file layout for `rust_library` sources `[chrome]`

**Rule:** All `.rs` source files of one `rust_library` (one Rust crate) **MUST** reside in a single directory. Subdirectories **MUST NOT** be used to organize files belonging to a single crate.

**Applies to:** `[chrome]` Chromium only.

**Details:**
- If organizing one crate's files into subdirectories feels necessary, that strongly indicates the crate has grown too large and should be split ([G-ORG-01](#g-org-01)).
- This layout is intentionally different from the conventional Cargo directory structure typically used in open-source repositories.

**Exceptions:**
- Rust code in `//third_party` that is also built with [Cargo](https://doc.rust-lang.org/cargo/) should follow the conventional Cargo directory structure.

**Rationale:**
- Given the preference for small crates ([G-ORG-01](#g-org-01)), most `rust_library` targets contain only one source file.
- So creating a directory per `rust_library` target would leave most directories with a single source file.
- Multi-file `rust_library` targets don't create dedicated directories for submodules either, which keeps the approach consistent.

<a id="g-org-03"></a>

## G-ORG-03 — Name source files and leaf modules after the primary user-defined item

**Rule:** When a leaf module has a primary user-defined item (e.g., `Foo`), the module **SHOULD** be named after that item (e.g., `mod foo` and `foo.rs`).

**Exceptions:**
- Leaf modules not primarily about any particular abstraction **MAY** be named as desired.

**Rationale:**
- Makes the content/purpose of a source file or leaf module identifiable at a glance.

**Example:**
```rust
// Good: foo.rs, whose primary item is `Foo`
mod foo;
```

<a id="g-org-04"></a>

## G-ORG-04 — Don't implement APIs in non-leaf modules

**Rule:** Non-leaf modules **SHOULD** contain only module declarations (not definitions) and re-exports.

**Details:**
- The definition of leaf modules allows the submodules `mod test` and cross-language binding modules like `#[cxx::bridge]`; a module containing only those still counts as a leaf module.

**Exceptions:**
- A small amount of core "ties things together" implementation code in a non-leaf module may be fine. The goal is to not have a lot of code in these modules.

**Rationale:**
- Makes things easier to find and avoids mixing structure and contents.

<a id="g-org-05"></a>

## G-ORG-05 — Don't use glob imports

**Rule:** Code **MUST NOT** use glob imports (`use path::*`), except in the cases listed below, where you **MAY** use them at your own discretion.

**Details:**
- Defining preludes **SHOULD** be avoided (follows from this rule).

**Exceptions:**
- `use super::*` in inline modules.
- `use enum::*` in method bodies of enums, and in functions that primarily deal with that enum.
- `pub use submodule::*` in non-leaf modules.
- C++ interop: because the same C++ namespace is a submodule in different Rust crates, you may merge the namespaces into a private copy with glob imports. This **MUST** only be used for merging C++ namespaces, and the resulting merged module **MUST NOT** be public. Hypothetical example: `absl::Status` and `absl::StrCat` can then be used from Rust as in C++, instead of `status::absl::Status` and `strings::absl::StrCat`.
- Third-party crates that encourage use of a prelude: allowing a glob import of it is up to the reviewer's discretion.

**Rationale:**
- Glob imports increase toil for code janitors running LSCs, and make it harder to know where a type comes from in contexts without xrefs.

**Example:**
```rust
// OK: private merge of C++ namespaces
mod absl {
    pub use status::absl::*;
    pub use strings::absl::*;
}
```

**See also:** [`imports_granularity`](../references/rustfmt-config.md#imports_granularity), [Rust Style Guide — items](../rust-style-guide/02-items.md)

<a id="g-org-06"></a>

## G-ORG-06 — A `use` statement should import items from one module

**Rule:** Each `use` statement **SHOULD** import items from a single module: there should be no `::` inside the `{..}` block of a `use` statement.

**Details:**
- Corresponds to the rustfmt setting [`imports_granularity = Module`](../references/rustfmt-config.md#imports_granularity).

**Example:**
```rust
// Good
use foo::Foo;
use foo::bar::Bar;
use foo::bar::Baz;

// Good
use foo::Foo;
use foo::bar::{Bar, Baz};

// Bad: `::` inside the braces
use foo::{Foo, bar::Bar};
```

**See also:** [Rust Style Guide — items](../rust-style-guide/02-items.md)

<a id="g-org-07"></a>

## G-ORG-07 — Prefer locally-scoped trait imports

**Rule:** Traits **SHOULD** be imported within function bodies, unless the trait's methods are used extensively throughout the file.

**Example:**
```rust
// Good
fn foo(s: &str) -> Result<u8, std::num::ParseIntError> {
    // Import the trait within a function, not at the module scope.
    use std::str::FromStr;

    u8::from_str(s)
}
```

<a id="g-org-08"></a>

## G-ORG-08 — Qualify calls to top-level functions with a module name

**Rule:** To use top-level functions defined in other modules or crates, you **MUST** import their module and call them as `modulename::functionname()`.

**Exceptions:**
- Functions with sufficiently disambiguated names, e.g., functions from C++ (where calling top-level (free) functions directly is typical).
- In Chromium, functions from small, single-purpose first-party crates ([G-ORG-01](#g-org-01)).
- When the module name itself is prone to collisions, e.g., the module name is a C++ namespace name and multiple crates define the same submodules.

**Rationale:**
- Many Rust codebases consistently assume unqualified lowercase names are local, so `foo()` can be assumed to come from the same module.
- The rule largely exists to avoid reviewer disagreements; the benefit is mostly consistency plus a minor edge when reading code without xrefs.

**Example:**
```rust
// Good
use std::mem;

let x = mem::transmute(y);

// Bad
use std::mem::transmute;

let x = transmute(y);
```

<a id="g-org-09"></a>

## G-ORG-09 — Type names should be imported and used unqualified

**Rule:** By default, types and traits **SHOULD** be referenced without their module path (e.g., `Regex`, not `regex::Regex`), with a `use regex::Regex` at the top of the file or potentially in the relevant function.

**Exceptions:**
- Ambiguities: directly referencing types with common names like `io::Error` is acceptable; so is renaming (`use std::io::Error as IoError`).
- A large "group" of types, e.g. the [`hir` module in rustc](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_hir/hir/index.html), which contains a large number of HIR types.
- Macros and generated code, where requiring a `use` is counterproductive.
- Code with lots of dependencies, where importing everything would produce a very large import block with little benefit.

**Rationale:**
- The rule largely exists to avoid reviewer disagreements.

**Example:**
```rust
// Good
use regex::Regex;

fn parse(re: &Regex) { /* ... */ }

// Bad
fn parse(re: &regex::Regex) { /* ... */ }

// OK: ambiguous common name used qualified
use std::io;

fn read_config() -> Result<(), io::Error> { /* ... */ }
```

<a id="g-org-10"></a>

## G-ORG-10 — Use the appropriate import mechanism `[chrome]`

**Rule:** In Chromium, code **MUST** use `chromium::import!()` as the import mechanism.

**Applies to:** `[chrome]` Chromium only.

**See also:** [Platform notes](../references/platform-notes.md)
