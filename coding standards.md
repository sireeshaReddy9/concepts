###  PEP 8
**PEP 8** stands for **Python Enhancement Proposal 8**.
It is the official style guide for writing clean and readable Python code.

Why PEP 8?

*  Code readability
*  Collaboration
*  Professional coding standards

---


##  PEP 8 Rules


## Indentation

## Use 4 spaces per indentation level


---

# Maximum Line Length

* Limit lines to **79 characters**
* Docstrings/comments: **72 characters**

---

# Blank Lines

 * Between top-level functions - **2**           
 * Between class methods       - **1**           

---

# Imports

###  Rules:

* Imports should be at the top of the file
* One import per line

```python
import os
import sys

```

---

# Naming Conventions

* Variables & Functions - snake_case
* Classes - PascalCase
* Constants - UPPER_CASE

---

# Spaces Around Operators

###  Correct

```python
x = a + b
```
---

## Avoid comparing to True/False

```python
if is_valid:
```

# Docstrings

Every module, class, and function should have a docstring.

###  Correct

```python
def add(a, b):
    """
    Add two numbers and return the result.
    """
    return a + b
```

---

# Comments

* Write complete sentences.
* Start with a capital letter.
* Use inline comments carefully.

---


# Trailing Commas

Useful for multi-line lists.

```python
fruits = [
    "apple",
    "banana",
    "cherry",
]
```
# Virtual Environment

A **Virtual Environment** is an isolated Python environment.

It allows you to:

* Install packages separately for each project
* Avoid version conflicts

---

Project A needs:

```bash
Django==3.2
```

Project B needs:

```bash
Django==4.2
```

With a virtual environment, no conflict 

---

##  Create virtual environment

```bash
python -m venv myenv
```

## Activate virtual environment

### 🔹 Windows

```bash
myenv\Scripts\activate
```

### 🔹 Linux / Mac

```bash
source myenv/bin/activate
```

##  Deactivate environment

```bash
deactivate
```
#  pip(Pip Installs Packages)

It is Python’s official package manager

---

#  Why is pip important?

* Install third-party libraries
* Manage project dependencies
* Upgrade or remove packages

##  Install a package

```bash
pip install package_name
```

##  Install specific version

```bash
pip install package_name==1.2.0
```


##  Upgrade a package

```bash
pip install --upgrade package_name
```

## Uninstall a package

```bash
pip uninstall package_name
```

## Freeze dependencies

```bash
pip freeze
```

