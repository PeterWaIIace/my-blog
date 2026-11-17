---
title: Frama-C WP and ACSL
---

CBMC is bounded: it unrolls loops a fixed number of times, so its proofs cover inputs up to that bound. What if you want a guarantee for _all_ inputs, with no bound at all? That's **deductive verification**, and in the C world the tool for it is **Frama-C**'s **WP** plugin. Instead of exploring inputs, WP proves that your code satisfies a written _contract_, using induction. The contract lives in **ACSL** (ANSI/ISO C Specification Language) as special comments that the compiler ignores, so the same source still builds normally:

- `requires` — what the caller must guarantee (preconditions)
- `ensures` — what the function guarantees afterwards (postconditions), with `\result` for the return value
- `assigns` — which memory the function may write
- `invariant` — loop invariants (the induction step)
- `\valid_read(p + (i..j))` — a range of memory is readable

Install: `opam install frama-c`, plus Why3 and an SMT solver (Z3, Alt-Ergo, or CVC5): `why3 config detect`.

## Writing the contract for a fixed-length key validator

Here's what a contract looks like in ACSL:

```c
/*@
  requires \valid_read(s + (0 .. strlen(s)));  // s is a readable, NUL-terminated string
  assigns  \nothing;                            // pure function, no side effects
  ensures  \result == 0 || \result == 1;
  ensures  \result == 1 ==> strlen(s) == 6;    // accepted keys are exactly 6 chars
*/
bool is_valid_key(const char *s) {
  ...
}
```

WP turns this into _verification conditions_ (via Why3) and discharges them with the SMT solver. If every condition is proved, the contract holds unconditionally — all inputs, no bound. Anything left unproved shows up as a residual goal, and that list is basically your to-do list.

## CBMC vs. WP: when to reach for which

|             | [[CBMC]]                                         | Frama-C WP + ACSL              |
| ----------- | ------------------------------------------------ | ------------------------------ |
| Annotations | none (asserts in a harness)                      | contracts + loop invariants    |
| Coverage    | bounded (loops unrolled N times)                 | unbounded (inductive)          |
| Best for    | bug-finding, memory safety, small pure functions | proving a full functional spec |
| Output      | pass/fail + counterexample                       | pass/fail per proof obligation |

In practice they complement each other, and the natural workflow is: CBMC first to shake out memory-safety bugs cheaply, then WP/ACSL to pin the functional spec.

## Where this fits in an embedded C project

Firmware repos typically already compile their pure logic on host with mocked hardware registers (Unity-style tests), so WP and CBMC can verify the same translation units on a PC — no device needed. Natural targets: a control-loop threshold check that never fires inside its configured deadband, or a parser that rejects every malformed frame. [[case-study-fixed-length-key-validator|The case study]] walks through the validator properties.
