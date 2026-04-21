# SKILL: Detecting and Refactoring Dead Code — Python

## Source
Based on: https://refactoring.guru/smells/dead-code

---

## 1. What is Dead Code

Variables, parameters, attributes, functions, methods, or entire classes that are no longer used anywhere. Dead code clutters the codebase, misleads future readers into thinking it is relevant, and must still be read and understood even though it does nothing.

**Why this happens:**
- Requirements changed but the old implementation was not removed
- A feature was disabled without deleting the supporting code
- Refactoring produced unreachable code paths that were never cleaned up
- Fear of deletion: "we might need it later"

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `flake8`, `pyflakes`, or IDE highlights an unused import or variable
- [ ] A function is prefixed with `_` (private) and has no callers in the module
- [ ] Code inside an `if` branch can never execute (impossible condition)
- [ ] A class has no instantiations anywhere in the project
- [ ] Commented-out code blocks that have been there for more than one sprint
- [ ] A parameter is never read inside the function

---

## 3. Treatment techniques (in order of preference)

| Situation found                               | Recommended technique        |
|-----------------------------------------------|------------------------------|
| Unused function, attribute, or variable       | Delete it                    |
| Unused parameter                              | Remove Parameter             |
| Entire unused class or module                 | Delete it                    |
| Commented-out code block                      | Delete it (git has the history) |
| Unreachable code path                         | Delete it                    |

**Rule:** `git` is your safety net. Delete confidently — the code is not gone, just archived.

---

## 4. Example

**BEFORE — not accepted:**
```python
import csv           # never used since migration to pandas
import json          # never used

class CustomerService:
    def __init__(self):
        self.legacy_system_id = None  # never set or read since v2 migration

    def find_by_id(self, customer_id: int): ...

    # Old search before ElasticSearch was integrated
    # def _linear_search(self, name: str):
    #     return next((c for c in self.all_customers if c.name == name), None)

    def _sync_with_legacy_system(self):
        # TODO: remove after migration complete (migration was done 18 months ago)
        pass

    def notify_customer(self, customer, format: str, legacy: bool = False):
        if legacy:
            # legacy path — removed in v2, this branch is never True
            self._send_fax(customer)
        else:
            self._send_email(customer)
```

**AFTER — expected:**
```python
class CustomerService:
    def find_by_id(self, customer_id: int): ...

    def notify_customer(self, customer) -> None:
        self._send_email(customer)
```

**Why this pattern:**
- Every remaining line is load-bearing; readers can trust nothing is orphaned
- Smaller classes are faster to understand, test, and change

---

## 5. Negative examples — what NOT to do

**Mistake 1: Commenting out code instead of deleting it**
```python
# Not accepted — commented-out code is dead code in disguise
# def old_method(self): ...
```

**Mistake 2: Keeping dead code "just in case"**
```python
# Not accepted — git stores the history; the fear of deletion is unfounded
def _migrate_to_legacy_format(self): ...  # "might need later"
```

**Mistake 3: Using a deprecation warning instead of deleting**
```python
# Not accepted — keeps the dead code visible and confusing
import warnings
def old_notify(self, customer):
    warnings.warn("Use notify_customer instead", DeprecationWarning)
    ...
```

---

## 6. Benefits

- **Signal-to-noise:** Every remaining line is relevant — readers trust the codebase
- **Import speed:** Fewer imports and modules means faster startup
- **Clarity:** Dead code misleads — its absence removes false trails
