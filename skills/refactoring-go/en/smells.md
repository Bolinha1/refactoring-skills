# Code Smells — Go

Identify which smell is present, then choose a technique from `techniques.md`.
For full detail on any smell, read `details/smells/{name}.md`.

---

## Alternative Classes With Different Interfaces

Two or more types perform the same (or very similar) job but expose different method names and signatures, so callers cannot treat them interchangeably. The duplication is hidden behind the naming mismatch.

→ `details/smells/alternative-classes-with-different-interfaces.md`

---

## Comments

Comments that explain *what* the code does rather than *why* it does it. When code needs a prose annotation to be understood, it usually means the code itself is unclear — the comment is a deodorant masking a deeper problem.

→ `details/smells/comments.md`

---

## Data Class

A struct that contains only fields (and possibly simple getters/setters) but no real behavior. All the logic that operates on the struct lives in other packages, making the type a passive data bag rather than a domain object.

→ `details/smells/data-class.md`

---

## Data Clumps

Groups of fields or parameters that always appear together across multiple structs or function signatures. Because they travel as a pack, they belong in their own named type.

→ `details/smells/data-clumps.md`

---

## Dead Code

Code that is never executed or reachable: unreachable statements after a `return`, unexported identifiers nobody references, build-tag-disabled blocks, or functions kept "just in case". Dead code misleads readers and silently accumulates technical debt.

→ `details/smells/dead-code.md`

---

## Divergent Change

A single struct or package that must be modified for many different, unrelated reasons. Every time a new feature is added — payment logic, report format, third-party integration — the same file is touched. The type has absorbed multiple responsibilities.

→ `details/smells/divergent-change.md`

---

## Duplicate Code

Two or more code fragments look nearly identical or are structurally equivalent across different parts of the codebase. Any change to the logic must be replicated in every copy, and missing one creates a silent bug.

→ `details/smells/duplicate-code.md`

---

## Feature Envy

A function accesses the data of another struct more than it accesses its own. The function seems to want to live in the other type — it is "envious" of that type's data.

→ `details/smells/feature-envy.md`

---

## Inappropriate Intimacy

Two types are too tightly coupled: they access each other's internal details, manipulate each other's unexported state through accessor tricks, or are mutually dependent in ways that make them impossible to test or change independently.

→ `details/smells/inappropriate-intimacy.md`

---

## Incomplete Library Class

A type from an external package or the standard library lacks the behavior your code needs, but you cannot add methods to it directly (Go does not allow adding methods to types defined in other packages). The smell appears as ad-hoc workarounds scattered across your codebase.

→ `details/smells/incomplete-library-class.md`

---

## Large Class

A struct that has too many fields, methods, or lines of code. When a type tries to do too many things it accumulates responsibilities that should be distributed across focused types and packages.

→ `details/smells/large-class.md`

---

## Lazy Class

A type (struct, interface, or package) that does so little it no longer justifies its existence. It may have been meaningful during a previous design phase, or it was created speculatively and never grew into its intended role.

→ `details/smells/lazy-class.md`

---

## Long Method

A function or method that has grown too long to be understood at a glance. Long functions accumulate multiple levels of abstraction, making them difficult to read, test, and change safely.

→ `details/smells/long-method.md`

---

## Long Parameter List

A function or method that accepts too many parameters. Long parameter lists are hard to read, easy to call in the wrong order, and signal that the function is doing too much or that related parameters should be grouped into a struct.

→ `details/smells/long-parameter-list.md`

---

## Message Chains

A sequence of method or field accesses chained together: `a.GetB().GetC().GetD()`. Each link in the chain couples the caller to the internal structure of every intermediate object, violating the Law of Demeter ("talk to your immediate neighbors only").

→ `details/smells/message-chains.md`

---

## Middle Man

A type whose methods do nothing but delegate to another type. If more than half of a type's methods are one-line pass-throughs to another object, the middle man adds indirection without adding value.

→ `details/smells/middle-man.md`

---

## Parallel Inheritance Hierarchies

Two families of types mirror each other: every time a new type is added to one family, a corresponding type must be added to the other. In Go this manifests as parallel sets of structs and interfaces that must always be extended in lockstep.

→ `details/smells/parallel-inheritance-hierarchies.md`

---

## Primitive Obsession

Using primitive types (`string`, `int`, `float64`) to represent domain concepts that deserve their own types. A phone number is not just a `string`; a price is not just a `float64`; a user ID is not just an `int`.

→ `details/smells/primitive-obsession.md`

---

## Refused Bequest

A struct embeds another struct (or implements an interface) but does not use most of the promoted methods — or overrides them with panics. In Go, inheritance is replaced by embedding and interfaces, so this smell manifests as embedding abuse: a type

→ `details/smells/refused-bequest.md`

---

## Shotgun Surgery

A single logical change requires modifying many different packages or structs at the same time. Adding or renaming a concept "fires" edits across the codebase like a shotgun blast — touching many files for what should be a localized fix.

→ `details/smells/shotgun-surgery.md`

---

## Speculative Generality

Code written to handle requirements that do not yet exist — and may never exist. Interfaces with a single implementation, functions with parameters that are always `nil` or zero, and structs that serve only as "future extension points" are all signs

→ `details/smells/speculative-generality.md`

---

## Switch Statements

Complex `switch` or `if/else` chains that branch on the type or state of a value. The same switch logic tends to be duplicated across the codebase — when a new case is added, every switch must be found and updated.

→ `details/smells/switch-statements.md`

---

## Temporary Field

A struct contains a field that is only set and used during one method call or algorithm. The rest of the time it is zero-valued or meaningless. Other code that reads the field cannot tell when it holds a valid value and when it doesn't.

→ `details/smells/temporary-field.md`

---
