# SKILL: Detecting and Refactoring Data Clumps — Java

## Source
Based on: https://refactoring.guru/smells/data-clumps

---

## 1. What is Data Clumps

Different parts of the code contain identical groups of variables (a clump) — the same fields in multiple classes, or the same parameters appearing together in many method signatures. If you removed one item from the clump and the rest stopped making sense, you have a data clump.

**Why this happens:**
- The relationship between the data items was never formalized as a class
- Copy-paste propagated the group across unrelated places
- The concept existed informally in developers' heads but not in the code

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Three or more fields that appear together in multiple classes
- [ ] The same 2–3 parameters consistently passed together to methods
- [ ] Variables like `startDate`/`endDate`, `latitude`/`longitude`, `street`/`city`/`zip` appearing as separate primitives
- [ ] You must update the same group of fields in several places for one logical change

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique         |
|----------------------------------------------------------|-------------------------------|
| Clump appears as fields in a class                       | Extract Class                 |
| Clump appears in method parameter lists                  | Introduce Parameter Object    |
| A method receives an object but only uses part of it     | Preserve Whole Object         |

**Key test:** delete one item from the clump. If the others lose meaning, the group deserves its own class.

---

## 4. Example

**BEFORE — not accepted:**
```java
public class Order {
    private String street;
    private String city;
    private String zipCode;
    private String country;
    private String customerName;
    private String customerEmail;
    // ...
}

public class Invoice {
    private String street;      // same clump again
    private String city;
    private String zipCode;
    private String country;
    // ...
}

public void ship(String street, String city, String zipCode, String country) { ... }
```

**AFTER — expected:**
```java
public class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    private final String country;

    public Address(String street, String city, String zipCode, String country) {
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
        this.country = country;
    }
    // getters, equals, hashCode
}

public class Order {
    private Address shippingAddress;
    private String customerName;
    private String customerEmail;
}

public class Invoice {
    private Address billingAddress;
}

public void ship(Address destination) { ... }
```

**Why this pattern:**
- `Address` is a named concept — its validation, formatting, and comparison now live in one place
- Every class that holds an address benefits from any improvement to `Address`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping the clump into a Map or generic container**
```java
// Not accepted — loses type safety and intent
Map<String, String> address = new HashMap<>();
address.put("street", "Main St");
```

**Mistake 2: Creating the class but leaving the old separate fields too**
```java
// Not accepted — now there are two representations of the same data
private Address shippingAddress;
private String street;    // duplicate
private String city;      // duplicate
```

**Mistake 3: Grouping data with no cohesion just because they appear together**
```java
// Not accepted — CustomerPreferences bundles unrelated fields into one class
// just because they happened to be in the same method signature
```

---

## 6. Benefits

- **Single source of truth:** The clump's logic (validation, formatting) is centralized
- **Readability:** `order.getShippingAddress()` is more expressive than four separate fields
- **Extensibility:** Adding a new address field requires changing only the `Address` class
