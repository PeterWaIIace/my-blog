---
title: Formal Verification Tools for Python
---

Python makes formal verification harder, not easier: dynamic types, duck typing, and a huge runtime are a hostile environment for a proof. But "harder" splits into a practical ladder — and the same [[case-study-fixed-length-key-validator|fixed-length key validator]] works as a running example on every rung.

## The ladder

1. **Machine-checked types** — `mypy --strict` or [pyright](https://github.com/microsoft/pyright) prove type consistency across the codebase. Not full verification, but they eliminate whole bug classes and already run in CI.
2. **Property-based testing** — [Hypothesis](https://hypothesis.readthedocs.io/) generates hundreds of inputs and shrinks counterexamples. Still testing, but the closest Python gets to "all inputs" outside fixed domains.
3. **Symbolic checking** — [CrossHair](https://crosshair.readthedocs.io/) executes your functions with symbolic inputs, like [[CBMC]] does for C, and hunts for concrete counterexamples to your contracts.
4. **SMT encoding** — for fixed-size domains, encode the property in [Z3](https://github.com/Z3Prover/z3) and get an actual proof of equivalence with a reference.
5. **Deductive verification** — Nagini (a research prototype) translates annotated Python to Viper and proves contracts; [Dafny](https://dafny.org/) goes the other way: write verified code in Dafny and compile it _to_ Python.

## CrossHair: CBMC for Python

CrossHair is the closest Python gets to CBMC: you write ordinary functions plus contracts (plain `assert`s, or `icontract`/`deal` decorators), and `crosshair check` symbolically executes the code looking for inputs that break them:

```python
import string


def is_valid_key(s: str) -> bool:
    if len(s) != 6:
        return False
    return (
        s[0] in string.ascii_uppercase
        and s[1] in string.ascii_uppercase
        and all(c in string.digits for c in s[2:])
    )


def check_no_short_accepts(s: str) -> None:
    # CrossHair tries to falsify this for arbitrary s
    if is_valid_key(s):
        assert len(s) == 6
```

Run `crosshair check keys.py` and it either covers the property or prints a concrete failing input — which you can drop straight into your pytest suite as a regression test, the same trick as CBMC counterexample traces.

## Z3 for fixed-size domains

The key validator takes exactly 6 characters, so the input domain is finite — and that makes a real proof cheap. Encode the grammar as SMT constraints and ask Z3 whether the implementation and the spec can ever disagree:

```python
from z3 import Int, Solver, And, Not, unsat

c = [Int(f"c{i}") for i in range(6)]  # six symbolic key characters
upper = lambda ch: And(ord("A") <= ch, ch <= ord("Z"))
digit = lambda ch: And(ord("0") <= ch, ch <= ord("9"))

spec = And(upper(c[0]), upper(c[1]), *[digit(c[i]) for i in range(2, 6)])
impl = <BoolRef mirroring is_valid_key's logic over c[0..5]>

solver = Solver()
solver.add(Not(spec == impl))  # any 6-char input where spec and impl disagree?
assert solver.check() == unsat  # unsat => equivalent on ALL 6-char inputs
```

For unbounded inputs you'd need induction instead of enumeration — that's where the deductive tools come in.

## The honest limits

- CrossHair and Z3-style checking shine on pure functions over simple types (`str`, `int`, `bytes`, lists). Framework code, I/O, and global state have to be stubbed or excluded.
- mypy/pyright prove types, not behavior; Hypothesis tests, it doesn't prove. They stack nicely: types in CI, Hypothesis in CI, CrossHair/Z3 on the critical validators.
- Nagini is a research prototype — powerful, but not plug-and-play. Dafny-to-Python is the pragmatic "verified by construction" route: prove it in Dafny, emit Python.
- Runtime contract libraries (`icontract`, `deal`) check contracts on real executions — useful, but they test; they don't prove.

Related: [[what-is-formal-verification|intro]], [[cbmc|CBMC]], [[cpp-tools|C++ tools]], [[case-study-fixed-length-key-validator|the case study]].
