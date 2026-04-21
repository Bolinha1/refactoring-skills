# TECHNIQUE: Hide Delegate — Python

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

- A client calls `server.get_delegate().do_something()`
- A change in the delegate class requires changes in all clients that navigate to it
- The delegation chain creates Message Chains smell
- You want clients to depend only on the server's public interface, not on its internal structure

---

## 4. Refactoring steps

1. For each method of the delegate that clients call through the server:
   - Create a delegating method on the server class
2. Update the client to call the new server method instead of navigating the chain
3. If no client accesses the delegate directly anymore, remove the attribute/property for the delegate
4. Run tests

---

## 5. Example

**BEFORE — not accepted:**
```python
class Person:
    def __init__(self, department: "Department"):
        self.department = department  # exposed — clients navigate through it


class Department:
    def __init__(self, manager: "Person"):
        self.manager = manager


# Client code — must know about both Person and Department
manager = person.department.manager
```

**AFTER — expected:**
```python
class Person:
    def __init__(self, department: "Department"):
        self._department = department  # private

    @property
    def manager(self) -> "Person":
        return self._department.manager  # delegates, hides Department from client


# Client code — only knows about Person
manager = person.manager
```

**Why this pattern:**
- Clients are decoupled from the existence and structure of `Department`
- Changing how `Person` finds its manager is now a change in one place

---

## 6. Negative examples — what NOT to do

**Mistake 1: Adding the delegation property but keeping the public attribute too**
```python
# Not accepted — department is still public; clients will use both paths
class Person:
    def __init__(self, department):
        self.department = department  # still public — should be private
    
    @property
    def manager(self):
        return self.department.manager
```

**Mistake 2: Creating many pass-through properties for everything in the delegate**
```python
# Not accepted — if clients need many different pieces of Department,
# 10 pass-through properties just shift complexity without reducing it
# (consider Remove Middle Man instead)
```

**Mistake 3: Not removing the chain after adding the wrapper**
```python
# Not accepted — the property exists but callers still use the old chain
person.department.manager  # still works — old habits persist
```

---

## 7. Benefits

- **Encapsulation:** The server's internal structure is hidden from clients
- **Resilience:** Changing the delegate's API only requires changing the server class
- **Simplicity:** Clients express intent (`person.manager`) instead of navigating structure
