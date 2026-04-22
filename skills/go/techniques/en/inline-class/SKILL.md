# TECHNIQUE: Inline Class (Inline Struct) — Go

## Source
Based on: https://refactoring.guru/inline-class

---

## 1. Problem

A struct is no longer pulling its weight. It has too few responsibilities — perhaps after other refactorings — and its existence adds indirection without adding clarity.

---

## 2. Solution

Move the struct's fields and methods into the struct that uses it. Remove the now-empty struct and update all references.

---

## 3. When to apply

- A struct has only one or two trivial methods and one owner
- The struct's concept does not add domain meaning — it is just a bag of data
- The struct was created by a previous refactoring and is no longer justified
- Callers always access the struct through a single owner and never independently

---

## 4. Refactoring steps

1. Declare all fields and methods of the inlined struct directly on the target struct
2. Delegate each method of the original struct to the new copy temporarily
3. Update all callers to use the target struct's methods directly
4. Remove the delegating methods from the original struct
5. Remove the original struct and its field from the target struct
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type TelephoneNumber struct {
    areaCode string
    number   string
}

func (t TelephoneNumber) String() string {
    return "(" + t.areaCode + ") " + t.number
}

type Person struct {
    name  string
    phone TelephoneNumber
}

func (p *Person) TelephoneNumber() string { return p.phone.String() }
```

**AFTER — expected:**
```go
type Person struct {
    name      string
    areaCode  string
    number    string
}

func (p *Person) TelephoneNumber() string {
    return "(" + p.areaCode + ") " + p.number
}
```

**Why this pattern:**
- `TelephoneNumber` added no domain value here — its only owner was `Person`
- Removing it reduces the number of types callers need to understand

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a struct that is reused by multiple other structs**
```go
// Not accepted — if Address is used by Person, Company, and Vendor,
// inlining duplicates the concept and its logic across all three
```

**Mistake 2: Inlining a struct that has validation or complex behaviour**
```go
// Not accepted — if TelephoneNumber has a Validate() method with real rules,
// inlining it spreads that logic to the owner and removes the clean abstraction
```

---

## 7. Benefits

- **Reduced complexity:** Fewer types to understand and navigate
- **Less indirection:** Callers access data directly without going through an extra struct
- **Clarity:** The codebase only contains abstractions that earn their keep
