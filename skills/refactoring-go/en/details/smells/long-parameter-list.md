# SKILL: Detecting and Refactoring Long Parameter List — Go

## Source
Based on: https://refactoring.guru/smells/long-parameter-list

---

## 1. What is Long Parameter List

A function or method that accepts too many parameters. Long parameter lists are hard to read, easy to call in the wrong order, and signal that the function is doing too much or that related parameters should be grouped into a struct.

**Why this happens:**
- Parameters were added one at a time as requirements grew
- The function merged what should have been separate concerns
- Related data was not grouped into a struct because it seemed "simpler" to pass primitives

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A function has more than three or four parameters
- [ ] Multiple parameters of the same type appear next to each other (easy to swap accidentally)
- [ ] Several parameters are always passed together from every call site
- [ ] A boolean parameter changes the behavior of the function significantly
- [ ] A call site passes `nil`, `""`, or `0` for parameters that are "not applicable"

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Parameters always travel together | Introduce Parameter Object (new struct) |
| Parameter comes from another object | Replace Parameter with Method Call |
| Boolean flag changes function behavior | Split into two separate functions |
| Optional parameters with defaults | Use a config struct or functional options pattern |
| Too many unrelated parameters | Split the function into focused functions |

---

## 4. Example

**BEFORE — not accepted:**
```go
package email

func SendWelcome(
	toName string,
	toEmail string,
	fromName string,
	fromEmail string,
	subject string,
	templateID string,
	locale string,
	sendAt time.Time,
) error {
	// ...
	return nil
}

// Call site — which string is which?
err := SendWelcome("Alice", "alice@example.com", "Support", "support@co.com",
	"Welcome!", "tmpl_001", "en-US", time.Now())
```

**AFTER — expected:**
```go
package email

type Address struct {
	Name  string
	Email string
}

type WelcomeEmailOptions struct {
	To         Address
	From       Address
	Subject    string
	TemplateID string
	Locale     string
	SendAt     time.Time
}

func SendWelcome(opts WelcomeEmailOptions) error {
	// ...
	return nil
}

// Call site — self-documenting
err := SendWelcome(email.WelcomeEmailOptions{
	To:         email.Address{Name: "Alice", Email: "alice@example.com"},
	From:       email.Address{Name: "Support", Email: "support@co.com"},
	Subject:    "Welcome!",
	TemplateID: "tmpl_001",
	Locale:     "en-US",
	SendAt:     time.Now(),
})
```

**Why this pattern:**
- Named struct fields make call sites self-documenting
- Related addresses are grouped into the `Address` type (Data Clumps resolved)
- Adding a new option requires only a new struct field, not a signature change for all callers

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping unrelated parameters into one struct**
```go
// Not accepted — mixing email content, auth credentials, and schedule in one blob
type SendParams struct {
	ToEmail   string
	APISecret string // auth concern mixed in
	RetryMax  int    // infrastructure concern mixed in
	Subject   string
}
```

**Mistake 2: Using variadic interface{} to avoid defining a struct**
```go
// Not accepted — loses all type safety and IDE support
func Send(params ...interface{}) error { ... }
```

**Mistake 3: Splitting into many boolean flags instead**
```go
// Not accepted — boolean trap; callers can't tell what true/false means
func Send(to, from, subject string, isHTML bool, isScheduled bool, sendNow bool) error { ... }
```

---

## 6. Benefits

- **Readability:** Named struct fields make call sites self-documenting
- **Safety:** Struct fields cannot be passed in the wrong order
- **Extensibility:** New options are added as struct fields without breaking existing callers
