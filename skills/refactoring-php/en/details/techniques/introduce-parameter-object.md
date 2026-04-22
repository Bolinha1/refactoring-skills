# TECHNIQUE: Introduce Parameter Object — PHP

## Source
Based on: https://refactoring.guru/introduce-parameter-object

---

## 1. Problem

Several parameters that naturally belong together are always passed as a group to the same methods. The group represents a concept not yet formalized in the codebase.

---

## 2. Solution

Replace the group of parameters with a single value object (DTO or readonly class) that represents the concept. The object becomes a first-class citizen in the domain model.

---

## 3. When to apply

- Three or more parameters that always appear together in method signatures
- The same group appears in multiple methods across the codebase
- The group represents a recognizable domain concept (date range, address, coordinates)
- You want to add behavior (validation, formatting) to the group of data

---

## 4. Refactoring steps

1. Create a new class to represent the parameter group
2. Add the parameters as readonly fields
3. Add constructor validation for invariants
4. Update each method that takes the parameter group to accept the new class instead
5. Update all call sites to construct the new object and pass it
6. Look for behavior (checks, computations) in the callers that could move into the new class
7. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class ReportService
{
    public function findSalesByRange(\DateTimeImmutable $startDate, \DateTimeImmutable $endDate): array { ... }
    public function sumRevenueByRange(\DateTimeImmutable $startDate, \DateTimeImmutable $endDate): float { ... }
    public function invoicesInRange(\DateTimeImmutable $startDate, \DateTimeImmutable $endDate): array { ... }
}
```

**AFTER — expected:**
```php
final class DateRange
{
    public function __construct(
        public readonly \DateTimeImmutable $start,
        public readonly \DateTimeImmutable $end,
    ) {
        if ($end < $start) {
            throw new \InvalidArgumentException('end must be after start');
        }
    }

    public function includes(\DateTimeImmutable $date): bool
    {
        return $date >= $this->start && $date <= $this->end;
    }
}

class ReportService
{
    public function findSalesByRange(DateRange $range): array { ... }
    public function sumRevenueByRange(DateRange $range): float { ... }
    public function invoicesInRange(DateRange $range): array { ... }
}
```

**Why this pattern:**
- `DateRange` validates its own invariant once — callers cannot pass an invalid range
- The `includes()` method captures query logic previously duplicated in every caller

---

## 6. Negative examples — what NOT to do

**Mistake 1: Creating the object but not moving behavior into it**
```php
// Not accepted — DateRange is still just a bag of two dates with no behavior
class DateRange
{
    public \DateTimeImmutable $start;
    public \DateTimeImmutable $end;
}
```

**Mistake 2: Making the object mutable**
```php
// Not accepted — a mutable DateRange can be changed by callers after passing
class DateRange
{
    public function setStart(\DateTimeImmutable $start): void { $this->start = $start; }
    public function setEnd(\DateTimeImmutable $end): void { $this->end = $end; }
}
```

**Mistake 3: Grouping unrelated parameters just to reduce count**
```php
// Not accepted — RequestContext bundles customerId, sessionId, and locale
// for no reason other than there are three of them
```

---

## 7. Benefits

- **Expressiveness:** The parameter group gets a meaningful name in the domain
- **Validation:** The object validates its own invariants once, at construction time
- **Behavior migration:** Logic that works on the group can move inside the object
