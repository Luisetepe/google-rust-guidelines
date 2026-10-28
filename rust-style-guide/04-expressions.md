# Expressions

> **Source:** [Rust Style Guide — Expressions](https://doc.rust-lang.org/nightly/style-guide/expressions.html) · **License:** MIT OR Apache-2.0
>
> Formatting of every expression form: blocks, closures, literals, indexing, operators, calls, macros, chains, control flow, `match`, combinable expressions, ranges, hex literals and patterns.

Terms such as *small*, block indent, visual indent and maximum width are defined in [Formatting conventions](01-formatting-conventions.md). Google's rustfmt configuration (e.g. `max_width = 100`) is in [G-FMT-01](../google/03-formatting.md#g-fmt-01).

<a id="rs-expr-01"></a>

## RS-EXPR-01 — Blocks

**Rule:** A block expression **MUST** have a newline after the opening `{` and before the closing `}`, unless another rule allows it on a single line. A keyword before the block (e.g. `unsafe`, `async`) **MUST** be on the same line as the opening brace, separated by a single space. Indent the block contents.

**Details:**
- Put an attribute on a block on its own line before the block.
- **SHOULD NOT** write comments on the same line as either brace.
- Write an empty block as `{}`.
- Write a block on a single line only if all hold:
  - it is in expression position (not statement position), or is an `unsafe` block in statement position;
  - it contains a single-line expression and no statements;
  - it contains no comments.
- A single-line block has spaces after `{` and before `}`.

**Example:**
```rust
fn block_as_stmt() {
    a_call();

    {
        a_call_inside_a_block();

        // a comment in a block
        the_value
    }
}

fn block_as_expr() {
    let foo = {
        a_call_inside_a_block();

        // a comment in a block
        the_value
    };
}

fn unsafe_block_as_stmt() {
    a_call();

    unsafe {
        a_call_inside_a_block();

        // a comment in a block
        the_value
    }
}
```

```rust
fn block_as_stmt() {
    #[an_attribute]
    {
        #![an_inner_attribute]

        // a comment in a block
        the_value
    }
}
```

```rust
fn main() {
    // Single line
    let _ = { a_call() };
    let _ = unsafe { a_call() };

    // Not allowed on one line
    // Statement position.
    {
        a_call()
    }

    // Contains a statement
    let _ = {
        a_call();
    };
    unsafe {
        a_call();
    }

    // Contains a comment
    let _ = {
        // A comment
    };
    let _ = {
        // A comment
        a_call()
    };

    // Multiple lines
    let _ = {
        a_call();
        another_call()
    };
    let _ = {
        a_call(
            an_argument,
            another_arg,
        )
    };
}
```

<a id="rs-expr-02"></a>

## RS-EXPR-02 — Closures

**Rule:** **MUST NOT** put extra spaces before the first `|` (unless prefixed by a keyword such as `move`); put a space between the second `|` and the body. Between the `|`s use function-definition syntax, eliding types where possible. Omit the enclosing `{}` where possible.

**Details:**
- Add `{}` when there is a return type, statements, comments inside the closure, or the body is a control-flow expression spanning multiple lines.
- With braces, follow the [block rules](#rs-expr-01).

**Example:**
```rust
|arg1, arg2| expr

move |arg1: i32, arg2: i32| -> i32 {
    expr1;
    expr2
}

|| Foo {
    field1,
    field2: 0,
}

|| {
    if true {
        blah
    } else {
        boo
    }
}

|x| unsafe {
    expr
}
```

<a id="rs-expr-03"></a>

## RS-EXPR-03 — Struct literals

**Rule:** If a struct literal is *small*, format it on one line with no trailing comma; otherwise put each field on its own block-indented line with a trailing comma.

**Details:**
- For each `field: value`, put a space after the colon only.
- Put a space before the opening brace; in single-line form, put spaces after `{` and before `}`.
- Functional record update (`..base`) is treated like a field but **MUST NOT** have a trailing comma; no space after `..`.

**Example:**
```rust
Foo { field1, field2: 0 }
let f = Foo {
    field1,
    field2: an_expr,
};
```

```rust
let f = Foo {
    field1,
    ..an_expr
};
```

<a id="rs-expr-04"></a>

## RS-EXPR-04 — Unit literals

**Rule:** **MUST NOT** break between the parentheses of the `()` unit literal, even when the `)` would exceed the maximum line width.

<a id="rs-expr-05"></a>

## RS-EXPR-05 — Tuple literals

**Rule:** Use a single-line form where possible: no spaces after `(` or before `)`, elements separated by `, `. Otherwise put each element on its own block-indented line with a trailing comma.

**Example:**
```rust
(a, b, c)

let x = (
    a_long_expr,
    another_very_long_expr,
);
```

<a id="rs-expr-06"></a>

## RS-EXPR-06 — Tuple struct literals

**Rule:** **MUST NOT** put a space between the identifier and `(`; otherwise follow [tuple literal](#rs-expr-05) rules.

**Example:**
```rust
Foo(a, b, c)

let x = Foo(
    a_long_expr,
    another_very_long_expr,
);
```

<a id="rs-expr-07"></a>

## RS-EXPR-07 — Enum literals

**Rule:** Follow the rules for the corresponding struct literal form. **SHOULD** qualify variants with the enum name, unless the enum is in the prelude.

**Example:**
```rust
Foo::Bar(a, b)
Foo::Baz {
    field1,
    field2: 1001,
}
Ok(an_expr)
```

<a id="rs-expr-08"></a>

## RS-EXPR-08 — Array literals

**Rule:** Write small array literals on one line: no spaces after `[` or before `]`, elements separated by `, `. With the repeat initializer, put a space after the `;` only. Apply the same rules to `vec!` and similar array-like macros, which **MUST** always use square brackets.

**Details:**
- Breaking across lines: with a repeat initializer, break after the `;`, not before; otherwise follow the [function call](#rs-expr-16) rules.
- In all cases block-indent the contents and break after `[` and before `]`.

**Example:**
```rust
fn main() {
    let x = [1, 2, 3];
    let y = vec![a, b, c, d];
    let a = [42; 10];
}
```

```rust
fn main() {
    [
        a_long_expression();
        1234567890
    ]
    let x = [
        an_expression,
        another_expression,
        a_third_expression,
    ];
}
```

<a id="rs-expr-09"></a>

## RS-EXPR-09 — Array accesses, indexing, and slicing

**Rule:** **MUST NOT** put spaces around the square brackets. **SHOULD** avoid line breaks; **MUST NOT** break between the target expression and `[`.

**Details:**
- If the index expression must move to a subsequent line or itself spans multiple lines, block-indent it and break after `[` and before `]`.

**Example:**
```rust
fn main() {
    foo[42];
    &foo[..10];
    bar[0..100];
    foo[4 + 5 / bar];
    a_long_target[
        a_long_indexing_expression
    ];
}
```

<a id="rs-expr-10"></a>

## RS-EXPR-10 — Unary operations

**Rule:** **MUST NOT** put a space between a unary operator and its operand (`!x`, not `! x`), except that `&mut` **MUST** be followed by a space. **SHOULD** avoid breaking between a unary operator and its operand.

<a id="rs-expr-11"></a>

## RS-EXPR-11 — Binary operations

**Rule:** Put spaces around binary operators (`x + 1`, not `x+1`), including `=` and compound assignment operators (`+=`, `*=`, ...).

**Details:**
- Comparisons: since `&T op &U` is implemented whenever `T op U` is, with `t: &T` and `u: U` **SHOULD** prefer `*t op u` over `t op &u`. Generally within expressions prefer dereferencing to taking references unless necessary (e.g. to avoid an unnecessarily expensive operation).
- Use parentheses liberally; don't necessarily elide them because of precedence. Tools **SHOULD NOT** automatically insert or remove parentheses. **MUST NOT** use spaces to indicate precedence.
- Line-breaking: block-indent each subsequent line; break after assignment operators, and before all other operators (operator starts the next line). Put each sub-expression on its own line.
- **SHOULD** prefer breaking at an assignment operator (`=`, `+=`, ...) over other binary operators.

**Example:**
```rust
foo_bar
    + bar
    + baz
    + qux
    + whatever
```

<a id="rs-expr-12"></a>

## RS-EXPR-12 — Casts (`as`)

**Rule:** Format `as` like a binary operator: always spaces around `as`; when breaking, break before `as` (never after) and block-indent the next line. Format the target type using the [type rules](05-types-and-bounds.md).

**Details:**
- Chained casts: if breaking before the first `as` makes the rest fit on the next line, do not break before later `as`; keep the series of types on one line.
- If the next line still needs breaking, break and block-indent before each `as` as with other binary operators.

**Example:**
```rust
let cstr = very_long_expression()
    as *const str as *const [u8] as *const std::os::raw::c_char;
```

<a id="rs-expr-13"></a>

## RS-EXPR-13 — Control flow

**Rule:** **MUST NOT** add extraneous parentheses around `if` and `while` conditions.

**Exceptions:**
- Extraneous parentheses are fine if they make an arithmetic or logic expression easier to understand (`(x * 15) + (y * 20)`).

**Example:**
```rust
// Bad
if (true) {
}

// Good
if true {
}
```

<a id="rs-expr-14"></a>

## RS-EXPR-14 — Function calls

**Rule:** **MUST NOT** put a space between the function name and `(`, or between an argument and its following comma; put a space after each comma before the next argument. **SHOULD NOT** break inside the callee expression.

**Details:**
- Nullary calls (`func()`): never break within or put a space between the parentheses; always single-line, even if the `)` exceeds the maximum line width.

**See also:** [RS-EXPR-15](#rs-expr-15), [RS-EXPR-16](#rs-expr-16)

<a id="rs-expr-15"></a>

## RS-EXPR-15 — Single-line calls

**Rule:** **MUST NOT** put a space between the function name and `(`, after `(`, or before `)`. **MUST NOT** put a comma after the last argument.

**Example:**
```rust
foo(x, y, z)
```

<a id="rs-expr-16"></a>

## RS-EXPR-16 — Multi-line calls

**Rule:** Format a call across multiple lines if it is not *small*, would exceed the max width, or any argument or the callee is multi-line. Then put each argument on its own block-indented line, break after `(` and before `)`, and use a trailing comma.

**Example:**
```rust
a_function_call(
    arg1,
    a_nested_call(a, b),
)
```

**See also:** [RS-EXPR-26](#rs-expr-26)

<a id="rs-expr-17"></a>

## RS-EXPR-17 — Method calls

**Rule:** Follow the [function call](#rs-expr-14) rules. **MUST NOT** put spaces around the `.`.

**Example:**
```rust
x.foo().bar().baz(x, y, z);
```

<a id="rs-expr-18"></a>

## RS-EXPR-18 — Macro uses

**Rule:** If a macro use can be parsed like another construct, format it like that construct (e.g. `foo!(a, b, c)` parses like a function call ignoring the `!`, so use [function call](#rs-expr-14) rules).

**Details:**
- The style guide defines formatting for specific macros of the language/standard library only; it defines none for third-party macros, even if similar to std ones.

**Google note:** Google also formats macro matchers (`format_macro_matchers = true`) — see [G-FMT-01](../google/03-formatting.md#g-fmt-01) and [Google macro rules](../google/09-macros.md).

**See also:** [RS-STMT-03](03-statements.md#rs-stmt-03), [`format_macro_matchers`](../references/rustfmt-config.md#format_macro_matchers)

<a id="rs-expr-19"></a>

## RS-EXPR-19 — Format string macros

**Rule:** For macros taking a format string, if all other arguments are *small*, put the arguments before the format string on one line if they fit, the format string on its own line, and the arguments after it on one line if they fit. If arguments are not small or don't fit, put each on its own line as for a function call.

**Example:**
```rust
println!(
    "Hello {} and {}",
    name1, name2,
);

assert_eq!(
    x, y,
    "x and y were not equal, see {}",
    reason,
);
```

**See also:** [G-IMPL-08](../google/13-implementation-concerns.md#g-impl-08)

<a id="rs-expr-20"></a>

## RS-EXPR-20 — Chains of fields and method calls

**Rule:** A chain is a sequence of field accesses, method calls and/or `?` (e.g. `a.b.c().d`, `foo?.bar().baz?`). Format it on one line if it is *small* and otherwise possible; else put each field access or method call on its own block-indented line, breaking before the `.` and after any `?`.

**Details:**
- If the length of the last line of the first element plus its indentation is `<=` the indentation of the second line, combine the first and second lines if they fit. Apply recursively.

**Google note:** Google additionally limits long/complex iterator combinator chains (readability, not formatting) — see [G-IMPL-06](../google/13-implementation-concerns.md#g-impl-06).

**Example:**
```rust
let foo = bar
    .baz?
    .qux();
```

```rust
x.baz?
    .qux()

x.y.z
    .qux()

let foo = x
    .baz?
    .qux();

foo(
    expr1,
    expr2,
).baz?
    .qux();
```

<a id="rs-expr-21"></a>

## RS-EXPR-21 — Multi-line chain elements

**Rule:** If any chain element is formatted across multiple lines, put that element and all later elements on their own lines. **SHOULD** prefer the whole chain in multi-line style with each element on one line over mixing single-line and multi-line elements.

**Details:**
- In the first example, there is block indent from both the chain and the function call.

**Example:**
```rust
a.b.c()?
    .foo(
        an_expr,
        another_expr,
    )
    .bar
    .baz
```

```rust
// Better
self.pre_comment
    .as_ref()
    .map_or(false, |comment| comment.starts_with("//"))

// Worse
self.pre_comment.as_ref().map_or(
    false,
    |comment| comment.starts_with("//"),
)
```

<a id="rs-expr-22"></a>

## RS-EXPR-22 — Control flow expressions

**Rule:** Applies to `for`, `loop`, `if` and `while` expressions, including `if let`/`while let` and let-chains (one or more `let` sub-expressions plus one or more bool conditions, e.g. `if a && let Some(b) = c`). Put the keyword, initial clauses and opening brace on one line if they fit; format the block per [block rules](#rs-expr-01).

**Details:**
- `else`: put the closing brace, `else`, any following clause and the opening brace on the same line, with one space before and after `else`.
- Breaking the control line: prefer breaking after the `=` of any `let` sub-expression in `if`/`while` that doesn't fit, and before `in` in `for`; block-indent the following line.
- If the control line is broken for any reason, put the opening brace on its own line, not indented.
- Let-chains: a let-chain control line **MAY** be on one line only if it has exactly two clauses, the left operand is a literal or an `ident` (optionally preceded by any number of unary prefix operators), and the right operand is a single-line `let` clause. Otherwise the control line **MUST** be broken per the rules above.
- If the initial clause spans multiple lines, ends with one or more closing parentheses/brackets/braces with nothing else on that line, and that line is not indented beyond the first line of the expression, put the opening brace on that line after a space.

**Example:**
```rust
if ... {
    ...
} else {
    ...
}

if let ... {
    ...
} else if ... {
    ...
} else {
    ...
}
```

```rust
while let Some(foo)
    = a_long_expression
{
    ...
}

for foo
    in a_long_expression
{
    ...
}

if a_long_expression
    && another_long_expression
    || a_third_long_expression
{
    ...
}

if let Some(a) = b
    && another_long_expression
    && a_third_long_expression
{
    // ...
}

if let Some(relatively_long_thing)
    = a_long_expression
    && another_long_expression
    && a_third_long_expression
{
    // ...
}

if some_expr
    && another_long_expression
    && let Some(relatively_long_thing) =
        a_long_long_long_long_long_long_really_reallllllllllyyyyyyy_long_expression
    && a_third_long_expression
{
    // ...
}
```

```rust
// Single-line let-chains allowed
if a && let Some(b) = foo() {
    // ...
}

if true && let Some(b) = foo() {
    // ...
}

let operator = if !from_hir_call && let Some(p) = parent {
    // ...
};

// Must be broken
if let Some(b) = foo()
    && a
{
    // ..
}

if foo()
    && let Some(b) = bar
{
    // ...
}

if gen_pos != GenericArgPosition::Type
    && let Some(b) = gen_args.bindings.first()
{
    // ..
}
```

```rust
if !self.config.file_lines().intersects(
    &self.codemap.lookup_line_range(
        stmt.span,
    ),
) {  // Opening brace on same line as initial clause.
    ...
}
```

**See also:** [G-IMPL-06](../google/13-implementation-concerns.md#g-impl-06) (no combinator chains in `for` headers)

<a id="rs-expr-23"></a>

## RS-EXPR-23 — Single line `if else`

**Rule:** Put an `if else` or `if let else` on a single line if it is in expression context (not a standalone statement), has a single `else` clause, and is *small*.

**Example:**
```rust
let y = if x { 0 } else { 1 };

// Examples that must be multi-line.
let y = if something_very_long {
    not_small
} else {
    also_not_small
};

if x {
    0
} else {
    1
}
```

<a id="rs-expr-24"></a>

## RS-EXPR-24 — Match

**Rule:** **SHOULD NOT** break inside the discriminant expression. **MUST** always break after `{` and before `}`, and block-indent the arms once. Use a trailing comma on an arm if and only if it does not use a block. **MUST NOT** start an arm pattern with `|`.

**Details:**
- **SHOULD** avoid splitting the left-hand side (before `=>`).
- If the right-hand side stays on the same line as the left, **MUST NOT** use a block (unless the block is empty).
- Use a block if the right-hand side has multiple statements, has line comments, or its start doesn't fit on the left-hand side's line.
- **MUST NOT** flatten a right-hand-side block containing a single macro call: its expansion could contain a trailing semicolon.
- Block-indent the body of a block arm.
- If the body is a single expression with no line comments and is not a control-flow expression, start it on the same line as the left-hand side; otherwise it **MUST** be in a block. (Combinable bodies follow [RS-EXPR-26](#rs-expr-26).)

**Example:**
```rust
match foo {
    // arms
}

let x = match foo.bar.baz() {
    // arms
};
```

```rust
// Bad: leading `|`
match foo {
    // Don't do this.
    | foo => bar,
    // Or this.
    | a_very_long_pattern
    | another_pattern
    | yet_another_pattern
    | a_fourth_pattern => {
        ...
    }
}

// Good
match foo {
    foo => bar,
    a_very_long_pattern
    | another_pattern
    | yet_another_pattern
    | a_fourth_pattern => {
        ...
    }
}
```

```rust
match foo {
    foo => bar,
    a_very_long_pattern | another_pattern if an_expression() => {
        no_room_for_this_expression()
    }
    foo => {
        // A comment.
        an_expression()
    }
    foo => {
        let a = statement();
        an_expression()
    }
    bar => {}
    // Trailing comma on last item.
    foo => bar,
    baz => qux!(),
    lorem => {
        ipsum!()
    }
}
```

```rust
match foo {
    // A combinable expression.
    foo => a_function_call(another_call(
        argument1,
        argument2,
    )),
    // A non-combinable expression
    bar => {
        a_function_call(
            another_call(
                argument1,
                argument2,
            ),
            another_argument,
        )
    }
}
```

<a id="rs-expr-25"></a>

## RS-EXPR-25 — Match arm line-breaking

**Rule:** If using a block body on the right-hand side avoids breaking the left-hand side, **SHOULD** do that. **MUST NOT** break after `=>` without using the block form of the body.

**Details:**
- Left-hand side must be split and has an `if` guard: break before `if` and block-indent it; always use a block body starting on a new line (`{` on its own line).
- Pattern must be broken: put each clause on its own line with no extra indent, breaking before `|`; with an `if` guard use the form above.
- If the pattern is multi-line and its last line is narrower than the indent, do not put the `if` guard on a new line.
- If every clause is *small* but the whole pattern doesn't fit on one line, format it across lines with as many clauses per line as possible, again breaking before `|`.
- A pattern clause is *small* if it fits on one line and matches `small` in the grammar below (e.g. `&&Some(foo)` matches; `Foo(4, Bar)` does not).

```text
small:
    - small_no_tuple
    - unary tuple constructor: `(` small_no_tuple `,` `)`
    - `&` small

small_no_tuple:
    - single token
    - `&` small_no_tuple
```

**Example:**
```rust
    // Assuming the following line does not fit in the max width
    a_very_long_pattern | another_pattern => ALongStructName {
        ...
    },
    // Prefer this
    a_very_long_pattern | another_pattern => {
        ALongStructName {
            ...
        }
    }
    // To splitting the pattern.
```

```rust
    a_very_long_pattern | another_pattern
        if expr =>
    {
        ...
    }
```

```rust
    a_very_long_pattern
    | another_pattern
    | yet_another_pattern
    | a_forth_pattern => {
        ...
    }
    a_very_long_pattern
    | another_pattern
    | yet_another_pattern
    | a_forth_pattern
        if expr =>
    {
        ...
    }
```

```rust
    Token::Dimension {
         value,
         ref unit,
         ..
    } if num_context.is_ok(context.parsing_mode, value) => {
        ...
    }
```

```rust
    foo | bar | baz
    | qux => {
        ...
    }
```

<a id="rs-expr-26"></a>

## RS-EXPR-26 — Combinable expressions

**Rule:** When a function call has a single argument formatted across multiple lines, format the outer call as if single-line if the result fits. Apply the same combining to similar constructs with multi-line, block-indented, parenthesis-delimited sub-expression lists (e.g. macros, tuple struct literals). Apply recursively.

**Details:**
- Multiple arguments: also combine if the last argument is a multi-line closure with an explicit block, there are no other closure arguments, and all arguments plus the closure's first line fit on the first line.

**Example:**
```rust
foo(bar(
    an_expr,
    another_expr,
))

let x = foo(Bar {
    field: whatever,
});

foo(|param| {
    action();
    foo(param)
})

let x = combinable([
    an_expr,
    another_expr,
]);

let arr = [combinable(
    an_expr,
    another_expr,
)];
```

```rust
foo(first_arg, x, |param| {
    action();
    foo(param)
})
```

<a id="rs-expr-27"></a>

## RS-EXPR-27 — Ranges

**Rule:** **MUST NOT** put spaces in ranges (`0..10`, `x..=y`, `..x.len()`, `foo..`). If either bound is a compound expression, parenthesize it to show precedence (`..(x + 1)`, `(x.f)..(x.f.len())`, `0..(x - 10)`).

**Details:**
- With both bounds, if the line must break within the range, break before the range operator and block-indent the second line.

**Example:**
```rust
a_long_expression
    ..another_long_expression
```

<a id="rs-expr-28"></a>

## RS-EXPR-28 — Hexadecimal literals

**Rule:** Hex literals **MAY** use upper- or lower-case letters but **MUST NOT** mix cases within one literal. Projects **SHOULD** use the same case for all literals; the style guide recommends neither case.

**Google note:** Google's rustfmt configuration sets `hex_literal_case = Lower` (a codebase may skip it if the option is unavailable on its stable Rust); see [G-FMT-01](../google/03-formatting.md#g-fmt-01).

**Example:**
```rust
// Bad: mixed case within one literal
let x = 0xAbCd;

// Good (pick one case project-wide)
let x = 0xabcd;
let y = 0xABCD;
```

**See also:** [`hex_literal_case`](../references/rustfmt-config.md#hex_literal_case)

<a id="rs-expr-29"></a>

## RS-EXPR-29 — Patterns

**Rule:** Format patterns like their corresponding expressions. See [Match](#rs-expr-24) and [match arm line-breaking](#rs-expr-25) for match-arm pattern formatting.
