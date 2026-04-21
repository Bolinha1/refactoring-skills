# SMELL: Message Chains — PHP

## Source
Based on: https://refactoring.guru/smells/message-chains

---

## 1. What is it?

A client asks an object for another object, which asks yet another object, forming a chain: `$a->getB()->getC()->getD()->doSomething()`. The client is coupled to the entire chain of intermediaries. If any step changes its structure, the client breaks.

---

## 2. Warning signs

- [ ] Method calls are chained three or more levels deep: `$order->getCustomer()->getAddress()->getCity()`
- [ ] The same chain appears in multiple places in the codebase
- [ ] You have to navigate a chain of objects just to retrieve a single value
- [ ] A chain appears in a loop or conditional, amplifying coupling
- [ ] Adding a layer between two objects in the chain requires updating every call site

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Hide Delegate** | Add a shortcut method to the first object in the chain so clients don't need to navigate the intermediaries |
| **Extract Method** | Extract the chain into a named method that hides the navigation |

---

## 4. Example

**BEFORE — not accepted:**
```php
// Client navigates the full object graph just to get a city name
class InvoiceService {
    public function getDeliveryCity(Order $order): string {
        return $order->getCustomer()->getAddress()->getCity();
    }

    public function isLocalDelivery(Order $order): bool {
        return $order->getCustomer()->getAddress()->getCity() === 'São Paulo';
    }
}
```

**AFTER — expected:**
```php
// Add a shortcut on Order (Hide Delegate)
class Order {
    public function getDeliveryCity(): string {
        return $this->customer->getAddress()->getCity();
    }
}

// Client no longer knows about Customer or Address
class InvoiceService {
    public function getDeliveryCity(Order $order): string {
        return $order->getDeliveryCity();
    }

    public function isLocalDelivery(Order $order): bool {
        return $order->getDeliveryCity() === 'São Paulo';
    }
}
```

**Why this pattern:**
- `InvoiceService` is decoupled from `Customer` and `Address`
- Renaming `Address::getCity()` only requires one change instead of many

---

## 5. Negative examples — what NOT to do

**Mistake 1: Assigning the chain to a local variable without breaking the dependency**
```php
// Not accepted — the coupling is still there; only the syntax changed
$address = $order->getCustomer()->getAddress();
$city = $address->getCity();
```

**Mistake 2: Adding a method that returns the intermediate object**
```php
// Not accepted — now callers chain off the returned object; same problem, different entry point
public function getCustomer(): Customer { return $this->customer; }
```

---

## 6. Benefits

- **Lower coupling:** Clients depend only on their direct collaborator
- **Easier refactoring:** Internal navigation can be changed without updating every call site
- **More readable:** `$order->getDeliveryCity()` is clearer than `$order->getCustomer()->getAddress()->getCity()`
