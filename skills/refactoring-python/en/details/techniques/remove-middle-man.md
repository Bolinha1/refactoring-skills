# TECHNIQUE: Remove Middle Man — Python

## Source
Based on: https://refactoring.guru/remove-middle-man

---

## 1. Problem

A class has too many simple delegation methods that do nothing except forward calls to another object. The class is a Middle Man — it exists only to pass messages through.

---

## 2. Solution

Remove the delegation methods. Let the client access the delegate directly.

---

## 3. When to apply

- The server class has grown a large number of one-liner delegating properties or methods
- The server class delegates so much that it has no real behavior of its own
- Adding a new feature to the delegate requires adding a new pass-through in the server
- The Middle Man smell has been identified in the server class

**Note:** This is the inverse of Hide Delegate. Apply Hide Delegate when you want encapsulation; apply Remove Middle Man when the encapsulation has become an obstacle.

---

## 4. Refactoring steps

1. Identify all delegation methods/properties in the middle man class
2. For each delegation, find its callers
3. Update each caller to access the delegate directly
4. Remove each delegation property/method from the middle man
5. If the middle man now has no remaining behavior, delete it or merge it into the caller
6. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Person:
    def __init__(self, department: "Department"):
        self._department = department

    # Pure pass-through properties — the class is a middle man
    @property
    def manager(self): return self._department.manager
    @property
    def employees(self): return self._department.employees
    @property
    def department_name(self): return self._department.name
    @property
    def department_budget(self): return self._department.budget
```

**AFTER — expected:**
```python
class Person:
    def __init__(self, department: "Department"):
        self.department = department  # expose the delegate directly
    # All pass-through properties removed


# Callers access Department directly where they need it
manager = person.department.manager
team = person.department.employees
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Removing the middle man but exposing internal objects that should be hidden**
```python
# Not accepted — if Department is an internal detail, exposing it breaks encapsulation
# Only remove the middle man if direct access is appropriate for the architecture
```

**Mistake 2: Removing delegation properties that have real logic in them**
```python
# Not accepted — this is not pure delegation; it has a guard
@property
def manager(self):
    if self._department is None:  # real logic — don't remove
        return None
    return self._department.manager
```

**Mistake 3: Applying Remove Middle Man when Hide Delegate is actually correct**
```python
# Not accepted — if clients should not know about Department at all,
# Hide Delegate is the right choice; Remove Middle Man goes in the other direction
```

---

## 7. Benefits

- **Simplicity:** Removes unnecessary indirection
- **Transparency:** Callers have a direct path to what they need
- **Maintainability:** Adding features to the delegate no longer requires touching the middle man
