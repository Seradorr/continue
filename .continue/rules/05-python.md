---
name: Python Rules
description: |
  Python scripting, otomasyon ve veri isleme icin
  PEP 8, type hints ve anti-pattern kurallari.
globs:
  - "**/*.py"
  - "**/requirements.txt"
  - "**/pyproject.toml"
  - "**/setup.py"
  - "**/Pipfile"
alwaysApply: false
---

# PYTHON KURALLARI

Bu kurallar Python scripting, otomasyon ve veri isleme icin gecerlidir.

---

## 1. KOD STILI (PEP 8)

### Naming Convention

| TIP | FORMAT | ORNEK |
|-----|--------|-------|
| Module | snake_case | data_processor.py |
| Class | PascalCase | DataProcessor |
| Function/Method | snake_case | process_data |
| Variable | snake_case | file_path |
| Constant | UPPER_SNAKE_CASE | MAX_RETRIES |
| Private | _leading | _internal_method |

### Import Sirasi

```python
# 1. Standard library
import os
from pathlib import Path
from typing import Optional, List

# 2. Third-party
import numpy as np
import pandas as pd

# 3. Local
from myproject.utils import helpers
```

### Dosya Yapisi

```python
#!/usr/bin/env python3
"""Module docstring."""

import os
from typing import Optional

# Constants
DEFAULT_TIMEOUT = 30

# Classes
class MyClass:
    """Class docstring."""
    pass

# Functions
def my_function(param: str) -> bool:
    """Function docstring."""
    pass

# Main guard
if __name__ == "__main__":
    main()
```

---

## 2. TYPE HINTS

```python
from typing import Optional, List, Dict

def process_data(
    items: List[str],
    config: Dict[str, int],
    callback: Optional[Callable[[str], None]] = None
) -> tuple[bool, str]:
    pass

# Python 3.10+
def get_value(key: str) -> int | None:
    pass
```

---

## 3. ERROR HANDLING

```python
# YANLIS: Cok genel
try:
    process_data()
except:
    pass

# DOGRU: Spesifik
try:
    result = process_file(path)
except FileNotFoundError:
    logger.error(f"Not found: {path}")
    raise
except PermissionError as e:
    logger.error(f"Permission denied: {e}")
    return None
```

### Context Manager

```python
# YANLIS
f = open("file.txt")
try:
    data = f.read()
finally:
    f.close()

# DOGRU
with open("file.txt") as f:
    data = f.read()
```

---

## 4. PATHLIB

```python
from pathlib import Path

# YANLIS
import os
path = os.path.join("dir", "file.txt")

# DOGRU
path = Path("dir") / "file.txt"
path.mkdir(parents=True, exist_ok=True)
content = path.read_text()
path.write_text("content")
```

---

## 5. ANTI-PATTERNS

### Mutable Default

```python
# YANLIS
def add_item(item, items=[]):
    items.append(item)
    return items

# DOGRU
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### Global Degisken

```python
# YANLIS
counter = 0
def increment():
    global counter
    counter += 1

# DOGRU
class Counter:
    def __init__(self):
        self.value = 0
    def increment(self):
        self.value += 1
```

### == None

```python
# YANLIS
if value == None:
    pass

# DOGRU
if value is None:
    pass
```

### range(len())

```python
# YANLIS
for i in range(len(items)):
    print(items[i])

# DOGRU
for item in items:
    print(item)

# Index gerekiyorsa
for i, item in enumerate(items):
    print(f"{i}: {item}")
```

---

## 6. PYTHONIC YAPILAR

### List Comprehension

```python
squares = [x**2 for x in range(10)]
evens = [x for x in range(10) if x % 2 == 0]
word_lengths = {word: len(word) for word in words}
```

### Generator

```python
# Memory-efficient
sum_squares = sum(x**2 for x in range(1000000))

def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()
```

### Unpacking

```python
a, b = 1, 2
a, b = b, a  # Swap

first, *rest = [1, 2, 3, 4, 5]
defaults = {"timeout": 30}
config = {**defaults, "timeout": 60}
```

### Walrus Operator

```python
if (n := len(items)) > 10:
    print(f"Too many: {n}")
```

---

## 7. LOGGING

```python
import logging

logger = logging.getLogger(__name__)

def process_data(data):
    logger.debug("Processing %s items", len(data))  # Lazy eval
    try:
        result = transform(data)
        logger.info("Success: %s items", len(result))
        return result
    except ValueError as e:
        logger.error("Failed: %s", e, exc_info=True)
        raise
```

---

## 8. PROJE YAPISI

```
my_project/
├── src/
│   └── my_project/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
├── requirements.txt
├── pyproject.toml
└── README.md
```
