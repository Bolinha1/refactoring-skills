# TECHNIQUE: Remove Assignments to Parameters — Java

## Source
Based on: https://refactoring.guru/remove-assignments-to-parameters

---

## 1. Problem

A method parameter is reassigned inside the method body. This is confusing because it hides the original value, makes the method harder to understand, and can mislead callers who expect pass-by-value behaviour (objects are passed by reference in Java, so assigning a new object to the parameter does NOT affect the caller's variable, but mutating the object's state does).

---

## 2. Solution

Introduce a local variable to hold the modified value. Use that variable instead of reassigning the parameter.

---

## 3. When to apply

- A parameter is assigned a new value inside the method
- The reassignment does not need to be visible to the caller (pass-by-value semantics)
- The original parameter value is needed later in the method (or should be preserved for readability)

---

## 4. Refactoring steps

1. Find the parameter assignment
2. Declare a new local variable with a descriptive name, initialised with the parameter's value (or the new value)
3. Replace all uses of the parameter after the assignment with the new local variable
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public int discount(int inputVal, int quantity) {
    if (inputVal > 50) inputVal -= 2;    // reassigning the parameter
    if (quantity > 100) inputVal -= 1;   // reassigning again
    return inputVal;
}
```

**AFTER — expected:**
```java
public int discount(int inputVal, int quantity) {
    int result = inputVal;
    if (result > 50) result -= 2;
    if (quantity > 100) result -= 1;
    return result;
}
```

**Why this pattern:**
- `inputVal` always refers to the original value — the method cannot accidentally pass a modified value back
- `result` clearly signals "this is the value we are building toward returning"

---

## 6. Negative examples — what NOT to do

**Mistake 1: Mutating the state of the object the parameter refers to**
```java
// This is NOT a parameter assignment — this mutates the referenced object and IS visible to the caller
public void addItem(Order order, Item item) {
    order.addItem(item); // mutates the Order object; caller sees the change
}
```

**Mistake 2: Keeping the parameter reassignment and adding a comment**
```java
// Not accepted — the comment does not fix the readability problem
if (inputVal > 50) inputVal -= 2; // intentionally modifying the input
```

---

## 7. Benefits

- **Clarity:** The original parameter value is preserved and clearly separated from the computed result
- **Intent:** The output variable's name communicates the role of the final value
- **Safety:** Prevents confusing the "input" and the "working copy" of the same concept
