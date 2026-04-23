# SKILL: Detecting and Refactoring Speculative Generality — Go

## Source
Based on: https://refactoring.guru/smells/speculative-generality

---

## 1. What is it?

Code written to handle requirements that do not yet exist — and may never exist.
Interfaces with a single implementation, functions with parameters that are always
`nil` or zero, and structs that serve only as "future extension points" are all signs
that someone built in flexibility "just in case."

---

## 2. Warning signs

- [ ] An interface has only one concrete implementation and no plans for a second
- [ ] A function parameter is always passed `nil`, zero, or the same constant at every call site
- [ ] A struct or function exists only to support future requirements mentioned in a comment
- [ ] An extension point or hook has never been called since it was introduced
- [ ] Tests are the only callers of certain exported functions

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Inline Class** | A struct exists purely as a future extension point with no current consumers |
| **Collapse Hierarchy** | An interface has only one implementation and they can be merged |
| **Remove Parameter** | A parameter is always passed the same value at every call site |
| **Rename** | Abstract names were chosen to accommodate variety that never materialised |

---

## 4. Example

**BEFORE — not accepted:**
```go
// Notifier interface exists only because someone assumed other notifiers would come
type Notifier interface {
	Send(message, channel string) error
}

type EmailNotifier struct {
	client EmailClient
}

// channel is always "email" at every call site — the parameter is unused generality
func (n *EmailNotifier) Send(message, channel string) error {
	return n.client.Send(message)
}

// Every call site:
notifier.Send("Hello", "email")
```

**AFTER — expected:**
```go
type EmailNotifier struct {
	client EmailClient
}

func (n *EmailNotifier) Send(message string) error {
	return n.client.Send(message)
}

notifier.Send("Hello")
```

**Why this pattern:**
- The interface and unused `channel` parameter were speculative complexity
- If a second notifier type is ever needed, the interface can be introduced then,
  guided by real requirements

---

## 5. Negative examples — what NOT to do

**Mistake 1: Removing interfaces used for testing**
```go
// Caution — interfaces used to inject test doubles serve a real current purpose;
// verify there are no test mocks before removing
```

**Mistake 2: Removing an interface because it has one implementation today**
```go
// Caution — if a second implementation is already on the roadmap,
// the abstraction is not speculative; keep it
```

---

## 6. Benefits

- **Reduced complexity:** The codebase only contains code that serves current needs
- **Easier onboarding:** Developers don't need to understand hooks that do nothing yet
- **Simpler refactoring:** When real requirements arrive, abstractions are informed by actual use
