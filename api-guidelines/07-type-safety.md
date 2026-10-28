# Type safety

> **Source:** [Rust API Guidelines — Type safety](https://rust-lang.github.io/api-guidelines/type-safety.html) · **License:** MIT OR Apache-2.0
>
> Guidelines for leveraging the type system: newtypes, meaningful argument types, bitflags, and builders.

<a id="c-newtype"></a>

## C-NEWTYPE — Newtypes provide static distinctions

**Rule:** You **SHOULD** use newtypes to statically distinguish between different interpretations of the same underlying type.

**Details:**
- E.g. an `f64` might represent miles or kilometers; `Miles(f64)` and `Kilometers(f64)` keep the intended interpretation, so `fn are_we_there_yet(distance_travelled: Miles)` cannot accidentally be called with a `Kilometers` value, and the compiler reminds you to convert.

**Rationale:**
- Averts certain [catastrophic bugs](http://en.wikipedia.org/wiki/Mars_Climate_Orbiter) by making confusion a compile error.

**Google note:** Google lists when to prefer a newtype over a type alias: notable semantic differences, types warranting their own API, flexibility to change the underlying type later, and type safety where types are easily mixed up (e.g. `Day`/`Month`/`Year` inside a `Date`) ([G-TYP-01](../google/06-types.md#g-typ-01)).

**Example:**
```rust
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn to_kilometers(self) -> Kilometers { /* ... */ }
}
impl Kilometers {
    fn to_miles(self) -> Miles { /* ... */ }
}

// Cannot accidentally be called with a `Kilometers` value.
fn are_we_there_yet(distance_travelled: Miles) -> bool { /* ... */ }
```

**See also:** [C-NEWTYPE-HIDE](10-future-proofing.md#c-newtype-hide), [C-VALIDATE](08-dependability.md#c-validate), [newtype](../references/language-concepts.md#newtype)

<a id="c-custom-type"></a>

## C-CUSTOM-TYPE — Arguments convey meaning through types, not `bool` or `Option`

**Rule:** Arguments **SHOULD** use a deliberate type (enum, struct, or tuple) that conveys interpretation and invariants, rather than core types like `bool`, `u8` or `Option` that have many possible interpretations.

**Details:**
- `Widget::new(true, false)` is unclear without looking up the argument names; `Widget::new(Small, Round)` is suggestive.
- For a no-cost way to wrap an existing type with a distinguished name, see the newtype pattern ([C-NEWTYPE](#c-newtype)).

**Rationale:**
- Clarity at the call site.
- Custom types make it easier to expand options later, e.g. adding an `ExtraLarge` variant.

**Google note:** Google requires `enum` parameters instead of `bool` where the parameter's purpose is not obvious, and suggests considering an inline comment to document any `bool` argument you do use ([G-FIO-05](../google/05-function-inputs-and-outputs.md#g-fio-05)); it prefers named fields over tuples and non-newtype tuple structs ([G-TYP-02](../google/06-types.md#g-typ-02)); newtype-vs-alias criteria are in [G-TYP-01](../google/06-types.md#g-typ-01).

**Example:**
```rust
// Good
let w = Widget::new(Small, Round)

// Bad
let w = Widget::new(true, false)
```

<a id="c-bitflag"></a>

## C-BITFLAG — Types for a set of flags are `bitflags`, not enums

**Rule:** When an API's input is the presence or absence of a set of flags, the type **SHOULD** be a [`bitflags`](https://github.com/bitflags/bitflags) type, not an `enum`.

**Details:**
- An `enum` requests exactly one choice among many. Enums with explicitly specified discriminants (e.g. `Red = 0xff0000`) are useful when the enum must serialize to an integer compatibly with another system/language, and support typesafe APIs: a function taking `Color` rather than an integer is guaranteed well-formed input, even if it later views it as an integer.
- C code typically represents flag sets as one bit per flag in a single integer (say 32 or 64 flags); the `bitflags` crate is a typesafe representation of this pattern.

**Example:**
```rust
// An enum with explicit discriminants: exactly one choice
enum Color {
    Red = 0xff0000,
    Green = 0x00ff00,
    Blue = 0x0000ff,
}
```

```rust
// A set of flags: use bitflags
use bitflags::bitflags;

bitflags! {
    struct Flags: u32 {
        const FLAG_A = 0b00000001;
        const FLAG_B = 0b00000010;
        const FLAG_C = 0b00000100;
    }
}

fn f(settings: Flags) {
    if settings.contains(Flags::FLAG_A) {
        println!("doing thing A");
    }
    if settings.contains(Flags::FLAG_B) {
        println!("doing thing B");
    }
    if settings.contains(Flags::FLAG_C) {
        println!("doing thing C");
    }
}

fn main() {
    f(Flags::FLAG_A | Flags::FLAG_C);
}
```

<a id="c-builder"></a>

## C-BUILDER — Builders enable construction of complex values

**Rule:** If constructing a data structure `T` needs many inputs, compound data (e.g. slices), optional configuration, or a choice between several flavors (which leads to many constructors with many arguments), you **SHOULD** consider introducing a `T` builder.

**Details:**
- Builder shape:
  1. A separate type `TBuilder` incrementally configures a `T`. Choose a better name when possible: e.g. [`Command`](https://doc.rust-lang.org/std/process/struct.Command.html) is the builder for a [child process](https://doc.rust-lang.org/std/process/struct.Child.html); [`Url`](https://docs.rs/url/1.4.0/url/struct.Url.html) can be created from a [`ParseOptions`](https://docs.rs/url/1.4.0/url/struct.ParseOptions.html).
  2. The builder constructor **SHOULD** take only the data *required* to make a `T`.
  3. The builder **SHOULD** offer convenient configuration methods, including setting up compound inputs (like slices) incrementally; these **SHOULD** return `self` to allow chaining.
  4. The builder **SHOULD** provide one or more *terminal* methods that actually build a `T`.
- Builders are especially appropriate when building a `T` has side effects, such as spawning a task or launching a process.
- Two variants, differing in ownership:
- **Non-consuming builders (preferred):** when building `T` does not require consuming the builder.
  - E.g. a simplified [`std::process::Command`](https://doc.rust-lang.org/std/process/struct.Command.html): the terminal `spawn` takes `&self` because spawning does not need ownership of the configuration; so configuration methods take and return `&mut self`.
  - Benefit: borrows throughout make both one-liners and complex (conditional) configuration convenient.
- **Consuming builders:** when terminal methods must transfer ownership (take `self`, not `&self`), e.g. a `TaskBuilder` whose `stdout` writer must be moved into the task in `spawn`.
  - Trade-off: if configuration methods take/return `&mut self`, complex configuration works well but one-liners become impossible; if they take/return owned `self`, one-liners work but complex configuration is less convenient.
  - Under "make easy things easy and hard things possible", all methods of a consuming builder **SHOULD** take and return owned `self`. One-liners then work as before (ownership threads through to `spawn`); complex configuration is more verbose, requiring re-assignment of the builder at each step.

**Google note:** Google adds an alternative for option bags: structs with all-public fields used primarily as a "bag of options" for a function should implement `Default`, enabling struct update syntax ([G-TRT-09](../google/07-traits.md#g-trt-09)). See also secondary-constructor naming ([G-NAM-05](../google/10-naming.md#g-nam-05)).

**Example:**
```rust
// Non-consuming builder.
// NOTE: the actual Command API does not use owned Strings;
// this is a simplified version.

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // etc
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// Add an argument to pass to the program.
    pub fn arg(&mut self, arg: String) -> &mut Command {
        self.args.push(arg);
        self
    }

    /// Add multiple arguments to pass to the program.
    pub fn args(&mut self, args: &[String]) -> &mut Command {
        self.args.extend_from_slice(args);
        self
    }

    /// Set the working directory for the child process.
    pub fn current_dir(&mut self, dir: String) -> &mut Command {
        self.cwd = Some(dir);
        self
    }

    /// Executes the command as a child process, which is returned.
    pub fn spawn(&self) -> io::Result<Child> {
        /* ... */
    }
}
```

```rust
// One-liners
Command::new("/bin/cat").arg("file.txt").spawn();

// Complex configuration
let mut cmd = Command::new("/bin/ls");
if size_sorted {
    cmd.arg("-S");
}
cmd.arg(".");
cmd.spawn();
```

```rust
// Consuming builder
impl TaskBuilder {
    /// Name the task-to-be.
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// Redirect task-local stdout.
    pub fn stdout(mut self, stdout: Box<io::Write + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        self
    }

    /// Creates and executes a new child task.
    pub fn spawn<F>(self, f: F) where F: FnOnce() + Send {
        /* ... */
    }
}
```

```rust
// One-liners
TaskBuilder::new("my_task").spawn(|| { /* ... */ });

// Complex configuration
let mut task = TaskBuilder::new();
task = task.named("my_task_2"); // must re-assign to retain ownership
if reroute {
    task = task.stdout(mywriter);
}
task.spawn(|| { /* ... */ });
```

**See also:** [C-CTOR](05-predictability.md#c-ctor)
