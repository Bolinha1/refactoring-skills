# SKILL: Detecting and Refactoring Large Class — PHP

## Source
Based on: https://refactoring.guru/smells/large-class

---

## 1. What is Large Class

A class that contains too many fields, methods, or lines of code.
When a class tries to do too many things, it accumulates responsibilities
that should be distributed.

**Why this happens:**
- Classes start small and grow as the program evolves
- It is mentally easier to add functionality to an existing class than to create a new one
- The accumulation is gradual — nobody notices until the class has become a monolith

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Class with more than 200 lines
- [ ] Class with more than 10 public methods
- [ ] Class with more than 5 properties injected via constructor
- [ ] Class with clearly distinct responsibilities (SRP violation)
- [ ] Class that is modified for different reasons
- [ ] Many unrelated `use` statements at the top
- [ ] Class name is too generic (Manager, Processor, Handler, Utils)
- [ ] Properties used only by a subset of the methods

---

## 3. Treatment techniques (in order of preference)

| Situation found                                    | Recommended technique   |
|----------------------------------------------------|-------------------------|
| Behaviors groupable into an autonomous unit        | Extract Class           |
| Specialized or rarely-used functionality           | Extract Subclass        |
| Clients use only part of the interface             | Extract Interface       |
| GUI class with mixed data and logic                | Duplicate Observed Data |

**Golden rule:** If you can describe the class using the conjunction "and"
(e.g., "this class validates orders **and** calculates freight **and** sends email"),
it has too many responsibilities.

---

## 4. Example

**BEFORE — not accepted:**
```php
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private EmailService $emailService,
        private StockService $stockService,
        private InvoiceService $invoiceService,
    ) {}

    public function create(Order $order): void
    {
        if (empty($order->getItems())) {
            throw new \RuntimeException("No items");
        }
        if ($order->getCustomer() === null) {
            throw new \RuntimeException("No customer");
        }

        $total = 0;
        foreach ($order->getItems() as $item) {
            $total += $item->getPrice() * $item->getQuantity();
        }
        $order->setTotal($total);

        foreach ($order->getItems() as $item) {
            $this->stockService->reserve($item->getProductId(), $item->getQuantity());
        }

        $this->repository->save($order);

        $invoice = $this->invoiceService->generate($order);
        $order->setInvoice($invoice);

        $this->emailService->sendConfirmation($order->getCustomer()->getEmail(), $order);
    }

    public function cancel(Order $order): void { /* ... */ }
    public function recalculateShipping(Order $order): void { /* ... */ }
    public function applyCoupon(Order $order, string $coupon): void { /* ... */ }
    public function findByCustomer(int $customerId): array { /* ... */ }
    public function findByPeriod(\DateTime $start, \DateTime $end): array { /* ... */ }
    public function generateReport(array $orders): string { /* ... */ }
}
```

**AFTER — expected (Extract Class):**
```php
class OrderService
{
    public function __construct(
        private OrderRepository $repository,
        private OrderValidator $validator,
        private TotalCalculator $calculator,
        private StockReservation $stockReservation,
        private OrderNotification $notification,
    ) {}

    public function create(Order $order): void
    {
        $this->validator->validate($order);
        $order->setTotal($this->calculator->calculate($order));
        $this->stockReservation->reserve($order);
        $this->repository->save($order);
        $this->notification->sendConfirmation($order);
    }
}

class OrderValidator
{
    public function validate(Order $order): void
    {
        if (empty($order->getItems())) {
            throw new \RuntimeException("No items");
        }
        if ($order->getCustomer() === null) {
            throw new \RuntimeException("No customer");
        }
    }
}

class TotalCalculator
{
    public function calculate(Order $order): float
    {
        return array_reduce($order->getItems(), function (float $carry, Item $item) {
            return $carry + $item->getPrice() * $item->getQuantity();
        }, 0.0);
    }
}

class StockReservation
{
    public function __construct(private StockService $stockService) {}

    public function reserve(Order $order): void
    {
        foreach ($order->getItems() as $item) {
            $this->stockService->reserve($item->getProductId(), $item->getQuantity());
        }
    }
}

class OrderNotification
{
    public function __construct(
        private EmailService $emailService,
        private InvoiceService $invoiceService,
    ) {}

    public function sendConfirmation(Order $order): void
    {
        $invoice = $this->invoiceService->generate($order);
        $order->setInvoice($invoice);
        $this->emailService->sendConfirmation($order->getCustomer()->getEmail(), $order);
    }
}
```

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting a class without cohesion**
```php
// Not accepted — the extracted class still mixes responsibilities
class OrderHelper
{
    public function validate(Order $order): void { ... }
    public function calculateShipping(Order $order): float { ... }
    public function generateReport(array $orders): string { ... }
}
```

**Mistake 2: Creating anemic classes**
```php
// Not accepted — no real behavior, just delegates
class OrderValidator
{
    public function validate(Order $order): bool
    {
        return $order->isValid();
    }
}
```

**Mistake 3: Generic names for extracted classes**
```php
// Not accepted
class OrderUtils { ... }
class OrderManager { ... }
class OrderHelper2 { ... }
```

---

## 6. Benefits

- **Cognitive load:** Developers don't need to memorize excessive attributes and methods
- **Duplicate elimination:** Splitting large classes frequently eliminates redundant code
- **Maintainability:** Each class with a single responsibility is easier to test and modify
- **Reusability:** Smaller, focused classes are easier to reuse in other contexts
