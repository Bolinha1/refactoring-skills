# SKILL: Detecting and Refactoring Divergent Change — Python

## Source
Based on: https://refactoring.guru/smells/divergent-change

---

## 1. What is Divergent Change

A single class is changed for many different reasons. When you find yourself modifying the same class every time a new business rule, a new data source, or a new feature of unrelated concern is added, that class has divergent change.

The inverse of Shotgun Surgery: one class, many reasons to change.

**Why this happens:**
- Unrelated responsibilities were grouped into one class for convenience
- A class started small and grew by accumulation without being split
- "God class" tendencies: one class knows everything and does everything

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] You modify the same class every time a new database table is added
- [ ] You modify the same class every time a new report format is needed
- [ ] The class has methods that fall into clearly distinct conceptual groups
- [ ] The class has imports from many unrelated modules/subsystems
- [ ] Describing what the class does requires the word "and" more than once

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique   |
|----------------------------------------------------------|-------------------------|
| Class handles data access AND business logic             | Extract Class           |
| Class handles multiple unrelated business concepts       | Extract Class           |
| Methods that form a cluster can live elsewhere           | Move Method             |
| All methods relate but the class is too large            | Extract Subclass        |

**Rule of thumb:** each class should have exactly one reason to change.

---

## 4. Example

**BEFORE — not accepted:**
```python
class OrderService:
    # Reason 1: changes when DB schema changes
    def find_by_id(self, order_id: int): ...     # DB query
    def save(self, order): ...                    # DB insert/update

    # Reason 2: changes when business rules change
    def apply_discount(self, order): ...          # discount logic
    def calculate_tax(self, order) -> float: ...  # tax logic

    # Reason 3: changes when report format changes
    def generate_invoice_pdf(self, order) -> bytes: ...  # PDF generation
    def generate_csv_export(self, order) -> str: ...     # CSV export
```

**AFTER — expected:**
```python
class OrderRepository:
    def find_by_id(self, order_id: int): ...
    def save(self, order): ...


class OrderPricingService:
    def apply_discount(self, order): ...
    def calculate_tax(self, order) -> float: ...


class OrderReportService:
    def generate_invoice_pdf(self, order) -> bytes: ...
    def generate_csv_export(self, order) -> str: ...
```

**Why this pattern:**
- Each class has exactly one reason to change
- A new report format only touches `OrderReportService`; a tax rule change only touches `OrderPricingService`

---

## 5. Negative examples — what NOT to do

**Mistake 1: Splitting by layer instead of by responsibility**
```python
# Not accepted — OrderDataLayer still mixes DB schema and business validation
class OrderDataLayer: ...      # DB + validation
class OrderPresentation: ...   # all output formats
```

**Mistake 2: Creating many micro-classes with a single method each**
```python
# Not accepted — over-fragmentation that increases navigation overhead
class OrderFinder: ...
class OrderSaver: ...
class DiscountApplier: ...
```

**Mistake 3: Moving code without moving the data it operates on**
```python
# Not accepted — OrderPricingService must call back into OrderService to get data
# because the attributes it needs were not moved with the methods
```

---

## 6. Benefits

- **Single Responsibility:** Each class has one reason to change
- **Isolation:** Changes to one concern do not risk breaking another
- **Discoverability:** Developers know exactly which class to open for each concern
