# SMELL: Refused Bequest — Java

## Source
Based on: https://refactoring.guru/smells/refused-bequest

---

## 1. What is it?

A subclass inherits methods or data from its parent class but does not use or actively overrides them to throw exceptions. The child rejects part of what the parent gives it, which breaks the Liskov Substitution Principle: a subclass should be usable wherever its parent is expected.

---

## 2. Warning signs

- [ ] A subclass overrides a parent method only to throw `UnsupportedOperationException` or similar
- [ ] Inherited fields are never read or written by the subclass
- [ ] Tests that expect base-class behaviour fail when given the subclass
- [ ] The subclass uses only a small fraction of the parent's public API
- [ ] The relationship reads "is-a" only on paper — behaviourally it is not

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Replace Inheritance with Delegation** | The subclass wants some parent behaviour but not all — wrap instead of extend |
| **Extract Superclass** | Pull only the shared behaviour into a new parent; leave the rest in sibling classes |
| **Push Down Method / Push Down Field** | Move unwanted inherited members down out of the parent so the subclass never receives them |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class Bird {
    public String getName() { return name; }
    public void fly() { /* flap wings */ }
    public void land() { /* touchdown */ }
}

// Penguin IS-A Bird on paper, but cannot fly
public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins cannot fly");
    }
}

// Caller breaks at runtime
Bird b = new Penguin();
b.fly(); // throws!
```

**AFTER — expected:**
```java
// Split the hierarchy along actual capabilities
public abstract class Bird {
    public abstract String getName();
}

public interface Flyable {
    void fly();
    void land();
}

public class Sparrow extends Bird implements Flyable {
    @Override public String getName() { return "Sparrow"; }
    @Override public void fly() { /* flap */ }
    @Override public void land() { /* touchdown */ }
}

public class Penguin extends Bird {
    @Override public String getName() { return "Penguin"; }
    public void swim() { /* splash */ }
}
```

**Why this pattern:**
- `Bird` now only contains behaviour shared by ALL birds
- `Flyable` is an opt-in capability — penguins simply don't implement it
- Code that calls `fly()` must hold a `Flyable`, not just any `Bird`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the throw and documenting it**
```java
// Not accepted — LSP is still violated; callers cannot trust the contract
@Override
public void fly() {
    // Penguins don't fly — documented but still broken
    throw new UnsupportedOperationException();
}
```

**Mistake 2: Returning a no-op instead of throwing**
```java
// Not accepted — silent failures hide bugs; callers assume the action happened
@Override
public void fly() { /* do nothing */ }
```

---

## 6. Benefits

- **LSP compliance:** Every subclass honours the full contract of its parent
- **Safer polymorphism:** Code that iterates a collection of `Bird` objects won't crash
- **Cleaner hierarchy:** Capabilities become explicit interfaces rather than hidden exceptions
