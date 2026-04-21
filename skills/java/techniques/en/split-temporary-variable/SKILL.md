# TECHNIQUE: Split Temporary Variable — Java

## Source
Based on: https://refactoring.guru/split-temporary-variable

---

## 1. Problem

A local variable is assigned more than once, serving different purposes at different points in the method. Reusing the same name for different concepts makes the code hard to follow and prevents each assignment from being made `final`.

---

## 2. Solution

Create a separate variable for each assignment. Give each variable a name that describes its specific purpose.

---

## 3. When to apply

- A variable is assigned in one place, then reassigned with a completely different meaning later in the same method
- The variable name is generic (e.g., `temp`, `result`, `value`) because it covers multiple concepts
- Making the variable `final` would cause a compile error because of reassignment

---

## 4. Refactoring steps

1. Identify the variable that is assigned more than once for different purposes
2. Rename the first assignment to a name that describes the first purpose
3. Rename subsequent assignments to new variables with names describing their purposes
4. Make each variable `final`
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public double calculate(double height, double width) {
    double temp = 2 * (height + width);  // perimeter
    System.out.println("Perimeter: " + temp);
    temp = height * width;               // area — same variable, different concept
    System.out.println("Area: " + temp);
    return temp;
}
```

**AFTER — expected:**
```java
public double calculate(double height, double width) {
    final double perimeter = 2 * (height + width);
    System.out.println("Perimeter: " + perimeter);
    final double area = height * width;
    System.out.println("Area: " + area);
    return area;
}
```

**Why this pattern:**
- `perimeter` and `area` are distinct concepts that deserve distinct names
- Both variables are now `final`, preventing future accidental mutation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Splitting a loop accumulator**
```java
// Caution — a loop accumulator is intentionally reassigned; do not split it
double total = 0;
for (Item item : items) {
    total += item.getPrice(); // this reassignment is intentional
}
```

**Mistake 2: Keeping a generic suffix instead of naming by purpose**
```java
// Not accepted — temp1 and temp2 are as meaningless as temp
double temp1 = 2 * (height + width);
double temp2 = height * width;
```

---

## 7. Benefits

- **Clarity:** Each variable name makes its single purpose obvious
- **Immutability:** `final` variables cannot be accidentally reused for a third purpose
- **Easier debugging:** Each step in the calculation has a distinct named value
