# TECHNIQUE: Replace Conditional with Polymorphism — Java

## Source
Based on: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problem

You have a `switch` or `if/else` chain that performs different behaviours
depending on the object type or a property that simulates a type.

---

## 2. Solution

Create subclasses matching each branch of the conditional. In each subclass,
implement a method containing that branch's logic. Replace the conditional
with a polymorphic method call.

---

## 3. When to apply

- The conditional checks the object type (`instanceof`, type field, constant)
- The same conditional appears in multiple places
- Adding a new type would require modifying all existing conditionals (Open/Closed Principle violation)
- Each branch of the conditional represents a cohesive and distinct behaviour

---

## 4. Refactoring steps

1. If the conditional is inside a method with other responsibilities, apply **Extract Method** first
2. Create a base class (or interface) with the method to be polymorphized
3. For each branch, create a subclass that overrides the method with that branch's logic
4. Move the branch logic to the corresponding subclass method
5. Remove the branch from the original conditional
6. Repeat until the conditional is empty, then declare the method `abstract` in the base class
7. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public class Employee {
    private String type;   // "ENGINEER", "MANAGER", "SALESPERSON"
    private double baseSalary;
    private double sales;

    public double calculateBonus() {
        switch (type) {
            case "ENGINEER":
                return baseSalary * 0.10;
            case "MANAGER":
                return baseSalary * 0.20 + 1000;
            case "SALESPERSON":
                return sales * 0.05;
            default:
                throw new IllegalStateException("Unknown type: " + type);
        }
    }

    public String generateReport() {
        switch (type) {
            case "ENGINEER":
                return "Engineer: delivered projects";
            case "MANAGER":
                return "Manager: led teams";
            case "SALESPERSON":
                return "Salesperson: $" + sales + " in sales";
            default:
                throw new IllegalStateException("Unknown type: " + type);
        }
    }
}
```

**AFTER — expected:**
```java
public abstract class Employee {
    protected double baseSalary;

    public abstract double calculateBonus();
    public abstract String generateReport();
}

public class Engineer extends Employee {
    @Override
    public double calculateBonus() {
        return baseSalary * 0.10;
    }

    @Override
    public String generateReport() {
        return "Engineer: delivered projects";
    }
}

public class Manager extends Employee {
    @Override
    public double calculateBonus() {
        return baseSalary * 0.20 + 1000;
    }

    @Override
    public String generateReport() {
        return "Manager: led teams";
    }
}

public class Salesperson extends Employee {
    private double sales;

    @Override
    public double calculateBonus() {
        return sales * 0.05;
    }

    @Override
    public String generateReport() {
        return "Salesperson: $" + sales + " in sales";
    }
}
```

**Why this pattern:**
- Adding a new employee type → create a new subclass, no changes to existing ones
- No duplicated `switch` across `calculateBonus` and `generateReport`
- Each class has sole responsibility for its own behaviour

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the switch to a Factory and thinking it's solved**
```java
// Not accepted — the switch still exists, just moved to another place
public class EmployeeFactory {
    public static Employee create(String type) {
        switch (type) {
            case "ENGINEER": return new Employee("ENGINEER"); // same class
        }
    }
}
```

**Mistake 2: Creating subclasses but keeping the conditional in methods**
```java
// Not accepted — created inheritance but did not polymorphize the behaviour
public class Engineer extends Employee {
    @Override
    public double calculateBonus() {
        if (type.equals("ENGINEER")) {   // unnecessary conditional
            return baseSalary * 0.10;
        }
        return 0;
    }
}
```

**Mistake 3: Using polymorphism for trivial business conditionals**
```java
// Not accepted — over-engineering for a simple if
// e.g.: if (active) { ... } else { ... }
```

---

## 7. Benefits

- **Open/Closed Principle:** New types do not require modifying existing code
- **Duplicate elimination:** The same conditional does not need to be repeated
- **Readability:** Each class clearly expresses its own behaviour
- **Testability:** Each subclass can be tested in isolation
