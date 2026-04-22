# SKILL: Detecting and Refactoring Feature Envy — Python

## Source
Based on: https://refactoring.guru/smells/feature-envy

---

## 1. What is Feature Envy

A method accesses the data of another object more than it accesses the data of its own class. The method is "envious" of the other class — it seems to want to live there instead.

**Why this happens:**
- Fields were moved to a new class but the methods that use them were not
- Business logic was placed in a utility or service class that works on a domain object's internals
- A method was gradually extended to rely more and more on another object's attributes

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A method accesses several attributes of another object in sequence
- [ ] A method receives an object as parameter and uses mostly its fields/methods
- [ ] A method ignores `self` entirely and only manipulates another object's state
- [ ] The method's name naturally belongs to the other class ("calculate_order_total" in a ReportService)

---

## 3. Treatment techniques (in order of preference)

| Situation found                                         | Recommended technique             |
|---------------------------------------------------------|-----------------------------------|
| The whole method belongs in the other class             | Move Method                       |
| Only part of the method envies another class            | Extract Method, then Move Method  |
| The method uses data from several classes               | Move to the class with most data  |
| The method is a calculation that belongs on the model   | Move Method to domain object      |

---

## 4. Example

**BEFORE — not accepted:**
```python
class InvoicePrinter:
    def calculate_invoice_total(self, invoice) -> float:
        subtotal = sum(item.price * item.quantity for item in invoice.items)
        tax = subtotal * invoice.tax_rate
        discount = subtotal * invoice.discount_rate if invoice.has_discount else 0
        return subtotal + tax - discount
```

**AFTER — expected:**
```python
class Invoice:
    def calculate_total(self) -> float:
        subtotal = sum(item.price * item.quantity for item in self.items)
        tax = subtotal * self.tax_rate
        discount = subtotal * self.discount_rate if self.has_discount else 0
        return subtotal + tax - discount


class InvoicePrinter:
    def print(self, invoice: Invoice) -> None:
        total = invoice.calculate_total()  # delegate to owner
        # ... print logic
```

**Why this pattern:**
- `calculate_total` naturally belongs to `Invoice` — it only uses Invoice data
- `InvoicePrinter` is freed from knowing Invoice internals

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the envious method as a thin wrapper**
```python
# Not accepted — just a wrapper; delete the envious method entirely
def calculate_invoice_total(self, invoice) -> float:
    return invoice.calculate_total()
```

**Mistake 2: Moving the method but keeping raw field parameters**
```python
# Not accepted — moved but still receiving raw field values
def calculate_total(self, subtotal: float, tax_rate: float, discount_rate: float, has_discount: bool) -> float: ...
```

**Mistake 3: Moving a method that uses data from many classes equally**
```python
# Not accepted — if the method uses Order, Customer, and Coupon equally,
# moving it to any one class just shifts the envy; Extract Class is better
```

---

## 6. Benefits

- **Cohesion:** Each class owns the behavior that operates on its own data
- **Encapsulation:** The domain object exposes intent, not raw attributes
- **Maintainability:** Business rules are colocated with the data they govern
