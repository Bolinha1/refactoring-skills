# TECHNIQUE: Split Temporary Variable — Go

## Source
Based on: https://refactoring.guru/split-temporary-variable

---

## 1. Problem

A temporary variable is assigned more than once within the same method, but each
assignment serves a different purpose. Reusing the same name for unrelated values
makes the code harder to follow and prevents the compiler from catching type mismatches.

---

## 2. Solution

Create a separate variable for each distinct use. Name each variable after the value
it holds at that point.

---

## 3. When to apply

- A variable is assigned in two or more places with unrelated meanings
- Reading the variable mid-method requires remembering which assignment was last
- Loop variables are the exception — they are intentionally reassigned

---

## 4. Refactoring steps

1. Identify all the distinct purposes the variable serves
2. Rename the first use to reflect that specific purpose
3. Declare a new variable for the second use with its own descriptive name
4. Repeat for any further reuses
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func geometry(height, width float64) {
	temp := 2 * (height + width)
	fmt.Println("Perimeter:", temp)

	temp = height * width   // reused for a completely different value
	fmt.Println("Area:", temp)
}
```

**AFTER — expected:**
```go
func geometry(height, width float64) {
	perimeter := 2 * (height + width)
	fmt.Println("Perimeter:", perimeter)

	area := height * width
	fmt.Println("Area:", area)
}
```

**Why this pattern:**
- `perimeter` and `area` describe what each value represents
- No reader needs to track which assignment is "current"
- The compiler can catch accidental reuse across unrelated scopes

---

## 6. Negative examples — what NOT to do

**Mistake 1: Renaming without separating — still one variable**
```go
// Not accepted — still the same variable, just the comment changed
temp := 2 * (height + width) // perimeter
fmt.Println(temp)
temp = height * width // area — overwriting the same var
```

**Mistake 2: Splitting loop counters**
```go
// Not accepted — loop variables are intentionally reused
for i := 0; i < n; i++ { ... }
```

---

## 7. Benefits

- **Clarity:** Each variable has one meaning throughout its lifetime
- **Safety:** The compiler enforces that each variable is only used for its declared type
- **Refactoring ease:** Variables with single purposes are simpler to extract or inline later
