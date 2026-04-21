# SMELL: Inappropriate Intimacy — PHP

## Source
Based on: https://refactoring.guru/smells/inappropriate-intimacy

---

## 1. What is it?

One class accesses the private properties, internal arrays, or implementation details of another class too freely. The two classes are tightly coupled in a way that makes it hard to change one without affecting the other.

---

## 2. Warning signs

- [ ] A class calls multiple getters on another class to perform a calculation that should live in that other class
- [ ] A class directly mutates the array or internal state of another class
- [ ] Two classes reference each other in a bidirectional dependency
- [ ] A class knows the internal structure of another class (e.g., knows which properties are null in which state)
- [ ] Repeated "train wrecks": `$a->getB()->getC()->doSomething()`

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move the method that accesses foreign internals into the class that owns those internals |
| **Move Field** | Move a property to the class that uses it most |
| **Hide Delegate** | Add a forwarding method so the client doesn't reach through an intermediary |
| **Extract Class** | If two classes are too intimate because they share responsibilities, separate those responsibilities cleanly |

---

## 4. Example

**BEFORE — not accepted:**
```php
class Order {
    private array $items = [];
    private string $customerEmail = '';

    public function getItems(): array { return $this->items; } // exposes internals
    public function getCustomerEmail(): string { return $this->customerEmail; }
}

// Report digs into Order's guts
class OrderReport {
    public function generate(Order $order): string {
        $total = 0.0;
        foreach ($order->getItems() as $item) {  // reaching into Order
            $total += $item->getPrice() * $item->getQuantity();
        }
        return 'Order for ' . $order->getCustomerEmail() . ' total: ' . $total;
    }
}
```

**AFTER — expected:**
```php
class Order {
    public function __construct(
        private readonly array $items,
        private readonly string $customerEmail
    ) {}

    public function total(): float {
        return array_reduce(
            $this->items,
            fn(float $sum, OrderItem $item) => $sum + $item->getPrice() * $item->getQuantity(),
            0.0
        );
    }

    public function getCustomerEmail(): string { return $this->customerEmail; }
}

// Report only asks the Order for what it needs
class OrderReport {
    public function generate(Order $order): string {
        return 'Order for ' . $order->getCustomerEmail() . ' total: ' . $order->total();
    }
}
```

**Why this pattern:**
- `OrderReport` no longer depends on the internal structure of `Order`
- Changing how `Order` stores items doesn't affect `OrderReport`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the loop with array_reduce but still pulling the array**
```php
// Not accepted — getItems() still exposes the internals
$total = array_reduce($order->getItems(), fn($sum, $item) => $sum + $item->getPrice() * $item->getQuantity(), 0.0);
```

**Mistake 2: Creating a DTO that copies all Order properties**
```php
// Not accepted — an OrderDTO with all the same properties is just indirection; the coupling remains
```

---

## 6. Benefits

- **Lower coupling:** Classes can change their internals without affecting their clients
- **Better encapsulation:** The owning class controls how its data is computed and accessed
- **Easier testing:** Classes with fewer dependencies are simpler to test in isolation
