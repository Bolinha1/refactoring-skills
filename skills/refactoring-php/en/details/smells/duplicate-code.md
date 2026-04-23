# SKILL: Detecting and Refactoring Duplicate Code — PHP

## Source
Based on: https://refactoring.guru/smells/duplicate-code

---

## 1. What is Duplicate Code

Two or more code fragments look almost identical or are structurally similar across different places in the codebase. Any change to the logic must be replicated in every copy, and forgetting one creates a silent bug.

**Why this happens:**
- Copy-paste used as a shortcut instead of abstraction
- Parallel development where two developers solved the same problem independently
- Logic that started similar and diverged slightly, making the duplication invisible

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Same or nearly identical block of code in two or more methods
- [ ] Same computation or expression repeated in several places
- [ ] Two classes contain methods that do essentially the same thing
- [ ] Two subclasses implement methods with the same body
- [ ] You copy-pasted code and tweaked one variable

---

## 3. Treatment techniques (in order of preference)

| Situation found                                        | Recommended technique             |
|--------------------------------------------------------|-----------------------------------|
| Duplicate blocks in the same class                     | Extract Method                    |
| Duplicate blocks in sibling subclasses                 | Pull Up Method                    |
| Similar but not identical blocks in sibling subclasses | Form Template Method              |
| Duplicate code in two unrelated classes                | Extract Class or Move Method      |
| Long duplicate methods that differ only in algorithm   | Substitute Algorithm              |

---

## 4. Example

**BEFORE — not accepted:**
```php
class SalesReport
{
    public function calculateRevenue(array $orders): float
    {
        $total = 0;
        foreach ($orders as $order) {
            if ($order->getStatus() === 'paid') {
                $total += $order->getAmount();
            }
        }
        return $total;
    }
}

class FinanceReport
{
    public function computePaidOrdersTotal(array $orders): float
    {
        $total = 0;
        foreach ($orders as $order) {
            if ($order->getStatus() === 'paid') {
                $total += $order->getAmount();
            }
        }
        return $total;
    }
}
```

**AFTER — expected:**
```php
class OrderCalculator
{
    public static function sumPaidOrders(array $orders): float
    {
        return array_sum(
            array_map(
                fn($o) => $o->getAmount(),
                array_filter($orders, fn($o) => $o->getStatus() === 'paid')
            )
        );
    }
}

class SalesReport
{
    public function calculateRevenue(array $orders): float
    {
        return OrderCalculator::sumPaidOrders($orders);
    }
}

class FinanceReport
{
    public function computePaidOrdersTotal(array $orders): float
    {
        return OrderCalculator::sumPaidOrders($orders);
    }
}
```

**Why this pattern:**
- Business logic lives in one place — a single change fixes all callers
- Both report classes delegate to the shared utility instead of owning the logic

---

## 5. Negative examples — what NOT to do

**Mistake 1: Accepting small differences as justification for duplication**
```php
// Not accepted — "almost the same" still means duplicate
public function calculateRevenue(array $orders): float { /* loop */ }
public function calculateProjectedRevenue(array $orders): float { /* same loop, one extra multiply */ }
```

**Mistake 2: Extracting to a private method but duplicating it in each class**
```php
// Not accepted — sumPaid() is duplicated in both SalesReport and FinanceReport
private function sumPaid(array $orders): float { ... }
```

**Mistake 3: Merging duplicates into a god method with boolean flags**
```php
// Not accepted — creates Feature Envy / Long Method
public function calculate(array $orders, bool $paid, bool $projected, bool $withTaxes): float { ... }
```

---

## 6. Benefits

- **Maintenance:** A single bug fix or rule change applies everywhere automatically
- **Clarity:** Code that expresses a shared concept is easier to understand than scattered copies
- **Trust:** Developers can be confident there are no diverged copies hiding stale logic
