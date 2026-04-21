# TECHNIQUE: Inline Temp — Python

## Source
Based on: https://refactoring.guru/inline-temp

---

## 1. Problem

A local variable holds the result of a simple expression, and the variable name adds no information that the expression itself does not already communicate clearly.

---

## 2. Solution

Replace every reference to the variable with the expression itself. Remove the variable declaration.

---

## 3. When to apply

- The variable is assigned exactly once and the expression is clear on its own
- The variable name does not add meaning beyond restating the expression
- The variable is only used as an argument to another method or in a return statement
- Extract Method or Replace Temp with Query is being applied and the temp is in the way

---

## 4. Refactoring steps

1. Verify the variable is assigned only once
2. Find all uses of the variable
3. Replace each use with the right-hand-side expression
4. Remove the variable declaration
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def is_discount_eligible(self, order) -> bool:
    eligible = order.item_count > 10
    return eligible
```

**AFTER — expected:**
```python
def is_discount_eligible(self, order) -> bool:
    return order.item_count > 10
```

**Another example — temp used as intermediate argument:**
```python
# BEFORE
base_price = order.quantity * order.item_price
return base_price > 1000

# AFTER
return order.quantity * order.item_price > 1000
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a variable that captures a meaningful concept**
```python
# Not accepted — is_eligible_for_bulk_discount communicates intent;
# inlining it loses the business term
is_eligible_for_bulk_discount = order.item_count > 50
```

**Mistake 2: Inlining when the expression is expensive and called multiple times**
```python
# Not accepted — inlining causes the database call to run on every reference
customer = self.find_customer_by_id(order.customer_id)
# Used 3 times below — inlining would hit the database 3 times
```

---

## 7. Benefits

- **Removes clutter:** Eliminates variables that add no semantic value
- **Prepares for other refactorings:** Clearing temps unblocks Extract Method and other moves
- **Simpler code:** Less to read, less to name, less to maintain
