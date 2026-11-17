---
title: Case Study — Fixed-Length Key Validator
---

This is where it gets concrete. A representative embedded pattern: `is_valid_key()` validates a 6-character product key read from device flash, e.g. `AB1234`. Get the validation wrong and the wrong device profile gets applied — the kind of bug a unit test only catches if you happened to write the one input that triggers it, and exactly the kind formal methods are built to eliminate.

## The code under test

```c
static bool char_in_set(char c, const char *allowed) {
  return strchr(allowed, c) != NULL;
}

bool is_valid_key(const char *s) {
  size_t len = strlen(s);
  if (len != 6) return false;
  return char_in_set(s[0], "ABCDEFGHIJKLMNOPQRSTUVWXYZ") &&
         char_in_set(s[1], "ABCDEFGHIJKLMNOPQRSTUVWXYZ") &&
         char_in_set(s[2], "0123456789") &&
         char_in_set(s[3], "0123456789") &&
         char_in_set(s[4], "0123456789") &&
         char_in_set(s[5], "0123456789");
}
```

Grammar: `^[A-Z]{2}[0-9]{4}$`.

## Properties worth proving

Reading the code with proof-colored glasses, five properties stand out:

1. **Memory safety** — `s[0..5]` are only read after `strlen(s) == 6`, so for any NUL-terminated `s` there are no out-of-bounds reads. Provable with [[CBMC]] with zero annotations.
2. **Exact length** — `is_valid_key(s) == true` implies `strlen(s) == 6`. A clean ACSL `ensures` clause.
3. **Grammar equivalence** — the C implementation accepts exactly `^[A-Z]{2}[0-9]{4}$`; check against an independent reference over all 6-byte inputs (fixed-size input makes CBMC's bound exact).
4. **The `char_in_set` NUL quirk** — `strchr(allowed, '\0')` matches the string terminator, so `char_in_set('\0', ...)` returns **true**. Harmless today _only_ because the length guard means no NUL can appear in `s[0..5]`; reorder the checks and it becomes a false-accept. A written spec makes that dependency explicit instead of leaving it implicit in the order of the `if`s.
5. **Output buffer contract** — a companion `write_key(char *buffer, size_t len, const char *key)` that copies via `snprintf` writes a NUL-terminated string of at most `len - 1` chars; provable from snprintf's contract or with CBMC on the harness.

## Where I'd start

Properties 1 + 2 with CBMC on a harness over a symbolic 7-byte buffer — an afternoon of work, machine-checked end to end, and the counterexample traces become regression tests in the project's host test suite for free.

Related: [[what-is-formal-verification|intro]], [[cbmc|CBMC]], [[frama-c-wp-acsl|Frama-C WP and ACSL]].
