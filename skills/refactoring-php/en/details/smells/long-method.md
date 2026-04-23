# SKILL: Detecting and Refactoring Long Method — PHP

## Source
Based on: https://refactoring.guru/smells/long-method

---

## 1. What is Long Method

A method that contains too many lines of code.
As a rule of thumb: any method longer than 10 lines deserves attention.

**Why this happens:**
- Logic is always added to the method, never removed
- It is mentally easier to add two lines than to create a new method
- The problem grows silently until it becomes spaghetti code

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Method with more than 10 lines
- [ ] Code block you felt the urge to comment
- [ ] Complex conditional (nested if/else)
- [ ] Loop with non-trivial logic inside
- [ ] Temporary variables scattered throughout the method
- [ ] Method that clearly does more than one distinct thing

---

## 3. Treatment techniques (in order of preference)

| Situation found                              | Recommended technique             |
|----------------------------------------------|-----------------------------------|
| Commented or commentable code block          | Extract Method                    |
| Local variables prevent extraction           | Replace Temp with Query           |
| Too many parameters in extracted method      | Introduce Parameter Object        |
| Whole object being dismembered               | Preserve Whole Object             |
| None of the above works                      | Replace Method with Method Object |
| Complex conditional                          | Decompose Conditional             |
| Loop with complex internal logic             | Extract Method                    |

**Golden rule:** If you felt the urge to write a comment inside the method,
that block should become a method with a descriptive name.
The name replaces the comment.

---

## 4. Example

**BEFORE — not accepted:**
```php
public function processOrder(Order $order): void
{
    // validate order
    if (empty($order->getItems())) {
        throw new InvalidOrderException("Order has no items");
    }
    if ($order->getCustomer() === null) {
        throw new InvalidOrderException("Customer not provided");
    }

    // calculate total
    $total = 0;
    foreach ($order->getItems() as $item) {
        $total += $item->getPrice() * $item->getQuantity();
    }
    if ($order->hasDiscount()) {
        $total = $total * (1 - $order->getDiscount());
    }

    // persist
    $this->orderRepository->save($order);
    $this->emailService->sendConfirmation($order->getCustomer()->getEmail());
}
```

**AFTER — expected:**
```php
public function processOrder(Order $order): void
{
    $this->validateOrder($order);
    $total = $this->calculateTotal($order);
    $order->setTotal($total);
    $this->confirmOrder($order);
}

private function validateOrder(Order $order): void
{
    if (empty($order->getItems())) {
        throw new InvalidOrderException("Order has no items");
    }
    if ($order->getCustomer() === null) {
        throw new InvalidOrderException("Customer not provided");
    }
}

private function calculateTotal(Order $order): float
{
    $total = array_reduce($order->getItems(), function (float $carry, Item $item) {
        return $carry + $item->getPrice() * $item->getQuantity();
    }, 0.0);

    return $order->hasDiscount() ? $total * (1 - $order->getDiscount()) : $total;
}

private function confirmOrder(Order $order): void
{
    $this->orderRepository->save($order);
    $this->emailService->sendConfirmation($order->getCustomer()->getEmail());
}
```

**Why this pattern:**
- Each private method has a name that expresses business intent
- The main method became a readable narrative
- No comments needed — the name replaces them

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting without naming with intent**
```php
// Not accepted — name does not describe what it does
private function doSomething(Order $order): void { ... }
private function method1(Order $order): void { ... }
private function processPart2(Order $order): void { ... }
```

**Mistake 2: Commenting instead of extracting**
```php
// Not accepted — comment indicates it should be a method
// validates customer data
if ($customer === null || $customer->getTaxId() === null) { ... }
```

**Mistake 3: Extracting and leaving the method still long**
```php
// Not accepted — extracted one block but the original method
// still has 60 lines with multiple responsibilities
```

---

## 6. Benefits

- **Longevity:** Classes with short methods live longest
- **Maintainability:** Reduces difficulty in understanding and maintaining code
- **Quality:** Prevents duplicate code hidden in long methods
- **Performance:** Negligible negative impact; enables better optimization opportunities
- **Clarity:** Clear and understandable code makes it easier to identify real performance improvements
