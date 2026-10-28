# Formatting Conventions

> **Source:** [Rust Style Guide — Introduction / Formatting conventions](https://doc.rust-lang.org/nightly/style-guide/) · **License:** MIT OR Apache-2.0
>
> The default Rust style: why to use a formatter, and the general formatting conventions (indentation, width, commas, blank lines, whitespace, sorting, comments, attributes, *small* items).

## Motivation: why use a formatting tool?

- Formatting is mostly mechanical and costs time and mental effort; an automatic formatter frees programmers for more important work.
- An established style guide removes the need to invent ad hoc rules or debate them, saving time, communication overhead and mental energy.
- Humans comprehend by pattern matching; uniformly formatted Rust lowers the effort to comprehend a new project and the barrier to entry for new developers.
- So a formatting tool (such as `rustfmt`) gives productivity benefits, and community-consistent formatting, typically the tool's default settings, gives even larger ones.

## The default Rust style

The Rust Style Guide defines the default Rust style and *recommends* that developers and tools follow it; tools such as `rustfmt` use it as their reference. Everything in the guide, whether phrased as "must" or in the imperative ("insert a space ...", "break the line after ..."), refers to the default style. It does not forbid developers from following a non-default style, nor forbid tools from adding configuration options. In this repo, **MUST**/**MUST NOT** mark imperatives of the default style and **SHOULD** marks the source's "prefer"/"avoid"/recommendations.

## Bugs

- If the style guide and `rustfmt` differ, it may be a bug in either; report it to the style team, the rustfmt team, or both.
- A new formatting tool based on the style guide should be tested on the corpus of existing Rust code and avoid causing widespread breakage; doing so may surface bugs in the style guide, `rustfmt`, or the tool itself.
- Bugs are typically resolved in a way that avoids widespread breakage.

<a id="rs-fmt-01"></a>

## RS-FMT-01 — Indentation and line width

**Rule:** Indent with spaces; **MUST NOT** use tabs. Each indentation level **MUST** be 4 spaces. Lines **MUST NOT** exceed the maximum width of 100 characters.

**Details:**
- All indentation outside string literals and comments is a multiple of 4 spaces.

**Google note:** Google's rustfmt configuration keeps `max_width = 100` but lets a codebase skip options unavailable on its stable Rust; see [G-FMT-01](../google/03-formatting.md#g-fmt-01).

**See also:** [RS-FMT-02](#rs-fmt-02), [G-FMT-02](../google/03-formatting.md#g-fmt-02), [`max_width`](../references/rustfmt-config.md#max_width)

<a id="rs-fmt-02"></a>

## RS-FMT-02 — Block indent

**Rule:** **SHOULD** prefer block indent over visual indent.

**Rationale:**
- Smaller diffs (e.g., renaming `a_function_call` below does not re-indent the arguments) and less rightward drift.

**Example:**
```rust
// Good: block indent
a_function_call(
    foo,
    bar,
);

// Bad: visual indent
a_function_call(foo,
                bar);
```

<a id="rs-fmt-03"></a>

## RS-FMT-03 — Trailing commas

**Rule:** In comma-separated lists of any kind, **MUST** use a trailing comma when the last element is followed by a newline.

**Rationale:**
- Moving code (e.g., copy and paste) is easier, and diffs are smaller: appending or removing items does not modify another line to add or remove a comma.

**Example:**
```rust
function_call(
    argument,
    another_argument,
);

let array = [
    element,
    another_element,
    yet_another_element,
];
```

<a id="rs-fmt-04"></a>

## RS-FMT-04 — Blank lines

**Rule:** **MUST** separate items and statements by either zero or one blank lines (i.e., one or two newlines).

**Example:**
```rust
fn foo() {
    let x = ...;

    let y = ...;
    let z = ...;
}

fn bar() {}
fn baz() {}
```

<a id="rs-fmt-05"></a>

## RS-FMT-05 — Trailing whitespace

**Rule:** **MUST NOT** include trailing whitespace at the end of any line: blank lines, comment lines, code lines and string literals.

**Details:**
- Avoiding trailing whitespace inside string literals requires care to preserve the literal's value.

<a id="rs-fmt-06"></a>

## RS-FMT-06 — Sorting (version sorting)

**Rule:** Wherever the default style says to sort and specifies nothing else, sorting **MUST** be "version sorting", so that e.g. `x8` comes before `x16` even though `1` precedes `8`.

**Details:**
- Comparing two strings:
  - Split each string, from beginning to end, into maximal-length chunks: either a run of non-ASCII-digit characters or a run of ASCII digits (a numeric chunk). Compare corresponding chunks.
  - Two numeric chunks: compare by numeric value, ignoring leading zeroes. If the values are equal but the number of leading digits differs, and this is the first time that has happened for these strings, treat the chunks as equal (move on) but remember which string had more leading zeroes.
  - Two chunks that are not both numeric: compare by Unicode character lexicographically, with two exceptions:
    - `_` (underscore) sorts immediately after ` ` (space) but before any other character (treats underscore as a word separator, as in identifiers).
    - Unless otherwise specified, non-lowercase characters (those that can start an `UpperCamelCase` identifier) sort before lowercase characters.
  - If the end of the strings is reached with every chunk pair equal:
    - If a numeric comparison recorded the earliest point at which one string had more leading zeroes, the string with more leading zeroes sorts first.
    - Otherwise the strings are equal.
- Various other algorithms are called "version sorting" and differ (e.g., in handling leading zeroes). This one does not try to match any of them; it aims for a simple, satisfying result for symbols with the same number of leading zeroes, and an acceptable, easily understood result when the number of leading zeroes varies.
- Example: version sorting orders these strings as given:
  - `_ZYXW`
  - `_abcd`
  - `A2`
  - `ABCD`
  - `Z_YXW`
  - `ZY_XW`
  - `ZY_XW`
  - `ZYXW`
  - `ZYXW_`
  - `a1`
  - `abcd`
  - `u_zzz`
  - `u8`
  - `u16`
  - `u32`
  - `u64`
  - `u128`
  - `u256`
  - `ua`
  - `usize`
  - `uz`
  - `v000`
  - `v00`
  - `v0`
  - `v0s`
  - `v00t`
  - `v0u`
  - `v001`
  - `v01`
  - `v1`
  - `v009`
  - `v09`
  - `v9`
  - `v010`
  - `v10`
  - `w005s09t`
  - `w5s009t`
  - `x64`
  - `x86`
  - `x86_32`
  - `x86_64`
  - `x86_128`
  - `x87`
  - `zyxw`

**See also:** [RS-ITEM-01](02-items.md#rs-item-01), [RS-ITEM-20](02-items.md#rs-item-20), [RS-ITEM-21](02-items.md#rs-item-21)

## Construct-specific chapters

The formatting conventions continue per construct:

- [Module-level items](02-items.md)
- [Statements](03-statements.md)
- [Expressions](04-expressions.md)
- [Types](05-types-and-bounds.md)

<a id="rs-fmt-07"></a>

## RS-FMT-07 — Comments

**Rule:** These are recommendations only; a mechanical formatter may skip formatting comments. **SHOULD** prefer line comments (`//`) to block comments (`/* ... */`), and **SHOULD** put a comment on its own line.

**Details:**
- Line comments: a single space after the opening sigil.
- Single-line block comments: a single space after the opening sigil and before the closing sigil.
- Multi-line block comments: a newline after the opening sigil and a newline before the closing sigil.
- A comment following code: a single space before it.
- An inline block comment: surrounding whitespace as if it were an identifier or keyword.
- Comments should usually be complete sentences: start with a capital letter, end with a period (`.`). An inline block comment may be a note without punctuation.
- A source line that is entirely a comment should be limited to the smaller of: 80 characters (including comment sigils, excluding indentation), or the maximum line width (including comment sigils and indentation).

**Example:**
```rust
// A comment on an item.
struct Foo { ... }

fn foo() {} // A comment after an item.

pub fn foo(/* a comment before an argument */ x: T) {...}
```

```rust
// This comment goes up to the ................................. 80 char margin.

{
    // This comment is .............................................. 80 chars wide.
}

{
    {
        {
            {
                {
                    {
                        // This comment is limited by the ......................... 100 char margin.
                    }
                }
            }
        }
    }
}
```

**See also:** [RS-FMT-08](#rs-fmt-08)

<a id="doc-comments"></a>
<a id="rs-fmt-08"></a>

## RS-FMT-08 — Doc comments

**Rule:** **SHOULD** prefer line doc comments (`///`) to block doc comments (`/** ... */`). **SHOULD** prefer outer doc comments (`///` or `/** ... */`); use inner doc comments (`//!` and `/*! ... */`) only for module-level or crate-level documentation. **MUST** put doc comments before attributes.

**Google note:** Google adopts these rules and adds: `///` for items, `//!` at the top of the file for module/library docs, Markdown syntax, `//` for non-documentation comments, and hand-written doc comments over macro-generated docs; see [G-DOC](../google/11-documentation.md).

**Example:**
```rust
/// Good: doc comment before the attribute.
#[derive(Debug)]
struct Foo;
```

<a id="rs-fmt-09"></a>

## RS-FMT-09 — Attributes

**Rule:** **MUST** put each attribute on its own line, indented to the level of the item; inner attributes (`#!`) are indented to the level of the inside of the item. **SHOULD** prefer outer attributes where possible.

**Details:**
- Attributes with argument lists are formatted like functions (see [function calls](04-expressions.md)).
- Attributes with an equal sign: a single space before and after the `=`, e.g., `#[foo = 42]`.
- There **MUST** be only a single `derive` attribute.
- For tool authors: when combining multiple `derive` attributes into one, the order of the derived names must generally be preserved for correctness: `#[derive(Foo)] #[derive(Bar)] struct Baz;` must be formatted to `#[derive(Foo, Bar)] struct Baz;`.

**Example:**
```rust
#[repr(C)]
#[foo(foo, bar)]
#[long_multi_line_attribute(
    split,
    across,
    lines,
)]
struct CRepr {
    #![repr(C)]
    x: f32,
    y: f32,
}
```

**See also:** [RS-FMT-08](#rs-fmt-08)

<a id="rs-fmt-10"></a>

## RS-FMT-10 — *Small* items

**Rule:** Where this guide gives formatting that depends on a construct being *small* (e.g., single-line vs multi-line struct literals), each tool decides exactly what *small* means and **MAY** use different definitions in different circumstances.

**Details:**
- Suitable heuristics: the item's size in characters, or its complexity (e.g., all components must be simple names, not complex sub-expressions). Discussion: [fmt-rfcs issue #47](https://github.com/rust-lang-nursery/fmt-rfcs/issues/47).

**Example:**
```rust
// Normal formatting
Foo {
    f1: an_expression,
    f2: another_expression(),
}

// "small" formatting
Foo { f1, f2 }
```

**See also:** [RS-ITEM-05](02-items.md#rs-item-05)

## Other chapters

- [Non-formatting conventions](06-other-style-advice.md)
- [Cargo.toml conventions](07-cargo-toml.md)
- [Principles used for deciding these guidelines](08-guiding-principles.md)
