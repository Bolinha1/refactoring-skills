# SKILL: Detecting and Refactoring Shotgun Surgery — Python

## Source
Based on: https://refactoring.guru/smells/shotgun-surgery

---

## 1. What is Shotgun Surgery

A single logical change requires modifying many different modules or classes at the same time. Making one conceptual change "fires" edits across the codebase like a shotgun blast — touching many files for what should be a localized fix.

The inverse of Divergent Change: many classes, one reason to change.

**Why this happens:**
- A responsibility was fragmented across many modules instead of living in one place
- Logic that should be centralized was duplicated or distributed for "flexibility"
- Cross-cutting concerns (logging, validation, notifications) were handled inline everywhere

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Adding a new feature forces you to edit 5+ unrelated modules/classes
- [ ] Renaming a concept requires touching dozens of files
- [ ] A single business rule is enforced in multiple places
- [ ] You always need to make the same change in several places simultaneously
- [ ] Test failures cascade across many test files for a single-concept change

---

## 3. Treatment techniques (in order of preference)

| Situation found                                                | Recommended technique   |
|----------------------------------------------------------------|-------------------------|
| Scattered behavior all relates to one concept                  | Move Method             |
| Multiple small classes all contributing to one concept         | Inline Class            |
| Cross-cutting concern repeated everywhere                      | Extract Class           |
| Duplicated business rule enforced in many places               | Move Method to one owner |

---

## 4. Example

**BEFORE — not accepted:**
```python
# Adding a "currency" concept requires changing ALL of these:

class Product:
    price: float  # must add currency field here

class Order:
    def get_total(self) -> float: ...  # must format with currency

class Invoice:
    def format_amount(self, amount: float) -> str:
        return f"${amount:.2f}"  # must use currency

class Receipt:
    def print_total(self, amount: float) -> None:
        print(amount)  # must use currency
```

**AFTER — expected:**
```python
from decimal import Decimal
from dataclasses import dataclass

@dataclass(frozen=True)
class Money:
    amount: Decimal
    currency: str

    def __add__(self, other: "Money") -> "Money":
        if self.currency != other.currency:
            raise ValueError("Currency mismatch")
        return Money(self.amount + other.amount, self.currency)

    def format(self) -> str:
        symbols = {"USD": "$", "BRL": "R$", "EUR": "€"}
        symbol = symbols.get(self.currency, self.currency)
        return f"{symbol}{self.amount:.2f}"


# Now Product, Order, Invoice, Receipt all use Money — one concept, one place
@dataclass
class Product:
    price: Money
```

**Why this pattern:**
- `Money` centralizes all currency behavior — adding a new currency format touches one class
- All callers benefit automatically

---

## 5. Negative examples — what NOT to do

**Mistake 1: Centralizing the data but not the behavior**
```python
# Not accepted — MoneyDto groups the fields but formatting/arithmetic stay scattered
@dataclass
class MoneyDto:
    amount: float
    currency_code: str
```

**Mistake 2: Using a utility module as the single point**
```python
# Not accepted — money_utils is a hidden smell; logic still fragmented among callers
def format_money(amount: float, currency: str) -> str: ...
```

**Mistake 3: Inline class too aggressively, creating a new god class**
```python
# Not accepted — inlining all small classes into one megaclass trades Shotgun Surgery
# for Divergent Change
```

---

## 6. Benefits

- **Locality:** One business concept = one class to change
- **Safety:** Changes are easier to reason about when blast radius is small
- **Discoverability:** All behavior for a concept is in one obvious place
