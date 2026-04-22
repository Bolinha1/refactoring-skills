# SMELL: Message Chains — Python

## Source
Based on: https://refactoring.guru/smells/message-chains

---

## 1. What is it?

A client asks an object for another object, which asks yet another object, forming a chain: `a.get_b().get_c().get_d().do_something()`. The client is coupled to the entire chain of intermediaries. If any step changes its structure, the client breaks.

---

## 2. Warning signs

- [ ] Method calls are chained three or more levels deep: `order.get_customer().get_address().get_city()`
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
```python
# Client navigates the full object graph just to get a city name
class InvoiceService:
    def get_delivery_city(self, order) -> str:
        return order.get_customer().get_address().get_city()

    def is_local_delivery(self, order) -> bool:
        return order.get_customer().get_address().get_city() == "São Paulo"
```

**AFTER — expected:**
```python
# Add a shortcut on Order (Hide Delegate)
class Order:
    def get_delivery_city(self) -> str:
        return self._customer.get_address().get_city()


# Client no longer knows about Customer or Address
class InvoiceService:
    def get_delivery_city(self, order) -> str:
        return order.get_delivery_city()

    def is_local_delivery(self, order) -> bool:
        return order.get_delivery_city() == "São Paulo"
```

**Why this pattern:**
- `InvoiceService` is decoupled from `Customer` and `Address`
- Renaming `Address.get_city()` only requires one change instead of many

---

## 5. Negative examples — what NOT to do

**Mistake 1: Assigning the chain to a local variable without breaking the dependency**
```python
# Not accepted — the coupling is still there; only the syntax changed
address = order.get_customer().get_address()
city = address.get_city()
```

**Mistake 2: Adding a method that returns the intermediate object**
```python
# Not accepted — now callers chain off the returned object; same problem, different entry point
def get_customer(self): return self._customer
```

---

## 6. Benefits

- **Lower coupling:** Clients depend only on their direct collaborator
- **Easier refactoring:** Internal navigation can be changed without updating every call site
- **More readable:** `order.get_delivery_city()` is clearer than `order.get_customer().get_address().get_city()`
