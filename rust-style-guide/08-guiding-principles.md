# Guiding Principles and Rationale

> **Source:** [Rust Style Guide — Guiding principles and rationale](https://doc.rust-lang.org/nightly/style-guide/principles.html) · **License:** MIT OR Apache-2.0
>
> The principles the Rust style team uses when deciding style guidelines.

When deciding on style guidelines, the style team follows these principles in rough priority order: readability (RS-PRIN-01), aesthetics (RS-PRIN-02), specifics (RS-PRIN-03), application (RS-PRIN-04).

<a id="rs-prin-01"></a>

## RS-PRIN-01 — Readability

**Rule:** Readability is the highest-priority principle.

**Details:**
- Scan-ability.
- Avoiding misleading formatting.
- Accessibility: readable and editable by users on the widest variety of hardware, including non-visual accessibility interfaces.
- Readability of code without syntax highlighting or IDE assistance, such as rustc error messages, diffs, grep, and other plain-text contexts.

**Google note:** Google follows the public Rust style rules to the extent they don't conflict with the Google guide, which adds its own goals (e.g. smoothing large-scale changes); see [G-BG-01](../google/01-background-and-scope.md#g-bg-01).

<a id="rs-prin-02"></a>

## RS-PRIN-02 — Aesthetics

**Rule:** Aesthetics rank second, after readability.

**Details:**
- Sense of "beauty".
- Consistency with other languages/tools.

<a id="rs-prin-03"></a>

## RS-PRIN-03 — Specifics

**Rule:** Specifics rank third.

**Details:**
- Compatibility with version control practices: preserving diffs, merge-friendliness, etc.
- Preventing rightward drift.
- Minimising vertical space.

<a id="rs-prin-04"></a>

## RS-PRIN-04 — Application

**Rule:** Application concerns rank fourth.

**Details:**
- Ease of manual application.
- Ease of implementation (in `rustfmt`, and in other tools/editors/code generators).
- Internal consistency.
- Simplicity of formatting rules.
