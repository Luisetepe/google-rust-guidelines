# AGENTS.md

This repository is a rulebook for writing and reviewing Rust, not a codebase. It condenses Google's Rust Style Guide plus the public guides it builds on into 240 rules with stable IDs. Use it when you write, refactor, or review Rust code in any project.

## Quick start (every Rust task)

1. **Establish context** from the target project (its docs, `Cargo.toml`, build files, or ask the user):
   - **Library or application?** API-design rules are **binding** for library code and a **strong recommendation** for application code, which should favor simplicity over premature generalization ([G-BG-02](google/01-background-and-scope.md#g-bg-02)).
   - **Codebase:** Chromium, Fuchsia, Android, or other. Rules whose **Applies to** names a codebase (`[chrome]`, `[Fuchsia]`, `[android]`) apply only there; skip them elsewhere. Untagged rules apply everywhere.
2. **Load [`google/README.md`](google/README.md).** It lists every Google rule as one normative line. Keep it in context for the whole task.
3. **Open the chapters your task touches** (see [Task routing](#task-routing)) and read the full rule before acting on it. Index lines omit exceptions, conditions, and examples; "listed below" in an index line refers to the rule's own section.
4. **Format with rustfmt, never by hand.** Use the settings in [Google-aligned rustfmt.toml](references/rustfmt-config.md#google-aligned-rustfmttoml) and run `cargo fmt` (unstable options need `cargo +nightly fmt`). Read [`rust-style-guide/`](rust-style-guide/README.md) only for what rustfmt does not decide: comments, naming, `Cargo.toml`, macro bodies, and `#[rustfmt::skip]` regions ([G-FMT-02](google/03-formatting.md#g-fmt-02)).
5. **Public library API?** Also walk the [API Guidelines checklist](api-guidelines/README.md).

## Precedence

1. The target project's own documented conventions win over this repo; follow them and mention any conflict to the user.
2. Google rules (`G-*`) override the public guides.
3. Rust Style Guide (`RS-*`) and API Guidelines (`C-*`) apply where Google is silent. A **Google note** on one of these rules tells you how Google changes it; follow the note.
4. [`references/`](references/README.md) is context for the rules that cite it. Its `CR-*` (Comprehensive Rust) and `CPP-*` (C++ guide) entries support Google rules ([G-NAM-01](google/10-naming.md#g-nam-01), [G-NAM-06](google/10-naming.md#g-nam-06), the Goals in [background](google/01-background-and-scope.md)); they do not add rules of their own.

## Strength keywords

| Keyword | Meaning for you |
| --- | --- |
| **MUST** / **MUST NOT** | Required. Deviate only under one of the rule's listed **Exceptions**. |
| **SHOULD** / **SHOULD NOT** | Default. Deviate only for a concrete reason, and state it (in your reply, PR description, or review). |
| **MAY** | Optional. |
| no keyword | Mostly `RS-*` formatting rules: they describe the default style rustfmt produces. Treat them as required for code rustfmt formats. |

## Task routing

| Task | Read |
| --- | --- |
| Any new Rust code | [`google/README.md`](google/README.md), [rustfmt settings](references/rustfmt-config.md#google-aligned-rustfmttoml) |
| Crates, modules, files, imports | [G-ORG](google/02-code-organization.md); RS-ITEM-18..24 in [items](rust-style-guide/02-items.md) |
| Ownership, cloning, `Rc`/`Arc` | [G-OWN](google/04-ownership-and-borrowing.md); C-CALLER-CONTROL in [flexibility](api-guidelines/06-flexibility.md) |
| Function parameters and return types | [G-FIO](google/05-function-inputs-and-outputs.md); C-GENERIC, C-INTERMEDIATE in [flexibility](api-guidelines/06-flexibility.md); C-CUSTOM-TYPE in [type safety](api-guidelines/07-type-safety.md) |
| Errors, panics, `Option`/`Result` | G-FIO-06; G-IMPL-02, G-IMPL-03, G-IMPL-07 in [implementation](google/13-implementation-concerns.md); G-DOC-03; C-GOOD-ERR, C-FAILURE, C-VALIDATE, C-DTOR-FAIL |
| Structs, enums, newtypes, aliases | [G-TYP](google/06-types.md); G-FEAT-08, G-FEAT-09; G-IMPL-04; C-NEWTYPE, C-BUILDER, C-STRUCT-PRIVATE, C-NEWTYPE-HIDE |
| Traits, impls, `derive`, generics vs `dyn` | [G-TRT](google/07-traits.md); [interoperability](api-guidelines/02-interoperability.md); C-OBJECT, C-SEALED, C-DEBUG, C-STRUCT-BOUNDS |
| Constructors and conversions | G-NAM-05, G-TRT-08, G-TRT-09; C-CTOR, C-CONV, C-CONV-SPECIFIC, C-CONV-TRAITS |
| Naming anything | [G-NAM](google/10-naming.md); RS-ADV-02 in [other advice](rust-style-guide/06-other-style-advice.md); [API naming](api-guidelines/01-naming.md); [Comprehensive Rust naming](references/comprehensive-rust-naming-conventions.md) |
| Getters, setters, accessors | G-FEAT-03 in [other features](google/12-other-rust-features.md); C-GETTER |
| Doc comments and comments | [G-DOC](google/11-documentation.md); RS-FMT-07, RS-FMT-08 in [formatting](rust-style-guide/01-formatting-conventions.md); [API docs](api-guidelines/04-documentation.md) |
| `unsafe` | G-FEAT-01, G-DOC-04 |
| Threads, locks, `async` | [G-CON](google/08-concurrency.md); C-SEND-SYNC |
| Macros | [G-MAC](google/09-macros.md); [API macros](api-guidelines/03-macros.md); RS-ITEM-12, RS-EXPR-18 |
| Iterators and collections | G-FIO-02, G-FIO-03, G-IMPL-06; C-ITER, C-ITER-TY, C-COLLECT |
| `#[must_use]`, `#[non_exhaustive]`, `const fn`, `#[doc(hidden)]`, operator overloading, `len`/`is_empty` | G-FEAT-04..12 in [other features](google/12-other-rust-features.md); C-OVERLOAD, C-HIDDEN |
| Assertions and tests | G-IMPL-05; [G-TEST](google/14-testing.md) |
| Dependencies, unstable features, `Cargo.toml` | G-FEAT-02; [Cargo.toml](rust-style-guide/07-cargo-toml.md); C-METADATA, C-STABLE, C-PERMISSIVE |
| Formatting question rustfmt can't settle | [`rust-style-guide/`](rust-style-guide/README.md) |

## Looking up a rule

Each rule's anchor is its lowercase ID (`G-ORG-05` → `#g-org-05`), placed on the line above its `## ID — Title` heading. A rule section runs until the next `<a id=` line and has these fields, in this order, when the source provides them: **Rule**, **Applies to**, **Details**, **Exceptions**, **Rationale**, **Google note**, **Example**, **See also**.

```sh
# Which file holds a rule?
grep -rl 'id="c-getter"' --include='*.md' .

# Print one full rule.
awk -v id=g-org-05 '$0 ~ "^<a id=\"" id "\"></a>" {f=1; next} f && /^<a id="/ {exit} f' */*.md
```

Without a local clone, fetch files from `https://raw.githubusercontent.com/Luisetepe/google-rust-guidelines/main/<path>`.

## Writing code

1. Before writing, do the quick start above. Model new code on the rules' **Good** examples.
2. After writing, run rustfmt, then check the change against every rule you loaded for the task.
3. Where Clippy is available, these lints check parts of specific rules (the guide cites them for context and does not require enabling them): `ptr_arg`, `borrowed_box` → G-FIO-01; `should_implement_trait` → G-TRT-02; `new_without_default` → G-TRT-07; `wrong_self_convention` → G-NAM-01; `len_without_is_empty` → G-FEAT-04; `exhaustive_enums` → G-FEAT-08; rustc's `missing_docs` → G-DOC-01. Details: [`references/clippy-lints.md`](references/clippy-lints.md).
4. Don't put rule IDs in code comments unless the project asks for it. Comments the rules themselves require (for example `SAFETY:` comments, G-FEAT-01) are still required.

## Reviewing code

- Flag only rules that apply in context: library vs application, codebase tags, the rule's own conditions and **Exceptions**.
- Cite each finding by ID and strength, then the fix: `G-IMPL-03 (SHOULD): unwrap() on the config lookup; use expect() stating the invariant.`
- Order findings MUST before SHOULD. For formatting, say "run rustfmt" instead of listing individual style rules.
- A SHOULD deviation with a stated, concrete reason is not a finding.

## Using this repo from another project

Add this repo as a submodule (`git submodule add https://github.com/Luisetepe/google-rust-guidelines docs/rust-guidelines`) or fetch it by raw URL. Then add a block like this to the project's own `AGENTS.md`, so agents know the context up front:

```markdown
## Rust guidelines

Follow https://github.com/Luisetepe/google-rust-guidelines (start with its AGENTS.md).
- Code type: application        <!-- or: library -->
- Codebase tags in effect: none <!-- or: chrome, Fuchsia, android -->
- Formatting: rustfmt with the repo's "Google-aligned rustfmt.toml"
```

## Maintaining this repo

- Chapter files (`<section>/NN-*.md`) are the source of truth. The section `README.md` indexes are generated: never edit them by hand.
- Every rule keeps the format above. IDs are stable because projects cite them: never renumber or reuse one; give a new rule the next free number in its chapter.
- Only add content that a cited source states. Keyword strength mirrors the source: "should" → **SHOULD**, "must"/"never"/"don't" → **MUST (NOT)**, a plain imperative → **MUST** unless the rule says "prefer"/"consider"/"avoid".
- After any edit, run:

```sh
python3 tools/gen_indexes.py           # regenerate section indexes
python3 tools/gen_indexes.py --check   # stale indexes, broken links/anchors, ID problems; exits 1 on failure
```
