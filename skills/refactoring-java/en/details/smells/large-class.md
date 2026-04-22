# SKILL: Detecting and Refactoring Large Class — Java

## Source
Based on: https://refactoring.guru/smells/large-class

---

## 1. What is Large Class

A class that contains too many fields, methods, or lines of code.
When a class tries to do too many things, it accumulates responsibilities
that should be distributed.

**Why this happens:**
- Classes start small and grow as the program evolves
- It is mentally easier to add functionality to an existing class than to create a new one
- The accumulation is gradual — nobody notices until the class has become a monolith

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Class with more than 200 lines
- [ ] Class with more than 10 public methods
- [ ] Class with more than 5 fields/attributes
- [ ] Class with clearly distinct responsibilities (SRP violation)
- [ ] Class that is modified for different reasons
- [ ] Many imports that are unrelated to each other
- [ ] Class name is too generic (Manager, Processor, Handler, Utils)
- [ ] Fields that are only used by a subset of the methods

---

## 3. Treatment techniques (in order of preference)

| Situation found                                    | Recommended technique   |
|----------------------------------------------------|-------------------------|
| Behaviors groupable into an autonomous unit        | Extract Class           |
| Specialized or rarely-used functionality           | Extract Subclass        |
| Clients use only part of the interface             | Extract Interface       |
| GUI class with mixed data and logic                | Duplicate Observed Data |

**Golden rule:** If you can describe the class using the conjunction "and"
(e.g., "this class validates orders **and** calculates freight **and** sends email"),
it has too many responsibilities.

---

## 4. Example

**BEFORE — not accepted:**
```java
public class OrderService {
    private OrderRepository repository;
    private EmailService emailService;
    private StockService stockService;
    private InvoiceService invoiceService;

    public void create(Order order) {
        if (order.getItems().isEmpty()) throw new RuntimeException("No items");
        if (order.getCustomer() == null) throw new RuntimeException("No customer");

        double total = 0;
        for (Item item : order.getItems()) {
            total += item.getPrice() * item.getQuantity();
        }
        order.setTotal(total);

        for (Item item : order.getItems()) {
            stockService.reserve(item.getProductId(), item.getQuantity());
        }

        repository.save(order);

        Invoice invoice = invoiceService.generate(order);
        order.setInvoice(invoice);

        emailService.sendConfirmation(order.getCustomer().getEmail(), order);
    }

    public void cancel(Order order) { /* ... extensive logic ... */ }
    public void recalculateShipping(Order order) { /* ... extensive logic ... */ }
    public void applyCoupon(Order order, String coupon) { /* ... extensive logic ... */ }
    public List<Order> findByCustomer(Long customerId) { /* ... */ }
    public List<Order> findByPeriod(Date start, Date end) { /* ... */ }
    public byte[] generateReport(List<Order> orders) { /* ... */ }
}
```

**AFTER — expected (Extract Class):**
```java
public class OrderService {
    private OrderRepository repository;
    private OrderValidator validator;
    private TotalCalculator calculator;
    private StockReservation stockReservation;
    private OrderNotification notification;

    public void create(Order order) {
        validator.validate(order);
        double total = calculator.calculate(order);
        order.setTotal(total);
        stockReservation.reserve(order);
        repository.save(order);
        notification.sendConfirmation(order);
    }
}

public class OrderValidator {
    public void validate(Order order) {
        if (order.getItems().isEmpty()) throw new RuntimeException("No items");
        if (order.getCustomer() == null) throw new RuntimeException("No customer");
    }
}

public class TotalCalculator {
    public double calculate(Order order) {
        return order.getItems().stream()
            .mapToDouble(i -> i.getPrice() * i.getQuantity())
            .sum();
    }
}

public class StockReservation {
    private StockService stockService;

    public void reserve(Order order) {
        for (Item item : order.getItems()) {
            stockService.reserve(item.getProductId(), item.getQuantity());
        }
    }
}

public class OrderNotification {
    private EmailService emailService;
    private InvoiceService invoiceService;

    public void sendConfirmation(Order order) {
        Invoice invoice = invoiceService.generate(order);
        order.setInvoice(invoice);
        emailService.sendConfirmation(order.getCustomer().getEmail(), order);
    }
}
```

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting a class without cohesion**
```java
// Not accepted — the extracted class still mixes responsibilities
public class OrderHelper {
    public void validate(Order order) { ... }
    public void calculateShipping(Order order) { ... }
    public byte[] generateReport(List<Order> orders) { ... }
}
```

**Mistake 2: Creating anemic classes**
```java
// Not accepted — no real behavior, just delegates
public class OrderValidator {
    public boolean validate(Order order) {
        return order.isValid();
    }
}
```

**Mistake 3: Generic names for extracted classes**
```java
// Not accepted
public class OrderUtils { ... }
public class OrderManager { ... }
public class OrderHelper2 { ... }
```

---

## 6. Benefits

- **Cognitive load:** Developers don't need to memorize excessive attributes and methods
- **Duplicate elimination:** Splitting large classes frequently eliminates redundant code
- **Maintainability:** Each class with a single responsibility is easier to test and modify
- **Reusability:** Smaller, focused classes are easier to reuse in other contexts
