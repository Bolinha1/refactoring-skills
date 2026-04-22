# TECHNIQUE: Substitute Algorithm — Go

## Source
Based on: https://refactoring.guru/substitute-algorithm

---

## 1. Problem

You want to replace an existing algorithm with a cleaner, simpler, or more efficient
one. The algorithm works correctly but is hard to understand, hard to extend, or can
now be replaced with a Go standard-library function or idiom.

---

## 2. Solution

Replace the body of the function or method implementing the old algorithm with the
new algorithm. Keep the signature the same so callers are unaffected.

---

## 3. When to apply

- A simpler algorithm exists that produces the same results
- A Go built-in or stdlib function (`strings.Contains`, `slices.Contains`, map lookup)
  now covers what you implemented manually
- The current algorithm is difficult to extend incrementally — easier to rewrite than modify
- The algorithm has already been simplified as much as possible but is still unclear

---

## 4. Refactoring steps

1. Ensure the existing algorithm has thorough tests — you need them to verify the replacement
2. Write the new algorithm in a separate function (or on a branch)
3. Run the tests against the new algorithm; fix any failures
4. Once tests pass, replace the old algorithm body with the new one
5. Run the tests one final time
6. Delete any helper code that was only needed by the old algorithm

---

## 5. Example

**BEFORE — not accepted:**
```go
func findPerson(people []string) string {
	for _, person := range people {
		if person == "Don" {
			return "Don"
		}
		if person == "John" {
			return "John"
		}
		if person == "Kent" {
			return "Kent"
		}
	}
	return ""
}
```

**AFTER — expected:**
```go
func findPerson(people []string) string {
	candidates := map[string]bool{"Don": true, "John": true, "Kent": true}
	for _, person := range people {
		if candidates[person] {
			return person
		}
	}
	return ""
}
```

**Why this pattern:**
- Map lookup is O(1) vs. O(k) chained comparisons per element
- Adding a new candidate requires only one change (add to the map), not a new `if` block
- The intent — "first element in the set" — is immediately clear

---

## 6. Negative examples — what NOT to do

**Mistake 1: Substituting without a test suite**
```go
// Not accepted — without tests you cannot verify the new algorithm is equivalent;
// write the tests first before replacing anything
```

**Mistake 2: Incrementally patching the old algorithm instead of replacing it**
```go
// Not accepted — if the algorithm is fundamentally flawed, patches make it worse;
// substitute it entirely
```

---

## 7. Benefits

- **Simplicity:** The new algorithm is easier to read and maintain
- **Extensibility:** Well-structured algorithms are easier to modify
- **Idiomatic Go:** Uses maps and range loops instead of verbose if-chains
