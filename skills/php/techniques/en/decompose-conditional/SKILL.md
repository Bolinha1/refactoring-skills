# TECHNIQUE: Decompose Conditional — PHP

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
```php
public function calculateCharge(DateTimeImmutable $date, int $quantity, float $unitPrice): float
{
    if ($date >= SUMMER_START && $date <= SUMMER_END) {
        $charge = $quantity * $unitPrice * SUMMER_RATE;
    } else {
        $serviceCharge = $quantity > WINTER_THRESHOLD ? WINTER_SERVICE_CHARGE : 0;
        $charge = $quantity * $unitPrice * WINTER_RATE + $serviceCharge;
    }
    return $charge;
}
```

**AFTER — expected:**
```php
public function calculateCharge(DateTimeImmutable $date, int $quantity, float $unitPrice): float
{
    if ($this->isSummer($date)) {
        return $this->summerCharge($quantity, $unitPrice);
    }
    return $this->winterCharge($quantity, $unitPrice);
}

private function isSummer(DateTimeImmutable $date): bool
{
    return $date >= SUMMER_START && $date <= SUMMER_END;
}

private function summerCharge(int $quantity, float $unitPrice): float
{
    return $quantity * $unitPrice * SUMMER_RATE;
}

private function winterCharge(int $quantity, float $unitPrice): float
{
    $serviceCharge = $quantity > WINTER_THRESHOLD ? WINTER_SERVICE_CHARGE : 0;
    return $quantity * $unitPrice * WINTER_RATE + $serviceCharge;
}
```

**Why this pattern:**
- `isSummer` names the business rule — readers understand it without parsing the date logic
- `summerCharge` and `winterCharge` express pricing intent, not implementation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting the method but giving it a technical name**
```php
// Not accepted — the name describes the mechanism, not the business rule
private function checkDateRange(DateTimeImmutable $date): bool
{
    return $date >= SUMMER_START && $date <= SUMMER_END;
}
```

**Mistake 2: Extracting only the condition but not the branches**
```php
// Not accepted — partial extraction; the branches still need names
if ($this->isSummer($date)) {
    $charge = $quantity * $unitPrice * SUMMER_RATE; // still inline
}
```

**Mistake 3: Decomposing a trivial condition**
```php
// Not accepted — over-engineering for a simple check
private function isNull(mixed $obj): bool { return $obj === null; }
if ($this->isNull($customer)) { ... }
```

---

## 7. Benefits

- **Readability:** The `if` statement reads like a business rule, not a formula
- **Testability:** Each branch can be tested in isolation
- **Documentation:** Method names replace the need for comments
