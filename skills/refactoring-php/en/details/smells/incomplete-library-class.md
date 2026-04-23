# SMELL: Incomplete Library Class — PHP

## Source
Based on: https://refactoring.guru/smells/incomplete-library-class

---

## 1. What is it?

A library or framework class lacks a method you need, and because you cannot modify the library, you end up placing the missing behaviour in a utility class, a static helper, or a subclass. The logic that belongs to the library leaks into your own code.

---

## 2. Warning signs

- [ ] A utility class contains methods that only exist because a library class is missing them
- [ ] You subclass a library class just to add one or two missing methods
- [ ] Static helper methods receive a library object as their only parameter
- [ ] The same "missing" operations are re-implemented in multiple places across the codebase
- [ ] Comments like "should be in library X" appear next to helper methods

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Introduce Foreign Method** | Add the missing operation as a method in your own class, passing the library object as a parameter |
| **Introduce Local Extension** | Create a subclass or wrapper of the library class that adds all missing methods in one place |

---

## 4. Example

**BEFORE — not accepted:**
```php
use Carbon\Carbon;

// Missing helper scattered as a static method
class DateUtils {
    // Should be on Carbon itself, but we prefer not to monkey-patch
    public static function isWeekend(Carbon $date): bool {
        return $date->isWeekend();
    }

    public static function nextWorkingDay(Carbon $date): Carbon {
        $next = $date->copy()->addDay();
        while ($next->isWeekend()) {
            $next->addDay();
        }
        return $next;
    }
}

// Usage is awkward
$next = DateUtils::nextWorkingDay(Carbon::now());
```

**AFTER — expected:**
```php
use Carbon\Carbon;

// Introduce Local Extension: wrap Carbon with the missing behaviour
class BusinessDate {
    private Carbon $date;

    public function __construct(Carbon $date) {
        $this->date = $date->copy();
    }

    public static function now(): self {
        return new self(Carbon::now());
    }

    public function isWeekend(): bool {
        return $this->date->isWeekend();
    }

    public function nextWorkingDay(): self {
        $next = new self($this->date->copy()->addDay());
        while ($next->isWeekend()) {
            $next = new self($next->date->copy()->addDay());
        }
        return $next;
    }

    public function toCarbon(): Carbon {
        return $this->date->copy();
    }
}

// Usage reads naturally
$next = BusinessDate::now()->nextWorkingDay();
```

**Why this pattern:**
- All business-date logic lives in `BusinessDate` — no scattered static helpers
- Adding more missing behaviour has one obvious place

---

## 5. Negative examples — what NOT to do

**Mistake 1: Duplicating the missing method across multiple utility classes**
```php
// Not accepted — OrderUtils::isWeekend(), ReportUtils::isWeekend(), etc.
// creates divergence when the rule needs to change
```

**Mistake 2: Monkey-patching the library class**
```php
// Not accepted — modifying vendor code directly breaks on library updates
// and causes unexpected behaviour across the whole application
```

---

## 6. Benefits

- **Single source of truth:** Missing behaviour lives in one extension class
- **Readable call sites:** The extension reads like native methods on the original class
- **Isolated from library changes:** Wrapping is safer than monkey-patching when the library evolves
