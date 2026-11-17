---
title: Formal Verification Tools for C++
---

C gets [[CBMC]] and [[frama-c-wp-acsl|Frama-C WP]] — but what about C++? The honest answer: the tooling is thinner, because templates, exceptions, and the STL are hard to model symbolically. "Thinner" isn't "empty", though, and if your C++ project keeps a small pure core — the same fixed-length key validator as in the [[case-study-fixed-length-key-validator|case study]] — there are real tools that will prove things about it.

## Bounded model checking: ESBMC (and CBMC)

**ESBMC** is CBMC's successor with first-class C++ support: it parses C++ (classes, methods, inheritance, templates within limits), converts the program to a goto model, and checks it with SMT solvers. The workflow is exactly the [[cbmc|CBMC]] one — a harness, symbolic inputs, `assert`s, counterexamples:

```cpp
// harness.cpp — run: esbmc keys.cpp harness.cpp --function harness
#include <string.h>

bool is_valid_key(const char *s);

void harness() {
  char s[7];
  s[6] = '\0';                                     // NUL-terminated
  for (int i = 0; i < 6; i++)
    __ESBMC_assume(s[i] >= 1 && s[i] <= 127);      // arbitrary non-NUL bytes

  bool r = is_valid_key(s);

  // property 1: never accepts a string whose length is not exactly 6
  assert(!(r && strlen(s) != 6));
  // property 2: result is a clean 0/1
  assert(r == 0 || r == 1);
  // memory safety (OOB reads on s[0..5]) is checked automatically
}
```

CBMC itself also accepts a large subset of C++ (`cbmc keys.cpp harness.cpp --function harness`), which is handy when the code is basically C with classes. The practical rule for both: keep the verified unit C-like — free functions over plain data and `char*` — and keep `std::string`, exceptions, and virtual dispatch out of the harness unless you provide models for them (ESBMC ships models for parts of the STL).

## Deductive verification: VeriFast

For pointer-heavy C++ that needs real contracts — preconditions, postconditions, loop invariants — **VeriFast** verifies both C and C++ using separation logic. Contracts live in `//@` comments, so the code still compiles as ordinary C++:

```cpp
//@ requires s != NULL && chars(s, ?len, ?cs);  // readable, NUL-terminated buffer
//@ ensures \result == 0 || \result == 1;
//@ ensures \result == 1 ==> len == 6;           // accepted keys are exactly 6 chars
bool is_valid_key(const char *s) {
  ...
}
```

VeriFast checks every proof obligation and reports exactly which ones fail — the same "residual goals are your to-do list" experience as Frama-C WP, but with support for C++ classes, member functions, and ownership predicates. Install it from a binary release or via `opam install verifast`.

## The honest limits

- There is no ACSL for C++ — Frama-C WP is C-only — so deductive C++ verification means VeriFast or research tools.
- C++20 contracts (`expects`/`ensures`) were dropped from the standard; `assert` remains, and it's what model checkers consume as proof obligations.
- [cppcheck](https://cppcheck.sourceforge.io/) and clang-tidy are worth running — free, and they catch real bugs — but they are static analysis, not proofs: they can flag, never guarantee.
- Templates and heavy STL usage are where symbolic tools give up first. Verify the small pure core, and keep the rest behind tested interfaces.

Related: [[what-is-formal-verification|intro]], [[cbmc|CBMC]], [[frama-c-wp-acsl|Frama-C WP and ACSL]], [[python-tools|Python tools]], [[case-study-fixed-length-key-validator|the case study]].
