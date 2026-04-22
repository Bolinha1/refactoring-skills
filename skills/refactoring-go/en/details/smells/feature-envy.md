# SKILL: Detecting and Refactoring Feature Envy — Go

## Source
Based on: https://refactoring.guru/smells/feature-envy

---

## 1. What is Feature Envy

A function accesses the data of another struct more than it accesses its own. The function seems to want to live in the other type — it is "envious" of that type's data.

**Why this happens:**
- Fields were moved to a new struct but the functions that use them were not
- Business logic was placed in a service type that works on a domain struct's internals
- A function was gradually extended to rely more and more on another struct's fields

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A function calls several fields or methods on an argument struct in sequence
- [ ] A method on struct A uses mostly struct B's data, ignoring its own fields entirely
- [ ] The function's name naturally belongs to the other type ("calculateInvoiceTotal" in a `Printer`)
- [ ] Removing the function from its current type and adding it to the target type requires zero changes to the logic

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| The entire function belongs in the other struct | Move Method |
| Only part of the function envies another struct | Extract Function, then Move Method |
| The function uses data from several structs | Move to the struct that contributes most data |
| The function is a calculation that belongs on the model | Move to the domain struct as a method |

---

## 4. Example

**BEFORE — not accepted:**
```go
package billing

type InvoicePrinter struct{}

// calculateTotal lives in Printer but only uses Invoice data — classic Feature Envy
func (p *InvoicePrinter) calculateTotal(inv *Invoice) float64 {
	subtotal := 0.0
	for _, item := range inv.Items {
		subtotal += item.Price * float64(item.Quantity)
	}
	tax := subtotal * inv.TaxRate
	var discount float64
	if inv.HasDiscount {
		discount = subtotal * inv.DiscountRate
	}
	return subtotal + tax - discount
}

func (p *InvoicePrinter) Print(inv *Invoice) {
	total := p.calculateTotal(inv)
	fmt.Printf("Invoice total: %.2f\n", total)
}
```

**AFTER — expected:**
```go
package billing

// Total now lives on Invoice — it only uses Invoice data.
func (inv *Invoice) Total() float64 {
	subtotal := 0.0
	for _, item := range inv.Items {
		subtotal += item.Price * float64(item.Quantity)
	}
	tax := subtotal * inv.TaxRate
	var discount float64
	if inv.HasDiscount {
		discount = subtotal * inv.DiscountRate
	}
	return subtotal + tax - discount
}

type InvoicePrinter struct{}

func (p *InvoicePrinter) Print(inv *Invoice) {
	fmt.Printf("Invoice total: %.2f\n", inv.Total())
}
```

**Why this pattern:**
- `Total()` naturally belongs to `Invoice` — it only uses `Invoice` data
- `InvoicePrinter` is freed from knowing `Invoice` internals
- Callers across the codebase can now call `inv.Total()` directly

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the envious method and adding a pass-through**
```go
// Not accepted — delegation without removal creates indirection, not clarity
func (p *InvoicePrinter) calculateTotal(inv *Invoice) float64 {
	return inv.Total() // just a wrapper — delete the envious method entirely
}
```

**Mistake 2: Moving the method but leaking internals via parameters**
```go
// Not accepted — moved but still passing raw field values
func (inv *Invoice) Total(subtotal, taxRate, discountRate float64, hasDiscount bool) float64 { ... }
```

---

## 6. Benefits

- **Cohesion:** Each type owns the behavior that operates on its own data
- **Encapsulation:** The domain struct exposes intent, not raw fields
- **Maintainability:** Business rules are co-located with the data they govern
