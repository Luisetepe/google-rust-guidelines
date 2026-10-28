# Implementation concerns

> **Source:** [Google Rust Style Guide — Implementation concerns](https://google.github.io/styleguide/rustguide.html#implementation-concerns) · **License:** CC-BY 3.0
>
> Rules for writing function bodies: type annotations, panics, enum matching, assertions, iterator chains, unwinding and string building.

<a id="g-impl-01"></a>

## G-IMPL-01 — Don't overuse type inference

**Rule:** You **MUST** add enough explicit types so that someone familiar with the codebase can figure out what the inferred types are.

**Details:**
- **SHOULD** put the types on variable declarations rather than in [turbofishes](../references/language-concepts.md#turbofish) ([Rust reference glossary](https://doc.rust-lang.org/reference/glossary.html#turbofish)).
- If the compiler forces a turbofish, a `_` inference marker inside it is generally fine.
- Situations where you **SHOULD** probably add types:
  - Calling methods with a generic return type, such as `.into()` or `.parse()`.
  - The result of complex iterator chains.

**Example:**
```rust
// Bad: the reader has to work out what `.parse()` produces
let port = text.parse().unwrap();

// Good: type on the declaration, not in a turbofish
let port: u16 = text.parse().unwrap();

// Fine when a turbofish is forced: collecting into a set to count distinct names has
// no declaration type to put it on (the expression is chained into `.len()`), so `_` lets
// the element type be inferred
let unique_names = people.iter().map(|p| &p.name).collect::<HashSet<_>>().len();

// Good: when the result is bound directly, put the type on the declaration
let names: Vec<_> = people.iter().map(|p| &p.name).collect();
```

**See also:** [turbofish](../references/language-concepts.md#turbofish)

<a id="g-impl-02"></a>

## G-IMPL-02 — Prefer pattern matching over check-then-unwrap

**Rule:** **SHOULD NOT** use potentially panicking operations when the input data guarantees they cannot panic; use pattern matching to prove correctness to the compiler instead (e.g. instead of `Option::unwrap` on a value known to be valid).

**Example:**
```rust
// Good: the match arm proves to the compiler that `option` is `Some`.
// A panic is obviously not possible.
if let Some(value) = option {
    use_value(value);
}

// Bad: correctness of `unwrap` relies on a runtime check that `option` is `Some`.
if option.is_some() {
    use_value(option.unwrap());
}
```

**See also:** [G-IMPL-03](#g-impl-03)

<a id="g-impl-03"></a>

## G-IMPL-03 — Prefer `expect` over `unwrap`

**Rule:** If a panic is necessary because an invariant cannot be expressed in the type system, **SHOULD** use `expect` rather than `unwrap` to document the reason.

**See also:** [G-IMPL-02](#g-impl-02)

<a id="g-impl-04"></a>

## G-IMPL-04 — Don't omit patterns on `non_exhaustive` enums when matching

**Rule:** When matching on a `#[non_exhaustive]` enum, **SHOULD** list all its variants in `match` statements.

**Details:**
- Variants you don't care about go in the wildcard arm (alongside `_`).
- If you *only* care about specific variants, use `if let` or `let … else {}` instead of `match`.

**Exceptions:**
- You only care about specific variants *and* `match` is cleaner than `if let`.

**Example:**
```rust
#[non_exhaustive]
pub enum Shape {
    Circle,
    Square,
    Triangle,
}
```

```rust
// Good: every known variant listed
fn process_shape(shape: Shape) {
  match shape {
    Shape::Circle => println!("It's a circle"),
    Shape::Square => println!("It's a square"),
    // Triangle and any future variants don't require any actions.
    Shape::Triangle | _ => {}
  }
}

// Good: only one variant matters, so use `if let`
fn process_shape(shape: Shape) {
  if let Shape::Circle = shape {
    println!("It's a circle");
  }
}

// Bad: known variants silently swallowed by `_`
fn process_shape(shape: Shape) {
  match shape {
    Shape::Circle => println!("It's a circle"),
    _ => {}
  }
}
```

**See also:** [G-FEAT-08](12-other-rust-features.md#g-feat-08)

<a id="g-impl-05"></a>

## G-IMPL-05 — Use context strings to clarify assertions

**Rule:** **SHOULD** always consider passing the optional format-string argument to `assert!` to explain why the assertion is expected to hold or what caused its violation.

**Details:**
- Add context particularly when:
  - You're thinking of adding a comment to the assertion: use a format string instead, because it appears in logs.
  - You expect failing assertions to be seen by users: a human-readable explanation helps more than a Rust expression.
  - Logging a value helps in understanding why the assertion failed.
  - You need to clarify whether a failure means the client misused the API (they should change their use of it) or the implementation has a bug (the client should file a bug report).
- **MUST NOT** use the context string merely to repeat the condition.

**Example:**
```rust
// Good: explains why the assertion should hold
assert!(
    values.is_sorted(),
    "`values` is populated from a `PriorityQueue`, so we expect it to be sorted"
);

// Good: logs a value to clarify why the assertion failed
assert!(refund > 0, "Refund must be positive, got {refund:?}");

// Good: flags misuse of an API
impl Database {
    ...

    fn execute_query(&self, query: &str) {
        assert!(self.stream.is_some(), "Call connect() before executing queries");
    }

    ...
}

// Bad: merely repeats the condition
assert!(values.is_sorted(), "values should always be sorted");
```

<a id="g-impl-06"></a>

## G-IMPL-06 — Don't use iterator combinator chains when direct iteration is clearer

**Rule:** **SHOULD NOT** use combinator chains in ways that hurt readability.

**Details:**
- **SHOULD NOT** write long and complex combinator chains (e.g. ones involving conditionals or mutating external state).
- **MUST NOT** construct a combinator chain in the header of a `for` loop; assign it to a variable (whose name documents what the chain does) and consume that variable in the loop.
- **SHOULD NOT** use `Iterator::for_each` unless you're convinced you have a good reason; a `for` loop is usually easier to read.

**Exceptions:**
- Combinators *are* preferable for shorter chains and where the combinator itself provides a lot of logic that would otherwise have to be written out in the `for` loop.

**Example:**
```rust
// Bad: complex chain with conditionals that mutates external state
let mut error_count = 0;
let _processed_items: Vec<_> = items
    .into_iter()
    .map(transformation_that_may_fail)
    .map(|result| {
        if result.is_err() {
            error_count += 1;
        }
        result
    })
    .filter_map(Result::ok)
    .collect();

// Good: the corresponding `for` loop is easier to read
let mut error_count = 0;
let mut processed_items = Vec::with_capacity(items.len());

for item in items {
    match transformation_that_may_fail(item) {
        Ok(val) => processed_items.push(val),
        Err(_) => error_count += 1,
    }
}
```

```rust
// Bad: combinator chain in the `for` loop header
for bug in bug_descriptions
    .iter()
    .filter_map(|description| parse_bug_description(description).ok())
    .filter(|bug| bug.priority <= Priority::P1)
{
    assign_bug(bug);
}

// Good: less clutter, and the variable name documents the chain
let urgent_bugs = bug_descriptions
    .iter()
    .filter_map(|description| parse_bug_description(description).ok())
    .filter(|bug| bug.priority <= Priority::P1);

for bug in urgent_bugs {
    assign_bug(bug);
}
```

```rust
// Good: a combinator chain is preferable here
let threshold_exceeded = values.iter().any(|&v| v > threshold);

// Bad: harder to read and reinvents the logic in `any()`
let mut threshold_exceeded = false;
for v in values {
  if v > threshold {
    threshold_exceeded = true;
    break;
  }
}
```

<a id="g-impl-07"></a>

## G-IMPL-07 — Don't call `panic::catch_unwind`

**Rule:** Use of `panic::catch_unwind` is prohibited (**MUST NOT**) in first-party Rust code.

**Exceptions:**
- **MAY** be used as a defense-in-depth mechanism as long as it is never *relied* upon for any properties: a codebase already attempting to avoid panics may wrap critical entry points with a `catch_unwind` recovery mechanism in case a panic slips through.
  - Code authors **MUST** convince the reviewer that the codebase is doing most of what it can to avoid panics.
  - Only applies to code that may run with `panic=unwind`; it does not apply to `[chrome]` Chrome, `[android]` Android or `[Fuchsia]` Fuchsia.

**Rationale:**
- Rust code may be used in many environments, and `panic::catch_unwind` only functions correctly with `panic=unwind`.
- The final panic disposition of prod binaries has not yet been determined, but Chrome, Android and Fuchsia all require `panic=abort`.

**See also:** [Platform notes](../references/platform-notes.md)

<a id="g-impl-08"></a>

## G-IMPL-08 — Prefer `format!` / `write!` for concatenation and substitution

**Rule:** **SHOULD** use `format!()` and `write!()` rather than alternatives when concatenating more than two strings or substituting into a literal string.

**Details:**
- Concatenating more than two strings: use `format!("{a}{b}{c}")` instead of `a + b + c`.
- Performing substitutions on a literal string: use `format!`/`write!` instead of `str::replace`.

**Rationale:**
- `format!` / `write!` require fewer allocations than concatenating strings with `+`.
- Prefer standard format strings to inventing your own templating syntax.

**Example:**
```rust
// Bad
let s = a + b + c;
let greeting = "Hello, NAME!".replace("NAME", name);

// Good
let s = format!("{a}{b}{c}");
let greeting = format!("Hello, {name}!");
```
