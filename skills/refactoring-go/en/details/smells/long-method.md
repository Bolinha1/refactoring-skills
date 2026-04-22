# SKILL: Detecting and Refactoring Long Method — Go

## Source
Based on: https://refactoring.guru/smells/long-method

---

## 1. What is Long Method

A function or method that has grown too long to be understood at a glance. Long functions accumulate multiple levels of abstraction, making them difficult to read, test, and change safely.

**Why this happens:**
- It is mentally easier to add a few lines to an existing function than to create a new one
- A function acquires responsibility over time through a series of small, individually reasonable additions
- Developers avoid the overhead of naming sub-steps and instead inline everything

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A function exceeds 20-30 lines
- [ ] A function requires scrolling to read entirely
- [ ] A comment sits above a block of code inside the function, naming what the block does
- [ ] A function has more than two levels of indentation (nested loops + conditionals)
- [ ] A function handles both high-level orchestration and low-level detail
- [ ] Testing the function requires constructing a large context to exercise one branch

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Named block inside the function | Extract Function |
| Conditional logic that can be named | Extract Function returning bool |
| Loop body is complex | Extract Function for the loop body |
| Temporary variables used only in one section | Extract Function, pass only what it needs |
| Function both orchestrates and implements | Separate orchestration from implementation |

---

## 4. Example

**BEFORE — not accepted:**
```go
package order

func (s *Service) PlaceOrder(customerID int, items []Item, promoCode string) error {
	// validate
	if customerID <= 0 {
		return fmt.Errorf("invalid customer")
	}
	if len(items) == 0 {
		return fmt.Errorf("no items")
	}

	// calculate total
	total := 0.0
	for _, item := range items {
		if item.Quantity <= 0 {
			return fmt.Errorf("invalid quantity for %s", item.Name)
		}
		total += item.Price * float64(item.Quantity)
	}

	// apply promotion
	if promoCode == "SUMMER20" {
		total *= 0.80
	} else if promoCode == "VIP10" {
		total *= 0.90
	}

	// persist
	o := &Order{CustomerID: customerID, Items: items, Total: total}
	if err := s.repo.Save(o); err != nil {
		return fmt.Errorf("save order: %w", err)
	}

	// notify
	customer, err := s.customers.Find(customerID)
	if err != nil {
		return err
	}
	return s.mailer.Send(customer.Email, "Order confirmed", fmt.Sprintf("Total: %.2f", total))
}
```

**AFTER — expected:**
```go
package order

func (s *Service) PlaceOrder(customerID int, items []Item, promoCode string) error {
	if err := s.validate(customerID, items); err != nil {
		return err
	}
	total := s.calculateTotal(items, promoCode)
	o := &Order{CustomerID: customerID, Items: items, Total: total}
	if err := s.repo.Save(o); err != nil {
		return fmt.Errorf("save order: %w", err)
	}
	return s.notifyCustomer(customerID, total)
}

func (s *Service) validate(customerID int, items []Item) error {
	if customerID <= 0 {
		return fmt.Errorf("invalid customer")
	}
	for _, item := range items {
		if item.Quantity <= 0 {
			return fmt.Errorf("invalid quantity for %s", item.Name)
		}
	}
	if len(items) == 0 {
		return fmt.Errorf("no items")
	}
	return nil
}

func (s *Service) calculateTotal(items []Item, promoCode string) float64 {
	total := 0.0
	for _, item := range items {
		total += item.Price * float64(item.Quantity)
	}
	return applyPromo(total, promoCode)
}

func applyPromo(total float64, code string) float64 {
	switch code {
	case "SUMMER20":
		return total * 0.80
	case "VIP10":
		return total * 0.90
	}
	return total
}

func (s *Service) notifyCustomer(customerID int, total float64) error {
	customer, err := s.customers.Find(customerID)
	if err != nil {
		return err
	}
	return s.mailer.Send(customer.Email, "Order confirmed", fmt.Sprintf("Total: %.2f", total))
}
```

**Why this pattern:**
- `PlaceOrder` reads like a table of contents — each extracted function is a named step
- Each helper can be tested independently without the full order-placement context
- The `applyPromo` function is reusable and easy to extend

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting into a single helper that is still too long**
```go
// Not accepted — one big helper is not better than one big function
func (s *Service) processOrderDetails(customerID int, items []Item, promoCode string) error {
	// 40 lines of mixed concerns
}
```

**Mistake 2: Adding comments instead of extracting**
```go
// Not accepted — comments name what the code does, which is the job of function names
// Step 1: validate input
if customerID <= 0 { ... }
// Step 2: calculate total
total := 0.0 ...
```

---

## 6. Benefits

- **Readability:** Short, named functions read like prose
- **Testability:** Each extracted function can be unit-tested directly
- **Reusability:** Helpers like `applyPromo` surface as candidates for reuse
