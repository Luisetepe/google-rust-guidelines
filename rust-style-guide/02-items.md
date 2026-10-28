# Items

> **Source:** [Rust Style Guide — Items](https://doc.rust-lang.org/nightly/style-guide/items.html) · **License:** MIT OR Apache-2.0
>
> Formatting of items: ordering, functions, tuples, enums, structs, unions, traits, impls, modules, macros, generics, `where` clauses, type aliases, extern items and `use` imports.

Items are the things permitted at the top level of a module. Some items may also appear inside other items (e.g., inside a function); the same formatting conventions apply wherever an item appears. Keyword strength follows the [default-style convention](01-formatting-conventions.md#the-default-rust-style).

<a id="rs-item-01"></a>

## RS-ITEM-01 — Item ordering

**Rule:** `extern crate` statements **MUST** come first in a file, ordered alphabetically. `use` statements and module *declarations* (`mod foo;`, not `mod { ... }`) **MUST** come before other items, with imports before module declarations.

**Details:**
- Version-sort ([RS-FMT-06](01-formatting-conventions.md#rs-fmt-06)) imports and module declarations each, except that `self` and `super` **MUST** come before any other names.

**Exceptions:**
- Tools **MUST NOT** automatically move module declarations annotated with `#[macro_use]`, since that might change semantics.

**See also:** [RS-ITEM-20](#rs-item-20)

<a id="rs-item-02"></a>

## RS-ITEM-02 — Function definitions

**Rule:** Function definitions **MUST** be formatted so that searching for `fn [function-name]` finds them, using the order and spacing `[pub] [unsafe] [extern ["ABI"]] fn foo(arg1: i32, arg2: i32) -> i32 {`. **SHOULD** avoid comments within the signature.

**Details:**
- If the signature does not fit on one line, break after the opening parenthesis and before the closing parenthesis, and put each argument on its own block-indented line, with a trailing comma after the last argument.

**Example:**
```rust
[pub] [unsafe] [extern ["ABI"]] fn foo(arg1: i32, arg2: i32) -> i32 {
    ...
}
```

```rust
fn foo(
    arg1: i32,
    arg2: i32,
) -> i32 {
    ...
}
```

**See also:** [RS-FMT-03](01-formatting-conventions.md#rs-fmt-03), [RS-ITEM-17](#rs-item-17)

<a id="rs-item-03"></a>

## RS-ITEM-03 — Tuples and tuple structs

**Rule:** **MUST** write the type list of a tuple or tuple struct as a function parameter list ([RS-ITEM-02](#rs-item-02)) and **MUST** build a tuple or tuple struct as a function call ([expressions](04-expressions.md)).

**See also:** [RS-ITEM-07](#rs-item-07)

<a id="rs-item-04"></a>

## RS-ITEM-04 — Tuples and tuple structs: single-line

**Rule:** Single-line form of tuple structs, as shown below (the source gives only this example).

**Example:**
```rust
struct Bar(Type1, Type2);

let x = Bar(11, 22);
let y = (11, 22, 33);
```

<a id="rs-item-05"></a>

## RS-ITEM-05 — Enums

**Rule:** In an enum declaration, **MUST** put each variant on its own block-indented line, formatted as a struct (without the `struct` keyword), a tuple struct, or an identifier (no special formatting).

**Details:**
- A [*small*](01-formatting-conventions.md#rs-fmt-10) struct variant is formatted on one line, without a trailing comma in the field list and with spaces around each brace.
- In an enum with several struct variants, if any struct variant is multi-line, use the multi-line form for all struct variants. Such a situation may indicate that the variant's fields should be factored out into their own struct.

**Example:**
```rust
enum FooBar {
    First(u32),
    Second,
    Error {
        err: Box<Error>,
        line: u32,
    },
}
```

```rust
// Small struct variant
enum FooBar {
    Error { err: Box<Error>, line: u32 },
}
```

<a id="rs-item-06"></a>

## RS-ITEM-06 — Structs and unions

**Rule:** The struct name **MUST** follow the `struct` keyword on the same line, with the opening brace on that line when it fits within the right margin. Fields **MUST** be indented once and each end with a trailing comma; the closing brace is unindented on its own line.

**Details:**
- If and only if a field's type does not fit within the right margin, pull the type down to its own line and indent it again.
- **SHOULD** prefer a unit struct (`struct Foo;`) to an empty struct (`struct Foo();` or `struct Foo {}`, which exist only to simplify code generation). If an empty struct must be used, keep it on one line with no space between the braces: `struct Foo();` or `struct Foo {}`.
- Untagged `union` declarations follow the same rules.

**Example:**
```rust
struct Foo {
    a: A,
    b: B,
}
```

```rust
struct Foo {
    a: A,
    long_name:
        LongType,
}
```

```rust
union Foo {
    a: A,
    b: B,
    long_name:
        LongType,
}
```

<a id="rs-item-07"></a>

## RS-ITEM-07 — Tuple structs

**Rule:** Put the whole tuple struct on one line if possible, separating types with a comma and a space; **MUST NOT** use a trailing comma in a single-line tuple struct, and **MUST NOT** put spaces around the parentheses or semicolon.

**Details:**
- **SHOULD** prefer unit structs to empty tuple structs (which exist only to simplify code generation): `struct Foo;`, not `struct Foo();`.
- For more than a few fields (in particular if it does not fit on one line), **SHOULD** prefer a proper struct with named fields.
- A multi-line tuple struct is block-formatted: one field per line, with a trailing comma.

**Google note:** Google also prefers named fields over tuples and non-newtype tuple structs; see [G-TYP-02](../google/06-types.md#g-typ-02).

**Example:**
```rust
pub struct Foo(String, u8);
```

```rust
pub struct Foo(
    String,
    u8,
);
```

<a id="rs-item-08"></a>

## RS-ITEM-08 — Traits

**Rule:** **MUST** use block indent for trait items. A trait with no items **MUST** be formatted on a single line including its `{}`; otherwise break after the opening brace and before the closing brace.

**Details:**
- Bounds: a space after the colon but not before, and spaces around each `+`.
- **SHOULD** avoid line-breaking in the bounds (consider a `where` clause). **SHOULD** prefer breaking between bounds to breaking an individual bound.
- If the bounds must break: put each bound, including the first, on its own block-indented line, break before each `+`, and put the opening brace on its own line.

**Example:**
```rust
trait Foo {}

pub trait Bar {
    ...
}
```

```rust
trait Foo: Debug + Bar {}
```

```rust
pub trait IndexRanges:
    Index<Range<usize>, Output=Self>
    + Index<RangeTo<usize>, Output=Self>
    + Index<RangeFrom<usize>, Output=Self>
    + Index<RangeFull, Output=Self>
{
    ...
}
```

**See also:** [RS-ITEM-14](#rs-item-14)

<a id="rs-item-09"></a>

## RS-ITEM-09 — Impls

**Rule:** **MUST** use block indent for impl items. An impl with no items **MUST** be formatted on a single line including its `{}`; otherwise break after the opening brace and before the closing brace.

**Details:**
- **SHOULD** avoid line-breaking in the signature. If a break is required in a non-inherent impl, break immediately before `for`, block-indent the concrete type, and put the opening brace on its own line.

**Example:**
```rust
impl Foo {}

impl Bar for Foo {
    ...
}
```

```rust
impl Bar
    for Foo
{
    ...
}
```

<a id="rs-item-10"></a>

## RS-ITEM-10 — Extern crate

**Rule:** **MUST** use spaces around keywords and **MUST NOT** put spaces around the semicolon: `extern crate foo;`.

**See also:** [RS-ITEM-01](#rs-item-01)

<a id="rs-item-11"></a>

## RS-ITEM-11 — Modules

**Rule:** **MUST** use spaces around keywords and before the opening brace, and **MUST NOT** put spaces around the semicolon.

**Example:**
```rust
mod foo {
}
```

```rust
mod foo;
```

<a id="rs-item-12"></a>

## RS-ITEM-12 — `macro_rules!`

**Rule:** **MUST** use `{}` for the full definition of the macro.

**Example:**
```rust
macro_rules! foo {
}
```

<a id="rs-item-13"></a>

## RS-ITEM-13 — Generics

**Rule:** **SHOULD** put a generics clause on one line, breaking other parts of the item declaration rather than the generics clause. If a generics clause is large enough to require line-breaking, **SHOULD** prefer a `where` clause instead.

**Details:**
- No spaces before or after `<`, nor before `>`. A space after `>` only if it is followed by a word or opening brace, not an opening parenthesis.
- A space after each comma; no trailing comma in a single-line generics clause.
- If the clause must span multiple lines: each parameter on its own block-indented line, break after the opening `<` and before the closing `>`, and use a trailing comma.
- An associated type bound in a generic type has spaces around the `=`: `<T: Example<Item = u32>>`.
- **SHOULD** prefer single-letter names for generic parameters.

**Google note:** Google allows short single- or multi-character generic names indicative of the type, ideally from the most important trait bound; see [G-NAM-03](../google/10-naming.md#g-nam-03).

**Example:**
```rust
fn foo<T: Display, U: Debug>(x: Vec<T>, y: Vec<U>) ...

impl<T: Display, U: Debug> SomeType<T, U> { ...
```

```rust
fn foo<
    T: Display,
    U: Debug,
>(x: Vec<T>, y: Vec<U>) ...
```

```rust
<T: Example<Item = u32>>
```

**See also:** [RS-ITEM-14](#rs-item-14)

<a id="rs-item-14"></a>

## RS-ITEM-14 — `where` clauses

**Rule:** These rules apply to `where` clauses on any item. Directly after a closing bracket of any kind, **MUST** write `where` on the same line with a space before it; otherwise **MUST** put `where` on a new line at the same indentation level.

**Details:**
- Each component of the clause goes on its own block-indented line.
- Use a trailing comma, unless the clause is terminated with a semicolon.
- If the clause is followed by a block (or assignment), start that block on a new line.
- If a `where` clause is very short, **SHOULD** prefer an inline bound on the type parameter.
- If a component does not fit and contains `+`, break before each `+` and block-indent the continuation lines, one bound per line.
- `where` clauses on `type` aliases are not enforced and should not be used (but see [RS-ITEM-15](#rs-item-15) for their formatting).

**Example:**
```rust
fn function<T, U>(args)
where
    T: Bound,
    U: AnotherBound,
{
    body
}

fn foo<T>(
    args
) -> ReturnType
where
    T: Bound,
{
    body
}

fn foo<T, U>(
    args,
) where
    T: Bound,
    U: AnotherBound,
{
    body
}

fn foo<T, U>(
    args
) -> ReturnType
where
    T: Bound,
    U: AnotherBound;  // Note, no trailing comma.

// Note that where clauses on `type` aliases are not enforced and should not
// be used.
type Foo<T>
where
    T: Bound
= Bar<T>;
```

```rust
impl<T: ?Sized, Idx> IndexRanges<Idx> for T
where
    T: Index<Range<Idx>, Output = Self::Output>
        + Index<RangeTo<Idx>, Output = Self::Output>
        + Index<RangeFrom<Idx>, Output = Self::Output>
        + Index<RangeInclusive<Idx>, Output = Self::Output>
        + Index<RangeToInclusive<Idx>, Output = Self::Output>
        + Index<RangeFull>,
```

<a id="rs-item-15"></a>

## RS-ITEM-15 — Type aliases

**Rule:** **MUST** keep type aliases on one line when they fit; if a break is needed, break before the `=` and block-indent the right-hand side.

**Details:**
- Only a trailing `where` clause (none before the type): break before the `=` and indent; then break before `where` and format the clauses normally.
- A `where` clause before the type: format it normally and break after the last clause; do not indent before the `=`, keeping it visually distinct from the indented clauses above it. If there is also a trailing `where` clause, break before that `where` and format its clauses normally.

**Example:**
```rust
pub type Foo = Bar<T>;

// If multi-line is required
type VeryLongType<T, U: SomeBound>
    = AnEvenLongerType<T, U, Foo<T>>;
```

```rust
// With only a trailing where clause
type VeryLongType<T, U>
    = AnEvenLongerType<T, U, Foo<T>>
where
    T: U::AnAssociatedType,
    U: SomeBound;
```

```rust
// With only a preceding where clause.
type WithPrecedingWC<T, U>
where
    T: U::AnAssociatedType,
    U: SomeBound,
= AnEvenLongerType<T, U, Foo<T>>;

// Or with both a preceding and trailing where clause.
type WithPrecedingWC<T, U>
where
    T: U::AnAssociatedType,
    U: SomeBound,
= AnEvenLongerType<T, U, Foo<T>>
where
    T: U::AnAssociatedType2,
    U: SomeBound2;
```

<a id="rs-item-16"></a>

## RS-ITEM-16 — Associated types

**Rule:** **MUST** format associated types like type aliases ([RS-ITEM-15](#rs-item-15)). A bound on an associated type has a space after the colon but not before.

**Example:**
```rust
pub type Foo: Bar;
```

<a id="rs-item-17"></a>

## RS-ITEM-17 — Extern items

**Rule:** When writing extern items (such as `extern "C" fn`), **MUST** always specify the ABI.

**Example:**
```rust
// Good
extern "C" fn foo ...
unsafe extern "C" { ... }

// Bad: ABI omitted
extern fn foo ...
unsafe extern { ... }
```

<a id="rs-item-18"></a>

## RS-ITEM-18 — Imports (`use` statements)

**Rule:** Format imports on one line where possible. **MUST NOT** put spaces around braces.

**Google note:** Google forbids glob imports (`use a::b::d::*;`) outside listed exceptions ([G-ORG-05](../google/02-code-organization.md#g-org-05)) and says each `use` should import from a single module ([G-ORG-06](../google/02-code-organization.md#g-org-06)).

**Example:**
```rust
use a::b::c;
use a::b::d::*;
use a::b::{foo, bar, baz};
```

<a id="rs-item-19"></a>

## RS-ITEM-19 — Large list imports

**Rule:** **SHOULD** prefer multiple imports to a multi-line import; however, tools **SHOULD NOT** split imports by default.

**Details:**
- If an import requires multiple lines (a list of single names does not fit the max width, or the [nested imports](#rs-item-23) rules apply): break after the opening brace and before the closing brace, use a trailing comma, and block-indent the names.

**Google note:** Google's G-ORG-06 says each `use` should import from one module (no `::` inside `{..}`), corresponding to `imports_granularity = Module`; both split and merged per-module forms are acceptable; see [G-ORG-06](../google/02-code-organization.md#g-org-06).

**Example:**
```rust
// Prefer
foo::{long, list, of, imports};
foo::{more, imports};

// If necessary
foo::{
    long, list, of, imports, more,
    imports,  // Note trailing comma
};
```

<a id="rs-item-20"></a>

## RS-ITEM-20 — Ordering of imports

**Rule:** Within a group of imports, imports **MUST** be version-sorted ([RS-FMT-06](01-formatting-conventions.md#rs-fmt-06)). Groups of imports **MUST NOT** be merged or re-ordered.

**Details:**
- A *group* is a set of imports on the same or sequential lines; one or more blank lines or other items (e.g., a function) separate groups.
- Because of `macro_use`, attributes also start a new group and prevent re-ordering.

**Example:**
```rust
// Input
use d;
use c;

use b;
use a;
```

```rust
// Output
use c;
use d;

use a;
use b;
```

**See also:** [RS-ITEM-01](#rs-item-01)

<a id="rs-item-21"></a>

## RS-ITEM-21 — Ordering list import

**Rule:** Names in a list import **MUST** be version-sorted, except that `self` and `super` always come first if present, and groups and glob imports always come last if present.

**Details:**
- This applies recursively: `a::*` comes before `b::a`, but `a::b` comes before `a::*`. E.g., `use foo::bar::{a, b::c, b::d, b::d::{x, y, z}, b::{self, r, s}};`.

**Google note:** Under Google's [G-ORG-06](../google/02-code-organization.md#g-org-06) there are no `::` paths inside braces, so the recursive cases do not arise in Google code.

<a id="rs-item-22"></a>

## RS-ITEM-22 — Normalisation

**Rule:** Tools **MUST** make the following normalisations, recursively, and **MUST NOT** otherwise merge or un-merge import lists or adjust glob imports without an explicit option.

**Details:**
- `use a::self;` -> `use a;`
- `use a::{};` -> (nothing)
- `use a::{b};` -> `use a::b;`

<a id="rs-item-23"></a>

## RS-ITEM-23 — Nested imports

**Rule:** If a list import contains any nested imports, **MUST** use the multi-line form even if it fits on one line. Each nested import goes on its own line; non-nested imports are grouped on as few lines as possible.

**Google note:** Google says a `use` should not nest paths (no `::` inside `{..}`); see [G-ORG-06](../google/02-code-organization.md#g-org-06).

**Example:**
```rust
use a::b::{
    x, y, z,
    u::{...},
    w::{...},
};
```

<a id="rs-item-24"></a>

## RS-ITEM-24 — Merging/un-merging imports

**Rule:** Tools **MUST NOT** merge or un-merge imports by default. They **MAY** offer merging or un-merging as an option.

**Google note:** Google forbids only cross-module merging (`::` inside `{..}`) per G-ORG-06, which corresponds to `imports_granularity = Module`; it accepts both merged and unmerged imports from a single module; see [G-ORG-06](../google/02-code-organization.md#g-org-06).

**Example:**
```rust
// Un-merged
use a::b;
use a::c::d;

// Merged
use a::{b, c::d};
```

**See also:** [`imports_granularity`](../references/rustfmt-config.md#imports_granularity)
