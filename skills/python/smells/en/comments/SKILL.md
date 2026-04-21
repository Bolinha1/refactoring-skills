# SKILL: Detecting and Refactoring Comments — Python

## Source
Based on: https://refactoring.guru/smells/comments

---

## 1. What is Comments (as a code smell)

A function or method is filled with explanatory comments because the code itself is not clear enough to be understood without them. The comment is a symptom: it marks a place where the code should speak for itself but doesn't.

**Important:** Not all comments are smells. Comments that explain *why* a non-obvious decision was made are valuable. Comments that explain *what* the code does are the smell — they should be replaced by better names and smaller functions.

**Why this happens:**
- Code was written to work, not to communicate
- Complex logic was left inline instead of being extracted to named functions
- Variable and function names do not express intent

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A comment describes *what* a block of code does (not *why*)
- [ ] A comment precedes a block that could be named and extracted
- [ ] A variable needs a comment to explain what it holds
- [ ] A function body has section dividers (`# --- step 1 ---`)
- [ ] A comment restates the function name in prose
- [ ] A multi-line docstring describes the algorithm line by line

---

## 3. Treatment techniques (in order of preference)

| Situation found                                        | Recommended technique     |
|--------------------------------------------------------|---------------------------|
| Comment describes a block of code                      | Extract Method            |
| Comment explains what a variable holds                 | Rename variable or Extract Variable |
| Comment explains a complex condition                   | Extract Method with descriptive name |
| Comment marks a TODO that should be code               | Introduce Assertion        |
| Comment explains WHY (non-obvious constraint)          | Keep the comment — it adds value |

---

## 4. Example

**BEFORE — not accepted:**
```python
def process_payment(payment) -> None:
    # validate that payment is not expired
    if payment.expiry_date < date.today():
        raise PaymentExpiredError("Payment has expired")

    # calculate net amount after fees
    fee = payment.amount * 0.025
    net_amount = payment.amount - fee

    # send to payment gateway
    gateway.submit(payment.card_number, net_amount)

    # notify customer
    email_service.send_payment_confirmation(payment.customer_email, net_amount)
```

**AFTER — expected:**
```python
def process_payment(payment) -> None:
    _validate_not_expired(payment)
    net_amount = _calculate_net_amount(payment)
    gateway.submit(payment.card_number, net_amount)
    email_service.send_payment_confirmation(payment.customer_email, net_amount)


def _validate_not_expired(payment) -> None:
    if payment.expiry_date < date.today():
        raise PaymentExpiredError("Payment has expired")


def _calculate_net_amount(payment) -> float:
    fee = payment.amount * PROCESSING_FEE_RATE
    return payment.amount - fee
```

**Why this pattern:**
- Function names replace comments — `_validate_not_expired` is more precise than `# validate`
- The main function reads like a business narrative, not implementation details

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the comment after extracting**
```python
# Not accepted — the comment and the function name say the same thing
# validate payment
_validate_payment(payment)
```

**Mistake 2: Extracting with a vague name**
```python
# Not accepted — the function name doesn't replace the comment
def _do_validation(payment): ...
```

**Mistake 3: Removing explanatory WHY comments**
```python
# Not accepted — this comment explains a non-obvious constraint; it should stay
# Using floor() intentionally — the bank truncates fractions, never rounds
fee = math.floor(payment.amount * FEE_RATE * 100) / 100
```

---

## 6. Benefits

- **Self-documentation:** Code that reads as prose needs no comments
- **Maintenance:** Function names cannot go stale the way comments do
- **Discovery:** Extracted functions become reusable building blocks
