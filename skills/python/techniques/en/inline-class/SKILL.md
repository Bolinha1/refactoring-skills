# TECHNIQUE: Inline Class — Python

## Source
Based on: https://refactoring.guru/inline-class

---

## 1. Problem

A class no longer does enough to justify its existence. It has too few responsibilities and is just an unnecessary indirection.

---

## 2. Solution

Move all features of the class into another class, then delete it.

---

## 3. When to apply

- After a refactoring, one class was left with only one or two trivial methods
- A class is used in only one place and adds no real abstraction
- A class that was split too aggressively needs to be consolidated
- The class is a thin wrapper that just delegates everything to another class

---

## 4. Refactoring steps

1. Identify the class to inline and the target class (where features will move)
2. For each public feature in the class to inline:
   - Copy it into the target class
   - Update the target class to delegate to the inline class (safe transition)
3. Update all callers to use the target class directly
4. Remove the delegation from the target class (methods now work natively)
5. Delete the inlined class
6. Run tests after each step

---

## 5. Example

**BEFORE — not accepted:**
```python
class TelephoneNumber:
    def __init__(self, number: str):
        self.number = number

    def get_number(self) -> str:
        return self.number


class Person:
    def __init__(self, name: str, telephone: TelephoneNumber):
        self.name = name
        self._telephone = telephone

    def get_telephone_number(self) -> str:
        return self._telephone.get_number()  # TelephoneNumber is just a wrapper
```

**AFTER — expected:**
```python
class Person:
    def __init__(self, name: str, telephone_number: str):
        self.name = name
        self.telephone_number = telephone_number  # inlined directly

    def get_telephone_number(self) -> str:
        return self.telephone_number
```

**Use case: consolidating Shotgun Surgery**
```python
# BEFORE — four tiny service classes split too aggressively
class OrderFinder:
    def find(self, order_id: int): ...

class OrderSaver:
    def save(self, order): ...

class OrderUpdater:
    def update(self, order): ...

class OrderDeleter:
    def delete(self, order_id: int): ...

# AFTER — inline all into a single coherent repository
class OrderRepository:
    def find(self, order_id: int): ...
    def save(self, order): ...
    def update(self, order): ...
    def delete(self, order_id: int): ...
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a class that still has a meaningful concept**
```python
# Not accepted — Money has behavior (arithmetic, formatting) that belongs together
# Do not inline Money into Order just because it has few attributes
```

**Mistake 2: Inlining without removing the original class**
```python
# Not accepted — both Person.telephone_number and TelephoneNumber exist simultaneously
class Person:
    def __init__(self):
        self.telephone_number = ""      # inlined copy
        self._telephone = TelephoneNumber(...)  # still here — creates confusion
```

**Mistake 3: Inlining a class used in many places**
```python
# Not accepted — if TelephoneNumber is used by Person, Employee, and Contact,
# inlining would require duplicating the logic in three places
```

---

## 7. Benefits

- **Simplicity:** Removes unnecessary indirection from the codebase
- **Readability:** Fewer classes to navigate when the abstraction adds no value
- **Preparation:** Often precedes Extract Class — inline first, then re-extract correctly
