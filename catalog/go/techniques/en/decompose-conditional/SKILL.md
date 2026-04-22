# TECHNIQUE: Decompose Conditional — Go

## Source
Based on: https://refactoring.guru/decompose-conditional

---

## 1. Problem

A complex `if/else` or `switch` statement contains intricate logic in its condition and branches. The code is hard to read because the intent of each part is buried in low-level detail.

---

## 2. Solution

Extract the condition and each branch into separate, well-named functions or methods. The function names communicate the intent; the bodies contain the detail.

---

## 3. When to apply

- The condition expression requires a comment to be understood
- The true or false branch contains several lines of non-obvious logic
- The same complex condition appears in more than one place
- The method is hard to name because it does too many things

---

## 4. Refactoring steps

1. Extract the condition into a function with a name that states what it checks
2. Extract the true branch into a function with a name that states what it does
3. Extract the false branch (if present) into a separate function
4. Replace the original `if` with a call using the extracted names
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func (o *Order) charge() float64 {
    if o.date.Before(summerStart) || o.date.After(summerEnd) {
        return o.quantity*o.winterRate + o.winterServiceCharge
    }
    return o.quantity * o.summerRate
}
```

**AFTER — expected:**
```go
func (o *Order) charge() float64 {
    if o.isOffSeason() {
        return o.winterCharge()
    }
    return o.summerCharge()
}

func (o *Order) isOffSeason() bool {
    return o.date.Before(summerStart) || o.date.After(summerEnd)
}

func (o *Order) winterCharge() float64 {
    return o.quantity*o.winterRate + o.winterServiceCharge
}

func (o *Order) summerCharge() float64 {
    return o.quantity * o.summerRate
}
```

**Why this pattern:**
- `charge()` now reads as a high-level policy; readers understand it without decoding the date math
- Each helper can be tested and changed independently

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting with a name that restates the code instead of the intent**
```go
// Not accepted — the name repeats what the code says, not what it means
func (o *Order) isDateBeforeSummerStartOrAfterSummerEnd() bool { ... }
```

**Mistake 2: Extracting only the condition but leaving complex branches inline**
```go
// Not accepted — partial extraction leaves the branch logic still unreadable
func (o *Order) charge() float64 {
    if o.isOffSeason() {
        // 10 lines of inline winter calculation
    }
    return o.quantity * o.summerRate
}
```

---

## 7. Benefits

- **Readability:** The `if` statement reads like a business rule, not low-level logic
- **Reuse:** Extracted condition functions can be reused across the codebase
- **Testability:** Each branch can be tested independently
- **Maintainability:** Changes to seasonal rules are isolated to one function
