# TECHNIQUE: Introduce Parameter Object — Python

## Source
Based on: https://refactoring.guru/introduce-parameter-object

---

## 1. Problem

Several parameters that naturally belong together are always passed as a group to the same functions. The group represents a concept not yet formalized in the codebase.

---

## 2. Solution

Replace the group of parameters with a single dataclass (or class) that represents the concept. The object becomes a first-class citizen in the domain model.

---

## 3. When to apply

- Three or more parameters that always appear together in function signatures
- The same group appears in multiple functions across the codebase
- The group represents a recognizable domain concept (date range, address, coordinates)
- You want to add behavior (validation, formatting) to the group of data

---

## 4. Refactoring steps

1. Create a new dataclass or class to represent the parameter group
2. Add the parameters as fields — use `frozen=True` if the object represents a value
3. Add a `__post_init__` or `__init__` to validate invariants
4. Update each function that takes the parameter group to accept the new class instead
5. Update all call sites to construct the new object and pass it
6. Look for logic in the callers that could move into the new class as a method
7. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def find_sales_by_range(start_date: date, end_date: date) -> list: ...
def sum_revenue_by_range(start_date: date, end_date: date) -> float: ...
def invoices_in_range(start_date: date, end_date: date) -> list: ...
```

**AFTER — expected:**
```python
from dataclasses import dataclass
from datetime import date

@dataclass(frozen=True)
class DateRange:
    start: date
    end: date

    def __post_init__(self):
        if self.end < self.start:
            raise ValueError("end must be after start")

    def includes(self, d: date) -> bool:
        return self.start <= d <= self.end


def find_sales_by_range(date_range: DateRange) -> list: ...
def sum_revenue_by_range(date_range: DateRange) -> float: ...
def invoices_in_range(date_range: DateRange) -> list: ...
```

**Why this pattern:**
- `DateRange` validates its own invariant once — callers cannot pass an invalid range
- The `includes()` method captures query logic previously duplicated in every caller

---

## 6. Negative examples — what NOT to do

**Mistake 1: Creating the object but not moving behavior into it**
```python
# Not accepted — DateRange is still just a bag of two dates with no behavior
@dataclass
class DateRange:
    start: date
    end: date
```

**Mistake 2: Making the object mutable when it represents a value**
```python
# Not accepted — a mutable DateRange can be changed by callers after passing
class DateRange:
    def set_start(self, d: date): self.start = d  # bad — values should be immutable
```

**Mistake 3: Grouping unrelated parameters just to reduce count**
```python
# Not accepted — RequestContext bundles customer_id, session_id, and locale
# for no reason other than there are three of them
@dataclass
class RequestContext:
    customer_id: int
    session_id: str
    locale: str
```

---

## 7. Benefits

- **Expressiveness:** The parameter group gets a meaningful name in the domain
- **Validation:** The object validates its own invariants once, at construction time
- **Behavior migration:** Logic that works on the group can move inside the object
