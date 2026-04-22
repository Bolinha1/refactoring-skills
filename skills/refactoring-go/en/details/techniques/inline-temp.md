# TECHNIQUE: Inline Temp — Go

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
- The variable is only used as an argument to another function or in a return statement
- Extract Method or Replace Temp with Query is being applied and the temp is in the way

---

## 4. Refactoring steps

1. Verify the variable is assigned only once
2. Find all uses of the variable in its scope
3. Replace each use with the right-hand-side expression
4. Remove the variable declaration
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func isDiscountEligible(order *Order) bool {
    eligible := order.ItemCount > 10
    return eligible
}
```

**AFTER — expected:**
```go
func isDiscountEligible(order *Order) bool {
    return order.ItemCount > 10
}
```

**Another example — temp used as intermediate argument:**
```go
// BEFORE
basePrice := float64(order.Quantity) * order.ItemPrice
return basePrice > 1000

// AFTER
return float64(order.Quantity)*order.ItemPrice > 1000
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a variable that captures a meaningful business concept**
```go
// Not accepted — "isEligibleForBulkDiscount" communicates intent;
// inlining it loses the business term
isEligibleForBulkDiscount := order.ItemCount > 50
if isEligibleForBulkDiscount { ... }
```

**Mistake 2: Inlining when the expression is expensive and used multiple times**
```go
// Not accepted — inlining causes the database call to run on every reference
customer := findCustomerByID(order.CustomerID)
// used 3 times below — inlining would hit the database 3 times
```

---

## 7. Benefits

- **Removes clutter:** Eliminates variables that add no semantic value
- **Prepares for other refactorings:** Clearing temps unblocks Extract Method and other moves
- **Simpler code:** Less to read, less to name, less to maintain
