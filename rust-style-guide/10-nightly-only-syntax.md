# Nightly-only Syntax

> **Source:** [Rust Style Guide — Nightly](https://doc.rust-lang.org/nightly/style-guide/nightly.html) · **License:** MIT OR Apache-2.0
>
> Style and formatting for nightly-only (feature-gated) syntax.

This chapter is the only place nightly syntax appears; the rest of the guide covers stable syntax. Each section names its feature gate so searches (e.g. `git grep`) for a nightly feature in the Rust repository also find the style section. Nightly formatting **SHOULD** be moved into the appropriate stable sections of the guide when the feature stabilizes. Unlike the rest of the guide, this chapter has no stability guarantee; breaking changes follow the style team policy for nightly formatting.

<a id="rs-night-01"></a>

## RS-NIGHT-01 — Frontmatter

**Rule:** **SHOULD NOT** put blank lines between the frontmatter and either the start of the file or a shebang; there **MAY** be zero or one line between the frontmatter and any following content. Fences **SHOULD** use the minimum number of dashes needed.

**Applies to:** nightly only; feature gate `frontmatter`; tracking issue [#136889](https://github.com/rust-lang/rust/issues/136889). Location: placed before comments and attributes in the [root](01-formatting-conventions.md).

**Details:**
- Fence length: one more than the longest series of initial dashes in the content, with a minimum of 3 (needed to be recognized as frontmatter delimiters).
- If an infostring follows the opening fence, separate them with one space.
- Fence lines **SHOULD NOT** have trailing whitespace.

**Google note:** Google forbids unstable features (with codebase-specific exceptions), see [G-FEAT-02](../google/12-other-rust-features.md#g-feat-02).

**Example:**
```rust
#!/usr/bin/env cargo
--- cargo
[dependencies]
regex = "1"
---

fn main() {}
```
