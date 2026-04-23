# SKILL: Detecting and Refactoring Divergent Change — Java

## Source
Based on: https://refactoring.guru/smells/divergent-change

---

## 1. What is Divergent Change

A single class is changed for many different reasons. When you find yourself modifying the same class every time a new business rule, a new data source, or a new feature of unrelated concern is added, that class has divergent change.

The inverse of Shotgun Surgery: one class, many reasons to change.

**Why this happens:**
- Unrelated responsibilities were grouped into one class for convenience
- A class started small and grew by accumulation without being split
- "God class" tendencies: one class knows everything and does everything

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] You modify the same class every time a new database table is added
- [ ] You modify the same class every time a new report format is needed
- [ ] The class has methods that fall into clearly distinct conceptual groups
- [ ] The class has imports/dependencies from many unrelated subsystems
- [ ] Describing what the class does requires the word "and" more than once

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique   |
|----------------------------------------------------------|-------------------------|
| Class handles data access AND business logic             | Extract Class           |
| Class handles multiple unrelated business concepts       | Extract Class           |
| Methods that form a cluster can live elsewhere           | Move Method + Move Field |
| All methods relate but the class is too large            | Extract Subclass        |

**Rule of thumb:** each class should have exactly one reason to change.

---

## 4. Example

**BEFORE — not accepted:**
```java
public class OrderService {
    // Reason 1: changes when DB schema changes
    public Order findById(long id) { /* JDBC query */ }
    public void save(Order order) { /* JDBC insert/update */ }

    // Reason 2: changes when business rules change
    public void applyDiscount(Order order) { /* discount logic */ }
    public double calculateTax(Order order) { /* tax logic */ }

    // Reason 3: changes when report format changes
    public String generateInvoicePdf(Order order) { /* PDF generation */ }
    public String generateCsvExport(Order order) { /* CSV export */ }
}
```

**AFTER — expected:**
```java
public class OrderRepository {
    public Order findById(long id) { /* JDBC query */ }
    public void save(Order order) { /* JDBC insert/update */ }
}

public class OrderPricingService {
    public void applyDiscount(Order order) { /* discount logic */ }
    public double calculateTax(Order order) { /* tax logic */ }
}

public class OrderReportService {
    public String generateInvoicePdf(Order order) { /* PDF generation */ }
    public String generateCsvExport(Order order) { /* CSV export */ }
}
```

**Why this pattern:**
- Each class has exactly one reason to change
- A new report format only touches `OrderReportService`; a tax rule change only touches `OrderPricingService`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Splitting by layer instead of by responsibility**
```java
// Not accepted — OrderDataLayer still mixes DB schema and business validation
public class OrderDataLayer { /* DB + validation */ }
public class OrderPresentation { /* all output formats */ }
```

**Mistake 2: Creating many micro-classes with a single method each**
```java
// Not accepted — over-fragmentation that increases navigation overhead
public class OrderFinder { Order find(long id); }
public class OrderSaver { void save(Order o); }
public class OrderDiscountApplier { void apply(Order o); }
```

**Mistake 3: Moving code without moving the data it operates on**
```java
// Not accepted — OrderPricingService must call back into OrderService to get data
// because the fields it needs were not moved with the methods
```

---

## 6. Benefits

- **Single Responsibility:** Each class has one reason to change
- **Isolation:** Changes to one concern do not risk breaking another
- **Discoverability:** Developers know exactly which class to open for each concern
