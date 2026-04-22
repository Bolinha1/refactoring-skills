# SMELL: Alternative Classes with Different Interfaces — Java

## Source
Based on: https://refactoring.guru/smells/alternative-classes-with-different-interfaces

---

## 1. What is it?

Two or more classes perform similar or identical functions but have different method names and signatures. Because the interfaces differ, client code cannot treat them interchangeably and must know which concrete class it is dealing with.

---

## 2. Warning signs

- [ ] Two classes do similar things but have different method names for equivalent operations
- [ ] Switching from one implementation to the other requires changing all call sites
- [ ] A conditional (`if`/`instanceof`) selects between two classes that could be unified
- [ ] One class was created as a replacement for another but never shared its interface
- [ ] Duplicate logic appears in both classes because they cannot share a common abstraction

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Rename Method** | Align method names so both classes expose the same interface |
| **Move Method** | Move missing methods to the class that lacks them until both classes are equivalent |
| **Extract Superclass** | Once the interfaces match, pull the shared contract up into an abstract class or interface |

---

## 4. Example

**BEFORE — not accepted:**
```java
// Two loggers with completely different interfaces for the same operation
public class FileLogger {
    public void writeLog(String message) {
        // write to file
    }
}

public class DatabaseLogger {
    public void persistEntry(String text, String level) {
        // write to database
    }
}

// Client must branch on the concrete type
public class OrderService {
    private FileLogger fileLogger;
    private DatabaseLogger dbLogger;
    private boolean useDb;

    public void processOrder(Order order) {
        if (useDb) {
            dbLogger.persistEntry(order.toString(), "INFO");
        } else {
            fileLogger.writeLog(order.toString());
        }
    }
}
```

**AFTER — expected:**
```java
// Shared interface
public interface Logger {
    void log(String message, String level);
}

public class FileLogger implements Logger {
    @Override
    public void log(String message, String level) {
        // write to file
    }
}

public class DatabaseLogger implements Logger {
    @Override
    public void log(String message, String level) {
        // write to database
    }
}

// Client depends only on the interface
public class OrderService {
    private final Logger logger;

    public OrderService(Logger logger) {
        this.logger = logger;
    }

    public void processOrder(Order order) {
        logger.log(order.toString(), "INFO");
    }
}
```

**Why this pattern:**
- The client no longer needs to know which logger it has
- Adding a third logger requires zero changes to `OrderService`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Wrapping one class in an adapter just to avoid renaming**
```java
// Not accepted — adds indirection without fixing the root problem
public class DatabaseLoggerAdapter extends FileLogger {
    private DatabaseLogger inner;
    @Override
    public void writeLog(String message) {
        inner.persistEntry(message, "INFO"); // hidden delegation
    }
}
```

**Mistake 2: Leaving the conditional in place and adding more branches**
```java
// Not accepted — every new logger type adds another branch
if (type.equals("file")) { fileLogger.writeLog(msg); }
else if (type.equals("db")) { dbLogger.persistEntry(msg, "INFO"); }
else if (type.equals("cloud")) { cloudLogger.send(msg); }
```

---

## 6. Benefits

- **Interchangeability:** Classes with the same interface can be swapped without touching callers
- **Reduced duplication:** Shared behaviour can be moved to a common supertype
- **Extensibility:** New implementations only need to satisfy the shared contract
