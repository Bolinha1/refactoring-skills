# TECHNIQUE: Move Field — Go

## Source
Based on: https://refactoring.guru/move-field

---

## 1. Problem

A field in one struct is used more heavily by another struct, or it logically belongs to a concept that is modelled by a different struct. The field is in the wrong home, causing unnecessary coupling.

---

## 2. Solution

Move the field to the struct that uses it most or that conceptually owns it. Update all references to go through the new location.

---

## 3. When to apply

- A field is accessed more often from another struct than from the one that owns it
- The field's concept is more closely related to another struct
- You are applying Extract Struct and need to move fields to the new struct
- A field has to be passed as a parameter to many methods of another struct

---

## 4. Refactoring steps

1. Add the field to the target struct
2. If the source struct still needs access, add an accessor or method on the target struct
3. Update all direct references to the field to go through the new location
4. Remove the field from the source struct
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Account struct {
    accountType *AccountType
    interestRate float64 // belongs more to AccountType
}

func (a *Account) InterestFor(amount float64) float64 {
    return a.interestRate * amount
}
```

**AFTER — expected:**
```go
type AccountType struct {
    interestRate float64
}

func (at *AccountType) InterestFor(amount float64) float64 {
    return at.interestRate * amount
}

type Account struct {
    accountType *AccountType
}

func (a *Account) InterestFor(amount float64) float64 {
    return a.accountType.InterestFor(amount)
}
```

**Why this pattern:**
- Different account types can now have different interest rates without changing `Account`
- The rate logically belongs to the type, not to a specific account instance

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving a field and keeping a duplicate in the source struct**
```go
// Not accepted — having the field in both places creates inconsistency
type Account struct {
    accountType  *AccountType
    interestRate float64 // still here after moving to AccountType
}
```

**Mistake 2: Moving a field without updating all access paths**
```go
// Not accepted — some code still accesses the old field directly
// while other code uses the new location; the codebase is inconsistent
```

---

## 7. Benefits

- **Cohesion:** Data lives close to the behaviour that uses it
- **Reduced coupling:** Structs no longer reach into each other unnecessarily
- **Flexibility:** The owning struct can encapsulate changes to the field
