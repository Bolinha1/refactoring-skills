# TECHNIQUE: Extract Variable — Go

## Source
Based on: https://refactoring.guru/extract-variable

---

## 1. Problem

An expression is complex or hard to understand at a glance. Readers must mentally evaluate it before they can grasp what the surrounding code is doing.

---

## 2. Solution

Extract the expression into a named variable using `:=`. Choose a name that communicates the meaning of the result, not the mechanics of the calculation.

---

## 3. When to apply

- A boolean condition requires a comment or mental effort to decode
- The same sub-expression appears more than once in the same function
- A complex arithmetic expression is embedded inside a larger expression
- The expression's result has a meaningful business name

---

## 4. Refactoring steps

1. Identify the expression to extract
2. Declare a new variable with `:=` and assign the expression to it
3. Replace the original expression (and any duplicates in the same scope) with the variable
4. Choose a name that explains the intent, not the mechanics
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func (o *Order) price() float64 {
    return o.quantity*o.itemPrice -
        math.Max(0, float64(o.quantity-500))*o.itemPrice*0.05 +
        math.Min(float64(o.quantity)*o.itemPrice*0.1, 100.0)
}
```

**AFTER — expected:**
```go
func (o *Order) price() float64 {
    basePrice := float64(o.quantity) * o.itemPrice
    quantityDiscount := math.Max(0, float64(o.quantity-500)) * o.itemPrice * 0.05
    shipping := math.Min(basePrice*0.1, 100.0)
    return basePrice - quantityDiscount + shipping
}
```

**Why this pattern:**
- Each variable name captures a business concept: base price, discount, shipping
- The final return reads like a formula rather than a raw expression

---

## 6. Negative examples — what NOT to do

**Mistake 1: Using a name that only restates the expression**
```go
// Not accepted — "quantityTimesItemPrice" says what it is, not what it means
quantityTimesItemPrice := float64(o.quantity) * o.itemPrice
```

**Mistake 2: Extracting a trivial, already-readable expression**
```go
// Not accepted — adds noise without clarity benefit
isTrue := x == x
```

**Mistake 3: Introducing a variable and then using it only once in a trivial return**
```go
// Not accepted — just return the expression directly
result := x + 1
return result
```

---

## 7. Benefits

- **Readability:** Complex expressions become self-documenting
- **Debuggability:** Named variables are easy to inspect in a debugger
- **Removes duplication:** A repeated sub-expression is computed and named once
- **Prepares for Extract Method:** Named variables make further refactoring easier
