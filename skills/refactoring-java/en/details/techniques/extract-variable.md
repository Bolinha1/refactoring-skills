# TECHNIQUE: Extract Variable — Java

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
2. Declare a `final` (or effectively-final) local variable and assign the expression to it
3. Replace the original expression (and all duplicates in the same scope) with the variable
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public double price(Order order) {
    return order.getQuantity() * order.getItemPrice()
        - Math.max(0, order.getQuantity() - 500) * order.getItemPrice() * 0.05
        + Math.min(order.getQuantity() * order.getItemPrice() * 0.1, 100.0);
}
```

**AFTER — expected:**
```java
public double price(Order order) {
    final double basePrice = order.getQuantity() * order.getItemPrice();
    final double quantityDiscount = Math.max(0, order.getQuantity() - 500)
        * order.getItemPrice() * 0.05;
    final double shipping = Math.min(basePrice * 0.1, 100.0);
    return basePrice - quantityDiscount + shipping;
}
```

**Why this pattern:**
- Each variable name explains the intent of its sub-expression
- `basePrice` is computed once and reused, removing duplication

---

## 6. Negative examples — what NOT to do

**Mistake 1: Naming a variable after its type or position rather than its meaning**
```java
// Not accepted — "result", "temp", "value" add no information
final double result = order.getQuantity() * order.getItemPrice();
```

**Mistake 2: Over-extracting every single sub-expression**
```java
// Not accepted — too granular; single operations rarely need a variable
final int qty = order.getQuantity();
final double price = order.getItemPrice();
final double product = qty * price;
```

**Mistake 3: Mutating the variable after extraction**
```java
// Not accepted — a variable extracted for clarity should be final
double basePrice = order.getQuantity() * order.getItemPrice();
basePrice = basePrice * 1.1; // confusing — should be a new variable
```

---

## 7. Benefits

- **Readability:** Names make complex expressions self-documenting
- **Debuggability:** You can set a breakpoint on each named step
- **Reduced duplication:** A sub-expression used more than once is computed once and named
