# TECHNIQUE: Decompose Conditional — Python

## Source
Based on: https://refactoring.guru/decompose-conditional

---

## 1. Problem

A complex conditional expression (and the code in its branches) makes it hard to understand what is being tested and what happens in each case.

---

## 2. Solution

Extract the condition and each branch into well-named functions. The function names explain the intent; the bodies explain the implementation.

---

## 3. When to apply

- The condition itself is a complex boolean expression that requires thought to parse
- The code in the true or false branch is several lines and deserves a name
- The conditional appears inside an already-long function (combines with Long Method)
- Reading the condition requires knowledge of domain rules, not just programming logic

---

## 4. Refactoring steps

1. Extract the condition into a function named after the business rule it checks
2. Extract the true branch into a function named after what it does
3. Extract the false branch into a function named after what it does
4. Replace the original `if` with calls to the extracted functions
5. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def calculate_charge(date: date, quantity: int, unit_price: float) -> float:
    if SUMMER_START <= date <= SUMMER_END:
        charge = quantity * unit_price * SUMMER_RATE
    else:
        service_charge = WINTER_SERVICE_CHARGE if quantity > WINTER_THRESHOLD else 0
        charge = quantity * unit_price * WINTER_RATE + service_charge
    return charge
```

**AFTER — expected:**
```python
def calculate_charge(d: date, quantity: int, unit_price: float) -> float:
    if _is_summer(d):
        return _summer_charge(quantity, unit_price)
    return _winter_charge(quantity, unit_price)


def _is_summer(d: date) -> bool:
    return SUMMER_START <= d <= SUMMER_END


def _summer_charge(quantity: int, unit_price: float) -> float:
    return quantity * unit_price * SUMMER_RATE


def _winter_charge(quantity: int, unit_price: float) -> float:
    service_charge = WINTER_SERVICE_CHARGE if quantity > WINTER_THRESHOLD else 0
    return quantity * unit_price * WINTER_RATE + service_charge
```

**Why this pattern:**
- `_is_summer` names the business rule — readers understand it without parsing the date logic
- `_summer_charge` and `_winter_charge` express pricing intent, not implementation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting the function but giving it a technical name**
```python
# Not accepted — the name describes the mechanism, not the business rule
def _check_date_range(d: date) -> bool:
    return SUMMER_START <= d <= SUMMER_END
```

**Mistake 2: Extracting only the condition but not the branches**
```python
# Not accepted — partial extraction; the branches still need names
if _is_summer(d):
    charge = quantity * unit_price * SUMMER_RATE  # still inline
```

**Mistake 3: Decomposing a trivial condition**
```python
# Not accepted — over-engineering for a simple check
def _is_none(obj) -> bool: return obj is None
if _is_none(customer): ...
```

---

## 7. Benefits

- **Readability:** The `if` statement reads like a business rule, not a formula
- **Testability:** Each branch can be tested in isolation
- **Documentation:** Function names replace the need for comments
