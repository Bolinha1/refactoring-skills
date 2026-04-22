# SKILL: Detecting and Refactoring Lazy Class — Go

## Source
Based on: https://refactoring.guru/smells/lazy-class

---

## 1. What is Lazy Class

A type (struct, interface, or package) that does so little it no longer justifies its existence. It may have been meaningful during a previous design phase, or it was created speculatively and never grew into its intended role.

**Why this happens:**
- A type was created in anticipation of future complexity that never arrived
- Refactoring reduced a type's responsibilities but did not remove it
- A type was extracted from a larger one but turned out to hold only trivial delegation

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A struct has only one or two methods that each contain one line of logic
- [ ] A type exists only to call methods on another type (pure delegation)
- [ ] A package has a single exported function and nothing else
- [ ] An interface wraps only one method that maps 1-to-1 to a concrete type's method
- [ ] A type was kept "just in case" a feature was added but hasn't been touched in months

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Struct adds no behavior beyond delegating | Inline the type; use the dependency directly |
| Package has only one tiny function | Merge into the calling package |
| Interface used in only one place with one implementor | Remove the interface; use the concrete type |
| Type was created speculatively | Delete it |

---

## 4. Example

**BEFORE — not accepted:**
```go
package payment

// CurrencyFormatter does nothing beyond calling fmt.Sprintf — lazy type.
type CurrencyFormatter struct{}

func (f *CurrencyFormatter) Format(amount float64) string {
	return fmt.Sprintf("$%.2f", amount)
}

type Receipt struct {
	formatter *CurrencyFormatter
}

func (r *Receipt) Print(amount float64) {
	fmt.Println(r.formatter.Format(amount))
}
```

**AFTER — expected:**
```go
package payment

func formatCurrency(amount float64) string {
	return fmt.Sprintf("$%.2f", amount)
}

type Receipt struct{}

func (r *Receipt) Print(amount float64) {
	fmt.Println(formatCurrency(amount))
}
```

**Why this pattern:**
- `CurrencyFormatter` added no real abstraction — one unexported function replaces it
- `Receipt` is simpler: no field, no initialization, no dependency injection for a trivial helper
- If the formatting logic grows, extracting a type again is easy

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping a lazy type "for future extensibility"**
```go
// Not accepted — YAGNI applies; add the type when the complexity actually arrives
type CurrencyFormatter struct{} // "we might support multiple currencies someday"
```

**Mistake 2: Replacing a lazy struct with a lazy interface**
```go
// Not accepted — an interface with one implementation that does one thing is not better
type Formatter interface {
	Format(float64) string
}
```

---

## 6. Benefits

- **Simplicity:** Fewer types mean fewer files to navigate and fewer concepts to learn
- **Reduced overhead:** No constructor, no pointer, no dependency injection for trivial helpers
- **Cleaner package API:** Internal helpers are unexported functions, not exported types
