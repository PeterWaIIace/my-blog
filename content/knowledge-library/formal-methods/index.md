---
title: Formal Methods
---

Here's an uncomfortable truth about testing that I keep coming back to: a green test suite can show that bugs _exist_, but never that they're _absent_. Run a hundred inputs through a validator and you're still trusting that you thought of the right ones. Formal methods replace that trust with mathematics — instead of sampling inputs, a machine _proves_ that a property holds for all of them (or all of them within a chosen bound).

These are my working notes on doing that in practice, motivated by embedded C projects, where small pure functions — validators, parsers, threshold checks — sit between hardware reads and actuator writes, and a single off-by-one can apply the wrong device profile.

## The journey

- [[what-is-formal-verification|What is formal verification?]] — the test-vs-proof mindset shift, and why embedded C is a great first target
- [[cbmc|CBMC: bounded model checking for C]] — proving properties of C directly, no annotations, with a harness for a fixed-length key validator
- [[frama-c-wp-acsl|Frama-C WP and ACSL]] — deductive verification with contracts and loop invariants, for full unbounded proofs
- [[cpp-tools|Formal verification tools for C++]] — ESBMC/CBMC for C++, VeriFast for deductive contracts
- [[python-tools|Formal verification tools for Python]] — CrossHair, Z3, Hypothesis, and strict typing on a practical ladder
- [[case-study-fixed-length-key-validator|Case study: a fixed-length key validator]] — the concrete properties worth proving on a real-shaped example
