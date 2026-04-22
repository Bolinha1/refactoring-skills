# SKILL: Detecting and Refactoring Data Class — Go

## Source
Based on: https://refactoring.guru/smells/data-class

---

## 1. What is Data Class

A struct that contains only fields (and possibly simple getters/setters) but no real behavior. All the logic that operates on the struct lives in other packages, making the type a passive data bag rather than a domain object.

**Why this happens:**
- Developers from procedural backgrounds treat structs as plain records and put all logic in service types
- An entity was designed for serialization/transfer and never evolved to own its behavior
- Fear of "fat models" leads to pushing all logic into service layers

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A struct has only exported fields and no methods (or trivial accessor methods)
- [ ] All logic that uses the struct lives in external service or utility types
- [ ] The struct is passed around just to be read and written by others
- [ ] Methods elsewhere contain long chains of `obj.Field1`, `obj.Field2` accesses
- [ ] The struct name ends in `DTO`, `Data`, `Record`, `Info` but represents a domain concept

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Logic in another type that only uses this struct's data | Move Method onto the struct |
| Struct has fields only some methods use | Extract sub-struct for that responsibility |
| Struct is a true transfer object (across API boundary) | Keep it — DTOs are acceptable for I/O |
| Struct fields should be immutable after construction | Use unexported fields + constructor function |

---

## 4. Example

**BEFORE — not accepted:**
```go
package order

// Order is a pure data bag — all logic lives elsewhere.
type Order struct {
	Items    []Item
	Customer Customer
	Total    float64
	Paid     bool
}

// OrderService owns all the behavior
type OrderService struct{}

func (s *OrderService) CalculateTotal(o *Order) float64 {
	total := 0.0
	for _, item := range o.Items {
		total += item.Price * float64(item.Quantity)
	}
	return total
}

func (s *OrderService) IsEligibleForDiscount(o *Order) bool {
	return o.Total > 200 && !o.Paid
}
```

**AFTER — expected:**
```go
package order

type Order struct {
	items    []Item
	customer Customer
	paid     bool
}

func NewOrder(customer Customer, items []Item) *Order {
	return &Order{customer: customer, items: items}
}

func (o *Order) Total() float64 {
	total := 0.0
	for _, item := range o.items {
		total += item.Price * float64(item.Quantity)
	}
	return total
}

func (o *Order) IsEligibleForDiscount() bool {
	return o.Total() > 200 && !o.paid
}

func (o *Order) MarkPaid() {
	o.paid = true
}
```

**Why this pattern:**
- Behavior that only uses `Order` data now lives on `Order`
- Unexported fields enforce that state changes go through meaningful methods
- `OrderService` can focus on orchestration (persistence, notifications) rather than calculations

---

## 5. Negative examples — what NOT to do

**Mistake 1: Moving all logic to the struct regardless of dependencies**
```go
// Not accepted — logic that needs a DB or HTTP client does not belong on the domain struct
func (o *Order) SaveToDatabase(db *sql.DB) error { ... }
```

**Mistake 2: Treating every struct as a data class and piling logic into service types**
```go
// Not accepted — OrderValidator duplicates knowledge that Order already has
type OrderValidator struct{}
func (v *OrderValidator) Validate(o *Order) error {
	if len(o.Items) == 0 { ... }
}
```

---

## 6. Benefits

- **Encapsulation:** Internal state changes go through methods that enforce invariants
- **Cohesion:** Data and behavior that belong together live together
- **Reduced coupling:** External services no longer need to know the struct's internal layout
