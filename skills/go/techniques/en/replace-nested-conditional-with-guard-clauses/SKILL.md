# TECHNIQUE: Replace Nested Conditional with Guard Clauses — Go

## Source
Based on: https://refactoring.guru/replace-nested-conditional-with-guard-clauses

---

## 1. Problem

A function has deeply nested `if/else` blocks that make the happy path hard to see. Each level of nesting adds cognitive load and pushes the main logic to the right.

---

## 2. Solution

Use early returns (guard clauses) to handle exceptional or precondition cases at the top of the function. Once the special cases are dispatched, the remaining code is the main, un-nested path. This is very idiomatic Go.

---

## 3. When to apply

- A function has multiple layers of nested `if/else` blocks
- Several branches represent precondition failures, invalid states, or edge cases
- The main logic is buried at the deepest level of nesting
- The function checks multiple conditions before doing its real work

---

## 4. Refactoring steps

1. Identify the conditions that represent special cases (errors, early exits, edge cases)
2. For each special case, add an `if` at the top of the function that returns (or returns an error) immediately
3. Remove the corresponding `else` branch — it is no longer needed
4. Repeat until the main logic is at the top level with no surrounding `else`
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func (e *Employee) PayAmount() (float64, error) {
    if !e.IsDead {
        if e.IsSeparated {
            return separatedPay(e)
        } else {
            if e.IsRetired {
                return retiredPay(e)
            } else {
                return normalPay(e)
            }
        }
    } else {
        return 0, nil
    }
}
```

**AFTER — expected:**
```go
func (e *Employee) PayAmount() (float64, error) {
    if e.IsDead {
        return 0, nil
    }
    if e.IsSeparated {
        return separatedPay(e)
    }
    if e.IsRetired {
        return retiredPay(e)
    }
    return normalPay(e)
}
```

**Why this pattern:**
- Each guard clause is a standalone rule, easy to read and modify independently
- The main case (`normalPay`) is obvious — it is what remains after all guards pass
- This is the standard Go style: validate inputs early, then proceed with confidence

---

## 6. Negative examples — what NOT to do

**Mistake 1: Leaving else blocks after a return**
```go
// Not accepted — the else is unreachable after the return and adds noise
func (e *Employee) PayAmount() (float64, error) {
    if e.IsDead {
        return 0, nil
    } else { // unnecessary else
        return normalPay(e)
    }
}
```

**Mistake 2: Inverting conditions to create new nesting**
```go
// Not accepted — inverting the guard to use the positive form
// but then nesting the guard body creates the same problem
func (e *Employee) PayAmount() (float64, error) {
    if !e.IsDead {
        if !e.IsSeparated {
            // still nested
        }
    }
    return 0, nil
}
```

**Mistake 3: Using panic instead of returning an error for known failure cases**
```go
// Not accepted — in Go, use error returns for expected failure modes
if e.IsDead {
    panic("dead employee") // incorrect Go idiom
}
```

---

## 7. Benefits

- **Readability:** The main path is immediately visible without decoding nested conditions
- **Idiomatic Go:** Early returns are the standard Go pattern for handling preconditions
- **Maintainability:** Each guard clause is independent and easy to add, remove, or reorder
- **Reduced cognitive load:** Readers only need to track one branch at a time
