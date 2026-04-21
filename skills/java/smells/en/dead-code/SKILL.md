# SKILL: Detecting and Refactoring Dead Code — Java

## Source
Based on: https://refactoring.guru/smells/dead-code

---

## 1. What is Dead Code

Variables, parameters, fields, methods, or entire classes that are no longer used anywhere. Dead code clutters the codebase, misleads future readers into thinking it is relevant, and must still be read and understood even though it does nothing.

**Why this happens:**
- Requirements changed but the old implementation was not removed
- A feature was disabled without deleting the supporting code
- Refactoring produced unreachable code paths that were never cleaned up
- Fear of deletion: "we might need it later"

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] IDE highlights an unused import, field, variable, or method
- [ ] A method is `private` and has no callers in the class
- [ ] Code inside an `if` branch can never execute (impossible condition)
- [ ] A class has no instantiations or usages anywhere in the project
- [ ] Commented-out code blocks that have been there for more than one sprint
- [ ] A parameter is never read inside the method

---

## 3. Treatment techniques (in order of preference)

| Situation found                               | Recommended technique        |
|-----------------------------------------------|------------------------------|
| Unused method, field, or variable             | Delete it                    |
| Unused parameter                              | Remove Parameter             |
| Entire unused class                           | Delete it                    |
| Commented-out code block                      | Delete it (version control has the history) |
| Unreachable code path                         | Delete it                    |

**Rule:** version control is your safety net. Delete confidently — the code is not gone, just archived.

---

## 4. Example

**BEFORE — not accepted:**
```java
public class CustomerService {

    // Old import no longer needed
    import java.util.LinkedList;

    private String legacySystemId;  // never set or read since v2 migration

    public Customer findById(long id) { ... }

    // This was the old search before ElasticSearch was integrated
    // private Customer linearSearch(String name) {
    //     for (Customer c : allCustomers) {
    //         if (c.getName().equals(name)) return c;
    //     }
    //     return null;
    // }

    private void syncWithLegacySystem() {
        // TODO: remove after migration complete (migration was done 18 months ago)
    }

    public void notifyCustomer(Customer customer, String format, boolean legacy) {
        if (legacy) {
            // legacy path — removed in v2, this branch is never true
            sendFax(customer);
        } else {
            sendEmail(customer);
        }
    }
}
```

**AFTER — expected:**
```java
public class CustomerService {

    public Customer findById(long id) { ... }

    public void notifyCustomer(Customer customer) {
        sendEmail(customer);
    }
}
```

**Why this pattern:**
- Every line that remains is load-bearing; readers can trust nothing is orphaned
- Smaller classes are faster to understand, test, and change

---

## 5. Negative examples — what NOT to do

**Mistake 1: Commenting out code instead of deleting it**
```java
// Not accepted — commented-out code is just dead code in disguise
// public void oldMethod() { ... }
```

**Mistake 2: Keeping dead code "just in case"**
```java
// Not accepted — version control stores the history; the fear of deletion is unfounded
private void migrateToLegacyFormat() { ... } // "might need later"
```

**Mistake 3: Adding a deprecation annotation instead of deleting**
```java
// Not accepted — @Deprecated keeps the code visible and confusing
// Delete it if there are no external callers
@Deprecated
public void oldNotify(Customer customer) { ... }
```

---

## 6. Benefits

- **Signal-to-noise:** Every remaining line is relevant — readers trust the codebase
- **Build speed:** Fewer classes means faster compile and test cycles
- **Clarity:** Dead code misleads — its absence removes false trails
