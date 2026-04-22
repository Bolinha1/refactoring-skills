# SKILL: Detecting and Refactoring Comments — PHP

## Source
Based on: https://refactoring.guru/smells/comments

---

## 1. What is Comments (as a code smell)

A method is filled with explanatory comments because the code itself is not clear enough to be understood without them. The comment is a symptom: it marks a place where the code should speak for itself but doesn't.

**Important:** Not all comments are smells. Comments that explain *why* a non-obvious decision was made are valuable. Comments that explain *what* the code does are the smell — they should be replaced by better names and smaller methods.

**Why this happens:**
- Code was written to work, not to communicate
- Complex logic was left inline instead of being extracted to named methods
- Variable and method names do not express intent

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A comment describes *what* a block of code does (not *why*)
- [ ] A comment precedes a block that could be named and extracted
- [ ] A variable needs a comment to explain what it holds
- [ ] A method body has section dividers (`// --- step 1 ---`)
- [ ] A comment restates the method name in prose

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
```php
public function processPayment(Payment $payment): void
{
    // validate that payment is not expired
    if ($payment->getExpiryDate() < new \DateTime()) {
        throw new PaymentExpiredException('Payment has expired');
    }

    // calculate net amount after fees
    $fee = $payment->getAmount() * 0.025;
    $netAmount = $payment->getAmount() - $fee;

    // send to payment gateway
    $this->gateway->submit($payment->getCardNumber(), $netAmount);

    // notify customer
    $this->emailService->sendPaymentConfirmation($payment->getCustomerEmail(), $netAmount);
}
```

**AFTER — expected:**
```php
public function processPayment(Payment $payment): void
{
    $this->validateNotExpired($payment);
    $netAmount = $this->calculateNetAmount($payment);
    $this->gateway->submit($payment->getCardNumber(), $netAmount);
    $this->emailService->sendPaymentConfirmation($payment->getCustomerEmail(), $netAmount);
}

private function validateNotExpired(Payment $payment): void
{
    if ($payment->getExpiryDate() < new \DateTime()) {
        throw new PaymentExpiredException('Payment has expired');
    }
}

private function calculateNetAmount(Payment $payment): float
{
    $fee = $payment->getAmount() * self::PROCESSING_FEE_RATE;
    return $payment->getAmount() - $fee;
}
```

**Why this pattern:**
- Method names replace comments — `validateNotExpired` is more precise than `// validate`
- The main method reads like a business narrative, not implementation details

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping the comment after extracting**
```php
// Not accepted — the comment and the method name say the same thing
// validate payment
$this->validatePayment($payment);
```

**Mistake 2: Extracting with a vague name**
```php
// Not accepted — the method name doesn't replace the comment
private function doValidation(Payment $payment): void { ... }
```

**Mistake 3: Removing explanatory WHY comments**
```php
// Not accepted — this comment explains a non-obvious constraint; it should stay
// Using floor() intentionally — the bank truncates fractions, never rounds
$fee = floor($payment->getAmount() * self::FEE_RATE * 100) / 100;
```

---

## 6. Benefits

- **Self-documentation:** Code that reads as prose needs no comments
- **Maintenance:** Method names cannot go stale the way comments do
- **Discovery:** Extracted methods become reusable building blocks
