# TECHNIQUE: Substitute Algorithm — Python

## Source
Based on: https://refactoring.guru/substitute-algorithm

---

## 1. Problem

You want to replace an existing algorithm with a cleaner, simpler, or more efficient one. The algorithm works correctly but is hard to understand, hard to extend, or can now be replaced with a Python built-in or standard library function.

---

## 2. Solution

Replace the body of the method implementing the old algorithm with the new algorithm. Keep the method signature the same so callers are unaffected.

---

## 3. When to apply

- A simpler algorithm exists that produces the same results
- A Python built-in (`any`, `next`, set operations, list comprehensions) now covers what you implemented manually
- The current algorithm is difficult to extend incrementally — it is easier to rewrite than to modify
- The algorithm has already been simplified as much as possible but is still unclear

---

## 4. Refactoring steps

1. Ensure that the existing algorithm has thorough tests — you need them to verify the replacement
2. Write the new algorithm in a separate method (or on a branch)
3. Run the tests against the new algorithm; fix any failures
4. Once tests pass, replace the old algorithm body with the new one
5. Run the tests one final time
6. Delete any helper code that was only needed by the old algorithm

---

## 5. Example

**BEFORE — not accepted:**
```python
def find_person(self, people: list[str]) -> str:
    for person in people:
        if person == "Don":
            return "Don"
        if person == "John":
            return "John"
        if person == "Kent":
            return "Kent"
    return ""
```

**AFTER — expected:**
```python
def find_person(self, people: list[str]) -> str:
    candidates = {"Don", "John", "Kent"}
    return next((p for p in people if p in candidates), "")
```

**Why this pattern:**
- A set lookup (`in candidates`) is O(1) vs. O(n) string comparisons
- Adding a new candidate requires only one change (add to the set), not a new `if` branch
- `next(..., "")` expresses the "first match or empty string" intent directly

---

## 6. Negative examples — what NOT to do

**Mistake 1: Substituting without a test suite**
```python
# Not accepted — if you have no tests, you cannot verify the new algorithm is equivalent;
# write the tests first before replacing anything
```

**Mistake 2: Incrementally patching the old algorithm instead of replacing it**
```python
# Not accepted — if the algorithm is fundamentally flawed, patches make it worse;
# substitute it entirely
```

---

## 7. Benefits

- **Simplicity:** The new algorithm is easier to read and maintain
- **Extensibility:** Well-structured algorithms are easier to modify
- **Pythonic:** Uses Python idioms and built-ins instead of verbose imperative loops
