# SMELL: Speculative Generality — Java

## Source
Based on: https://refactoring.guru/smells/speculative-generality

---

## 1. What is it?

Code that was written to handle requirements that do not yet exist — and may never exist. Abstract classes with only one concrete subclass, hooks that are never called, parameters that are always passed `null`, and interfaces with a single implementation are all signs that someone built in flexibility "just in case."

---

## 2. Warning signs

- [ ] An abstract class or interface has only one concrete implementation
- [ ] A method has parameters that are always `null` or the same constant at every call site
- [ ] A class or method exists only to support future requirements mentioned in comments
- [ ] A hook or extension point has never been used since it was introduced
- [ ] Tests are the only callers of certain methods

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Collapse Hierarchy** | When an abstract class has only one concrete subclass and they can be merged |
| **Inline Class** | When a class exists purely as a future extension point with no current consumers |
| **Remove Parameter** | When a parameter is always passed the same value at every call site |
| **Rename Method** | When abstract names were chosen to accommodate a variety that never materialised |

---

## 4. Example

**BEFORE — not accepted:**
```java
// AbstractNotifier exists only because someone assumed other notifiers would come
public abstract class AbstractNotifier {
    public abstract void send(String message, String channel);
}

public class EmailNotifier extends AbstractNotifier {
    @Override
    public void send(String message, String channel) {
        // channel is always "email" at every call site
        emailClient.send(message);
    }
}

// Every call site passes "email" — the parameter adds no value
notifier.send("Hello", "email");
```

**AFTER — expected:**
```java
public class EmailNotifier {
    public void send(String message) {
        emailClient.send(message);
    }
}

notifier.send("Hello");
```

**Why this pattern:**
- The abstract class and unused parameter were speculative complexity
- If a second notifier type is ever needed, the abstraction can be introduced at that point with real requirements to guide it

---

## 5. Negative examples — what NOT to do

**Mistake 1: Removing abstractions that are genuinely open for extension**
```java
// Caution — if the product roadmap already specifies SMS and push notifications,
// the abstraction is not speculative; keep it
```

**Mistake 2: Removing an interface just because it has one implementation today**
```java
// Caution — interfaces used for dependency injection or testing (mockability) serve
// a real current purpose; verify before removing
```

---

## 6. Benefits

- **Reduced complexity:** The codebase only contains code that serves current needs
- **Easier onboarding:** Developers don't need to understand hooks and abstractions that do nothing yet
- **Simpler refactoring:** When real requirements arrive, you add abstractions informed by actual use cases
