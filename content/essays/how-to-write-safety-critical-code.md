---
title: How to write safety critical code
date: 2026-05-05
description: Revisiting NASA's "Power of Ten" safety rules, and how to turn them into daily coding habits.
tags:
  - HCI
  - systems
  - HCC
  - programming
  - c
draft: false
---
Software fails in ways that other engineered artifacts simply do not. When a bridge collapses under a heavy load, it fails visibly, leaving behind physical evidence that a post-hoc analysis can usually reconstruct. Software, by contrast, fails silently and conditionally. And some bugs emerge only under a precise, malicious combination of inputs that occurs once in every ten million executions.

Some NASA examples:

- **[The Mars Climate Orbiter (1999)](https://llis.nasa.gov/lesson/641):** Lost because two subsystems exchanged data using different unit conventions.
    
- **[The THERAC-25 Radiation Machine](https://www.computer.org/csdl/magazine/co/1993/07/r7018/13rRUx0xPDf):** Delivered lethal doses of radiation because a race condition in concurrent code slipped through standard testing.
    
- **[Ariane 5 Flight 501](http://sunnyday.mit.edu/nasa-class/Ariane5-report.html):** Destroyed itself 64 seconds after launch because software designed for a slower rocket was reused without revalidating its numeric assumptions.

In every single case, the code had been reviewed. It had passed whatever quality gates the organization threw at it. And in every single case, those gates were completely insufficient.

When software quality slips, the gut reaction is almost always bureaucratic AKA compliance theater. Write more rules, mandate more reviews, stack up the paperwork. It’s how we end up with massive, bloated coding standards where critical safety constraints are buried right alongside arbitrary formatting preferences.

The failure mode here is obvious. Nobody actually reads a 300-page manual (well most of us). It gets consulted selectively, applied inconsistently, and completely thrown out the window the second a deadline gets tight.

In 2006, Gerard J. Holzmann’s paper, *[The Power of Ten: Rules for Developing Safety-Critical Code](https://spinroot.com/gerard/pdf/P10.pdf),* gave a simple alternative. Rather than encoding *more* rules, it argued for *fewer* rules, selected by a single, uncompromising criterion: **each rule must be mechanically checkable by a static analysis tool.** A rule that can be verified automatically doesn’t depend on individual discipline, organizational culture, or the stress tolerance of a developer three days before a launch window. It either passes or it fails, and that determination happens long before the code ever ships.

Twenty years later, I'm revisiting this paper.

## The Ten Rules, Revisited

### 1. Restrict to simple control flow--no recursion or goto statements.

Back when I first read *The Power of Ten*, I was deep into functional programming dogma, consuming "functional bro" content like a maniac. Naturally, the recursion ban surprised me. It still surprises people. But Holzmann’s argument is structural: recursion introduces cycles into the call graph, which completely breaks bounded-execution proofs.

For ordinary application code, fine--recur away. But for the code keeping a rover alive, you build it iteratively. [Tail-call optimization](https://www.youtube.com/watch?v=-PX0BV9hGZY) (here is a fun video if you're unfamiliar) buys you almost nothing here; it helps the runtime, not the analyzer. Modern structured concurrency disciplines (eg, Rust’s scoped threads) are essentially doing for concurrent control flow what this rule does for sequential code: bounding it, giving it a parent, and making it static.

### 2. All loops must have a fixed upper bound.

This is the rule I find myself thinking about most, regardless of the language I'm using. In Rust, finite-collection iterators give you this guarantees for free. In C, I  reflexively add a counter and an `assert(i < MAX_ITERS)`, even on `while` loops I am absolutely certain will terminate. The cost is a single variable; the benefit is catching the day I turn out to be wrong.

Holzmann was careful to carve out the inverse case--event loops, schedulers, and systems meant to run forever--demanding a static proof that those loops *cannot* terminate. That symmetry is the nuance most people miss. The rule isn't "everything must end." It’s "your tooling must always know which state it's in."

### 3. No dynamic memory allocation after initialization.

To me, this remains the cleanest, most uncompromising line in the entire document. `malloc` is the root of an entire genus of software bugs--leaks, double-frees, use-after-free errors, fragmentation, and unpredictable latency. While Rust’s ownership model eliminates the misuse of memory, it doesn't eliminate the unpredictability of allocation itself. Embedded and real-time systems still demand static or arena-allocated memory.

The modern phrasing of this rule is the arena allocator: grab a massive block of memory up front, hand out slices from it as needed, and free the entire region at once when a phase ends. Watch the first 30 videos of the [Handmade Hero](https://guide.handmadehero.org/) to understand how Casey employees this rule. Zig builds its entire language ecosystem around this philosophy, and C++’s polymorphic memory resources are the exact same instinct in heavier clothing. Garbage-collected languages cannot satisfy this rule on their own, which is one of the quieter reasons safety-critical code is rarely written in them.

### 4. Limit functions to 60 lines or fewer.

This is all about mental real estate. If you can't see a function in its entirety without scrolling, you probably can't reason about it reliably either. However, I sometimes break this rule when using crazy verbose boilerplate APIs (I'm talking about you Vulkan).

### 5. Minimum of two assertions per function.

Modern testing practices have partially absorbed this instinct into property-based testing and fuzzing, but Holzmann's core logic remains. Inline assertions document invariants directly at the point of enforcement. Rust’s `debug_assert!` macro and the robust contract systems of Ada/SPARK are direct descendants of Tony Hoare’s 1969 work on logical reasoning and Bertrand Meyer's "design by contract."

Two assertions per function is a density target, not an arbitrary ritual; most of mine are used to check pre- and post-conditions. An assertion is the only kind of documentation a compiler will eventually punish you for getting wrong.

### 6. Declare data at the smallest possible scope.

Modern languages make compliance almost effortless: `let` in Rust, block-scoped variables in JavaScript, and list comprehensions in Python that encapsulate their own iterators. Where this rule still bites is in permissive legacy languages where loop variables leak or class state can be mutated from anywhere. In those environments, discipline matters more, not less.

The harder, more insidious version of this rule concerns *temporal* scope--not just *where* a variable can be referenced, but *when*. Closures violate this intentionally by capturing variables past their lexical lifetimes. Many of the worst closure bugs I’ve encountered stem from forgetting that a captured loop variable was shared rather than copied. Python's `nonlocal` keyword exists precisely because the language tried to enforce scope hygiene and ran head-first into the complexities of the closure problem.

### 7. Check return values and validate parameters internally.

This is the rule modern languages have solved most decisively. Rust’s `Result<T, E>` turns error handling into a compile-time requirement, while the `?` operator reduces error propagation to a single character of syntax. Go’s multiple-return convention forces the error directly in front of you at every call site. Haskell’s `Maybe` and `Either` types enforce the same strictness through pure type theory.

In C, however, you still have to remember to check. And the cost of forgetting just once can be a satellite. The deeper argument under this rule is that exceptions are a quiet violation of it--they allow a function to fail implicitly without forcing the caller to acknowledge it, unwinding the stack through layers of code that never explicitly opted into handling that failure. Languages that lean on exceptions for ordinary control flow are quietly fighting against this rule.

### 8. Limit the preprocessor.

While this rule is C-specific in letter, it is entirely universal in spirit: **avoid metaprogramming that obscures the code actually running on the machine.** C++ templates, Lisp macros, and Python decorators that invisibly mutate behavior are all the same hazard wearing different hats. They should be used sparingly and documented loudly.

Rust’s hygienic macros are an honest attempt to provide the power of the C preprocessor without its notorious foot-guns: they are scoped, typed, expand predictably, and can be inspected via `cargo expand`. Build-time code generation (like `build.rs` or OpenAPI codegen) occupies the same ecological niche. The underlying principle Holzmann is fighting for is parity: the code your reviewer reads should be the exact code the compiler executes. Anything that forces those two paths to diverge needs to fiercely earn its keep.

### 9. Restrict pointer use; completely avoid function pointers.

Rust’s borrow checker is essentially this rule, mechanized and built into a core language feature. The "no function pointers" clause is the one I find myself most frequently tempted to violate--callbacks and dispatch tables are simply too useful to abandon. But Holzmann’s warning is valid: every function pointer creates a blind spot in the static call graph. If you must use one, you owe your team an explicit comment explaining why.

The compromise I usually settle on is a bounded jump table: an enum, an index, and a strict switch statement. The dispatch remains data-driven, but the call graph is securely closed, allowing a static analyzer to walk it. C++ smart pointers and Rust references are the modern realization of this rule, embedding lifetime and ownership mechanics directly into the type system to make memory legible to the compiler.

### 10. Compile clean at the strictest warning level; run a static analyzer daily.

This rule has aged better than perhaps any other on the list. With an ecosystem boasting Clippy, Clang-Tidy, Mypy, Ruff, ESLint, Semgrep, and Cargo Audit, there is no longer any valid excuse for non-compliance. Wire them into your pre-commit hooks and your CI pipelines. Treat warnings as fatal errors using `-Werror`, `#![deny(warnings)]`, or whatever flag your specific toolchain requires. You pay the setup cost once; you reap the benefits forever.

The newest entrants in this space are LLM-backed and advanced semantic analyzers like Coverity or GitHub Code Scanning. While they aren't replacements for deterministic tools, they catch a completely different class of bugs, and the right approach is to use both. If an analyzer flags your code and you are convinced it’s a false positive, rewrite the code anyway. Holzmann was explicit about this: when the analyzer gets confused, the code that confused it is the code at fault.

## Living with the Rules

Reading these rules is easy. Living with them requires an entirely different set of operational habits, which are what actually drive bug rates down.

* **Automate Compliance:** Put as much of the rule set into your toolchain as possible so you don't have to waste willpower remembering it. Use pre-commit hooks for formatting, linting, and fast type-checking, and delegate slower processes--like fuzzing budgets and comprehensive dependency audits--to CI gates. When compliance is built into the pipeline, it ceases to be a personal virtue and becomes the default. That is the only way it survives a looming deadline.
* **Maintain an Assertion Budget:** When writing a new function, write the pre-condition assertion immediately after the signature, and the post-condition immediately before the return. This simple habit guarantees compliance with Rule 5, and it forces you to explicitly define the function's contract before writing its body--which is usually where the bugs would have lived.
* **Develop a Loop-Bound Reflex:** Every `while` loop gets a dedicated safety counter. Every `for` loop executing over a non-trivial data source gets a sanity check on its iteration count. It costs a single line of code, but the number of times it has caught me making a flawed assumption is genuinely embarrassing.
* **Practice Active Review Hygiene:** When reviewing a peer's code, keep Holzmann's list running as a mental checklist. Ask yourself systematically: *Can I map the call graph? Can I bound every loop? Where is this memory coming from? Are the errors checked? Are the assumptions asserted?* It takes far less time than it sounds, and it yields highly specific, actionable feedback rather than a sense of vague unease.

## The Ultimate Takeaway

The singular thread running through all ten rules is clear: **write code that a machine can prove things about, not just code that a human can understand.** Twenty years of language design have made many of these constraints significantly cheaper--and occasionally completely free--to implement. The ones that remain expensive, like strict bans on recursion or dynamic allocation, are expensive precisely because they are carrying the entire weight of the system's reliability.

I don't follow all ten rules when writing a throwaway script; pretending otherwise would just be cargo-culted rigor. But when the stakes go up--when the code touches a patient, a physical sensor, or a piece of aerospace hardware--the list comes back out. Holzmann’s seatbelt analogy is perfect: it's slightly uncomfortable for the first afternoon, but after that, operating without it becomes entirely unimaginable.