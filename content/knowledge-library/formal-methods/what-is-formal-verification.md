---
title: What Is Formal Verification?
---

A **test** runs the program on a finite set of inputs and checks the outputs. It can show the presence of bugs, never their absence — so even after a fully green run, you're still trusting that you thought of the right inputs. **Formal verification** replaces that trust with a machine-checked mathematical proof that a property holds for _all_ inputs in a stated domain. Not "I tried a hundred cases", but "there is no case".

## The ladder from testing to proof

Each rung buys a stronger guarantee for a higher price:

| Approach                                             | What it proves                                   | Cost                      |
| ---------------------------------------------------- | ------------------------------------------------ | ------------------------- |
| Unit tests (e.g. Unity-style host tests with mocks)  | Behavior on chosen inputs only                   | Low                       |
| Property-based tests (random inputs vs. a reference) | Many random inputs, shrunk counterexamples       | Low                       |
| Bounded model checking ([[CBMC]])                    | All inputs within a bound; memory safety         | Medium, automatic         |
| Deductive verification ([[Frama-C WP and ACSL]])     | Full spec, all inputs, via contracts + induction | Higher, needs annotations |

## Why embedded C is a good place to start

Bare-metal firmware is C on a microcontroller, and the risky parts aren't the drivers — they're the small, pure functions that sit between hardware reads and actuator writes: validators, parsers, state transitions. A single out-of-bounds read or an off-by-one in a length check is exactly the class of bug that formal tools catch well:

- **Memory safety comes almost for free**: CBMC checks pointer validity, array bounds, and overflow without any annotations.
- **Implicit invariants get made explicit**: a validator that reads `s[0..5]` only after a `strlen(s) == 6` guard is safe today because of that ordering — a proof obligation states the contract in the code instead of leaving it to code review.
- **Host-testable**: pure functions compile on a PC with stubs for hardware registers, so verification can run in CI without a device.

## The honest caveats

Worth saying out loud before getting too excited:

- Proofs are relative to a _specification_. "Proved correct" means "matches the spec" — and writing the spec is the hard part.
- Bounded checking is not a full proof for unbounded loops; deductive verification handles those, but needs loop invariants, which take effort.
- Hardware interactions (ADC, radio, timers) still need testing on-device; formal methods cover the logic around them.

See [[case-study-fixed-length-key-validator|the fixed-length key validator case study]] for what this looks like on real-shaped code. Language-specific tooling: [[cpp-tools|C++]] and [[python-tools|Python]].
