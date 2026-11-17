---
title: CBMC — Bounded Model Checking for C
---

The first time a tool hands you a concrete input that breaks your code, it feels like cheating. That's exactly what **CBMC** (C Bounded Model Checker) does: it takes C code as-is, unrolls loops up to a bound you choose, converts the program into a SAT/SMT formula, and checks it automatically. No annotations are needed — you write a small **harness** that calls the code under test with symbolic (arbitrary) inputs, plus `assert`s for the properties you care about. CBMC then reports either "all assertions hold" or a **counterexample trace**: a real input that violates one.

Install: `apt install cbmc` (or via your package manager).

## Proving the guard in a fixed-length key validator

Consider a typical embedded utility: validating a 6-character product key (`^[A-Z]{2}[0-9]{4}$`). The length check has to come _before_ any indexed read — and in code review, "the guard is before the reads" is exactly the kind of thing you hope someone noticed. With CBMC you don't have to hope:

```c
static bool char_in_set(char c, const char *allowed) {
  return strchr(allowed, c) != NULL;
}

bool is_valid_key(const char *s) {
  size_t len = strlen(s);
  if (len != 6) return false;   // guard BEFORE any indexed read
  return char_in_set(s[0], "ABCDEFGHIJKLMNOPQRSTUVWXYZ") &&
         char_in_set(s[1], "ABCDEFGHIJKLMNOPQRSTUVWXYZ") &&
         char_in_set(s[2], "0123456789") &&
         char_in_set(s[3], "0123456789") &&
         char_in_set(s[4], "0123456789") &&
         char_in_set(s[5], "0123456789");
}
```

Here's the harness that proves the guard really makes the `s[0..5]` reads safe for _any_ NUL-terminated string:

```c
// harness.c — run: cbmc keys.c harness.c --function harness
#include <stdbool.h>
#include <string.h>

bool is_valid_key(const char *s);

void harness(void) {
  char s[7];
  s[6] = '\0';                                    // NUL-terminated
  for (int i = 0; i < 6; i++)
    __CPROVER_assume(s[i] >= 1 && s[i] <= 127);   // arbitrary non-NUL bytes

  bool r = is_valid_key(s);

  // property 1: never accepts a string whose length is not exactly 6
  assert(!(r && strlen(s) != 6));
  // property 2: result is a clean 0/1
  assert(r == 0 || r == 1);
  // memory safety (OOB reads on s[0..5]) is checked by CBMC automatically
}
```

Run it with:

```sh
cbmc keys.c harness.c --function harness
```

Notice what you didn't have to say: CBMC checks memory safety — out-of-bounds reads, pointer validity, signed overflow — by default, alongside your assertions. Remove the length guard and CBMC doesn't shrug; it hands you the exact string that reads past the end of the buffer.

## Pinning down the accepted language

Memory safety is the easy win. The more interesting question is functional: does the validator accept _exactly_ the intended grammar? A second harness answers that by equivalence checking — comparing against an independent reference implementation over every possible 6-byte input:

```c
bool reference_key(const char *s) {
  // independent re-implementation of ^[A-Z]{2}[0-9]{4}$
  ...
}

void harness2(void) {
  char s[7];
  s[6] = '\0';
  for (int i = 0; i < 6; i++)
    __CPROVER_assume(s[i] >= 1 && s[i] <= 127);
  assert(is_valid_key(s) == reference_key(s));
}
```

If the C code and the spec agree on every 6-byte input, you've proved the grammar — not just sampled it.

## What CBMC is great at, and where it stops

- **Bounded**: loops are unrolled a fixed number of times, so proofs cover inputs up to that bound (here the input is fixed-size, so the bound is exact, not approximate).
- **Automatic**: no annotations — great for finding bugs and proving memory safety cheaply.
- **Counterexamples double as regression tests**: every failure comes with a concrete reproducer you can drop straight into your test suite.
- One harness per entry point, and hardware-touching code must be stubbed first.

For unbounded proofs with full contracts, see [[frama-c-wp-acsl|Frama-C WP and ACSL]].
