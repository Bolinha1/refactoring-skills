# TECHNIQUE: Remove Assignments to Parameters — Go

## Source
Based on: https://refactoring.guru/remove-assignments-to-parameters

---

## 1. Problem

A parameter variable is reassigned inside a function. In Go, scalar and struct parameters are passed by value, so reassigning the parameter variable has no effect on the caller — but it is still confusing. For pointer parameters, reassigning the pointer variable (not the pointed-to value) is similarly misleading.

---

## 2. Solution

Introduce a new local variable instead of reusing the parameter variable. Give it a name that clarifies its role. Leave the parameter variable unchanged throughout the function.

---

## 3. When to apply

- A parameter variable is reassigned after the function starts (not just used)
- The reassignment uses the parameter as a starting value but produces a different concept
- The function signature reads as if the parameter flows through unchanged, but it is actually transformed

---

## 4. Refactoring steps

1. Identify the assignment to the parameter variable
2. Declare a new local variable with `:=` using the parameter's initial value as the starting point
3. Replace all uses of the parameter variable after the assignment with the new variable
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func discount(inputVal int, quantity int) float64 {
    if inputVal > 50 {
        inputVal -= 2 // reassigning the parameter
    }
    if quantity > 100 {
        inputVal -= 1 // reassigning again
    }
    return float64(inputVal) * 0.1
}
```

**AFTER — expected:**
```go
func discount(inputVal int, quantity int) float64 {
    result := inputVal // local variable, parameter stays unchanged
    if inputVal > 50 {
        result -= 2
    }
    if quantity > 100 {
        result -= 1
    }
    return float64(result) * 0.1
}
```

**Why this pattern:**
- `inputVal` clearly represents the caller's input throughout; `result` is what the function builds
- Readers can always refer back to `inputVal` to know the original value

---

## 6. Negative examples — what NOT to do

**Mistake 1: Confusing pointer reassignment with modifying pointed-to data**
```go
// This is acceptable — modifying the struct through the pointer is intentional
func update(order *Order) {
    order.Status = "processed" // OK: modifying *content*, not reassigning the pointer
}

// Not accepted — reassigning the pointer variable itself is a bug or confusion
func update(order *Order) {
    order = &Order{Status: "processed"} // caller sees no change; likely a mistake
}
```

**Mistake 2: Keeping the parameter name for the result**
```go
// Not accepted — using inputVal for both the input and the computed result
// makes the transformation invisible
func discount(inputVal int, quantity int) float64 {
    inputVal = inputVal - computeReduction(inputVal, quantity)
    return float64(inputVal) * 0.1
}
```

---

## 7. Benefits

- **Clarity:** Parameters clearly represent inputs; local variables represent computed state
- **Readability:** The original value is always available for reference throughout the function
- **Bug prevention:** Accidental overwrites of the parameter value are avoided
