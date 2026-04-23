# SKILL: Detecting and Refactoring Middle Man — Go

## Source
Based on: https://refactoring.guru/smells/middle-man

---

## 1. What is Middle Man

A type whose methods do nothing but delegate to another type. If more than half of a type's methods are one-line pass-throughs to another object, the middle man adds indirection without adding value.

**Why this happens:**
- Delegation was introduced to hide an implementation but the wrapper never grew its own logic
- A type was extracted for a good reason but its responsibilities were later moved away
- Over-engineering: an extra layer was added "for flexibility" that never materialized

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] More than half of the type's methods are single-line delegations to another object
- [ ] Removing the type and calling the delegate directly would change nothing for callers
- [ ] The type holds no state of its own beyond a reference to the delegate
- [ ] The type was described as "just a wrapper" when it was created

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Pure pass-through type with no added logic | Remove Middle Man — callers use the delegate directly |
| Type delegates but adds a small amount of logic | Inline the delegation; keep only the real logic |
| Type was meant to be an interface boundary | Replace with an interface; use the concrete type where possible |
| Delegation is useful for testing (mock seam) | Keep it as a thin interface, not a struct |

---

## 4. Example

**BEFORE — not accepted:**
```go
package billing

type BillingService struct {
	invoicer *Invoicer
}

// Every method just delegates — BillingService is a middle man.
func (b *BillingService) GenerateInvoice(o *Order) *Invoice {
	return b.invoicer.GenerateInvoice(o)
}

func (b *BillingService) SendInvoice(inv *Invoice) error {
	return b.invoicer.SendInvoice(inv)
}

func (b *BillingService) VoidInvoice(id string) error {
	return b.invoicer.VoidInvoice(id)
}
```

**AFTER — expected:**
```go
package billing

// BillingService removed — callers use *Invoicer directly.
// If an interface is needed for testing, define a minimal one:

type InvoicerPort interface {
	GenerateInvoice(o *Order) *Invoice
	SendInvoice(inv *Invoice) error
	VoidInvoice(id string) error
}

// Callers depend on InvoicerPort; *Invoicer implements it implicitly.
```

**Why this pattern:**
- Callers interact with `*Invoicer` (or the `InvoicerPort` interface) directly — one fewer indirection
- If a mock is needed in tests, the interface is the right seam — not a struct wrapper
- `BillingService` is deleted; the codebase shrinks without losing capability

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the middle man because "we might add logic later"**
```go
// Not accepted — YAGNI applies; add the wrapper when the logic actually arrives
type BillingService struct{ invoicer *Invoicer }
func (b *BillingService) GenerateInvoice(o *Order) *Invoice { return b.invoicer.GenerateInvoice(o) }
```

**Mistake 2: Replacing the middle man with a bigger middle man**
```go
// Not accepted — AppService is now the new middle man
type AppService struct {
	billing *BillingService
	order   *OrderService
}
func (a *AppService) Generate(o *Order) *Invoice { return a.billing.GenerateInvoice(o) }
```

---

## 6. Benefits

- **Simplicity:** Removing indirection makes call graphs easier to follow
- **Performance:** One fewer allocation and function call per operation
- **Clarity:** The real implementation is visible to callers and tools (go to definition, call hierarchy)
