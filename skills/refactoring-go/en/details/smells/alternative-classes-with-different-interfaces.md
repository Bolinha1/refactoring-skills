# SKILL: Detecting and Refactoring Alternative Classes with Different Interfaces — Go

## Source
Based on: https://refactoring.guru/smells/alternative-classes-with-different-interfaces

---

## 1. What is Alternative Classes with Different Interfaces

Two or more types perform the same (or very similar) job but expose different method names and signatures, so callers cannot treat them interchangeably. The duplication is hidden behind the naming mismatch.

**Why this happens:**
- Two developers independently implemented the same concept without coordinating
- A type was copied and lightly renamed instead of reusing the existing one
- Libraries or packages evolved separately and converged on the same problem with different names

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Two structs have methods that do the same thing but with different names (e.g., `Send` vs `Deliver`, `Fetch` vs `Load`)
- [ ] Callers of both types contain parallel if/else or switch logic to call the right method
- [ ] One type is used in some packages and the other in other packages, with no clear reason
- [ ] A new feature must be implemented twice — once per type
- [ ] Unit tests for both types are near-identical clones

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Both types can share a common interface | Extract Interface, rename methods to match |
| One type is a subset of the other | Rename Method + unify under a single type |
| Types live in different packages | Define interface in the consumer package (Go idiom) |
| One type wraps an external dependency | Adapter pattern — wrap and expose canonical interface |

---

## 4. Example

**BEFORE — not accepted:**
```go
package notification

// EmailSender and SMSSender do the same job with different method names.

type EmailSender struct{ host string }

func (e *EmailSender) Send(to, body string) error {
	// send email
	return nil
}

type SMSSender struct{ apiKey string }

func (s *SMSSender) Deliver(to, message string) error {
	// send SMS
	return nil
}

// Caller must branch on the concrete type
func Notify(channel string, emailer *EmailSender, smser *SMSSender, recipient, text string) error {
	if channel == "email" {
		return emailer.Send(recipient, text)
	}
	return smser.Deliver(recipient, text)
}
```

**AFTER — expected:**
```go
package notification

// Notifier is the shared interface both senders now implement.
type Notifier interface {
	Notify(to, message string) error
}

type EmailSender struct{ host string }

func (e *EmailSender) Notify(to, message string) error {
	// send email
	return nil
}

type SMSSender struct{ apiKey string }

func (s *SMSSender) Notify(to, message string) error {
	// send SMS
	return nil
}

// Caller works with any Notifier — no branching needed.
func Send(n Notifier, recipient, text string) error {
	return n.Notify(recipient, text)
}
```

**Why this pattern:**
- The `Notifier` interface makes the shared contract explicit
- Adding a new channel (push, Slack) requires only a new struct — no branching in callers
- Go interfaces are implicit: no change to the struct declarations beyond renaming the method

---

## 5. Negative examples — what NOT to do

**Mistake 1: Adding a wrapper method without renaming the original**
```go
// Not accepted — the old mismatched method still exists; callers still diverge
func (s *SMSSender) Notify(to, msg string) error {
	return s.Deliver(to, msg) // just a pass-through; delete Deliver instead
}
```

**Mistake 2: Using an empty interface to paper over the mismatch**
```go
// Not accepted — loses type safety and requires type assertions everywhere
func Send(n interface{}, to, msg string) error { ... }
```

---

## 6. Benefits

- **Interchangeability:** Any `Notifier` can replace another without changing callers
- **Open/Closed:** New notification channels extend the system without modifying existing code
- **Testability:** Tests inject a mock `Notifier` instead of a concrete sender
