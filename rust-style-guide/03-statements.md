# Statements

> **Source:** [Rust Style Guide — Statements](https://doc.rust-lang.org/nightly/style-guide/statements.html) · **License:** MIT OR Apache-2.0
>
> Formatting of `let` (including let-else) statements, macro uses in statement position, and expression statements.

Terms such as *small*, *short*, block indent and maximum width are defined in [Formatting conventions](01-formatting-conventions.md).

<a id="rs-stmt-01"></a>

## RS-STMT-01 — Let statements

**Rule:** Put a space after the `:` and on both sides of the `=` (if present); **MUST NOT** put a space before the semicolon. Format the declaration on a single line if possible; otherwise break as described below, using block indentation.

**Details:**
- Line-breaking order:
  1. Single line if it fits.
  2. Otherwise split after the `=` if the declaration then fits on two lines; block-indent the expression.
  3. If the first line still does not fit, also split after the `:`, block-indented.
  4. If the type needs multiple lines even after breaking after `:`, put its first line on the same line as the `:`, subject to the [combining rules](04-expressions.md#rs-expr-26).
- Multi-line initializer expression:
  - If the expression's first line fits in the remaining space, it stays on the same line as the `=` and the rest of the expression is not further indented.
  - If the first line does not fit, put the expression on subsequent lines, block-indented.
  - If the expression is a block and the type or pattern spans multiple lines, put the opening brace on a new line, not indented (separates the block interior from the type); otherwise the opening brace follows the `=`.

**Example:**
```rust
// A comment.
let pattern: Type = expr;

let pattern;
let pattern: Type;
let pattern = expr;
```

```rust
// Split after `=`
let pattern: Type =
    expr;

// Split after `:` too
let pattern:
    Type =
    expr;
```

```rust
// Multi-line pattern/type
let Foo {
    f: abcd,
    g: qwer,
}: Foo<Bar> =
    Foo { f, g };

let (abcd,
    defg):
    Baz =
{ ... }
```

```rust
let foo = Foo {
    f: abcd,
    g: qwer,
};

let foo =
    ALongName {
        f: abcd,
        g: qwer,
    };

let foo: Type = {
    an_expression();
    ...
};

let foo:
    ALongType =
{
    an_expression();
    ...
};

let Foo {
    f: abcd,
    g: qwer,
}: Foo<Bar> = Foo {
    f: blimblimblim,
    g: blamblamblam,
};

let Foo {
    f: abcd,
    g: qwer,
}: Foo<Bar> = foo(
    blimblimblim,
    blamblamblam,
);
```

**See also:** [RS-STMT-02](#rs-stmt-02), [RS-EXPR-26](04-expressions.md#rs-expr-26)

<a id="rs-stmt-02"></a>

## RS-STMT-02 — else blocks (let-else statements)

**Rule:** In a let-else statement, format the part before `else` (`let pattern: Type = initializer_expr`) exactly as other [let statements](#rs-stmt-01). When breaking across lines, **MUST NOT** break between `else` and `{`, and **MUST** always break before the `}`.

**Details:**
- Single line: format the whole statement on one line only if all hold:
  - the entire statement is *short*;
  - the `else` block contains only a single-line expression and no statements;
  - the `else` block contains no comments;
  - the components before `else` fit on a single line.
- Components before `else` fit on one line, but the statement doesn't qualify for one line: put `else {` on the same line as the initializer, separated by a space, then break after `{`. Indent the closing `}` to match `let`; indent the block contents one step further.
- Components before `else` fit on one line, but `else {` does not fit on that line: break before `else` (with `else` at the `let`'s indentation).
- Multi-line initializer: put `else {` on the same line as the end of the initializer, separated by a space, if and only if all hold:
  - the initializer ends with one or more closing parentheses, square brackets and/or braces;
  - there is nothing else on that line;
  - that line has the same indentation level as the initial `let`.
- Otherwise put `else {` on the line after the end of the initializer, with `else` at the same indentation level as `let`.

**Example:**
```rust
// Single line
let Some(1) = opt else { return };

// `else {` on the initializer line
let Some(1) = opt else {
    return;
};

let Some(1) = opt else {
    // nope
    return
};
```

```rust
    // `else {` doesn't fit: break before `else`
    let Some(x) = some_really_really_really_really_really_really_really_really_really_long_name
    else {
        return;
    };
```

```rust
// Multi-line initializer ending in `)` at `let` indentation: `else {` on the same line
let Some(x) = y.foo(
    "abc",
    fairly_long_identifier,
    "def",
    "123456",
    "string",
    "cheese",
) else {
    bar()
}
```

```rust
// Otherwise: `else {` on the next line, aligned with `let`
fn main() {
    let Some(x) = abcdef()
        .foo(
            "abc",
            some_really_really_really_long_ident,
            "ident",
            "123456",
        )
        .bar()
        .baz()
        .qux("fffffffffffffffff")
    else {
        return
    };

    let Some(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa) =
        bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
    else {
        return;
    };

    let LongStructName(AnotherStruct {
        multi,
        line,
        pattern,
    }) = slice.as_ref()
    else {
        return;
    };

    let LongStructName(AnotherStruct {
        multi,
        line,
        pattern,
    }) = multi_line_function_call(
        arg1,
        arg2,
        arg3,
        arg4,
    ) else {
        return;
    };
}
```

**See also:** [RS-STMT-01](#rs-stmt-01), [G-IMPL-02](../google/13-implementation-concerns.md#g-impl-02)

<a id="rs-stmt-03"></a>

## RS-STMT-03 — Macros in statement position

**Rule:** A macro use in statement position **MUST** use parentheses or square brackets as delimiters and be terminated with a semicolon. **MUST NOT** put spaces around the name, `!`, the delimiters, or the `;`.

**Example:**
```rust
// A comment.
a_macro!(...);
```

**See also:** [RS-EXPR-18](04-expressions.md#rs-expr-18)

<a id="rs-stmt-04"></a>

## RS-STMT-04 — Expressions in statement position

**Rule:** **MUST NOT** put a space between the expression and the semicolon. Terminate every expression in statement position with a semicolon, unless it ends with a block or is used as the value of a block.

**Details:**
- Use a semicolon where an expression has void (`()`) type, even if the value could be propagated.

**Example:**
```text
<expr>;
```

```rust
{
    an_expression();
    expr_as_value()
}

return foo();

loop {
    break;
}
```

```rust
fn foo() { ... }

fn bar() {
    foo(); // semicolon even though `()` could be propagated
}
```
