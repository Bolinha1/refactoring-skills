# SKILL: Detecting and Refactoring Duplicate Code — Python

## Source
Based on: https://refactoring.guru/smells/duplicate-code

---

## 1. What is Duplicate Code

Two or more code fragments look almost identical or are structurally similar across different places in the codebase. Any change to the logic must be replicated in every copy, and forgetting one creates a silent bug.

**Why this happens:**
- Copy-paste used as a shortcut instead of abstraction
- Parallel development where two developers solved the same problem independently
- Logic that started similar and diverged slightly, making the duplication invisible

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Same or nearly identical block of code in two or more functions/methods
- [ ] Same computation or expression repeated in several places
- [ ] Two classes contain methods that do essentially the same thing
- [ ] Two subclasses implement methods with the same body
- [ ] You copy-pasted code and tweaked one variable

---

## 3. Treatment techniques (in order of preference)

| Situation found                                        | Recommended technique             |
|--------------------------------------------------------|-----------------------------------|
| Duplicate blocks in the same class                     | Extract Method                    |
| Duplicate blocks in sibling subclasses                 | Pull Up Method                    |
| Similar but not identical blocks in sibling subclasses | Form Template Method              |
| Duplicate code in two unrelated classes                | Extract Class or Move Method      |
| Long duplicate methods that differ only in algorithm   | Substitute Algorithm              |

---

## 4. Example

**BEFORE — not accepted:**
```python
class SalesReport:
    def calculate_revenue(self, orders: list) -> float:
        total = 0
        for order in orders:
            if order.status == "paid":
                total += order.amount
        return total


class FinanceReport:
    def compute_paid_orders_total(self, orders: list) -> float:
        total = 0
        for order in orders:
            if order.status == "paid":
                total += order.amount
        return total
```

**AFTER — expected:**
```python
def sum_paid_orders(orders: list) -> float:
    return sum(order.amount for order in orders if order.status == "paid")


class SalesReport:
    def calculate_revenue(self, orders: list) -> float:
        return sum_paid_orders(orders)


class FinanceReport:
    def compute_paid_orders_total(self, orders: list) -> float:
        return sum_paid_orders(orders)
```

**Why this pattern:**
- Business logic lives in one place — a single change fixes all callers
- Both report classes delegate to the shared utility instead of owning the logic

---

## 5. Negative examples — what NOT to do

**Mistake 1: Accepting small differences as justification for duplication**
```python
# Not accepted — "almost the same" still means duplicate
def calculate_revenue(self, orders): ...      # filters paid
def calculate_projected_revenue(self, orders): ...  # same loop, one extra multiply
```

**Mistake 2: Extracting to a private method but duplicating it in each class**
```python
# Not accepted — _sum_paid is duplicated in both SalesReport and FinanceReport
def _sum_paid(self, orders): ...
```

**Mistake 3: Merging duplicates into a god function with boolean flags**
```python
# Not accepted — creates Feature Envy / Long Method
def calculate(orders, paid=False, projected=False, with_taxes=False): ...
```

---

## 6. Benefits

- **Maintenance:** A single bug fix or rule change applies everywhere automatically
- **Clarity:** Code that expresses a shared concept is easier to understand than scattered copies
- **Trust:** Developers can be confident there are no diverged copies hiding stale logic
