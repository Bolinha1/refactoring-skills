# TECHNIQUE: Inline Method — PHP

## Source
Based on: https://refactoring.guru/inline-method

---

## 1. Problem

A method body is as obvious as its name, or the method is only used once and the indirection adds no value. Keeping a trivial wrapper around a single expression makes the code harder to follow, not easier.

---

## 2. Solution

Replace every call to the method with the method's body. Delete the method.

---

## 3. When to apply

- The method body is a single line and the name adds nothing beyond restating the code
- The method is called in only one place and the indirection hinders readability
- A sequence of short methods was created during refactoring and the structure is now over-fragmented
- The method was a delegation shim whose delegate has been inlined or removed

---

## 4. Refactoring steps

1. Check that the method is not overridden in a subclass
2. Find all call sites
3. For each call site, replace the call with the method body (substituting parameters for arguments)
4. Delete the method
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```php
class OrderService {
    public function getDiscount(Order $order): float {
        return $this->moreThanFiftyItems($order) ? 0.1 : 0.0;
    }

    private function moreThanFiftyItems(Order $order): bool {
        return $order->getItemCount() > 50;
    }
}
```

**AFTER — expected:**
```php
class OrderService {
    public function getDiscount(Order $order): float {
        return $order->getItemCount() > 50 ? 0.1 : 0.0;
    }
}
```

**Why this pattern:**
- `moreThanFiftyItems` added no clarity — the condition is readable on its own
- One fewer method to search for, test, and maintain

---

## 6. Negative examples — what NOT to do

**Mistake 1: Inlining a method that is called in multiple places**
```php
// Not accepted — inlining duplicates the condition in every call site
if ($this->moreThanFiftyItems($order)) { ... } // called here and in three other places
```

**Mistake 2: Inlining a method that is overridden in a subclass**
```php
// Not accepted — inlining removes the polymorphic hook used by subclasses
protected function moreThanFiftyItems(Order $order): bool { ... }
// PremiumOrderService overrides this
```

**Mistake 3: Inlining a method whose name explains a non-obvious invariant**
```php
// Not accepted — the name explains domain meaning that the raw expression does not
private function isEligibleForFreeShipping(Order $order): bool {
    return $order->getTotal() > 100;
}
```

---

## 7. Benefits

- **Reduced indirection:** No need to jump to another method to understand a one-liner
- **Simpler codebase:** Removes methods that exist purely as leftover scaffolding
- **Cleaner public API:** Fewer public methods expose less surface area
