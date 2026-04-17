# TECHNIQUE: Move Method — PHP

## Source
Based on: https://refactoring.guru/move-method

---

## 1. Problem

A method is used more in another class than in its own class.
This creates unnecessary coupling and violates the cohesion principle.

---

## 2. Solution

Declare the method in the class that uses it most frequently. Move the original code there.
In the original location, replace the method body with a delegation to the new method —
or remove it entirely if it is no longer needed.

---

## 3. When to apply

- The method accesses properties of another class more than its own class's properties
- The method would be more useful in the class that consumes it
- Moving the method would reduce or eliminate dependencies between classes
- The **Feature Envy** smell is present (method "envies" another class's data)

---

## 4. Refactoring steps

1. Analyze the method's dependencies within its current class
2. Check if the method exists in parent or child classes (avoid breaking polymorphism)
3. Declare the method in the target class with a contextually appropriate name
4. Obtain a reference to the target class (via injected property, parameter, or local instance)
5. Turn the original method into a delegation — or delete it if there are no external callers
6. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class Account
{
    public function __construct(
        private float $balance,
        private ContractType $contract,
    ) {}

    public function calculateCharge(): float
    {
        // this method uses almost only ContractType data — it's in the wrong place
        if ($this->contract->getType() === 'special') {
            return $this->contract->getSpecialRate() * $this->balance * 30;
        }
        return $this->contract->getStandardRate() * $this->balance * 30;
    }
}

class ContractType
{
    public function __construct(
        private string $type,
        private float $standardRate,
        private float $specialRate,
    ) {}

    public function getType(): string        { return $this->type; }
    public function getStandardRate(): float { return $this->standardRate; }
    public function getSpecialRate(): float  { return $this->specialRate; }
}
```

**AFTER — expected:**
```php
class Account
{
    public function __construct(
        private float $balance,
        private ContractType $contract,
    ) {}

    public function calculateCharge(): float
    {
        // delegates to whoever actually owns the data
        return $this->contract->calculateCharge($this->balance);
    }
}

class ContractType
{
    public function __construct(
        private string $type,
        private float $standardRate,
        private float $specialRate,
    ) {}

    public function calculateCharge(float $balance): float
    {
        if ($this->type === 'special') {
            return $this->specialRate * $balance * 30;
        }
        return $this->standardRate * $balance * 30;
    }
}
```

**Why this pattern:**
- `calculateCharge` used `getType()`, `getSpecialRate()` and `getStandardRate()` from `ContractType`
- The method belongs to `ContractType` — that is where the required data lives
- `Account` now only coordinates, without needing to know the details of `ContractType`

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving and creating a mutual dependency**
```php
// Not accepted — creates circular coupling
class ContractType
{
    public function calculateCharge(Account $account): float
    {
        return $this->rate * $account->getBalance() * $account->getDays(); // now depends on Account
    }
}
```

**Mistake 2: Moving a method overridden in subclasses without evaluating the impact**
```php
// Not accepted — if calculateCharge() is overridden in SpecialAccount,
// moving without care breaks the polymorphic contract
```

**Mistake 3: Keeping the original version with duplicated logic**
```php
// Not accepted — two methods with the same behaviour in different classes
```

---

## 7. Benefits

- **Cohesion:** Each class contains the methods that belong to its data
- **Reduced coupling:** Eliminates unnecessary dependencies between classes
- **Maintainability:** Logic changes only affect the correct class
