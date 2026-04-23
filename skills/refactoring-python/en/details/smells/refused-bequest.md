# SMELL: Refused Bequest — Python

## Source
Based on: https://refactoring.guru/smells/refused-bequest

---

## 1. What is it?

A subclass inherits methods or data from its parent class but does not use or actively overrides them to raise exceptions. The child rejects part of what the parent gives it, which breaks the Liskov Substitution Principle: a subclass should be usable wherever its parent is expected.

---

## 2. Warning signs

- [ ] A subclass overrides a parent method only to raise `NotImplementedError` or similar
- [ ] Inherited attributes are never read or written by the subclass
- [ ] Tests that expect base-class behaviour fail when given the subclass
- [ ] The subclass uses only a small fraction of the parent's public API
- [ ] The relationship reads "is-a" only on paper — behaviourally it is not

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Replace Inheritance with Delegation** | The subclass wants some parent behaviour but not all — compose instead of extend |
| **Extract Superclass** | Pull only the shared behaviour into a new parent; leave the rest in sibling classes |
| **Push Down Method / Push Down Field** | Move unwanted inherited members down out of the parent so the subclass never receives them |

---

## 4. Example

**BEFORE — not accepted:**
```python
class Bird:
    def get_name(self) -> str:
        return self.name

    def fly(self) -> None:
        pass  # flap wings

    def land(self) -> None:
        pass  # touchdown


# Penguin IS-A Bird on paper, but cannot fly
class Penguin(Bird):
    def fly(self) -> None:
        raise NotImplementedError("Penguins cannot fly")


# Caller breaks at runtime
b: Bird = Penguin()
b.fly()  # raises!
```

**AFTER — expected:**
```python
from abc import ABC, abstractmethod
from typing import Protocol

class Bird(ABC):
    @abstractmethod
    def get_name(self) -> str: ...


class Flyable(Protocol):
    def fly(self) -> None: ...
    def land(self) -> None: ...


class Sparrow(Bird):
    def get_name(self) -> str:
        return "Sparrow"

    def fly(self) -> None:
        pass  # flap

    def land(self) -> None:
        pass  # touchdown


class Penguin(Bird):
    def get_name(self) -> str:
        return "Penguin"

    def swim(self) -> None:
        pass  # splash
```

**Why this pattern:**
- `Bird` now only contains behaviour shared by ALL birds
- `Flyable` is a structural Protocol — penguins simply don't implement it
- Code that calls `fly()` must accept `Flyable`, not just any `Bird`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the raise and documenting it**
```python
# Not accepted — LSP is still violated; callers cannot trust the contract
def fly(self) -> None:
    # Penguins don't fly — documented but still broken
    raise NotImplementedError
```

**Mistake 2: Returning a no-op instead of raising**
```python
# Not accepted — silent failures hide bugs; callers assume the action happened
def fly(self) -> None:
    pass  # do nothing
```

---

## 6. Benefits

- **LSP compliance:** Every subclass honours the full contract of its parent
- **Safer polymorphism:** Code that iterates a list of `Bird` objects won't crash
- **Cleaner hierarchy:** Capabilities become explicit protocols rather than hidden exceptions
