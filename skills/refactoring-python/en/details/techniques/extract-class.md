# TECHNIQUE: Extract Class — Python

## Source
Based on: https://refactoring.guru/extract-class

---

## 1. Problem

One class does the work of two. A subset of its attributes and methods forms a coherent concept that would make sense as its own class.

---

## 2. Solution

Create a new class (or dataclass) and move the attributes and methods that belong to that concept into it. Replace the original attributes with a reference to the new class.

---

## 3. When to apply

- The class has attributes that are always used together (Data Clumps)
- The class has methods that only touch a subset of its attributes
- The class has grown to handle more than one distinct responsibility (Divergent Change)
- A subset of the class could be reused independently in another context
- The class name cannot describe what it does without using "and"

---

## 4. Refactoring steps

1. Identify the group of attributes and methods to extract — they must form a coherent concept
2. Create a new class with a name that expresses that concept
3. Create an instance of the new class as an attribute in the original class
4. Move the identified attributes to the new class
5. Move the identified methods to the new class
6. Decide on exposure:
   - Export the class from the module if it can stand alone or be reused
   - Keep it module-private (prefixed `_`) if it is an internal detail
7. Run tests after each move

---

## 5. Example

**BEFORE — not accepted:**
```python
class Person:
    def __init__(self, name: str, office_area_code: str, office_number: str):
        self.name = name
        self.office_area_code = office_area_code
        self.office_number = office_number

    def get_telephone_number(self) -> str:
        return f"({self.office_area_code}) {self.office_number}"
```

**AFTER — expected:**
```python
from dataclasses import dataclass

@dataclass
class TelephoneNumber:
    area_code: str
    number: str

    def format(self) -> str:
        return f"({self.area_code}) {self.number}"


class Person:
    def __init__(self, name: str, office_telephone: TelephoneNumber):
        self.name = name
        self.office_telephone = office_telephone

    def get_telephone_number(self) -> str:
        return self.office_telephone.format()
```

**Variant — extracting a data clump:**
```python
# BEFORE — address attributes scattered across Person and Invoice
class Person:
    def __init__(self):
        self.street = ""
        self.city = ""
        self.zip_code = ""

# AFTER — Address is an independent, reusable concept
@dataclass
class Address:
    street: str
    city: str
    zip_code: str

    def format(self) -> str:
        return f"{self.street}, {self.city} {self.zip_code}"

class Person:
    def __init__(self, home_address: Address):
        self.home_address = home_address
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting a class that has no cohesion**
```python
# Not accepted — PersonData groups attributes that have no natural relationship
@dataclass
class PersonData:
    name: str
    office_code: str
    salary: float
    hire_date: date
```

**Mistake 2: Moving methods without moving the attributes they need**
```python
# Not accepted — TelephoneNumber.format() still receives raw fields from Person
class TelephoneNumber:
    def format(self, area_code: str, number: str) -> str: ...  # still coupled
```

**Mistake 3: Extracting but leaving the original attributes in place**
```python
# Not accepted — now there are two sources of truth for telephone data
class Person:
    def __init__(self):
        self.telephone = TelephoneNumber(...)  # new
        self.office_area_code = ""             # old — should be deleted
        self.office_number = ""               # old — should be deleted
```

---

## 7. Benefits

- **Single Responsibility:** Each class has one clear concept to represent
- **Reuse:** The extracted class can be used by other classes independently
- **Encapsulation:** Validation and formatting logic for the concept live in one place
