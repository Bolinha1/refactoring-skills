# SKILL: Detecting and Refactoring Primitive Obsession — PHP

## Source
Based on: https://refactoring.guru/smells/primitive-obsession

---

## 1. What is Primitive Obsession

Overuse of basic types (`string`, `int`, `float`, `bool`, `array`)
to represent domain concepts that deserve their own classes or value objects.

**Why this happens:**
- Creating a new class feels like overkill for something "simple"
- Associative arrays are convenient and quick to use
- The code grows and the array keys become magic strings that are hard to track

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] `string` representing tax ID, phone, ZIP code, email, product code
- [ ] `int` or `string` constant simulating a type (e.g., `const ROLE_ADMIN = 1`)
- [ ] Associative array with magic string keys to structure domain data
- [ ] Multiple primitive parameters that always appear together (e.g., `float $amount, string $currency`)
- [ ] Validation of the same primitive repeated in multiple places

---

## 3. Treatment techniques (in order of preference)

| Situation found                                      | Recommended technique          |
|------------------------------------------------------|-------------------------------|
| Primitive with its own validation rules              | Replace Data Value with Object |
| Multiple primitives that travel together             | Introduce Parameter Object     |
| Primitive passed as a whole group                    | Preserve Whole Object          |
| `int`/`string` simulating an enumerated type         | Replace Type Code with Class   |
| Associative array as a data structure                | Replace Array with Object      |

---

## 4. Example

**BEFORE — not accepted:**
```php
class Customer
{
    public function __construct(
        private string $name,
        private string $taxId,      // raw string, no centralized validation
        private string $phone,
        private string $email,
    ) {}
}

class OrderService
{
    public function create(
        string $customerTaxId,
        float $amount,
        string $currency,
        int $paymentType    // 1 = card, 2 = bank slip, 3 = pix — magic constant
    ): void {
        if ($paymentType === 1) {
            // ...
        }
        // tax ID validation duplicated in multiple places
        if (empty($customerTaxId) || strlen($customerTaxId) !== 11) {
            throw new \InvalidArgumentException("Invalid tax ID");
        }
    }

    public function getAddress(array $data): void
    {
        // magic keys in the array
        $street  = $data['street'];
        $number  = $data['number'];
        $city    = $data['city'];
        $zipCode = $data['zip_code'];
    }
}
```

**AFTER — expected:**
```php
// Value Object for tax ID
final class TaxId
{
    private readonly string $value;

    public function __construct(string $value)
    {
        if (!preg_match('/^\d{11}$/', $value)) {
            throw new \InvalidArgumentException("Invalid tax ID");
        }
        $this->value = $value;
    }

    public function getValue(): string { return $this->value; }
}

// Value Object for Email
final class Email
{
    private readonly string $address;

    public function __construct(string $address)
    {
        if (!filter_var($address, FILTER_VALIDATE_EMAIL)) {
            throw new \InvalidArgumentException("Invalid email");
        }
        $this->address = $address;
    }

    public function getAddress(): string { return $this->address; }
}

// Parameter Object for data that travels together
final class Money
{
    public function __construct(
        private readonly float $amount,
        private readonly string $currency,
    ) {}

    public function getAmount(): float   { return $this->amount; }
    public function getCurrency(): string { return $this->currency; }
}

// Enum instead of integer constant (PHP 8.1+)
enum PaymentType: int
{
    case Card     = 1;
    case BankSlip = 2;
    case Pix      = 3;
}

// Replace Array with Object
final class Address
{
    public function __construct(
        private readonly string $street,
        private readonly string $number,
        private readonly string $city,
        private readonly string $zipCode,
    ) {}

    public function getStreet(): string  { return $this->street; }
    public function getNumber(): string  { return $this->number; }
    public function getCity(): string    { return $this->city; }
    public function getZipCode(): string { return $this->zipCode; }
}

class Customer
{
    public function __construct(
        private string $name,
        private TaxId $taxId,
        private Email $email,
    ) {}
}

class OrderService
{
    public function create(TaxId $customerTaxId, Money $amount, PaymentType $paymentType): void
    {
        if ($paymentType === PaymentType::Card) {
            // ...
        }
    }

    public function getAddress(Address $address): void
    {
        $street = $address->getStreet();
        $city   = $address->getCity();
    }
}
```

**Why this pattern:**
- Tax ID validation is centralized in `TaxId` — not repeated
- `PaymentType` is self-explanatory, no magic integers
- `Money` groups amount and currency — prevents them from travelling separately
- `Address` replaces the array with typed, named properties

---

## 5. Negative examples — what NOT to do

**Mistake 1: `string` for everything**
```php
// Not accepted
public function process(string $taxId, string $status, string $paymentType): void
{
    if ($status === 'active' && $paymentType === 'pix') { ... }
}
```

**Mistake 2: Integer constants instead of Enum**
```php
// Not accepted
const PAYMENT_CARD     = 1;
const PAYMENT_BANK_SLIP = 2;
if ($type === self::PAYMENT_CARD) { ... }
```

**Mistake 3: Associative array with magic keys**
```php
// Not accepted
$address = [
    'street'   => 'Main St',
    'number'   => '123',
    'city'     => 'New York',
    'zip_code' => '10001',
];
```

---

## 6. Benefits

- **Flexibility:** Business rules are encapsulated in the correct object
- **Readability:** Parameters and fields express domain intent
- **Maintainability:** Centralized validation — change in one place, applies everywhere
- **Safety:** Strict type hints eliminate type errors at static analysis time
