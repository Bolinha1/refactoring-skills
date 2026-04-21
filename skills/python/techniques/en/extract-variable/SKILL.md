# TECHNIQUE: Extract Variable — Python

## Source
Based on: https://refactoring.guru/extract-variable

---

## 1. Problem

A complex expression is hard to understand at a glance. Intermediate results are computed inline, making it difficult to see what each part means or to debug the calculation.

---

## 2. Solution

Assign the expression (or a sub-expression) to a clearly named temporary variable. Use the variable name to convey the meaning of the result.

---

## 3. When to apply

- A conditional expression has multiple parts that each deserve a name
- A long arithmetic or string expression hides what it is computing
- The same sub-expression appears more than once in the same scope
- A debug breakpoint is needed on an intermediate result

---

## 4. Refactoring steps

1. Identify the expression or sub-expression to name
2. Assign the expression to a descriptive local variable
3. Replace the original expression (and all duplicates in the same scope) with the variable
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def price(order) -> float:
    return (order.quantity * order.item_price
            - max(0, order.quantity - 500) * order.item_price * 0.05
            + min(order.quantity * order.item_price * 0.1, 100.0))
```

**AFTER — expected:**
```python
def price(order) -> float:
    base_price = order.quantity * order.item_price
    quantity_discount = max(0, order.quantity - 500) * order.item_price * 0.05
    shipping = min(base_price * 0.1, 100.0)
    return base_price - quantity_discount + shipping
```

**Why this pattern:**
- Each variable name explains the intent of its sub-expression
- `base_price` is computed once and reused, removing duplication

---

## 6. Negative examples — what NOT to do

**Mistake 1: Naming a variable after its type or position rather than its meaning**
```python
# Not accepted — result, temp, value add no information
result = order.quantity * order.item_price
```

**Mistake 2: Over-extracting every single sub-expression**
```python
# Not accepted — too granular; single operations rarely need a variable
qty = order.quantity
price = order.item_price
product = qty * price
```

**Mistake 3: Mutating the variable after extraction**
```python
# Not accepted — a variable extracted for clarity should not be reassigned
base_price = order.quantity * order.item_price
base_price = base_price * 1.1  # confusing — should be a new variable
```

---

## 7. Benefits

- **Readability:** Names make complex expressions self-documenting
- **Debuggability:** You can set a breakpoint or print each named step
- **Reduced duplication:** A sub-expression used more than once is computed once and named
