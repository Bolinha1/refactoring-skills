# SMELL: Data Class — Python

## Source
Based on: https://refactoring.guru/smells/data-class

---

## 1. What is it?

A class that contains only attributes and accessors — but no real behaviour. All the logic that operates on this data lives in other classes. The class is essentially a dumb data container and acts as a passive record rather than a responsible object.

---

## 2. Warning signs

- [ ] A class has only attributes and maybe `__init__` with no methods
- [ ] Business logic that should belong to the class is scattered across service or manager classes
- [ ] Other classes manipulate the data class's attributes directly
- [ ] The class is used only as a parameter bag passed between functions/methods
- [ ] The class is a plain dataclass (`@dataclass`) but all operations on it live elsewhere

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move behaviour from service classes into the data class, giving it responsibility |
| **Extract Class** | If the data class has grown large, split off a cohesive subset into its own class with behaviour |
| **Hide Method** | Use properties with only a getter (no setter) to protect attributes from external mutation |
| **Encapsulate Field** | Replace direct attribute access with properties as a first step toward adding validation or logic |

---

## 4. Example

**BEFORE — not accepted:**
```python
from dataclasses import dataclass, field
from typing import List

@dataclass
class Order:
    items: List['OrderItem'] = field(default_factory=list)
    status: str = 'PENDING'

# All logic lives in an external service
class OrderService:
    def calculate_total(self, order: Order) -> float:
        return sum(item.price * item.quantity for item in order.items)

    def can_be_cancelled(self, order: Order) -> bool:
        return order.status == 'PENDING'
```

**AFTER — expected:**
```python
class Order:
    def __init__(self, items: list['OrderItem']) -> None:
        self._items = list(items)
        self._status = 'PENDING'

    def total(self) -> float:
        return sum(item.price * item.quantity for item in self._items)

    def can_be_cancelled(self) -> bool:
        return self._status == 'PENDING'

    def cancel(self) -> None:
        if not self.can_be_cancelled():
            raise ValueError(f"Cannot cancel order in status: {self._status}")
        self._status = 'CANCELLED'
```

**Why this pattern:**
- `Order` now owns its business rules — callers ask the order itself
- The `_status` attribute is protected: transitions happen through domain methods

---

## 5. Negative examples — what NOT to do

**Mistake 1: Moving all service methods into the data class without discrimination**
```python
# Not accepted — the class becomes a Large Class; only move cohesive behaviour
class Order:
    def send_confirmation_email(self): ...  # unrelated to Order's domain
    def save_to_database(self): ...         # infrastructure concern
```

**Mistake 2: Keeping public attribute mutation after adding domain logic**
```python
# Not accepted — any caller can bypass the domain rule by setting order.status directly
order.status = 'CANCELLED'  # bypasses can_be_cancelled() check
```

---

## 6. Benefits

- **Encapsulation:** Business rules live next to the data they govern
- **Reduced duplication:** Logic that was copied across service classes is now in one place
- **Tell, don't ask:** Callers ask the object to do something instead of reading its attributes and deciding externally
