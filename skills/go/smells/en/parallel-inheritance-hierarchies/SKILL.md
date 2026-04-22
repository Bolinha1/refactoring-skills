# SKILL: Detecting and Refactoring Parallel Inheritance Hierarchies — Go

## Source
Based on: https://refactoring.guru/smells/parallel-inheritance-hierarchies

---

## 1. What is Parallel Inheritance Hierarchies

Two families of types mirror each other: every time a new type is added to one family, a corresponding type must be added to the other. In Go this manifests as parallel sets of structs and interfaces that must always be extended in lockstep.

**Why this happens:**
- Behavior and data were separated into parallel type trees instead of being co-located
- Strategy or visitor types were created for each domain type individually
- A code generator pattern was introduced without considering the long-term maintenance burden

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Adding a new domain type (e.g., `CryptoPayment`) requires adding a parallel type in another package (e.g., `CryptoPaymentSerializer`)
- [ ] Two sets of types have names that mirror each other: `FooProcessor`, `BarProcessor` alongside `Foo`, `Bar`
- [ ] A switch or type-assertion block in one family mirrors a switch in the other family
- [ ] The two families always change together but live in separate packages

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Behavior can be moved onto the domain type | Move Method — collapse the parallel hierarchy |
| Both hierarchies share a common abstraction | Merge into one interface with the behavior included |
| Separation is required (e.g., serialization layer) | Use a single generic function or interface, not parallel types |
| One hierarchy is pure data | Use a data-driven approach (maps, tables) instead of types |

---

## 4. Example

**BEFORE — not accepted:**
```go
package payment

// Domain hierarchy
type CreditCardPayment struct{ Amount float64 }
type BankTransferPayment struct{ Amount float64 }

// Parallel serializer hierarchy — mirrors the domain 1-to-1
type CreditCardSerializer struct{}
type BankTransferSerializer struct{}

func (s *CreditCardSerializer) Serialize(p *CreditCardPayment) string {
	return fmt.Sprintf("cc:%.2f", p.Amount)
}

func (s *BankTransferSerializer) Serialize(p *BankTransferPayment) string {
	return fmt.Sprintf("bt:%.2f", p.Amount)
}

// Adding WirePayment requires both WirePayment AND WirePaymentSerializer
```

**AFTER — expected:**
```go
package payment

// A single interface carries both the data and the serialization behavior.
type Payment interface {
	Serialize() string
}

type CreditCardPayment struct{ Amount float64 }

func (p *CreditCardPayment) Serialize() string {
	return fmt.Sprintf("cc:%.2f", p.Amount)
}

type BankTransferPayment struct{ Amount float64 }

func (p *BankTransferPayment) Serialize() string {
	return fmt.Sprintf("bt:%.2f", p.Amount)
}

// Adding WirePayment requires only one new type — no parallel hierarchy.
```

**Why this pattern:**
- Each payment type owns its serialization — no separate serializer family to maintain
- Adding a new payment type requires exactly one new struct, not two
- The `Payment` interface is the single abstraction callers depend on

---

## 5. Negative examples — what NOT to do

**Mistake 1: Solving with a type switch (which must be updated in each parallel family)**
```go
// Not accepted — the switch must be duplicated in every parallel family member
func Serialize(p interface{}) string {
	switch v := p.(type) {
	case *CreditCardPayment:  return fmt.Sprintf("cc:%.2f", v.Amount)
	case *BankTransferPayment: return fmt.Sprintf("bt:%.2f", v.Amount)
	}
	return ""
}
```

**Mistake 2: Using code generation to automate the parallel hierarchy**
```go
// Not accepted — generation hides the smell but does not remove it;
// the maintenance burden just moves to the generator template
```

---

## 6. Benefits

- **Locality:** Each type carries its own behavior — one place to look
- **Scalability:** Adding a new variant requires one new type, not two
- **Interface stability:** Callers depend on `Payment`, not on parallel serializer types
