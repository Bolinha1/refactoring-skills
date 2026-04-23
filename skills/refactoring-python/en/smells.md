# Code Smells — Python

Identify which smell is present, then choose a technique from `techniques.md`.
For full detail on any smell, read `details/smells/{name}.md`.

---

## Alternative Classes With Different Interfaces

Two or more classes perform similar or identical functions but have different method names and signatures. Because the interfaces differ, client code cannot treat them interchangeably and must know which concrete class it is dealing with.

→ `details/smells/alternative-classes-with-different-interfaces.md`

---

## Comments

A function or method is filled with explanatory comments because the code itself is not clear enough to be understood without them. The comment is a symptom: it marks a place where the code should speak for itself but doesn't. **Important:** Not all comments are smells. Comments that explain *why* a non-obvious decision was made are valuable. Comments that explain *what* the code does are the smell — they should be replaced by better names and smaller functions.

→ `details/smells/comments.md`

---

## Data Class

A class that contains only attributes and accessors — but no real behaviour. All the logic that operates on this data lives in other classes. The class is essentially a dumb data container and acts as a passive record rather than a responsible object.

→ `details/smells/data-class.md`

---

## Data Clumps

Different parts of the code contain identical groups of variables (a clump) — the same attributes in multiple classes, or the same parameters appearing together in many function signatures. If you removed one item from the clump and the rest stopped making sense, you have a data clump.

→ `details/smells/data-clumps.md`

---

## Dead Code

Variables, parameters, attributes, functions, methods, or entire classes that are no longer used anywhere. Dead code clutters the codebase, misleads future readers into thinking it is relevant, and must still be read and understood even though it does nothing.

→ `details/smells/dead-code.md`

---

## Divergent Change

A single class is changed for many different reasons. When you find yourself modifying the same class every time a new business rule, a new data source, or a new feature of unrelated concern is added, that class has divergent change. The inverse of Shotgun Surgery: one class, many reasons to change.

→ `details/smells/divergent-change.md`

---

## Duplicate Code

Two or more code fragments look almost identical or are structurally similar across different places in the codebase. Any change to the logic must be replicated in every copy, and forgetting one creates a silent bug.

→ `details/smells/duplicate-code.md`

---

## Feature Envy

A method accesses the data of another object more than it accesses the data of its own class. The method is "envious" of the other class — it seems to want to live there instead.

→ `details/smells/feature-envy.md`

---

## Inappropriate Intimacy

One class accesses the private attributes, internal lists, or implementation details of another class too freely. The two classes are tightly coupled in a way that makes it hard to change one without affecting the other.

→ `details/smells/inappropriate-intimacy.md`

---

## Incomplete Library Class

A library or third-party class lacks a method you need, and because you cannot (or should not) modify the library, you end up placing the missing behaviour in a utility function, a static helper, or a subclass. The logic that belongs to the library leaks into your own code.

→ `details/smells/incomplete-library-class.md`

---

## Large Class

A class that contains too many fields, methods, or lines of code. When a class tries to do too many things, it accumulates responsibilities that should be distributed.

→ `details/smells/large-class.md`

---

## Lazy Class

A class that does so little that it is not worth the overhead of understanding and maintaining it. It may have been useful in the past, may exist in anticipation of future growth, or may be the leftover shell of a refactoring that extracted most of its logic.

→ `details/smells/lazy-class.md`

---

## Long Method

A method that contains too many lines of code. As a rule of thumb: any method longer than 10 lines deserves attention.

→ `details/smells/long-method.md`

---

## Long Parameter List

A function or method has too many parameters — typically more than three or four. Long parameter lists are hard to understand, easy to confuse, and painful to call. They often indicate that related data should be grouped into a dataclass or object.

→ `details/smells/long-parameter-list.md`

---

## Message Chains

A client asks an object for another object, which asks yet another object, forming a chain: `a.get_b().get_c().get_d().do_something()`. The client is coupled to the entire chain of intermediaries. If any step changes its structure, the client breaks.

→ `details/smells/message-chains.md`

---

## Middle Man

A class whose only purpose is to delegate every call to another class. It adds a layer of indirection without adding any value. This is the inverse of Feature Envy: instead of a class that reaches into others, this is a class that others reach through unnecessarily.

→ `details/smells/middle-man.md`

---

## Parallel Inheritance Hierarchies

Every time you add a subclass to one class hierarchy, you are forced to add a corresponding subclass to another hierarchy. The two trees mirror each other: adding `PremiumUser` in one tree means adding `PremiumUserReport` in another tree.

→ `details/smells/parallel-inheritance-hierarchies.md`

---

## Primitive Obsession

Overuse of basic types (`str`, `int`, `float`, `bool`, `dict`, `list`) to represent domain concepts that deserve their own classes or dataclasses.

→ `details/smells/primitive-obsession.md`

---

## Refused Bequest

A subclass inherits methods or data from its parent class but does not use or actively overrides them to raise exceptions. The child rejects part of what the parent gives it, which breaks the Liskov Substitution Principle: a subclass should be usable wherever its parent is expected.

→ `details/smells/refused-bequest.md`

---

## Shotgun Surgery

A single logical change requires modifying many different modules or classes at the same time. Making one conceptual change "fires" edits across the codebase like a shotgun blast — touching many files for what should be a localized fix. The inverse of Divergent Change: many classes, one reason to change.

→ `details/smells/shotgun-surgery.md`

---

## Speculative Generality

Code that was written to handle requirements that do not yet exist — and may never exist. Abstract base classes with only one concrete subclass, hooks that are never called, parameters that are always passed `None`, and protocols with a single implementation are all signs that someone built in flexibility "just in case."

→ `details/smells/speculative-generality.md`

---

## Switch Statements

Complex if/elif chains (or match/case in Python 3.10+) that branch on the type or state of an object. The same switching logic tends to be duplicated across the codebase — when a new case is added, every branch must be found and updated.

→ `details/smells/switch-statements.md`

---

## Temporary Field

A class contains a field that is only set and used in certain situations — during one algorithm or method call. The rest of the time it is `None`, empty, or meaningless. Other code that reads the field cannot tell when it holds a valid value and when it doesn't.

→ `details/smells/temporary-field.md`

---
