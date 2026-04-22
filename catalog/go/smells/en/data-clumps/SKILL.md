# SKILL: Detecting and Refactoring Data Clumps — Go

## Source
Based on: https://refactoring.guru/smells/data-clumps

---

## 1. What is Data Clumps

Groups of fields or parameters that always appear together across multiple structs or function signatures. Because they travel as a pack, they belong in their own named type.

**Why this happens:**
- Related concepts were broken into individual primitives for simplicity
- The same group of fields was copy-pasted into multiple structs
- Function signatures grew one parameter at a time until a natural group emerged

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Three or more fields appear together in multiple structs
- [ ] A function signature has three or more parameters that are always passed together
- [ ] Removing one field from the group makes the remaining fields meaningless
- [ ] The same validation or formatting logic for the group is duplicated across callers
- [ ] Code that uses the group always accesses all its members together

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Clump of fields in multiple structs | Extract Struct, embed or reference it |
| Clump of parameters in many functions | Introduce Parameter Object (new struct) |
| Clump has its own validation logic | Move validation onto the new struct |
| Clump is a well-known domain concept | Name it clearly and make it a first-class type |

---

## 4. Example

**BEFORE — not accepted:**
```go
package shipping

type Order struct {
	ID              int
	Street          string
	City            string
	State           string
	ZipCode         string
	RecipientName   string
}

type Warehouse struct {
	Name    string
	Street  string
	City    string
	State   string
	ZipCode string
}

func CalculateShipping(street, city, state, zipCode string, weight float64) float64 {
	// repeated address handling
	_ = street + ", " + city + ", " + state + " " + zipCode
	return weight * 1.5
}
```

**AFTER — expected:**
```go
package shipping

type Address struct {
	Street  string
	City    string
	State   string
	ZipCode string
}

func (a Address) String() string {
	return a.Street + ", " + a.City + ", " + a.State + " " + a.ZipCode
}

func (a Address) Validate() error {
	if a.ZipCode == "" {
		return fmt.Errorf("zip code is required")
	}
	return nil
}

type Order struct {
	ID            int
	ShipTo        Address
	RecipientName string
}

type Warehouse struct {
	Name    string
	Location Address
}

func CalculateShipping(destination Address, weight float64) float64 {
	_ = destination.String()
	return weight * 1.5
}
```

**Why this pattern:**
- `Address` is defined once; changes to its structure propagate automatically
- Validation lives on the type, not scattered across callers
- Function signatures shrink: one `Address` replaces four string parameters

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping unrelated fields just to shorten a signature**
```go
// Not accepted — UserContext mixes authentication and shipping concerns
type UserContext struct {
	UserID  int
	Street  string
	City    string
	Token   string
}
```

**Mistake 2: Using a map instead of a struct**
```go
// Not accepted — map[string]string loses type safety and discoverability
func CalculateShipping(address map[string]string, weight float64) float64 { ... }
```

---

## 6. Benefits

- **Single source of truth:** Address format and validation are defined once
- **Cleaner signatures:** Functions accept meaningful objects, not primitive lists
- **Discoverability:** A named type is searchable; a group of strings is not
