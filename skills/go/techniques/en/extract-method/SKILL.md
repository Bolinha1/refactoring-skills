# TECHNIQUE: Extract Method — Go

## Source
Based on: https://refactoring.guru/extract-method

---

## 1. Problem

A function or method contains a fragment that can be grouped into a meaningful unit, but it is embedded alongside other responsibilities. The long body makes the intent hard to follow at a glance.

---

## 2. Solution

Move the fragment to a new function or method with a name that describes its intent. Replace the original fragment with a call to the new function.

---

## 3. When to apply

- The code block would deserve an explanatory comment to be understood
- The same block of logic appears in more than one place
- The current function clearly does more than one distinct thing
- It is difficult to give a short, precise name to the current function

---

## 4. Refactoring steps

1. Create a new function (or method on the same receiver) with a name that expresses the fragment's intent
2. Copy the code fragment to the new function
3. Identify variables used by the fragment:
   - Used only inside → become local variables of the new function
   - Declared before and read inside → become parameters
   - Modified inside and used after → the new function must return them
4. Replace the original fragment with a call to the new function
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```go
func printInvoice(inv *Invoice) {
    // print header
    fmt.Println("***********************")
    fmt.Printf("***   INVOICE #%s   ***\n", inv.Number)
    fmt.Println("***********************")

    // print items
    for _, item := range inv.Items {
        fmt.Printf("%s\t%d\t%.2f\n", item.Name, item.Quantity, item.Price)
    }

    // print total
    total := 0.0
    for _, item := range inv.Items {
        total += float64(item.Quantity) * item.Price
    }
    fmt.Printf("TOTAL: $%.2f\n", total)
}
```

**AFTER — expected:**
```go
func printInvoice(inv *Invoice) {
    printHeader(inv)
    printItems(inv)
    printTotal(inv)
}

func printHeader(inv *Invoice) {
    fmt.Println("***********************")
    fmt.Printf("***   INVOICE #%s   ***\n", inv.Number)
    fmt.Println("***********************")
}

func printItems(inv *Invoice) {
    for _, item := range inv.Items {
        fmt.Printf("%s\t%d\t%.2f\n", item.Name, item.Quantity, item.Price)
    }
}

func printTotal(inv *Invoice) {
    total := 0.0
    for _, item := range inv.Items {
        total += float64(item.Quantity) * item.Price
    }
    fmt.Printf("TOTAL: $%.2f\n", total)
}
```

**Variant — extraction with return value:**
```go
// BEFORE — subtotal computed inline
func calculateDiscount(order *Order) float64 {
    subtotal := 0.0
    for _, item := range order.Items {
        subtotal += item.Price * float64(item.Quantity)
    }
    return subtotal * order.DiscountRate
}

// AFTER — subtotal extracted
func calculateDiscount(order *Order) float64 {
    return subtotal(order) * order.DiscountRate
}

func subtotal(order *Order) float64 {
    total := 0.0
    for _, item := range order.Items {
        total += item.Price * float64(item.Quantity)
    }
    return total
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Vague name that does not express intent**
```go
// Not accepted
func processPart1(inv *Invoice) { ... }
func helper(inv *Invoice) { ... }
```

**Mistake 2: Extracting a single trivial line with no readability gain**
```go
// Not accepted — one fmt.Println does not need its own function
func printNewLine() {
    fmt.Println()
}
```

**Mistake 3: Leaving the original function still large after extraction**
```go
// Not accepted — extracted one block but the function still has 50 lines
func printInvoice(inv *Invoice) {
    printHeader(inv)
    // ... 40 remaining lines without extraction ...
}
```

---

## 7. Benefits

- **Readability:** The main function becomes a high-level narrative
- **Reuse:** The extracted function can be called from other contexts
- **Testability:** Smaller functions are easier to unit-test in isolation
- **Error isolation:** Changes affect only the function that contains them
