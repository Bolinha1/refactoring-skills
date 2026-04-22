# SMELL: Temporary Field — PHP

## Source
Based on: https://refactoring.guru/smells/temporary-field

---

## 1. What is it?

A class contains a field that is only set and used in certain situations — during one algorithm or method call. The rest of the time it is `null`, empty, or meaningless. Other code that reads the field cannot tell when it holds a valid value and when it doesn't.

---

## 2. Warning signs

- [ ] A field is `null` or uninitialized most of the time
- [ ] A field is set in one method and consumed in another, but never held across object lifetime
- [ ] A field exists only because a long algorithm needed somewhere to put intermediate state
- [ ] The class has a "prepare then execute" pattern where fields are filled right before use
- [ ] Null checks for instance fields appear throughout the class

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Extract Class** | Move the temporary fields (and the methods that use them) into a dedicated class |
| **Introduce Null Object** | Replace the null/empty state with a proper Null Object so callers don't need to check |

---

## 4. Example

**BEFORE — not accepted:**
```php
// RouteCalculator holds fields that are only valid during calculateRoute()
class RouteCalculator {
    private ?array $stops = null;       // only set during calculation
    private int $totalDistance = 0;     // only valid after calculate()
    private ?string $fastestPath = null; // null unless calculate() ran

    public function setStops(array $stops): void {
        $this->stops = $stops;
    }

    public function calculate(): void {
        // uses $this->stops, sets $this->totalDistance and $this->fastestPath
        $this->totalDistance = $this->computeDistance($this->stops);
        $this->fastestPath = $this->findFastestPath($this->stops);
    }

    public function getFastestPath(): ?string {
        return $this->fastestPath; // null if calculate() wasn't called first
    }
}
```

**AFTER — expected:**
```php
// Extract the temporary state into a dedicated value object
class RouteResult {
    public function __construct(
        private readonly int $totalDistance,
        private readonly string $fastestPath
    ) {}

    public function getTotalDistance(): int { return $this->totalDistance; }
    public function getFastestPath(): string { return $this->fastestPath; }
}

class RouteCalculator {
    public function calculate(array $stops): RouteResult {
        $distance = $this->computeDistance($stops);
        $path = $this->findFastestPath($stops);
        return new RouteResult($distance, $path);
    }
}
```

**Why this pattern:**
- `RouteResult` only exists when it holds valid data — there is no null state
- `RouteCalculator` is stateless: easy to reuse and test in parallel

---

## 5. Negative examples — what NOT to do

**Mistake 1: Adding an `isReady` flag instead of fixing the design**
```php
// Not accepted — flag discipline is fragile; Extract Class removes the need for it
private bool $isCalculated = false;
public function getFastestPath(): string {
    if (!$this->isCalculated) {
        throw new \RuntimeException('Call calculate() first');
    }
    return $this->fastestPath;
}
```

**Mistake 2: Moving the temporary field to a subclass**
```php
// Not accepted — the field still travels with the object; the smell remains
class CalculatingRouteCalculator extends RouteCalculator {
    private ?string $fastestPath = null; // same problem, just renamed
}
```

---

## 6. Benefits

- **Correctness:** No more null state masquerading as a valid value
- **Clarity:** The class's fields are always meaningful regardless of when you look at them
- **Testability:** The extracted class can be tested with valid, fully initialized data
