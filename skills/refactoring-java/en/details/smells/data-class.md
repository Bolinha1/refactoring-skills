# SMELL: Data Class — Java

## Source
Based on: https://refactoring.guru/smells/data-class

---

## 1. What is it?

A class that contains only fields, getters, and setters — but no real behaviour. All the logic that operates on this data lives in other classes. The class is essentially a dumb data container and acts as a passive record rather than a responsible object.

---

## 2. Warning signs

- [ ] A class has only fields, getters, and setters (and maybe a constructor)
- [ ] Business logic that should belong to the class is scattered across service or manager classes
- [ ] Other classes manipulate the data class's fields directly through getters/setters
- [ ] The class is used only as a parameter bag passed between methods
- [ ] No methods exist beyond trivial accessors

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move behaviour from service classes into the data class, giving it responsibility |
| **Extract Class** | If the data class has grown large, split off a cohesive subset into its own class with behaviour |
| **Hide Method** | Make setters private or package-private if outside code shouldn't modify the field directly |
| **Encapsulate Field** | Replace direct field access with getters/setters as a first step toward adding validation or logic |

---

## 4. Example

**BEFORE — not accepted:**
```java
// Pure data container — no behaviour
public class Order {
    private List<OrderItem> items;
    private String status;

    public List<OrderItem> getItems() { return items; }
    public void setItems(List<OrderItem> items) { this.items = items; }
    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }
}

// All logic lives in an external service
public class OrderService {
    public double calculateTotal(Order order) {
        return order.getItems().stream()
            .mapToDouble(i -> i.getPrice() * i.getQuantity())
            .sum();
    }

    public boolean canBeCancelled(Order order) {
        return order.getStatus().equals("PENDING");
    }
}
```

**AFTER — expected:**
```java
public class Order {
    private final List<OrderItem> items;
    private String status;

    public Order(List<OrderItem> items) {
        this.items = List.copyOf(items);
        this.status = "PENDING";
    }

    public double total() {
        return items.stream()
            .mapToDouble(i -> i.getPrice() * i.getQuantity())
            .sum();
    }

    public boolean canBeCancelled() {
        return "PENDING".equals(status);
    }

    public void cancel() {
        if (!canBeCancelled()) throw new IllegalStateException("Cannot cancel order in status: " + status);
        this.status = "CANCELLED";
    }
}
```

**Why this pattern:**
- `Order` now owns its business rules — callers ask the order itself
- The `status` field is protected: transitions happen through domain methods

---

## 5. Negative examples — what NOT to do

**Mistake 1: Moving all service methods into the data class without discrimination**
```java
// Not accepted — the class becomes a Large Class; only move cohesive behaviour
public class Order {
    public void sendConfirmationEmail() { /* unrelated to Order's domain */ }
    public void writeToDatabase() { /* infrastructure concern */ }
}
```

**Mistake 2: Keeping public setters after adding domain logic**
```java
// Not accepted — any caller can bypass the domain rule by calling setStatus() directly
public void setStatus(String status) { this.status = status; }
```

---

## 6. Benefits

- **Encapsulation:** Business rules live next to the data they govern
- **Reduced duplication:** Logic that was copied across service classes is now in one place
- **Tell, don't ask:** Callers ask the object to do something instead of reading its fields and deciding externally
