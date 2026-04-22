# TECHNIQUE: Hide Delegate — Java

## Source
Based on: https://refactoring.guru/hide-delegate

---

## 1. Problem

A client accesses a delegate object by navigating a chain through the server object:
`client → server → delegate`. When the delegate changes, the client must change too.

---

## 2. Solution

Create a method on the server that hides the delegate. The client now only talks to the server.

---

## 3. When to apply

- A client calls `server.getDelegate().doSomething()`
- A change in the delegate class requires changes in all clients that navigate to it
- The delegation chain creates Message Chains smell
- You want clients to depend only on the server's public interface, not on its internal structure

---

## 4. Refactoring steps

1. For each method of the delegate that clients call through the server:
   - Create a delegating method on the server class
2. Update the client to call the new server method instead of navigating the chain
3. If no client accesses the delegate directly anymore, remove the getter for the delegate
4. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```java
// Client navigates the chain: person → department → manager
public class Person {
    private Department department;
    public Department getDepartment() { return department; }
}

public class Department {
    private Person manager;
    public Person getManager() { return manager; }
}

// Client code — must know about both Person and Department
Person manager = person.getDepartment().getManager();
```

**AFTER — expected:**
```java
public class Person {
    private Department department;

    public Person getManager() {
        return department.getManager(); // delegates, but hides Department from client
    }
    // getDepartment() can be removed if no one else needs it
}

// Client code — only knows about Person
Person manager = person.getManager();
```

**Why this pattern:**
- Clients are decoupled from the existence and structure of `Department`
- Changing how `Person` finds its manager is now a change in one place

---

## 6. Negative examples — what NOT to do

**Mistake 1: Adding the delegation method but keeping the getter too**
```java
// Not accepted — getDepartment() still exposes the delegate; clients will use both paths
public Department getDepartment() { return department; }  // should be removed
public Person getManager() { return department.getManager(); }
```

**Mistake 2: Hiding everything behind one-liner wrappers**
```java
// Not accepted — if clients need many different pieces of Department,
// creating 10 pass-through methods just shifts complexity without reducing it
// (consider removing middle man instead)
```

**Mistake 3: Not removing the chain after adding the wrapper**
```java
// Not accepted — the method exists but callers still use the old chain
person.getDepartment().getManager(); // still compiles — old habits persist
```

---

## 7. Benefits

- **Encapsulation:** The server's internal structure is hidden from clients
- **Resilience:** Changing the delegate's API only requires changing the server class
- **Simplicity:** Clients express intent (`getManager`) instead of navigating structure
