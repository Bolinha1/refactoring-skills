# SKILL: Detecting and Refactoring Duplicate Code — Go

## Source
Based on: https://refactoring.guru/smells/duplicate-code

---

## 1. What is Duplicate Code

Two or more code fragments look nearly identical or are structurally equivalent across different parts of the codebase. Any change to the logic must be replicated in every copy, and missing one creates a silent bug.

**Why this happens:**
- Copy-paste used as a shortcut instead of abstraction
- Parallel development where two developers solved the same problem independently
- Logic that started similar and diverged slightly, making the duplication invisible over time

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] The same or nearly identical block of code exists in two or more functions
- [ ] The same computation is repeated in several places
- [ ] Two structs contain methods that do essentially the same thing
- [ ] You copy-pasted code and tweaked one variable name
- [ ] A bug fix must be applied in more than one place

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Duplicate blocks in the same package | Extract Function |
| Duplicate logic across two structs | Move to a shared helper or standalone function |
| Near-identical methods on two types | Extract Interface + shared implementation |
| Duplicate logic differs only by a small variation | Parameterise the extracted function |
| Entire files look like copies | Consolidate into a single generic or parameterised implementation |

---

## 4. Example

**BEFORE — not accepted:**
```go
package report

type SalesReport struct{}

func (r *SalesReport) Revenue(orders []Order) float64 {
	total := 0.0
	for _, o := range orders {
		if o.Status == StatusPaid {
			total += o.Amount
		}
	}
	return total
}

type FinanceReport struct{}

func (r *FinanceReport) PaidTotal(orders []Order) float64 {
	total := 0.0
	for _, o := range orders {
		if o.Status == StatusPaid {
			total += o.Amount
		}
	}
	return total
}
```

**AFTER — expected:**
```go
package report

// sumPaidOrders is the single source of truth for paid-order aggregation.
func sumPaidOrders(orders []Order) float64 {
	total := 0.0
	for _, o := range orders {
		if o.Status == StatusPaid {
			total += o.Amount
		}
	}
	return total
}

type SalesReport struct{}

func (r *SalesReport) Revenue(orders []Order) float64 {
	return sumPaidOrders(orders)
}

type FinanceReport struct{}

func (r *FinanceReport) PaidTotal(orders []Order) float64 {
	return sumPaidOrders(orders)
}
```

**Why this pattern:**
- Business logic lives in one place — a single change fixes all callers
- Both report types delegate to the shared helper instead of owning the logic
- The unexported function is package-private, keeping the API surface small

---

## 5. Negative examples — what NOT to do

**Mistake 1: Accepting small differences as justification for duplication**
```go
// Not accepted — "almost the same" still means duplicate
func (r *SalesReport) Revenue(orders []Order) float64          { /* loop */ }
func (r *SalesReport) ProjectedRevenue(orders []Order) float64 { /* same loop + 1.1 multiplier */ }
```

**Mistake 2: Extracting the function but duplicating it in each file**
```go
// Not accepted — the helper itself is duplicated in sales.go and finance.go
func sumPaid(orders []Order) float64 { ... } // in sales.go
func sumPaid(orders []Order) float64 { ... } // in finance.go
```

**Mistake 3: Merging duplicates into a god function with boolean flags**
```go
// Not accepted — creates Long Method and obscures intent
func calculate(orders []Order, paid bool, projected bool, withTax bool) float64 { ... }
```

---

## 6. Benefits

- **Maintenance:** A single bug fix or rule change applies everywhere automatically
- **Clarity:** Code that expresses a shared concept is easier to understand than scattered copies
- **Trust:** Developers can be confident no diverged copies are hiding stale logic
