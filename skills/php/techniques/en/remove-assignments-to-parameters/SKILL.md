# TECHNIQUE: Remove Assignments to Parameters — PHP

## Source
Based on: https://refactoring.guru/remove-assignments-to-parameters

---

## 1. Problem

A method parameter is reassigned inside the method body. This hides the original value and makes the method harder to understand. In PHP, scalar parameters are passed by value, so reassigning a parameter does not affect the caller — but the confusion it creates is still a code smell.

---

## 2. Solution

Introduce a local variable to hold the modified value. Use that variable instead of reassigning the parameter.

---

## 3. When to apply

- A parameter is assigned a new value inside the method
- The reassignment does not need to be visible to the caller
- The original parameter value might be needed later in the method (or clarity is improved by preserving it)

---

## 4. Refactoring steps

1. Find the parameter assignment
2. Declare a new local variable with a descriptive name, initialised with the parameter's value (or the new value)
3. Replace all uses of the parameter after the assignment with the new local variable
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
public function discount(int $inputVal, int $quantity): int {
    if ($inputVal > 50) $inputVal -= 2;    // reassigning the parameter
    if ($quantity > 100) $inputVal -= 1;   // reassigning again
    return $inputVal;
}
```

**AFTER — expected:**
```php
public function discount(int $inputVal, int $quantity): int {
    $result = $inputVal;
    if ($result > 50) $result -= 2;
    if ($quantity > 100) $result -= 1;
    return $result;
}
```

**Why this pattern:**
- `$inputVal` always refers to the original value
- `$result` clearly signals "this is the value we are building toward returning"

---

## 6. Negative examples — what NOT to do

**Mistake 1: Mutating the object the parameter refers to**
```php
// This is NOT a parameter assignment — this mutates the object and IS visible to the caller
public function addItem(Order $order, Item $item): void {
    $order->addItem($item); // mutates the Order object; caller sees the change
}
```

**Mistake 2: Using a reference parameter intentionally**
```php
// This is a legitimate use of by-reference; do not apply this refactoring
public function increment(int &$counter): void {
    $counter++;
}
```

---

## 7. Benefits

- **Clarity:** The original parameter value is preserved and clearly separated from the computed result
- **Intent:** The output variable's name communicates the role of the final value
- **Safety:** Prevents confusing the "input" and the "working copy" of the same concept
