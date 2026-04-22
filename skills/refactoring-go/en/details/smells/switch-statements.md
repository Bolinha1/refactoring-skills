# SKILL: Detecting and Refactoring Switch Statements — Go

## Source
Based on: https://refactoring.guru/smells/switch-statements

---

## 1. What is Switch Statements

Complex `switch` or `if/else` chains that branch on the type or state of a value.
The same switch logic tends to be duplicated across the codebase — when a new case
is added, every switch must be found and updated.

In Go this also manifests as type switches (`switch v := x.(type)`) that should be
replaced by interface dispatch.

**Why this happens:**
- Type-based behaviour was implemented procedurally instead of through interfaces
- A string or integer constant was used to encode a "type" instead of a Go interface
- A single struct grew to handle multiple variants of a concept

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `switch` or `if/else if` block branching on a type field, status string, or iota constant
- [ ] A type switch (`switch v := x.(type)`) with business logic in each case
- [ ] The same switch appears in more than one place
- [ ] Adding a new "type" requires searching for and updating every switch
- [ ] Each branch does something very different from the others

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Switch on type to decide behaviour | Replace Conditional with Polymorphism (interface) |
| Switch on type to decide which struct to create | Factory function returning an interface |
| Only a few cases and adding new ones is rare | Leave it — not every switch is a smell |
| Type switch on a concrete type field | Replace the field with an interface |

---

## 4. Example

**BEFORE — not accepted:**
```go
type ShippingType string

const (
	Standard  ShippingType = "standard"
	Express   ShippingType = "express"
	Overnight ShippingType = "overnight"
)

func calculateShipping(order Order) float64 {
	switch order.ShippingType {
	case Standard:
		return order.Weight * 1.5
	case Express:
		return order.Weight*3.0 + 5.0
	case Overnight:
		return order.Weight*5.0 + 20.0
	default:
		panic("unknown shipping type")
	}
}
```

**AFTER — expected:**
```go
type ShippingStrategy interface {
	Cost(weight float64) float64
}

type StandardShipping struct{}
func (s StandardShipping) Cost(weight float64) float64 { return weight * 1.5 }

type ExpressShipping struct{}
func (e ExpressShipping) Cost(weight float64) float64 { return weight*3.0 + 5.0 }

type OvernightShipping struct{}
func (o OvernightShipping) Cost(weight float64) float64 { return weight*5.0 + 20.0 }

type Order struct {
	Weight   float64
	Shipping ShippingStrategy
}

func (o *Order) ShippingCost() float64 {
	return o.Shipping.Cost(o.Weight)
}
```

**Why this pattern:**
- Adding a new shipping type requires only a new struct implementing `ShippingStrategy`
- No existing code changes; no switch to hunt down
- Each strategy is independently testable

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the switch with an if/else chain**
```go
// Not accepted — same smell, different syntax
if order.ShippingType == Standard { ... } else if order.ShippingType == Express { ... }
```

**Mistake 2: Centralizing into a factory without interface dispatch**
```go
// Not accepted — the switch still exists, just moved
func newShipping(t ShippingType) ShippingStrategy {
	switch t { ... } // same problem
}
```

**Mistake 3: Applying polymorphism to a boolean flag**
```go
// Not accepted — two structs for true/false is over-engineering
switch order.IsPriority { ... }
```

---

## 6. Benefits

- **Open/Closed:** New variants require a new struct, not editing existing ones
- **Locality:** Each variant's behaviour is in one place
- **Testability:** Each interface implementation can be tested in isolation
