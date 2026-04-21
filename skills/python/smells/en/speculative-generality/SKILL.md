# SMELL: Speculative Generality — Python

## Source
Based on: https://refactoring.guru/smells/speculative-generality

---

## 1. What is it?

Code that was written to handle requirements that do not yet exist — and may never exist. Abstract base classes with only one concrete subclass, hooks that are never called, parameters that are always passed `None`, and protocols with a single implementation are all signs that someone built in flexibility "just in case."

---

## 2. Warning signs

- [ ] An ABC or Protocol has only one concrete implementation
- [ ] A function has parameters that are always `None` or the same constant at every call site
- [ ] A class or function exists only to support future requirements mentioned in comments
- [ ] A hook or extension point has never been used since it was introduced
- [ ] Tests are the only callers of certain functions

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Collapse Hierarchy** | When an ABC has only one concrete subclass and they can be merged |
| **Inline Class** | When a class exists purely as a future extension point with no current consumers |
| **Remove Parameter** | When a parameter is always passed the same value at every call site |
| **Rename Method** | When abstract names were chosen to accommodate a variety that never materialised |

---

## 4. Example

**BEFORE — not accepted:**
```python
from abc import ABC, abstractmethod

# AbstractNotifier exists only because someone assumed other notifiers would come
class AbstractNotifier(ABC):
    @abstractmethod
    def send(self, message: str, channel: str) -> None: ...


class EmailNotifier(AbstractNotifier):
    def send(self, message: str, channel: str) -> None:
        # channel is always "email" at every call site
        self._email_client.send(message)


# Every call site passes "email" — the parameter adds no value
notifier.send("Hello", "email")
```

**AFTER — expected:**
```python
class EmailNotifier:
    def send(self, message: str) -> None:
        self._email_client.send(message)


notifier.send("Hello")
```

**Why this pattern:**
- The ABC and unused parameter were speculative complexity
- If a second notifier type is ever needed, the abstraction can be introduced at that point with real requirements to guide it

---

## 5. Negative examples — what NOT to do

**Mistake 1: Removing abstractions that are genuinely open for extension**
```python
# Caution — if the product roadmap already specifies SMS and push notifications,
# the abstraction is not speculative; keep it
```

**Mistake 2: Removing an ABC just because it has one implementation today**
```python
# Caution — ABCs used for dependency injection or testing serve a real current
# purpose; verify before removing
```

---

## 6. Benefits

- **Reduced complexity:** The codebase only contains code that serves current needs
- **Easier onboarding:** Developers don't need to understand hooks and abstractions that do nothing yet
- **Simpler refactoring:** When real requirements arrive, you add abstractions informed by actual use cases
