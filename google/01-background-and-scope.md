# Background and Scope

> **Source:** [Google Rust Style Guide — Introduction, Background, Follow public Rust style rules, Apply API design rules in context](https://google.github.io/styleguide/rustguide.html#background) · **License:** CC-BY 3.0
>
> What the Google Rust Style Guide is, where it applies, why its rules exist, and how it relates to the public Rust style rules and to library vs application code.

## What this guide is

This is a *style guide* for using Rust at Google. "Style" covers more than formatting: it includes guidance on API design and implementation design.

Rust is used in [google3](https://opensource.google/documentation/reference/glossary#google3), Android, ChromeOS, Fuchsia and Chrome. The guide aims for Rust usage at Google to be consistent both within and across these codebases, and to establish a standard of quality.

## Scope

Current scope of authority of the rules in this guide:

- **In Android, Chromium, and other codebases:** the guide was developed in consultation with these codebases, and they intend to start applying it. Each codebase may have its own rules about how strictly it is applied. More information is expected as these codebases adopt the guide.

Rules tagged `[chrome]`, `[android]` or `[Fuchsia]` apply only to that codebase (see [platform notes](../references/platform-notes.md)).

## Goals of the style guide

By and large the guide exists for [the same reasons as Google's C++ style guide](https://google.github.io/styleguide/cppguide.html#Goals) (see also [C++ style guide excerpts](../references/cpp-style-guide-excerpts.md)). Most style rules derive from one or more of these goals:

1. **Prioritize readability and maintainability over writeability.** Code lives for a long time; a little extra effort while writing is worth it if the rule improves readability.
2. **Favor better automation.** Large-Scale Changes (LSCs) are common at Google, and consistency makes LSCs easier to carry out.
3. **Be consistent with the wider Rust ecosystem where possible.** The ecosystem is already fairly consistent in style; relying on it reduces surprise and makes Google's Rust style less work to learn.
4. **Pull their weight.** Every Rust developer and reviewer at Google must keep these rules in mind, so only rules with a tangible benefit are included.
5. **May help avoid minor style disagreements.** Where reasonable people disagree, a rule avoids rehashing the debate on every CL. Rules whose only benefit is this should be kept to a minimum.

<a id="g-bg-01"></a>

## G-BG-01 — Follow public Rust style rules

**Rule:** Code **MUST** follow the official public Rust style rules to the extent they don't conflict with this style guide.

**Details:**
- Public rules to follow:
  - [Rust Style Guide](https://doc.rust-lang.org/nightly/style-guide/) (condensed in [`../rust-style-guide/`](../rust-style-guide/01-formatting-conventions.md)). Most of its formatting conventions are enforced automatically by rustfmt (see [G-FMT-01](03-formatting.md#g-fmt-01)).
  - [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) (condensed in [`../api-guidelines/`](../api-guidelines/01-naming.md)).
- A "conflict" is a case where this guide suggests a pattern that the Rust style guide discourages, or that runs counter to the Rust style guide's primary encouraged pattern. In a conflict, this guide wins.

**Rationale:**
- Most style guides have three goals: consistency, readability, and encouraging good API design. Whatever the specific choices, deciding them up front limits bikeshedding.
- This guide shares those goals but adds one less common goal: smoothing the way for Large-Scale Changes (LSCs). Large codebases like many at Google occasionally run codebase-spanning, tool-assisted refactorings; for this to work well, code must be easy for tooling to work with. Consistency helps, and so do deliberate refactor-friendly patterns. This is why the guide sometimes deviates from regular Rust style rules.

**See also:** Rules where Google overrides public style: [G-ORG-05](02-code-organization.md#g-org-05), [G-ORG-06](02-code-organization.md#g-org-06), [G-FMT-01](03-formatting.md#g-fmt-01), [G-FEAT-03](12-other-rust-features.md#g-feat-03), [G-FEAT-08](12-other-rust-features.md#g-feat-08), [G-FEAT-09](12-other-rust-features.md#g-feat-09), [G-NAM-01](10-naming.md#g-nam-01)

<a id="g-bg-02"></a>

## G-BG-02 — Apply API design rules in context

**Rule:** When applying API design rules, distinguish library code from application code. For library code, API design rules are binding (**MUST**); for application code, they are a strong recommendation (**SHOULD**).

**Details:**
- **Library code:** intended for use by other projects (applications or other libraries); solves a relatively general problem.
- **Application code:** intended for use in a single application; solves a more specific problem.
- When applying the API design rules to application code, remember that it solves a specific problem and should generally favor simplicity and directness over broad reusability.
- Application code **SHOULD NOT** generalize or abstract prematurely; focus on what is required to meet the immediate need.

**See also:** [Rust API Guidelines](../api-guidelines/01-naming.md), [G-FIO](05-function-inputs-and-outputs.md), [G-TRT](07-traits.md)
