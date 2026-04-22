# SKILL: Detecting and Refactoring Inappropriate Intimacy — Go

## Source
Based on: https://refactoring.guru/smells/inappropriate-intimacy

---

## 1. What is Inappropriate Intimacy

Two types are too tightly coupled: they access each other's internal details, manipulate each other's unexported state through accessor tricks, or are mutually dependent in ways that make them impossible to test or change independently.

**Why this happens:**
- A large type was split into two but the methods were not properly redistributed
- Two types evolved together and each absorbed knowledge of the other's internals
- Bidirectional references were added without considering encapsulation

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Type A accepts a `*B` and modifies B's fields directly instead of calling B's methods
- [ ] Two types import each other (circular imports — Go will refuse to compile this)
- [ ] A method in package A reaches into package B's internal struct fields
- [ ] Two types share so much logic they must always be changed together
- [ ] A type's method returns raw internal slices or maps that callers mutate

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Method in A uses mostly B's data | Move Method to B |
| A and B share bidirectional knowledge | Extract a third type to mediate |
| A exposes internals through raw pointers/slices | Return copies or use accessor methods |
| Circular package dependency | Introduce an interface in a shared package |

---

## 4. Example

**BEFORE — not accepted:**
```go
package store

type Cart struct {
	items []CartItem
}

type Checkout struct{}

// Checkout reaches into Cart's unexported slice directly via a getter hack
func (c *Checkout) ApplyPromotion(cart *Cart, code string) {
	// Reaches into Cart internals and mutates them
	for i := range cart.items {
		if code == "SALE10" {
			cart.items[i].Price *= 0.90 // Checkout knows Cart's internal layout
		}
	}
}
```

**AFTER — expected:**
```go
package store

type Cart struct {
	items []CartItem
}

// Cart owns its own mutation logic
func (cart *Cart) ApplyPromotion(code string) {
	for i := range cart.items {
		if code == "SALE10" {
			cart.items[i].Price *= 0.90
		}
	}
}

type Checkout struct{}

// Checkout delegates to Cart's own method — no internal knowledge needed
func (c *Checkout) Process(cart *Cart, promoCode string) {
	cart.ApplyPromotion(promoCode)
	// ... proceed with payment
}
```

**Why this pattern:**
- `Cart` encapsulates its own state changes; `Checkout` does not need to know the slice layout
- The promotion logic is testable via `Cart` alone, without instantiating `Checkout`
- Changing `CartItem` internals only requires updating `Cart`, not every type that touches it

---

## 5. Negative examples — what NOT to do

**Mistake 1: Exposing internals through a getter**
```go
// Not accepted — returning a pointer to the internal slice lets callers mutate it
func (cart *Cart) Items() *[]CartItem {
	return &cart.items
}
```

**Mistake 2: Breaking intimacy by adding more types that all know the internals**
```go
// Not accepted — DiscountApplier, TaxCalculator, and ShippingEstimator all
// reach into Cart's fields. The intimacy is multiplied, not resolved.
type DiscountApplier struct{}
func (d *DiscountApplier) Apply(cart *Cart) { cart.items[0].Price = 0 }
```

---

## 6. Benefits

- **Encapsulation:** Internal details stay private; callers use a stable public API
- **Independent testability:** Types can be tested without wiring up their intimate partners
- **Change safety:** Modifying internal layout of one type does not cascade to others
