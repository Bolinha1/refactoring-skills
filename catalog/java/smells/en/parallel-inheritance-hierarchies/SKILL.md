# SMELL: Parallel Inheritance Hierarchies — Java

## Source
Based on: https://refactoring.guru/smells/parallel-inheritance-hierarchies

---

## 1. What is it?

Every time you add a subclass to one class hierarchy, you are forced to add a corresponding subclass to another hierarchy. The two trees mirror each other: adding `PremiumUser` in one tree means adding `PremiumUserReport` in another tree.

---

## 2. Warning signs

- [ ] Class names in two hierarchies share a common prefix or suffix pattern
- [ ] Adding a subclass to hierarchy A always requires a matching subclass in hierarchy B
- [ ] A factory or switch maps types from hierarchy A to types from hierarchy B
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
```java
// Hierarchy A: shapes
abstract class Shape { abstract double area(); }
class Circle extends Shape { double area() { return Math.PI * r * r; } }
class Rectangle extends Shape { double area() { return w * h; } }

// Hierarchy B: renderers — mirrors hierarchy A exactly
abstract class ShapeRenderer { abstract void render(Shape s); }
class CircleRenderer extends ShapeRenderer { void render(Shape s) { /* draw circle */ } }
class RectangleRenderer extends ShapeRenderer { void render(Shape s) { /* draw rect */ } }

// Factory ties them together — adding a new shape always needs a new renderer class too
ShapeRenderer rendererFor(Shape s) {
    if (s instanceof Circle) return new CircleRenderer();
    if (s instanceof Rectangle) return new RectangleRenderer();
    throw new IllegalArgumentException("Unknown shape");
}
```

**AFTER — expected:**
```java
// Merge rendering into the shape hierarchy
abstract class Shape {
    abstract double area();
    abstract void render();   // each shape knows how to render itself
}

class Circle extends Shape {
    double area() { return Math.PI * r * r; }
    void render() { /* draw circle */ }
}

class Rectangle extends Shape {
    double area() { return w * h; }
    void render() { /* draw rect */ }
}

// No renderer hierarchy needed; no factory needed
shapes.forEach(Shape::render);
```

**Why this pattern:**
- Adding `Triangle` only requires one new class, not two
- The factory mapping disappears — every `Shape` renders itself

---

## 5. Negative examples — what NOT to do

**Mistake 1: Keeping both hierarchies and adding a bridge pattern**
```java
// Not accepted — bridge is a valid pattern but here it perpetuates the parallel trees
// instead of collapsing them; only use bridge when variation across two dimensions is
// genuinely independent
```

**Mistake 2: Moving only part of the parallel hierarchy**
```java
// Not accepted — partial collapse leaves an inconsistent design where some shapes
// render themselves and others rely on an external renderer class
```

---

## 6. Benefits

- **Single place to change:** A new shape type lives entirely in one class
- **No synchronisation burden:** The two trees can never fall out of sync because there is only one tree
- **Simpler factory/dispatch:** Polymorphism replaces conditional type-mapping
