# TECHNIQUE: Extract Class — Java

## Source
Based on: https://refactoring.guru/extract-class

---

## 1. Problem

One class does the work of two. A subset of its fields and methods forms a coherent concept that would make sense as its own class.

---

## 2. Solution

Create a new class and move the fields and methods that belong to that concept into it. Replace the original fields with a reference to the new class.

---

## 3. When to apply

- The class has fields that are always used together (Data Clumps)
- The class has methods that only touch a subset of its fields
- The class has grown to handle more than one distinct responsibility (Divergent Change)
- A subset of the class could be reused independently in another context
- The class name cannot describe what it does without using "and"

---

## 4. Refactoring steps

1. Identify the group of fields and methods to extract — they must form a coherent concept
2. Create a new class with a name that expresses that concept
3. Create an instance of the new class as a field in the original class
4. Move the identified fields to the new class (use Move Field for each)
5. Move the identified methods to the new class (use Move Method for each)
6. Decide on the visibility of the new class:
   - Make it public if it can stand alone or be reused
   - Keep it package-private if it is an internal detail
7. Compile and run tests after each move

---

## 5. Example

**BEFORE — not accepted:**
```java
public class Person {
    private String name;
    private String officeAreaCode;
    private String officeNumber;

    public String getTelephoneNumber() {
        return "(" + officeAreaCode + ") " + officeNumber;
    }
}
```

**AFTER — expected:**
```java
public class TelephoneNumber {
    private final String areaCode;
    private final String number;

    public TelephoneNumber(String areaCode, String number) {
        this.areaCode = areaCode;
        this.number = number;
    }

    public String format() {
        return "(" + areaCode + ") " + number;
    }
}

public class Person {
    private String name;
    private TelephoneNumber officeTelephone;

    public String getTelephoneNumber() {
        return officeTelephone.format();
    }
}
```

**Variant — extracting a data clump:**
```java
// BEFORE — address fields scattered across Person and Invoice
public class Person {
    private String street;
    private String city;
    private String zipCode;
}

// AFTER — Address is an independent, reusable concept
public class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    // constructor, getters, validation, formatting
}

public class Person {
    private Address homeAddress;
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting a class that has no cohesion**
```java
// Not accepted — PersonData groups fields that have no natural relationship
public class PersonData {
    private String name;
    private String officeCode;
    private double salary;
    private Date hireDate;
}
```

**Mistake 2: Moving methods without moving the fields they need**
```java
// Not accepted — TelephoneNumber.format() still reads areaCode from Person
public class TelephoneNumber {
    public String format(String areaCode, String number) { ... } // still coupled
}
```

**Mistake 3: Extracting but leaving the original fields in place**
```java
// Not accepted — now there are two sources of truth for telephone data
public class Person {
    private TelephoneNumber telephone; // new
    private String officeAreaCode;     // old — should be deleted
    private String officeNumber;       // old — should be deleted
}
```

---

## 7. Benefits

- **Single Responsibility:** Each class has one clear concept to represent
- **Reuse:** The extracted class can be used by other classes independently
- **Encapsulation:** Validation and formatting logic for the concept live in one place
