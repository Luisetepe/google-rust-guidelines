# Predictability

> **Source:** [Rust API Guidelines — Predictability](https://rust-lang.github.io/api-guidelines/predictability.html) · **License:** MIT OR Apache-2.0
>
> Guidelines that keep a crate's code legible and acting how it looks: smart pointers, conversions, methods, return values, operators, `Deref`, and constructors.

<a id="c-smart-ptr"></a>

## C-SMART-PTR — Smart pointers do not add inherent methods

**Rule:** Smart pointer types **MUST NOT** add inherent methods that take `self`; define such operations as associated functions called with the type path (e.g. `Box::into_raw(b)`).

**Details:**
- This is why [`Box::into_raw`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw) is defined as `fn into_raw(b: Box<T>)` rather than `fn into_raw(self)`.

**Rationale:**
- Methods on the pointee are reachable through the smart pointer's `Deref` impl; an inherent method on the pointer makes it unclear at the call site whether a method belongs to `Box<T>` or to `T`.

**Example:**
```rust
// Good: associated function, called with the type path
impl<T> Box<T> where T: ?Sized {
    fn into_raw(b: Box<T>) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;
let ptr = Box::into_raw(boxed_str);
```

```rust
// Bad: inherent method on the smart pointer
impl<T> Box<T> where T: ?Sized {
    // Do not do this.
    fn into_raw(self) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;

// This is a method on str accessed through the smart pointer Deref impl.
boxed_str.chars()

// This is a method on Box<str>...?
boxed_str.into_raw()
```

**See also:** [C-DEREF](#c-deref), [inherent impls](../references/language-concepts.md#inherent-impls)

<a id="c-conv-specific"></a>

## C-CONV-SPECIFIC — Conversions live on the most specific type involved

**Rule:** Conversions between two types **SHOULD** live on the more specific type (the one adding an invariant or interpretation the other lacks). When in doubt, you **SHOULD** prefer `to_`/`as_`/`into_` over `from_`.

**Details:**
- `to_`/`as_`/`into_` are more ergonomic and can be chained with other methods.
- Example of specificity: [`str`](https://doc.rust-lang.org/std/primitive.str.html) is more specific than `&[u8]` because it is a UTF-8 encoded byte sequence; so `str` provides both [`as_bytes`](https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes) and the [`from_utf8`](https://doc.rust-lang.org/std/str/fn.from_utf8.html) constructor for converting to and from `&[u8]`.

**Rationale:**
- Intuitive, and avoids polluting concrete types like `&[u8]` with endless conversion methods.

**Google note:** Google requires standard-library method naming (`as_`/`to_`/`into_`, [G-NAM-01](../google/10-naming.md#g-nam-01)) and names value-to-value conversion constructors `from_foo()` with an associated `From` impl ([G-NAM-05](../google/10-naming.md#g-nam-05), [G-TRT-08](../google/07-traits.md#g-trt-08)).

**See also:** [C-CONV](01-naming.md#c-conv), [C-CONV-TRAITS](02-interoperability.md#c-conv-traits), [C-CTOR](#c-ctor), [`From` conversions](../references/language-concepts.md#from-conversions)

<a id="c-method"></a>

## C-METHOD — Functions with a clear receiver are methods

**Rule:** Any operation clearly associated with a particular type **SHOULD** be a method on that type rather than a free function taking it as a parameter.

**Rationale:**
- Methods need not be imported or qualified: a value of the type is enough.
- Method invocation performs autoborrowing (including mutable borrows).
- Methods make it easy to answer "what can I do with a value of type `T`?" (especially in rustdoc).
- `self` notation is more concise and often conveys ownership distinctions more clearly.

**Google note:** Google adds that a trait is defined only for generic programming, `impl Trait`/`dyn Trait`, or extension traits; otherwise use an inherent impl ([G-TRT-05](../google/07-traits.md#g-trt-05)). Free functions that remain are called module-qualified ([G-ORG-08](../google/02-code-organization.md#g-org-08)).

**Example:**
```rust
// Good
impl Foo {
    pub fn frob(&self, w: widget) { /* ... */ }
}

// Bad
pub fn frob(foo: &Foo, w: widget) { /* ... */ }
```

**See also:** [inherent impls](../references/language-concepts.md#inherent-impls)

<a id="c-no-out"></a>

## C-NO-OUT — Functions do not take out-parameters

**Rule:** A function returning multiple values **SHOULD** return them via a compound type (tuple or struct) rather than through `&mut` out-parameters.

**Exceptions:**
- The primary exception: the function is meant to modify data the caller already owns, e.g. to re-use a buffer (`fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>`).

**Rationale:**
- Compound return types like tuples and structs are compiled efficiently and require no heap allocation.

**Google note:** For the compound return type, Google prefers structs with named fields over tuples, except in very local contexts ([G-TYP-02](../google/06-types.md#g-typ-02)).

**Example:**
```rust
// Good: returning multiple Bar values
fn foo() -> (Bar, Bar)

// Bad
fn foo(output: &mut Bar) -> Bar

// OK (exception): modifies a caller-owned buffer
fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>
```

<a id="c-overload"></a>

## C-OVERLOAD — Operator overloads are unsurprising

**Rule:** Operators with built-in syntax (`*`, `|`, ...) provided via the [`std::ops`](https://doc.rust-lang.org/std/ops/index.html#traits) traits **MUST** be implemented only for operations that resemble the operator's usual meaning and share its expected properties.

**Details:**
- E.g. implement `Mul` only for an operation resembling multiplication that shares its expected properties (such as associativity); likewise for the other traits.

**Rationale:**
- These operators come with strong expectations.

**Google note:** Google narrows this: use operator overloads only for their original mathematical/logical purpose, never merely as concise "syntax sugar" ([G-FEAT-07](../google/12-other-rust-features.md#g-feat-07)).

<a id="c-deref"></a>

## C-DEREF — Only smart pointers implement `Deref` and `DerefMut`

**Rule:** `Deref` and `DerefMut` **SHOULD** be implemented only by smart pointers.

**Details:**
- Std precedents: [`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html); [`String`](https://doc.rust-lang.org/std/string/struct.String.html) (a smart pointer to [`str`](https://doc.rust-lang.org/std/primitive.str.html)); [`Rc<T>`](https://doc.rust-lang.org/std/rc/struct.Rc.html); [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html); [`Cow<'a, T>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html).

**Rationale:**
- The compiler uses the `Deref` traits implicitly in many circumstances and they interact with method resolution; those rules are designed specifically for smart pointers.

**See also:** [C-SMART-PTR](#c-smart-ptr)

<a id="c-ctor"></a>

## C-CTOR — Constructors are static, inherent methods

**Rule:** Constructors **SHOULD** be static (no `self`) inherent methods on the type they construct, and `new` **SHOULD** generally be the name of the primary constructor.

**Details:**
- "Constructors" are only a convention in Rust; naming conventions vary and the distinctions are often subtle.
- The most basic constructor is `new` with no arguments; `new` may also take arguments (e.g. [`Box::new`](https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new) takes the value to box).
- Combined with fully importing type names, this gives informative, concise construction: `Example::new()`.
- Some types, notably I/O resources, use domain-appropriate names instead: [`File::open`](https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open), [`Mmap::open`](https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open), [`TcpStream::connect`](https://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect), [`UdpSocket::bind`](https://doc.rust-lang.org/stable/std/net/struct.UdpSocket.html#method.bind).
- Secondary constructors are commonly suffixed `_with_foo`, e.g. [`Mmap::open_with_offset`](https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset). If a type has a multiplicity of construction options, consider a builder ([C-BUILDER](07-type-safety.md#c-builder)) instead.
- Conversion constructors (creating the type from a value of another type) typically start with `from_`, e.g. [`std::io::Error::from_raw_os_error`](https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error). They differ from a `From<T>` impl ([C-CONV-TRAITS](02-interoperability.md#c-conv-traits)) in three ways:
  - A `from_` constructor can be `unsafe`; a `From` impl cannot (e.g. [`Box::from_raw`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw)).
  - A `from_` constructor can take extra arguments to disambiguate the source data (e.g. [`u64::from_str_radix`](https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix)).
  - A `From` impl is appropriate only when the source type alone determines the output encoding; for a "bag of bits" input like [`u64::from_be`](https://doc.rust-lang.org/std/primitive.u64.html#method.from_be) or [`String::from_utf8`](https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8), the constructor name identifies the meaning.
- Implementing both `Default` and `new` is common and expected; when both exist they **SHOULD** behave the same. Either **MAY** be implemented in terms of the other.
- Std precedents:
  - [`std::io::Error::new`](https://doc.rust-lang.org/std/io/struct.Error.html#method.new): the commonly used constructor for an I/O error.
  - [`std::io::Error::from_raw_os_error`](https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error): conversion constructor from an OS error code.
  - [`Box::new`](https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new): creates a new container, taking a single argument.
  - [`File::open`](https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open): opens a file resource.
  - [`Mmap::open_with_offset`](https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset): opens a memory-mapped file with additional options.

**Google note:** Google names secondary constructors `new_with_foo()`, fallible constructors `try_new()`, and value-to-value conversions `from_foo()` with an associated `From` impl ([G-NAM-05](../google/10-naming.md#g-nam-05)); an important `From` impl must be paired with an inherent `::new()`/`::new_from_foo()`/`::from_foo()` ([G-TRT-08](../google/07-traits.md#g-trt-08)); implement `Default` if the type has `new()` ([G-TRT-07](../google/07-traits.md#g-trt-07)).

**Example:**
```rust
impl<T> Example<T> {
    pub fn new() -> Example<T> { /* ... */ }
}
```

```rust
use example::Example;

// Construct a new Example.
let ex = Example::new();
```

**See also:** [C-BUILDER](07-type-safety.md#c-builder), [C-CONV-SPECIFIC](#c-conv-specific), [`clippy::new_without_default`](../references/clippy-lints.md#new_without_default), [inherent impls](../references/language-concepts.md#inherent-impls), [`From` conversions](../references/language-concepts.md#from-conversions)
