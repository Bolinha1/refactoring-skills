# SKILL: Detecting and Refactoring Comments — Go

## Source
Based on: https://refactoring.guru/smells/comments

---

## 1. What is the Comments smell

Comments that explain *what* the code does rather than *why* it does it. When code needs a prose annotation to be understood, it usually means the code itself is unclear — the comment is a deodorant masking a deeper problem.

**Why this happens:**
- Complex logic was written quickly and a comment was added instead of simplifying
- Method and variable names are too short or generic to convey intent
- Business rules buried inside low-level loops are invisible without explanation

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A comment restates what the next line of code already says
- [ ] A block of code is preceded by a comment that names what it does (extract-method candidate)
- [ ] A comment explains a magic number or string literal instead of naming a constant
- [ ] Commented-out code blocks exist with no explanation of why they were kept
- [ ] Every function has a doc-comment that only paraphrases the function signature
- [ ] A comment apologizes for confusing code ("// tricky — don't touch")

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Comment names a block of code | Extract Function with a descriptive name |
| Comment explains a magic value | Introduce Explaining Variable or named constant |
| Comment explains a complex condition | Extract Function that returns a named boolean |
| Comment is an apology for messy code | Refactor the code, then delete the comment |
| Comment explains *why* a non-obvious decision was made | Keep it — this is the one good use of comments |

---

## 4. Example

**BEFORE — not accepted:**
```go
package order

func (o *Order) Process() error {
	// check if order has items
	if len(o.Items) == 0 {
		return fmt.Errorf("order has no items")
	}

	// calculate total
	total := 0.0
	for _, item := range o.Items {
		total += item.Price * float64(item.Quantity)
	}

	// apply 10% discount for orders over 100
	if total > 100 {
		total = total * 0.90
	}

	o.Total = total
	return nil
}
```

**AFTER — expected:**
```go
package order

const bulkDiscountThreshold = 100.0
const bulkDiscountRate = 0.90

func (o *Order) Process() error {
	if err := o.validate(); err != nil {
		return err
	}
	o.Total = o.calculateTotal()
	return nil
}

func (o *Order) validate() error {
	if len(o.Items) == 0 {
		return fmt.Errorf("order has no items")
	}
	return nil
}

func (o *Order) calculateTotal() float64 {
	subtotal := o.sumItems()
	if o.qualifiesForBulkDiscount(subtotal) {
		return subtotal * bulkDiscountRate
	}
	return subtotal
}

func (o *Order) sumItems() float64 {
	total := 0.0
	for _, item := range o.Items {
		total += item.Price * float64(item.Quantity)
	}
	return total
}

func (o *Order) qualifiesForBulkDiscount(total float64) bool {
	return total > bulkDiscountThreshold
}
```

**Why this pattern:**
- Each extracted function's name replaces its former comment
- Named constants replace magic numbers — no comment needed
- The one comment worth keeping is a *why* comment, e.g., `// Bulk discount per commercial agreement v3`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Doc-comment that mirrors the signature**
```go
// Not accepted — the comment adds no information beyond the name
// GetUserByID returns a user by their ID.
func GetUserByID(id int) (*User, error) { ... }
```

**Mistake 2: Keeping commented-out code "just in case"**
```go
// Not accepted — use version control; dead code confuses readers
// total = total * 0.85  // old discount rate
total = total * 0.90
```

---

## 6. Benefits

- **Readability:** Self-documenting code is faster to read than code + prose
- **Maintainability:** Renaming a function updates all call sites; a comment silently rots
- **Go convention:** Exported symbols get a single doc-comment above them — not inline annotations
