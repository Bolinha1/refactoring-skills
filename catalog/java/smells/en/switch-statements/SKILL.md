# SKILL: Detecting and Refactoring Switch Statements — Java

## Source
Based on: https://refactoring.guru/smells/switch-statements

---

## 1. What is Switch Statements

Complex switch or if/else chains that branch on the type or state of an object. The same switch logic tends to be duplicated across the codebase — when a new case is added, every switch must be found and updated.

**Why this happens:**
- Type-based behavior was implemented procedurally instead of using polymorphism
- A single class grew to handle multiple variants of a concept
- The concept of "type" was encoded as a string or integer rather than as a class hierarchy

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `switch` or `if/else if` block that branches on a type field, status, or enum
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
```java
public class ShippingCalculator {
    public double calculateCost(Order order) {
        switch (order.getShippingType()) {
            case "standard":
                return order.getWeight() * 1.5;
            case "express":
                return order.getWeight() * 3.0 + 5.0;
            case "overnight":
                return order.getWeight() * 5.0 + 20.0;
            default:
                throw new IllegalArgumentException("Unknown shipping type");
        }
    }
}
```

**AFTER — expected:**
```java
public interface ShippingStrategy {
    double calculateCost(double weight);
}

public class StandardShipping implements ShippingStrategy {
    public double calculateCost(double weight) { return weight * 1.5; }
}

public class ExpressShipping implements ShippingStrategy {
    public double calculateCost(double weight) { return weight * 3.0 + 5.0; }
}

public class OvernightShipping implements ShippingStrategy {
    public double calculateCost(double weight) { return weight * 5.0 + 20.0; }
}

public class Order {
    private final ShippingStrategy shippingStrategy;

    public double calculateShippingCost() {
        return shippingStrategy.calculateCost(this.weight);
    }
}
```

**Why this pattern:**
- Adding a new shipping type requires only a new class, not a search-and-update in every switch
- Each strategy is independently testable and cohesive

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the switch with an if/else chain**
```java
// Not accepted — same smell, different syntax
if (type.equals("standard")) { ... }
else if (type.equals("express")) { ... }
else if (type.equals("overnight")) { ... }
```

**Mistake 2: Centralizing into a single god factory without polymorphism**
```java
// Not accepted — the switch still exists, just moved
public static ShippingCalculator create(String type) {
    switch (type) { ... }
}
```

**Mistake 3: Applying polymorphism to a switch that will never grow**
```java
// Not accepted — creating 3 classes for true/false toggle is over-engineering
switch (order.isPriority()) {
    case true: return priorityPrice;
    case false: return standardPrice;
}
```

---

## 6. Benefits

- **Open/Closed:** Adding a new variant requires a new class, not editing existing ones
- **Locality:** Each variant's behavior is contained in one place
- **Testability:** Each polymorphic variant can be tested in isolation
