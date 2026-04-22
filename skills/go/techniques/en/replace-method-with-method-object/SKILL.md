# TECHNIQUE: Replace Method with Method Object — Go

## Source
Based on: https://refactoring.guru/replace-method-with-method-object

---

## 1. Problem

A method is so large and complex that extracting sub-functions is awkward because they would all share many local variables. The variables are too intertwined to pass as parameters without creating unwieldy signatures.

---

## 2. Solution

Extract the method into a new struct. Each local variable becomes a field of that struct. The method body becomes a `Compute()` method, which can now be freely split into sub-methods without any parameter passing, since shared state lives in the struct's fields.

---

## 3. When to apply

- The method has many local variables used across multiple logical phases
- Extract Method would require passing too many parameters between the new functions
- The algorithm is complex enough to deserve its own abstraction
- You want to test the algorithm in isolation

---

## 4. Refactoring steps

1. Create a new struct named after the method (e.g., `PriceCalculator` for `Price()`)
2. Add a field for the struct that originally contained the method
3. Add a field for each local variable and each parameter of the original method
4. Create a constructor `NewXxx` that takes the source struct and all parameters
5. Copy the original method body into a `Compute()` method; local variable references become field references
6. Replace the original method body with `return NewXxx(o, params...).Compute()`
7. Apply Extract Method freely on `Compute()` — no parameters needed since state is in fields
8. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
type Order struct {
    quantity    int
    primaryRate float64
    isPremium   bool
}

func (o *Order) Price() float64 {
    primaryBasePrice := float64(o.quantity) * o.primaryRate
    secondaryBasePrice := primaryBasePrice * 0.7
    tertiaryBasePrice := math.Max(primaryBasePrice, secondaryBasePrice)
    // complex discount logic using all three variables
    if o.isPremium {
        return tertiaryBasePrice - primaryBasePrice*0.1
    }
    return tertiaryBasePrice - secondaryBasePrice*0.05
}
```

**AFTER — expected:**
```go
type PriceCalculator struct {
    order              *Order
    primaryBasePrice   float64
    secondaryBasePrice float64
    tertiaryBasePrice  float64
}

func NewPriceCalculator(o *Order) *PriceCalculator {
    return &PriceCalculator{order: o}
}

func (pc *PriceCalculator) Compute() float64 {
    pc.primaryBasePrice = float64(pc.order.quantity) * pc.order.primaryRate
    pc.secondaryBasePrice = pc.primaryBasePrice * 0.7
    pc.tertiaryBasePrice = math.Max(pc.primaryBasePrice, pc.secondaryBasePrice)
    return pc.tertiaryBasePrice - pc.discount()
}

func (pc *PriceCalculator) discount() float64 {
    if pc.order.isPremium {
        return pc.primaryBasePrice * 0.1
    }
    return pc.secondaryBasePrice * 0.05
}

// Original method now delegates
func (o *Order) Price() float64 {
    return NewPriceCalculator(o).Compute()
}
```

**Why this pattern:**
- `discount()` can read all three price fields without any parameters
- `PriceCalculator` can be instantiated and tested independently of `Order`

---

## 6. Negative examples — what NOT to do

**Mistake 1: Applying this when Extract Method with a few parameters would suffice**
```go
// Not accepted — if the method only has 2-3 local variables,
// Extract Method with explicit parameters is simpler and clearer
```

**Mistake 2: Naming the struct vaguely**
```go
// Not accepted — too generic, does not communicate what it calculates
type OrderHelper struct { ... }
type OrderProcessor struct { ... }

// Preferred — named after the specific algorithm
type OrderPriceCalculator struct { ... }
```

---

## 7. Benefits

- **Enables Extract Method:** Sub-methods share state via fields without parameter passing
- **Testability:** The calculation struct can be instantiated and tested independently
- **Readability:** Each phase of the algorithm is a clearly named method
- **Extensibility:** The struct can be embedded or composed to vary parts of the algorithm
