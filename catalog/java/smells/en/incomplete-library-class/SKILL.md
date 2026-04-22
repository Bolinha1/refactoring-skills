# SMELL: Incomplete Library Class — Java

## Source
Based on: https://refactoring.guru/smells/incomplete-library-class

---

## 1. What is it?

A library or framework class lacks a method you need, and because you cannot modify the library, you end up placing the missing behaviour in a utility class, a static helper, or a subclass. The logic that belongs to the library leaks into your own code.

---

## 2. Warning signs

- [ ] A utility class contains methods that only exist because a library class is missing them
- [ ] You subclass a library class just to add one or two missing methods
- [ ] Static helper methods receive a library object as their only parameter
- [ ] The same "missing" operations are re-implemented in multiple places across the codebase
- [ ] Comments like "should be in library X" appear next to helper methods

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Introduce Foreign Method** | Add the missing operation as a method in your own class, passing the library object as a parameter |
| **Introduce Local Extension** | Create a subclass or wrapper of the library class that adds all missing methods in one place |

---

## 4. Example

**BEFORE — not accepted:**
```java
import java.time.LocalDate;

// Missing helper scattered as a static method
public class DateUtils {
    // Should be on LocalDate itself, but we can't modify it
    public static boolean isWeekend(LocalDate date) {
        return date.getDayOfWeek().getValue() >= 6;
    }

    public static LocalDate nextWorkingDay(LocalDate date) {
        LocalDate next = date.plusDays(1);
        while (isWeekend(next)) {
            next = next.plusDays(1);
        }
        return next;
    }
}

// Usage is awkward — you always have to remember to call DateUtils instead of the date itself
LocalDate next = DateUtils.nextWorkingDay(LocalDate.now());
```

**AFTER — expected:**
```java
import java.time.LocalDate;

// Introduce Local Extension: wrap LocalDate with the missing behaviour
public class BusinessDate {
    private final LocalDate date;

    public BusinessDate(LocalDate date) { this.date = date; }

    public static BusinessDate of(LocalDate date) { return new BusinessDate(date); }

    public boolean isWeekend() {
        return date.getDayOfWeek().getValue() >= 6;
    }

    public BusinessDate nextWorkingDay() {
        BusinessDate next = new BusinessDate(date.plusDays(1));
        while (next.isWeekend()) {
            next = new BusinessDate(next.date.plusDays(1));
        }
        return next;
    }

    public LocalDate toLocalDate() { return date; }
}

// Usage reads naturally
BusinessDate next = BusinessDate.of(LocalDate.now()).nextWorkingDay();
```

**Why this pattern:**
- All business-date logic lives in `BusinessDate` — no scattered static helpers
- Adding more missing behaviour has one obvious place

---

## 5. Negative examples — what NOT to do

**Mistake 1: Duplicating the missing method across multiple utility classes**
```java
// Not accepted — OrderUtils.isWeekend(), ReportUtils.isWeekend(), etc.
// creates divergence when the rule needs to change
```

**Mistake 2: Subclassing a final library class**
```java
// Not accepted — many library classes are final or sealed; use composition/wrapping instead
class ExtendedLocalDate extends LocalDate { ... } // compile error if LocalDate is final
```

---

## 6. Benefits

- **Single source of truth:** Missing behaviour lives in one extension class
- **Readable call sites:** The extension reads like native methods on the original class
- **Isolated from library changes:** Wrapping is safer than subclassing when the library evolves
