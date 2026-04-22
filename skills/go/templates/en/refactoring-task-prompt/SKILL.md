# TEMPLATE: Refactoring Task Prompt

## How to use
Fill in the fields marked with `[...]` and use the result as a prompt
when requesting a refactoring from an LLM or describing a task to a colleague.

---

## Template

```
I need to refactor the following piece of code.

### Context
[Briefly describe what this code does and where it lives in the system.
E.g.: "Order processing function in the service layer of an e-commerce application."]

### Current code
[Paste the code that needs to be refactored here]

### Identified smell
[State the detected code smell. E.g.: Long Method, Large Class, Primitive Obsession]

### Recommended technique
[State the indicated refactoring technique. E.g.: Extract Method, Replace Type Code with Enum]

### Constraints
[List what must NOT change. E.g.:]
- The public function signature cannot change (used by N callers)
- There are no automated tests — refactoring must be conservative
- Language: Go [version]
- [Other relevant constraints]

### Expected result
[Describe how the code should look after refactoring. E.g.:]
- The main function should have at most 10 lines
- Each responsibility should be in a private function with an intent-expressing name
- No explanatory comments should be necessary
- Existing tests must continue to pass
```

---

## Filled examples

### Example 1 — Long Method with Extract Method

```
I need to refactor the following piece of code.

### Context
Function `ProcessOrder` in the `OrderService` struct of a Go e-commerce system.
Responsible for validating, calculating total, reserving stock, persisting and notifying the customer.

### Current code
func (s *OrderService) ProcessOrder(order *Order) error {
    if len(order.Items) == 0 {
        return fmt.Errorf("order has no items")
    }
    if order.Customer == nil {
        return fmt.Errorf("order has no customer")
    }

    total := 0.0
    for _, item := range order.Items {
        total += item.Price * float64(item.Quantity)
    }
    order.Total = total

    if err := s.stock.Reserve(order); err != nil {
        return fmt.Errorf("reserving stock: %w", err)
    }
    if err := s.repo.Save(order); err != nil {
        return fmt.Errorf("saving order: %w", err)
    }
    return s.email.SendConfirmation(order.Customer.Email)
}

### Identified smell
Long Method — the function has 20+ lines and does 4 distinct things (validate, calculate, persist, notify).

### Recommended technique
Extract Method

### Constraints
- The signature `ProcessOrder(order *Order) error` cannot change
- Language: Go 1.22
- Existing integration tests must continue to pass

### Expected result
- The main function should have at most 5 lines
- Each responsibility should become a private method with a business name
- No comments — the method name replaces them
- All errors must still be wrapped with context
```

---

### Example 2 — Primitive Obsession with Replace Data Value with Object

```
I need to refactor the following piece of code.

### Context
`Customer` struct in a Go CRM system. The tax ID is used as `string` throughout the system
and its validation is duplicated in 6 different places.

### Current code
type Customer struct {
    Name  string
    TaxID string
    Email string
}

### Identified smell
Primitive Obsession — `TaxID` and `Email` are raw strings with no centralized validation.

### Recommended technique
Replace Data Value with Object

### Constraints
- Go 1.22
- Must not break the public API of the Customer struct
- New value types must be immutable (no exported setters)

### Expected result
- `TaxID` as an immutable type with validation in a constructor func `NewTaxID`
- `Email` as an immutable type with validation in a constructor func `NewEmail`
- `Customer` fields use the new types
- Validation does not repeat anywhere else
```

---

## Usage tips

- The more specific the context, the better the suggested refactoring
- Always state the Go version — standard library and idioms evolve between versions
- Listing constraints avoids suggestions that break existing contracts
- If you don't know the exact technique, describe the problem and ask for a diagnosis before refactoring
- Mention whether the code is concurrent — goroutines and channels change what refactorings are safe
