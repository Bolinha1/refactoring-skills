# SKILL: Detecting and Refactoring Switch Statements — PHP

## Source
Based on: https://refactoring.guru/smells/switch-statements

---

## 1. What is Switch Statements

Complex switch or if/elseif chains that branch on the type or state of an object. The same switch logic tends to be duplicated across the codebase — when a new case is added, every switch must be found and updated.

**Why this happens:**
- Type-based behavior was implemented procedurally instead of using polymorphism
- A single class grew to handle multiple variants of a concept
- The concept of "type" was encoded as a string or integer rather than as a class hierarchy

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `switch` or `if/elseif` block that branches on a type field, status, or string
- [ ] The same switch appears in more than one place
- [ ] Adding a new "type" requires searching for and updating every switch in the codebase
- [ ] Each branch of the switch does something very different from the others
- [ ] Switch on a type to decide which object to instantiate

---

## 3. Treatment techniques (in order of preference)

| Situation found                                           | Recommended technique                  |
|-----------------------------------------------------------|----------------------------------------|
| Switch on type to decide behavior                         | Replace Conditional with Polymorphism  |
| Switch on type to decide which class to create            | Replace Constructor with Factory Method |
| Only a few cases and adding new ones is rare              | Replace Type Code with Subclasses      |
| The type has mutable state that changes at runtime        | Replace Type Code with State/Strategy  |
| Switch is simple and used in only one place               | Leave it — not every switch is a smell |

---

## 4. Example

**BEFORE — not accepted:**
```php
class ShippingCalculator
{
    public function calculateCost(Order $order): float
    {
        switch ($order->getShippingType()) {
            case 'standard':
                return $order->getWeight() * 1.5;
            case 'express':
                return $order->getWeight() * 3.0 + 5.0;
            case 'overnight':
                return $order->getWeight() * 5.0 + 20.0;
            default:
                throw new \InvalidArgumentException('Unknown shipping type');
        }
    }
}
```

**AFTER — expected:**
```php
interface ShippingStrategy
{
    public function calculateCost(float $weight): float;
}

class StandardShipping implements ShippingStrategy
{
    public function calculateCost(float $weight): float { return $weight * 1.5; }
}

class ExpressShipping implements ShippingStrategy
{
    public function calculateCost(float $weight): float { return $weight * 3.0 + 5.0; }
}

class OvernightShipping implements ShippingStrategy
{
    public function calculateCost(float $weight): float { return $weight * 5.0 + 20.0; }
}

class Order
{
    public function __construct(
        private float $weight,
        private ShippingStrategy $shippingStrategy,
    ) {}

    public function calculateShippingCost(): float
    {
        return $this->shippingStrategy->calculateCost($this->weight);
    }
}
```

**Why this pattern:**
- Adding a new shipping type requires only a new class, not a search-and-update in every switch
- Each strategy is independently testable and cohesive

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the switch with an if/elseif chain**
```php
// Not accepted — same smell, different syntax
if ($type === 'standard') { ... }
elseif ($type === 'express') { ... }
elseif ($type === 'overnight') { ... }
```

**Mistake 2: Centralizing into a single factory without polymorphism**
```php
// Not accepted — the switch still exists, just moved
public static function create(string $type): ShippingCalculator
{
    switch ($type) { ... }
}
```

**Mistake 3: Applying polymorphism to a switch that will never grow**
```php
// Not accepted — creating 2 classes for a boolean toggle is over-engineering
if ($order->isPriority()) {
    return $priorityPrice;
}
return $standardPrice;
```

---

## 6. Benefits

- **Open/Closed:** Adding a new variant requires a new class, not editing existing ones
- **Locality:** Each variant's behavior is contained in one place
- **Testability:** Each polymorphic variant can be tested in isolation
