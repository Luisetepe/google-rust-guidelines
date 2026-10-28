# Macros

> **Source:** [Google Rust Style Guide — Macros](https://google.github.io/styleguide/rustguide.html#macros) · **License:** CC-BY 3.0
>
> Preference order for API mechanisms (plain Rust before macros and code generation) and the costs to weigh before using macros.

<a id="g-mac-01"></a>

## G-MAC-01 — Prefer non-macro facilities, in order

**Rule:** When considering how to express an API, you **SHOULD** prefer, in order:
1. Regular Rust functions, types, traits, objects, and modules (avoid macros, code generation, and metaprogramming where possible).
2. `macro_rules!`-style macros.
3. Other code generation strategies (e.g. `genrule`).
4. `proc_macro` macros.

**Rationale:**
- **Prefer non-macro facilities:** macros can create dialects of the language that make it harder for engineers to work across the codebase. A reader seeing a macro usage can make fewer assumptions about that code than without a macro. Macros are harder to write correctly than regular Rust code.
- **Use `macro_rules!` for simple code generation:** since Rust macros operate on the AST, they can be less fragile than textual code generation.

**See also:** [G-MAC-02](#g-mac-02)

<a id="g-mac-02"></a>

## G-MAC-02 — Weigh the costs of macros and code generation

**Rule:** When you decide to use a macro or code generation, you **SHOULD** weigh the benefits against the costs.

**Details:**
- Costs to weigh:
  - IDEs, Code Search and other developer tools work less well on code that uses macros.
  - Macros can create dialects of the language that make it harder for engineers to work across the codebase.
  - Large scale changes are more difficult and expensive to apply.

**See also:** [G-MAC-01](#g-mac-01)
