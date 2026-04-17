# TECHNIQUE: Extract Method — Python

## Source
Based on: https://refactoring.guru/extract-method

---

## 1. Problem

You have a code fragment that can be grouped into a meaningful unit,
but it is embedded in a larger method alongside other responsibilities.

---

## 2. Solution

Move that fragment to a new method (private, prefixed with `_`) with a name that describes
its intent. Replace the original fragment with a call to the new method.

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
5. Run the tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def print_invoice(self, invoice):
    # print header
    print("***********************")
    print(f"***   INVOICE #{invoice.number}   ***")
    print("***********************")

    # print items
    for item in invoice.items:
        print(f"{item.name}\t{item.quantity}\t{item.price}")

    # print total
    total = sum(i.quantity * i.price for i in invoice.items)
    print(f"TOTAL: $ {total:.2f}")
```

**AFTER — expected:**
```python
def print_invoice(self, invoice):
    self._print_header(invoice)
    self._print_items(invoice)
    self._print_total(invoice)

def _print_header(self, invoice):
    print("***********************")
    print(f"***   INVOICE #{invoice.number}   ***")
    print("***********************")

def _print_items(self, invoice):
    for item in invoice.items:
        print(f"{item.name}\t{item.quantity}\t{item.price}")

def _print_total(self, invoice):
    total = sum(i.quantity * i.price for i in invoice.items)
    print(f"TOTAL: $ {total:.2f}")
```

**Variant — method with return value:**
```python
# BEFORE — temporary variable calculated and used later
def calculate_discount(self, order):
    subtotal = 0
    for item in order.items:
        subtotal += item.price * item.quantity
    # ... other things ...
    return subtotal * order.discount_rate

# AFTER — extraction with return
def calculate_discount(self, order):
    subtotal = self._calculate_subtotal(order)
    return subtotal * order.discount_rate

def _calculate_subtotal(self, order):
    return sum(i.price * i.quantity for i in order.items)
```

**Variant — standalone function (no class state):**
```python
# When the extracted fragment does not depend on self, it can be a module-level function
def _calculate_subtotal(items):
    return sum(i.price * i.quantity for i in items)

class OrderService:
    def calculate_discount(self, order):
        subtotal = _calculate_subtotal(order.items)
        return subtotal * order.discount_rate
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Vague name that does not express intent**
```python
# Not accepted
def _process_part1(self, invoice): ...
def _helper(self, invoice): ...
```

**Mistake 2: Extracting a very small fragment without readability gain**
```python
# Not accepted — one line does not need to become a method
def _print_new_line(self):
    print()
```

**Mistake 3: Leaving the original method still large after extraction**
```python
# Not accepted — extracted one block but the method still has 50 lines
def print_invoice(self, invoice):
    self._print_header(invoice)
    # ... 40 remaining lines without extraction ...
```

---

## 7. Benefits

- **Readability:** The main method becomes a high-level narrative
- **Reuse:** The extracted method can be reused in other contexts
- **Testability:** Smaller methods are easier to test in isolation
- **Error isolation:** Changes only affect the method that contains them
