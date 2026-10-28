# Documentation

> **Source:** [Rust API Guidelines — Documentation](https://rust-lang.github.io/api-guidelines/documentation.html) · **License:** MIT OR Apache-2.0
>
> How a crate is abundantly documented: crate docs, per-item examples, fallible examples, error/panic/safety sections, links, Cargo metadata, release notes, and hiding implementation details.

<a id="c-crate-doc"></a>

## C-CRATE-DOC — Crate level docs are thorough and include examples

**Rule:** Crate-level documentation **SHOULD** be thorough and include examples, as described in [RFC 1687](https://github.com/rust-lang/rfcs/pull/1687).

**Google note:** Google requires `//!` comments for crate/module docs, which should be placed at the top of the file, and asks that enough functionality be covered by examples *somewhere* to show how the crate works ([G-DOC](../google/11-documentation.md)).

**See also:** [C-EXAMPLE](#c-example), [C-RELNOTES](#c-relnotes)

<a id="c-example"></a>

## C-EXAMPLE — All items have a rustdoc example

**Rule:** Every public module, trait, struct, enum, function, method, macro and type definition **SHOULD** have an example that exercises the functionality, applied within reason.

**Details:**
- An example's purpose is often not to show *how to use* the item (readers can be expected to know how to invoke functions, match on enums and other fundamentals) but *why someone would want to use* it.

**Exceptions:**
- A link to an applicable example on another item **MAY** suffice. E.g. if exactly one function uses a particular type, write a single example on either the function or the type and link to it from the other.

**Google note:** Google relaxes this: not every function needs examples, as long as enough functionality is covered in examples somewhere to give an idea of how the crate works ([G-DOC](../google/11-documentation.md)).

**Example:**
```rust
// Bad: a poor example of using clone(). It mechanically shows *how* to
// call clone(), but does nothing to show *why* somebody would want this.
fn main() {
    let hello = "hello";

    hello.clone();
}
```

**See also:** [C-QUESTION-MARK](#c-question-mark)

<a id="c-question-mark"></a>

## C-QUESTION-MARK — Examples use `?`, not `try!`, not `unwrap`

**Rule:** Fallible example code **SHOULD** propagate errors with `?`, not `try!` and not `unwrap`.

**Details:**
- Example code is often copied verbatim by users; unwrapping an error should be a conscious decision the user makes.
- Common structure: wrap the example in a hidden `fn main() -> Result<(), Box<dyn Error>>`. Lines beginning with `#` are compiled by `cargo test` when building the example but do not appear in user-visible rustdoc.

**Example:**
````rust
/// ```rust
/// # use std::error::Error;
/// #
/// # fn main() -> Result<(), Box<dyn Error>> {
/// your;
/// example?;
/// code;
/// #
/// #     Ok(())
/// # }
/// ```
````

<a id="c-failure"></a>

## C-FAILURE — Function docs include error, panic, and safety considerations

**Rule:** Error conditions **SHOULD** be documented in an "Errors" section, panic conditions in a "Panics" section, and unsafe functions **SHOULD** have a "Safety" section explaining all invariants the caller is responsible for upholding to use the function correctly.

**Details:**
- "Errors" and "Panics" sections apply to trait methods too: trait methods whose implementations are allowed or expected to return an error / panic should document it.
- Std examples: some implementations of the trait method [`std::io::Read::read`](https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read) may return an error; [`Vec::insert`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert) may panic; the unsafe [`std::ptr::read`](https://doc.rust-lang.org/std/ptr/fn.read.html) places requirements on the caller.

**Exceptions:**
- Not every conceivable panic case needs documenting, especially if the panic occurs in logic provided by the caller (e.g. documenting the `Display` panic below seems excessive). When in doubt, err on the side of documenting more panic cases.

**Google note:** Google requires always documenting panic cases for functions that may panic on malformed input and clearly documenting invariants of unsafe functions and traits ([G-DOC](../google/11-documentation.md)); unsafe blocks additionally need `SAFETY:` comments ([G-FEAT-01](../google/12-other-rust-features.md#g-feat-01)).

**Example:**
```rust
/// Pull some bytes from this source into the specified buffer, returning
/// how many bytes were read.
///
/// ... lots more info ...
///
/// # Errors
///
/// If this function encounters any form of I/O or other error, an error
/// variant will be returned. If an error is returned then it must be
/// guaranteed that no bytes were read.
```

```rust
/// Inserts an element at position `index` within the vector, shifting all
/// elements after it to the right.
///
/// # Panics
///
/// Panics if `index` is out of bounds.
```

```rust
// Excessive: the panic comes from caller-provided logic.
/// # Panics
///
/// This function panics if `T`'s implementation of `Display` panics.
pub fn print<T: Display>(t: T) {
    println!("{}", t.to_string());
}
```

```rust
/// Reads the value from `src` without moving it. This leaves the
/// memory in `src` unchanged.
///
/// # Safety
///
/// Beyond accepting a raw pointer, this is unsafe because it semantically
/// moves the value out of `src` without preventing further usage of `src`.
/// If `T` is not `Copy`, then care must be taken to ensure that the value at
/// `src` is not used before the data is overwritten again (e.g. with `write`,
/// `zero_memory`, or `copy_memory`). Note that `*src = foo` counts as a use
/// because it will attempt to drop the value previously at `*src`.
///
/// The pointer must be aligned; use `read_unaligned` if that is not the case.
```

**See also:** [C-GOOD-ERR](02-interoperability.md#c-good-err), [G-FIO-06](../google/05-function-inputs-and-outputs.md#g-fio-06)

<a id="c-link"></a>

## C-LINK — Prose contains hyperlinks to relevant things

**Rule:** Documentation prose **SHOULD** hyperlink to relevant things, as officially recommended by RFC 1574 under ["Link all the things"](https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md#link-all-the-things).

**Details:**
- Regular links: inline markdown `[text](url)`.
- Links to other types: mark them ``[`text`]`` and add the link target on a new line at the end of the docstring as ``[`text`]: <target>``.
- Targets for methods within the same type usually look like `#method.serialize_struct`; for other types like `trait.Deserialize.html`; they may also point to a parent or child module.

**Example:**
```md
[`serialize_struct`]: #method.serialize_struct
[`Deserialize`]: trait.Deserialize.html
[`Value`]: ../enum.Value.html
[`DeserializeOwned`]: de/trait.DeserializeOwned.html
```

<a id="c-metadata"></a>

## C-METADATA — Cargo.toml includes all common metadata

**Rule:** The `[package]` section of `Cargo.toml` **SHOULD** include `authors`, `description`, `license`, `repository`, `keywords` and `categories`. `documentation` and `homepage` are optional.

**Details:**
- *crates.io* links to the crate's documentation on [*docs.rs*](https://docs.rs) by default. Set `documentation` only if docs are hosted elsewhere, e.g. because the crate links against a shared library unavailable in the *docs.rs* build environment.
- Set `homepage` only if there is a unique website for the crate other than the source repository or API documentation. `homepage` **MUST NOT** be redundant with `documentation` or `repository`. E.g. serde sets `homepage` to *https://serde.rs*, a dedicated website.

**See also:** [Rust Style Guide — Cargo.toml conventions](../rust-style-guide/07-cargo-toml.md)

<a id="c-relnotes"></a>

## C-RELNOTES — Release notes document all significant changes

**Rule:** Release notes **SHOULD** summarize what changed in each published release, and a link to them (or the notes themselves) **SHOULD** be included in the crate-level docs and/or the repository linked in Cargo.toml. Breaking changes (as defined in [RFC 1105](https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md)) **SHOULD** be clearly identified.

**Details:**
- With Git, every release published to *crates.io* **SHOULD** have a corresponding tag identifying the published commit; non-Git VCS tools should use a similar process.
- Annotated tags are preferred (**SHOULD**), because some Git commands ignore unannotated tags if any annotated tags exist.
- Examples: [Serde 1.0.0 release notes](https://github.com/serde-rs/serde/releases/tag/v1.0.0), [Serde 0.9.8 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.8), [Serde 0.9.0 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.0), [Diesel change log](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md).

**Example:**
```bash
# Tag the current commit
GIT_COMMITTER_DATE=$(git log -n1 --pretty=%aD) git tag -a -m "Release 0.3.0" 0.3.0
git push --tags
```

<a id="c-hidden"></a>

## C-HIDDEN — Rustdoc does not show unhelpful implementation details

**Rule:** Rustdoc **SHOULD** include everything users need to use the crate fully and nothing more; implementation details **SHOULD NOT** be real entries in the documentation (explaining relevant ones in prose is fine).

**Details:**
- Be especially selective about which impls are visible: all that users need to use the crate fully, no others. Below, rustdoc for `PublicError` would by default show the `From<PrivateError>` impl; it is hidden with `#[doc(hidden)]` because users can never have a `PrivateError`, so the impl is never relevant to them.
- [`pub(crate)`](https://github.com/rust-lang/rfcs/blob/master/text/1422-pub-restricted.md) is another great tool for removing implementation details from the public API: items are usable outside their own module but not outside the crate.

**Google note:** Google adds the consumer side: treat items marked `#[doc(hidden)]` as private and don't access them unless you own the code ([G-FEAT-05](../google/12-other-rust-features.md#g-feat-05)).

**Example:**
```rust
// This error type is returned to users.
pub struct PublicError { /* ... */ }

// This error type is returned by some private helper functions.
struct PrivateError { /* ... */ }

// Enable use of `?` operator.
#[doc(hidden)]
impl From<PrivateError> for PublicError {
    fn from(err: PrivateError) -> PublicError {
        /* ... */
    }
}
```
