# TECHNIQUE: Extract Method — Java

## Source
Based on: https://refactoring.guru/extract-method

---

## 1. Problem

You have a code fragment that can be grouped into a meaningful unit,
but it is embedded in a larger method alongside other responsibilities.

---

## 2. Solution

Move that fragment to a new private method with a name that describes its intent.
Replace the original fragment with a call to the new method.

---

## 3. When to apply

- The code block would deserve an explanatory comment to be understood
- The same block of logic appears in more than one place
- The current method clearly does more than one distinct thing
- It is difficult to give a short, precise name to the current method

---

## 4. Refactoring steps

1. Create a new method with a name that expresses the intent of the fragment
2. Copy the code fragment to the new method
3. Identify local variables used by the fragment:
   - Used only inside the fragment → become local variables of the new method
   - Declared before and read inside → become parameters
   - Modified inside and used after → the new method must return them
4. Replace the original fragment with a call to the new method
5. Compile and run the tests

---

## 5. Example

**BEFORE — not accepted:**
```java
public void printInvoice(Invoice invoice) {
    // print header
    System.out.println("***********************");
    System.out.println("***   INVOICE #" + invoice.getNumber() + "   ***");
    System.out.println("***********************");

    // print items
    for (InvoiceItem item : invoice.getItems()) {
        System.out.println(item.getName() + "\t" + item.getQuantity() + "\t" + item.getPrice());
    }

    // print total
    double total = invoice.getItems().stream()
        .mapToDouble(i -> i.getQuantity() * i.getPrice())
        .sum();
    System.out.println("TOTAL: $ " + total);
}
```

**AFTER — expected:**
```java
public void printInvoice(Invoice invoice) {
    printHeader(invoice);
    printItems(invoice);
    printTotal(invoice);
}

private void printHeader(Invoice invoice) {
    System.out.println("***********************");
    System.out.println("***   INVOICE #" + invoice.getNumber() + "   ***");
    System.out.println("***********************");
}

private void printItems(Invoice invoice) {
    for (InvoiceItem item : invoice.getItems()) {
        System.out.println(item.getName() + "\t" + item.getQuantity() + "\t" + item.getPrice());
    }
}

private void printTotal(Invoice invoice) {
    double total = invoice.getItems().stream()
        .mapToDouble(i -> i.getQuantity() * i.getPrice())
        .sum();
    System.out.println("TOTAL: $ " + total);
}
```

**Variant — method with return value:**
```java
// BEFORE — temporary variable modified and used later
public double calculateDiscount(Order order) {
    double subtotal = 0;
    for (Item item : order.getItems()) {
        subtotal += item.getPrice() * item.getQuantity();
    }
    // ... other things ...
    return subtotal * order.getDiscountRate();
}

// AFTER — extraction with return
public double calculateDiscount(Order order) {
    double subtotal = calculateSubtotal(order);
    return subtotal * order.getDiscountRate();
}

private double calculateSubtotal(Order order) {
    return order.getItems().stream()
        .mapToDouble(i -> i.getPrice() * i.getQuantity())
        .sum();
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Vague name that does not express intent**
```java
// Not accepted
private void processPart1(Invoice invoice) { ... }
private void helper(Invoice invoice) { ... }
```

**Mistake 2: Extracting a very small fragment without readability gain**
```java
// Not accepted — one line does not need to become a method
private void printNewLine() {
    System.out.println();
}
```

**Mistake 3: Leaving the original method still large after extraction**
```java
// Not accepted — extracted one block but the method still has 50 lines
public void printInvoice(Invoice invoice) {
    printHeader(invoice);
    // ... 40 remaining lines without extraction ...
}
```

---

## 7. Benefits

- **Readability:** The main method becomes a high-level narrative
- **Reuse:** The extracted method can be reused in other contexts
- **Testability:** Smaller methods are easier to test in isolation
- **Error isolation:** Changes only affect the method that contains them
