# Dependability

> **Source:** [Rust API Guidelines — Dependability](https://rust-lang.github.io/api-guidelines/dependability.html) · **License:** MIT OR Apache-2.0
>
> Guidelines that make a crate unlikely to do the wrong thing: argument validation and well-behaved destructors.

<a id="c-validate"></a>

## C-VALIDATE — Functions validate their arguments

**Rule:** Rust code **SHOULD** enforce the validity of its input whenever practical, rather than following the [robustness principle](http://en.wikipedia.org/wiki/Robustness_principle) ("be conservative in what you send; be liberal in what you accept").

**Details:**
- Enforcement mechanisms, in order of preference:
  1. **Static enforcement:** choose an argument type that rules out bad inputs, e.g. `fn foo(a: Ascii)` over `fn foo(a: u8)`, where `Ascii` is a wrapper around `u8` guaranteeing the highest bit is zero (see newtypes, [C-NEWTYPE](07-type-safety.md#c-newtype)).
     - Usually little run-time cost: it pushes costs to the boundaries (e.g. when a `u8` is first converted into an `Ascii`), and catches bugs at compile time rather than through run-time failures.
     - Some properties are difficult or impossible to express using types.
  2. **Dynamic enforcement:** validate input as it is processed (or ahead of time, if necessary). Often easier to implement than static checking, but has downsides:
     - Runtime overhead (unless checking can be done as part of processing the input).
     - Delayed detection of bugs.
     - Introduces failure cases, via `panic!` or `Result`/`Option`, which client code must then handle.
  3. **Dynamic enforcement with `debug_assert!`:** as dynamic enforcement, but expensive checks can easily be turned off for production builds.
  4. **Dynamic enforcement with opt-out:** as dynamic enforcement, plus sibling functions that skip the checks.
     - Convention: mark opt-out functions with a suffix like `_unchecked`, or place them in a `raw` submodule.
     - Use unchecked functions judiciously, only when (1) performance dictates avoiding checks and (2) the client is otherwise confident the inputs are valid.

**Google note:** For the failure cases dynamic checks introduce, Google prescribes the mechanism: `Option` for simple failures with an obvious cause, `Result` for recoverable failures where the caller needs context, `panic!` for bugs and violated invariants (some codebases forbid panics entirely) ([G-FIO-06](../google/05-function-inputs-and-outputs.md#g-fio-06)); unavoidable panics should use `expect` ([G-IMPL-03](../google/13-implementation-concerns.md#g-impl-03)), and you should always consider adding context strings to assertions ([G-IMPL-05](../google/13-implementation-concerns.md#g-impl-05)).

**Example:**
```rust
// Good: static enforcement; `Ascii` guarantees the highest bit is zero
fn foo(a: Ascii) { /* ... */ }

// Bad: any u8 accepted
fn foo(a: u8) { /* ... */ }
```

**See also:** [C-FAILURE](04-documentation.md#c-failure), [newtype](../references/language-concepts.md#newtype)

<a id="c-dtor-fail"></a>

## C-DTOR-FAIL — Destructors never fail

**Rule:** Destructors **MUST NOT** fail. Instead, provide a separate method for checking clean teardown (e.g. `close`) that returns a `Result` to signal problems.

**Details:**
- If the `close` method is not called, the `Drop` implementation **SHOULD** perform the teardown and ignore or log/trace any errors it produces.

**Rationale:**
- Destructors run while panicking, and a failing destructor in that context aborts the program.

**Google note:** The `close` method's `Result` follows Google's error-reporting contract (recoverable failures the caller needs context for) ([G-FIO-06](../google/05-function-inputs-and-outputs.md#g-fio-06)).

**See also:** [C-DTOR-BLOCK](#c-dtor-block)

<a id="c-dtor-block"></a>

## C-DTOR-BLOCK — Destructors that may block have alternatives

**Rule:** Destructors **SHOULD NOT** invoke blocking operations; consider providing a separate method that prepares for an infallible, nonblocking teardown.

**Rationale:**
- Blocking in destructors can make debugging much more difficult.

**See also:** [C-DTOR-FAIL](#c-dtor-fail)
