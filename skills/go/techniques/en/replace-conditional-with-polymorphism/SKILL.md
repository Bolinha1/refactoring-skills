# TECHNIQUE: Replace Conditional with Polymorphism — Go

## Source
Based on: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problem

You have a `switch` or `if/else` chain that performs different behaviour depending on a type field or a string that simulates a type. The same conditional tends to appear in multiple places and must be updated whenever a new type is added.

---

## 2. Solution

Define a Go interface with the method to be polymorphized. Create a concrete struct for each case that implements the interface. Replace every conditional with an interface method call.

---

## 3. When to apply

- The conditional checks a type field, a string constant, or uses type assertions
- The same conditional appears in multiple methods
- Adding a new case would require modifying all existing conditionals (Open/Closed Principle violation)
- Each branch represents a cohesive and distinct behaviour

---

## 4. Refactoring steps

1. If the conditional is inside a method with other responsibilities, apply Extract Method first
2. Define an interface with the method to be polymorphized
3. For each branch, create a struct that implements the interface with that branch's logic
4. Update the code that constructs the original type to construct the appropriate concrete struct instead
5. Remove the conditional; the interface dispatch now selects the right behaviour
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Employee struct {
    Type       string // "ENGINEER", "MANAGER", "SALESPERSON"
    BaseSalary float64
    Sales      float64
}

func (e *Employee) CalculateBonus() float64 {
    switch e.Type {
    case "ENGINEER":
        return e.BaseSalary * 0.10
    case "MANAGER":
        return e.BaseSalary*0.20 + 1000
    case "SALESPERSON":
        return e.Sales * 0.05
    default:
        panic("unknown type: " + e.Type)
    }
}

func (e *Employee) GenerateReport() string {
    switch e.Type {
    case "ENGINEER":
        return "Engineer: delivered projects"
    case "MANAGER":
        return "Manager: led teams"
    case "SALESPERSON":
        return fmt.Sprintf("Salesperson: $%.2f in sales", e.Sales)
    default:
        panic("unknown type: " + e.Type)
    }
}
```

**AFTER — expected:**
```go
type Employee interface {
    CalculateBonus() float64
    GenerateReport() string
}

type Engineer struct{ BaseSalary float64 }

func (e *Engineer) CalculateBonus() float64  { return e.BaseSalary * 0.10 }
func (e *Engineer) GenerateReport() string   { return "Engineer: delivered projects" }

type Manager struct{ BaseSalary float64 }

func (m *Manager) CalculateBonus() float64  { return m.BaseSalary*0.20 + 1000 }
func (m *Manager) GenerateReport() string   { return "Manager: led teams" }

type Salesperson struct{ Sales float64 }

func (s *Salesperson) CalculateBonus() float64 { return s.Sales * 0.05 }
func (s *Salesperson) GenerateReport() string {
    return fmt.Sprintf("Salesperson: $%.2f in sales", s.Sales)
}
```

**Why this pattern:**
- Adding a new employee type requires only a new struct — no existing code changes
- The same `switch` does not have to be repeated across `CalculateBonus` and `GenerateReport`
- Each struct is responsible only for its own behaviour

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the switch to a factory and calling it solved**
```go
// Not accepted — the switch still exists, just relocated; the problem remains
func NewEmployee(empType string) Employee {
    switch empType {
    case "ENGINEER": return &Employee{Type: "ENGINEER"} // same struct, no polymorphism
    }
}
```

**Mistake 2: Creating structs but keeping the conditional inside them**
```go
// Not accepted — polymorphism was not applied; the conditional is just moved
func (e *Engineer) CalculateBonus() float64 {
    if e.empType == "ENGINEER" { // unnecessary; the type is already Engineer
        return e.BaseSalary * 0.10
    }
    return 0
}
```

**Mistake 3: Using interfaces for trivial two-branch conditionals**
```go
// Not accepted — over-engineering a simple boolean check
// e.g.: if active { doX() } else { doY() }
```

---

## 7. Benefits

- **Open/Closed Principle:** New types require only a new struct, not changes to existing ones
- **Duplicate elimination:** The type-based switch is not repeated across methods
- **Readability:** Each struct clearly expresses its own behaviour
- **Testability:** Each concrete struct can be tested independently
