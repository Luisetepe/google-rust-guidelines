# Interoperability

> **Source:** [Rust API Guidelines — Interoperability](https://rust-lang.github.io/api-guidelines/interoperability.html) · **License:** MIT OR Apache-2.0
>
> How a crate interacts nicely with other library functionality: common std traits, conversion and collection traits, Serde, `Send`/`Sync`, error types, number formatting, and reader/writer parameters.

<a id="c-common-traits"></a>

## C-COMMON-TRAITS — Types eagerly implement common traits

**Rule:** Crates that define new types **SHOULD** eagerly implement all applicable common traits.

**Details:**
- The most important common `std` traits: [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html), [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html), [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html), [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html), [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html), [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html), [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html), [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html), [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html), [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html).
- It is common and expected for a type to implement both `Default` and an empty `new` constructor. `new` is the Rust constructor convention and users expect it, so if it is reasonable for the basic constructor to take no arguments, it **SHOULD**, even if functionally identical to `default`.

**Rationale:**
- Rust does not allow _orphans_: roughly, every `impl` must live in the crate that defines the trait or the implementing type.
- E.g. `std` defines `Display`; `url` defines `Url` without implementing `Display`; `webapp` imports both. `webapp` cannot add `Display` to `Url`, since it defines neither. (The newtype pattern is an efficient but inconvenient workaround.)

**Google note:** Google narrows "eagerly" to low-level library code; application code adds trait impls only as needed ([G-TRT-01](../google/07-traits.md#g-trt-01), [G-BG-02](../google/01-background-and-scope.md#g-bg-02)), and gives per-trait guidance on `Copy`, `Clone`, `Default`, `PartialEq`/`Eq`, `PartialOrd`/`Ord`, `Hash` ([G-TRT-07](../google/07-traits.md#g-trt-07)), prefers `#[derive()]` ([G-TRT-03](../google/07-traits.md#g-trt-03)) and says options structs should implement `Default` ([G-TRT-09](../google/07-traits.md#g-trt-09)).

**See also:** [orphan rules](../references/language-concepts.md#orphan-rules), [newtype](../references/language-concepts.md#newtype), [C-DEBUG](09-debuggability.md#c-debug), [`clippy::new_without_default`](../references/clippy-lints.md#new_without_default)

<a id="c-conv-traits"></a>

## C-CONV-TRAITS — Conversions use the standard traits `From`, `AsRef`, `AsMut`

**Rule:** Conversions **SHOULD** implement [`From`](https://doc.rust-lang.org/std/convert/trait.From.html), [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html), [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html) and [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html) where it makes sense. [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html) and [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html) **SHOULD NOT** be implemented.

**Details:**
- `Into` and `TryInto` have blanket impls based on `From` and `TryFrom`; implement those instead.
- Std examples:
  - `From<u16>` is implemented for `u32`: a smaller integer can always be converted to a bigger one.
  - `From<u32>` is *not* implemented for `u16`: the conversion may be impossible if the integer is too big.
  - `TryFrom<u32>` is implemented for `u16` and returns an error if the integer is too big to fit in `u16`.
  - [`From<Ipv6Addr>`](https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html) is implemented for [`IpAddr`](https://doc.rust-lang.org/std/net/enum.IpAddr.html), a type that can represent both v4 and v6 IP addresses.

**Google note:** Google requires implementing a matching std trait instead of a custom method ([G-TRT-02](../google/07-traits.md#g-trt-02)), pairing important `From` impls with an explicit inherent constructor ([G-TRT-08](../google/07-traits.md#g-trt-08)), naming such constructors `from_foo()` ([G-NAM-05](../google/10-naming.md#g-nam-05)), and governs when APIs should accept `impl Into<T>` ([G-FIO-04](../google/05-function-inputs-and-outputs.md#g-fio-04)).

**See also:** [`From` conversions](../references/language-concepts.md#from-conversions), [C-CONV](01-naming.md#c-conv), [`clippy::should_implement_trait`](../references/clippy-lints.md#should_implement_trait)

<a id="c-collect"></a>

## C-COLLECT — Collections implement `FromIterator` and `Extend`

**Rule:** Collections **SHOULD** implement [`FromIterator`](https://doc.rust-lang.org/std/iter/trait.FromIterator.html) and [`Extend`](https://doc.rust-lang.org/std/iter/trait.Extend.html).

**Details:**
- These enable convenient use with [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect), [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition) and [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip).
- `FromIterator` creates a new collection from an iterator's items; `Extend` adds an iterator's items to an existing collection.
- Std example: [`Vec<T>`](https://doc.rust-lang.org/std/vec/struct.Vec.html) implements both `FromIterator<T>` and `Extend<T>`.

<a id="c-serde"></a>

## C-SERDE — Data structures implement Serde's `Serialize`, `Deserialize`

**Rule:** Types that play the role of a data structure **SHOULD** implement [`Serialize`](https://docs.serde.rs/serde/trait.Serialize.html) and [`Deserialize`](https://docs.serde.rs/serde/trait.Deserialize.html).

**Details:**
- There is a continuum between clearly data structures and clearly not, with gray area between. [`LinkedHashMap`](https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html) and [`IpAddr`](https://doc.rust-lang.org/std/net/enum.IpAddr.html) are data structures: reading one from JSON or sending one over IPC is completely reasonable. [`LittleEndian`](https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html) is not: it is a `byteorder` compile-time marker and an instance can never exist at runtime. The #rust or #serde IRC channels can help assess ambiguous cases.
- A crate that does not already depend on Serde for other reasons **MAY** gate Serde impls behind a Cargo cfg, so downstream libraries only pay Serde's compile cost if they need the impls.
- The Cargo cfg **SHOULD** be named simply `"serde"` for consistency with other Serde-based libraries; **MUST NOT** use names like `"serde_impls"` or `"serde_serialization"`.

**Example:**
```toml
# Without derive
[dependencies]
serde = { version = "1.0", optional = true }
```

```rust
pub struct T { /* ... */ }

#[cfg(feature = "serde")]
impl Serialize for T { /* ... */ }

#[cfg(feature = "serde")]
impl<'de> Deserialize<'de> for T { /* ... */ }
```

```toml
# With derive
[dependencies]
serde = { version = "1.0", optional = true, features = ["derive"] }
```

```rust
#[cfg_attr(feature = "serde", derive(Serialize, Deserialize))]
pub struct T { /* ... */ }
```

**See also:** [C-FEATURE](01-naming.md#c-feature)

<a id="c-send-sync"></a>

## C-SEND-SYNC — Types are `Send` and `Sync` where possible

**Rule:** Types **SHOULD** be [`Send`](https://doc.rust-lang.org/std/marker/trait.Send.html) and [`Sync`](https://doc.rust-lang.org/std/marker/trait.Sync.html) where possible. In types that manipulate raw pointers, the `Send`/`Sync` status **MUST** accurately reflect the type's thread-safety characteristics.

**Details:**
- `Send` and `Sync` are implemented automatically when the compiler determines it is appropriate.
- Tests like the example below help catch unintentional regressions in whether a type implements `Send` or `Sync`.

**Google note:** Raw-pointer manipulation means unsafe code, which Google allows only when unavoidable, encapsulated and justified with `SAFETY:` comments ([G-FEAT-01](../google/12-other-rust-features.md#g-feat-01)); write such tests in the codebase's own test framework rather than plain `#[test]` where one exists ([G-TEST-01](../google/14-testing.md#g-test-01)).

**Example:**
```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```

**See also:** [C-GOOD-ERR](#c-good-err)

<a id="c-good-err"></a>

## C-GOOD-ERR — Error types are meaningful and well-behaved

**Rule:** Every error type (any `E` in a `Result<T, E>` returned by a public function of the crate) **SHOULD** implement [`std::error::Error`](https://doc.rust-lang.org/std/error/trait.Error.html), `Send` and `Sync`. `()` **MUST NOT** be used as an error type, even where the error carries no useful information.

**Details:**
- `std::error::Error` is how error-handling libraries like [`error-chain`](https://docs.rs/error-chain) abstract over error types, and lets the error be the [`source()`](https://doc.rust-lang.org/std/error/trait.Error.html#method.source) of another error.
- A non-`Send` error cannot be returned by a thread run with [`thread::spawn`](https://doc.rust-lang.org/std/thread/fn.spawn.html); a non-`Sync` error cannot be passed across threads via an [`Arc`](https://doc.rust-lang.org/std/sync/struct.Arc.html). Both are common requirements in multithreaded applications.
- `Send` and `Sync` are also needed to wrap a custom error in an IO error via [`std::io::Error::new`](https://doc.rust-lang.org/std/io/struct.Error.html#method.new), which requires `Error + Send + Sync`.
- Be vigilant in functions returning `Error` trait objects, e.g. [`reqwest::Error::get_ref`](https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref): typically `Error + Send + Sync + 'static` is most useful for callers; `'static` allows use with [`Error::downcast_ref`](https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2).
- Why not `()`:
  - It does not implement `Error`, so it cannot be used with error-handling libraries like `error-chain`.
  - It does not implement `Display`, so users must write their own message to fail on it.
  - Its `Debug` representation is unhelpful to users who `unwrap()` the error.
  - A downstream library cannot meaningfully implement `From<()>` for its error type, so `()` cannot be used with the `?` operator.
- Instead define a meaningful error type specific to the crate or the individual function, with appropriate `Error` and `Display` impls. With no useful information to carry, it can be a unit struct.
- The `Display` message **SHOULD** be lowercase without trailing punctuation, and typically concise.
- [`Error::description()`](https://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description) **SHOULD NOT** be implemented: it is deprecated; users should always use `Display` instead.
- Std example: [`ParseBoolError`](https://doc.rust-lang.org/std/str/struct.ParseBoolError.html) is returned when parsing a bool from a string fails.
- Example messages: "unexpected end of file"; "provided string was not \`true\` or \`false\`"; "invalid IP address syntax"; "second time provided was later than self"; "invalid UTF-8 sequence of {} bytes from index {}"; "environment variable was not valid unicode: {:?}".

**Google note:** Google first decides between `Option`, `Result` and `panic!` by API contract ([G-FIO-06](../google/05-function-inputs-and-outputs.md#g-fio-06)) and adds that error types should implement `Copy` whenever possible ([G-TRT-07](../google/07-traits.md#g-trt-07)).

**Example:**
```rust
use std::error::Error;
use std::fmt::Display;

// Bad: () as the error type
fn do_the_thing() -> Result<Wow, ()>

// Good
fn do_the_thing() -> Result<Wow, DoError>

#[derive(Debug)]
struct DoError;

impl Display for DoError { /* ... */ }
impl Error for DoError { /* ... */ }
```

**See also:** [C-WORD-ORDER](01-naming.md#c-word-order), [C-SEND-SYNC](#c-send-sync)

<a id="c-num-fmt"></a>

## C-NUM-FMT — Binary number types provide `Hex`, `Octal`, `Binary` formatting

**Rule:** Any number type on which you would consider bitwise manipulations like `|` or `&` **MUST** implement [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html), [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html), [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html) and [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html).

**Details:**
- These traits control formatting under the `{:X}`, `{:x}`, `{:o}` and `{:b}` specifiers.
- Especially appropriate for bitflag types.

**Exceptions:**
- Numeric quantity types like `struct Nanoseconds(u64)` probably do not need these.

**See also:** [C-BITFLAG](07-type-safety.md#c-bitflag)

<a id="c-rw-value"></a>

## C-RW-VALUE — Generic reader/writer functions take `R: Read` and `W: Write` by value

**Rule:** Generic reader/writer functions **SHOULD** accept `R: Read` / `W: Write` by value, and their docs **MUST** briefly remind users that a mut reference can be passed.

**Details:**
- The std impls below mean any function taking `R: Read` or `W: Write` by value can be called with a mut reference if necessary.
- New Rust users often struggle here: having opened a file, they want to read several pieces of data out of it, but the function reading one piece consumes the reader. The solution is to pass `&mut f` instead of `f` as the reader.
- Examples: [`flate2::read::GzDecoder::new`](https://docs.rs/flate2/0.2/flate2/read/struct.GzDecoder.html#method.new), [`flate2::write::GzEncoder::new`](https://docs.rs/flate2/0.2/flate2/write/struct.GzEncoder.html#method.new), [`serde_json::from_reader`](https://docs.serde.rs/serde_json/fn.from_reader.html), [`serde_json::to_writer`](https://docs.serde.rs/serde_json/fn.to_writer.html).

**Example:**
```rust
impl<'a, R: Read + ?Sized> Read for &'a mut R { /* ... */ }

impl<'a, W: Write + ?Sized> Write for &'a mut W { /* ... */ }
```

**See also:** [C-CALLER-CONTROL](06-flexibility.md#c-caller-control), [C-GENERIC](06-flexibility.md#c-generic)
