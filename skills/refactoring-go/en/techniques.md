# Refactoring Techniques — Go

Choose the technique that fits the smell you identified in `smells.md`.
For full detail on any technique, read `details/techniques/{name}.md`.

---

## Decompose Conditional

A complex `if/else` or `switch` statement contains intricate logic in its condition and branches. The code is hard to read because the intent of each part is buried in low-level detail.

→ `details/techniques/decompose-conditional.md`

---

## Extract Class

A struct is doing work that should be done by two. It has grown to hold fields and methods that belong to a distinct concept, making it harder to understand and change.

→ `details/techniques/extract-class.md`

---

## Extract Method

A function or method contains a fragment that can be grouped into a meaningful unit, but it is embedded alongside other responsibilities. The long body makes the intent hard to follow at a glance.

→ `details/techniques/extract-method.md`

---

## Extract Variable

An expression is complex or hard to understand at a glance. Readers must mentally evaluate it before they can grasp what the surrounding code is doing.

→ `details/techniques/extract-variable.md`

---

## Hide Delegate

A client reaches through one object to call a method on another object it obtained from the first. The client is now coupled to both objects, so any change to the intermediate object's API ripples out to every caller.

→ `details/techniques/hide-delegate.md`

---

## Inline Class

A struct is no longer pulling its weight. It has too few responsibilities — perhaps after other refactorings — and its existence adds indirection without adding clarity.

→ `details/techniques/inline-class.md`

---

## Inline Method

A method's body is as clear as its name, or the method is only a thin wrapper that adds no value. The extra indirection makes the code harder to follow rather than easier.

→ `details/techniques/inline-method.md`

---

## Inline Temp

A local variable holds the result of a simple expression, and the variable name adds no information that the expression itself does not already communicate clearly.

→ `details/techniques/inline-temp.md`

---

## Introduce Parameter Object

Several functions share the same group of parameters. Passing them individually is verbose, error-prone, and makes signatures hard to read. When a new related parameter must be added, every function signature must change.

→ `details/techniques/introduce-parameter-object.md`

---

## Move Field

A field in one struct is used more heavily by another struct, or it logically belongs to a concept that is modelled by a different struct. The field is in the wrong home, causing unnecessary coupling.

→ `details/techniques/move-field.md`

---

## Move Method

A method uses data or behaviour from another struct more than from the struct it currently belongs to. Its current location creates unnecessary coupling and makes the other struct harder to use independently.

→ `details/techniques/move-method.md`

---

## Remove Assignments To Parameters

A parameter variable is reassigned inside a function. In Go, scalar and struct parameters are passed by value, so reassigning the parameter variable has no effect on the caller — but it is still confusing. For pointer parameters, reassigning the pointer variable (not the pointed-to value) is similarly misleading.

→ `details/techniques/remove-assignments-to-parameters.md`

---

## Remove Middle Man

A struct has grown a collection of wrapper methods that do nothing but forward calls to a delegate. Every time the delegate adds a new method, the middle-man struct must add a corresponding forwarder. The wrapper adds indirection without adding value.

→ `details/techniques/remove-middle-man.md`

---

## Replace Conditional With Polymorphism

You have a `switch` or `if/else` chain that performs different behaviour depending on a type field or a string that simulates a type. The same conditional tends to appear in multiple places and must be updated whenever a new type is added.

→ `details/techniques/replace-conditional-with-polymorphism.md`

---

## Replace Method With Method Object

A method is so large and complex that extracting sub-functions is awkward because they would all share many local variables. The variables are too intertwined to pass as parameters without creating unwieldy signatures.

→ `details/techniques/replace-method-with-method-object.md`

---

## Replace Nested Conditional With Guard Clauses

A function has deeply nested `if/else` blocks that make the happy path hard to see. Each level of nesting adds cognitive load and pushes the main logic to the right.

→ `details/techniques/replace-nested-conditional-with-guard-clauses.md`

---

## Replace Temp With Query

You use a temporary variable to store the result of an expression, then reference that variable later in the same method. The expression is re-evaluated every time the method runs but is never surfaced as reusable logic.

→ `details/techniques/replace-temp-with-query.md`

---

## Split Temporary Variable

A temporary variable is assigned more than once within the same method, but each assignment serves a different purpose. Reusing the same name for unrelated values makes the code harder to follow and prevents the compiler from catching type mismatches.

→ `details/techniques/split-temporary-variable.md`

---

## Substitute Algorithm

You want to replace an existing algorithm with a cleaner, simpler, or more efficient one. The algorithm works correctly but is hard to understand, hard to extend, or can now be replaced with a Go standard-library function or idiom.

→ `details/techniques/substitute-algorithm.md`

---
