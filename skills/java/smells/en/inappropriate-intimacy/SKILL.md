# SMELL: Inappropriate Intimacy — Java

## Source
Based on: https://refactoring.guru/smells/inappropriate-intimacy

---

## 1. What is it?

One class accesses the private fields, internal collections, or implementation details of another class too freely. The two classes are tightly coupled in a way that makes it hard to change one without affecting the other.

---

## 2. Warning signs

- [ ] A class calls multiple getters on another class to perform a calculation that should live in that other class
- [ ] A class directly mutates the collection or internal state of another class
- [ ] Two classes reference each other in a bidirectional dependency
- [ ] A class knows the internal structure of another class (e.g., knows which fields are null in which state)
- [ ] Repeated "train wrecks": `a.getB().getC().doSomething()`

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move the method that accesses foreign internals into the class that owns those internals |
| **Move Field** | Move a field to the class that uses it most |
| **Hide Delegate** | Add a forwarding method so the client doesn't reach through an intermediary |
| **Change Bidirectional Association to Unidirectional** | Remove one direction of a two-way reference when one direction is sufficient |
| **Extract Class** | If two classes are too intimate because they share responsibilities, separate those responsibilities cleanly |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class Order {
    private List<OrderItem> items = new ArrayList<>();
    private String customerEmail;

    public List<OrderItem> getItems() { return items; } // exposes internals
    public String getCustomerEmail() { return customerEmail; }
}

// Report digs into Order's guts
public class OrderReport {
    public String generate(Order order) {
        double total = 0;
        for (OrderItem item : order.getItems()) {   // reaching into Order
            total += item.getPrice() * item.getQuantity();
        }
        return "Order for " + order.getCustomerEmail() + " total: " + total;
    }
}
```

**AFTER — expected:**
```java
public class Order {
    private final List<OrderItem> items;
    private final String customerEmail;

    public double total() {
        return items.stream()
            .mapToDouble(i -> i.getPrice() * i.getQuantity())
            .sum();
    }

    public String getCustomerEmail() { return customerEmail; }
}

// Report only asks the Order for what it needs
public class OrderReport {
    public String generate(Order order) {
        return "Order for " + order.getCustomerEmail() + " total: " + order.total();
    }
}
```

**Why this pattern:**
- `OrderReport` no longer depends on the internal structure of `Order`
- Changing how `Order` stores items doesn't affect `OrderReport`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Replacing the loop with a stream but still pulling the list**
```java
// Not accepted — getItems() still exposes the internals
double total = order.getItems().stream()
    .mapToDouble(i -> i.getPrice() * i.getQuantity())
    .sum();
```

**Mistake 2: Creating a DTO that copies all Order fields to avoid the getter chain**
```java
// Not accepted — an OrderDTO with all the same fields is just indirection; the coupling remains
```

---

## 6. Benefits

- **Lower coupling:** Classes can change their internals without affecting their clients
- **Better encapsulation:** The owning class controls how its data is computed and accessed
- **Easier testing:** Classes with fewer dependencies are simpler to test in isolation
