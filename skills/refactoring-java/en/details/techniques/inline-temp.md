# TECHNIQUE: Inline Temp — Java

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

1. Verify the variable is assigned only once (it should be `final`)
2. Find all uses of the variable
3. Replace each use with the right-hand-side expression
4. Remove the variable declaration
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public boolean isDiscountEligible(Order order) {
    boolean eligible = order.getItemCount() > 10;
    return eligible;
}
```

**AFTER — expected:**
```java
public boolean isDiscountEligible(Order order) {
    return order.getItemCount() > 10;
}
```

**Another example — temp used as intermediate argument:**
```java
// BEFORE
double basePrice = order.getQuantity() * order.getItemPrice();
return basePrice > 1000;

// AFTER
return order.getQuantity() * order.getItemPrice() > 1000;
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a variable that captures a meaningful concept**
```java
// Not accepted — "isEligibleForBulkDiscount" communicates intent;
// inlining it loses the business term
boolean isEligibleForBulkDiscount = order.getItemCount() > 50;
```

**Mistake 2: Inlining when the expression is expensive and called multiple times**
```java
// Not accepted — inlining causes the database call to run on every reference
Customer customer = findCustomerById(order.getCustomerId());
// Used 3 times below — inlining would hit the database 3 times
```

---

## 7. Benefits

- **Removes clutter:** Eliminates variables that add no semantic value
- **Prepares for other refactorings:** Clearing temps unblocks Extract Method and other moves
- **Simpler code:** Less to read, less to name, less to maintain
