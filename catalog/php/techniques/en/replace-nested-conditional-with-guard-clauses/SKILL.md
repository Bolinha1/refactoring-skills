# TECHNIQUE: Replace Nested Conditional with Guard Clauses — PHP

## Source
Based on: https://refactoring.guru/replace-nested-conditional-with-guard-clauses

---

## 1. Problem

A method has deeply nested conditionals that obscure the main happy path. Readers must mentally track every nesting level to understand what the method normally does.

---

## 2. Solution

Replace special-case conditions with early returns (guard clauses) at the top of the method. The main logic runs unindented at the bottom.

---

## 3. When to apply

- The method has 2+ levels of nesting that encode edge cases or preconditions
- The else branch of a top-level `if` contains the main logic
- Each nested condition checks a precondition or exceptional case before the real work
- The method ends with a single return buried inside multiple else blocks

---

## 4. Refactoring steps

1. Identify each special case (null check, error condition, early exit)
2. For each special case, move its condition to the top of the method
3. Return early (or throw) inside that guard clause
4. Remove the now-unnecessary else nesting
5. Verify the main logic runs unindented
6. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```php
public function getPayAmount(Employee $employee): float
{
    if ($employee->isDead()) {
        $result = $this->deadAmount();
    } else {
        if ($employee->isSeparated()) {
            $result = $this->separatedAmount();
        } else {
            if ($employee->isRetired()) {
                $result = $this->retiredAmount();
            } else {
                $result = $this->normalPayAmount();
            }
        }
    }
    return $result;
}
```

**AFTER — expected:**
```php
public function getPayAmount(Employee $employee): float
{
    if ($employee->isDead())      return $this->deadAmount();
    if ($employee->isSeparated()) return $this->separatedAmount();
    if ($employee->isRetired())   return $this->retiredAmount();
    return $this->normalPayAmount();
}
```

**Why this pattern:**
- The exceptional cases are dispatched at the top — readers skip past them immediately
- `normalPayAmount()` stands out as the default: no nesting, no else, no variable accumulation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Using guard clauses for the main case, not the exceptions**
```php
// Not accepted — guard clauses should handle the exceptions; normal flow should be at bottom
public function getPayAmount(Employee $employee): float
{
    if (!$employee->isDead() && !$employee->isSeparated() && !$employee->isRetired()) {
        return $this->normalPayAmount(); // inverted — checking for normality is the guard
    }
    // edge cases below...
}
```

**Mistake 2: Adding a guard clause that duplicates an existing guarantee**
```php
// Not accepted — type hint already guarantees $employee is never null
if ($employee === null) return 0.0; // adds noise, not safety
if ($employee->isDead()) return $this->deadAmount();
```

**Mistake 3: Flattening nested conditions that aren't independent guard clauses**
```php
// Not accepted — these conditions are not guards; they represent business logic branches
// Use Decompose Conditional instead of guard clauses here
if ($plan === 'summer') {
    return $this->summerCharge(); // not an exception — it's a valid main path
}
```

---

## 7. Benefits

- **Readability:** The method's normal execution path is immediately visible
- **Reduced nesting:** Each guard clause removes one level of indentation
- **Intention clarity:** Early returns signal "this case is exceptional — stop here"
