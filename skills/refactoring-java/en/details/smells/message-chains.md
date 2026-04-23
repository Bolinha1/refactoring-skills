# SMELL: Message Chains — Java

## Source
Based on: https://refactoring.guru/smells/message-chains

---

## 1. What is it?

A client asks an object for another object, which asks yet another object, forming a chain: `a.getB().getC().getD().doSomething()`. The client is coupled to the entire chain of intermediaries. If any step changes its structure, the client breaks.

---

## 2. Warning signs

- [ ] Method calls are chained three or more levels deep: `order.getCustomer().getAddress().getCity()`
- [ ] The same chain appears in multiple places in the codebase
- [ ] You have to navigate a chain of objects just to retrieve a single value
- [ ] A chain appears in a loop or conditional, amplifying coupling
- [ ] Adding a layer between two objects in the chain requires updating every call site

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Hide Delegate** | Add a shortcut method to the first object in the chain so clients don't need to navigate the intermediaries |
| **Extract Method** | Extract the chain into a named method that hides the navigation |

---

## 4. Example

**BEFORE — not accepted:**
```java
// Client navigates the full object graph just to get a city name
public class InvoiceService {
    public String getDeliveryCity(Order order) {
        return order.getCustomer().getAddress().getCity();
    }

    public boolean isLocalDelivery(Order order) {
        return order.getCustomer().getAddress().getCity().equals("São Paulo");
    }
}
```

**AFTER — expected:**
```java
// Add a shortcut on Order (Hide Delegate)
public class Order {
    public String getDeliveryCity() {
        return customer.getAddress().getCity();
    }
}

// Client no longer knows about Customer or Address
public class InvoiceService {
    public String getDeliveryCity(Order order) {
        return order.getDeliveryCity();
    }

    public boolean isLocalDelivery(Order order) {
        return "São Paulo".equals(order.getDeliveryCity());
    }
}
```

**Why this pattern:**
- `InvoiceService` is decoupled from `Customer` and `Address`
- Renaming `Address.getCity()` to `Address.getCityName()` only requires one change instead of many

---

## 5. Negative examples — what NOT to do

**Mistake 1: Assigning the chain to a local variable without breaking the dependency**
```java
// Not accepted — the coupling is still there; only the syntax changed
Address address = order.getCustomer().getAddress();
String city = address.getCity();
```

**Mistake 2: Adding a method that returns the intermediate object**
```java
// Not accepted — now callers chain off the returned object; same problem, different entry point
public Customer getCustomer() { return this.customer; }
```

---

## 6. Benefits

- **Lower coupling:** Clients depend only on their direct collaborator
- **Easier refactoring:** Internal navigation can be changed without updating every call site
- **More readable:** `order.getDeliveryCity()` is clearer than `order.getCustomer().getAddress().getCity()`
