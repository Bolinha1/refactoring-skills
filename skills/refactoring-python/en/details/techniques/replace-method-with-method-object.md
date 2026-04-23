# TECHNIQUE: Replace Method with Method Object — Python

## Source
Based on: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problem

A method is so large and complex that extracting sub-methods is awkward because they would all share many local variables. The local variables are too intertwined to be easily passed as parameters.

---

## 2. Solution

Turn the method into a separate class. Each local variable becomes an instance attribute of that class. The method body becomes the main `compute()` method of the class, which now can be freely split into sub-methods without parameter passing.

---

## 3. When to apply

- The method has many local variables that are used across multiple logical phases
- Extract Method would require passing too many parameters between the new methods
- The algorithm is complex enough to deserve its own abstraction
- You want to be able to test the algorithm in isolation or subclass parts of it

---

## 4. Refactoring steps

1. Create a new class named after the method
2. Add `__init__` that accepts the source object and all parameters of the original method
3. Store each parameter and local variable as an instance attribute
4. Copy the original method body into a `compute()` method; local variable references become `self.` attribute references
5. Replace the original method body with: `return MethodNameObject(self, param1).compute()`
6. Apply Extract Method freely on `compute()` — no parameters needed since shared state lives in `self`
7. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Order:
    def price(self) -> float:
        primary_base_price = self.quantity * self._primary_rate()
        secondary_base_price = primary_base_price * 0.7
        tertiary_base_price = max(primary_base_price, secondary_base_price)
        return tertiary_base_price - self._discount(primary_base_price, secondary_base_price)
```

**AFTER — expected:**
```python
class Order:
    def price(self) -> float:
        return PriceCalculator(self).compute()


class PriceCalculator:
    def __init__(self, order: "Order") -> None:
        self.order = order
        self.primary_base_price = 0.0
        self.secondary_base_price = 0.0
        self.tertiary_base_price = 0.0

    def compute(self) -> float:
        self.primary_base_price = self.order.quantity * self.order._primary_rate()
        self.secondary_base_price = self.primary_base_price * 0.7
        self.tertiary_base_price = max(self.primary_base_price, self.secondary_base_price)
        return self.tertiary_base_price - self._discount()

    def _discount(self) -> float:
        if self.order.is_premium:
            return self.primary_base_price * 0.1
        return self.secondary_base_price * 0.05
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Applying this when Extract Method would suffice**
```python
# Not accepted — if the method only has 2–3 local variables,
# Extract Method with explicit parameters is simpler
```

**Mistake 2: Naming the class vaguely**
```python
# Not accepted — Helper, Processor, Calculator are too generic
class OrderHelper: ...
# Preferred: name it after the specific algorithm
class OrderPriceCalculator: ...
```

---

## 7. Benefits

- **Enables Extract Method:** Sub-methods can share state via `self` without passing parameters
- **Testability:** The calculation object can be instantiated and tested independently
- **Extensibility:** The method object class can be subclassed to vary parts of the algorithm
