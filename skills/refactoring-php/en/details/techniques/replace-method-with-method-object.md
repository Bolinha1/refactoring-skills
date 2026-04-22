# TECHNIQUE: Replace Method with Method Object — PHP

## Source
Based on: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problem

A method is so large and complex that extracting sub-methods is awkward because they would all share many local variables. The local variables are too intertwined to be easily passed as parameters.

---

## 2. Solution

Turn the method into a separate class. Each local variable becomes a field of that class. The method body becomes the main `compute()` method of the class, which now can be freely split into sub-methods without parameter passing.

---

## 3. When to apply

- The method has many local variables that are used across multiple logical phases
- Extract Method would require passing too many parameters between the new methods
- The algorithm is complex enough to deserve its own abstraction
- You want to be able to test the algorithm in isolation or subclass parts of it

---

## 4. Refactoring steps

1. Create a new class named after the method
2. Add a property for the object that originally contained the method (`$source`)
3. Add a property for each local variable and each parameter of the original method
4. Create a constructor that takes the source object and all parameters, assigning them to the properties
5. Copy the original method body into a `compute()` method; references to local variables become property references
6. Replace the original method body with: `return (new MethodNameObject($this, $param1))->compute();`
7. Apply Extract Method freely on `compute()` — no parameters needed since shared state lives in the properties
8. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class Order {
    public function price(): float {
        $primaryBasePrice = $this->quantity * $this->primaryRate();
        $secondaryBasePrice = $primaryBasePrice * 0.7;
        $tertiaryBasePrice = max($primaryBasePrice, $secondaryBasePrice);
        return $tertiaryBasePrice - $this->discount($primaryBasePrice, $secondaryBasePrice);
    }
}
```

**AFTER — expected:**
```php
class Order {
    public function price(): float {
        return (new PriceCalculator($this))->compute();
    }
}

class PriceCalculator {
    private Order $order;
    private float $primaryBasePrice;
    private float $secondaryBasePrice;
    private float $tertiaryBasePrice;

    public function __construct(Order $order) {
        $this->order = $order;
    }

    public function compute(): float {
        $this->primaryBasePrice = $this->order->getQuantity() * $this->order->primaryRate();
        $this->secondaryBasePrice = $this->primaryBasePrice * 0.7;
        $this->tertiaryBasePrice = max($this->primaryBasePrice, $this->secondaryBasePrice);
        return $this->tertiaryBasePrice - $this->discount();
    }

    private function discount(): float {
        return $this->order->isPremium()
            ? $this->primaryBasePrice * 0.1
            : $this->secondaryBasePrice * 0.05;
    }
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Applying this when Extract Method would suffice**
```php
// Not accepted — if the method only has 2–3 local variables,
// Extract Method with explicit parameters is simpler
```

**Mistake 2: Naming the class vaguely**
```php
// Not accepted — Helper, Processor, Calculator are too generic
class OrderHelper { ... }
// Preferred: name it after the specific algorithm
class OrderPriceCalculator { ... }
```

---

## 7. Benefits

- **Enables Extract Method:** Sub-methods can share state via properties without passing parameters
- **Testability:** The calculation object can be instantiated and tested independently
- **Extensibility:** The method object class can be subclassed to vary parts of the algorithm
