# TECHNIQUE: Replace Nested Conditional with Guard Clauses — Python

## Source
Based on: https://refactoring.guru/replace-nested-conditional-with-guard-clauses

---

## 1. Problem

A function has deeply nested conditionals that obscure the main happy path. Readers must mentally track every nesting level to understand what the function normally does.

---

## 2. Solution

Replace special-case conditions with early returns (guard clauses) at the top of the function. The main logic runs unindented at the bottom.

---

## 3. When to apply

- The function has 2+ levels of nesting that encode edge cases or preconditions
- The else branch of a top-level `if` contains the main logic
- Each nested condition checks a precondition or exceptional case before the real work
- The function ends with a single return buried inside multiple else blocks

---

## 4. Refactoring steps

1. Identify each special case (None check, error condition, early exit)
2. For each special case, move its condition to the top of the function
3. Return early (or raise) inside that guard clause
4. Remove the now-unnecessary else nesting
5. Verify the main logic runs unindented
6. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
def get_pay_amount(employee: Employee) -> float:
    if employee.is_dead:
        result = dead_amount()
    else:
        if employee.is_separated:
            result = separated_amount()
        else:
            if employee.is_retired:
                result = retired_amount()
            else:
                result = normal_pay_amount()
    return result
```

**AFTER — expected:**
```python
def get_pay_amount(employee: Employee) -> float:
    if employee.is_dead:      return dead_amount()
    if employee.is_separated: return separated_amount()
    if employee.is_retired:   return retired_amount()
    return normal_pay_amount()
```

**Why this pattern:**
- The exceptional cases are dispatched at the top — readers skip past them immediately
- `normal_pay_amount()` stands out as the default: no nesting, no else, no variable accumulation

---

## 6. Negative examples — what NOT to do

**Mistake 1: Using guard clauses for the main case, not the exceptions**
```python
# Not accepted — guard clauses should handle the exceptions; normal flow should be at bottom
def get_pay_amount(employee: Employee) -> float:
    if not employee.is_dead and not employee.is_separated and not employee.is_retired:
        return normal_pay_amount()  # inverted — checking for normality is the guard
    # edge cases below...
```

**Mistake 2: Adding a guard clause that duplicates an existing guarantee**
```python
# Not accepted — caller contract guarantees employee is never None
if employee is None:
    return 0.0  # adds noise without adding safety
if employee.is_dead:
    return dead_amount()
```

**Mistake 3: Flattening nested conditions that aren't independent guard clauses**
```python
# Not accepted — these conditions are not guards; they represent business logic branches
# Use Decompose Conditional instead of guard clauses here
if plan == "summer":
    return summer_charge()  # not an exception — it's a valid main path
```

---

## 7. Benefits

- **Readability:** The function's normal execution path is immediately visible
- **Reduced nesting:** Each guard clause removes one level of indentation
- **Intention clarity:** Early returns signal "this case is exceptional — stop here"
