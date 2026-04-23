# SMELL: Alternative Classes with Different Interfaces — PHP

## Source
Based on: https://refactoring.guru/smells/alternative-classes-with-different-interfaces

---

## 1. What is it?

Two or more classes perform similar or identical functions but have different method names and signatures. Because the interfaces differ, client code cannot treat them interchangeably and must know which concrete class it is dealing with.

---

## 2. Warning signs

- [ ] Two classes do similar things but have different method names for equivalent operations
- [ ] Switching from one implementation to the other requires changing all call sites
- [ ] A conditional selects between two classes that could be unified
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
```php
// Two loggers with completely different interfaces for the same operation
class FileLogger {
    public function writeLog(string $message): void {
        // write to file
    }
}

class DatabaseLogger {
    public function persistEntry(string $text, string $level): void {
        // write to database
    }
}

// Client must branch on the concrete type
class OrderService {
    private FileLogger $fileLogger;
    private DatabaseLogger $dbLogger;
    private bool $useDb;

    public function processOrder(Order $order): void {
        if ($this->useDb) {
            $this->dbLogger->persistEntry((string) $order, 'INFO');
        } else {
            $this->fileLogger->writeLog((string) $order);
        }
    }
}
```

**AFTER — expected:**
```php
// Shared interface
interface Logger {
    public function log(string $message, string $level): void;
}

class FileLogger implements Logger {
    public function log(string $message, string $level): void {
        // write to file
    }
}

class DatabaseLogger implements Logger {
    public function log(string $message, string $level): void {
        // write to database
    }
}

// Client depends only on the interface
class OrderService {
    public function __construct(private readonly Logger $logger) {}

    public function processOrder(Order $order): void {
        $this->logger->log((string) $order, 'INFO');
    }
}
```

**Why this pattern:**
- The client no longer needs to know which logger it has
- Adding a third logger requires zero changes to `OrderService`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Wrapping one class in an adapter just to avoid renaming**
```php
// Not accepted — adds indirection without fixing the root problem
class DatabaseLoggerAdapter extends FileLogger {
    private DatabaseLogger $inner;
    public function writeLog(string $message): void {
        $this->inner->persistEntry($message, 'INFO'); // hidden delegation
    }
}
```

**Mistake 2: Leaving the conditional in place and adding more branches**
```php
// Not accepted — every new logger type adds another branch
if ($type === 'file') { $fileLogger->writeLog($msg); }
elseif ($type === 'db') { $dbLogger->persistEntry($msg, 'INFO'); }
elseif ($type === 'cloud') { $cloudLogger->send($msg); }
```

---

## 6. Benefits

- **Interchangeability:** Classes with the same interface can be swapped without touching callers
- **Reduced duplication:** Shared behaviour can be moved to a common supertype
- **Extensibility:** New implementations only need to satisfy the shared contract
