# SKILL: Detecting and Refactoring Incomplete Library Class — Go

## Source
Based on: https://refactoring.guru/smells/incomplete-library-class

---

## 1. What is Incomplete Library Class

A type from an external package or the standard library lacks the behavior your code needs, but you cannot add methods to it directly (Go does not allow adding methods to types defined in other packages). The smell appears as ad-hoc workarounds scattered across your codebase.

**Why this happens:**
- Library authors cannot anticipate every use case
- Extending a library type forces workarounds instead of clean extensions
- Copy-paste adapter code accumulates in multiple places rather than being centralized

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] The same helper function appears in multiple packages to work around a library type's missing method
- [ ] A function accepts a library type and immediately wraps or adapts it before doing real work
- [ ] You find yourself writing `// wish time.Time had a method for this` style comments
- [ ] Utility functions like `formatDate`, `parseAmount`, `normalizeEmail` are scattered across packages
- [ ] The missing behavior is cohesive enough that it logically belongs on the external type

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Need to add methods to an external type | Define a local wrapper type (type alias approach) |
| Adapter logic repeated across many callers | Extract a standalone helper package with well-named functions |
| The external type is used everywhere with identical setup | Introduce a factory function that returns a configured instance |
| Behavior is a pure function of the external type's data | Standalone function in a focused package |

---

## 4. Example

**BEFORE — not accepted:**
```go
package invoice

import "time"

// The same date-formatting helper is copy-pasted into multiple packages
func formatDueDate(t time.Time) string {
	return t.Format("02 Jan 2006")
}

func PrintInvoice(issueDate, dueDate time.Time) {
	fmt.Printf("Issued: %s\n", formatDueDate(issueDate))
	fmt.Printf("Due:    %s\n", formatDueDate(dueDate))
}

// --- also in package report ---
// func formatDueDate(t time.Time) string { return t.Format("02 Jan 2006") }
```

**AFTER — expected:**
```go
// package timeutil — one place for time extensions
package timeutil

import "time"

// BillingDate wraps time.Time with billing-domain formatting.
type BillingDate struct{ time.Time }

func (d BillingDate) Display() string {
	return d.Format("02 Jan 2006")
}

func NewBillingDate(t time.Time) BillingDate {
	return BillingDate{t}
}
```

```go
package invoice

import "timeutil"

func PrintInvoice(issueDate, dueDate timeutil.BillingDate) {
	fmt.Printf("Issued: %s\n", issueDate.Display())
	fmt.Printf("Due:    %s\n", dueDate.Display())
}
```

**Why this pattern:**
- The wrapper type centralizes all billing-date behavior in one package
- `time.Time` methods are still accessible via embedding
- No more copy-pasted helpers; a single change propagates everywhere

---

## 5. Negative examples — what NOT to do

**Mistake 1: Scattering helpers across every package that needs them**
```go
// Not accepted — formatDate is now defined in 5 different packages with subtle variations
// invoice/util.go, report/util.go, export/util.go ...
func formatDate(t time.Time) string { return t.Format("02 Jan 2006") }
```

**Mistake 2: Using an empty interface to accept both the wrapper and the original**
```go
// Not accepted — loses all type safety
func PrintDate(d interface{}) string { ... }
```

---

## 6. Benefits

- **Single definition:** Extension behavior is defined once and imported everywhere
- **Discoverability:** A named wrapper type appears in IDE completions; a scattered helper does not
- **Consistency:** All callers use the same formatting or transformation logic
