# TECHNIQUE: Remove Middle Man — PHP

## Source
Based on: https://refactoring.guru/remove-middle-man

---

## 1. Problem

A class has too many simple delegation methods that do nothing except forward calls to another object. The class is a Middle Man — it exists only to pass messages through.

---

## 2. Solution

Remove the delegation methods. Let the client access the delegate directly.

---

## 3. When to apply

- The server class has grown a large number of one-liner delegating methods
- The server class delegates so much that it has no real behavior of its own
- Adding a new feature to the delegate requires adding a new pass-through in the server
- The Middle Man smell has been identified in the server class

**Note:** This is the inverse of Hide Delegate. Apply Hide Delegate when you want encapsulation; apply Remove Middle Man when the encapsulation has become an obstacle.

---

## 4. Refactoring steps

1. Identify all delegation methods in the middle man class
2. For each delegation method, find its callers
3. Update each caller to call the delegate directly
4. Remove each delegation method from the middle man
5. If the middle man now has no remaining behavior, delete it or merge it into the caller
6. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```php
// Person has become a pure middle man — it just forwards to Department
class Person
{
    public function __construct(private Department $department) {}

    public function getManager(): Person          { return $this->department->getManager(); }
    public function getEmployees(): array         { return $this->department->getEmployees(); }
    public function getDepartmentName(): string   { return $this->department->getName(); }
    public function getDepartmentBudget(): int    { return $this->department->getBudget(); }
}
```

**AFTER — expected:**
```php
class Person
{
    public function __construct(private Department $department) {}

    public function getDepartment(): Department
    {
        return $this->department; // expose the delegate directly
    }
    // All pass-through methods removed
}

// Callers access Department directly where they need it
$dept = $person->getDepartment();
$manager = $dept->getManager();
$team = $dept->getEmployees();
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Removing the middle man but exposing internal objects that should be hidden**
```php
// Not accepted — if Department is an internal detail, exposing it breaks encapsulation
// Only remove the middle man if direct access is appropriate for the architecture
```

**Mistake 2: Removing delegation methods that have real logic in them**
```php
// Not accepted — this is not pure delegation; it has a guard
public function getManager(): ?Person
{
    if ($this->department === null) {  // real logic — don't remove
        return null;
    }
    return $this->department->getManager();
}
```

**Mistake 3: Applying Remove Middle Man when Hide Delegate is actually correct**
```php
// Not accepted — if clients should not know about Department at all,
// Hide Delegate is the right choice; Remove Middle Man goes in the other direction
```

---

## 7. Benefits

- **Simplicity:** Removes unnecessary indirection
- **Transparency:** Callers have a direct path to what they need
- **Maintainability:** Adding features to the delegate no longer requires touching the middle man
