# TECHNIQUE: Replace Conditional with Polymorphism — PHP

## Source
Based on: https://refactoring.guru/replace-conditional-with-polymorphism

---

## 1. Problem

You have a `switch`/`match` or `if/elseif/else` chain that performs different behaviours
depending on the object type or a property that simulates a type.

---

## 2. Solution

Create subclasses matching each branch of the conditional. In each subclass,
implement a method containing that branch's logic. Replace the conditional
with a polymorphic method call.

---

## 3. When to apply

- The conditional checks a type field, magic string, or class constant
- The same conditional appears in multiple methods
- Adding a new type would require modifying all existing conditionals (Open/Closed Principle violation)
- Each branch of the conditional represents a cohesive and distinct behaviour

---

## 4. Refactoring steps

1. If the conditional is inside a method with other responsibilities, apply **Extract Method** first
2. Create an `abstract` base class (or interface) with the method to be polymorphized
3. For each branch, create a subclass that overrides the method
4. Move the branch logic to the corresponding subclass method
5. Remove the branch from the original conditional
6. Repeat until the conditional is empty, then declare the method `abstract` in the base class
7. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class Employee
{
    public function __construct(
        private string $type,         // "engineer", "manager", "salesperson"
        private float $baseSalary,
        private float $sales = 0,
    ) {}

    public function calculateBonus(): float
    {
        return match ($this->type) {
            'engineer'    => $this->baseSalary * 0.10,
            'manager'     => $this->baseSalary * 0.20 + 1000,
            'salesperson' => $this->sales * 0.05,
            default       => throw new \InvalidArgumentException("Unknown type: {$this->type}"),
        };
    }

    public function generateReport(): string
    {
        return match ($this->type) {
            'engineer'    => "Engineer: delivered projects",
            'manager'     => "Manager: led teams",
            'salesperson' => "Salesperson: \${$this->sales} in sales",
            default       => throw new \InvalidArgumentException("Unknown type: {$this->type}"),
        };
    }
}
```

**AFTER — expected:**
```php
abstract class Employee
{
    public function __construct(protected float $baseSalary) {}

    abstract public function calculateBonus(): float;
    abstract public function generateReport(): string;
}

class Engineer extends Employee
{
    public function calculateBonus(): float
    {
        return $this->baseSalary * 0.10;
    }

    public function generateReport(): string
    {
        return "Engineer: delivered projects";
    }
}

class Manager extends Employee
{
    public function calculateBonus(): float
    {
        return $this->baseSalary * 0.20 + 1000;
    }

    public function generateReport(): string
    {
        return "Manager: led teams";
    }
}

class Salesperson extends Employee
{
    public function __construct(float $baseSalary, private float $sales)
    {
        parent::__construct($baseSalary);
    }

    public function calculateBonus(): float
    {
        return $this->sales * 0.05;
    }

    public function generateReport(): string
    {
        return "Salesperson: \${$this->sales} in sales";
    }
}
```

**Why this pattern:**
- Adding a new employee type → create a new subclass, no changes to existing ones
- No duplicated `match`/`switch` across `calculateBonus` and `generateReport`
- `abstract` guarantees at definition time that no subclass forgets to implement the methods

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the switch to a Factory and thinking it's solved**
```php
// Not accepted — the conditional still exists, just moved to another place
class EmployeeFactory
{
    public static function create(string $type, float $salary): Employee
    {
        return match ($type) {
            'engineer' => new Employee('engineer', $salary), // same class
        };
    }
}
```

**Mistake 2: Creating subclasses but keeping the conditional in methods**
```php
// Not accepted — created inheritance but did not polymorphize the behaviour
class Engineer extends Employee
{
    public function calculateBonus(): float
    {
        if ($this->type === 'engineer') {   // unnecessary conditional
            return $this->baseSalary * 0.10;
        }
        return 0;
    }
}
```

**Mistake 3: Using polymorphism for trivial business conditionals**
```php
// Not accepted — over-engineering for a simple if
// e.g.: if ($active) { ... } else { ... }
```

---

## 7. Benefits

- **Open/Closed Principle:** New types do not require modifying existing code
- **Duplicate elimination:** The same conditional does not need to be repeated
- **Readability:** Each class clearly expresses its own behaviour
- **Testability:** Each subclass can be tested in isolation
