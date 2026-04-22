# SMELL: Incomplete Library Class — Python

## Source
Based on: https://refactoring.guru/smells/incomplete-library-class

---

## 1. What is it?

A library or third-party class lacks a method you need, and because you cannot (or should not) modify the library, you end up placing the missing behaviour in a utility function, a static helper, or a subclass. The logic that belongs to the library leaks into your own code.

---

## 2. Warning signs

- [ ] A utility module contains functions that only exist because a library class is missing them
- [ ] You subclass a library class just to add one or two missing methods
- [ ] Helper functions receive a library object as their only parameter
- [ ] The same "missing" operations are re-implemented in multiple modules
- [ ] Comments like "should be in library X" appear next to helper functions

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Introduce Foreign Method** | Add the missing operation as a function in your own module, taking the library object as the first parameter |
| **Introduce Local Extension** | Create a wrapper class around the library class that adds all missing methods in one place |

---

## 4. Example

**BEFORE — not accepted:**
```python
from datetime import date

# Missing helper scattered as a standalone function
def is_weekend(d: date) -> bool:
    return d.weekday() >= 5

def next_working_day(d: date) -> date:
    next_day = d.replace(day=d.day + 1)  # simplified
    from datetime import timedelta
    candidate = d + timedelta(days=1)
    while is_weekend(candidate):
        candidate += timedelta(days=1)
    return candidate

# Usage is awkward
next_day = next_working_day(date.today())
```

**AFTER — expected:**
```python
from datetime import date, timedelta

# Introduce Local Extension: wrap date with the missing behaviour
class BusinessDate:
    def __init__(self, d: date) -> None:
        self._date = d

    @classmethod
    def today(cls) -> 'BusinessDate':
        return cls(date.today())

    def is_weekend(self) -> bool:
        return self._date.weekday() >= 5

    def next_working_day(self) -> 'BusinessDate':
        candidate = BusinessDate(self._date + timedelta(days=1))
        while candidate.is_weekend():
            candidate = BusinessDate(candidate._date + timedelta(days=1))
        return candidate

    def to_date(self) -> date:
        return self._date

# Usage reads naturally
next_day = BusinessDate.today().next_working_day()
```

**Why this pattern:**
- All business-date logic lives in `BusinessDate` — no scattered utility functions
- Adding more missing behaviour has one obvious place

---

## 5. Negative examples — what NOT to do

**Mistake 1: Duplicating the missing function across multiple utility modules**
```python
# Not accepted — order_utils.is_weekend(), report_utils.is_weekend(), etc.
# creates divergence when the rule needs to change
```

**Mistake 2: Monkey-patching the library class**
```python
# Not accepted — modifying third-party classes at runtime causes subtle bugs
# and makes the code hard to understand
date.is_weekend = lambda self: self.weekday() >= 5  # monkey-patch
```

---

## 6. Benefits

- **Single source of truth:** Missing behaviour lives in one extension class
- **Readable call sites:** The extension reads like native methods on the original class
- **Isolated from library changes:** Wrapping is safer than monkey-patching when the library evolves
