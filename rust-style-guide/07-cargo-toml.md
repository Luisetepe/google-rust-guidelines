# Cargo.toml Conventions

> **Source:** [Rust Style Guide — `Cargo.toml` conventions](https://doc.rust-lang.org/nightly/style-guide/cargo.html) · **License:** MIT OR Apache-2.0
>
> Formatting and metadata conventions for `Cargo.toml` manifests.

<a id="rs-cargo-01"></a>

## RS-CARGO-01 — Formatting conventions

**Rule:** Use the same line width and indentation as Rust code, and lay out sections, keys, strings, arrays and tables as detailed below.

**Details:**
- Blank lines: put a blank line between the last key-value pair in a section and the header of the next section. **MUST NOT** place a blank line between a section header and its key-value pairs, or between key-value pairs in a section.
- Key order: version-sort key names within each section, except the `[package]` section.
- `[package]`: put it at the top of the file; put `name` then `version` at the top of the section, followed by the remaining keys other than `description` in order, followed by `description` at the end of the section.
- Key quoting: don't quote standard key names; use bare keys. Only use quoted keys for non-standard keys whose names require them, and avoid introducing such key names when possible (see the [TOML specification](https://toml.io/en/v1.0.0#keys)).
- Put a single space before and after the `=` between a key and value. Don't indent key names; start every key name at the start of a line.
- Multi-line string values (e.g. the crate description): use multi-line strings, not newline escape sequences.
- Array values (e.g. a list of features): put the entire list on the same line as the key if it fits. Otherwise block-indent: newline after the opening `[`, each item indented one level, a comma after each item (including the last), closing `]` at the start of its own line after the last item.
- Table values (e.g. a dependency with a path): write the whole table inline with curly braces and commas on the same line as the key if it fits; otherwise move it to a separate section with key-value pairs.

**Google note:** Google's rustfmt settings set Rust code width to `max_width = 100` ([G-FMT-01](../google/03-formatting.md#g-fmt-01)).

**Example:**
```toml
some_feature = [
    "another_feature",
    "yet_another_feature",
    "some_dependency?/some_feature",
]
```

```toml
[dependencies]
crate1 = { path = "crate1", version = "1.2.3" }

[dependencies.extremely_long_crate_name_goes_here]
path = "extremely_long_path_name_goes_right_here"
version = "4.5.6"
```

**See also:** [Sorting / version sorting](01-formatting-conventions.md)

<a id="rs-cargo-02"></a>

## RS-CARGO-02 — Metadata conventions

**Rule:** Metadata fields follow the conventions below for `authors`, `license`, `homepage` and `description`.

**Details:**
- `authors` (if present): **SHOULD** consist of strings each containing an author name followed by an email address in angle brackets: `Full Name <email@address>`. **SHOULD NOT** contain bare email addresses or names without email addresses. It **MAY** also include a mailing list address without an associated name.
- `license`: **MUST** contain a valid [SPDX expression](https://spdx.org/spdx-specification-21-web-version#h.jxpfx0ykyb60) using valid [SPDX license names](https://spdx.org/licenses/).
- `homepage` (if present): **MUST** be a single URL including the scheme (e.g. `https://example.org/`, not just `example.org`).
- `description`: wrap text at 80 columns. Don't start it with the crate name (e.g. "cratename is a ..."); just describe the crate. For a multi-sentence description, the first sentence **SHOULD** go on a line by itself and summarize the crate, like the subject of an email or commit message; subsequent sentences can then describe the crate in more detail.

**Exceptions:**
- By widespread convention, `license` **MAY** use `/` in place of ` OR `, e.g. `MIT/Apache-2.0`.

**Example:**
```toml
[package]
name = "cratename"
version = "1.2.3"
authors = ["Full Name <email@address>"]
homepage = "https://example.org/"
license = "MIT OR Apache-2.0"
description = """
Summarizes the crate in one sentence.
Further sentences describe the crate in more detail.
"""
```
