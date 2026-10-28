# Testing

> **Source:** [Google Rust Style Guide — Use the right per-codebase test framework](https://google.github.io/styleguide/rustguide.html#test-framework) · **License:** CC-BY 3.0
>
> Which test framework to use in each Google codebase.

<a id="g-test-01"></a>

## G-TEST-01 — Use the right per-codebase test framework

**Rule:** **MUST** use your codebase's specific test framework; most codebases use their own rather than plain `#[test]`.

**Details:**
- `[chrome]` Chrome: use [rust_gtest_interop](https://source.chromium.org/chromium/chromium/src/+/main:testing/rust_gtest_interop/), which integrates Rust tests into C++ test binaries.
- `[Fuchsia]` Fuchsia: use `#[fuchsia::test]`; see [Fuchsia Rust testing](https://fuchsia.dev/fuchsia-src/development/languages/rust/testing.md) for details.

**See also:** [Platform notes](../references/platform-notes.md)
