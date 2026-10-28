# Function inputs and outputs

> **Source:** [Google Rust Style Guide — Function inputs and outputs](https://google.github.io/styleguide/rustguide.html#inputs-and-outputs) · **License:** CC-BY 3.0
>
> Choosing parameter and return types (references, iterators, `Into`, enums) and the error-reporting mechanism for a function.

Several rules here distinguish highly reusable **library code** (favor flexible, performant APIs) from **application code** (usage patterns are easier to predict, so simpler APIs are often fine).

<a id="g-fio-01"></a>

## G-FIO-01 — Accept flexible references for immutable data

**Rule:** When a function accepts borrowed, immutable data, it **MUST** use the reference type that affords the most flexibility: pass references to the data itself, not references to smart pointers, containers, or wrappers.

**Details:**

| Use          | Instead of   | Why?                                                         |
| ------------ | ------------ | ------------------------------------------------------------ |
| `&T`         | `&Box<T>`    | Borrow the data inside the box                               |
| `&[T]`       | `&Vec<T>`    | Accept any slice instead of specific containers              |
| `&str`       | `&String`    | Accept a string slice instead of references to owned strings |
| `Option<&T>` | `&Option<T>` | Borrow the data inside the Option                            |

**Exceptions:**
- When expecting non-linear character data such as Rope/Cord.

**Rationale:**
- Reference types that don't mention specific smart pointers, containers, or wrappers make functions more versatile and performant: callers can pass data held in a variety of ways without copying it first.

**Example:**
```rust
// Bad: forces callers to hold a String / Vec / Box / Option.
fn f(name: &String, items: &Vec<u32>, cfg: &Box<Config>, extra: &Option<Extra>) { ... }

// Good
fn f(name: &str, items: &[u32], cfg: &Config, extra: Option<&Extra>) { ... }
```

**See also:** [`clippy::borrowed_box`](../references/clippy-lints.md#borrowed_box), [`clippy::ptr_arg`](../references/clippy-lints.md#ptr_arg), [G-OWN-02](04-ownership-and-borrowing.md#g-own-02), [C-GENERIC](../api-guidelines/06-flexibility.md#c-generic)

<a id="g-fio-02"></a>

## G-FIO-02 — Iterators vs. owned collections in return types

**Rule:** You **SHOULD** prefer returning iterators (either concrete types or `impl Iterator`) over returning `Vec<T>` where possible.

**Details:**
- In application code, returning a `Vec<T>` is fine if it simplifies the code and the flexibility and performance advantages of iterators are not clearly needed.

**Exceptions:**
- Most callers will need to construct an owned vector.
- The function's implementation naturally constructs a vector.
- The returned value will likely need to be randomly indexed.
- The algorithm works better as a `for` loop and the forced allocation doesn't matter too much.
- The function's intended purpose is creating a new owned collection.
- The function is returning a pre-existing vector.

**Rationale:**
- Highly reusable library code should generally strive to provide more flexible and performant APIs. Returning an iterator lets callers choose how to consume the data: collect it into a vector, process it in a loop, or chain further iterator adapters; some of these avoid unnecessary allocations.
- In application code, usage patterns are often easier to predict. If performance is clearly not a concern, or the data would be collected into a vector anyway, returning a vector makes for a simpler API.

**See also:** [G-FIO-03](#g-fio-03)

<a id="g-fio-03"></a>

## G-FIO-03 — Iterators vs. slices in function parameters

**Rule:** Highly reusable library code **SHOULD** accept iterators or `impl IntoIterator` if it only needs to iterate over the input.

**Applies to:** highly reusable library code

**Details:**
- In application code, accepting a slice `&[T]` is fine if it simplifies the code and the flexibility and performance advantages of iterators are not clearly needed.

**Rationale:**
- Highly reusable library code should generally strive to provide more flexible and performant APIs. Accepting iterators offers callers:
  - More flexibility (data can be supplied from any collection).
  - Efficiency (data does not need to be copied into a contiguous piece of memory).
  - Composability (the function works better with iterator chains).
- In application code, usage patterns are often easier to predict. If performance is clearly not a concern, or callers would always have a vector anyway, accepting a slice makes for a simpler API.

**See also:** [G-FIO-02](#g-fio-02), [C-GENERIC](../api-guidelines/06-flexibility.md#c-generic)

<a id="g-fio-04"></a>

## G-FIO-04 — Accepting `Into`

**Rule:** When designing library APIs that accept a `T` but users are likely to have various types, you **SHOULD** consider accepting `impl Into<T>` instead of `T` if **call-site ergonomics are paramount**.

**Applies to:** library APIs

**Details:**
- Balance this against code bloat concerns; it is not a requirement to use lots of `Into<T>` in your code.
- Application code should almost never use `impl Into<T>` parameters.

**Rationale:**
- `impl Into<T>` lets a function directly accept arguments of various compatible types, e.g. a function accepting `impl Into<String>` can be called with a `String` or a `&str` literal directly.
- The tradeoff is the convenience of avoiding `.into()` at the call site versus the cost of generics: more complex API signature, increased code size, sometimes even type inference difficulties.

**Example:**
```rust
fn set_name(name: impl Into<String>) { ... }

set_name("literal");           // &str
set_name(String::from("owned")); // String
```

**See also:** [G-TRT-08](07-traits.md#g-trt-08), [C-GENERIC](../api-guidelines/06-flexibility.md#c-generic)

<a id="g-fio-05"></a>

## G-FIO-05 — Prefer `enum` over `bool` in APIs

**Rule:** You **SHOULD** use `enum` function parameters instead of `bool` where the purpose of the parameter is not obvious.

**Details:**
- If you do use `bool`, consider using an inline comment to document the purpose of the `bool` in an argument list.

**Example:**
```rust
// Bad: what does `true` mean?
widget.redraw(true);

// Acceptable with bool: inline comment documents the argument
widget.redraw(/* force= */ true);

// Good
widget.redraw(Redraw::Force);
```

**See also:** [C-CUSTOM-TYPE](../api-guidelines/07-type-safety.md#c-custom-type)

<a id="g-fio-06"></a>

## G-FIO-06 — Reporting errors

**Rule:** You **MUST** choose the error reporting mechanism that fits your API contract: `Option`, `Result`, or `panic!`.

**Details:**
- **Use `Option` for simple failures** where the cause is obvious and the caller would not benefit from further details. Best for standard, anticipated outcomes like a missing value or a parsing mismatch.
- **Use `Result` for recoverable failures where the caller needs context** to handle the error. These are often "unavoidable" errors caused by external factors (like I/O or network) that even correctly implemented code cannot prevent with checks ahead of time. The caller can still reasonably anticipate the kinds of errors (a file-related function can fail with an I/O error); the error details let the caller distinguish failure modes and recover appropriately.
- **Use `panic!` to report bugs and violated invariants.** These errors are avoidable: the program is missing a check and calls an operation without fulfilling a precondition. Such defects must be fixed in the code, not handled at runtime, although coarse-grained recovery might be possible with `catch_unwind` (see [G-IMPL-07](13-implementation-concerns.md#g-impl-07)).
- Some codebases may choose to avoid panics entirely and require even broken invariants to be reported via `Result`.

**Exceptions:**
- Developer-facing binaries may use panics for non-fatal errors.
- Tests may use panics for assertions.

**Example:**
```rust
// Good: The cause of the failure is obvious (integer overflow). No further
// details could be reasonably provided.
fn checked_add(a: u32, b: u32) -> Option<u32> { ... }
```

```rust
// Good: The caller needs to know *why* it failed (NotFound vs PermissionDenied).
fn open_file(path: &Path) -> Result<File, io::Error> { ... }
```

```rust
// Good: Panics if the invariant (index must be in bounds) is violated.
// This indicates a bug in the caller's logic.
impl<T> MyVec<T> {
    pub fn remove(&mut self, index: usize) -> T {
        if index >= self.len() {
            panic!("index out of bounds");
        }
        // ...
    }
}
```

**See also:** [G-IMPL-03](13-implementation-concerns.md#g-impl-03), [G-IMPL-07](13-implementation-concerns.md#g-impl-07), [C-GOOD-ERR](../api-guidelines/02-interoperability.md#c-good-err), [C-FAILURE](../api-guidelines/04-documentation.md#c-failure)
