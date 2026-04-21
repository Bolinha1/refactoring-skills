# SMELL: Middle Man — Java

## Source
Based on: https://refactoring.guru/smells/middle-man

---

## 1. What is it?

A class whose only purpose is to delegate every call to another class. It adds a layer of indirection without adding any value. This is the inverse of Feature Envy: instead of a class that reaches into others, this is a class that others reach through unnecessarily.

---

## 2. Warning signs

- [ ] More than half of a class's methods just delegate to another class with no added logic
- [ ] Removing the class and calling the delegate directly would simplify the code
- [ ] The class was introduced as a facade but the facade now does nothing useful
- [ ] The class has no state of its own and all methods are pass-throughs

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Remove Middle Man** | Let clients call the delegate directly and delete the intermediary class |
| **Inline Class** | When the middle man is small, absorb it into its caller |
| **Replace Delegation with Inheritance** | If the middle man always delegates to the same object, consider having the class extend the delegate instead (only when the IS-A relationship is genuine) |

---

## 4. Example

**BEFORE — not accepted:**
```java
// OrderFacade delegates everything to OrderRepository with no added value
public class OrderFacade {
    private final OrderRepository repository;

    public OrderFacade(OrderRepository repository) {
        this.repository = repository;
    }

    public Order findById(long id) {
        return repository.findById(id);   // pure delegation
    }

    public void save(Order order) {
        repository.save(order);           // pure delegation
    }

    public List<Order> findAll() {
        return repository.findAll();      // pure delegation
    }
}

// Client uses the facade unnecessarily
public class OrderController {
    private final OrderFacade facade;

    public Order getOrder(long id) {
        return facade.findById(id);
    }
}
```

**AFTER — expected:**
```java
// Client depends directly on the repository
public class OrderController {
    private final OrderRepository repository;

    public Order getOrder(long id) {
        return repository.findById(id);
    }
}
```

**Why this pattern:**
- `OrderFacade` added no behaviour — removing it simplifies the dependency graph
- One fewer class to test, navigate, and maintain

---

## 5. Negative examples — what NOT to do

**Mistake 1: Removing a facade that adds cross-cutting concerns**
```java
// Caution — if OrderFacade adds logging, caching, or access control, it is NOT a middle man
public Order findById(long id) {
    log.info("Finding order {}", id);          // real added value
    return repository.findById(id);
}
```

**Mistake 2: Removing a facade used as a seam for testing**
```java
// Caution — if tests mock OrderFacade to isolate the controller, it serves a purpose;
// consider whether dependency injection on the real repository is sufficient before removing
```

---

## 6. Benefits

- **Reduced layers:** Fewer hops between the caller and the real work
- **Simpler mental model:** Developers follow one fewer level of indirection
- **Less maintenance:** Removing unnecessary delegation reduces the surface area to update when the delegate changes
