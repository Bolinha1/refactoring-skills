# SKILL: Detecting and Refactoring Primitive Obsession — Python

## Source
Based on: https://refactoring.guru/smells/primitive-obsession

---

## 1. What is Primitive Obsession

Overuse of basic types (`str`, `int`, `float`, `bool`, `dict`, `list`)
to represent domain concepts that deserve their own classes or dataclasses.

**Why this happens:**
- Creating a new class feels like overkill for something "simple"
- `dict` is convenient and quick to use
- The code grows and the dict keys become magic strings that are hard to track

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `str` representing tax ID, phone, ZIP code, email, product code
- [ ] `int` or `str` constant simulating a type (e.g., `ROLE_ADMIN = "admin"`)
- [ ] `dict` with magic string keys to structure domain data
- [ ] Multiple primitive parameters that always appear together (e.g., `amount: float, currency: str`)
- [ ] Validation of the same primitive repeated in multiple places

---

## 3. Treatment techniques (in order of preference)

| Situation found                                      | Recommended technique          |
|------------------------------------------------------|-------------------------------|
| Primitive with its own validation rules              | Replace Data Value with Object |
| Multiple primitives that travel together             | Introduce Parameter Object     |
| Primitive passed as a whole group                    | Preserve Whole Object          |
| `str`/`int` simulating an enumerated type            | Replace Type Code with Class   |
| `dict` or `list` as a data structure                 | Replace Array with Object      |

---

## 4. Example

**BEFORE — not accepted:**
```python
class Customer:
    def __init__(self, name: str, tax_id: str, phone: str, email: str):
        self.name = name
        self.tax_id = tax_id    # raw string, no centralized validation
        self.phone = phone
        self.email = email

class OrderService:
    def create(self, customer_tax_id: str, amount: float, currency: str, payment_type: str):
        # payment_type: "card", "bank_slip", "pix" — magic string
        if payment_type == "card":
            ...
        # tax ID validation duplicated in multiple places
        if not customer_tax_id or len(customer_tax_id) != 11:
            raise ValueError("Invalid tax ID")

    def get_address(self, data: dict):
        # magic keys in the dict
        street = data["street"]
        number = data["number"]
        city   = data["city"]
        zip_code = data["zip_code"]
```

**AFTER — expected:**
```python
from dataclasses import dataclass
from enum import Enum
import re


# Value Object for tax ID
class TaxId:
    def __init__(self, value: str):
        if not value or not re.fullmatch(r"\d{11}", value):
            raise ValueError("Invalid tax ID")
        self._value = value

    @property
    def value(self) -> str:
        return self._value

    def __eq__(self, other):
        return isinstance(other, TaxId) and self._value == other._value

    def __hash__(self):
        return hash(self._value)


# Value Object for Email
class Email:
    def __init__(self, address: str):
        if not address or "@" not in address:
            raise ValueError("Invalid email")
        self._address = address

    @property
    def address(self) -> str:
        return self._address


# Parameter Object for data that travels together
@dataclass(frozen=True)
class Money:
    amount: float
    currency: str


# Enum instead of magic string
class PaymentType(Enum):
    CARD      = "card"
    BANK_SLIP = "bank_slip"
    PIX       = "pix"


# Replace Array with Object
@dataclass
class Address:
    street: str
    number: str
    city: str
    zip_code: str


class Customer:
    def __init__(self, name: str, tax_id: TaxId, email: Email):
        self.name = name
        self.tax_id = tax_id
        self.email = email


class OrderService:
    def create(self, customer_tax_id: TaxId, amount: Money, payment_type: PaymentType):
        if payment_type == PaymentType.CARD:
            ...

    def get_address(self, address: Address):
        street = address.street
        city   = address.city
```

**Why this pattern:**
- Tax ID validation is centralized in `TaxId` — not repeated
- `PaymentType` is self-explanatory, no magic strings
- `Money` groups amount and currency with `frozen=True` ensuring immutability
- `Address` replaces the dict with typed, named fields

---

## 5. Negative examples — what NOT to do

**Mistake 1: `str` for everything**
```python
# Not accepted
def process(tax_id: str, status: str, payment_type: str):
    if status == "active" and payment_type == "pix":
        ...
```

**Mistake 2: String constants instead of Enum**
```python
# Not accepted
PAYMENT_CARD     = "card"
PAYMENT_BANK_SLIP = "bank_slip"
if payment_type == PAYMENT_CARD:
    ...
```

**Mistake 3: `dict` with magic keys**
```python
# Not accepted
address = {
    "street":   "Main St",
    "number":   "123",
    "city":     "New York",
    "zip_code": "10001",
}
```

---

## 6. Benefits

- **Flexibility:** Business rules are encapsulated in the correct object
- **Readability:** Parameters and fields express domain intent
- **Maintainability:** Centralized validation — change in one place, applies everywhere
- **Safety:** Strong typing facilitates static analysis with `mypy`
