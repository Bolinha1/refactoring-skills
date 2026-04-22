# TECHNIQUE: Substitute Algorithm — PHP

## Source
Based on: https://refactoring.guru/substitute-algorithm

---

## 1. Problem

You want to replace an existing algorithm with a cleaner, simpler, or more efficient one. The algorithm works correctly but is hard to understand, hard to extend, or can now be replaced with a built-in PHP function.

---

## 2. Solution

Replace the body of the method implementing the old algorithm with the new algorithm. Keep the method signature the same so callers are unaffected.

---

## 3. When to apply

- A simpler algorithm exists that produces the same results
- A PHP built-in function (e.g., `array_filter`, `in_array`, `array_map`) now covers what you implemented manually
- The current algorithm is difficult to extend incrementally — it is easier to rewrite than to modify
- The algorithm has already been simplified as much as possible but is still unclear

---

## 4. Refactoring steps

1. Ensure that the existing algorithm has thorough tests — you need them to verify the replacement
2. Write the new algorithm in a separate method (or on a branch)
3. Run the tests against the new algorithm; fix any failures
4. Once tests pass, replace the old algorithm body with the new one
5. Run the tests one final time
6. Delete any helper code that was only needed by the old algorithm

---

## 5. Example

**BEFORE — not accepted:**
```php
public function findPerson(array $people): string {
    foreach ($people as $person) {
        if ($person === 'Don') return 'Don';
        if ($person === 'John') return 'John';
        if ($person === 'Kent') return 'Kent';
    }
    return '';
}
```

**AFTER — expected:**
```php
public function findPerson(array $people): string {
    $candidates = ['Don', 'John', 'Kent'];
    foreach ($people as $person) {
        if (in_array($person, $candidates, true)) {
            return $person;
        }
    }
    return '';
}
```

**Why this pattern:**
- The new algorithm is easier to extend: to add a new candidate, add one entry to the array
- `in_array` with strict comparison is more readable than chained `===` checks

---

## 6. Negative examples — what NOT to do

**Mistake 1: Substituting without a test suite**
```php
// Not accepted — if you have no tests, you cannot verify the new algorithm is equivalent;
// write the tests first before replacing anything
```

**Mistake 2: Incrementally patching the old algorithm instead of replacing it**
```php
// Not accepted — if the algorithm is fundamentally flawed, patches make it worse;
// substitute it entirely
```

---

## 7. Benefits

- **Simplicity:** The new algorithm is easier to read and maintain
- **Extensibility:** Well-structured algorithms are easier to modify
- **Leverages built-ins:** Uses PHP's native functions instead of reinventing them
