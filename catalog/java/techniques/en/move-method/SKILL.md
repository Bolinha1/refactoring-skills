# TECHNIQUE: Move Method — Java

## Source
Based on: https://refactoring.guru/move-method

---

## 1. Problem

A method is used more in another class than in its own class.
This creates unnecessary coupling and violates the cohesion principle.

---

## 2. Solution

Declare the method in the class that uses it most frequently. Move the original code there.
In the original location, replace the method body with a delegation to the new method —
or remove it entirely if it is no longer needed.

---

## 3. When to apply

- The method accesses data from another class more than its own class's data
- The method would be more useful in the class that consumes it
- Moving the method would reduce or eliminate dependencies between classes
- The **Feature Envy** smell is present (method "envies" another class's data)

---

## 4. Refactoring steps

1. Analyze the method's dependencies within its current class
2. Check if the method exists in superclasses or subclasses (avoid breaking polymorphism)
3. Declare the method in the target class with a contextually appropriate name
4. Obtain a reference to the target class (via field, parameter, or local creation)
5. Turn the original method into a delegation — or delete it if there are no external callers
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public class Account {
    private double balance;
    private ContractType contract;

    public double calculateCharge() {
        // this method uses almost only ContractType data — it's in the wrong place
        if (contract.getType() == ContractType.SPECIAL) {
            return contract.getSpecialRate() * balance * 30;
        }
        return contract.getStandardRate() * balance * 30;
    }
}

public class ContractType {
    private String type;
    private double standardRate;
    private double specialRate;

    // getters...
}
```

**AFTER — expected:**
```java
public class Account {
    private double balance;
    private ContractType contract;

    public double calculateCharge() {
        // delegates to whoever actually owns the data
        return contract.calculateCharge(balance);
    }
}

public class ContractType {
    private String type;
    private double standardRate;
    private double specialRate;

    public double calculateCharge(double balance) {
        if (type.equals("SPECIAL")) {
            return specialRate * balance * 30;
        }
        return standardRate * balance * 30;
    }
}
```

**Why this pattern:**
- `calculateCharge` used `contract.getType()`, `contract.getSpecialRate()` and `contract.getStandardRate()`
- The method belongs to `ContractType` — that is where the required data lives
- `Account` now only coordinates, without needing to know the details of `ContractType`

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving and creating a circular dependency**
```java
// Not accepted — creates mutual dependency
public class ContractType {
    public double calculateCharge(Account account) {
        return rate * account.getBalance() * account.getDays(); // now depends on Account
    }
}
```

**Mistake 2: Moving a method that is part of a polymorphic hierarchy**
```java
// Not accepted — if calculateCharge() is overridden in subclasses,
// moving without care breaks the polymorphic contract
```

**Mistake 3: Moving and keeping the original version, creating duplication**
```java
// Not accepted — two methods with the same behaviour in different classes
```

---

## 7. Benefits

- **Cohesion:** Each class contains the methods that belong to its data
- **Reduced coupling:** Eliminates unnecessary dependencies between classes
- **Maintainability:** Logic changes only affect the correct class
