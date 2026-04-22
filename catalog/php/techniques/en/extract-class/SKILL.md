# TECHNIQUE: Extract Class — PHP

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
6. Decide on visibility:
   - Make it `public` if it can stand alone or be reused
   - Keep it internal if it is a private detail of the original class
7. Compile and run tests after each move

---

## 5. Example

**BEFORE — not accepted:**
```php
class Person
{
    private string $name;
    private string $officeAreaCode;
    private string $officeNumber;

    public function getTelephoneNumber(): string
    {
        return "({$this->officeAreaCode}) {$this->officeNumber}";
    }
}
```

**AFTER — expected:**
```php
final class TelephoneNumber
{
    public function __construct(
        private readonly string $areaCode,
        private readonly string $number,
    ) {}

    public function format(): string
    {
        return "({$this->areaCode}) {$this->number}";
    }
}

class Person
{
    public function __construct(
        private string $name,
        private TelephoneNumber $officeTelephone,
    ) {}

    public function getTelephoneNumber(): string
    {
        return $this->officeTelephone->format();
    }
}
```

**Variant — extracting a data clump:**
```php
// BEFORE — address fields scattered across Person and Invoice
class Person
{
    private string $street;
    private string $city;
    private string $zipCode;
}

// AFTER — Address is an independent, reusable concept
final class Address
{
    public function __construct(
        public readonly string $street,
        public readonly string $city,
        public readonly string $zipCode,
    ) {}

    public function format(): string
    {
        return "{$this->street}, {$this->city} {$this->zipCode}";
    }
}

class Person
{
    public function __construct(private Address $homeAddress) {}
}
```

---

## 6. Negative examples — what NOT to do

**Mistake 1: Extracting a class that has no cohesion**
```php
// Not accepted — PersonData groups fields that have no natural relationship
class PersonData
{
    public string $name;
    public string $officeCode;
    public float $salary;
    public \DateTime $hireDate;
}
```

**Mistake 2: Moving methods without moving the fields they need**
```php
// Not accepted — TelephoneNumber::format() still receives raw fields from Person
class TelephoneNumber
{
    public function format(string $areaCode, string $number): string { ... } // still coupled
}
```

**Mistake 3: Extracting but leaving the original fields in place**
```php
// Not accepted — now there are two sources of truth for telephone data
class Person
{
    private TelephoneNumber $telephone; // new
    private string $officeAreaCode;     // old — should be deleted
    private string $officeNumber;       // old — should be deleted
}
```

---

## 7. Benefits

- **Single Responsibility:** Each class has one clear concept to represent
- **Reuse:** The extracted class can be used by other classes independently
- **Encapsulation:** Validation and formatting logic for the concept live in one place
