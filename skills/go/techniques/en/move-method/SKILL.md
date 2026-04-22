# TECHNIQUE: Move Method — Go

## Source
Based on: https://refactoring.guru/move-method

---

## 1. Problem

A method uses data or behaviour from another struct more than from the struct it currently belongs to. Its current location creates unnecessary coupling and makes the other struct harder to use independently.

---

## 2. Solution

Move the method to the struct it interacts with most. Update the original struct to call the new location if it still needs the behaviour, or remove the old method entirely.

---

## 3. When to apply

- The method references fields or methods of another struct more than its own
- The method makes more sense as a method on the receiver it receives as a parameter
- Moving the method reduces coupling between two structs
- The method is needed in another struct and would need to be duplicated otherwise

---

## 4. Refactoring steps

1. Identify the target struct (the one the method interacts with most)
2. Create the method on the target struct; if the source struct is needed, pass it as a parameter or via a field
3. Update the method body to use the target struct's receiver instead of the parameter
4. Update callers of the original method to call the new location
5. Either remove or delegate from the original method
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Account struct {
    accountType *AccountType
    daysOverdrawn int
}

// This method uses AccountType more than Account
func (a *Account) OverdraftCharge() float64 {
    if a.accountType.IsPremium() {
        baseCharge := 10.0
        if a.daysOverdrawn <= 7 {
            return baseCharge
        }
        return baseCharge + float64(a.daysOverdrawn-7)*0.85
    }
    return float64(a.daysOverdrawn) * 1.75
}
```

**AFTER — expected:**
```go
type AccountType struct {
    premium bool
}

func (at *AccountType) IsPremium() bool { return at.premium }

// Method moved to AccountType; daysOverdrawn passed as parameter
func (at *AccountType) OverdraftCharge(daysOverdrawn int) float64 {
    if at.IsPremium() {
        baseCharge := 10.0
        if daysOverdrawn <= 7 {
            return baseCharge
        }
        return baseCharge + float64(daysOverdrawn-7)*0.85
    }
    return float64(daysOverdrawn) * 1.75
}

type Account struct {
    accountType   *AccountType
    daysOverdrawn int
}

func (a *Account) OverdraftCharge() float64 {
    return a.accountType.OverdraftCharge(a.daysOverdrawn)
}
```

**Why this pattern:**
- The charge logic now lives in `AccountType`, the struct that drives it
- Different account types can override the charge behaviour independently

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the method but leaving the original unreachable**
```go
// Not accepted — the original method still exists and confuses callers
// about which one to use; remove or fully delegate it
```

**Mistake 2: Moving a method that equally belongs to both structs without clear justification**
```go
// Not accepted — if the method genuinely uses both structs equally,
// consider Extract Method first to separate the parts before moving
```

---

## 7. Benefits

- **Cohesion:** Behaviour lives with the data it operates on
- **Reduced coupling:** The source struct no longer depends on internal details of the target
- **Discoverability:** Callers find behaviour where they expect it — on the struct it concerns
