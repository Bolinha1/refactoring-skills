# TECHNIQUE: Hide Delegate — PHP

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

- A client calls `$server->getDelegate()->doSomething()`
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
```php
class Person
{
    public function __construct(private Department $department) {}

    public function getDepartment(): Department
    {
        return $this->department; // exposed — clients navigate through it
    }
}

class Department
{
    public function __construct(private Person $manager) {}

    public function getManager(): Person
    {
        return $this->manager;
    }
}

// Client code — must know about both Person and Department
$manager = $person->getDepartment()->getManager();
```

**AFTER — expected:**
```php
class Person
{
    public function __construct(private Department $department) {}

    public function getManager(): Person
    {
        return $this->department->getManager(); // delegates, hides Department from client
    }
    // getDepartment() can be removed if no one else needs it
}

// Client code — only knows about Person
$manager = $person->getManager();
```

**Why this pattern:**
- Clients are decoupled from the existence and structure of `Department`
- Changing how `Person` finds its manager is now a change in one place

---

## 6. Negative examples — what NOT to do

**Mistake 1: Adding the delegation method but keeping the getter too**
```php
// Not accepted — getDepartment() still exposes the delegate; clients will use both paths
public function getDepartment(): Department { return $this->department; }  // should be removed
public function getManager(): Person { return $this->department->getManager(); }
```

**Mistake 2: Creating many pass-through methods for everything in the delegate**
```php
// Not accepted — if clients need many different pieces of Department,
// 10 pass-through methods just shift complexity without reducing it
// (consider Remove Middle Man instead)
```

**Mistake 3: Not removing the chain after adding the wrapper**
```php
// Not accepted — the method exists but callers still use the old chain
$person->getDepartment()->getManager(); // still compiles — old habits persist
```

---

## 7. Benefits

- **Encapsulation:** The server's internal structure is hidden from clients
- **Resilience:** Changing the delegate's API only requires changing the server class
- **Simplicity:** Clients express intent (`getManager()`) instead of navigating structure
