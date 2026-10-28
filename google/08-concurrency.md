# Concurrency

> **Source:** [Google Rust Style Guide — Concurrency](https://google.github.io/styleguide/rustguide.html#concurrency) · **License:** CC-BY 3.0
>
> Mutex poisoning, plus codebase-specific restrictions on async, threads and mutex types.

<a id="g-con-01"></a>

## G-CON-01 — Handling poison

**Rule:** You **MUST NOT** reach inside a poisoned mutex to get at its insides, no matter how tempting the use case may seem.

<a id="g-con-02"></a>

## G-CON-02 — Don't use `async` / `.await` `[chrome]`

**Rule:** You **MUST NOT** use `async` / `.await` in Chromium.

**Applies to:** `[chrome]` Chromium only

<a id="g-con-03"></a>

## G-CON-03 — Don't start new threads through the Rust standard library `[chrome]`

**Rule:** You **SHOULD NOT** start new threads through the Rust standard library in Chromium; prefer sequences instead of threads.

**Applies to:** `[chrome]` Chromium only

<a id="g-con-04"></a>

## G-CON-04 — Use `fuchsia_sync::Mutex` `[Fuchsia]`

**Rule:** You **MUST** use Fuchsia's mutex type (`fuchsia_sync::Mutex`) rather than one from `std::sync` or `parking_lot`.

**Applies to:** `[Fuchsia]` Fuchsia only
