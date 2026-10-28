# Necessities

> **Source:** [Rust API Guidelines — Necessities](https://rust-lang.github.io/api-guidelines/necessities.html) · **License:** MIT OR Apache-2.0
>
> Requirements that, to whom they matter, really matter: stable public dependencies and permissive licensing.

<a id="c-stable"></a>

## C-STABLE — Public dependencies of a stable crate are stable

**Rule:** A crate **MUST NOT** be declared stable (>=1.0.0) unless all of its public dependencies are stable.

**Details:**
- Public dependencies are crates whose types are used in the current crate's public API; e.g. `pub fn do_my_thing(arg: other_crate::TheirThing)` makes `other_crate` public.
- Public dependencies can sneak in at unexpected places: a private field or private enum holding `other_crate::Error` is fine, but a public `impl From<other_crate::Error> for Error` puts `other_crate` into the public API.

**Google note:** Unrelated sense of "stable": Google separately forbids unstable Rust language features, with Android/Chrome/Fuchsia exceptions ([G-FEAT-02](../google/12-other-rust-features.md#g-feat-02)).

**Example:**
```rust
// A crate containing this function cannot be stable unless `other_crate` is also stable.
pub fn do_my_thing(arg: other_crate::TheirThing) { /* ... */ }
```

```rust
pub struct Error {
    private: ErrorImpl,
}

enum ErrorImpl {
    Io(io::Error),
    // Should be okay even if other_crate isn't
    // stable, because ErrorImpl is private.
    Dep(other_crate::Error),
}

// Oh no! This puts other_crate into the public API
// of the current crate.
impl From<other_crate::Error> for Error {
    fn from(err: other_crate::Error) -> Self {
        Error { private: ErrorImpl::Dep(err) }
    }
}
```

**See also:** [C-GOOD-ERR](02-interoperability.md#c-good-err), [C-CONV-TRAITS](02-interoperability.md#c-conv-traits)

<a id="c-permissive"></a>

## C-PERMISSIVE — Crate and its dependencies have a permissive license

**Rule:** Crates needing maximum compatibility with the Rust ecosystem are **RECOMMENDED** to dual-license under [MIT](https://github.com/rust-lang/rust/blob/master/LICENSE-MIT) OR [Apache 2.0](https://github.com/rust-lang/rust/blob/master/LICENSE-APACHE), like the Rust project. A permissively-licensed crate **SHOULD** generally depend only on permissively-licensed crates.

**Details:**
- To apply the Rust license: set `license = "MIT OR Apache-2.0"` in `Cargo.toml`; add `LICENSE-APACHE` and `LICENSE-MIT` with the license texts to the repository root (texts available e.g. from choosealicense.com: [Apache-2.0](https://choosealicense.com/licenses/apache-2.0/), [MIT](https://choosealicense.com/licenses/mit/)); and add a License/Contribution section toward the end of `README.md` (below).
- A single permissive license such as MIT or BSD is also common and entirely compatible with Rust's, since it imposes only the minimal restrictions of Rust's MIT license.
- Crates desiring perfect license compatibility with Rust **SHOULD NOT** choose only the Apache license: though permissive, it imposes restrictions beyond MIT and BSD that can discourage or prevent use in some scenarios, so Apache-only software cannot be used in some situations where most of the Rust runtime stack can.
- A dependency's license can restrict distribution of the crate itself.
- These guidelines do not explain Rust's license in detail (a little is said in the [Rust FAQ](https://github.com/dtolnay/rust-faq#why-a-dual-mitasl2-license)); they concern interoperability with Rust and are not comprehensive over licensing options.

**Example:**
```toml
[package]
name = "..."
version = "..."
authors = ["..."]
license = "MIT OR Apache-2.0"
```

```markdown
## License

Licensed under either of

 * Apache License, Version 2.0
   ([LICENSE-APACHE](LICENSE-APACHE) or <http://www.apache.org/licenses/LICENSE-2.0>)
 * MIT license
   ([LICENSE-MIT](LICENSE-MIT) or <http://opensource.org/licenses/MIT>)

at your option.

## Contribution

Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in the work by you, as defined in the Apache-2.0 license, shall be
dual licensed as above, without any additional terms or conditions.
```

**See also:** [C-METADATA](04-documentation.md#c-metadata)
