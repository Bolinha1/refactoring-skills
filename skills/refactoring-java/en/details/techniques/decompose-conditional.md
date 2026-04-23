# TECHNIQUE: Decompose Conditional — Java

## Source
Based on: https://refactoring.guru/decompose-conditional

---

## 1. Problem

A complex conditional expression (and the code in its branches) makes it hard to understand what is being tested and what happens in each case.

---

## 2. Solution

Extract the condition and each branch into well-named methods. The method names explain the intent; the bodies explain the implementation.

---

## 3. When to apply

- The condition itself is a complex boolean expression that requires thought to parse
- The code in the true or false branch is several lines and deserves a name
- The conditional appears inside an already-long method (combines with Long Method)
- Reading the condition requires knowledge of domain rules, not just programming logic

---

## 4. Refactoring steps

1. Extract the condition into a method named after the business rule it checks
2. Extract the true branch into a method named after what it does
3. Extract the false branch into a method named after what it does
4. Replace the original `if` with calls to the extracted methods
5. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public double calculateCharge(LocalDate date, int quantity, double unitPrice) {
    double charge;
    if (!date.isBefore(SUMMER_START) && !date.isAfter(SUMMER_END)) {
        charge = quantity * unitPrice * SUMMER_RATE;
    } else {
        charge = quantity * unitPrice * WINTER_RATE +
                 (quantity > WINTER_SERVICE_CHARGE_THRESHOLD ? WINTER_SERVICE_CHARGE : 0);
    }
    return charge;
}
```

**AFTER — expected:**
```java
public double calculateCharge(LocalDate date, int quantity, double unitPrice) {
    if (isSummer(date)) {
        return summerCharge(quantity, unitPrice);
    } else {
        return winterCharge(quantity, unitPrice);
    }
}

private boolean isSummer(LocalDate date) {
    return !date.isBefore(SUMMER_START) && !date.isAfter(SUMMER_END);
}

private double summerCharge(int quantity, double unitPrice) {
    return quantity * unitPrice * SUMMER_RATE;
}

private double winterCharge(int quantity, double unitPrice) {
    double base = quantity * unitPrice * WINTER_RATE;
    double serviceCharge = quantity > WINTER_SERVICE_CHARGE_THRESHOLD ? WINTER_SERVICE_CHARGE : 0;
    return base + serviceCharge;
}
```

**Why this pattern:**
- `isSummer` names the business rule — readers understand it without parsing the date logic
- `summerCharge` and `winterCharge` express pricing intent, not implementation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting the method but giving it a technical name**
```java
// Not accepted — the name describes the mechanism, not the business rule
private boolean checkDateRange(LocalDate date) {
    return !date.isBefore(SUMMER_START) && !date.isAfter(SUMMER_END);
}
```

**Mistake 2: Extracting only the condition but not the branches**
```java
// Not accepted — partial extraction; the branches still need names
if (isSummer(date)) {
    charge = quantity * unitPrice * SUMMER_RATE; // still inline
}
```

**Mistake 3: Decomposing a trivial condition**
```java
// Not accepted — over-engineering for a simple check
private boolean isNull(Object obj) { return obj == null; }
if (isNull(customer)) { ... }
```

---

## 7. Benefits

- **Readability:** The `if` statement reads like a business rule, not a formula
- **Testability:** Each branch can be tested in isolation
- **Documentation:** Method names replace the need for comments
