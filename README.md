
# ATdispatcher

**ATdispatcher** is a flexible Python dispatcher library for creating functions and methods with **multiple options** (overloads), **default arguments**, **type checking**, and **automatic handling of instance attributes** for methods. It allows a simple API to manage multiple variations of the same function or method.

---

## Features

* Dispatch multiple **function variations** under the same name.
* Support for **default arguments**.
* Support for **type hints** to select the correct variant.
* Dispatch **methods** with automatic `self` and `SelfAttr` handling.
* Simple API with `@dispatcher` for functions and `@method_dispatcher` for methods.
* Easily extendable with multiple registrations using `.reg()`.

---

## Installation

Currently, ATdispatcher is a standalone module. You can include it in your project directly:

```bash
git clone https://github.com/avitwil/ATdispatcher.git
or
pip install ATdispatcher
```

Or copy `ATdispatcher.py` into your project directory.

---

## Usage

### Importing

```python
from ATdispatcher import dispatcher, method_dispatcher, SelfAttr
```

---

### 1. Function Dispatching

```python
@dispatcher
def func(a: int, b: int):
    return a + b

@func.reg()
def _(a: int, b: int, c: int):
    return a + b + c

@func.reg()
def _(a: int, b: int, c: int = 3):
    return a * b * c

print(func(5, 6))        # Output: 11  (matches the 1st variant: a + b)
print(func(5, 6, 7))     # Output: 18  (matches the 2nd variant: a + b + c)
print(func(5, 6, 3))     # Output: 14  (matches the 2nd variant again, NOT the 3rd)
```

⚠️ **Dispatch order matters — this is first-match-wins, not most-specific-match-wins.**

`FuncDispatcher` tries variants **in registration order** (the function passed to
`@dispatcher` first, then each `@func.reg()` in the order it was applied) and calls
the *first* one whose signature can bind the given arguments — it does not look
ahead for a "better" or more specific match further down the list.

In the example above, `func(5, 6, 3)` successfully binds against the 2nd variant
`(a, b, c: int)` with `c=3`, so that variant runs and returns `5 + 6 + 3 == 14`. The
3rd variant, `(a, b, c: int = 3)`, is never reached even though it also matches,
because the search already stopped at the 2nd one. If you need a more specific
variant to win, register it *before* any earlier, more permissive variant that would
also match the same call.

✅ Notes:

* You can register multiple variants with different signatures using `.reg()`.
* Type hints are used to select the correct variant.
* Default arguments are supported.
* Variants are tried in registration order; the first one that matches wins, even if
  a later variant would also match.

---

### 2. Method Dispatching with `SelfAttr`

`SelfAttr` allows method defaults to refer to instance attributes automatically.

```python
class MyClass:
    def __init__(self):
        self.mult = 2

    @method_dispatcher
    def method(self, x: int, y: int = SelfAttr("mult")):
        return x * y

obj = MyClass()

print(obj.method(10))     # Output: 20  (y defaults to obj.mult)
print(obj.method(10, 5))  # Output: 50  (y explicitly passed)
```

✅ Notes:

* `SelfAttr("attr_name")` automatically fetches `self.attr_name` as the default.
* Works with multiple method registrations using `.reg()` as well.

---

### 3. Multiple Method Variants

```python
class MyClass:
    def __init__(self):
        self.mult = 3

    @method_dispatcher
    def calc(self, x: int):
        return x * 2

    @calc.reg()
    def _(self, x: int, y: int = SelfAttr("mult")):
        return x * y

obj = MyClass()

print(obj.calc(5))        # Output: 10  (first variant: x * 2)
print(obj.calc(5, 4))     # Output: 10  (still the first variant!)
print(obj.calc(5, 3))     # Output: 10  (still the first variant!)
```

⚠️ As with `FuncDispatcher` (see above), dispatch is first-match-wins: `calc(self,
x: int)` only declares one positional parameter after `self`, so `MethodDispatcher`
resolves `x` and stops there — any extra positional argument (the `4` or `3` above)
is simply never consumed or checked, and the call still counts as a match against
this variant. Because of that, the second variant, `_(self, x: int, y: int =
SelfAttr("mult"))`, is **unreachable** in this particular example: there is no call
that fails to match the first variant's single required `x` parameter while still
providing a valid `x`, so the search never continues to the second one.

If you want a second variant to actually be reachable, don't register a shorter,
"looser" variant ahead of a longer one that only adds optional/defaulted
parameters — register the more specific/complete variant first, and make sure any
earlier variant's required parameters can't be satisfied by calls meant for a later
variant (e.g. give them genuinely different required arities or incompatible type
hints).

---

### 4. Error Handling

If no matching signature is found:

```python
try:
    func("a", 5)
except TypeError as e:
    print(e)  # Output: No matching function signature found
```

✅ Type checking ensures invalid calls raise `TypeError`.

---

### 5. API Reference

| Class / Function          | Description                                                         |
| ------------------------- | ------------------------------------------------------------------- |
| `dispatcher(func)`        | Create a function dispatcher.                                       |
| `.reg()`                  | Register a new variant for the same dispatcher.                     |
| `method_dispatcher(func)` | Create a method dispatcher for instance methods.                    |
| `SelfAttr("attr")`        | Used for method default arguments that reference `self` attributes. |

---

### 6. Example: Combined Usage

```python
from ATdispatcher import dispatcher, method_dispatcher, SelfAttr

@dispatcher
def add(a: int, b: int):
    return a + b

@add.reg()
def _(a: int, b: int, c: int = 10):
    return a + b + c

class Calc:
    def __init__(self):
        self.multiplier = 5

    @method_dispatcher
    def multiply(self, x: int, y: int = SelfAttr("multiplier")):
        return x * y

calc = Calc()

print(add(1, 2))          # 3
print(add(1, 2, 3))       # 6
print(calc.multiply(4))   # 20
print(calc.multiply(4, 2))# 8
```

---

### License

MIT License – free to use, modify, and distribute.
