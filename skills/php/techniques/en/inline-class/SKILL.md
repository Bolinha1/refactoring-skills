# TECHNIQUE: Inline Class — PHP

## Source
Based on: https://refactoring.guru/inline-class

---

## 1. Problem

A class no longer does enough to justify its existence. It has too few responsibilities and is just an unnecessary indirection.

---

## 2. Solution

Move all features of the class into another class, then delete it.

---

## 3. When to apply

- After a refactoring, one class was left with only one or two trivial methods
- A class is used in only one place and adds no real abstraction
- A class that was split too aggressively needs to be consolidated
- The class is a thin wrapper that just delegates everything to another class

---

## 4. Refactoring steps

1. Identify the class to inline and the target class (where features will move)
2. For each public feature in the class to inline:
   - Copy it into the target class
   - Update the target class to delegate to the inline class (safe transition)
3. Update all callers to use the target class directly
4. Remove the delegation from the target class (methods now work natively)
5. Delete the inlined class
6. Compile and run tests after each step

---

## 5. Example

**BEFORE — not accepted:**
```php
class TelephoneNumber
{
    public function __construct(private string $number) {}

    public function getNumber(): string
    {
        return $this->number;
    }
}

class Person
{
    public function __construct(
        private string $name,
        private TelephoneNumber $telephone,
    ) {}

    public function getTelephoneNumber(): string
    {
        return $this->telephone->getNumber(); // TelephoneNumber is just a wrapper
    }
}
```

**AFTER — expected:**
```php
class Person
{
    public function __construct(
        private string $name,
        private string $telephoneNumber, // inlined directly
    ) {}

    public function getTelephoneNumber(): string
    {
        return $this->telephoneNumber;
    }
}
```

**Use case: consolidating Shotgun Surgery**
```php
// BEFORE — four tiny service classes split too aggressively
class OrderFinder   { public function find(int $id): Order { ... } }
class OrderSaver    { public function save(Order $o): void { ... } }
class OrderUpdater  { public function update(Order $o): void { ... } }
class OrderDeleter  { public function delete(int $id): void { ... } }

// AFTER — inline all into a single coherent repository
class OrderRepository
{
    public function find(int $id): Order { ... }
    public function save(Order $o): void { ... }
    public function update(Order $o): void { ... }
    public function delete(int $id): void { ... }
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a class that still has a meaningful concept**
```php
// Not accepted — Money has behavior (arithmetic, formatting) that belongs together
// Do not inline Money into Order just because it has few fields
```

**Mistake 2: Inlining without removing the original class**
```php
// Not accepted — both Person.$telephoneNumber and TelephoneNumber exist simultaneously
class Person
{
    private string $telephoneNumber;     // inlined copy
    private TelephoneNumber $telephone;  // still here — creates confusion
}
```

**Mistake 3: Inlining a class used in many places**
```php
// Not accepted — if TelephoneNumber is used by Person, Employee, and Contact,
// inlining would require duplicating the logic in three places
```

---

## 7. Benefits

- **Simplicity:** Removes unnecessary indirection from the codebase
- **Readability:** Fewer classes to navigate when the abstraction adds no value
- **Preparation:** Often precedes Extract Class — inline first, then re-extract correctly
