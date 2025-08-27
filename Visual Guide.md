
# ATdispatcher – Visual Guide

**ATdispatcher** is a Python library for advanced function/method dispatching with multiple options, default arguments, type checking, and automatic handling of instance attributes (`SelfAttr`).

This visual guide shows how the dispatcher decides which function or method variant to call.

---

## 1. Function Dispatch Flow

```
Caller: func(5, 6, 3)
            |
            v
Dispatcher iterates over registered variants
            |
            +-- Variant 1: func(a:int, b:int)          -> Not matched (too many args)
            |
            +-- Variant 2: _(a:int, b:int, c:int)     -> Matched
            |
            +-- Variant 3: _(a:int, b:int, c:int=3)  -> Also matched
            |
Dispatcher selects the best match
            |
            v
Calls: _(a=5, b=6, c=3)
```

✅ Notes:

* Variants are tested in registration order.
* Type hints are checked.
* Default arguments are filled.
* The variant with the highest match is chosen.

---

## 2. Method Dispatch Flow with SelfAttr

Example:

```python
class MyClass:
    def __init__(self):
        self.mult = 2

    @method_dispatcher
    def method(self, x: int, y: int = SelfAttr("mult")):
        return x * y
```

### Call: `obj.method(10)`

```
Caller: obj.method(10)
            |
            v
__get__ descriptor adds `self` -> bound_method(*args)
            |
            v
MethodDispatcher.__call__(self_obj=obj, args=(10,))
            |
            v
Fills parameters:
    x = 10
    y = SelfAttr("mult") -> replaced with obj.mult = 2
            |
            v
Type check:
    x -> int  OK
    y -> int  OK
            |
            v
Call the method:
    method(self=obj, x=10, y=2)
            |
            v
Return: 20
```

---

## 3. Multiple Method Variants

```
@method_dispatcher
def calc(self, x: int):
    return x * 2

@calc.reg()
def _(self, x: int, y: int = SelfAttr("mult")):
    return x * y
```

### Call Examples:

| Call             | Chosen Variant               | Result |
| ---------------- | ---------------------------- | ------ |
| `obj.calc(5)`    | `calc(self, x:int)`          | 10     |
| `obj.calc(5, 4)` | `_(self, x:int, y:int)`      | 20     |
| `obj.calc(5, 3)` | `_(self, x:int, y:SelfAttr)` | 15     |

---

## 4. Dispatcher Decision Diagram

```
                Call func(...)
                      |
               Iterate over variants
                      |
          +-----------+-----------+
          |                       |
      Variant 1               Variant 2 ...
   Check args & types       Check args & types
          |                       |
        Match?                   Match?
          |Yes                     |Yes
          v                         v
     Compute match score      Compute match score
          |                         |
          +-----------+-------------+
                      |
               Choose highest score
                      |
                    Call
```

✅ Notes:

* Match score can be based on exact arguments or more precise type matching.
* SelfAttr values are replaced before type checking.
* Raises `TypeError` if no variant matches.

---

## 5. Summary

* **Functions:** Use `@dispatcher` + `.reg()` for multiple variants.
* **Methods:** Use `@method_dispatcher` + `.reg()` and `SelfAttr` for default instance attributes.
* **Default arguments** and **type hints** determine correct variant.
* **Dispatch flow** is automatic and transparent with visualized decision logic.

