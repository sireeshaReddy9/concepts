# Python Programming Guide

A comprehensive guide covering Python fundamentals, data structures, OOP concepts, and advanced topics.

---

## Table of Contents
- [Introduction to Python](#introduction-to-python)
- [Python Basics](#python-basics)
- [Data Types](#data-types)
- [Functions](#functions)
- [Strings](#strings)
- [Regular Expressions](#regular-expressions)
- [Lists](#lists)
- [Tuples](#tuples)
- [Sets](#sets)
- [Dictionaries](#dictionaries)
- [Object-Oriented Programming](#object-oriented-programming)
- [Exception Handling](#exception-handling)

---

## Introduction to Python

Python is an **interpreted, high-level programming language** with the following characteristics:

- Supports multiple programming paradigms:
  - Procedural
  - Object-oriented
  - Functional programming
- Concise and expressive
- Requires less time, effort, and lines of code

### Compiler vs Interpreter

| Compiler | Interpreter |
|----------|-------------|
| Takes entire program as input | Takes single instruction as input |
| Generates intermediate object code (requires more memory) | No intermediate code |
| Displays errors after entire program is checked | Displays errors line by line |

**Python Execution Flow:**
```
Source File → Interpreter → Machine Language
```

### Interactive vs Script Mode

| Interactive Mode | Script Mode |
|------------------|-------------|
| Type commands at prompt | Read and execute from script file |
| Cannot save and edit code | Can save and edit code |

---

## Python Basics

### Data Types

Data types represent real-world data in bits.

**Mutable Types:**
- `list`
- `dict`
- `set`

**Immutable Types:**
- `tuple`
- `int`
- `bool`
- `str`

### Type Casting

Converting one type to another:

```python
int(), str(), set(), hex(), oct(), chr(), ord(), tuple(), float()
```

**Important Notes:**
- When modifying immutable types, a new object is created
- When two variables have the same value, both point to the same object (for immutables)

---

## Functions

A function is a block of code that takes input and performs operations.

```python
def function_name():
    # code here
    return something
```

**Benefits:**
- Modularity
- Reusability

### Types of Functions

1. **User-defined functions:** Defined by the user
2. **Built-in functions:** Already defined in Python libraries
3. **Lambda functions:** Can take any number of arguments but only one expression
4. **Recursive functions:** Functions that call themselves

### Lambda Functions

```python
# Syntax
lambda arguments: expression

# Example
square = lambda x: x**2
result = square(5)  # 25
```

**Common uses with higher-order functions:**

```python
# filter
filter(lambda x: (x % 2 == 0), lst)

# reduce
from functools import reduce
reduce(lambda x, y: x + y, lst)

# map
map(lambda x: x**2, lst)
```

### Function Arguments

1. **Positional Arguments:** Order matters
2. **Default Arguments:**
   ```python
   def add(a, b=20):
       return a + b
   ```

3. **Keyword Arguments:**
   ```python
   def add(a, b):
       return a + b
   add(b=10, a=20)
   ```

4. **Variable Length Arguments:**
   ```python
   def fun_name(*args):
       # args is a tuple
       pass
   ```

5. **Variable Keyword Arguments:**
   ```python
   def data(**kwargs):
       # kwargs is a dict
       pass
   data(name='siri', age=22, gender='F')
   ```

### Modules and Scripts

**Module:** Collection of functions that can be imported

```python
from mymodule import *
import mymodule as mm
```

**Script:** File executed in command line

- `__name__` contains `__main__` in script module
- If imported in other file, `__name__` will be the file name

### User Input

```python
num = int(input('Enter number\n'))
result = eval(expression)  # Evaluates string as Python expression
```

### Print Function

```python
print(*values, sep=' ', end='\n', file=file, flush=flush)
```

**Falsy Values:**
- `False`, `None`, `0`
- Empty sequences: `''`, `()`, `[]`
- Empty mapping: `{}`

### Range Function

```python
range(start, stop, step)
```

### Variable Scope

```python
# Global variable
global_var = 10

def function():
    # Local variable
    local_var = 20
    
    # Nonlocal (for nested functions)
    nonlocal outer_var
    
    # Built-in functions
    globals()  # Returns dict of global variables
    locals()   # Returns dict of local variables
```

---

## Strings

### Escape Character

`\` - Anything after this is considered part of the string

```python
path = "C:\\Users\\Documents"
```

### String Slicing

```python
# Syntax: string_name[start : stop+1 : step]
s = "Hello World"
s[0:5]    # "Hello"
s[::2]    # "HloWrd"
s[::-1]   # "dlroW olleH" (reverse)
```

### String Comparisons

```python
# Compare values
"hello" == "hello"  # True

# Compare references
id(str1) == id(str2)
str1 is str2
```

### String Methods

```python
s.lower()
s.upper()
s.startswith('https')
s.endswith('com')
s.swapcase()
s.title()       # Makes first letter of each word capital
s.capitalize()  # Makes only first letter capital
```

### String Operations

```python
# Concatenation
s1 + s2

# Replication
s * 3

# Join
"".join(iterable)
```

### String Translation

```python
tab = s.maketrans('aeiou', 'AEIOU', '0123456789')
s.translate(tab)
```

### String Formatting

**1. format() method:**
```python
s = "Hi {}, how is weather in {}".format(name, place)
s = "{} {} {}".format(10, 20, 30)
s = "{2} {0} {1}".format(10, 20, 30)  # Positional

# Alignment
s = "{0:>10}".format(999)  # Right align: "       999"
s = "{0:<10}".format(999)  # Left align
s = "{0:^10}".format(999)  # Center align

# List indexing
s = "{0[0]} {0[1]} {0[2]}".format([10, 20, 30])
```

**2. f-strings (Formatted String Literals):**
```python
# Less prone to errors and faster
f"{name} {place}"
f"{math.pi:.4f}"
```

**3. Raw String Literals:**
```python
name = r"Ro\nhit"  # Output: Ro\nhit (not interpreted)
```

---

## Regular Expressions

Sequence of characters that forms a search pattern.

```python
import re

text = 'python is easy'
regex = r"python"
match = re.match(regex, text)

if match:
    print(match)  # <re.Match object; span=(0,6) match='python'>
    print(text[match.start():match.end()])
```

### Key Functions

- **`match()`:** Searches from beginning only
- **`search()`:** Searches entire string, returns first match
- **`findall()`:** Returns all matches

### Regex Patterns

| Pattern | Description |
|---------|-------------|
| `\` | Escape meta character |
| `\|` | OR operator: `r"python\|easy"` |
| `*` | 0 or more occurrences |
| `?` | 0 or 1 occurrence |
| `+` | 1 or more occurrences |
| `{m,n}` | m to n occurrences: `r"go{2,5}gle"` |
| `^` | Match at start: `r"^python"` |
| `$` | Match at end: `r"python$"` |

### Character Classes

```python
[abc]        # Match any of a, b, or c
[^abc]       # Match anything except a, b, or c
[a-zA-Z0-9]  # All letters and digits

\w           # Word character [a-zA-Z0-9_]
\W           # Non-word character
\d           # Digit [0-9]
\D           # Non-digit
\s           # Whitespace character
\S           # Non-whitespace character
\b           # Word boundary
\B           # Non-word boundary
```

### Word Boundary Examples

```python
text = 'abcdefsyvhs'

# Matches 'abc' at start
regex = r'\babc'

# Matches 'abc' with characters on both sides
regex = r'\Babc\B'
```

---

## Lists

Lists follow **referential array format**:
- Store heterogeneous data
- Can be resized
- Mutable

### List Operations

```python
# Concatenation
lst3 = lst1 + lst2

# Replication
lst = [0] * 10

# Modification
lst[0] = 10
lst[1:4] = [99]
```

### Built-in Functions

```python
max(lst)
min(lst)
len(lst)
sum(lst)
sorted(lst)
any(lst)
all(lst)
range(start, stop, step)
enumerate(lst)
list(iterable)
```

### List Methods

```python
lst.insert(index, element)  # Insert at index
lst.append(element)         # Add to end
lst.extend([1, 2, 3])       # Add multiple elements
lst.pop()                   # Remove and return last
lst.pop(index)              # Remove at index
lst.remove(value)           # Remove first occurrence
lst.count(value)            # Count occurrences
lst.reverse()               # Reverse in place
lst.index(value, start, end) # Find index
lst.clear()                 # Remove all elements
lst.sort()                  # Sort in place
```

**Note:** 
- `sort()` modifies the list in place (efficient)
- `sorted()` creates a new list (memory overhead)
- `reversed(lst)` returns an iterator
- `lst.reverse()` is more efficient

### Deleting Elements

```python
del lst[3]         # Delete at index
del lst[1:3]       # Delete slice
del lst[::2]       # Delete every other element
```

### Copying Lists

**Shallow Copy:**
```python
lst2 = lst1[:]
lst2 = list(lst1)
```

**Deep Copy (for nested lists):**
```python
import copy
lst2 = copy.deepcopy(lst1)
```

**Shallow vs Deep Copy:**
- **Shallow copy:** Creates new list but inner objects are references
- **Deep copy:** Creates new list with copies of all nested objects

### List Comprehension

```python
# Basic syntax
[expression for item in iterable if condition]

# Example: Even numbers
result = [n for n in nums if n % 2 == 0]

# With if-else
result = ["even" if n % 2 == 0 else "odd" for n in nums]

# Structure with if-else
[expression_if_true if condition else expression_if_false for item in list]
```

### zip() Function

Ties elements from multiple lists together, position by position.

```python
names = ["Ram", "Sita", "Laxman"]
marks = [85, 90, 78]

result = list(zip(names, marks))
# [('Ram', 85), ('Sita', 90), ('Laxman', 78)]

# Unzipping
pairs = [(1, 10), (2, 20)]
a, b = zip(*pairs)
# a = (1, 2), b = (10, 20)
```

### Time Complexity

| Operation | Complexity |
|-----------|------------|
| Access `arr[index]` | O(1) |
| Append at end | O(1) |
| Insert | O(n) |
| Extend | O(n²) |
| Pop (last) | O(1) |
| Pop (first) | O(n) |
| Remove | O(n) |
| Reverse | O(n) |
| Index | O(1) |
| len() | O(1) |

---

## Tuples

Tuples are **immutable** and use **static arrays** internally.

```python
# Creating tuples
t = tuple()
t = ()
t = (1,)        # Singleton tuple (comma required)
t = 1, 2, 3     # Tuple packing
```

**Key Difference from Lists:**
- Cannot grow or shrink in size
- More memory efficient
- Faster than lists

---

## Sets

**Unordered and unindexed** collection that doesn't allow duplicates. Uses **hashing technique**.

**Why sets can't store mutable types:**
- Hash key is generated from data
- If data changes, hash changes
- Cannot have two hash keys for same object

### Set Methods

```python
s.add(99)
s.remove(99)     # Raises error if not present
s.discard(99)    # No error if not present
```

### Set Operations

```python
# Union
s1 | s2
s1.union(s2)

# Intersection
s1 & s2
s1.intersection(s2)

# Difference
s1 - s2          # Elements in s1 but not in s2

# Symmetric Difference
s1 ^ s2
s1.symmetric_difference(s2)  # Elements in either but not both
```

### Set Update Methods

```python
s1.intersection_update(s2)  # Update s1 with intersection
s1.difference_update(s2)    # Update s1 with difference
```

### Set Relationships

- **Superset:** All elements of s2 are in s1
- **Subset:** All elements are contained in another set
- **Disjoint:** Intersection is null (nothing in common)

### Hash Function

```python
hash_value = sum(data) % size
```

### Time Complexity

| Operation | Complexity |
|-----------|------------|
| Union, Intersection, Symmetric Difference | O(n1 + n2) |
| Subset | O(n2) |
| Superset | O(n2) |
| Disjoint | O(n1 + n2) |

---

## Dictionaries

Store duplicate elements with **immutable keys** and index-based access without compromising time efficiency.

**Key Concept:** Instead of values passing through hash function, the **key** is passed. Based on the hash key generated, the key-value pair is stored.

### Dictionary Methods

```python
# Update
d.update({5: 'c', 6: 'java'})
d.update(seven='js', eight='php')

# Pop
d.pop(3)                    # Pops and returns value
d.pop(3, 'not found')       # With default
d.popitem()                 # Pops last item

# Delete
del d[4]
```

### Collections Module

**Counter:**
```python
from collections import Counter

s = "mississippi"
c = Counter(s)

c.elements()        # Iterator over elements
c.most_common(1)    # Most common elements

# Counter arithmetic
c1 = Counter(a=4, b=3, c=2)
c2 = Counter(a=3, b=2, c=1)
c3 = c1 + c2        # Addition
c3 = c1 - c2        # Subtraction
c3 = c1 | c2        # Union
c3 = c1 & c2        # Intersection
```

**namedtuple:**
```python
from collections import namedtuple

Student = namedtuple('Student', ('name', 'age', 'stream', 'avg'))
s1 = Student('Alex', 22, 'CS', 78)

print(s1)           # Student(name='Alex', age=22, stream='CS', avg=78)
print(s1[0])        # 'Alex'
print(s1.name)      # 'Alex'
```

**deque (Double-ended queue):**
```python
from collections import deque

dq = deque([1, 2, 3, 4])
dq.append(5)
dq.appendleft(99)
```

---

## Object-Oriented Programming

### Creating Instance Variables

Three ways to create instance variables:

```python
# 1. During object creation
cr = Footballer('Cristiano', 22)

# 2. After object creation
cr.age = 22

# 3. Using setattr()
setattr(cr, 'age', 22)
```

**Other Attribute Functions:**
```python
getattr(obj, 'attribute')
hasattr(obj, 'attribute')
```

**All instance variables are stored in `__dict__`:**
```python
print(cr.__dict__)
```

### Running Script vs Import

```python
if __name__ == '__main__':
    main()
```

---

## Decorators

Functions are **first-class objects** in Python - everything a regular object can do, functions can do too.

### First-Order Functions

**1. Functions can be assigned to variables:**
```python
def fun():
    print('fun')

f = fun()   # Calls method
f2 = fun    # Stores reference
```

**2. Functions can be passed as input:**
```python
def fun(ref):
    print('fun')
    ref()

def fun1():
    print('fun1')

fun(fun1)
```

**3. Functions can be returned:**
```python
def sum_list(lst):
    return sum(lst)

def product(lst):
    result = 1
    for i in lst:
        result *= i
    return result

def fun(choice):
    if choice == 'sum':
        return sum_list
    else:
        return product

f1 = fun('sum')
f1([1, 2, 3, 4, 5])
```

**4. Nested functions:**
```python
def outer():
    def inner():
        print("Inner function")
    inner()
```

### Decorator Syntax

```python
def outer(ref):
    print('outer')
    def inner(lst):
        if '0' in lst:
            print('0 is present')
        else:
            ref(lst)
    return inner

@outer
def product(lst):
    pr = 1
    for i in lst:
        pr *= i
    print(pr)

product([1, 2, 3, 4, 5])
```

### Types of Methods

**1. Instance Method:**
```python
def show(self):
    print(self.name)
```

**2. Class Method:**
```python
@classmethod
def bmw(cls):
    return cls("BMW", 3000)
```

**3. Static Method:**
```python
@staticmethod
def is_luxury(cc):
    return cc > 2000
```

**Example:**
```python
class Car:
    def __init__(self, name, cc):
        self.name = name
        self.cc = cc

    @classmethod
    def bmw(cls):
        return cls("BMW", 3000)

    @staticmethod
    def is_luxury(cc):
        return cc > 2000

    def show(self):
        print(self.name, self.cc, Car.is_luxury(self.cc))

c = Car.bmw()
c.show()
```

---

## Encapsulation

### Name Mangling

Double underscore `__` creates name mangling:

```python
class AccountHolder:
    def __init__(self):
        self.__bal = 10000  # Name mangling: _AccountHolder__bal

    def get_bal(self):
        return self.__bal

    def set_bal(self, amt):
        if amt > 0:
            self.__bal = amt
        else:
            print("Invalid Amount")

    bal = property(get_bal, set_bal)

# Usage
ah = AccountHolder()
print(ah.bal)       # Calls get_bal()
ah.bal = 20000      # Calls set_bal()
ah.bal = -5000      # Rejected
ah.__bal = 99999    # Creates NEW variable (doesn't modify real balance)
```

### Property Decorator

```python
class AccountHolder:
    def __init__(self):
        self.__bal = 10000

    @property
    def bal(self):
        return self.__bal

    @bal.setter
    def bal(self, amt):
        if amt > 0:
            self.__bal = amt
        else:
            print("❌ Invalid amount")

# Usage
ah = AccountHolder()
print(ah.bal)     # Direct access
ah.bal = 1000     # Direct setting
```

---

## Inheritance

### super() Keyword

Refers to superclass (parent) objects.

```python
class Customer:
    def __init__(self, name, ph, email):
        self.name = name
        self.ph = ph
        self.email = email

class PlatinumCustomer(Customer):
    def __init__(self, name, ph, email, plat_id):
        super().__init__(name, ph, email)
        self.plat_id = plat_id
```

### Calling Parent Methods

```python
class A:
    def fun(self):
        print('A')

class B(A):
    def fun(self):
        print('B')

class C(B):
    def fun(self):
        super(B, self).fun()  # Calls A's fun()
        print('C')
```

### Type Checking

```python
isinstance(obj, int)        # Check if instance of type
issubclass(ChildClass, ParentClass)  # Check subclass relationship
```

---

## Polymorphism

Polymorphism establishes **1:many** relationship. Child class can have methods with same name as parent class.

### Duck Typing

Don't check types - check for presence of methods/attributes.

```python
class WhatsApp:
    def send_message(self):
        print("Message sent using WhatsApp")

class FacebookMessenger:
    def send_message(self):
        print("Message sent using Facebook")

def use_messenger(ref):
    ref.send_message()  # Duck typing - we don't check type

wa = WhatsApp()
fb = FacebookMessenger()

use_messenger(wa)
use_messenger(fb)
```

### Monkey Patching

Adding methods to classes at runtime:

```python
def send_live_location(self):
    print("Live location sent")

WhatsApp.send_live_location = send_live_location
```

---

## Dunder Methods (Magic Methods)

Special methods with double underscores.

```python
class BankAccount:
    def __init__(self, name, balance):
        # Object creation
        self.name = name
        self.balance = balance

    def __str__(self):
        # String representation
        return f"Account: {self.name}, Balance: ₹{self.balance}"

    def __add__(self, other):
        # Addition operator
        return self.balance + other.balance

    def __len__(self):
        # len() function
        return len(self.name)

    def __eq__(self, other):
        # Equality operator
        return self.balance == other.balance

    def __lt__(self, other):
        # Less than operator
        return self.balance < other.balance

    def __call__(self, amount):
        # Make object callable
        self.balance += amount
        return self.balance

# Usage
acc1 = BankAccount("Asha", 5000)
acc2 = BankAccount("Ravi", 8000)

print(acc1)             # __str__
print(acc1 + acc2)      # __add__
print(len(acc1))        # __len__
print(acc1 == acc2)     # __eq__
print(acc1 < acc2)      # __lt__
print(acc1(2000))       # __call__
```

---

## Composition vs Aggregation

### Composition
One class is container, other is content. If container is deleted, contents are also deleted.

### Aggregation
Weak form of composition. If container is deleted, contents can still exist.

```python
class DeliveryAgent:
    def __init__(self, name, rating, ph_no):
        self.name = name
        self.rating = rating
        self.ph_no = ph_no

class Restaurant:
    class FoodItem:  # Composition
        def __init__(self, name, price, rating, is_veg):
            self.name = name
            self.price = price
            self.rating = rating
            self.is_veg = is_veg

    def __init__(self, name, address, rating):
        self.name = name
        self.address = address
        self.rating = rating
        self.pizza = Restaurant.FoodItem("Pizza", 500, 4.5, True)

    def assign_delivery_agent(self, agent):  # Aggregation
        self.agent = agent

# Usage
r = Restaurant("XYZ", "Bangalore", 4.4)
steve = DeliveryAgent("Steve", 4.7, 9988665555)
r.assign_delivery_agent(steve)
```

---

## Abstraction

Abstract classes define interface but don't provide implementation.

```python
from abc import ABC, abstractmethod

class VoiceAssistant(ABC):
    
    @abstractmethod
    def activate_assistant(self):
        pass

    @abstractmethod
    def perform_task(self):
        pass

    @abstractmethod
    def use_builtin_apps(self):
        pass

class Siri(VoiceAssistant):
    def activate_assistant(self):
        print("Hey Siri")

    def perform_task(self):
        print("Using Apple servers")

    def use_builtin_apps(self):
        print("Using iOS apps")

class Alexa(VoiceAssistant):
    def activate_assistant(self):
        print("Alexa")

    def perform_task(self):
        print("Using Amazon servers")

    def use_builtin_apps(self):
        print("Using Fire OS apps")

# Polymorphic function
def use_assistant(ref):
    ref.activate_assistant()
    ref.perform_task()
    ref.use_builtin_apps()

# Usage
s = Siri()
a = Alexa()

use_assistant(s)
use_assistant(a)
```

---

## Exception Handling

### Basic Structure

- **try:** Test code for errors
- **except:** Handle the error
- **else:** Execute if no errors
- **raise:** Throw an exception
- **finally:** Always executes

### Valid Combinations

```python
# Basic
try:
    # code
except:
    # handle

# With else
try:
    # code
except:
    # handle
else:
    # no error

# With finally
try:
    # code
except:
    # handle
finally:
    # always runs

# All together
try:
    # code
except:
    # handle
else:
    # no error
finally:
    # always runs
```

### Important Notes

- Generic exception handlers should be at the end
- If no except block in current method, traces back to caller
- If nowhere found, program terminates

### Custom Exceptions

```python
class DuplicateUserError(Exception):
    pass

class WeakPasswordError(Exception):
    pass

class User:
    user_name_set = set()

    def __init__(self, un, mob, pwd):
        self.un = un
        self.mob = mob
        self.pwd = pwd
        self.add_user()
        self.validate()

    def add_user(self):
        if self.un in User.user_name_set:
            raise DuplicateUserError("Username already exists")
        else:
            User.user_name_set.add(self.un)

    def validate(self):
        uc = lc = nu = sp = 0
        for i in self.pwd:
            if i.isupper():
                uc += 1
            elif i.islower():
                lc += 1
            elif i.isdigit():
                nu += 1
            else:
                sp += 1

        if len(self.pwd) < 6 or uc == 0 or lc == 0 or nu == 0 or sp == 0:
            raise WeakPasswordError("Password not strong enough")

# Usage
try:
    u1 = User("john", 1234567890, "Pass@123")
    u2 = User("john", 9876543210, "Pass@456")
except DuplicateUserError as e:
    print(e)
except WeakPasswordError as e:
    print(e)
else:
    print("Account created successfully")
```

---
