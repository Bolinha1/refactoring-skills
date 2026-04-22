# SKILL: Detecting and Refactoring Primitive Obsession — Go

## Source
Based on: https://refactoring.guru/smells/primitive-obsession

---

## 1. What is Primitive Obsession

Using primitive types (`string`, `int`, `float64`) to represent domain concepts that deserve their own types. A phone number is not just a `string`; a price is not just a `float64`; a user ID is not just an `int`.

**Why this happens:**
- Primitives are always available and require no extra definition
- The domain concept seemed too simple to warrant a type at first
- Developers coming from dynamically typed languages underestimate type safety benefits

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A `string` parameter named `phone`, `email`, `zipCode`, or `currency` could be passed any string
- [ ] Two `int` or `string` parameters of the same type appear next to each other and could be swapped silently
- [ ] Validation logic for a primitive is duplicated across every caller
- [ ] A constant group like `const StatusPending = "pending"` represents a finite domain concept
- [ ] A `float64` is used for money without any protection against floating-point errors

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Primitive represents a domain concept | Replace Primitive with Object (define a named type) |
| Group of related primitives always travel together | Replace Data Value with Object (struct) |
| Constant group encodes a finite set | Replace Type Code with iota or a dedicated type |
| Validation is duplicated across callers | Move validation into the constructor of the new type |

---

## 4. Example

**BEFORE — not accepted:**
```go
package billing

func CreateInvoice(customerID int, amount float64, currency string, email string) error {
	if amount <= 0 {
		return fmt.Errorf("invalid amount")
	}
	if currency == "" {
		return fmt.Errorf("currency required")
	}
	// email validation duplicated here and in 5 other places
	if !strings.Contains(email, "@") {
		return fmt.Errorf("invalid email")
	}
	// ...
	return nil
}
```

**AFTER — expected:**
```go
package billing

type CustomerID int

type Money struct {
	Amount   int64  // cents — no floating-point errors
	Currency string
}

func NewMoney(amount int64, currency string) (Money, error) {
	if amount <= 0 {
		return Money{}, fmt.Errorf("amount must be positive")
	}
	if currency == "" {
		return Money{}, fmt.Errorf("currency required")
	}
	return Money{Amount: amount, Currency: currency}, nil
}

type Email string

func NewEmail(s string) (Email, error) {
	if !strings.Contains(s, "@") {
		return "", fmt.Errorf("invalid email: %s", s)
	}
	return Email(s), nil
}

func CreateInvoice(customerID CustomerID, amount Money, email Email) error {
	// All validation already done by the type constructors
	return nil
}
```

**Why this pattern:**
- `CustomerID` and `int` are distinct types — the compiler prevents accidental swaps
- `Money` centralizes amount validation and eliminates floating-point errors
- `Email` validation runs once, at the boundary — not duplicated in every caller

---

## 5. Negative examples — what NOT to do

**Mistake 1: Defining a type alias without a constructor**
```go
// Not accepted — the alias adds no protection because any string can be assigned
type Email = string // type alias, not a new type
```

**Mistake 2: Using a struct for every single string without judgment**
```go
// Not accepted — over-engineering; first and last name are not worth wrapping
type FirstName struct{ Value string }
type LastName struct{ Value string }
```

---

## 6. Benefits

- **Type safety:** The compiler catches mix-ups between `CustomerID` and `OrderID` even though both are `int` underneath
- **Validation locality:** Domain rules live in the constructor, not scattered across callers
- **Expressiveness:** Function signatures read as a domain vocabulary, not a list of primitives
