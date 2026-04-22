# SKILL: Detecting and Refactoring Feature Envy — Java

## Source
Based on: https://refactoring.guru/smells/feature-envy

---

## 1. What is Feature Envy

A method accesses the data of another object more than it accesses the data of its own class. The method is "envious" of the other class — it seems to want to live there instead.

**Why this happens:**
- Fields were moved to a new class but the methods that use them were not
- Business logic was placed in a utility or service class that works on a domain object's internals
- A method was gradually extended to rely more and more on another object's getters

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A method calls several getters on another object in sequence
- [ ] A method receives an object as parameter and uses mostly its fields/methods
- [ ] A method ignores `this` entirely and only manipulates another object's state
- [ ] The method's name naturally belongs to the other class ("calculateOrderTotal" in a ReportService)

---

## 3. Treatment techniques (in order of preference)

| Situation found                                         | Recommended technique             |
|---------------------------------------------------------|-----------------------------------|
| The whole method belongs in the other class             | Move Method                       |
| Only part of the method envies another class            | Extract Method, then Move Method  |
| The method uses data from several classes               | Move to the class with most data  |
| The method is a calculation that belongs on the model   | Move Method to domain object      |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class InvoicePrinter {
    public double calculateInvoiceTotal(Invoice invoice) {
        double subtotal = 0;
        for (InvoiceItem item : invoice.getItems()) {
            subtotal += item.getPrice() * item.getQuantity();
        }
        double tax = subtotal * invoice.getTaxRate();
        double discount = invoice.hasDiscount() ? subtotal * invoice.getDiscountRate() : 0;
        return subtotal + tax - discount;
    }
}
```

**AFTER — expected:**
```java
public class Invoice {
    public double calculateTotal() {
        double subtotal = items.stream()
            .mapToDouble(i -> i.getPrice() * i.getQuantity())
            .sum();
        double tax = subtotal * taxRate;
        double discount = hasDiscount() ? subtotal * discountRate : 0;
        return subtotal + tax - discount;
    }
}

public class InvoicePrinter {
    public void print(Invoice invoice) {
        double total = invoice.calculateTotal(); // delegate to owner
        // ... print logic
    }
}
```

**Why this pattern:**
- `calculateTotal` naturally belongs to `Invoice` — it only uses Invoice data
- `InvoicePrinter` is freed from knowing Invoice internals

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the envious method and adding a pass-through**
```java
// Not accepted — delegation without moving creates indirection, not clarity
public double calculateInvoiceTotal(Invoice invoice) {
    return invoice.calculateTotal(); // just a wrapper — delete the envious method entirely
}
```

**Mistake 2: Moving the method but keeping parameters that expose internals**
```java
// Not accepted — moved but still passing raw field values
public double calculateTotal(double subtotal, double taxRate, double discountRate, boolean hasDiscount) { ... }
```

**Mistake 3: Moving a method that uses data from many classes**
```java
// Not accepted — if the method uses Order, Customer, and Coupon equally,
// moving it to any one class just shifts the envy; Extract Class is better
```

---

## 6. Benefits

- **Cohesion:** Each class owns the behavior that operates on its own data
- **Encapsulation:** The domain object exposes intent, not raw fields
- **Maintainability:** Business rules are colocated with the data they govern
