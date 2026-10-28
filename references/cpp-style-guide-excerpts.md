# Google C++ Style Guide — Excerpts Cited by the Rust Guide

> **Source:** [Google C++ Style Guide — Goals of the Style Guide](https://google.github.io/styleguide/cppguide.html#Goals) and [Google C++ Style Guide — Choosing Names](https://google.github.io/styleguide/cppguide.html#General_Naming_Rules) · **License:** CC-BY 3.0 (code samples Apache-2.0)
>
> The two sections of the Google C++ Style Guide that the Google Rust Style Guide cites: the goals behind Google's style rules, and the general rules for choosing names.

Only these two sections are cited by the Google Rust Style Guide, so only they are condensed here; the rest of the C++ guide is out of scope. The Goals section is cited from [the Rust guide's Goals](../google/01-background-and-scope.md) ("we have this document for the same reasons we have a C++ style guide"); the General Naming Rules section is cited from [G-NAM-06 — Avoid abbreviations](../google/10-naming.md#g-nam-06). C++-specific examples are kept as written.

## Goals: purpose

The goals are the fundamental "why"s underlying all individual rules. Knowing which goal a rule serves makes it clearer when a rule may be waived (some can be) and what argument or alternative would be needed to change it.

<a id="cpp-goal-01"></a>

## CPP-GOAL-01 — Style rules should pull their weight

**Rule:** A style rule's benefit **MUST** be large enough to justify asking all engineers to remember it.

**Details:**
- Benefit is measured relative to the codebase without the rule, so a rule against a very harmful practice may still have small benefit if people are unlikely to do it anyway.
- This mostly explains rules the guide does *not* have: e.g. `goto` contravenes many principles but is already vanishingly rare, so the guide doesn't discuss it.

<a id="cpp-goal-02"></a>

## CPP-GOAL-02 — Optimize for the reader, not the writer

**Rule:** Code **SHOULD** be optimized for the experience of the average engineer reading, maintaining and debugging it, rather than for ease of writing it.

**Details:**
- The codebase is long-lived, so more time is spent reading most code than writing it.
- "Leave a trace for the reader": when something surprising or unusual happens (e.g. transfer of pointer ownership), leave textual hints at the point of use (`std::unique_ptr` shows the ownership transfer unambiguously at the call site).

<a id="cpp-goal-03"></a>

## CPP-GOAL-03 — Be consistent with existing code

**Rule:** One style **SHOULD** be used consistently throughout the codebase.

**Details:**
- Consistency lets people focus on more important issues and enables automation (formatters and include-fixers only work when code matches the tooling's expectations).
- Many "Be Consistent" rules boil down to "just pick one and stop worrying about it": the value of flexibility is outweighed by the cost of arguing.
- Consistency is a good tie-breaker when there is no clear technical argument nor long-term direction; it applies more heavily locally (per file, or a tightly related set of interfaces).

**Exceptions:**
- Consistency **SHOULD NOT** generally justify doing things in an old style without considering the benefits of the new style, or the codebase's tendency to converge on newer styles over time.

<a id="cpp-goal-04"></a>

## CPP-GOAL-04 — Be consistent with the broader C++ community when appropriate

**Rule:** Standard features and widely known, accepted idioms **SHOULD** be used when they solve the problem, for the same reasons as internal consistency.

**Exceptions:**
- Standard features and idioms that are flawed or designed without the codebase's needs in mind may be constrained or banned.
- A homegrown or third-party library may be preferred over the C++ Standard library, for perceived superiority or because transitioning the codebase to the standard interface has insufficient value.

<a id="cpp-goal-05"></a>

## CPP-GOAL-05 — Avoid surprising or dangerous constructs

**Rule:** Constructs that are more surprising or dangerous than they appear **SHOULD** be avoided; restrictions exist to prevent these pitfalls.

**Details:**
- There is a high bar for waivers of such restrictions, because waiving them often directly risks program correctness.

<a id="cpp-goal-06"></a>

## CPP-GOAL-06 — Avoid constructs that the average programmer would find tricky or hard to maintain

**Rule:** Features whose complexity makes them not generally appropriate **SHOULD** be avoided.

**Details:**
- In widely used code, trickier constructs may be more acceptable: benefits multiply with usage, and the cost of understanding them isn't paid again for each new part of the codebase.
- When in doubt, seek waivers by asking your project leads.
- Ownership and team membership change over time; current understanding of code is not guaranteed to hold a few years later.

<a id="cpp-goal-07"></a>

## CPP-GOAL-07 — Be mindful of our scale

**Rule:** Decisions **SHOULD** account for scale (100+ million lines, thousands of engineers), where one engineer's mistake or simplification can become costly for many.

**Details:**
- E.g. avoid polluting the global namespace: name collisions across hundreds of millions of lines are hard to work with and hard to avoid if everyone uses the global namespace.

<a id="cpp-goal-08"></a>

## CPP-GOAL-08 — Concede to optimization when necessary

**Rule:** Performance optimizations **MAY** be used when necessary and appropriate, even when they conflict with the other principles.

<a id="cpp-goal-09"></a>

## CPP-GOAL-09 — Common sense and good taste prevail

**Rule:** The guide aims for maximal guidance with reasonable restriction; common sense and good taste — meaning the established conventions of the entire Google C++ community, not personal or team preferences — **SHOULD** prevail. Clever or unusual constructs **SHOULD** be treated with skepticism and reluctance.

**Details:**
- The absence of a prohibition is not a license to proceed.
- Use your judgment; if unsure, ask your project leads for additional input.

## Naming terminology

From the intro of the C++ guide's Naming section (included because CPP-NAM-03 relies on it): a "word" is anything you would write in English without internal spaces. Words are either all lowercase with underscores between them ([snake_case](https://en.wikipedia.org/wiki/Snake_case)) or mixed case with each word's first letter capitalized ([camelCase](https://en.wikipedia.org/wiki/Camel_case) or [PascalCase](https://en.wiktionary.org/wiki/Pascal_case)).

<a id="cpp-nam-01"></a>

## CPP-NAM-01 — Make the purpose understandable to a new reader

**Rule:** Names **SHOULD** make their purpose or intent understandable to a new reader, even someone on a different team than the owners.

**Details:**
- Don't worry about saving horizontal space; immediate understandability by a new reader matters far more.

<a id="cpp-nam-02"></a>

## CPP-NAM-02 — Scale descriptiveness with scope and context

**Rule:** A name **SHOULD** be descriptive even when used far from where it is made available, but **SHOULD NOT** distract the reader by repeating information present in the immediate context.

**Details:**
- Generally, descriptiveness should be proportional to the name's scope of visibility.
- A free function declared in a header should probably mention the header's library; a local variable probably shouldn't explain which function it's in.

<a id="cpp-nam-03"></a>

## CPP-NAM-03 — Minimize unfamiliar abbreviations

**Rule:** Abbreviations likely unknown outside your project (especially acronyms and initialisms) **SHOULD** be minimized; you **MUST NOT** abbreviate by deleting letters within a word.

**Details:**
- When an abbreviation is used, prefer capitalizing it as a single "word": `StartRpc()`, not `StartRPC()`.
- Rule of thumb: an abbreviation is probably OK if it's listed in Wikipedia.

**Exceptions:**
- Certain universally known abbreviations are OK, e.g. `i` for a loop index and `T` for a template parameter.

**See also:** [G-NAM-06](../google/10-naming.md#g-nam-06)

<a id="cpp-nam-04"></a>

## CPP-NAM-04 — Vocabulary names

**Rule:** If you define a new "vocabulary" name, you **SHOULD** get additional design review to make sure it works well when used widely.

**Details:**
- A small number of vocabulary names are reused so widely that they are always in context; they tend to be short or even abbreviated, and their full meaning comes from explicit long-form documentation rather than comments on their definition or the words in the name.
- E.g. `absl::Status` has a dedicated [page in a devguide](https://abseil.io/docs/cpp/guides/status) documenting its proper use.
- You probably won't define new vocabulary names very often.

## Examples for CPP-NAM-01 to CPP-NAM-04

The current page has no separate abbreviation table; its naming and abbreviation examples are the annotated good/bad code samples below.

```cpp
// Good
class MyClass {
 public:
  int CountFooErrors(const std::vector<Foo>& foos) {
    int n = 0;  // Clear meaning given limited scope and context
    for (const auto& foo : foos) {
      ...
      ++n;
    }
    return n;
  }
  // Function comment doesn't need to explain that this returns non-OK on
  // failure as that is implied by the `absl::Status` return type, but it
  // might document behavior for some specific codes.
  absl::Status DoSomethingImportant() {
    std::string fqdn = ...;  // Well-known abbreviation for Fully Qualified Domain Name
    return absl::OkStatus();
  }
 private:
  const int kMaxAllowedConnections = ...;  // Clear meaning within context
};
```

```cpp
// Bad
class MyClass {
 public:
  int CountFooErrors(const std::vector<Foo>& foos) {
    int total_number_of_foo_errors = 0;  // Overly verbose given limited scope and context
    for (int foo_index = 0; foo_index < foos.size(); ++foo_index) {  // Use idiomatic `i`
      ...
      ++total_number_of_foo_errors;
    }
    return total_number_of_foo_errors;
  }
  // A return type with a generic name is unclear without widespread education.
  Result DoSomethingImportant() {
    int cstmr_id = ...;  // Deletes internal letters
  }
 private:
  const int kNum = ...;  // Unclear meaning within broad scope
};
```
