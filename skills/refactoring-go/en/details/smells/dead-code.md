# SKILL: Detecting and Refactoring Dead Code — Go

## Source
Based on: https://refactoring.guru/smells/dead-code

---

## 1. What is Dead Code

Code that is never executed or reachable: unreachable statements after a `return`, unexported identifiers nobody references, build-tag-disabled blocks, or functions kept "just in case". Dead code misleads readers and silently accumulates technical debt.

**Why this happens:**
- A feature was removed but its implementation was left behind
- A refactoring made code unreachable without cleaning up the old path
- Developers keep old code "for reference" instead of trusting version control
- Build tags or feature flags were retired but the guarded code was not removed

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Code after a `return`, `panic`, or `os.Exit` in the same block
- [ ] Unexported function, type, or variable with zero references in the package
- [ ] A condition that is always true or always false
- [ ] A `//go:build ignore` or retired feature flag protecting a code block
- [ ] Commented-out code blocks that have been there for more than one sprint
- [ ] An `else` branch that can never be reached given the preceding `if`

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Unreachable statement after return/panic | Delete the dead statement |
| Unexported symbol with no references | Delete the symbol (use `gopls` references to verify) |
| Always-true/always-false condition | Simplify or delete the branch |
| Commented-out code | Delete it — version control preserves history |
| Retired build-tag block | Delete the block and the build tag |

---

## 4. Example

**BEFORE — not accepted:**
```go
package payment

func (p *Processor) Charge(amount float64) error {
	if amount <= 0 {
		return fmt.Errorf("invalid amount")
	}
	err := p.gateway.Charge(amount)
	if err != nil {
		return err
	}
	return nil
	// TODO: add retry logic someday
	p.retry(amount) // unreachable
}

// legacyCharge was replaced by Charge in v2 — kept just in case
func (p *Processor) legacyCharge(amount float64) error {
	// ...
	return nil
}

// debugDump was used during development
// func (p *Processor) debugDump() { fmt.Println(p) }
```

**AFTER — expected:**
```go
package payment

func (p *Processor) Charge(amount float64) error {
	if amount <= 0 {
		return fmt.Errorf("invalid amount")
	}
	return p.gateway.Charge(amount)
}
```

**Why this pattern:**
- Every remaining line is reachable and purposeful
- `legacyCharge` is gone — git history preserves it if ever needed
- The commented-out debug helper is deleted rather than left as noise

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping dead code "just in case"**
```go
// Not accepted — version control is the safety net, not commented-out code
// func (p *Processor) legacyCharge(amount float64) error { ... }
```

**Mistake 2: Guarding dead code with a constant flag**
```go
// Not accepted — the code is still dead; the flag just makes it less obvious
const enableLegacy = false
if enableLegacy {
	p.legacyCharge(amount)
}
```

**Mistake 3: Leaving a TODO above unreachable code**
```go
// Not accepted — the TODO does not make the code live
return result
// TODO: log result here
fmt.Println(result)
```

---

## 6. Benefits

- **Clarity:** Readers focus on code that actually runs
- **Confidence:** Static analysis tools (`go vet`, `staticcheck`) report fewer false positives
- **Build size:** Unexported dead code is eliminated by the compiler, but removing it speeds up reads and reviews
