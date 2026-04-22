# TECHNIQUE: Inline Class — Java

## Source
Based on: https://refactoring.guru/inline-class

---

## 1. Problem

A class no longer does enough to justify its existence. It has too few responsibilities and is just an unnecessary indirection.

---

## 2. Solution

Move all features of the class into another class, then delete it.

---

## 3. When to apply

- After a refactoring, one class was left with only one or two trivial methods
- A class is used in only one place and adds no real abstraction
- A class that was split too aggressively needs to be consolidated
- The class is a thin wrapper that just delegates everything to another class

---

## 4. Refactoring steps

1. Identify the class to inline and the target class (where features will move)
2. For each public feature in the class to inline:
   - Copy it into the target class
   - Update the target class to delegate to the inline class (for a safe, step-by-step transition)
3. Update all callers to use the target class directly
4. Remove the delegation from the target class (methods now work natively)
5. Delete the inlined class
6. Compile and run tests after each step

---

## 5. Example

**BEFORE — not accepted:**
```java
public class TelephoneNumber {
    private String number;

    public String getNumber() { return number; }
    public void setNumber(String number) { this.number = number; }
}

public class Person {
    private String name;
    private TelephoneNumber telephone;

    public String getTelephoneNumber() {
        return telephone.getNumber(); // TelephoneNumber is just a wrapper
    }
}
```

**AFTER — expected:**
```java
public class Person {
    private String name;
    private String telephoneNumber; // inlined directly

    public String getTelephoneNumber() {
        return telephoneNumber;
    }

    public void setTelephoneNumber(String number) {
        this.telephoneNumber = number;
    }
}
```

**Use case: consolidating Shotgun Surgery**
```java
// BEFORE — four tiny service classes were split too aggressively
public class OrderFinder   { Order find(long id) { ... } }
public class OrderSaver    { void save(Order o) { ... } }
public class OrderUpdater  { void update(Order o) { ... } }
public class OrderDeleter  { void delete(long id) { ... } }

// AFTER — inline all into a single coherent repository
public class OrderRepository {
    public Order find(long id) { ... }
    public void save(Order o) { ... }
    public void update(Order o) { ... }
    public void delete(long id) { ... }
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a class that still has a meaningful concept**
```java
// Not accepted — Money has behavior (arithmetic, formatting) that belongs together
// Do not inline Money into Order just because it has few fields
```

**Mistake 2: Inlining without removing the original class**
```java
// Not accepted — both Person.telephoneNumber and TelephoneNumber exist simultaneously
public class Person {
    private String telephoneNumber;  // inlined copy
    private TelephoneNumber telephone; // still here — creates confusion
}
```

**Mistake 3: Inlining a class used in many places**
```java
// Not accepted — if TelephoneNumber is used by Person, Employee, and Contact,
// inlining it would require duplicating the logic in three places
```

---

## 7. Benefits

- **Simplicity:** Removes unnecessary indirection from the codebase
- **Readability:** Fewer classes to navigate when the abstraction adds no value
- **Preparation:** Often precedes Extract Class — inline first, then re-extract correctly
