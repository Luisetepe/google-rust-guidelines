# Ownership and borrowing

> **Source:** [Google Rust Style Guide — Ownership and borrowing](https://google.github.io/styleguide/rustguide.html#ownership-borrowing) · **License:** CC-BY 3.0
>
> When to use shared-ownership smart pointers and when to clone versus borrow.

<a id="g-own-01"></a>

## G-OWN-01 — Avoid `Rc` and `Arc`

**Rule:** You **SHOULD** prefer unique ownership and borrowing over `Rc` and `Arc` for managing memory.

**Exceptions:**
- Use `Rc` and `Arc` when the alternative ownership and borrowing structure would be unreasonably complex to write, understand, or maintain. This is a judgment call to be discussed during code review.

**Rationale:**
- Rust generally allows representing object graphs without shared-ownership smart pointers.
- These pointers add runtime overhead and their own unique complexity.
- Engineers coming from other languages may tend to reach for `Rc` and `Arc` too quickly.

**See also:** [G-OWN-02](#g-own-02)

<a id="g-own-02"></a>

## G-OWN-02 — Prefer borrows over deep copies

**Rule:** Use `.clone()` when you genuinely need a deep copy of an object. You **SHOULD NOT** use `.clone()` as a substitute for borrowing; design code that works naturally with Rust's ownership and borrowing rules.

**Details:**
- If borrowing relationships become excessively complex, `.clone()` **MAY** be a pragmatic choice.
- Consider the performance needs of the code when choosing between cloning and borrowing.

**Exceptions:**
- This advice does not apply to cloning `Rc` or `Arc` pointers: cloning a reference-counted pointer does not create a deep copy and is an expected way to use them.

**Rationale:**
- Deep copies can be expensive due to memory allocation and data copying.

**See also:** [G-OWN-01](#g-own-01), [G-FIO-01](05-function-inputs-and-outputs.md#g-fio-01)
