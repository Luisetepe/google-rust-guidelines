# Platform Notes — Fuchsia, Chromium, google3

> **Source:** Fuchsia, Chromium and Google Open Source pages linked from the [Google Rust Style Guide](https://google.github.io/styleguide/rustguide.html) (per-section sources below) · **License:** as stated per section
>
> Condensed procedures and rules from the codebase-specific pages the Google guide links for unstable features, test frameworks and the google3 codebase.

<a id="fuchsia-unstable"></a>

## Fuchsia — Unstable Rust features

> **Source:** [Fuchsia — Unstable Rust Features](https://fuchsia.dev/fuchsia-src/development/languages/rust/unstable) · **License:** CC-BY 4.0 (code samples Apache-2.0)

**Google guide context:** [G-FEAT-02 — No unstable features](../google/12-other-rust-features.md#g-feat-02) forbids unstable features, with a `[Fuchsia]` exception pointing to this page.

**Background:**
- `#![feature(...)]` enables unstable features (language, library, compiler and other capabilities without Rust's stability guarantees) on the `nightly` channel or a manually built `rustc` with the right flags. Most are temporarily unstable and stabilize after testing, discussion and design; some are intentionally permanently unstable (internal compiler use); some get removed when a better solution is found or the downsides outweigh the advantages. Each has a [tracking issue on `rust-lang/rust`](https://GitHub.com/rust-lang/rust/issues?q=is%3Aopen+is%3Aissue+label%3AC-tracking-issue).
- Fuchsia builds with a pinned revision of upstream Rust's master branch, mirrored in [fuchsia.googlesource.com/third_party/rust](https://fuchsia.googlesource.com/third_party/rust/). The version is set in the [`prebuilts` manifest](https://fuchsia.googlesource.com/integration/+/HEAD/prebuilts); the latest settable revision is the most recently completed [Rust CI build](https://ci.chromium.org/p/fuchsia/g/rust/console). Fuchsia updates Rust fairly often to pick up changes it upstreams (e.g. syscalls used by `std`).

**Goal:**
- Fuchsia wants to be able to roll forward or backward to other versions of Rust to pick up bugfixes or roll back problematic changes; depending on too many unstable features makes this extremely painful.
- Unstable or rapidly changing features make code harder to understand or modify, and are often poorly documented or documented out of date.
- Some features are designed explicitly for Fuchsia's use cases, give great readability or performance benefits, and Fuchsia's use helps move them toward stabilization.

**Approval process:**
- Send unstable feature requests to the [rust-users@fuchsia.dev](https://groups.google.com/a/fuchsia.dev/g/rust-users) mailing list, as a proposal hosted on docs.google.com (so it can be commented on).
- The proposal should include: a quick summary of the feature; what it is used for in Fuchsia; a summary of what remains before stabilization; and an owner who follows the tracking issue, participates in discussion on modifying/stabilizing the feature, and manages Fuchsia code updates caused by breaking changes to, or removal of, the feature.
- A week-long comment period follows the email; arguments for or against are laid out on the doc.
- Afterwards, a group of reviewers (the Rust Working Group) meets and reaches a consensus decision based on the arguments already on the doc; reviewers do not bring new arguments. If new arguments surface, they are added to the doc and more time is given for responses.
- If approved, the feature summary, usage, stabilization report and owner are added to the "Currently Used Features" documentation, which must be checked in before the feature can be used.
- Currently used unstable features are tracked by issues under the [`Rust>unstable`](https://issues.fuchsia.dev/issues?q=status:open%20componentid:1489345) component.

<a id="fuchsia-testing"></a>

## Fuchsia — Testing Rust code

> **Source:** [Fuchsia — Testing Rust code](https://fuchsia.dev/fuchsia-src/development/languages/rust/testing.md) · **License:** CC-BY 4.0 (code samples Apache-2.0)

**Google guide context:** [G-TEST-01 — Use the right per-codebase test framework](../google/14-testing.md#g-test-01) says `[Fuchsia]` code uses `#[fuchsia::test]` and points here.

**Scope:** Best practices for tests written inside `fuchsia.git`; the workflow is unlikely to work for IDK consumers. See also the [components testing guide](https://fuchsia.dev/fuchsia-src/development/testing/components/test_component); example source is in [`//examples/hello_world/rust`](https://cs.opensource.google/fuchsia/fuchsia/+/main:/examples/hello_world/rust).

**Writing unit tests:**
- Use the idiomatic Rust pattern: a `#[cfg(test)] mod tests` at the bottom of the file, included only when building unit tests. Functions annotated `#[fuchsia::test]` run as tests and pass if they return successfully.
- `#[fuchsia::test]` also supports async tests.
- Plain `#[test]` also works but doesn't support async tests or logging out of the box. Prefer `#[test]` where the crate might be used outside the Fuchsia codebase.

```rust
#[cfg(test)]
mod tests {
    #[fuchsia::test]
    fn it_works() {
        assert_eq!(true, true);
    }
}
```

```rust
#[fuchsia::test]
async fn my_test() {
    let some_future = async { 4 };
    assert_eq!(some_future.await, 4);
}
```

**Building tests:**
- Unit tests are built by the `rustc_binary` / `rustc_library` GN templates (imported from `//build/rust/rustc_binary.gni` / `//build/rust/rustc_library.gni`) only when `with_unit_tests = true` is set.
- For a binary target, this generates two executables: the provided `name` and one with `_bin_test` appended (e.g. `hello_world_rust` and `hello_world_rust_bin_test`), plus a `<target_name>_test` build target (e.g. `bin_test`).
- For a library target, the test binary is named differently, with `_lib_test` appended (e.g. `hello_world_rust_lib_test`). These binary names are used in later steps.
- By default `with_unit_tests = true` defines `<target_name>_test` in every toolchain where the `BUILD.gn` is evaluated. To restrict it, set `with_unit_tests` to a string (matching the Bazel Rust macros' interface): `"host"` (host toolchains only), `"fuchsia"` (Fuchsia toolchains only) or `"both"`.

```gn
import("//build/rust/rustc_binary.gni")

rustc_binary("bin") {
  name = "hello_world_rust"

  # Generates the "bin_test" build target
  with_unit_tests = true
  edition = "2024"

  deps = []
  test_deps = [ "//src/lib/fuchsia" ]

  sources = [ "src/main.rs" ]
}
```

```gn
rustc_library("lib") {
  name = "my_lib"
  edition = "2024"
  with_unit_tests = "host"
  sources = [ "src/lib.rs" ]
}

group("tests") {
  testonly = true
  deps = [ ":lib_test($host_toolchain)" ]
}
```

**Packaging and running tests:**
- Tests run as **test components**, whose [component manifest](https://fuchsia.dev/fuchsia-src/glossary#component-manifest) declares the capabilities they need; import `//build/components.gni`.
- `fuchsia_test_package()` collects multiple test components and their dependencies into one package (typically for integration tests).
- `fuchsia_unittest_package()` is specialized for unit tests and can generate a minimal component manifest for tests needing no capabilities.
- Run with `fx test <package-name>`. See [test components](https://fuchsia.dev/fuchsia-src/development/testing/components/test_component) and [building components](https://fuchsia.dev/fuchsia-src/development/components/build#test-packages).

```gn
fuchsia_unittest_package("hello-world-rust-tests") {
  deps = [ ":bin_test" ]
}
```

```sh
fx test hello-world-rust-tests
```

**Helpful in-tree crates** (add under `test_deps`):
- [`assert_matches`](https://fuchsia-docs.firebaseapp.com/rust/assert_matches/index.html): `assert_matches!` for ergonomic pattern assertions.
- [`pretty_assertions`](https://fuchsia-docs.firebaseapp.com/rust/pretty_assertions/index.html): an alternative `assert_eq!` showing a colored diff on failure.

```gn
rustc_binary("bin") {
  name = "my_test"
  with_unit_tests = true
  edition = "2024"

  test_deps = [
    "//third_party/rust_crates:matches",
    "//third_party/rust_crates:pretty_assertions",
  ]
}
```

<a id="chromium-rust-gtest-interop"></a>

## Chromium — `rust_gtest_interop`

> **Source:** [Chromium `testing/rust_gtest_interop/`](https://source.chromium.org/chromium/chromium/src/+/main:testing/rust_gtest_interop/) — content taken from the directory's [README.md](https://chromium.googlesource.com/chromium/src/+/main/testing/rust_gtest_interop/README.md), since source.chromium.org is a script-rendered code browser · **License:** Chromium's BSD-style license (repository `LICENSE`); the README states no license of its own

**Google guide context:** [G-TEST-01 — Use the right per-codebase test framework](../google/14-testing.md#g-test-01) says `[chrome]` code uses `rust_gtest_interop`, which integrates Rust tests into C++ test binaries.

**Overview:** Tools for writing gtest-based tests in Rust and integrating them into Chromium's C++ gtest binaries, all accessible through the `rust_gtest_interop` target, which is automatically included in test targets that depend on `//testing/gtest`.

**Adding Rust tests to a C++ gtest target:**
- Add the Rust file to the `test()` target's `sources`. Unlike other Rust crates, `crate_root` is not specified; it is generated from the sources list.
- Transitively depending on a `rust_static_library` includes its tests (as with C++ tests); that `rust_static_library` should explicitly depend on `//testing/rust_gtest_interop` and set `testonly` and `is_gtest_unittests`.

```gn
test("some_unittests") {
  sources = [
    "a_cpp_file.cc",
    "another_cpp_file.cc",
    "a_rust_file.rs",
  ]
  deps = [
    "//testing/gtest",
  ]
}
```

```gn
rust_static_library("my_rust_lib_unittests") {
  testonly = true
  is_gtest_unittests = true
  crate_root = "my_rust_lib_unittest.rs"
  sources = [ "my_rust_lib_unittest.rs" ]
  deps = [
    ":my_rust_lib",
    "//testing/rust_gtest_interop",
  ]
}

test("some_unittests") {
  ...
  deps += [ ":my_rust_lib_unittests" ]
}
```

**Writing a test:**
- Decorate a function with `#[gtest(TestSuiteName, TestName)]`, the equivalent of C++ `TEST()`.
- `#[gtest]` is exported from the `rust_gtest_interop::prelude` module; test files typically start with `use rust_gtest_interop::prelude::*;` (like `#include "testing/gtest/include/gtest/gtest.h"` in C++).

```rust
use rust_gtest_interop::prelude::*;  // Provides all the gtest macros.

#[gtest(MyTestSuite, MyTestOfThing)]
fn test() {
  ...
}
```

```cpp
#include "testing/gtest/include/gtest/gtest.h"  // Provides all the gtest macros.

TEST(MyTestSuite, MyTestOfThing) {
  ...
}
```

**Expectations:** Many familiar gtest EXPECT macros are available with Rust macro syntax:

```rust
expect_true!(is_friday());
expect_false!(is_saturday());

expect_eq!(2, 1 + 1);  // A == B
expect_ne!(3, 1 + 2);  // A != B

expect_lt!(1 * 1, 1 * 2);  // A < B
expect_gt!(4 * 1, 1 * 2);  // A > B
expect_le!(2 * 1, 1 * 2);  // A <= B
expect_ge!(3 * 1, 2 * 3);  // A >= B
```

**Returning a `Result`:**
- A test may return `()` or `std::result::Result<(), E>` for any `E` convertible to `std::error::Error` (i.e. `Into<std::error::Error>`), e.g. `std::io::Error` or `String`; this allows using the [`?` operator](https://doc.rust-lang.org/reference/expressions/operator-expr.html#the-question-mark-operator).
- If the test returns `Result::Err`, it fails and displays the error.

```rust
// Fails if file.txt can't be read or doesn't contain "hello world".
// Return type as written in the source; the std alias needs a type argument (std::io::Result<()>).
#[gtest(TestingIO, ReadFile)]
fn test() -> std::io::Result {
  let s = std::fs::read_to_string("file.txt")?;
  expect_eq!(s, "hello world");
  Ok(())
}
```

**Shared helper utilities:**
- Helpers shared across test files should go in a separate GN target, typically named with a `_test_support` suffix (e.g. `starship_test_support` for `starship_unittests`), usually in a `test/` subdirectory.
- E.g. the `starship_unittests` `test()` target includes unit test files such as `starship_unittest.rs`; the `starship_test_support` `rust_static_library()` includes the `test/` files.

```text
src/
  starship/
    starship_unittest.rs
    test/
      starship_test_helper.rs
      starship_test_things.rs
```

<a id="google3"></a>

## google3

> **Source:** [Google Open Source — Glossary: google3](https://opensource.google/documentation/reference/glossary#google3) · **License:** CC-BY 4.0 (code samples Apache-2.0)

**Google guide context:** The [Google guide's background](../google/01-background-and-scope.md) lists google3 among the codebases using Rust (with Android, ChromeOS, Fuchsia and Chrome).

**Definition:** google3 is the internal name for Google's main [Piper](https://opensource.google/documentation/reference/glossary#Piper) source repository (Piper being Google's internal source control system), and identifies the third incarnation of the source layout for Google production code.
