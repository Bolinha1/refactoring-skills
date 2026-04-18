# SKILL: Detecting and Refactoring Primitive Obsession — Java

## Source
Based on: https://refactoring.guru/smells/primitive-obsession

---

## 1. What is Primitive Obsession

Overuse of primitive types (`String`, `int`, `double`, `boolean`) to represent
domain concepts that deserve their own classes.

**Why this happens:**
- Creating a new class feels like overkill for something "simple"
- Primitives are easy and quick to use at the start
- The code grows and the primitive becomes a magic field that is hard to track

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `String` representing tax ID, phone, ZIP code, email, product code
- [ ] `int` or `String` constant simulating an enumerated type (e.g., `ROLE_ADMIN = 1`)
- [ ] Multiple primitive parameters that always appear together (e.g., `double amount, String currency`)
- [ ] Array or `Map` with magic String keys/indices to structure data
- [ ] Validation of the same primitive repeated in multiple places

---

## 3. Treatment techniques (in order of preference)

| Situation found                                      | Recommended technique          |
|------------------------------------------------------|-------------------------------|
| Primitive with its own validation rules              | Replace Data Value with Object |
| Multiple primitives that travel together             | Introduce Parameter Object     |
| Primitive passed as a whole group                    | Preserve Whole Object          |
| `int`/`String` simulating an enumerated type         | Replace Type Code with Class   |
| Conditional code based on the simulated type         | Replace Type Code with Subclasses / State/Strategy |
| `Object[]` or `Map<String, Object>` as a structure  | Replace Array with Object      |

---

## 4. Example

**BEFORE — not accepted:**
```java
public class Customer {
    private String name;
    private String taxId;        // raw string, no centralized validation
    private String phone;        // raw string
    private String email;        // raw string
}

public class OrderService {
    public void create(String customerTaxId, double amount, String currency, int paymentType) {
        // paymentType: 1 = card, 2 = bank slip, 3 = pix — magic constant
        if (paymentType == 1) {
            // ...
        }
        // tax ID validation duplicated in multiple places
        if (customerTaxId == null || customerTaxId.length() != 11) {
            throw new IllegalArgumentException("Invalid tax ID");
        }
    }
}
```

**AFTER — expected:**
```java
// Value Objects for domain-meaningful primitives
public final class TaxId {
    private final String value;

    public TaxId(String value) {
        if (value == null || !value.matches("\\d{11}")) {
            throw new IllegalArgumentException("Invalid tax ID");
        }
        this.value = value;
    }

    public String getValue() { return value; }
}

public final class Email {
    private final String address;

    public Email(String address) {
        if (address == null || !address.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        this.address = address;
    }
}

// Parameter Object for data that travels together
public class Money {
    private final double amount;
    private final String currency;

    public Money(double amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
}

// Enum instead of integer constant
public enum PaymentType {
    CARD, BANK_SLIP, PIX
}

public class Customer {
    private String name;
    private TaxId taxId;
    private Email email;
}

public class OrderService {
    public void create(TaxId customerTaxId, Money amount, PaymentType paymentType) {
        if (paymentType == PaymentType.CARD) {
            // ...
        }
    }
}
```

**Why this pattern:**
- Tax ID validation is centralized in `TaxId` — not repeated
- `PaymentType` is self-explanatory, no magic integers
- `Money` groups amount and currency — prevents them from travelling separately

---

## 5. Negative examples — what NOT to do

**Mistake 1: String for everything**
```java
// Not accepted
public void process(String taxId, String status, String paymentType) {
    if (status.equals("ACTIVE") && paymentType.equals("PIX")) { ... }
}
```

**Mistake 2: Integer constants instead of enum**
```java
// Not accepted
public static final int PAYMENT_CARD = 1;
public static final int PAYMENT_SLIP = 2;
if (type == PAYMENT_CARD) { ... }
```

**Mistake 3: Array with magic indices**
```java
// Not accepted
String[] address = new String[4];
address[0] = "Main St";
address[1] = "123";
address[2] = "NY";
address[3] = "10001";
```

---

## 6. Benefits

- **Flexibility:** Business rules are encapsulated in the correct object
- **Readability:** Parameters and fields express domain intent
- **Maintainability:** Centralized validation — change in one place, applies everywhere
- **Safety:** Impossible to pass a tax ID where an email is expected (strong typing)
