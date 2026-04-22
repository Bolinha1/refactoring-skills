# SKILL: Detecting and Refactoring Long Method — Python

## Source
Based on: https://refactoring.guru/smells/long-method

---

## 1. What is Long Method

A method that contains too many lines of code.
As a rule of thumb: any method longer than 10 lines deserves attention.

**Why this happens:**
- Logic is always added to the method, never removed
- It is mentally easier to add two lines than to create a new method
- The problem grows silently until it becomes spaghetti code

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Method with more than 10 lines
- [ ] Code block you felt the urge to comment
- [ ] Complex conditional (nested if/elif)
- [ ] Loop with non-trivial logic inside
- [ ] Temporary variables scattered throughout the method
- [ ] Method that clearly does more than one distinct thing

---

## 3. Treatment techniques (in order of preference)

| Situation found                              | Recommended technique             |
|----------------------------------------------|-----------------------------------|
| Commented or commentable code block          | Extract Method                    |
| Local variables prevent extraction           | Replace Temp with Query           |
| Too many parameters in extracted method      | Introduce Parameter Object        |
| Whole object being dismembered               | Preserve Whole Object             |
| None of the above works                      | Replace Method with Method Object |
| Complex conditional                          | Decompose Conditional             |
| Loop with complex internal logic             | Extract Method                    |

**Golden rule:** If you felt the urge to write a comment inside the method,
that block should become a method with a descriptive name.
The name replaces the comment.

---

## 4. Example

**BEFORE — not accepted:**
```python
def process_order(self, order):
    # validate order
    if not order.items:
        raise InvalidOrderError("Order has no items")
    if order.customer is None:
        raise InvalidOrderError("Customer not provided")

    # calculate total
    total = 0
    for item in order.items:
        total += item.price * item.quantity
    if order.has_discount:
        total = total * (1 - order.discount)

    # persist
    self.order_repository.save(order)
    self.email_service.send_confirmation(order.customer.email)
```

**AFTER — expected:**
```python
def process_order(self, order):
    self._validate_order(order)
    total = self._calculate_total(order)
    order.total = total
    self._confirm_order(order)

def _validate_order(self, order):
    if not order.items:
        raise InvalidOrderError("Order has no items")
    if order.customer is None:
        raise InvalidOrderError("Customer not provided")

def _calculate_total(self, order):
    total = sum(item.price * item.quantity for item in order.items)
    return total * (1 - order.discount) if order.has_discount else total

def _confirm_order(self, order):
    self.order_repository.save(order)
    self.email_service.send_confirmation(order.customer.email)
```

**Why this pattern:**
- Each private method (`_`) has a name that expresses business intent
- The main method became a readable narrative
- No comments needed — the name replaces them

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting without naming with intent**
```python
# Not accepted — name does not describe what it does
def _do_something(self, order): ...
def _method1(self, order): ...
def _process_part2(self, order): ...
```

**Mistake 2: Commenting instead of extracting**
```python
# Not accepted — comment indicates it should be a method
# validates customer data
if customer is None or customer.tax_id is None:
    ...
```

**Mistake 3: Extracting and leaving the method still long**
```python
# Not accepted — extracted one block but the original method
# still has 60 lines with multiple responsibilities
```

---

## 6. Benefits

- **Longevity:** Classes with short methods live longest
- **Maintainability:** Reduces difficulty in understanding and maintaining code
- **Quality:** Prevents duplicate code hidden in long methods
- **Performance:** Negligible negative impact; enables better optimization opportunities
- **Clarity:** Clear and understandable code makes it easier to identify real performance improvements
