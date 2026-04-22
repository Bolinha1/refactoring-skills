# TECHNIQUE: Inline Method — Go

## Source
Based on: https://refactoring.guru/inline-method

---

## 1. Problem

A method's body is as clear as its name, or the method is only a thin wrapper that adds no value. The extra indirection makes the code harder to follow rather than easier.

---

## 2. Solution

Replace every call to the method with the method's body. Remove the method declaration.

---

## 3. When to apply

- The method's body is one line and its name communicates nothing more than that line
- The method is called in only one place and was only created for an intermediate refactoring step
- The method is so trivial that it adds indirection without adding clarity
- You are consolidating a group of poorly designed methods before restructuring them

---

## 4. Refactoring steps

1. Check that the method is not overriding an interface method or used via an interface
2. Find all call sites
3. Replace each call with the method body (adjusting parameter references as needed)
4. Run the tests after each replacement
5. Remove the method declaration
6. Run the tests one final time

---

## 5. Example

**BEFORE — not accepted:**
```go
func (o *Order) hasMoreThanFiveItems() bool {
    return len(o.items) > 5
}

func (o *Order) isEligibleForDiscount() bool {
    return o.hasMoreThanFiveItems()
}
```

**AFTER — expected:**
```go
func (o *Order) isEligibleForDiscount() bool {
    return len(o.items) > 5
}
```

**Another example — delegating function:**
```go
// BEFORE
func rating(driver *Driver) int {
    return moreThanFiveLateDeliveries(driver)
}

func moreThanFiveLateDeliveries(driver *Driver) int {
    if driver.NumberOfLateDeliveries > 5 {
        return 2
    }
    return 1
}

// AFTER — inlined since rating() added no new meaning
func rating(driver *Driver) int {
    if driver.NumberOfLateDeliveries > 5 {
        return 2
    }
    return 1
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a method that satisfies an interface**
```go
// Not accepted — inlining removes the method from the type,
// breaking the interface contract
type Discountable interface {
    hasMoreThanFiveItems() bool
}
```

**Mistake 2: Inlining a method with meaningful domain logic just to reduce line count**
```go
// Not accepted — if isEligibleForBulkDiscount() contains real business rules,
// keeping it named preserves business vocabulary
```

---

## 7. Benefits

- **Removes useless indirection:** Code that adds nothing is removed
- **Simplicity:** Fewer symbols to understand and navigate
- **Prepares for restructuring:** Collapsing small methods can make the next refactoring clearer
