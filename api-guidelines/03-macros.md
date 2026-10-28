# Macros

> **Source:** [Rust API Guidelines — Macros](https://rust-lang.github.io/api-guidelines/macros.html) · **License:** MIT OR Apache-2.0
>
> How a crate presents well-behaved macros: evocative input syntax, attribute support, placement anywhere items are allowed, visibility specifiers, and flexible type fragments.

<a id="c-evocative"></a>

## C-EVOCATIVE — Input syntax is evocative of the output

**Rule:** Macro input syntax **SHOULD** stay familiar and cohesive with the rest of users' code by mirroring existing Rust syntax where possible, especially the keywords and punctuation that appear in the macro's output.

**Details:**
- Pay attention to the choice and placement of keywords and punctuation.
- If the macro declares a struct with a name given in the input, preface the name with the keyword `struct` to signal that a struct is being declared.
- Semicolons vs commas: Rust constants are followed by semicolons, so a macro declaring a chain of constants should likely use semicolons, even if its syntax otherwise differs slightly from Rust's.
- Macros are so diverse that these specific examples won't always be relevant; apply the same principles to your situation.

**Google note:** Google prefers, in order: regular functions, types, traits, objects and modules (avoiding macros, code generation and metaprogramming where possible), then `macro_rules!` macros, then other code generation strategies, then `proc_macro` macros ([G-MAC](../google/09-macros.md)).

**Example:**
```rust
// Good: `struct` keyword, as in the output
bitflags! {
    struct S: u32 { /* ... */ }
}

// Bad: no keyword
bitflags! {
    S: u32 { /* ... */ }
}

// Bad: ad-hoc word
bitflags! {
    flags S: u32 { /* ... */ }
}
```

```rust
// Ordinary constants use semicolons.
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// Good
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// Bad: commas instead of semicolons
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

<a id="c-macro-attr"></a>

## C-MACRO-ATTR — Item macros compose well with attributes

**Rule:** Macros that produce more than one output item **SHOULD** support adding attributes to any one of those items. Macros that produce a struct or enum **SHOULD** support attributes so the output can be used with derive.

**Details:**
- A common use case is putting individual items behind a `cfg`.

**Google note:** Google's macro policy (prefer non-macro facilities; weigh tooling, dialect and LSC costs) applies before writing such macros ([G-MAC](../google/09-macros.md)).

**Example:**
```rust
// Attributes on individual output items
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

```rust
// Attributes (derive) on the produced struct
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```

<a id="c-anywhere"></a>

## C-ANYWHERE — Item macros work anywhere that items are allowed

**Rule:** Item macros **SHOULD** work as well as ordinary items everywhere items are allowed (module level and tighter scopes like a function). The test suite **SHOULD** invoke the macro in at least module scope and function scope.

**Details:**
- The `broken!` example below works in module scope but fails in function scope: `super::U` refers to the containing module, not `g`.

**Google note:** Write these tests in the codebase's own test framework rather than plain `#[test]` where one exists ([G-TEST-01](../google/14-testing.md#g-test-01)).

**Example:**
```rust
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

```rust
// Bad: works in a module, fails in a function
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // okay, expands to T and m::T

fn g() {
    broken!(m::U); // fails to compile, super::U refers to the containing module not g
}
```

<a id="c-macro-vis"></a>

## C-MACRO-VIS — Item macros support visibility specifiers

**Rule:** Items produced by a macro **MUST** follow Rust visibility syntax: private by default, public if `pub` is specified.

**Example:**
```rust
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```

<a id="c-macro-ty"></a>

## C-MACRO-TY — Type fragments are flexible

**Rule:** A macro accepting a type fragment like `$t:ty` **SHOULD** work with all of: primitives (`u8`, `&str`), relative paths (`m::Data`), absolute paths (`::base::Data`), upward relative paths (`super::Data`) and generics (`Vec<String>`).

**Details:**
- The `broken!` example below works with primitives and absolute paths but fails with relative paths.

**Example:**
```rust
// Bad: relative paths break inside the generated module
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // okay

broken!(b => ::std::marker::PhantomData<()>); // okay

struct S;
broken!(c => S); // fails to compile
```
