# TECHNIQUE: Move Field — PHP

## Source
Based on: https://refactoring.guru/move-field

---

## 1. Problem

A field is used more by another class than by its own class — other classes read or write it more frequently than the class that contains it.

---

## 2. Solution

Create a new field in the target class. Update all references to use the new location, then delete the old field.

---

## 3. When to apply

- Methods in another class reference the field more than the class that owns it
- You are doing Extract Class and need to move fields with their methods
- A field is a piece of information that conceptually belongs to another class
- After a Move Method, the moved method now references fields left in the original class

---

## 4. Refactoring steps

1. Check if the field is used by methods in its own class — if so, consider moving those methods too (Move Method)
2. Create a getter and setter for the field in its current class (if not already present)
3. Add a new field with the same name to the target class
4. Set the new field in the target class and remove it from the original (or redirect via getter/setter)
5. Update all references:
   - In the original class, replace direct access with delegation to the target class
   - Or, move the dependent methods (Move Method) so they access the field natively
6. Delete the original field (and its getter/setter if no longer needed)
7. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class Account
{
    public function __construct(
        private AccountType $type,
        private float $interestRate, // used by type-related logic
    ) {}

    public function interestForAmount(float $amount): float
    {
        return $this->interestRate * $amount;
    }
}

class AccountType
{
    // interestRate conceptually belongs here
}
```

**AFTER — expected:**
```php
class AccountType
{
    public function __construct(private float $interestRate) {}

    public function getInterestRate(): float
    {
        return $this->interestRate;
    }
}

class Account
{
    public function __construct(private AccountType $type) {}

    public function interestForAmount(float $amount): float
    {
        return $this->type->getInterestRate() * $amount; // delegates to AccountType
    }
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Moving the field but keeping a duplicate in the original class**
```php
// Not accepted — now interestRate exists in both Account and AccountType
class Account
{
    private float $interestRate; // old — should be deleted
    private AccountType $type;
}
```

**Mistake 2: Moving the field without moving the methods that use it**
```php
// Not accepted — interestForAmount still lives in Account but now must call back
// to AccountType — this introduces Feature Envy instead of fixing it
```

**Mistake 3: Moving a field used intensively by its original class**
```php
// Not accepted — if Account uses interestRate in 10 methods and AccountType
// uses it in 1, the field should stay in Account
```

---

## 7. Benefits

- **Cohesion:** Data and the behavior that uses it live in the same class
- **Encapsulation:** The owning class controls all access to the field
- **Reduces coupling:** Other classes no longer need to reach into the original class for data
