# TECHNIQUE: Replace Method with Method Object — Java

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
2. Add a field for the object that originally contained the method (`source`)
3. Add a field for each local variable and each parameter of the original method
4. Create a constructor that takes the source object and all parameters, assigning them to the fields
5. Copy the original method body into a `compute()` method; references to local variables become field references
6. Replace the original method body with: `return new MethodNameObject(this, param1, param2).compute();`
7. Apply Extract Method freely on `compute()` — no parameters needed since shared state lives in the fields
8. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public class Order {
    public double price() {
        double primaryBasePrice;
        double secondaryBasePrice;
        double tertiaryBasePrice;
        // long and complex computation involving all three variables
        primaryBasePrice = quantity * primaryRate();
        secondaryBasePrice = primaryBasePrice * 0.7;
        tertiaryBasePrice = Math.max(primaryBasePrice, secondaryBasePrice);
        return tertiaryBasePrice - discount(primaryBasePrice, secondaryBasePrice);
    }
}
```

**AFTER — expected:**
```java
public class Order {
    public double price() {
        return new PriceCalculator(this).compute();
    }
}

class PriceCalculator {
    private final Order order;
    private double primaryBasePrice;
    private double secondaryBasePrice;
    private double tertiaryBasePrice;

    PriceCalculator(Order order) {
        this.order = order;
    }

    double compute() {
        primaryBasePrice = order.getQuantity() * order.primaryRate();
        secondaryBasePrice = primaryBasePrice * 0.7;
        tertiaryBasePrice = Math.max(primaryBasePrice, secondaryBasePrice);
        return tertiaryBasePrice - discount();
    }

    private double discount() {
        return order.isPremium()
            ? primaryBasePrice * 0.1
            : secondaryBasePrice * 0.05;
    }
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Applying this when Extract Method would suffice**
```java
// Not accepted — if the method only has 2–3 local variables,
// Extract Method with explicit parameters is simpler
```

**Mistake 2: Naming the class vaguely**
```java
// Not accepted — Helper, Processor, Calculator are too generic
class OrderHelper { ... }
// Preferred: name it after the specific algorithm
class OrderPriceCalculator { ... }
```

---

## 7. Benefits

- **Enables Extract Method:** Sub-methods can share state via fields without passing parameters
- **Testability:** The calculation object can be instantiated and tested independently
- **Extensibility:** The method object class can be subclassed to vary parts of the algorithm
