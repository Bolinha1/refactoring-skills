# TECHNIQUE: Remove Middle Man — Go

## Source
Based on: https://refactoring.guru/remove-middle-man

---

## 1. Problem

A struct has grown a collection of wrapper methods that do nothing but forward calls to a delegate. Every time the delegate adds a new method, the middle-man struct must add a corresponding forwarder. The wrapper adds indirection without adding value.

---

## 2. Solution

Remove the forwarding methods from the middle-man struct. Expose the delegate directly (or make it accessible) so callers can call it themselves.

---

## 3. When to apply

- Most of a struct's public methods are simple one-line delegations to the same internal struct
- The middle-man struct adds no logic, transformation, or protection — it just routes calls
- The wrapper was created by a previous "Hide Delegate" refactoring that is no longer justified

---

## 4. Refactoring steps

1. Expose an accessor for the delegate on the middle-man struct (if not already accessible)
2. For each forwarding method, find all callers and update them to call the delegate directly
3. Remove the forwarding methods from the middle-man struct one by one
4. If the middle-man struct now has no remaining responsibilities, consider removing it entirely
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Department struct {
    manager *Person
}

func (d *Department) Manager() *Person { return d.manager }

type Person struct {
    department *Department
}

// Pure forwarding — adds nothing
func (p *Person) Manager() *Person {
    return p.department.Manager()
}

// Caller
manager := employee.Manager()
```

**AFTER — expected:**
```go
type Department struct {
    manager *Person
}

func (d *Department) Manager() *Person { return d.manager }

type Person struct {
    Department *Department // now exported so callers can reach through
}

// Caller reaches the delegate directly
manager := employee.Department.Manager()
```

**Why this pattern:**
- `Person` no longer needs to be updated every time `Department` adds a method
- The relationship between the caller and `Department` is explicit

---

## 6. Negative examples — what NOT to do

**Mistake 1: Removing the middle man when it adds real value**
```go
// Not accepted — if Person.Manager() includes access control,
// caching, or logging, it is NOT a pure middle man; keep it
func (p *Person) Manager() *Person {
    p.auditAccess("manager")
    return p.department.Manager()
}
```

**Mistake 2: Exposing the delegate and keeping the forwarding methods**
```go
// Not accepted — now callers have two ways to do the same thing;
// remove the forwarders once the delegate is accessible
func (p *Person) Manager() *Person { return p.Department.Manager() } // redundant
```

---

## 7. Benefits

- **Less boilerplate:** No forwarding methods to maintain
- **Transparency:** The relationship between caller and delegate is explicit
- **Easier evolution:** Adding methods to the delegate does not require touching the middle man
