# SKILL: Detecting and Refactoring Duplicate Code — Java

## Source
Based on: https://refactoring.guru/smells/duplicate-code

---

## 1. What is Duplicate Code

Two or more code fragments look almost identical or are structurally similar across different places in the codebase. Any change to the logic must be replicated in every copy, and forgetting one creates a silent bug.

**Why this happens:**
- Copy-paste used as a shortcut instead of abstraction
- Parallel development where two developers solved the same problem independently
- Logic that started similar and diverged slightly, making the duplication invisible

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Same or nearly identical block of code in two or more methods
- [ ] Same computation or expression repeated in several places
- [ ] Two classes contain methods that do essentially the same thing
- [ ] Two subclasses implement methods with the same body
- [ ] You copy-pasted code and tweaked one variable

---

## 3. Treatment techniques (in order of preference)

| Situation found                                        | Recommended technique             |
|--------------------------------------------------------|-----------------------------------|
| Duplicate blocks in the same class                     | Extract Method                    |
| Duplicate blocks in sibling subclasses                 | Pull Up Method                    |
| Similar but not identical blocks in sibling subclasses | Form Template Method              |
| Duplicate code in two unrelated classes                | Extract Class or Move Method      |
| Long duplicate methods that differ only in algorithm   | Substitute Algorithm              |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class SalesReport {
    public double calculateRevenue(List<Order> orders) {
        double total = 0;
        for (Order order : orders) {
            if (order.getStatus() == OrderStatus.PAID) {
                total += order.getAmount();
            }
        }
        return total;
    }
}

public class FinanceReport {
    public double computePaidOrdersTotal(List<Order> orders) {
        double total = 0;
        for (Order order : orders) {
            if (order.getStatus() == OrderStatus.PAID) {
                total += order.getAmount();
            }
        }
        return total;
    }
}
```

**AFTER — expected:**
```java
public class OrderCalculator {
    public static double sumPaidOrders(List<Order> orders) {
        return orders.stream()
            .filter(o -> o.getStatus() == OrderStatus.PAID)
            .mapToDouble(Order::getAmount)
            .sum();
    }
}

public class SalesReport {
    public double calculateRevenue(List<Order> orders) {
        return OrderCalculator.sumPaidOrders(orders);
    }
}

public class FinanceReport {
    public double computePaidOrdersTotal(List<Order> orders) {
        return OrderCalculator.sumPaidOrders(orders);
    }
}
```

**Why this pattern:**
- Business logic lives in one place — a single change fixes all callers
- Both report classes delegate to the shared utility instead of owning the logic

---

## 5. Negative examples — what NOT to do

**Mistake 1: Accepting small differences as justification for duplication**
```java
// Not accepted — "almost the same" still means duplicate
public double calculateRevenue(List<Order> orders) { /* loop */ }
public double calculateProjectedRevenue(List<Order> orders) { /* same loop, one extra multiply */ }
```

**Mistake 2: Extracting to a private method but duplicating it in each class**
```java
// Not accepted — the private method itself is duplicated in SalesReport and FinanceReport
private double sumPaid(List<Order> orders) { ... }
```

**Mistake 3: Merging duplicates into a god method with boolean flags**
```java
// Not accepted — creates a new smell (Feature Envy / Long Method)
public double calculate(List<Order> orders, boolean paid, boolean projected, boolean withTaxes) { ... }
```

---

## 6. Benefits

- **Maintenance:** A single bug fix or rule change applies everywhere automatically
- **Clarity:** Code that expresses a shared concept is easier to understand than scattered copies
- **Trust:** Developers can be confident there are no diverged copies hiding stale logic
