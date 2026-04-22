# SKILL: Detecting and Refactoring Shotgun Surgery — Go

## Source
Based on: https://refactoring.guru/smells/shotgun-surgery

---

## 1. What is Shotgun Surgery

A single logical change requires modifying many different packages or structs at the
same time. Adding or renaming a concept "fires" edits across the codebase like a
shotgun blast — touching many files for what should be a localized fix.

The inverse of Divergent Change: many types, one reason to change.

**Why this happens:**
- A responsibility was fragmented across many types instead of living in one place
- Logic that should be centralized was duplicated for "flexibility"
- Cross-cutting concerns (logging, currency formatting, validation) were handled inline everywhere

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Adding a feature forces editing 5+ unrelated structs or packages
- [ ] Renaming a concept requires touching dozens of files
- [ ] A single business rule is enforced in multiple places
- [ ] You always make the same change in several packages simultaneously
- [ ] Test failures cascade across many packages for a single-concept change

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Scattered behaviour all relates to one concept | Move Method + Move Field |
| Multiple small types all contributing to one concept | Inline Class |
| Cross-cutting concern repeated everywhere | Extract Class (new struct) |
| Duplicated business rule in many places | Move Method to one owner |

---

## 4. Example

**BEFORE — not accepted:**
```go
// Adding a "currency" concept requires changing ALL of these:

type Product struct {
	Price float64 // must add CurrencyCode field here
}

func (o *Order) Total() float64 { ... } // must format with currency

func (i *Invoice) FormatAmount(amount float64) string {
	return fmt.Sprintf("$%.2f", amount) // must use currency
}

func (r *Receipt) PrintTotal(amount float64) {
	fmt.Println(amount) // must use currency
}
```

**AFTER — expected:**
```go
type Money struct {
	Amount   float64
	Currency string
}

func (m Money) Format() string {
	switch m.Currency {
	case "USD":
		return fmt.Sprintf("$%.2f", m.Amount)
	case "BRL":
		return fmt.Sprintf("R$%.2f", m.Amount)
	default:
		return fmt.Sprintf("%.2f %s", m.Amount, m.Currency)
	}
}

func (m Money) Add(other Money) (Money, error) {
	if m.Currency != other.Currency {
		return Money{}, fmt.Errorf("currency mismatch: %s vs %s", m.Currency, other.Currency)
	}
	return Money{Amount: m.Amount + other.Amount, Currency: m.Currency}, nil
}

// Now Product, Order, Invoice, Receipt all use Money
type Product struct {
	Price Money
}
```

**Why this pattern:**
- `Money` centralizes all currency behaviour — one type to change for new formatting rules
- All callers benefit automatically without modification

---

## 5. Negative examples — what NOT to do

**Mistake 1: Centralizing data but not behaviour**
```go
// Not accepted — groups the fields but formatting/arithmetic stay scattered
type MoneyDTO struct {
	Amount       float64
	CurrencyCode string
}
```

**Mistake 2: Using a package-level utility function**
```go
// Not accepted — callers still scatter logic; the function is just a named dump
func FormatMoney(amount float64, currency string) string { ... }
```

---

## 6. Benefits

- **Locality:** One business concept = one type to change
- **Safety:** Changes are easier to reason about when blast radius is small
- **Discoverability:** All behaviour for a concept is in one obvious place
