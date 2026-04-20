# TECHNIQUE: Replace Conditional with Polymorphism — Python

## Source
Based on: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problem

You have an `if/elif/else` chain that performs different behaviours
depending on the object type or a property that simulates a type.

---

## 2. Solution

Create subclasses matching each branch of the conditional. In each subclass,
implement a method containing that branch's logic. Replace the conditional
with a polymorphic method call.

---

## 3. When to apply

- The conditional checks `isinstance()`, a type field, or a string/enum
- The same conditional appears in multiple methods
- Adding a new type would require modifying all existing conditionals (Open/Closed Principle violation)
- Each branch of the conditional represents a cohesive and distinct behaviour

---

## 4. Refactoring steps

1. If the conditional is inside a method with other responsibilities, apply **Extract Method** first
2. Create a base class with the method to be polymorphized (use `ABC` to enforce implementation)
3. For each branch, create a subclass that overrides the method
4. Move the branch logic to the corresponding subclass method
5. Remove the branch from the original conditional
6. Repeat until the conditional is empty
7. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Employee:
    def __init__(self, type: str, base_salary: float, sales: float = 0):
        self.type = type           # "engineer", "manager", "salesperson"
        self.base_salary = base_salary
        self.sales = sales

    def calculate_bonus(self) -> float:
        if self.type == "engineer":
            return self.base_salary * 0.10
        elif self.type == "manager":
            return self.base_salary * 0.20 + 1000
        elif self.type == "salesperson":
            return self.sales * 0.05
        else:
            raise ValueError(f"Unknown type: {self.type}")

    def generate_report(self) -> str:
        if self.type == "engineer":
            return "Engineer: delivered projects"
        elif self.type == "manager":
            return "Manager: led teams"
        elif self.type == "salesperson":
            return f"Salesperson: ${self.sales:.2f} in sales"
        else:
            raise ValueError(f"Unknown type: {self.type}")
```

**AFTER — expected:**
```python
from abc import ABC, abstractmethod


class Employee(ABC):
    def __init__(self, base_salary: float):
        self.base_salary = base_salary

    @abstractmethod
    def calculate_bonus(self) -> float: ...

    @abstractmethod
    def generate_report(self) -> str: ...


class Engineer(Employee):
    def calculate_bonus(self) -> float:
        return self.base_salary * 0.10

    def generate_report(self) -> str:
        return "Engineer: delivered projects"


class Manager(Employee):
    def calculate_bonus(self) -> float:
        return self.base_salary * 0.20 + 1000

    def generate_report(self) -> str:
        return "Manager: led teams"


class Salesperson(Employee):
    def __init__(self, base_salary: float, sales: float):
        super().__init__(base_salary)
        self.sales = sales

    def calculate_bonus(self) -> float:
        return self.sales * 0.05

    def generate_report(self) -> str:
        return f"Salesperson: ${self.sales:.2f} in sales"
```

**Why this pattern:**
- Adding a new employee type → create a new subclass, no changes to existing ones
- No duplicated `if/elif` across `calculate_bonus` and `generate_report`
- `ABC` guarantees at definition time that no subclass forgets to implement the methods

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the if/elif to a factory and thinking it's solved**
```python
# Not accepted — the conditional still exists, just moved to another place
def create_employee(type: str, base_salary: float) -> Employee:
    if type == "engineer":
        return Employee("engineer", base_salary)  # same class
    # ...
```

**Mistake 2: Creating subclasses but keeping the conditional in methods**
```python
# Not accepted — created inheritance but did not polymorphize the behaviour
class Engineer(Employee):
    def calculate_bonus(self) -> float:
        if self.type == "engineer":  # unnecessary conditional
            return self.base_salary * 0.10
        return 0
```

**Mistake 3: Using polymorphism for trivial business conditionals**
```python
# Not accepted — over-engineering for a simple if
# e.g.: if active: ... else: ...
```

---

## 7. Benefits

- **Open/Closed Principle:** New types do not require modifying existing code
- **Duplicate elimination:** The same conditional does not need to be repeated
- **Readability:** Each class clearly expresses its own behaviour
- **Testability:** Each subclass can be tested in isolation
