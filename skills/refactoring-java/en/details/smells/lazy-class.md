# SMELL: Lazy Class — Java

## Source
Based on: https://refactoring.guru/smells/lazy-class

---

## 1. What is it?

A class that does so little that it is not worth the overhead of understanding and maintaining it. It may have been useful in the past, may exist in anticipation of future growth, or may be the leftover shell of a refactoring that extracted most of its logic.

---

## 2. Warning signs

- [ ] A class has only one or two trivial methods
- [ ] A class delegates everything to another single class with minimal added value
- [ ] A class was created to abstract something that never needed abstracting
- [ ] The class could be replaced by a single method or function elsewhere
- [ ] The class has few usages across the codebase

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Inline Class** | When the class does so little that its logic can be absorbed into the caller |
| **Collapse Hierarchy** | When the lazy class is a nearly-empty subclass or superclass in an inheritance hierarchy |

---

## 4. Example

**BEFORE — not accepted:**
```java
// TaxCalculator does only one trivial thing
public class TaxCalculator {
    public double calculate(double amount) {
        return amount * 0.2;
    }
}

// OrderService only delegates to TaxCalculator
public class OrderService {
    private final TaxCalculator taxCalculator = new TaxCalculator();

    public double totalWithTax(double amount) {
        return amount + taxCalculator.calculate(amount);
    }
}
```

**AFTER — expected:**
```java
public class OrderService {
    private static final double TAX_RATE = 0.2;

    public double totalWithTax(double amount) {
        return amount * (1 + TAX_RATE);
    }
}
```

**Why this pattern:**
- The tax logic is simple enough to live directly in the service
- One fewer class means one fewer file to navigate, test, and maintain

---

## 5. Negative examples — what NOT to do

**Mistake 1: Inlining a class that encapsulates a changing rule**
```java
// Caution — if tax rate differs per country or product category, TaxCalculator
// may be about to grow; check the future before inlining
```

**Mistake 2: Inlining a class shared by multiple callers**
```java
// Not accepted — if TaxCalculator is used by OrderService, InvoiceService, and
// QuoteService, inlining it duplicates the logic in three places
```

---

## 6. Benefits

- **Reduced complexity:** Fewer classes means a smaller mental map of the system
- **Simpler navigation:** Developers don't follow a chain of delegations to find trivial logic
- **Less boilerplate:** No constructor, field, or import needed for a class that does almost nothing
