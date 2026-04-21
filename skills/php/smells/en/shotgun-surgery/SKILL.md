# SKILL: Detecting and Refactoring Shotgun Surgery — PHP

## Source
Based on: https://refactoring.guru/smells/shotgun-surgery

---

## 1. What is Shotgun Surgery

A single logical change requires modifying many different classes at the same time. Making one conceptual change "fires" edits across the codebase like a shotgun blast — touching many files for what should be a localized fix.

The inverse of Divergent Change: many classes, one reason to change.

**Why this happens:**
- A responsibility was fragmented across many classes instead of living in one place
- Logic that should be centralized was duplicated or distributed for "flexibility"
- Cross-cutting concerns (logging, auditing, notifications) were handled inline everywhere

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Adding a new feature forces you to edit 5+ unrelated classes
- [ ] Renaming a concept requires touching dozens of files
- [ ] A single business rule is enforced in multiple places
- [ ] You always need to make the same change in several places simultaneously
- [ ] Test failures cascade across many test classes for a single-concept change

---

## 3. Treatment techniques (in order of preference)

| Situation found                                                | Recommended technique   |
|----------------------------------------------------------------|-------------------------|
| Scattered behavior all relates to one concept                  | Move Method + Move Field |
| Multiple small classes all contributing to one concept         | Inline Class            |
| Cross-cutting concern repeated everywhere                      | Extract Class           |
| Duplicated business rule enforced in many places               | Move Method to one owner |

---

## 4. Example

**BEFORE — not accepted:**
```php
// Adding a "currency" concept requires changing ALL of these:

class Product
{
    private float $price; // must add currency field here
}

class Order
{
    public function getTotal(): float { ... } // must format with currency
}

class Invoice
{
    public function formatAmount(float $amount): string
    {
        return '$' . number_format($amount, 2); // must use currency
    }
}

class Receipt
{
    public function printTotal(float $amount): void
    {
        echo $amount; // must use currency
    }
}
```

**AFTER — expected:**
```php
final class Money
{
    public function __construct(
        private readonly float $amount,
        private readonly string $currency,
    ) {}

    public function add(Money $other): Money
    {
        if ($this->currency !== $other->currency) {
            throw new \InvalidArgumentException('Currency mismatch');
        }
        return new Money($this->amount + $other->amount, $this->currency);
    }

    public function format(): string
    {
        $symbols = ['USD' => '$', 'BRL' => 'R$', 'EUR' => '€'];
        $symbol = $symbols[$this->currency] ?? $this->currency;
        return $symbol . number_format($this->amount, 2);
    }
}

// Now Product, Order, Invoice, Receipt all use Money — one concept, one place
class Product
{
    public function __construct(private Money $price) {}
}
```

**Why this pattern:**
- `Money` centralizes all currency behavior — adding a new currency format touches one class
- All callers benefit automatically

---

## 5. Negative examples — what NOT to do

**Mistake 1: Centralizing the data but not the behavior**
```php
// Not accepted — MoneyDto groups the fields but formatting/arithmetic stay scattered
class MoneyDto
{
    public float $amount;
    public string $currencyCode;
}
```

**Mistake 2: Using a static utility class as the single point**
```php
// Not accepted — MoneyHelper is a hidden smell; logic still fragmented among callers
class MoneyHelper
{
    public static function format(float $amount, string $currency): string { ... }
}
```

**Mistake 3: Inline class too aggressively, creating a new god class**
```php
// Not accepted — inlining all small classes into one megaclass trades Shotgun Surgery
// for Divergent Change
```

---

## 6. Benefits

- **Locality:** One business concept = one class to change
- **Safety:** Changes are easier to reason about when blast radius is small
- **Discoverability:** All behavior for a concept is in one obvious place
