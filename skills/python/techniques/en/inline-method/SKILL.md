# TECHNIQUE: Inline Method — Python

## Source
Based on: https://refactoring.guru/inline-method

---

## 1. Problem

A method body is as obvious as its name, or the method is only used once and the indirection adds no value. Keeping a trivial wrapper around a single expression makes the code harder to follow, not easier.

---

## 2. Solution

Replace every call to the method with the method's body. Delete the method.

---

## 3. When to apply

- The method body is a single line and the name adds nothing beyond restating the code
- The method is called in only one place and the indirection hinders readability
- A sequence of short methods was created during refactoring and the structure is now over-fragmented
- The method was a delegation shim whose delegate has been inlined or removed

---

## 4. Refactoring steps

1. Check that the method is not overridden in a subclass
2. Find all call sites (use `grep` or your IDE's "Find Usages")
3. For each call site, replace the call with the method body (substituting parameters for arguments)
4. Delete the method
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class OrderService:
    def get_discount(self, order) -> float:
        return 0.1 if self._more_than_fifty_items(order) else 0.0

    def _more_than_fifty_items(self, order) -> bool:
        return order.item_count > 50
```

**AFTER — expected:**
```python
class OrderService:
    def get_discount(self, order) -> float:
        return 0.1 if order.item_count > 50 else 0.0
```

**Why this pattern:**
- `_more_than_fifty_items` added no clarity — the condition is readable on its own
- One fewer method to search for, test, and maintain

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a method that is called in multiple places**
```python
# Not accepted — inlining duplicates the condition in every call site
if self._more_than_fifty_items(order):  # called here and in three other places
    ...
```

**Mistake 2: Inlining a method that is overridden in a subclass**
```python
# Not accepted — inlining removes the polymorphic hook used by subclasses
class PremiumOrderService(OrderService):
    def _more_than_fifty_items(self, order) -> bool:
        return order.item_count > 30  # different threshold
```

**Mistake 3: Inlining a method whose name explains a non-obvious invariant**
```python
# Not accepted — the name explains domain meaning that the raw expression does not
def _is_eligible_for_free_shipping(self, order) -> bool:
    return order.total > 100
```

---

## 7. Benefits

- **Reduced indirection:** No need to jump to another method to understand a one-liner
- **Simpler codebase:** Removes methods that exist purely as leftover scaffolding
- **Cleaner API:** Fewer public/private methods reduce the cognitive load of understanding a class
