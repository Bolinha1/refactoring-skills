# SKILL: Detecting and Refactoring Temporary Field — Go

## Source
Based on: https://refactoring.guru/smells/temporary-field

---

## 1. What is it?

A struct contains a field that is only set and used during one method call or algorithm.
The rest of the time it is zero-valued or meaningless. Other code that reads the field
cannot tell when it holds a valid value and when it doesn't.

---

## 2. Warning signs

- [ ] A field is zero or `nil` most of the time
- [ ] A field is set in one method and consumed in another, but never held across the struct's lifetime
- [ ] A field exists only because a long algorithm needed somewhere to store intermediate state
- [ ] The struct has a "prepare then execute" pattern where fields are filled right before use
- [ ] Nil/zero checks for struct fields appear throughout the receiver methods

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Extract Class (Extract Struct)** | Move the temporary fields and the methods that use them into a dedicated struct |
| **Introduce Null Object** | Replace the zero/nil state with a proper value so callers don't need to check |

---

## 4. Example

**BEFORE — not accepted:**
```go
// RouteCalculator holds fields that are only valid during Calculate()
type RouteCalculator struct {
	stops         []string // only set during calculation
	totalDistance int      // only valid after Calculate()
	fastestPath   string   // empty unless Calculate() ran
}

func (r *RouteCalculator) SetStops(stops []string) {
	r.stops = stops
}

func (r *RouteCalculator) Calculate() {
	r.totalDistance = computeDistance(r.stops)
	r.fastestPath = findFastestPath(r.stops)
}

func (r *RouteCalculator) FastestPath() string {
	return r.fastestPath // empty string if Calculate() wasn't called first
}
```

**AFTER — expected:**
```go
// RouteResult only exists when it holds valid data
type RouteResult struct {
	TotalDistance int
	FastestPath   string
}

// RouteCalculator is now stateless
type RouteCalculator struct{}

func (r *RouteCalculator) Calculate(stops []string) RouteResult {
	return RouteResult{
		TotalDistance: computeDistance(stops),
		FastestPath:   findFastestPath(stops),
	}
}
```

**Why this pattern:**
- `RouteResult` only exists when it holds valid data — no zero/nil state to guard against
- `RouteCalculator` is stateless: safe to reuse and test concurrently

---

## 5. Negative examples — what NOT to do

**Mistake 1: Adding a `ready` boolean flag instead of fixing the design**
```go
// Not accepted — flag discipline is fragile; Extract Struct removes the need for it
type RouteCalculator struct {
	calculated  bool
	fastestPath string
}

func (r *RouteCalculator) FastestPath() string {
	if !r.calculated {
		panic("call Calculate() first")
	}
	return r.fastestPath
}
```

**Mistake 2: Moving the temporary field to an embedded struct**
```go
// Not accepted — the field still travels with the outer struct; the smell remains
type calculationState struct {
	fastestPath string
}

type RouteCalculator struct {
	calculationState // same problem, just renamed
}
```

---

## 6. Benefits

- **Correctness:** No zero state masquerading as a valid value
- **Clarity:** Every struct field is meaningful regardless of when you read it
- **Concurrency:** Stateless calculators can be called from multiple goroutines safely
