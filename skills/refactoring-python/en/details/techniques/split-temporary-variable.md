# TECHNIQUE: Split Temporary Variable — Python

## Source
Based on: https://refactoring.guru/split-temporary-variable

---

## 1. Problem

A local variable is assigned more than once, serving different purposes at different points in the method. Reusing the same name for different concepts makes the code hard to follow.

---

## 2. Solution

Create a separate variable for each assignment. Give each variable a name that describes its specific purpose.

---

## 3. When to apply

- A variable is assigned in one place, then reassigned with a completely different meaning later in the same method
- The variable name is generic (e.g., `temp`, `result`, `value`) because it covers multiple concepts
- The variable's dual purpose makes the method hard to read or test

---

## 4. Refactoring steps

1. Identify the variable that is assigned more than once for different purposes
2. Rename the first assignment to a name that describes the first purpose
3. Rename subsequent assignments to new variables with names describing their purposes
4. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def calculate(self, height: float, width: float) -> float:
    temp = 2 * (height + width)   # perimeter
    print(f"Perimeter: {temp}")
    temp = height * width         # area — same variable, different concept
    print(f"Area: {temp}")
    return temp
```

**AFTER — expected:**
```python
def calculate(self, height: float, width: float) -> float:
    perimeter = 2 * (height + width)
    print(f"Perimeter: {perimeter}")
    area = height * width
    print(f"Area: {area}")
    return area
```

**Why this pattern:**
- `perimeter` and `area` are distinct concepts that deserve distinct names
- Each variable is only assigned once, making the code easier to follow

---

## 6. Negative examples — what NOT to do

**Mistake 1: Splitting a loop accumulator**
```python
# Caution — a loop accumulator is intentionally reassigned; do not split it
total = 0
for item in items:
    total += item.price  # this reassignment is intentional
```

**Mistake 2: Keeping a generic suffix instead of naming by purpose**
```python
# Not accepted — temp1 and temp2 are as meaningless as temp
temp1 = 2 * (height + width)
temp2 = height * width
```

---

## 7. Benefits

- **Clarity:** Each variable name makes its single purpose obvious
- **Easier debugging:** Each step in the calculation has a distinct named value
- **Readability:** Readers do not need to mentally track what `temp` means at each point
