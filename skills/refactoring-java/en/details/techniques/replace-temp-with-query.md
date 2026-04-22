# TECHNIQUE: Replace Temp with Query — Java

## Source
Based on: https://refactoring.guru/replace-temp-with-query

---

## 1. Problem

A local variable holds the result of an expression. That value is used later in the method, but the variable prevents the expression from being reused in other methods or subclasses. The method grows because it carries state that could be pushed into a reusable query.

---

## 2. Solution

Extract the expression into a method. Replace every use of the local variable with a call to the new method.

---

## 3. When to apply

- The same local variable is computed at the top and used far below, making the method long
- The expression would be useful in other methods of the same class
- The variable is read-only (never reassigned after initialisation)
- A subclass might want to override the expression with different logic

---

## 4. Refactoring steps

1. Ensure the expression has no side effects
2. Extract the expression into a private method with a clear name
3. Replace every reference to the local variable with a call to the new method
4. Remove the local variable
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public double price(Order order) {
    final double basePrice = order.getQuantity() * order.getItemPrice();
    final double discountFactor = basePrice > 1000 ? 0.95 : 0.98;
    return basePrice * discountFactor;
}
```

**AFTER — expected:**
```java
public double price(Order order) {
    return basePrice(order) * discountFactor(order);
}

private double basePrice(Order order) {
    return order.getQuantity() * order.getItemPrice();
}

private double discountFactor(Order order) {
    return basePrice(order) > 1000 ? 0.95 : 0.98;
}
```

**Why this pattern:**
- `basePrice()` and `discountFactor()` are reusable query methods
- A subclass can override `discountFactor()` to apply a different pricing rule

---

## 6. Negative examples — what NOT to do

**Mistake 1: Applying this technique when the expression has side effects**
```java
// Not accepted — a method called multiple times must not have side effects
private int nextId() {
    return ++this.counter; // side effect; calling it twice gives different results
}
```

**Mistake 2: Creating a query method that requires complex setup not available outside the current method**
```java
// Not accepted — if the expression depends on many local variables that would all
// become parameters, Extract Method is more appropriate
```

---

## 7. Benefits

- **Reuse:** The extracted method can be called from other methods in the same class
- **Extensibility:** Subclasses can override individual query methods
- **Readability:** The main method reads as a high-level formula with named sub-expressions
