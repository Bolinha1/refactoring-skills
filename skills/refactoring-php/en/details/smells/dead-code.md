# SKILL: Detecting and Refactoring Dead Code — PHP

## Source
Based on: https://refactoring.guru/smells/dead-code

---

## 1. What is Dead Code

Variables, parameters, fields, methods, or entire classes that are no longer used anywhere. Dead code clutters the codebase, misleads future readers into thinking it is relevant, and must still be read and understood even though it does nothing.

**Why this happens:**
- Requirements changed but the old implementation was not removed
- A feature was disabled without deleting the supporting code
- Refactoring produced unreachable code paths that were never cleaned up
- Fear of deletion: "we might need it later"

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] IDE or static analysis (PHPStan, Psalm) highlights an unused import, property, or method
- [ ] A method is `private` and has no callers in the class
- [ ] Code inside an `if` branch can never execute (impossible condition)
- [ ] A class has no instantiations or usages anywhere in the project
- [ ] Commented-out code blocks that have been there for more than one sprint
- [ ] A parameter is never read inside the method

---

## 3. Treatment techniques (in order of preference)

| Situation found                               | Recommended technique        |
|-----------------------------------------------|------------------------------|
| Unused method, property, or variable          | Delete it                    |
| Unused parameter                              | Remove Parameter             |
| Entire unused class                           | Delete it                    |
| Commented-out code block                      | Delete it (git has the history) |
| Unreachable code path                         | Delete it                    |

**Rule:** version control is your safety net. Delete confidently — the code is not gone, just archived.

---

## 4. Example

**BEFORE — not accepted:**
```php
use App\Legacy\LegacyClient; // never used since migration

class CustomerService
{
    private ?string $legacySystemId = null; // never set or read since v2 migration

    public function findById(int $id): Customer { ... }

    // Old search before ElasticSearch was integrated
    // private function linearSearch(string $name): ?Customer
    // {
    //     foreach ($this->allCustomers as $customer) {
    //         if ($customer->getName() === $name) return $customer;
    //     }
    //     return null;
    // }

    private function syncWithLegacySystem(): void
    {
        // TODO: remove after migration complete (migration was done 18 months ago)
    }

    public function notifyCustomer(Customer $customer, string $format, bool $legacy = false): void
    {
        if ($legacy) {
            // legacy path — removed in v2, this branch is never true
            $this->sendFax($customer);
        } else {
            $this->sendEmail($customer);
        }
    }
}
```

**AFTER — expected:**
```php
class CustomerService
{
    public function findById(int $id): Customer { ... }

    public function notifyCustomer(Customer $customer): void
    {
        $this->sendEmail($customer);
    }
}
```

**Why this pattern:**
- Every remaining line is load-bearing; readers can trust nothing is orphaned
- Smaller classes are faster to understand, test, and change

---

## 5. Negative examples — what NOT to do

**Mistake 1: Commenting out code instead of deleting it**
```php
// Not accepted — commented-out code is dead code in disguise
// public function oldMethod(): void { ... }
```

**Mistake 2: Keeping dead code "just in case"**
```php
// Not accepted — git stores the history; the fear of deletion is unfounded
private function migrateToLegacyFormat(): void { ... } // "might need later"
```

**Mistake 3: Using @deprecated instead of deleting**
```php
// Not accepted — keeps the dead code visible and confusing
/** @deprecated Use notifyCustomer() instead */
public function oldNotify(Customer $customer): void { ... }
```

---

## 6. Benefits

- **Signal-to-noise:** Every remaining line is relevant — readers trust the codebase
- **Autoload speed:** Fewer classes means faster autoloader resolution
- **Clarity:** Dead code misleads — its absence removes false trails
