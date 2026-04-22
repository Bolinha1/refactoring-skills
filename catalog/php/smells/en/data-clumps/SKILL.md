# SKILL: Detecting and Refactoring Data Clumps — PHP

## Source
Based on: https://refactoring.guru/smells/data-clumps

---

## 1. What is Data Clumps

Different parts of the code contain identical groups of variables (a clump) — the same fields in multiple classes, or the same parameters appearing together in many method signatures. If you removed one item from the clump and the rest stopped making sense, you have a data clump.

**Why this happens:**
- The relationship between the data items was never formalized as a value object
- Copy-paste propagated the group across unrelated places
- The concept existed informally in developers' heads but not in the code

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Three or more fields that appear together in multiple classes
- [ ] The same 2–3 parameters consistently passed together to methods
- [ ] Variables like `$startDate`/`$endDate`, `$latitude`/`$longitude`, `$street`/`$city`/`$zip` as separate primitives
- [ ] You must update the same group of fields in several places for one logical change

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique         |
|----------------------------------------------------------|-------------------------------|
| Clump appears as fields in a class                       | Extract Class                 |
| Clump appears in method parameter lists                  | Introduce Parameter Object    |
| A method receives an object but uses only part of it     | Preserve Whole Object         |

**Key test:** delete one item from the clump. If the others lose meaning, the group deserves its own class.

---

## 4. Example

**BEFORE — not accepted:**
```php
class Order
{
    private string $street;
    private string $city;
    private string $zipCode;
    private string $country;
    private string $customerName;
    private string $customerEmail;
}

class Invoice
{
    private string $street;     // same clump again
    private string $city;
    private string $zipCode;
    private string $country;
}

public function ship(string $street, string $city, string $zipCode, string $country): void { ... }
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

    public function format(): string
    {
        return "{$this->street}, {$this->city} {$this->zipCode}, {$this->country}";
    }
}

class Order
{
    public function __construct(
        private Address $shippingAddress,
        private string $customerName,
        private string $customerEmail,
    ) {}
}

class Invoice
{
    public function __construct(private Address $billingAddress) {}
}

public function ship(Address $destination): void { ... }
```

**Why this pattern:**
- `Address` is a named concept — its validation, formatting, and comparison now live in one place
- Every class that holds an address benefits from any improvement to `Address`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping the clump into an array or generic container**
```php
// Not accepted — loses type safety and intent
$address = ['street' => 'Main St', 'city' => 'Springfield'];
```

**Mistake 2: Creating the class but leaving the old separate fields too**
```php
// Not accepted — now there are two representations of the same data
private Address $shippingAddress;
private string $street;    // duplicate
private string $city;      // duplicate
```

**Mistake 3: Grouping data with no cohesion just because they appear together**
```php
// Not accepted — CustomerPreferences bundles unrelated fields into one class
// just because they happened to be in the same method signature
```

---

## 6. Benefits

- **Single source of truth:** The clump's logic (validation, formatting) is centralized
- **Readability:** `$order->getShippingAddress()` is more expressive than four separate fields
- **Extensibility:** Adding a new address field requires changing only the `Address` class
