# SKILL: Detecting and Refactoring Long Parameter List — PHP

## Source
Based on: https://refactoring.guru/smells/long-parameter-list

---

## 1. What is Long Parameter List

A method has too many parameters — typically more than three or four. Long parameter lists are hard to understand, easy to confuse, and painful to call. They often indicate that related data should be grouped into a value object or DTO.

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
```php
class OrderService
{
    public function createOrder(
        string $customerId,
        string $customerName,
        string $customerEmail,
        string $street,
        string $city,
        string $zipCode,
        string $country,
        array $productIds,
        ?string $couponCode = null
    ): Receipt {
        // ... long logic
    }
}
```

**AFTER — expected:**
```php
class Address
{
    public function __construct(
        public readonly string $street,
        public readonly string $city,
        public readonly string $zipCode,
        public readonly string $country,
    ) {}
}

class OrderRequest
{
    public function __construct(
        public readonly string $customerId,
        public readonly string $customerName,
        public readonly string $customerEmail,
        public readonly Address $shippingAddress,
        public readonly array $productIds,
        public readonly ?string $couponCode = null,
    ) {}
}

class OrderService
{
    public function createOrder(OrderRequest $request): Receipt
    {
        // logic uses $request->customerId, $request->shippingAddress, etc.
    }
}
```

**Why this pattern:**
- `OrderRequest` and `Address` are first-class concepts — they can be validated, reused, and tested independently
- Callers construct a meaningful object, not a positional argument list

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping unrelated parameters into a class just to reduce count**
```php
// Not accepted — DataHolder has no domain meaning
public function createOrder(DataHolder $holder): Receipt { ... }
```

**Mistake 2: Using an array as the "parameter object"**
```php
// Not accepted — loses type safety, IDE support, and discoverability
public function createOrder(array $params): Receipt { ... }
```

**Mistake 3: Keeping the long list but adding default-value overloads**
```php
// Not accepted — multiplies the problem without addressing the cause
public function createOrder(string $id, string $name): Receipt { ... }
public function createOrderWithEmail(string $id, string $name, string $email): Receipt { ... }
```

---

## 6. Benefits

- **Readability:** Named parameter objects make call sites self-documenting
- **Safety:** Positional argument confusion is eliminated
- **Extensibility:** Adding a field to the parameter object does not change every call site
