# SKILL: Detecting and Refactoring Data Clumps — Python

## Source
Based on: https://refactoring.guru/smells/data-clumps

---

## 1. What is Data Clumps

Different parts of the code contain identical groups of variables (a clump) — the same attributes in multiple classes, or the same parameters appearing together in many function signatures. If you removed one item from the clump and the rest stopped making sense, you have a data clump.

**Why this happens:**
- The relationship between the data items was never formalized as a dataclass or class
- Copy-paste propagated the group across unrelated places
- The concept existed informally in developers' heads but not in the code

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Three or more attributes that appear together in multiple classes
- [ ] The same 2–3 parameters consistently passed together to functions
- [ ] Variables like `start_date`/`end_date`, `latitude`/`longitude`, `street`/`city`/`zip` as separate primitives
- [ ] You must update the same group of attributes in several places for one logical change

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique         |
|----------------------------------------------------------|-------------------------------|
| Clump appears as attributes in a class                   | Extract Class                 |
| Clump appears in function parameter lists                | Introduce Parameter Object    |
| A function receives an object but uses only part of it   | Preserve Whole Object         |

**Key test:** delete one item from the clump. If the others lose meaning, the group deserves its own dataclass.

---

## 4. Example

**BEFORE — not accepted:**
```python
class Order:
    def __init__(self):
        self.street = ""
        self.city = ""
        self.zip_code = ""
        self.country = ""
        self.customer_name = ""
        self.customer_email = ""

class Invoice:
    def __init__(self):
        self.street = ""    # same clump again
        self.city = ""
        self.zip_code = ""
        self.country = ""

def ship(street: str, city: str, zip_code: str, country: str) -> None: ...
```

**AFTER — expected:**
```python
from dataclasses import dataclass

@dataclass
class Address:
    street: str
    city: str
    zip_code: str
    country: str

    def format(self) -> str:
        return f"{self.street}, {self.city} {self.zip_code}, {self.country}"


@dataclass
class Order:
    shipping_address: Address
    customer_name: str
    customer_email: str

@dataclass
class Invoice:
    billing_address: Address

def ship(destination: Address) -> None: ...
```

**Why this pattern:**
- `Address` is a named concept — its validation, formatting, and comparison now live in one place
- Every class that holds an address benefits from any improvement to `Address`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping the clump into a dict or generic container**
```python
# Not accepted — loses type safety and intent
address = {"street": "Main St", "city": "Springfield"}
```

**Mistake 2: Creating the dataclass but leaving the old separate attributes too**
```python
# Not accepted — now there are two representations of the same data
self.shipping_address = Address(...)
self.street = ""   # duplicate
self.city = ""     # duplicate
```

**Mistake 3: Grouping data with no cohesion just because they appear together**
```python
# Not accepted — CustomerPreferences bundles unrelated fields into one class
# just because they happened to be in the same function signature
```

---

## 6. Benefits

- **Single source of truth:** The clump's logic (validation, formatting) is centralized
- **Readability:** `order.shipping_address` is more expressive than four separate attributes
- **Extensibility:** Adding a new address field requires changing only the `Address` dataclass
