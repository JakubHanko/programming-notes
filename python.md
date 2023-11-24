```table-of-contents
style: nestedList # TOC style (nestedList|inlineFirstLevel)
maxLevel: 0 # Include headings up to the speficied level
includeLinks: true # Make headings clickable
debugInConsole: false # Print debug info in Obsidian console
```

# 1. f-strings

## 1.1. Number formatting

```python
>>> number = 150
>>> print(f"number: {number:.2f}")
number: 150.00

>>> print(f"hex: {number:#0x}")
hex: 0x96

>>> print(f"binary: {number:b}")
binary: 10010110

>>> print(f"octal: {number:o}")
octal: 226

>>> print(f"scientific: {number:e}")
scientific: 1.500000e+02

>>> print(f"scientific: {number:.1e}")
scientific: 1.5e+02

>>> print(f"number: {number:09}")
number: 000000150

>>> ratio = 1 / 2
>>> print(f"percentage = {ratio:.2%}")
percentage = 50.00%

>>> number = 150000
>>> print(f"number = {number:,}")
number = 150,000

>>> print(f"number = {number:_}")
number = 150_000

>>> from math import pi
>>> n = 5
>>> print(f"π to {n} places is {pi:.{n}f}")
π to 5 places is 3.14159
```

## 1.2. Self-documenting expression

```python
>>> a, b = 5, 15
>>> print(f"{a = }")
a = 5

>>> print(f"{a + b = }")
a + b = 20

>>> print(f"{a + b = :.2f}")
a + b = 20.0
```

## 1.3. Date formatting

```python
>>> import datetime

>>> today = datetime.datetime.now()
>>> print(f"datetime : {today}")
datetime : 2023-11-02 20:38:06.749602

>>> print(f"date time: {today:%m/%d/%Y %H:%M:%S}")
date time: 11/02/2023 20:38:06

>>> print(f"date: {today:%m/%d/%Y}")
date: 11/02/2023

>>> print(f"time: {today:%H:%M:%S %p}")
time: 20:38:06 PM
```

## 1.4. String formatting

The string formatting specifier is in this format: `:{chr}{alignment}{N}` where:

* `{chr}` is the character to fill with (default = space)
* `{alignment}` is `<` (left = default), `>` (right) and `^` (center)
* `{N}` number of characters in the resulting string

```python
>>> name = 'deep'
>>> a, b, c = "-", "^", 10

>>> f"{name:10}"
'deep      '
>>> f"{name:<10}"
'deep      '
>>> f"{name:>10}"
'      deep'
>>> f"{name:^10}"
'   deep   '

>>> f"{name:!<10}"
'deep!!!!!!'
>>> f"{name:{a}{b}{c}}"
'---deep---'
```

## 1.5. Other modifiers

* `!a` applies `ascii()`
* `!r` applies `repr()`
* `!s` applies `str()` -- applied by default

```python
class Person:
    def __init__(self, name):
        self.name = name

    def __str__(self):
        return f"I am {self.name}."

    def __repr__(self):
        return f'Person("{self.name}")'

>>> me = Person("Deep")

>>> f"{me}"
'I am Deep.'
>>> f"{me!s}"
'I am Deep.'
>>> f"{me!r}"
'Person("Deep")'

>>> emj = "😊"

>>> f"{emj!a}"
"'\\U0001f60a'"
```

## 1.6. Caveat with logging

In case of logging, the old, printf-style string formatting is preferred.

```python
logger.info('look: %s', untrusted_string)                # OK
logger.info('look: %(foo)s', {'foo', untrusted_string})  # OK
logger.info(f'look: {untrusted_string}')                 # OK (security-wise)
logger.info(f'look: {untrusted_string}', some_dict)      # DANGER!
```

In the fourth case, `untrusted_string` will be interpreted as a string template by the logger. If `untrusted_string` has the value `%(foo)999999999s` (where foo is present in `some_dict`), 
logging gets stuck trying to add over a gigabyte of whitespace to a string. 
In other words: a Denial-of-Service attack.

Moreover, the printf-style formatting is lazy -- it is evaluated the moment it is needed whereas the f-strings are evaluated immediately on creation. This could lead to undesired behaviour in case the logging is set to `warn` and `logging.info( long_complicated_function(x))` is called. `logging.info` should be a no-op.

## 1.7. Sources

* [Reddit - You should know these f-string tricks](https://www.reddit.com/r/Python/comments/17hgu5o/you_should_know_these_fstring_tricks/)
* [Reddit - You should know these f-string tricks (Part 2)](https://www.reddit.com/r/Python/comments/17iaikb/you_should_know_these_fstring_tricks_part_2/)
* [Python - Issue 46200](https://bugs.python.org/issue46200)
