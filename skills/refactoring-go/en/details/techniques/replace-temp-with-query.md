# TECHNIQUE: Replace Temp with Query — Go

## Source
Based on: https://refactoring.guru/replace-temp-with-query

---

## 1. Problem

You use a temporary variable to store the result of an expression, then reference
that variable later in the same method. The expression is re-evaluated every time
the method runs but is never surfaced as reusable logic.

---

## 2. Solution

Extract the expression into a method (or a private helper function). Replace all
references to the temporary variable with calls to that method.

---

## 3. When to apply

- The same expression appears in multiple methods and you want to avoid duplication
- The temporary variable obscures what the value represents
- The method is long and splitting out sub-computations improves readability
- The expression has a name-worthy meaning in the domain

---

## 4. Refactoring steps

1. Ensure the expression is free of side effects (pure computation)
2. Extract the expression into a new method with a descriptive name
3. Replace every reference to the temp variable with a call to the new method
4. Remove the temp variable declaration
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func (o *Order) PriceWithDiscount() float64 {
	basePrice := float64(o.Quantity) * o.ItemPrice
	var discountFactor float64
	if basePrice > 1000 {
		discountFactor = 0.95
	} else {
		discountFactor = 0.98
	}
	return basePrice * discountFactor
}
```

**AFTER — expected:**
```go
func (o *Order) PriceWithDiscount() float64 {
	return o.basePrice() * o.discountFactor()
}

func (o *Order) basePrice() float64 {
	return float64(o.Quantity) * o.ItemPrice
}

func (o *Order) discountFactor() float64 {
	if o.basePrice() > 1000 {
		return 0.95
	}
	return 0.98
}
```

**Why this pattern:**
- `basePrice()` and `discountFactor()` can be reused in other methods on `Order`
- The main method reads as a formula, not an imperative sequence
- Each helper can be tested independently

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting a method with side effects**
```go
// Not accepted — calling this twice produces different results
func (o *Order) nextID() int {
	o.counter++
	return o.counter
}
```

**Mistake 2: Extracting a one-word expression that adds no clarity**
```go
// Not accepted — the variable already had a clear name; wrapping it gains nothing
func (o *Order) qty() int { return o.Quantity }
```

---

## 7. Benefits

- **Reusability:** The extracted method is available to the whole struct, not just one method
- **Readability:** The calling method describes what it computes, not how
- **Testability:** Small, pure methods are easy to unit-test in isolation
