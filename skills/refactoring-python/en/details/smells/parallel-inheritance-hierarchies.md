# SMELL: Parallel Inheritance Hierarchies — Python

## Source
Based on: https://refactoring.guru/smells/parallel-inheritance-hierarchies

---

## 1. What is it?

Every time you add a subclass to one class hierarchy, you are forced to add a corresponding subclass to another hierarchy. The two trees mirror each other: adding `PremiumUser` in one tree means adding `PremiumUserReport` in another tree.

---

## 2. Warning signs

- [ ] Class names in two hierarchies share a common prefix or suffix pattern
- [ ] Adding a subclass to hierarchy A always requires a matching subclass in hierarchy B
- [ ] A factory or dispatch dict maps types from hierarchy A to types from hierarchy B
- [ ] Both hierarchies grow in lockstep with each new feature
- [ ] One hierarchy holds "entities" while the other holds "behaviour" or "representation" — but they must always match

---

## 3. Treatment techniques

| Technique | When to use |
|---|---|
| **Move Method** | Move behaviour from the parallel hierarchy into the main hierarchy, eliminating the need for the second tree |
| **Move Field** | Move data that lives in the parallel class back into the primary class |
| **Inline Class** | Collapse a class from the parallel hierarchy into its corresponding primary class when the parallel class is small |

---

## 4. Example

**BEFORE — not accepted:**
```python
import math

# Hierarchy A: shapes
class Shape:
    def area(self) -> float: ...

class Circle(Shape):
    def area(self) -> float: return math.pi * self.r ** 2

class Rectangle(Shape):
    def area(self) -> float: return self.w * self.h


# Hierarchy B: renderers — mirrors hierarchy A exactly
class ShapeRenderer:
    def render(self, shape: Shape) -> None: ...

class CircleRenderer(ShapeRenderer):
    def render(self, shape: Shape) -> None: pass  # draw circle

class RectangleRenderer(ShapeRenderer):
    def render(self, shape: Shape) -> None: pass  # draw rect


# Factory ties them — adding a new shape always needs a new renderer class too
_RENDERERS = {Circle: CircleRenderer, Rectangle: RectangleRenderer}

def renderer_for(shape: Shape) -> ShapeRenderer:
    return _RENDERERS[type(shape)]()
```

**AFTER — expected:**
```python
import math
from abc import ABC, abstractmethod

# Merge rendering into the shape hierarchy
class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def render(self) -> None: ...  # each shape renders itself


class Circle(Shape):
    def area(self) -> float: return math.pi * self.r ** 2
    def render(self) -> None: pass  # draw circle


class Rectangle(Shape):
    def area(self) -> float: return self.w * self.h
    def render(self) -> None: pass  # draw rect


# No renderer hierarchy needed; no factory needed
for shape in shapes:
    shape.render()
```

**Why this pattern:**
- Adding `Triangle` only requires one new class, not two
- The factory mapping disappears — every `Shape` renders itself

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping both hierarchies and using a registry dict**
```python
# Not accepted — a registry that maps Shape → Renderer is just the factory in disguise;
# the parallel hierarchy problem remains
RENDERERS: dict[type, type] = {Circle: CircleRenderer, Rectangle: RectangleRenderer}
```

**Mistake 2: Moving only part of the parallel hierarchy**
```python
# Not accepted — partial collapse leaves an inconsistent design where some shapes
# render themselves and others still rely on external renderer classes
```

---

## 6. Benefits

- **Single place to change:** A new shape type lives entirely in one class
- **No synchronisation burden:** The two trees can never fall out of sync because there is only one tree
- **Simpler dispatch:** Polymorphism replaces conditional type-mapping
