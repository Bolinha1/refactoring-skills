# SKILL: Detecting and Refactoring Long Parameter List — Java

## Source
Based on: https://refactoring.guru/smells/long-parameter-list

---

## 1. What is Long Parameter List

A method has too many parameters — typically more than three or four. Long parameter lists are hard to understand, easy to confuse, and painful to call. They often indicate that related data should be grouped into an object.

**Why this happens:**
- Methods were merged without creating a proper abstraction for the combined data
- Algorithms became more complex over time, requiring more control flags
- Data that naturally belongs together is passed as individual primitives

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Method signature with 4 or more parameters
- [ ] Several parameters of the same type in a row (easy to swap accidentally)
- [ ] Parameters that always appear together across multiple call sites
- [ ] Boolean "flag" parameters that switch the method's behavior
- [ ] Caller must construct many locals just to call the method

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique              |
|----------------------------------------------------------|------------------------------------|
| Parameters are all fields of an existing object          | Preserve Whole Object              |
| Parameters represent a new concept not yet in the model  | Introduce Parameter Object         |
| A parameter can be computed from other parameters        | Replace Parameter with Method Call |
| A boolean flag changes the method behavior               | Split into two explicit methods    |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class OrderService {
    public Receipt createOrder(
            String customerId,
            String customerName,
            String customerEmail,
            String street,
            String city,
            String zipCode,
            String country,
            List<String> productIds,
            String couponCode) {
        // ... long logic
    }
}
```

**AFTER — expected:**
```java
public class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    private final String country;
    // constructor + getters
}

public class OrderRequest {
    private final String customerId;
    private final String customerName;
    private final String customerEmail;
    private final Address shippingAddress;
    private final List<String> productIds;
    private final String couponCode;
    // constructor + getters
}

public class OrderService {
    public Receipt createOrder(OrderRequest request) {
        // ... logic uses request.getCustomerId(), request.getShippingAddress(), etc.
    }
}
```

**Why this pattern:**
- `OrderRequest` and `Address` are first-class concepts — they can be validated, reused, and tested independently
- Callers construct a meaningful object, not a positional argument list

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping unrelated parameters into an object just to reduce count**
```java
// Not accepted — DataHolder has no domain meaning
public Receipt createOrder(DataHolder holder) { ... }
```

**Mistake 2: Using a Map<String, Object> as the "parameter object"**
```java
// Not accepted — loses type safety, IDE support, and discoverability
public Receipt createOrder(Map<String, Object> params) { ... }
```

**Mistake 3: Keeping the long list but adding default-value overloads**
```java
// Not accepted — multiplies the problem without addressing the cause
public Receipt createOrder(String customerId, String name) { ... }
public Receipt createOrder(String customerId, String name, String email) { ... }
public Receipt createOrder(String customerId, String name, String email, Address address) { ... }
```

---

## 6. Benefits

- **Readability:** Named parameter objects make call sites self-documenting
- **Safety:** Positional argument confusion is eliminated
- **Extensibility:** Adding a field to the parameter object does not change every call site
