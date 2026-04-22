# TECHNIQUE: Extract Struct — Go

## Source
Based on: https://refactoring.guru/extract-class

---

## 1. Problem

A struct is doing work that should be done by two. It has grown to hold fields and methods that belong to a distinct concept, making it harder to understand and change.

---

## 2. Solution

Create a new struct, move the related fields and methods to it, and embed or reference it from the original struct. Each struct now has a single, clear responsibility.

---

## 3. When to apply

- A subset of fields and methods always change together or are always used together
- The struct has grown too large and is hard to explain in a single sentence
- A group of fields could be named and reused in a different context
- You notice a natural cluster of fields with a cohesive meaning (e.g., address, money amount, date range)

---

## 4. Refactoring steps

1. Identify the cluster of fields and methods to extract
2. Create a new struct for the extracted concept with a clear name
3. Move the fields to the new struct
4. Move the related methods to the new struct (update receivers)
5. Add a field in the original struct that holds the new struct (by value or pointer)
6. Update all references in the original struct's methods to go through the new field
7. Decide whether to expose the new struct directly or hide it behind wrapper methods
8. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Person struct {
    Name        string
    PhoneNumber string
    AreaCode    string
    OfficeCode  string
}

func (p *Person) TelephoneNumber() string {
    return "(" + p.AreaCode + ") " + p.PhoneNumber
}
```

**AFTER — expected:**
```go
type TelephoneNumber struct {
    AreaCode string
    Number   string
}

func (t TelephoneNumber) String() string {
    return "(" + t.AreaCode + ") " + t.Number
}

type Person struct {
    Name   string
    Office TelephoneNumber
}

func (p *Person) TelephoneNumber() string {
    return p.Office.String()
}
```

**Why this pattern:**
- `TelephoneNumber` can now be reused on other structs (e.g., `Company`, `Vendor`)
- Validation logic for phone numbers lives in one place

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting a struct with only one field**
```go
// Not accepted — wrapping a single string adds indirection with no benefit
type Name struct {
    Value string
}
```

**Mistake 2: Extracting but keeping methods on the original struct**
```go
// Not accepted — the point is to move the behaviour, not just the data
type TelephoneNumber struct { AreaCode, Number string }

func (p *Person) TelephoneNumber() string {
    return "(" + p.Office.AreaCode + ") " + p.Office.Number // logic still here
}
```

---

## 7. Benefits

- **Single Responsibility:** Each struct has one clear role
- **Reuse:** The extracted struct can be used wherever that concept appears
- **Cohesion:** Related data and behaviour live together
- **Testability:** The new struct can be tested independently
