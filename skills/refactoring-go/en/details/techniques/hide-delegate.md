# TECHNIQUE: Hide Delegate — Go

## Source
Based on: https://refactoring.guru/hide-delegate

---

## 1. Problem

A client reaches through one object to call a method on another object it obtained from the first. The client is now coupled to both objects, so any change to the intermediate object's API ripples out to every caller.

---

## 2. Solution

Add wrapper methods on the outer struct so callers never see the delegate. Callers talk only to the outer struct, and the delegate becomes an implementation detail.

---

## 3. When to apply

- Callers chain calls like `employee.GetDepartment().GetManager()`
- The intermediate struct's type is exposed unnecessarily to the client
- Multiple callers reach through the same chain and you want one place to update it

---

## 4. Refactoring steps

1. Identify every method on the delegate that clients access through the owner
2. For each such method, create a wrapper method on the owner that delegates to it
3. Update each client call to use the wrapper method instead of chaining
4. If no client now accesses the delegate directly, remove the accessor that exposes it
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
    name       string
    department *Department
}

func (p *Person) Department() *Department { return p.department }

// Client code — coupled to Department
manager := employee.Department().Manager()
```

**AFTER — expected:**
```go
type Department struct {
    manager *Person
}

func (d *Department) Manager() *Person { return d.manager }

type Person struct {
    name       string
    department *Department
}

// Wrapper hides the delegate
func (p *Person) Manager() *Person {
    return p.department.Manager()
}

// Client code — only knows about Person
manager := employee.Manager()
```

**Why this pattern:**
- If `Department` is ever restructured, only `Person.Manager()` needs to change, not every caller

---

## 6. Negative examples — what NOT to do

**Mistake 1: Adding wrapper methods but still exposing the delegate accessor**
```go
// Not accepted — the delegate is still reachable; the wrapping is incomplete
func (p *Person) Department() *Department { return p.department } // still public
func (p *Person) Manager() *Person        { return p.department.Manager() }
```

**Mistake 2: Wrapping every single method of every embedded type**
```go
// Not accepted — mechanical wrapping without a real coupling problem
// only adds boilerplate; use Go embedding (`Department`) instead
```

---

## 7. Benefits

- **Reduced coupling:** Clients depend on fewer types
- **Encapsulation:** The delegate is an implementation detail that can change freely
- **Single change point:** Modifications to the delegate's API only affect the wrapper
