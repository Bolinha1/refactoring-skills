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
E.g.: "Order processing method in the service layer of a PHP e-commerce application."]

### Current code
[Paste the code that needs to be refactored here]

### Identified smell
[State the detected code smell. E.g.: Long Method, Large Class, Primitive Obsession]

### Recommended technique
[State the indicated refactoring technique. E.g.: Extract Method, Replace Type Code with Enum]

### Constraints
[List what must NOT change. E.g.:]
- The public method signature cannot change (used by N callers)
- There are no automated tests — refactoring must be conservative
- Language is PHP version [X]
- [Other relevant constraints]

### Expected result
[Describe how the code should look after refactoring. E.g.:]
- The main method should have at most 10 lines
- Each responsibility should be in a private method with an intent-expressing name
- No explanatory comments should be necessary
- Existing tests must continue to pass
```

---

## Filled examples

### Example 1 — Long Method with Extract Method

```
I need to refactor the following piece of code.

### Context
Method `processOrder` in the `OrderService` class of a PHP e-commerce system.
Responsible for validating, calculating total, reserving stock, persisting and notifying the customer.

### Current code
public function processOrder(Order $order): void
{
    if (empty($order->getItems())) throw new \RuntimeException("No items");
    if ($order->getCustomer() === null) throw new \RuntimeException("No customer");

    $total = 0;
    foreach ($order->getItems() as $item) {
        $total += $item->getPrice() * $item->getQuantity();
    }
    $order->setTotal($total);

    $this->stockService->reserve($order);
    $this->repository->save($order);
    $this->emailService->sendConfirmation($order->getCustomer()->getEmail());
}

### Identified smell
Long Method — the method has 14 lines and does 4 distinct things (validate, calculate, persist, notify).

### Recommended technique
Extract Method

### Constraints
- The signature `processOrder(Order $order): void` cannot change
- PHP 8.2
- Existing integration tests must continue to pass

### Expected result
- The main method should have at most 5 lines
- Each responsibility should become a private method with a business name
- No comments — the method name replaces them
```

---

### Example 2 — Primitive Obsession with Replace Data Value with Object

```
I need to refactor the following piece of code.

### Context
`Customer` class in a PHP CRM system. The tax ID is used as `string` throughout the system
and its validation is duplicated in 6 different places.

### Current code
class Customer
{
    public function __construct(
        private string $name,
        private string $taxId,
        private string $email,
    ) {}
}

### Identified smell
Primitive Obsession — `taxId` and `email` are raw strings with no centralized validation.

### Recommended technique
Replace Data Value with Object

### Constraints
- PHP 8.2
- Must not break the public API of the Customer class
- New value objects must be immutable (use readonly properties)

### Expected result
- `TaxId` as an immutable final class with validation in the constructor
- `Email` as an immutable final class with validation in the constructor
- `Customer` instances use the new types
- Validation does not repeat anywhere else
```

---

## Usage tips

- The more specific the context, the better the suggested refactoring
- Always state the language and version — idioms change between versions
- Listing constraints avoids suggestions that break existing contracts
- If you don't know the exact technique, describe the problem and ask for a diagnosis before refactoring
