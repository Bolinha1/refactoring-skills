# Refactoring Techniques — Java

Choose the technique that fits the smell you identified in `smells.md`.
For full detail on any technique, read `details/techniques/{name}.md`.

---

## Decompose Conditional

A complex conditional expression (and the code in its branches) makes it hard to understand what is being tested and what happens in each case.

→ `details/techniques/decompose-conditional.md`

---

## Extract Class

One class does the work of two. A subset of its fields and methods forms a coherent concept that would make sense as its own class.

→ `details/techniques/extract-class.md`

---

## Extract Method

You have a code fragment that can be grouped into a meaningful unit, but it is embedded in a larger method alongside other responsibilities.

→ `details/techniques/extract-method.md`

---

## Extract Variable

A complex expression is hard to understand at a glance. Intermediate results are computed inline, making it difficult to see what each part means or to debug the calculation.

→ `details/techniques/extract-variable.md`

---

## Hide Delegate

A client accesses a delegate object by navigating a chain through the server object: `client → server → delegate`. When the delegate changes, the client must change too.

→ `details/techniques/hide-delegate.md`

---

## Inline Class

A class no longer does enough to justify its existence. It has too few responsibilities and is just an unnecessary indirection.

→ `details/techniques/inline-class.md`

---

## Inline Method

A method body is as obvious as its name, or the method is only used once and the indirection adds no value. Keeping a trivial wrapper around a single expression makes the code harder to follow, not easier.

→ `details/techniques/inline-method.md`

---

## Inline Temp

A local variable holds the result of a simple expression, and the variable name adds no information that the expression itself does not already communicate clearly.

→ `details/techniques/inline-temp.md`

---

## Introduce Parameter Object

Several parameters that naturally belong together are always passed as a group to the same methods. The group represents a concept not yet formalized in the codebase.

→ `details/techniques/introduce-parameter-object.md`

---

## Move Field

A field is used more by another class than by its own class — other classes read or write it more frequently than the class that contains it.

→ `details/techniques/move-field.md`

---

## Move Method

A method is used more in another class than in its own class. This creates unnecessary coupling and violates the cohesion principle.

→ `details/techniques/move-method.md`

---

## Remove Assignments To Parameters

A method parameter is reassigned inside the method body. This is confusing because it hides the original value, makes the method harder to understand, and can mislead callers who expect pass-by-value behaviour (objects are passed by reference in Java, so assigning a new object to the parameter does NOT affect the caller's variable, but mutating the object's state does).

→ `details/techniques/remove-assignments-to-parameters.md`

---

## Remove Middle Man

A class has too many simple delegation methods that do nothing except forward calls to another object. The class is a Middle Man — it exists only to pass messages through.

→ `details/techniques/remove-middle-man.md`

---

## Replace Conditional With Polymorphism

You have a `switch` or `if/else` chain that performs different behaviours depending on the object type or a property that simulates a type.

→ `details/techniques/replace-conditional-with-polymorphism.md`

---

## Replace Method With Method Object

A method is so large and complex that extracting sub-methods is awkward because they would all share many local variables. The local variables are too intertwined to be easily passed as parameters.

→ `details/techniques/replace-method-with-method-object.md`

---

## Replace Nested Conditional With Guard Clauses

A method has deeply nested conditionals that obscure the main happy path. Readers must mentally track every nesting level to understand what the method normally does.

→ `details/techniques/replace-nested-conditional-with-guard-clauses.md`

---

## Replace Temp With Query

A local variable holds the result of an expression. That value is used later in the method, but the variable prevents the expression from being reused in other methods or subclasses. The method grows because it carries state that could be pushed into a reusable query.

→ `details/techniques/replace-temp-with-query.md`

---

## Split Temporary Variable

A local variable is assigned more than once, serving different purposes at different points in the method. Reusing the same name for different concepts makes the code hard to follow and prevents each assignment from being made `final`.

→ `details/techniques/split-temporary-variable.md`

---

## Substitute Algorithm

You want to replace an existing algorithm with a cleaner, simpler, or more efficient one. The algorithm works correctly but is hard to understand, hard to extend, or can now be replaced with a library method.

→ `details/techniques/substitute-algorithm.md`

---
