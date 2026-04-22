# TECHNIQUE: Move Method — Python

## Source
Based on: https://refactoring.guru/move-method

---

## 1. Problem

A method is used more in another class than in its own class.
This creates unnecessary coupling and violates the cohesion principle.

---

## 2. Solution

Declare the method in the class that uses it most frequently. Move the original code there.
In the original location, replace the method body with a delegation to the new method —
or remove it entirely if it is no longer needed.

---

## 3. When to apply

- The method accesses attributes of another class more than its own class's attributes
- The method would be more useful in the class that consumes it
- Moving the method would reduce or eliminate dependencies between classes
- The **Feature Envy** smell is present (method "envies" another class's data)

---

## 4. Refactoring steps

1. Analyze the method's dependencies within its current class
2. Check if the method is overridden in subclasses (avoid breaking polymorphism)
3. Declare the method in the target class with a contextually appropriate name
4. Obtain a reference to the target class (via attribute, parameter, or local instance)
5. Turn the original method into a delegation — or delete it if there are no external callers
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Account:
    def __init__(self, balance: float, contract: "ContractType"):
        self.balance = balance
        self.contract = contract

    def calculate_charge(self) -> float:
        # this method uses almost only ContractType data — it's in the wrong place
        if self.contract.type == "special":
            return self.contract.special_rate * self.balance * 30
        return self.contract.standard_rate * self.balance * 30


class ContractType:
    def __init__(self, type: str, standard_rate: float, special_rate: float):
        self.type = type
        self.standard_rate = standard_rate
        self.special_rate = special_rate
```

**AFTER — expected:**
```python
class Account:
    def __init__(self, balance: float, contract: "ContractType"):
        self.balance = balance
        self.contract = contract

    def calculate_charge(self) -> float:
        # delegates to whoever actually owns the data
        return self.contract.calculate_charge(self.balance)


class ContractType:
    def __init__(self, type: str, standard_rate: float, special_rate: float):
        self.type = type
        self.standard_rate = standard_rate
        self.special_rate = special_rate

    def calculate_charge(self, balance: float) -> float:
        if self.type == "special":
            return self.special_rate * balance * 30
        return self.standard_rate * balance * 30
```

**Why this pattern:**
- `calculate_charge` used `contract.type`, `contract.special_rate` and `contract.standard_rate`
- The method belongs to `ContractType` — that is where the required data lives
- `Account` now only coordinates, without needing to know the details of `ContractType`

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving and creating a mutual dependency**
```python
# Not accepted — creates circular coupling
class ContractType:
    def calculate_charge(self, account: Account) -> float:
        return self.rate * account.balance * account.days  # now depends on Account
```

**Mistake 2: Moving a method overridden in subclasses without evaluating the impact**
```python
# Not accepted — if calculate_charge() is overridden in SpecialAccount,
# moving without care breaks the polymorphic contract
```

**Mistake 3: Keeping the original version with duplicated logic**
```python
# Not accepted — two methods with the same behaviour in different classes
```

---

## 7. Benefits

- **Cohesion:** Each class contains the methods that belong to its data
- **Reduced coupling:** Eliminates unnecessary dependencies between classes
- **Maintainability:** Logic changes only affect the correct class
