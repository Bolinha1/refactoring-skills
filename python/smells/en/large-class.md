# SKILL: Detecting and Refactoring Large Class — Python

## Source
Based on: https://refactoring.guru/smells/large-class

---

## 1. What is Large Class

A class that contains too many fields, methods, or lines of code.
When a class tries to do too many things, it accumulates responsibilities
that should be distributed.

**Why this happens:**
- Classes start small and grow as the program evolves
- It is mentally easier to add functionality to an existing class than to create a new one
- The accumulation is gradual — nobody notices until the class has become a monolith

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Class with more than 200 lines
- [ ] Class with more than 10 public methods
- [ ] Class with more than 5 instance attributes in `__init__`
- [ ] Class with clearly distinct responsibilities (SRP violation)
- [ ] Class that is modified for different reasons
- [ ] Many unrelated imports at the top of the file
- [ ] Class name is too generic (Manager, Processor, Handler, Utils)
- [ ] Attributes that are only used by a subset of the methods

---

## 3. Treatment techniques (in order of preference)

| Situation found                                    | Recommended technique   |
|----------------------------------------------------|-------------------------|
| Behaviors groupable into an autonomous unit        | Extract Class           |
| Specialized or rarely-used functionality           | Extract Subclass        |
| Clients use only part of the interface             | Extract Interface       |
| GUI class with mixed data and logic                | Duplicate Observed Data |

**Golden rule:** If you can describe the class using the conjunction "and"
(e.g., "this class validates orders **and** calculates freight **and** sends email"),
it has too many responsibilities.

---

## 4. Example

**BEFORE — not accepted:**
```python
class OrderService:
    def __init__(self, repository, email_service, stock_service, invoice_service):
        self.repository = repository
        self.email_service = email_service
        self.stock_service = stock_service
        self.invoice_service = invoice_service

    def create(self, order):
        if not order.items:
            raise ValueError("No items")
        if order.customer is None:
            raise ValueError("No customer")

        total = sum(i.price * i.quantity for i in order.items)
        order.total = total

        for item in order.items:
            self.stock_service.reserve(item.product_id, item.quantity)

        self.repository.save(order)

        invoice = self.invoice_service.generate(order)
        order.invoice = invoice

        self.email_service.send_confirmation(order.customer.email, order)

    def cancel(self, order): ...
    def recalculate_shipping(self, order): ...
    def apply_coupon(self, order, coupon): ...
    def find_by_customer(self, customer_id): ...
    def find_by_period(self, start, end): ...
    def generate_report(self, orders): ...
```

**AFTER — expected (Extract Class):**
```python
class OrderService:
    def __init__(self, repository, validator, calculator, stock_reservation, notification):
        self.repository = repository
        self.validator = validator
        self.calculator = calculator
        self.stock_reservation = stock_reservation
        self.notification = notification

    def create(self, order):
        self.validator.validate(order)
        order.total = self.calculator.calculate(order)
        self.stock_reservation.reserve(order)
        self.repository.save(order)
        self.notification.send_confirmation(order)


class OrderValidator:
    def validate(self, order):
        if not order.items:
            raise ValueError("No items")
        if order.customer is None:
            raise ValueError("No customer")


class TotalCalculator:
    def calculate(self, order):
        return sum(i.price * i.quantity for i in order.items)


class StockReservation:
    def __init__(self, stock_service):
        self.stock_service = stock_service

    def reserve(self, order):
        for item in order.items:
            self.stock_service.reserve(item.product_id, item.quantity)


class OrderNotification:
    def __init__(self, email_service, invoice_service):
        self.email_service = email_service
        self.invoice_service = invoice_service

    def send_confirmation(self, order):
        invoice = self.invoice_service.generate(order)
        order.invoice = invoice
        self.email_service.send_confirmation(order.customer.email, order)
```

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting a class without cohesion**
```python
# Not accepted — the extracted class still mixes responsibilities
class OrderHelper:
    def validate(self, order): ...
    def calculate_shipping(self, order): ...
    def generate_report(self, orders): ...
```

**Mistake 2: Creating anemic classes**
```python
# Not accepted — no real behavior, just delegates
class OrderValidator:
    def validate(self, order):
        return order.is_valid()
```

**Mistake 3: Generic names for extracted classes**
```python
# Not accepted
class OrderUtils: ...
class OrderManager: ...
class OrderHelper2: ...
```

---

## 6. Benefits

- **Cognitive load:** Developers don't need to memorize excessive attributes and methods
- **Duplicate elimination:** Splitting large classes frequently eliminates redundant code
- **Maintainability:** Each class with a single responsibility is easier to test and modify
- **Reusability:** Smaller, focused classes are easier to reuse in other contexts
