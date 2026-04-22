# SMELL: Alternative Classes with Different Interfaces — Python

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
| **Extract Superclass** | Once the interfaces match, pull the shared contract up into an abstract base class or Protocol |

---

## 4. Example

**BEFORE — not accepted:**
```python
# Two loggers with completely different interfaces for the same operation
class FileLogger:
    def write_log(self, message: str) -> None:
        # write to file
        pass

class DatabaseLogger:
    def persist_entry(self, text: str, level: str) -> None:
        # write to database
        pass

# Client must branch on the concrete type
class OrderService:
    def __init__(self, use_db: bool):
        self._file_logger = FileLogger()
        self._db_logger = DatabaseLogger()
        self._use_db = use_db

    def process_order(self, order) -> None:
        if self._use_db:
            self._db_logger.persist_entry(str(order), "INFO")
        else:
            self._file_logger.write_log(str(order))
```

**AFTER — expected:**
```python
from abc import ABC, abstractmethod

# Shared interface using ABC or Protocol
class Logger(ABC):
    @abstractmethod
    def log(self, message: str, level: str) -> None: ...


class FileLogger(Logger):
    def log(self, message: str, level: str) -> None:
        # write to file
        pass


class DatabaseLogger(Logger):
    def log(self, message: str, level: str) -> None:
        # write to database
        pass


# Client depends only on the interface
class OrderService:
    def __init__(self, logger: Logger):
        self._logger = logger

    def process_order(self, order) -> None:
        self._logger.log(str(order), "INFO")
```

**Why this pattern:**
- The client no longer needs to know which logger it has
- Adding a third logger requires zero changes to `OrderService`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Wrapping one class in an adapter just to avoid renaming**
```python
# Not accepted — adds indirection without fixing the root problem
class DatabaseLoggerAdapter(FileLogger):
    def __init__(self):
        self._inner = DatabaseLogger()
    def write_log(self, message: str) -> None:
        self._inner.persist_entry(message, "INFO")  # hidden delegation
```

**Mistake 2: Leaving the conditional in place and adding more branches**
```python
# Not accepted — every new logger type adds another branch
if log_type == "file":
    file_logger.write_log(msg)
elif log_type == "db":
    db_logger.persist_entry(msg, "INFO")
elif log_type == "cloud":
    cloud_logger.send(msg)
```

---

## 6. Benefits

- **Interchangeability:** Classes with the same interface can be swapped without touching callers
- **Reduced duplication:** Shared behaviour can be moved to a common supertype
- **Extensibility:** New implementations only need to satisfy the shared contract
