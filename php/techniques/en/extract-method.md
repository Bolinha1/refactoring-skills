# TECHNIQUE: Extract Method — PHP

## Source
Based on: https://refactoring.guru/extract-method

---

## 1. Problem

You have a code fragment that can be grouped into a meaningful unit,
but it is embedded in a larger method alongside other responsibilities.

---

## 2. Solution

Move that fragment to a new `private` method with a name that describes its intent.
Replace the original fragment with a call to the new method.

---

## 3. When to apply

- The code block would deserve an explanatory comment to be understood
- The same block of logic appears in more than one place
- The current method clearly does more than one distinct thing
- It is difficult to give a short, precise name to the current method

---

## 4. Refactoring steps

1. Create a new `private` method with a name that expresses the intent of the fragment
2. Copy the code fragment to the new method
3. Identify local variables used by the fragment:
   - Used only inside the fragment → become local variables of the new method
   - Declared before and read inside → become parameters
   - Modified inside and used after → the new method must return them
4. Replace the original fragment with a call to the new method
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
public function printInvoice(Invoice $invoice): void
{
    // print header
    echo "***********************\n";
    echo "***   INVOICE #{$invoice->getNumber()}   ***\n";
    echo "***********************\n";

    // print items
    foreach ($invoice->getItems() as $item) {
        echo "{$item->getName()}\t{$item->getQuantity()}\t{$item->getPrice()}\n";
    }

    // print total
    $total = array_reduce($invoice->getItems(), function (float $carry, InvoiceItem $item) {
        return $carry + $item->getQuantity() * $item->getPrice();
    }, 0.0);
    echo "TOTAL: $ {$total}\n";
}
```

**AFTER — expected:**
```php
public function printInvoice(Invoice $invoice): void
{
    $this->printHeader($invoice);
    $this->printItems($invoice);
    $this->printTotal($invoice);
}

private function printHeader(Invoice $invoice): void
{
    echo "***********************\n";
    echo "***   INVOICE #{$invoice->getNumber()}   ***\n";
    echo "***********************\n";
}

private function printItems(Invoice $invoice): void
{
    foreach ($invoice->getItems() as $item) {
        echo "{$item->getName()}\t{$item->getQuantity()}\t{$item->getPrice()}\n";
    }
}

private function printTotal(Invoice $invoice): void
{
    $total = array_reduce($invoice->getItems(), function (float $carry, InvoiceItem $item) {
        return $carry + $item->getQuantity() * $item->getPrice();
    }, 0.0);
    echo "TOTAL: $ {$total}\n";
}
```

**Variant — method with return value:**
```php
// BEFORE — temporary variable calculated and used later
public function calculateDiscount(Order $order): float
{
    $subtotal = 0.0;
    foreach ($order->getItems() as $item) {
        $subtotal += $item->getPrice() * $item->getQuantity();
    }
    // ... other things ...
    return $subtotal * $order->getDiscountRate();
}

// AFTER — extraction with return
public function calculateDiscount(Order $order): float
{
    $subtotal = $this->calculateSubtotal($order);
    return $subtotal * $order->getDiscountRate();
}

private function calculateSubtotal(Order $order): float
{
    return array_reduce($order->getItems(), function (float $carry, Item $item) {
        return $carry + $item->getPrice() * $item->getQuantity();
    }, 0.0);
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Vague name that does not express intent**
```php
// Not accepted
private function processPart1(Invoice $invoice): void { ... }
private function helper(Invoice $invoice): void { ... }
```

**Mistake 2: Extracting a very small fragment without readability gain**
```php
// Not accepted — one line does not need to become a method
private function printNewLine(): void
{
    echo "\n";
}
```

**Mistake 3: Leaving the original method still large after extraction**
```php
// Not accepted — extracted one block but the method still has 50 lines
public function printInvoice(Invoice $invoice): void
{
    $this->printHeader($invoice);
    // ... 40 remaining lines without extraction ...
}
```

---

## 7. Benefits

- **Readability:** The main method becomes a high-level narrative
- **Reuse:** The extracted method can be reused in other contexts
- **Testability:** Smaller methods are easier to test in isolation
- **Error isolation:** Changes only affect the method that contains them
