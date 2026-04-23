# SMELL: Inappropriate Intimacy — Python

## Source
Based on: https://refactoring.guru/smells/inappropriate-intimacy

---

## 1. What is it?

One class accesses the private attributes, internal lists, or implementation details of another class too freely. The two classes are tightly coupled in a way that makes it hard to change one without affecting the other.

---

## 2. Warning signs

- [ ] A class reads multiple attributes of another class to perform a calculation that should live in that other class
- [ ] A class directly mutates the list or internal state of another class
- [ ] Two classes reference each other in a bidirectional dependency
- [ ] A class knows the internal structure of another class (e.g., knows which attributes are None in which state)
- [ ] Repeated "train wrecks": `a.get_b().get_c().do_something()`

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move the method that accesses foreign internals into the class that owns those internals |
| **Move Field** | Move an attribute to the class that uses it most |
| **Hide Delegate** | Add a forwarding method so the client doesn't reach through an intermediary |
| **Extract Class** | If two classes are too intimate because they share responsibilities, separate those responsibilities cleanly |

---

## 4. Example

**BEFORE — not accepted:**
```python
class Order:
    def __init__(self, items: list, customer_email: str) -> None:
        self.items = items             # public — exposes internals
        self.customer_email = customer_email

# Report digs into Order's guts
class OrderReport:
    def generate(self, order: Order) -> str:
        total = sum(item.price * item.quantity for item in order.items)  # reaching in
        return f"Order for {order.customer_email} total: {total}"
```

**AFTER — expected:**
```python
class Order:
    def __init__(self, items: list, customer_email: str) -> None:
        self._items = items
        self._customer_email = customer_email

    def total(self) -> float:
        return sum(item.price * item.quantity for item in self._items)

    @property
    def customer_email(self) -> str:
        return self._customer_email

# Report only asks the Order for what it needs
class OrderReport:
    def generate(self, order: Order) -> str:
        return f"Order for {order.customer_email} total: {order.total()}"
```

**Why this pattern:**
- `OrderReport` no longer depends on the internal structure of `Order`
- Changing how `Order` stores items doesn't affect `OrderReport`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Using a generator expression but still pulling the list**
```python
# Not accepted — order.items still exposes the internals
total = sum(item.price * item.quantity for item in order.items)
```

**Mistake 2: Creating a dataclass copy with all Order fields**
```python
# Not accepted — an OrderData with all the same fields is just indirection; the coupling remains
```

---

## 6. Benefits

- **Lower coupling:** Classes can change their internals without affecting their clients
- **Better encapsulation:** The owning class controls how its data is computed and accessed
- **Easier testing:** Classes with fewer dependencies are simpler to test in isolation
