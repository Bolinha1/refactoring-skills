# SKILL: Detecting and Refactoring Switch Statements — Python

## Source
Based on: https://refactoring.guru/smells/switch-statements

---

## 1. What is Switch Statements

Complex if/elif chains (or match/case in Python 3.10+) that branch on the type or state of an object. The same switching logic tends to be duplicated across the codebase — when a new case is added, every branch must be found and updated.

**Why this happens:**
- Type-based behavior was implemented procedurally instead of using polymorphism
- A single class grew to handle multiple variants of a concept
- The concept of "type" was encoded as a string or integer rather than as a class hierarchy

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `if/elif` chain or `match/case` that branches on a type field, status, or string
- [ ] The same branching logic appears in more than one place
- [ ] Adding a new "type" requires searching for and updating every branch in the codebase
- [ ] Each branch does something very different from the others
- [ ] Branching on a type to decide which object to instantiate

---

## 3. Treatment techniques (in order of preference)

| Situation found                                           | Recommended technique                  |
|-----------------------------------------------------------|----------------------------------------|
| Branching on type to decide behavior                      | Replace Conditional with Polymorphism  |
| Branching on type to decide which class to create         | Replace Constructor with Factory Method |
| Only a few cases and adding new ones is rare              | Replace Type Code with Subclasses      |
| The type has mutable state that changes at runtime        | Replace Type Code with State/Strategy  |
| Branch is simple and used in only one place               | Leave it — not every branch is a smell |

---

## 4. Example

**BEFORE — not accepted:**
```python
def calculate_shipping_cost(order) -> float:
    if order.shipping_type == "standard":
        return order.weight * 1.5
    elif order.shipping_type == "express":
        return order.weight * 3.0 + 5.0
    elif order.shipping_type == "overnight":
        return order.weight * 5.0 + 20.0
    else:
        raise ValueError(f"Unknown shipping type: {order.shipping_type}")
```

**AFTER — expected:**
```python
from abc import ABC, abstractmethod

class ShippingStrategy(ABC):
    @abstractmethod
    def calculate_cost(self, weight: float) -> float: ...

class StandardShipping(ShippingStrategy):
    def calculate_cost(self, weight: float) -> float:
        return weight * 1.5

class ExpressShipping(ShippingStrategy):
    def calculate_cost(self, weight: float) -> float:
        return weight * 3.0 + 5.0

class OvernightShipping(ShippingStrategy):
    def calculate_cost(self, weight: float) -> float:
        return weight * 5.0 + 20.0


class Order:
    def __init__(self, weight: float, shipping_strategy: ShippingStrategy):
        self.weight = weight
        self._shipping_strategy = shipping_strategy

    def calculate_shipping_cost(self) -> float:
        return self._shipping_strategy.calculate_cost(self.weight)
```

**Why this pattern:**
- Adding a new shipping type requires only a new class, not a search-and-update in every branch
- Each strategy is independently testable and cohesive

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the branch with a dict dispatch but keeping the logic inline**
```python
# Not accepted — the logic is still scattered; dict dispatch is only good for simple lookups
handlers = {
    "standard": lambda w: w * 1.5,
    "express": lambda w: w * 3.0 + 5.0,
}
```

**Mistake 2: Centralizing into a single factory without polymorphism**
```python
# Not accepted — the branch still exists, just moved
def create_calculator(shipping_type: str):
    if shipping_type == "standard": ...
    elif shipping_type == "express": ...
```

**Mistake 3: Applying polymorphism to a branch that will never grow**
```python
# Not accepted — creating 2 subclasses for a boolean toggle is over-engineering
if order.is_priority:
    return priority_price
return standard_price
```

---

## 6. Benefits

- **Open/Closed:** Adding a new variant requires a new class, not editing existing ones
- **Locality:** Each variant's behavior is contained in one place
- **Testability:** Each polymorphic variant can be tested in isolation
