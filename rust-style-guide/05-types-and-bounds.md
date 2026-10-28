# Types and Bounds

> **Source:** [Rust Style Guide — Types and Bounds](https://doc.rust-lang.org/nightly/style-guide/types.html) · **License:** MIT OR Apache-2.0
>
> Spacing of types written on one line, where to break types that do not fit, and formatting of precise capturing bounds.

Terms such as block indent and maximum width are defined in [Formatting conventions](01-formatting-conventions.md).

<a id="rs-type-01"></a>

## RS-TYPE-01 — Single line formatting

**Rule:** Format types on a single line with the spacing below. **MUST NOT** put spaces around parentheses used in types, e.g. `(Foo)`.

**Details:**
- `[T]`: no spaces.
- `[T; expr]`, e.g. `[u32; 42]`, `[Vec<Foo>; 10 * 2 + foo()]`: space after the `;`, no spaces around the square brackets.
- `*const T`, `*mut T`: no space after `*`, space before the type.
- `&'a T`, `&T`, `&'a mut T`, `&mut T`: no space after `&`, single spaces separating other words.
- `unsafe extern "C" fn<'a, 'b, 'c>(T, U, V) -> W` or `fn()`: single spaces around keywords and sigils and after commas; no trailing commas; no spaces around brackets.
- `!` is treated like any other type name, e.g. `Name`.
- `(A, B, C, D)`: spaces after commas, no spaces around parens, no trailing comma unless it is a one-tuple (`(A,)`).
- `<Baz<T> as SomeTrait>::Foo::Bar`, `Foo::Bar`, `::Foo::Bar`: no spaces around `::` or angle brackets; single spaces around `as`.
- `Foo::Bar<T, U, V>`: spaces after commas, no trailing comma, no spaces around angle brackets.
- `T + T + T`: single spaces between types and `+`.
- `impl T + T + T`: single spaces between the keyword, types, and `+`.

<a id="rs-type-02"></a>

## RS-TYPE-02 — Line breaks

**Rule:** **SHOULD** avoid breaking lines in types where possible; when a break is needed, **SHOULD** prefer breaking at the outermost scope.

**Details:**
- `[T; expr]`: break after the `;` if necessary.
- Function types: break following the rules for function declarations ([Items](02-items.md)).
- Generic types: break following the rules for generics ([Items](02-items.md)).
- Types with `+`: break before the `+` and block-indent the subsequent lines. When breaking such a type, break before *every* `+`.

**Example:**
```rust
// Good: break at the outermost scope
Foo<
    Bar,
    Baz<Type1, Type2>,
>

// Bad: break in an inner scope
Foo<Bar, Baz<
    Type1,
    Type2,
>>
```

```rust
// Breaking `+` types: break before every `+`
impl Clone
    + Copy
    + Debug

Box<
    Clone
    + Copy
    + Debug
>
```

<a id="rs-type-03"></a>

## RS-TYPE-03 — Precise capturing bounds

**Rule:** Format a `use<'a, T>` precise capturing bound as if it were a single path segment with non-turbofished angle-bracketed args, i.e. like a trait bound whose identifier is `use`.

**Example:**
```rust
fn foo() -> impl Sized + use<'a> {}

// is formatted analogously to:

fn foo() -> impl Sized + Use<'a> {}
```
