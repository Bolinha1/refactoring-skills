# SKILL: Detecting and Refactoring Long Parameter List — Python

## Source
Based on: https://refactoring.guru/smells/long-parameter-list

---

## 1. What is Long Parameter List

A function or method has too many parameters — typically more than three or four. Long parameter lists are hard to understand, easy to confuse, and painful to call. They often indicate that related data should be grouped into a dataclass or object.

**Why this happens:**
- Functions were merged without creating a proper abstraction for the combined data
- Algorithms became more complex over time, requiring more control flags
- Data that naturally belongs together is passed as individual primitives

---

## 2. Warning signs (triggers for this SKILL)

Activate this SKILL when you identify any of the items below:

- [ ] Function/method signature with 4 or more parameters (excluding `self`)
- [ ] Several parameters of the same type in a row (easy to swap accidentally)
- [ ] Parameters that always appear together across multiple call sites
- [ ] Boolean "flag" parameters that switch the function's behavior
- [ ] Caller must construct many locals just to call the function

---

## 3. Treatment techniques (in order of preference)

| Situation found                                          | Recommended technique              |
|----------------------------------------------------------|------------------------------------|
| Parameters are all fields of an existing object          | Preserve Whole Object              |
| Parameters represent a new concept not yet in the model  | Introduce Parameter Object         |
| A parameter can be computed from other parameters        | Replace Parameter with Method Call |
| A boolean flag changes the function behavior             | Split into two explicit functions  |

---

## 4. Example

**BEFORE — not accepted:**
```python
def create_order(
    customer_id: str,
    customer_name: str,
    customer_email: str,
    street: str,
    city: str,
    zip_code: str,
    country: str,
    product_ids: list[str],
    coupon_code: str | None = None,
) -> Receipt:
    ...
```

**AFTER — expected:**
```python
from dataclasses import dataclass

@dataclass
class Address:
    street: str
    city: str
    zip_code: str
    country: str

@dataclass
class OrderRequest:
    customer_id: str
    customer_name: str
    customer_email: str
    shipping_address: Address
    product_ids: list[str]
    coupon_code: str | None = None


def create_order(request: OrderRequest) -> Receipt:
    # logic uses request.customer_id, request.shipping_address, etc.
    ...
```

**Why this pattern:**
- `OrderRequest` and `Address` are first-class concepts — they can be validated, reused, and tested independently
- Callers construct a meaningful object, not a positional argument list

---

## 5. Negative examples — what NOT to do

**Mistake 1: Grouping unrelated parameters into a dataclass just to reduce count**
```python
# Not accepted — DataHolder has no domain meaning
@dataclass
class DataHolder:
    a: str
    b: str
    c: str
    ...
```

**Mistake 2: Using a dict as the "parameter object"**
```python
# Not accepted — loses type safety, IDE support, and discoverability
def create_order(params: dict) -> Receipt: ...
```

**Mistake 3: Exploiting *args/**kwargs to hide the long list**
```python
# Not accepted — makes the API invisible and untypeable
def create_order(*args, **kwargs) -> Receipt: ...
```

---

## 6. Benefits

- **Readability:** Named parameter objects make call sites self-documenting
- **Safety:** Positional argument confusion is eliminated
- **Extensibility:** Adding a field to the parameter object does not change every call site
