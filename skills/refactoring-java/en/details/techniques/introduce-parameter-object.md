# TECHNIQUE: Introduce Parameter Object — Java

## Source
Based on: https://refactoring.guru/introduce-parameter-object

---

## 1. Problem

Several parameters that naturally belong together are always passed as a group to the same methods. The group represents a concept not yet formalized in the codebase.

---

## 2. Solution

Replace the group of parameters with a single object that represents the concept. The object becomes a first-class citizen in the domain model.

---

## 3. When to apply

- Three or more parameters that always appear together in method signatures
- The same group appears in multiple methods across the codebase
- The group represents a recognizable domain concept (date range, address, coordinates)
- You want to add behavior (validation, formatting) to the group of data

---

## 4. Refactoring steps

1. Create a new class to represent the parameter group
2. Add the parameters as fields in the new class (make them immutable if they represent a value)
3. Add a constructor that takes the original parameters
4. Update each method that takes the parameter group to accept the new class instead
5. Update all call sites to construct the new object and pass it
6. Look for behavior (checks, computations) in the callers that could move into the new class
7. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public class ReportService {
    public List<Sale> findSalesByRange(LocalDate startDate, LocalDate endDate) { ... }
    public double sumRevenueByRange(LocalDate startDate, LocalDate endDate) { ... }
    public List<Invoice> invoicesInRange(LocalDate startDate, LocalDate endDate) { ... }
}
```

**AFTER — expected:**
```java
public class DateRange {
    private final LocalDate start;
    private final LocalDate end;

    public DateRange(LocalDate start, LocalDate end) {
        if (end.isBefore(start)) throw new IllegalArgumentException("end must be after start");
        this.start = start;
        this.end = end;
    }

    public boolean includes(LocalDate date) {
        return !date.isBefore(start) && !date.isAfter(end);
    }

    public LocalDate getStart() { return start; }
    public LocalDate getEnd() { return end; }
}

public class ReportService {
    public List<Sale> findSalesByRange(DateRange range) { ... }
    public double sumRevenueByRange(DateRange range) { ... }
    public List<Invoice> invoicesInRange(DateRange range) { ... }
}
```

**Why this pattern:**
- `DateRange` validates its own invariant (end ≥ start) once — callers cannot pass an invalid range
- The `includes()` method captures query logic that was previously duplicated in every caller

---

## 6. Negative examples — what NOT to do

**Mistake 1: Creating the object but not moving behavior into it**
```java
// Not accepted — DateRange is still just a bag of two dates with no behavior
public class DateRange {
    public LocalDate start;
    public LocalDate end;
}
```

**Mistake 2: Making the object mutable**
```java
// Not accepted — a mutable parameter object can be changed by callers after passing
public class DateRange {
    public void setStart(LocalDate start) { this.start = start; }
    public void setEnd(LocalDate end) { this.end = end; }
}
```

**Mistake 3: Grouping unrelated parameters just to reduce count**
```java
// Not accepted — RequestContext bundles customerId, sessionId, and locale
// for no reason other than there are three of them
```

---

## 7. Benefits

- **Expressiveness:** The parameter group gets a meaningful name in the domain
- **Validation:** The object validates its own invariants once, at construction time
- **Behavior migration:** Logic that works on the group can move inside the object
