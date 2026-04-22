# TECHNIQUE: Introduce Parameter Object — Go

## Source
Based on: https://refactoring.guru/introduce-parameter-object

---

## 1. Problem

Several functions share the same group of parameters. Passing them individually is verbose, error-prone, and makes signatures hard to read. When a new related parameter must be added, every function signature must change.

---

## 2. Solution

Group the related parameters into a new struct. Pass the struct instead of the individual values. Functions that previously shared those parameters now share the struct type.

---

## 3. When to apply

- Two or more functions accept the same set of parameters repeatedly
- Parameters always travel together and represent a single concept (e.g., a date range, a search filter)
- Adding a new parameter in the group would require updating many function signatures
- The group of parameters has validation or behaviour that currently lives scattered in the callers

---

## 4. Refactoring steps

1. Create a new struct with fields for each parameter in the group
2. Update each function signature to accept the struct instead of the individual parameters
3. Update all call sites to construct and pass the struct
4. Move any repeated validation or derived calculations into methods on the new struct
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func (r *Report) Generate(startDate, endDate time.Time, minAmount float64) []Entry {
    // ...
}

func (r *Report) Summary(startDate, endDate time.Time, minAmount float64) string {
    // ...
}
```

**AFTER — expected:**
```go
type ReportParams struct {
    StartDate time.Time
    EndDate   time.Time
    MinAmount float64
}

func (p ReportParams) IsValid() bool {
    return p.EndDate.After(p.StartDate) && p.MinAmount >= 0
}

func (r *Report) Generate(params ReportParams) []Entry {
    // ...
}

func (r *Report) Summary(params ReportParams) string {
    // ...
}
```

**Why this pattern:**
- Adding a `MaxAmount` field only changes `ReportParams`, not every function signature
- Validation is centralised in `ReportParams.IsValid()` instead of duplicated in callers

---

## 6. Negative examples — what NOT to do

**Mistake 1: Creating a parameter object for unrelated parameters**
```go
// Not accepted — name, price, and userID do not form a coherent concept
type Params struct {
    Name   string
    Price  float64
    UserID int
}
```

**Mistake 2: Adding every possible parameter to the struct "just in case"**
```go
// Not accepted — a catch-all struct becomes a dump for everything;
// only group parameters that naturally belong together
type EverythingParams struct {
    StartDate  time.Time
    EndDate    time.Time
    UserID     int
    Format     string
    DebugMode  bool
    // ...
}
```

---

## 7. Benefits

- **Shorter signatures:** Functions become easier to read and call
- **Single change point:** Adding a related parameter touches only the struct
- **Cohesion:** Validation and behaviour for the parameter group live together
- **Reuse:** The struct can be passed through multiple layers without re-listing parameters
