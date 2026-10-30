---
title: Python classes, ABCs, and metaclasses
tags:
  - python
  - programming
---

Python lets us define interfaces with ordinary base classes, abstract base classes, or more specialized metaclass machinery. The important differences are what each mechanism controls and when it catches mistakes.

## An interface without ABC

A regular base class can describe the methods its subclasses should provide:

```python
class BaseFunc:
    def evoke(self, data):
        raise NotImplementedError


class MySolution(BaseFunc):
    pass


solution = MySolution()  # Allowed
solution.evoke([])       # Raises NotImplementedError
```

This works as a convention, but Python allows us to instantiate an incomplete subclass. We discover the missing implementation only when we call the method.

## What ABC adds

`ABC` means **Abstract Base Class**. Together with `@abstractmethod`, it prevents us from instantiating a class while required methods remain abstract:

```python
from abc import ABC, abstractmethod


class BaseFunc(ABC):
    @abstractmethod
    def evoke(self, data):
        pass


class IncompleteSolution(BaseFunc):
    pass


# IncompleteSolution() would raise TypeError immediately.


class MySolution(BaseFunc):
    def evoke(self, data):
        return [1, 2, 3]


solution = MySolution()  # Allowed
```

The difference is **enforcement at instantiation time**, rather than failure when a missing method is called. Defining an incomplete subclass is still allowed; creating its instances is not.

An ABC can also contain ordinary methods, shared helpers, and abstract methods with reusable implementations. However, abstract-method enforcement does not check parameter signatures, return types, or whether the implementation is correct.

## What is a metaclass?

A class creates instances. A **metaclass creates classes**.

Python's usual metaclass is `type`:

```python
class Solution:
    pass


solution = Solution()

assert type(solution) is Solution
assert type(Solution) is type
```

When Python executes a class statement, its metaclass participates in building the resulting class object. A custom metaclass can inspect or modify that process:

```python
class RequireVersion(type):
    def __new__(mcls, name, bases, namespace):
        if "version" not in namespace:
            raise TypeError(f"{name} must declare a version")

        return super().__new__(mcls, name, bases, namespace)


class Solution(metaclass=RequireVersion):
    version = "1.0"
```

Here, validation happens **when the class is defined**, before we create an instance. This particular check requires `version` directly in the class body; an inherited attribute does not satisfy it.

Custom metaclasses can:

- Register classes automatically.
- Validate class declarations.
- Add attributes or wrap methods.
- Prepare the namespace before the class body executes, using `__prepare__`.
- Intercept instance creation, using the metaclass's `__call__`.

A custom metaclass derived from `type` does **not** automatically enforce abstract methods.

## ABC versus ABCMeta

`ABCMeta` is the metaclass that implements abstract-method enforcement. `ABC` is a convenience base class that uses it.

These two definitions provide the same abstract-method enforcement:

```python
from abc import ABC, ABCMeta, abstractmethod


class First(ABC):
    @abstractmethod
    def run(self):
        pass


class Second(metaclass=ABCMeta):
    @abstractmethod
    def run(self):
        pass
```

Neither `First()` nor `Second()` can be instantiated. A subclass must implement `run()` first.

Conceptually, `ABC` is simply a base class declared with `metaclass=ABCMeta`. For ordinary abstract base classes, inheriting from `ABC` is the convenient option.

To combine custom metaclass behavior with abstract-method enforcement, derive the metaclass from `ABCMeta`:

```python
class MyMeta(ABCMeta):
    def __new__(mcls, name, bases, namespace):
        cls = super().__new__(mcls, name, bases, namespace)
        cls.description = f"Solution class: {name}"
        return cls


class BaseSolution(metaclass=MyMeta):
    @abstractmethod
    def evoke(self, data):
        pass
```

`BaseSolution` receives the custom `description` attribute and remains abstract. Calling the superclass implementation preserves `ABCMeta`'s machinery.

## Can an ABC register or validate subclasses too?

Yes. Many tasks that sound like metaclass work can be handled by `__init_subclass__`, a hook available on ordinary base classes as well as ABCs:

```python
class BaseSolution(ABC):
    registry = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        BaseSolution.registry[cls.__name__] = cls

    @abstractmethod
    def evoke(self, data):
        pass


class RadarSolution(BaseSolution):
    def evoke(self, data):
        return [1, 2, 3]


assert BaseSolution.registry["RadarSolution"] is RadarSolution
```

Here, `ABC` enforces abstract methods, while `__init_subclass__` registers subclasses as they are defined. This simple registry includes abstract subclasses too; it is not a registry of only instantiable solutions.

The same hook can validate subclass attributes or modify methods. We do not need a custom metaclass just to do those things.

## Which tool should I use?

| Need                                                                         | Typical choice                   |
| ---------------------------------------------------------------------------- | -------------------------------- |
| Shared implementation or an informal interface                               | Regular base class               |
| Prevent instantiation until required methods are implemented                 | `ABC` and `@abstractmethod`      |
| Register or validate subclasses                                              | `__init_subclass__`              |
| Modify an individual class explicitly                                        | Class decorator                  |
| Control namespace preparation or the class-level instance-creation machinery | Custom metaclass                 |
| Custom metaclass behavior plus abstract-method enforcement                   | Metaclass derived from `ABCMeta` |

The key relationship is: **`ABC` uses `ABCMeta`; `ABCMeta` is a specialized metaclass; metaclasses in general can control much more than abstract methods.**
