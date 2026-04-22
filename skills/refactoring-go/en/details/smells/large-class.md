# SKILL: Detecting and Refactoring Large Class — Go

## Source
Based on: https://refactoring.guru/smells/large-class

---

## 1. What is Large Class

A struct that has too many fields, methods, or lines of code. When a type tries to do too many things it accumulates responsibilities that should be distributed across focused types and packages.

**Why this happens:**
- Structs start small and grow as the program evolves
- It is mentally easier to add a method to an existing struct than to create a new one
- The accumulation is gradual — nobody notices until the struct has become a monolith

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A struct file exceeds 200 lines
- [ ] A struct has more than 10 methods
- [ ] A struct has more than 5 fields
- [ ] The struct has clearly distinct responsibilities (SRP violation)
- [ ] The struct is modified for different reasons by different teams
- [ ] Many imports in the file are unrelated to each other
- [ ] The struct name is too generic (`Manager`, `Processor`, `Handler`, `Service`)
- [ ] Some fields are only used by a subset of the methods

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Behaviors groupable into an autonomous unit | Extract Struct (new focused type) |
| Specialized or rarely-used functionality | Move to a separate package |
| Clients use only part of the interface | Extract Interface |
| Fields only used by some methods | Extract sub-struct, embed or reference |

**Golden rule:** If you can describe the struct using "and" more than twice ("this type validates orders *and* calculates freight *and* sends email *and* generates PDFs"), it has too many responsibilities.

---

## 4. Example

**BEFORE — not accepted:**
```go
package app

type OrderService struct {
	db      *sql.DB
	mailer  Mailer
	stock   StockService
	invoice InvoiceService
}

func (s *OrderService) Create(o *Order) error {
	if len(o.Items) == 0 {
		return fmt.Errorf("no items")
	}
	total := 0.0
	for _, item := range o.Items {
		total += item.Price * float64(item.Quantity)
		if err := s.stock.Reserve(item.ProductID, item.Quantity); err != nil {
			return err
		}
	}
	o.Total = total
	if err := s.db.Save(o); err != nil {
		return err
	}
	inv := s.invoice.Generate(o)
	o.Invoice = inv
	return s.mailer.SendConfirmation(o.Customer.Email, o)
}

func (s *OrderService) Cancel(o *Order) error          { /* extensive logic */ return nil }
func (s *OrderService) RecalcShipping(o *Order) error  { /* extensive logic */ return nil }
func (s *OrderService) ApplyCoupon(o *Order, c string) error { /* extensive logic */ return nil }
func (s *OrderService) FindByCustomer(id int) ([]*Order, error) { return nil, nil }
func (s *OrderService) GenerateReport(orders []*Order) []byte  { return nil }
```

**AFTER — expected:**
```go
package order

type Validator struct{}

func (v *Validator) Validate(o *Order) error {
	if len(o.Items) == 0 {
		return fmt.Errorf("no items")
	}
	return nil
}

type TotalCalculator struct{}

func (c *TotalCalculator) Calculate(o *Order) float64 {
	total := 0.0
	for _, item := range o.Items {
		total += item.Price * float64(item.Quantity)
	}
	return total
}

type Repository struct{ db *sql.DB }

func (r *Repository) Save(o *Order) error { return r.db.Save(o) }

type Service struct {
	validator  *Validator
	calculator *TotalCalculator
	repo       *Repository
	stock      StockService
	notifier   Notifier
}

func (s *Service) Create(o *Order) error {
	if err := s.validator.Validate(o); err != nil {
		return err
	}
	o.Total = s.calculator.Calculate(o)
	if err := s.repo.Save(o); err != nil {
		return err
	}
	return s.notifier.SendConfirmation(o)
}
```

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting a struct without cohesion**
```go
// Not accepted — the extracted type still mixes responsibilities
type OrderHelper struct{}
func (h *OrderHelper) Validate(o *Order) error      { ... }
func (h *OrderHelper) CalcShipping(o *Order) error  { ... }
func (h *OrderHelper) GenerateReport(os []*Order) []byte { ... }
```

**Mistake 2: Generic names for extracted types**
```go
// Not accepted
type OrderUtils struct{}
type OrderManager struct{}
type OrderHelper2 struct{}
```

**Mistake 3: Splitting into files but keeping one massive struct**
```go
// Not accepted — files are cosmetic; the struct still has 30 methods
// order_create.go, order_cancel.go, order_report.go — all with methods on *OrderService
```

---

## 6. Benefits

- **Cognitive load:** Developers do not need to hold an entire monolith in their heads
- **Duplicate elimination:** Splitting large structs frequently eliminates redundant code
- **Maintainability:** Each type with a single responsibility is easier to test and modify
- **Reusability:** Smaller, focused types are easier to reuse in other contexts
