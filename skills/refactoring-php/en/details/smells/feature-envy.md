# SKILL: Detecting and Refactoring Feature Envy — PHP

## Source
Based on: https://refactoring.guru/smells/feature-envy

---

## 1. What is Feature Envy

A method accesses the data of another object more than it accesses the data of its own class. The method is "envious" of the other class — it seems to want to live there instead.

**Why this happens:**
- Fields were moved to a new class but the methods that use them were not
- Business logic was placed in a service or utility class that works on a domain object's internals
- A method was gradually extended to rely more and more on another object's getters

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A method calls several getters on another object in sequence
- [ ] A method receives an object as parameter and uses mostly its fields/methods
- [ ] A method ignores `$this` entirely and only manipulates another object's state
- [ ] The method's name naturally belongs to the other class ("calculateOrderTotal" in a ReportService)

---

## 3. Treatment techniques (in order of preference)

| Situation found                                         | Recommended technique             |
|---------------------------------------------------------|-----------------------------------|
| The whole method belongs in the other class             | Move Method                       |
| Only part of the method envies another class            | Extract Method, then Move Method  |
| The method uses data from several classes               | Move to the class with most data  |
| The method is a calculation that belongs on the model   | Move Method to domain object      |

---

## 4. Example

**BEFORE — not accepted:**
```php
class InvoicePrinter
{
    public function calculateInvoiceTotal(Invoice $invoice): float
    {
        $subtotal = 0;
        foreach ($invoice->getItems() as $item) {
            $subtotal += $item->getPrice() * $item->getQuantity();
        }
        $tax = $subtotal * $invoice->getTaxRate();
        $discount = $invoice->hasDiscount() ? $subtotal * $invoice->getDiscountRate() : 0;
        return $subtotal + $tax - $discount;
    }
}
```

**AFTER — expected:**
```php
class Invoice
{
    public function calculateTotal(): float
    {
        $subtotal = array_sum(
            array_map(fn($i) => $i->getPrice() * $i->getQuantity(), $this->items)
        );
        $tax = $subtotal * $this->taxRate;
        $discount = $this->hasDiscount() ? $subtotal * $this->discountRate : 0;
        return $subtotal + $tax - $discount;
    }
}

class InvoicePrinter
{
    public function print(Invoice $invoice): void
    {
        $total = $invoice->calculateTotal(); // delegate to owner
        // ... print logic
    }
}
```

**Why this pattern:**
- `calculateTotal` naturally belongs to `Invoice` — it only uses Invoice data
- `InvoicePrinter` is freed from knowing Invoice internals

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the envious method as a thin wrapper**
```php
// Not accepted — just a wrapper; delete the envious method entirely
public function calculateInvoiceTotal(Invoice $invoice): float
{
    return $invoice->calculateTotal();
}
```

**Mistake 2: Moving the method but keeping raw field parameters**
```php
// Not accepted — moved but still receiving raw field values
public function calculateTotal(float $subtotal, float $taxRate, float $discountRate, bool $hasDiscount): float { ... }
```

**Mistake 3: Moving a method that uses data from many classes equally**
```php
// Not accepted — if the method uses Order, Customer, and Coupon equally,
// moving it to any one class just shifts the envy; Extract Class is better
```

---

## 6. Benefits

- **Cohesion:** Each class owns the behavior that operates on its own data
- **Encapsulation:** The domain object exposes intent, not raw fields
- **Maintainability:** Business rules are colocated with the data they govern
