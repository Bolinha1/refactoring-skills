# SMELL: Temporary Field — Python

## Source
Based on: https://refactoring.guru/smells/temporary-field

---

## 1. What is it?

A class contains a field that is only set and used in certain situations — during one algorithm or method call. The rest of the time it is `None`, empty, or meaningless. Other code that reads the field cannot tell when it holds a valid value and when it doesn't.

---

## 2. Warning signs

- [ ] An attribute is `None` or uninitialized most of the time
- [ ] An attribute is set in one method and consumed in another, but never held across object lifetime
- [ ] An attribute exists only because a long algorithm needed somewhere to put intermediate state
- [ ] The class has a "prepare then execute" pattern where attributes are filled right before use
- [ ] `None` checks for instance attributes appear throughout the class

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Extract Class** | Move the temporary attributes (and the methods that use them) into a dedicated class |
| **Introduce Null Object** | Replace the None/empty state with a proper Null Object so callers don't need to check |

---

## 4. Example

**BEFORE — not accepted:**
```python
# RouteCalculator holds attributes that are only valid during calculate()
class RouteCalculator:
    def __init__(self):
        self.stops = None          # only set during calculation
        self.total_distance = 0    # only valid after calculate()
        self.fastest_path = None   # None unless calculate() ran

    def set_stops(self, stops: list[str]) -> None:
        self.stops = stops

    def calculate(self) -> None:
        # uses self.stops, sets self.total_distance and self.fastest_path
        self.total_distance = self._compute_distance(self.stops)
        self.fastest_path = self._find_fastest_path(self.stops)

    def get_fastest_path(self) -> str | None:
        return self.fastest_path  # None if calculate() wasn't called first
```

**AFTER — expected:**
```python
from dataclasses import dataclass

# Extract the temporary state into a dedicated value object
@dataclass(frozen=True)
class RouteResult:
    total_distance: int
    fastest_path: str


class RouteCalculator:
    def calculate(self, stops: list[str]) -> RouteResult:
        distance = self._compute_distance(stops)
        path = self._find_fastest_path(stops)
        return RouteResult(total_distance=distance, fastest_path=path)
```

**Why this pattern:**
- `RouteResult` only exists when it holds valid data — there is no None state
- `RouteCalculator` is stateless: easy to reuse and test in parallel

---

## 5. Negative examples — what NOT to do

**Mistake 1: Adding an `is_ready` flag instead of fixing the design**
```python
# Not accepted — flag discipline is fragile; Extract Class removes the need for it
self._is_calculated = False

def get_fastest_path(self) -> str:
    if not self._is_calculated:
        raise RuntimeError("Call calculate() first")
    return self.fastest_path
```

**Mistake 2: Moving the temporary attribute to a subclass**
```python
# Not accepted — the attribute still travels with the object; the smell remains
class CalculatingRouteCalculator(RouteCalculator):
    def __init__(self):
        super().__init__()
        self.fastest_path = None  # same problem, just renamed
```

---

## 6. Benefits

- **Correctness:** No more None state masquerading as a valid value
- **Clarity:** The class's attributes are always meaningful regardless of when you look at them
- **Testability:** The extracted class can be tested with valid, fully initialized data
