# SMELL: Data Class — PHP

## Source
Based on: https://refactoring.guru/smells/data-class

---

## 1. What is it?

A class that contains only fields, getters, and setters — but no real behaviour. All the logic that operates on this data lives in other classes. The class is essentially a dumb data container and acts as a passive record rather than a responsible object.

---

## 2. Warning signs

- [ ] A class has only properties, getters, and setters (and maybe a constructor)
- [ ] Business logic that should belong to the class is scattered across service or manager classes
- [ ] Other classes manipulate the data class's properties directly through getters/setters
- [ ] The class is used only as a parameter bag passed between methods
- [ ] No methods exist beyond trivial accessors

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move behaviour from service classes into the data class, giving it responsibility |
| **Extract Class** | If the data class has grown large, split off a cohesive subset into its own class with behaviour |
| **Hide Method** | Make setters private if outside code shouldn't modify the property directly |
| **Encapsulate Field** | Replace direct property access with getters/setters as a first step toward adding validation or logic |

---

## 4. Example

**BEFORE — not accepted:**
```php
// Pure data container — no behaviour
class Order {
    private array $items = [];
    private string $status = '';

    public function getItems(): array { return $this->items; }
    public function setItems(array $items): void { $this->items = $items; }
    public function getStatus(): string { return $this->status; }
    public function setStatus(string $status): void { $this->status = $status; }
}

// All logic lives in an external service
class OrderService {
    public function calculateTotal(Order $order): float {
        return array_reduce(
            $order->getItems(),
            fn(float $sum, OrderItem $item) => $sum + $item->getPrice() * $item->getQuantity(),
            0.0
        );
    }

    public function canBeCancelled(Order $order): bool {
        return $order->getStatus() === 'PENDING';
    }
}
```

**AFTER — expected:**
```php
class Order {
    private string $status = 'PENDING';

    public function __construct(private readonly array $items) {}

    public function total(): float {
        return array_reduce(
            $this->items,
            fn(float $sum, OrderItem $item) => $sum + $item->getPrice() * $item->getQuantity(),
            0.0
        );
    }

    public function canBeCancelled(): bool {
        return $this->status === 'PENDING';
    }

    public function cancel(): void {
        if (!$this->canBeCancelled()) {
            throw new \DomainException("Cannot cancel order in status: {$this->status}");
        }
        $this->status = 'CANCELLED';
    }
}
```

**Why this pattern:**
- `Order` now owns its business rules — callers ask the order itself
- The `status` property is protected: transitions happen through domain methods

---

## 5. Negative examples — what NOT to do

**Mistake 1: Moving all service methods into the data class without discrimination**
```php
// Not accepted — the class becomes a Large Class; only move cohesive behaviour
class Order {
    public function sendConfirmationEmail(): void { /* unrelated to Order's domain */ }
    public function saveToDatabase(): void { /* infrastructure concern */ }
}
```

**Mistake 2: Keeping public setters after adding domain logic**
```php
// Not accepted — any caller can bypass the domain rule by calling setStatus() directly
public function setStatus(string $status): void { $this->status = $status; }
```

---

## 6. Benefits

- **Encapsulation:** Business rules live next to the data they govern
- **Reduced duplication:** Logic that was copied across service classes is now in one place
- **Tell, don't ask:** Callers ask the object to do something instead of reading its properties and deciding externally
