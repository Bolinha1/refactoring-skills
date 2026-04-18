# SKILL: Detecting and Refactoring Long Method — Java

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
- [ ] Complex conditional (nested if/else)
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
```java
public void processOrder(Order order) {
    // validate order
    if (order.getItems().isEmpty()) {
        throw new InvalidOrderException("Order has no items");
    }
    if (order.getCustomer() == null) {
        throw new InvalidOrderException("Customer not provided");
    }

    // calculate total
    double total = 0;
    for (Item item : order.getItems()) {
        total += item.getPrice() * item.getQuantity();
    }
    if (order.hasDiscount()) {
        total = total * (1 - order.getDiscount());
    }

    // persist
    orderRepository.save(order);
    emailService.sendConfirmation(order.getCustomer().getEmail());
}
```

**AFTER — expected:**
```java
public void processOrder(Order order) {
    validateOrder(order);
    double total = calculateTotal(order);
    order.setTotal(total);
    confirmOrder(order);
}

private void validateOrder(Order order) {
    if (order.getItems().isEmpty()) {
        throw new InvalidOrderException("Order has no items");
    }
    if (order.getCustomer() == null) {
        throw new InvalidOrderException("Customer not provided");
    }
}

private double calculateTotal(Order order) {
    double total = order.getItems().stream()
        .mapToDouble(i -> i.getPrice() * i.getQuantity())
        .sum();
    return order.hasDiscount() ? total * (1 - order.getDiscount()) : total;
}

private void confirmOrder(Order order) {
    orderRepository.save(order);
    emailService.sendConfirmation(order.getCustomer().getEmail());
}
```

**Why this pattern:**
- Each extracted method has a name that expresses business intent
- The main method became a readable narrative
- No comments needed — the name replaces them

---

## 5. Negative examples — what NOT to do

**Mistake 1: Extracting without naming with intent**
```java
// Not accepted — name does not describe what it does
private void doSomething(Order order) { ... }
private void method1(Order order) { ... }
private void processPart2(Order order) { ... }
```

**Mistake 2: Commenting instead of extracting**
```java
// Not accepted — comment indicates it should be a method
// validates customer data
if (customer == null || customer.getTaxId() == null) { ... }
```

**Mistake 3: Extracting and leaving the method still long**
```java
// Not accepted — extracted one block but the original method
// still has 60 lines with multiple responsibilities
```

---

## 6. Benefits

- **Longevity:** Classes with short methods live longest
- **Maintainability:** Reduces difficulty in understanding and maintaining code
- **Quality:** Prevents duplicate code hidden in long methods
- **Performance:** Negligible negative impact; enables better optimization opportunities
- **Clarity:** Clear and understandable code makes it easier to identify real performance improvements
