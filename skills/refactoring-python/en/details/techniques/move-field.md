# TECHNIQUE: Move Field — Python

## Source
Based on: https://refactoring.guru/move-field

---

## 1. Problem

An attribute is used more by another class than by its own class — other classes read or write it more frequently than the class that contains it.

---

## 2. Solution

Create a new attribute in the target class. Update all references to use the new location, then delete the old attribute.

---

## 3. When to apply

- Methods in another class reference the attribute more than the class that owns it
- You are doing Extract Class and need to move attributes with their methods
- An attribute is a piece of information that conceptually belongs to another class
- After a Move Method, the moved method now references attributes left in the original class

---

## 4. Refactoring steps

1. Check if the attribute is used by methods in its own class — if so, consider moving those methods too (Move Method)
2. Add a property or accessor for the attribute if it needs controlled access
3. Add the attribute to the target class
4. Update the original class to delegate attribute access to the target class (or move dependent methods)
5. Update all external references to use the target class directly
6. Delete the original attribute
7. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Account:
    def __init__(self, account_type: "AccountType", interest_rate: float):
        self.account_type = account_type
        self.interest_rate = interest_rate  # used by type-related logic

    def interest_for_amount(self, amount: float) -> float:
        return self.interest_rate * amount


class AccountType:
    # interest_rate conceptually belongs here
    pass
```

**AFTER — expected:**
```python
class AccountType:
    def __init__(self, interest_rate: float):
        self.interest_rate = interest_rate


class Account:
    def __init__(self, account_type: AccountType):
        self.account_type = account_type

    def interest_for_amount(self, amount: float) -> float:
        return self.account_type.interest_rate * amount  # delegates to AccountType
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the attribute but keeping a duplicate in the original class**
```python
# Not accepted — now interest_rate exists in both Account and AccountType
class Account:
    def __init__(self):
        self.interest_rate = 0.0      # old — should be deleted
        self.account_type = None
```

**Mistake 2: Moving the attribute without moving the methods that use it**
```python
# Not accepted — interest_for_amount still lives in Account but now must call back
# to AccountType — this introduces Feature Envy instead of fixing it
```

**Mistake 3: Moving an attribute used intensively by its original class**
```python
# Not accepted — if Account uses interest_rate in 10 methods and AccountType
# uses it in 1, the attribute should stay in Account
```

---

## 7. Benefits

- **Cohesion:** Data and the behavior that uses it live in the same class
- **Encapsulation:** The owning class controls all access to the attribute
- **Reduces coupling:** Other classes no longer need to reach into the original class for data
