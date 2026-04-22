# SKILL: Detecting and Refactoring Message Chains — Go

## Source
Based on: https://refactoring.guru/smells/message-chains

---

## 1. What is Message Chains

A sequence of method or field accesses chained together: `a.GetB().GetC().GetD()`. Each link in the chain couples the caller to the internal structure of every intermediate object, violating the Law of Demeter ("talk to your immediate neighbors only").

**Why this happens:**
- Convenience getters were added to expose internals without thinking about coupling
- Navigation logic accumulated inside callers instead of being delegated to the owner
- Data structures were accessed directly rather than through meaningful domain operations

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] A single expression contains three or more `.` accesses to reach a value
- [ ] The same chain appears in multiple callers
- [ ] Changing an intermediate object's structure requires updates across many call sites
- [ ] A caller holds a reference to an object only to access something two levels deeper
- [ ] Temporary variables named `order`, `customer`, `address` are used just to drill one level deeper

---

## 3. Treatment techniques (in order of preference)

| Situation found | Recommended technique |
|---|---|
| Chain reaches a value used in one operation | Hide Delegate: add a method on the closest object |
| Chain is navigating to call a behavior | Move Method closer to the data |
| Chain duplicated across several callers | Extract Function that encapsulates the navigation |
| Chain navigates to build a value | Replace chain with a dedicated query method |

---

## 4. Example

**BEFORE — not accepted:**
```go
package shipping

func PrintShippingLabel(order *Order) {
	// Caller must know Order → Customer → Address → City
	city := order.GetCustomer().GetAddress().GetCity()
	zip  := order.GetCustomer().GetAddress().GetZipCode()
	name := order.GetCustomer().GetName()

	fmt.Printf("Ship to: %s, %s %s\n", name, city, zip)
}
```

**AFTER — expected:**
```go
package shipping

// Hide Delegate: Order exposes ShippingLabel directly.
func (o *Order) ShippingLabel() string {
	return fmt.Sprintf("Ship to: %s, %s %s",
		o.customer.Name,
		o.customer.address.City,
		o.customer.address.ZipCode,
	)
}

func PrintShippingLabel(order *Order) {
	fmt.Println(order.ShippingLabel())
}
```

**Why this pattern:**
- `PrintShippingLabel` no longer knows that `Order` contains `Customer` which contains `Address`
- Changing the address structure only requires updating `Order.ShippingLabel`, not all callers
- The method name `ShippingLabel` expresses intent; the raw chain did not

---

## 5. Negative examples — what NOT to do

**Mistake 1: Breaking the chain into intermediate variables without removing the coupling**
```go
// Not accepted — the coupling is still there; it is just less visible
customer := order.GetCustomer()
address  := customer.GetAddress()
city     := address.GetCity()
```

**Mistake 2: Exposing all intermediates as public fields to shorten the chain**
```go
// Not accepted — public fields increase coupling further; any change is a breaking change
city := order.Customer.Address.City
```

---

## 6. Benefits

- **Reduced coupling:** Callers depend only on the object they hold, not its entire object graph
- **Change safety:** Internal restructuring does not cascade to callers
- **Expressiveness:** Delegate-hiding methods carry domain names, not navigation paths
