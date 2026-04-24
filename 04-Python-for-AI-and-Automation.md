# Python — Fundamentals to Expert for AI and QA Automation

> Subject #4 of the Master Learning Plan.
> Owner: Govind Divekar. Prerequisites: none (complete beginners welcome). Estimated time: ~80 hours.
> Target outcome: Write idiomatic Python for automation scripts, pytest test suites, and AI/ML workflows with NumPy, pandas, scikit-learn, and PyTorch basics.

## 1. Overview & Learning Outcomes

This subject covers Python from install through production—specifically tooled for QA automation and AI/ML. You'll learn to write clean, typed Python; build test automation frameworks with pytest; and leverage libraries like pandas and PyTorch for data-driven testing and ML pipelines.

**By the end, you will:**
- Set up Python environments (venv, poetry, uv) and understand dependency management.
- Write idiomatic, type-hinted Python following PEP 8 and PEP 484.
- Build automation tools with requests, Playwright, pytest, and async/await.
- Use pandas to wrangle data; NumPy for numerical work; scikit-learn for classical ML.
- Train simple neural networks in PyTorch; serve models via FastAPI.
- Ship a package to PyPI; integrate tests in CI/CD.
- Recognize when to use Hugging Face, LangChain, or direct APIs for LLM workflows.

**Core thesis:** Python is the lingua franca of automation and AI. Master its idioms, tooling, and ecosystem. By module 38, you'll have hands-on experience with the entire stack.

---

## 2. Detailed Syllabus

### Part A: Core Python (35 hours)

| Module | Topic | Hours | Key Outcomes |
|--------|-------|-------|--------------|
| 1 | Install Python 3.11+, pip, pipx; PATH & shells | 2 | Runnable Python executable; understand PATH; use pipx for CLI tools |
| 2 | Virtual environments: venv, poetry, uv | 3 | Create isolated envs; use lockfiles (poetry.lock, uv.lock); reproduce installs |
| 3 | Syntax: variables, types, f-strings, truthiness | 3 | Name unpacking; numeric/str/bool operators; f-string formatting |
| 4 | Collections: list, tuple, set, dict; comprehensions | 4 | Nested structures; list/dict/set comprehensions; defaultdict, Counter |
| 5 | Control flow: if/elif/else, for, while, match/case | 3 | Loop unpacking; walrus operator; match patterns (3.10+) |
| 6 | Functions: args, *args/**kwargs, defaults, keyword-only | 3 | Mutable default trap; *args/**kwargs unpacking; functools.partial |
| 7 | Type hints: basic, Optional, Union, Generics; PEP 604 | 3 | Annotate args/returns; use typing.List, Dict, Callable; X \| Y syntax |
| 8 | Modules, packages, imports; `__name__`, `__main__` | 2 | Relative imports; package structure; `if __name__ == "__main__"` idiom |
| 9 | Files, context managers, pathlib | 2 | Use `with` for files; Path API; avoid string paths |
| 10 | Error handling: try/except/else/finally; exceptions | 2 | Specific exception catching; chaining; raise from; custom exceptions |
| 11 | OOP: class, dunder, properties, inheritance, ABC, dataclasses | 3 | `__init__`, `__str__`, `__eq__`; @property; inheritance and MRO; frozen dataclasses |
| 12 | Iterators, generators, yield; generator expressions | 2 | `iter()`, `next()`; generator protocol; memory efficiency |
| 13 | Decorators, closures, functools | 2 | @wraps; @lru_cache; partial; compose decorators |
| 14 | Context managers: contextlib | 1 | @contextmanager decorator; @asynccontextmanager |
| 15 | Concurrency: threading, multiprocessing, asyncio | 4 | I/O threads vs CPU processes; async/await; asyncio.gather, Queue |
| 16 | Logging: logging module, handlers, structlog | 2 | Logger hierarchy; handlers; JSON logging; integrate with tests |
| 17 | Stdlib gems: collections, itertools, datetime, json, csv, sqlite3, subprocess | 3 | Counter, deque, chain, compress; strftime; json.dumps; csv.DictWriter; subprocess.run |

**Part A Total: 35 hours**

### Part B: Automation & Testing (28 hours)

| Module | Topic | Hours | Key Outcomes |
|--------|-------|-------|--------------|
| 18 | requests / httpx: sync & async API clients | 3 | Response codes; JSON; auth; retries with tenacity; async batching |
| 19 | CLI tools: argparse, click, typer | 2 | Subcommands; type annotations; help text; typer for modern UX |
| 20 | pytest: discovery, assertions, fixtures, parametrize | 4 | conftest.py; @pytest.fixture; scope (function/session); parametrize combinations |
| 21 | pytest advanced: markers, xfail/skip, plugins | 2 | @pytest.mark.skip; xfail reason; pytest-xdist for parallel; html reports |
| 22 | Mocking: unittest.mock, monkeypatch, responses | 3 | Mock objects; side_effect; monkeypatch in fixtures; responses for HTTP |
| 23 | Playwright Python; Selenium legacy overview | 2 | Page object model; locators; wait strategies; trace & video recording |
| 24 | Web scraping: requests + BeautifulSoup, Playwright for JS | 2 | CSS selectors; parse tables; handle pagination; robots.txt respect |
| 25 | Automation patterns: page objects, data factories, Faker | 2 | Separate locators from logic; @dataclass factories; Faker for synthetic data |
| 26 | Packaging: pyproject.toml, hatch, poetry; PyPI | 3 | PEP 517/518; build backends; upload to TestPyPI; semantic versioning |

**Part B Total: 23 hours**

### Part C: AI/ML & Advanced (22 hours)

| Module | Topic | Hours | Key Outcomes |
|--------|-------|-------|--------------|
| 27 | NumPy: arrays, broadcasting, dtypes, vectorization | 3 | n-d arrays; slicing; broadcasting rules; avoid Python loops |
| 28 | pandas: Series, DataFrame, indexing, groupby, merge, datetime | 4 | .loc/.iloc; .apply; .groupby().agg(); .merge(); .read_csv(); missing data |
| 29 | Visualization: Matplotlib, Seaborn, Plotly | 2 | Line/scatter/bar plots; faceting; interactive Plotly charts |
| 30 | scikit-learn: datasets, pipelines, estimators, cross-validation, tuning | 3 | train_test_split; Pipeline; GridSearchCV; classification metrics (precision/recall) |
| 31 | PyTorch essentials: tensors, autograd, nn.Module, training loops, GPU | 3 | Tensor operations; backward(); DataLoader; .to(device); saving/loading weights |
| 32 | TensorFlow/Keras: intro, comparison with PyTorch | 1 | Sequential vs Functional API; tf.data; when to choose TF vs PyTorch |
| 33 | Hugging Face transformers & datasets | 2 | Load pretrained models; tokenizers; HF datasets; fine-tuning patterns |
| 34 | LangChain vs LlamaIndex vs direct APIs; RAG patterns | 2 | When to use abstractions; prompt chains; vector stores; embeddings |
| 35 | Embeddings & vector search: sentence-transformers, FAISS | 1 | Semantic search; cosine similarity; FAISS indexing for scale |
| 36 | Model evaluation: sklearn metrics, HF evaluate | 1 | Precision/recall/F1; confusion matrix; custom metrics |
| 37 | Notebook vs script: Jupyter, nbconvert, papermill | 1 | When each is right; parametrize notebooks; convert to .py |
| 38 | Productionizing: FastAPI, pydantic schemas, model serving | 2 | /predict endpoints; request validation; OpenAPI schema; async views |

**Part C Total: 25 hours**

**Grand Total: 83 hours**

---

## 3. Topics — Explanations with Examples

### Part A: Core Python

#### Module 1: Install Python 3.11+, pip, pipx; PATH & Shells

**Concept:**
Python 3.11 ships with better error messages, faster performance, and exception groups. Use `pyenv` (macOS/Linux) or Windows Store / installer to manage versions. `pip` installs packages; `pipx` isolates CLI tools (black, ruff, pytest, pdoc).

**Example:**
```python
# After installing Python 3.11, test it:
python --version  # Python 3.11.x
python -m pip install --upgrade pip
pipx install black ruff pytest  # Isolated CLIs

# Check PATH:
which python  # /usr/bin/python3.11 or similar
echo $PATH    # Should include ~/.local/bin for pipx
```

**Pitfalls:**
- Multiple Pythons on PATH; use `which python` to verify.
- pip installs globally, breaking isolation; always use venv or pipx.
- On Windows, use `python` not `py.exe` if you want portability in scripts.

---

#### Module 2: Virtual Environments: venv, poetry, uv

**Concept:**
Isolate project dependencies. `venv` is built-in; `poetry` adds lock files and package metadata; `uv` is fast and modern. Use lockfiles (poetry.lock, uv.lock) for reproducibility.

**Example:**
```python
# venv (built-in)
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install requests pytest

# poetry (more ergonomic)
poetry new myproject
cd myproject
poetry add requests pytest
poetry lock
poetry install

# uv (fastest; https://github.com/astral-sh/uv)
uv venv
source .venv/bin/activate
uv pip install requests pytest

# Lockfile ensures reproducibility
poetry install  # Uses poetry.lock, not latest
```

**Pitfalls:**
- Forgetting to activate; check `which python` is in venv.
- Modifying lockfiles manually; use `poetry add/remove`.
- Installing without a venv; creates global clutter and conflicts.

---

#### Module 3: Syntax — Variables, Types, f-strings, Truthiness

**Concept:**
Python is dynamically typed at runtime but supports type hints for static analysis. Learn numeric types (int, float, complex), str, bool, and truthiness rules (empty collections, 0, None are falsy).

**Example:**
```python
# Variables and types
name: str = "Govind"
age: int = 35
salary: float = 75_000.50  # Underscore for readability
is_manager: bool = True

# f-strings (PEP 498; Python 3.6+)
print(f"{name} is {age} years old and earns ${salary:,.2f}")
# Govind is 35 years old and earns $75,000.50

# f-string expressions
numbers = [1, 2, 3]
print(f"Sum: {sum(numbers)}")

# Truthiness
if name:  # Non-empty str is truthy
    print("Name exists")
if age > 30:  # Comparison returns bool
    print("Over 30")
if [] or {} or "":  # All falsy
    print("Never prints")
if None or 0 or False:  # All falsy
    pass

# Type checking (requires mypy)
from typing import Optional
def greet(name: Optional[str] = None) -> str:
    return f"Hello, {name or 'World'}!"
```

**Pitfalls:**
- Confusing `=` (assignment) with `==` (comparison).
- Mutable default arguments (covered in module 6).
- f-strings don't support backslashes inside {} expressions.

---

#### Module 4: Collections — list, tuple, set, dict; Comprehensions

**Concept:**
Lists are mutable arrays; tuples are immutable; sets are unique items; dicts map keys to values. Comprehensions create collections concisely. Understand when each is appropriate.

**Example:**
```python
# Lists: mutable, ordered, indexed
numbers = [1, 2, 3, 4, 5]
numbers.append(6)
numbers[0] = 10
print(numbers[1:3])  # [2, 3]

# Tuples: immutable, ordered, hashable (can be dict keys)
coord = (40.7128, -74.0060)  # Latitude, longitude
x, y = coord  # Unpacking
locations = {coord: "New York"}

# Sets: unique, unordered, fast membership
tags = {"python", "automation", "ai"}
tags.add("testing")
print("python" in tags)  # O(1) lookup

# Dicts: key-value pairs
person = {"name": "Govind", "age": 35, "city": "NYC"}
person["email"] = "govind@example.com"
print(person.get("phone", "No phone"))  # Default if missing

# List comprehension
squares = [x**2 for x in range(10) if x % 2 == 0]
# [0, 4, 16, 36, 64]

# Dict comprehension
word_lengths = {word: len(word) for word in ["python", "automation", "ai"]}
# {"python": 6, "automation": 10, "ai": 2}

# Set comprehension
unique_lengths = {len(word) for word in ["python", "automation", "ai"]}
# {2, 6, 10}

# Nested comprehension
matrix = [[i*j for j in range(3)] for i in range(3)]
# [[0, 0, 0], [0, 1, 2], [0, 2, 4]]

# defaultdict and Counter
from collections import defaultdict, Counter
word_count = Counter("abracadabra")
# Counter({'a': 5, 'b': 2, 'r': 2, 'c': 1, 'd': 1})
student_scores = defaultdict(list)
student_scores["Alice"].append(95)
```

**Pitfalls:**
- Tuples with one element need trailing comma: `(1,)` not `(1)`.
- Mutable values as dict keys (lists, dicts, sets) cause errors; only hashable (int, str, tuple) work.
- Modifying a list during iteration causes skips.

---

#### Module 5: Control Flow — if/elif/else, for, while, match/case

**Concept:**
Conditionals and loops are foundational. Python 3.10+ adds `match/case` for pattern matching, replacing complex if-elif chains.

**Example:**
```python
# if/elif/else
age = 25
if age < 13:
    category = "child"
elif age < 18:
    category = "teenager"
else:
    category = "adult"

# for loops
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

for name in ["Alice", "Bob", "Charlie"]:
    print(f"Hello, {name}")

# Unpacking in loops
pairs = [(1, "a"), (2, "b"), (3, "c")]
for num, letter in pairs:
    print(f"{num}: {letter}")

# while loop
count = 0
while count < 3:
    print(count)
    count += 1

# Walrus operator (:=) in loops
while (line := input("Enter text (quit to exit): ")) != "quit":
    print(f"You said: {line}")

# match/case (Python 3.10+)
status_code = 404
match status_code:
    case 200:
        message = "OK"
    case 404:
        message = "Not Found"
    case 500 | 502 | 503:
        message = "Server Error"
    case _:  # Default case
        message = "Unknown"

# match with guards
point = (0, 5)
match point:
    case (0, 0):
        description = "Origin"
    case (0, y) if y > 0:
        description = f"On positive y-axis"
    case (x, 0) if x > 0:
        description = "On positive x-axis"
    case _:
        description = "Other"
```

**Pitfalls:**
- `range(n)` starts at 0, not 1. Use `range(1, n+1)` for 1-based.
- `break` exits only the innermost loop.
- Indentation errors cause SyntaxError; be consistent (4 spaces).

---

#### Module 6: Functions — args, *args/**kwargs, defaults, keyword-only

**Concept:**
Functions are objects in Python. Learn positional, default, *args (variadic), **kwargs (keyword), and keyword-only parameters.

**Example:**
```python
# Basic function
def add(a: int, b: int) -> int:
    return a + b

# Default arguments
def greet(name: str = "World") -> str:
    return f"Hello, {name}!"

# *args: collect positional arguments as tuple
def sum_all(*args: int) -> int:
    return sum(args)

print(sum_all(1, 2, 3, 4))  # 10

# **kwargs: collect keyword arguments as dict
def print_info(**kwargs: str) -> None:
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Govind", city="NYC", job="QA")
# name: Govind
# city: NYC
# job: QA

# Combine positional, *args, **kwargs
def configure(host: str, port: int, *args, **kwargs):
    print(f"Host: {host}, Port: {port}")
    print(f"Extra positional: {args}")
    print(f"Extra keyword: {kwargs}")

# Keyword-only arguments (after *)
def create_user(name: str, *, email: str, role: str = "user") -> dict:
    return {"name": name, "email": email, "role": role}

user = create_user("Alice", email="alice@example.com", role="admin")
# create_user("Alice", "alice@example.com")  # Error: missing 'email' keyword

# Unpacking arguments
args_tuple = (3, 5)
print(add(*args_tuple))  # 8

kwargs_dict = {"name": "Govind"}
print(greet(**kwargs_dict))  # Hello, Govind!

# functools.partial for fixed args
from functools import partial
add_5 = partial(add, 5)
print(add_5(10))  # 15
```

**Pitfalls:**
- Mutable default arguments are shared across calls:
  ```python
  def bad_append(item, lst=[]):
      lst.append(item)
      return lst
  print(bad_append(1))  # [1]
  print(bad_append(2))  # [1, 2] — shared!
  
  def good_append(item, lst=None):
      if lst is None:
          lst = []
      lst.append(item)
      return lst
  ```
- *args and **kwargs consume positional/keyword args; later parameters must be keyword-only or in dict.

---

#### Module 7: Type Hints — basic, Optional, Union, Generics; PEP 604

**Concept:**
Type hints improve code readability and enable static analysis via `mypy`. Use `typing` module (or `builtins` in 3.9+) for complex types. PEP 604 allows `X | Y` instead of `Union[X, Y]` (Python 3.10+).

**Example:**
```python
from typing import Optional, Union, List, Dict, Callable, Tuple, Any
from collections.abc import Sequence

# Basic types
def add(a: int, b: int) -> int:
    return a + b

# Optional (x or None)
def find_user(user_id: int) -> Optional[dict]:
    if user_id > 0:
        return {"id": user_id, "name": "Govind"}
    return None

# Union (one of multiple types)
def process(value: Union[int, str]) -> str:
    if isinstance(value, int):
        return f"Integer: {value}"
    else:
        return f"String: {value}"

# PEP 604 syntax (Python 3.10+)
def process_new(value: int | str) -> str:
    return f"Value: {value}"

# Collections with type parameters
def get_scores() -> List[int]:
    return [95, 87, 92]

def get_mapping() -> Dict[str, float]:
    return {"Alice": 95.5, "Bob": 87.0}

# Callable for function types
def apply_operation(a: int, b: int, op: Callable[[int, int], int]) -> int:
    return op(a, b)

add_op = lambda x, y: x + y
print(apply_operation(5, 3, add_op))  # 8

# Tuple types
def get_coordinate() -> Tuple[float, float]:
    return (40.7128, -74.0060)

# Generic types for collections
T = TypeVar('T')
def first(items: List[T]) -> T:
    return items[0]

# Any (avoid when possible)
def unsafe_process(value: Any) -> Any:
    return value * 2

# Using typing.cast for narrowing
from typing import cast
def risky_cast(obj: object) -> int:
    return cast(int, obj) + 1
```

**Pitfalls:**
- Type hints are not enforced at runtime; use `mypy` for static checking.
- `Optional[X]` is `Union[X, None]`, not a default value.
- Generic TypeVar requires `from typing import TypeVar; T = TypeVar('T')`.

---

#### Module 8: Modules, Packages, Imports; `__name__`, `__main__`

**Concept:**
Organize code into modules (.py files) and packages (directories with `__init__.py`). Use relative imports within packages. The `__name__` variable is `"__main__"` only when the file is executed directly.

**Example:**
```python
# Structure:
# myproject/
#   __init__.py
#   utils.py
#   models/
#     __init__.py
#     user.py
#   main.py

# --- utils.py ---
def helper_function():
    return "I help"

# --- models/user.py ---
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str

# --- models/__init__.py ---
from .user import User
__all__ = ["User"]

# --- main.py ---
# Absolute import (from package root)
from myproject.utils import helper_function
from myproject.models import User

# Relative import (within package)
from .utils import helper_function

# Conditional execution
if __name__ == "__main__":
    user = User(name="Govind", email="govind@example.com")
    print(user)

# --- Command line ---
python main.py  # Runs if __name__ == "__main__" block
python -m myproject.main  # Also works; treats module as script
import main  # Does not run the if __name__ == "__main__" block
```

**Pitfalls:**
- Circular imports: module A imports B, B imports A. Refactor to avoid.
- Relative imports only work in packages; use `python -m` when running with relative imports.
- `__all__` in `__init__.py` controls what `from package import *` exposes.

---

#### Module 9: Files, Context Managers, pathlib

**Concept:**
Always use context managers (`with` statement) for files to ensure cleanup. Prefer `pathlib.Path` over string paths for cross-platform safety.

**Example:**
```python
from pathlib import Path
import json

# Reading files
file_path = Path("data.txt")
with open(file_path, "r") as f:
    content = f.read()

# Writing files
with open(file_path, "w") as f:
    f.write("Hello, World!")

# Reading lines
with open(file_path, "r") as f:
    for line in f:
        print(line.strip())

# JSON files
data = {"name": "Govind", "age": 35}
with open("config.json", "w") as f:
    json.dump(data, f, indent=2)

with open("config.json", "r") as f:
    loaded_data = json.load(f)

# pathlib operations
p = Path("myproject/data/file.txt")
print(p.exists())          # True/False
print(p.is_file())         # True/False
print(p.parent)            # Path("myproject/data")
print(p.stem)              # "file"
print(p.suffix)            # ".txt"
print(p.parts)             # ("myproject", "data", "file.txt")

# Create directories
p.parent.mkdir(parents=True, exist_ok=True)

# Glob patterns
for txt_file in Path("myproject").glob("**/*.txt"):
    print(txt_file)

# String conversion (when needed)
file_str = str(p)
```

**Pitfalls:**
- Not closing files; always use `with` or `.close()`.
- String paths break on Windows (\ vs /); use pathlib.Path.
- `glob()` without `**` doesn't recurse; use `**/*` for recursive.

---

#### Module 10: Error Handling — try/except/else/finally, Custom Exceptions

**Concept:**
Catch specific exceptions, not bare `except`. Use `else` for code that runs only if no exception occurred. Use `finally` for cleanup. Chain exceptions with `raise ... from`.

**Example:**
```python
# Basic try/except
try:
    result = 10 / 0
except ZeroDivisionError as e:
    print(f"Error: {e}")

# Multiple exceptions
try:
    value = int("not a number")
except (ValueError, TypeError) as e:
    print(f"Conversion failed: {e}")

# Catching all exceptions (avoid in production)
try:
    risky_operation()
except Exception as e:
    print(f"Unexpected error: {e}")

# try/else/finally
try:
    file = open("data.txt", "r")
    data = file.read()
except FileNotFoundError:
    print("File not found")
else:
    print(f"Read {len(data)} bytes")
finally:
    file.close()  # Always runs

# Exception chaining
try:
    result = 10 / 0
except ZeroDivisionError as e:
    raise ValueError("Invalid calculation") from e

# Custom exceptions
class ConfigError(Exception):
    pass

def load_config(path: str) -> dict:
    if not Path(path).exists():
        raise ConfigError(f"Config file not found: {path}")
    return {}

# Using context to add info
try:
    config = load_config("missing.json")
except ConfigError as e:
    print(f"Config error: {e}")

# Contextlib for custom cleanup
import contextlib
@contextlib.contextmanager
def temporary_dir(path: str):
    try:
        p = Path(path)
        p.mkdir()
        yield p
    finally:
        import shutil
        shutil.rmtree(p)

with temporary_dir("temp") as tmpdir:
    print(f"Using {tmpdir}")
# tmpdir cleaned up automatically
```

**Pitfalls:**
- Bare `except:` silences all errors, including KeyboardInterrupt; use specific exceptions.
- Re-raising without context (`raise e`); use `raise ... from` to chain.
- Not closing resources if an exception occurs; use `with` or `finally`.

---

#### Module 11: OOP — class, dunder, properties, inheritance, ABC, dataclasses

**Concept:**
Classes encapsulate state and behavior. Use `__init__` for initialization, `__str__` for user-friendly repr, `__eq__` for equality. `@property` exposes data as attributes. Inheritance enables code reuse. Abstract Base Classes (ABC) enforce interfaces. Dataclasses reduce boilerplate.

**Example:**
```python
from abc import ABC, abstractmethod
from dataclasses import dataclass, field
from typing import List

# Basic class
class Person:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age
    
    def __str__(self) -> str:
        return f"{self.name} ({self.age})"
    
    def __repr__(self) -> str:
        return f"Person(name={self.name!r}, age={self.age})"
    
    def __eq__(self, other) -> bool:
        if not isinstance(other, Person):
            return False
        return self.name == other.name and self.age == other.age

# Properties
class User:
    def __init__(self, email: str):
        self._email = email
    
    @property
    def email(self) -> str:
        return self._email
    
    @email.setter
    def email(self, value: str) -> None:
        if "@" not in value:
            raise ValueError("Invalid email")
        self._email = value

# Inheritance
class Admin(Person):
    def __init__(self, name: str, age: int, role: str):
        super().__init__(name, age)
        self.role = role
    
    def __str__(self) -> str:
        return f"{super().__str__()} [Admin: {self.role}]"

# Abstract Base Class
class Repository(ABC):
    @abstractmethod
    def get(self, id: int):
        pass
    
    @abstractmethod
    def save(self, obj):
        pass

class UserRepository(Repository):
    def __init__(self):
        self.storage = {}
    
    def get(self, id: int):
        return self.storage.get(id)
    
    def save(self, obj: Person):
        self.storage[id(obj)] = obj

# Dataclasses (less boilerplate)
@dataclass
class Book:
    title: str
    author: str
    pages: int = 0

@dataclass(frozen=True)
class ImmutablePoint:
    x: float
    y: float
    # Cannot modify after creation

@dataclass
class Library:
    name: str
    books: List[Book] = field(default_factory=list)

# Class methods and static methods
class Config:
    _instance = None
    
    @staticmethod
    def parse_env(key: str) -> str:
        import os
        return os.getenv(key, "")
    
    @classmethod
    def from_file(cls, path: str):
        return cls()

# Usage
person = Person("Govind", 35)
print(person)  # Govind (35)
admin = Admin("Alice", 40, "Manager")
print(admin)  # Alice (40) [Admin: Manager]

book = Book(title="Python Crash Course", author="Eric Matthes")
print(book)  # Book(title='Python Crash Course', author='Eric Matthes', pages=0)
```

**Pitfalls:**
- Not calling `super().__init__()` in subclass constructors.
- Mutable default in dataclass; use `field(default_factory=list)`.
- Mixing `@property` and setters can be confusing; document intent.

---

#### Module 12: Iterators, Generators, yield; Generator Expressions

**Concept:**
Iterators implement `__iter__()` and `__next__()`. Generators use `yield` to produce values lazily, saving memory. Generator expressions are compact syntax.

**Example:**
```python
# Iterator protocol
class CountUp:
    def __init__(self, max: int):
        self.max = max
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        else:
            raise StopIteration

for num in CountUp(3):
    print(num)  # 1, 2, 3

# Generator function
def count_up(max: int):
    current = 0
    while current < max:
        current += 1
        yield current

for num in count_up(3):
    print(num)  # 1, 2, 3

# Generator with send()
def echo():
    while True:
        x = yield
        print(f"Echo: {x}")

gen = echo()
next(gen)  # Prime the generator
gen.send("Hello")  # Echo: Hello

# Generator expression (like list comprehension but lazy)
numbers = (x**2 for x in range(1000000))
print(next(numbers))  # 0 (doesn't compute all 1M squares upfront)

# yield from (delegate to another generator)
def flatten(nested: list):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item

nested = [1, [2, 3, [4, 5]], 6]
print(list(flatten(nested)))  # [1, 2, 3, 4, 5, 6]
```

**Pitfalls:**
- Generators are single-pass; iterate again requires calling the function again.
- `StopIteration` should not leak outside a generator; Python 3.7+ converts it to `RuntimeError`.
- Generator expression syntax: `(x for x in ...)` not `[x for x in ...]` (bracket creates list).

---

#### Module 13: Decorators, Closures, functools

**Concept:**
Decorators are functions that modify other functions. Closures capture variables from enclosing scope. `functools` provides utilities like `@wraps`, `@lru_cache`, and `partial`.

**Example:**
```python
from functools import wraps, lru_cache, partial
import time

# Simple decorator
def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.4f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(0.1)
    return "Done"

slow_function()  # slow_function took 0.1001s

# Decorator with arguments
def retry(max_attempts: int):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed, retrying...")
            return None
        return wrapper
    return decorator

@retry(max_attempts=3)
def flaky_operation():
    import random
    if random.random() < 0.7:
        raise ValueError("Random failure")
    return "Success"

# Stacking decorators
@timer
@retry(max_attempts=2)
def combined():
    return "Result"

# Closures
def multiplier(factor: int):
    def inner(x: int) -> int:
        return x * factor
    return inner

double = multiplier(2)
triple = multiplier(3)
print(double(5))  # 10
print(triple(5))  # 15

# functools.lru_cache (memoization)
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))  # Computed quickly due to caching

# functools.partial
def power(base: float, exponent: float) -> float:
    return base ** exponent

square = partial(power, exponent=2)
print(square(5))  # 25
```

**Pitfalls:**
- Forgetting `@wraps(func)` causes decorator to hide function name and docstring.
- Decorator order matters: `@a @b def f()` applies b first, then a.
- LRU cache doesn't work with unhashable arguments (e.g., lists, dicts).

---

#### Module 14: Context Managers with contextlib

**Concept:**
Context managers (`with` statement) ensure setup and teardown. Use `@contextmanager` decorator to create them concisely.

**Example:**
```python
from contextlib import contextmanager, asynccontextmanager
import tempfile
from pathlib import Path

# Class-based context manager
class DatabaseConnection:
    def __init__(self, connection_string: str):
        self.conn_str = connection_string
        self.connection = None
    
    def __enter__(self):
        print(f"Connecting to {self.conn_str}")
        self.connection = f"<Connection to {self.conn_str}>"
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Closing connection")
        self.connection = None
        return False  # Don't suppress exceptions

with DatabaseConnection("localhost:5432") as conn:
    print(f"Using {conn}")

# Decorator-based context manager
@contextmanager
def temporary_file(suffix: str = ".txt"):
    try:
        fd, path = tempfile.mkstemp(suffix=suffix)
        yield Path(path)
    finally:
        import os
        os.close(fd)
        Path(path).unlink()

with temporary_file() as tmp:
    tmp.write_text("Hello, World!")
    print(f"Created {tmp}")

# Async context manager
@asynccontextmanager
async def async_resource():
    print("Acquiring async resource")
    try:
        yield "Resource"
    finally:
        print("Releasing async resource")

# Suppress exceptions
from contextlib import suppress
with suppress(FileNotFoundError):
    Path("nonexistent.txt").unlink()
# No error raised
```

**Pitfalls:**
- Exceptions in `__enter__` don't call `__exit__`.
- Returning `True` from `__exit__` suppresses exceptions; usually return `False`.
- Async context managers require `async with`, not `with`.

---

#### Module 15: Concurrency — threading, multiprocessing, asyncio

**Concept:**
Use threading for I/O-bound tasks (network, files). Use multiprocessing for CPU-bound tasks (compute). Use asyncio for highly concurrent I/O (thousands of tasks). Understand GIL (Global Interpreter Lock).

**Example:**
```python
import threading
import multiprocessing
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

# Threading (I/O-bound)
def fetch_url(url: str):
    print(f"Fetching {url}...")
    time.sleep(1)  # Simulate network request
    return f"Data from {url}"

threads = []
for url in ["http://a.com", "http://b.com", "http://c.com"]:
    t = threading.Thread(target=fetch_url, args=(url,))
    t.start()
    threads.append(t)

for t in threads:
    t.join()
print("All fetches done")

# ThreadPoolExecutor (cleaner)
with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(fetch_url, ["http://a.com", "http://b.com"]))

# Multiprocessing (CPU-bound)
def compute(n: int) -> int:
    result = 0
    for i in range(n):
        result += i
    return result

with ProcessPoolExecutor(max_workers=2) as executor:
    results = list(executor.map(compute, [10**7, 10**7, 10**7]))
    print(results)

# asyncio (highly concurrent I/O)
async def fetch_async(url: str):
    print(f"Fetching {url}...")
    await asyncio.sleep(1)  # Simulate I/O
    return f"Data from {url}"

async def main():
    tasks = [
        fetch_async("http://a.com"),
        fetch_async("http://b.com"),
        fetch_async("http://c.com"),
    ]
    results = await asyncio.gather(*tasks)
    print(results)

asyncio.run(main())

# asyncio with Queue
async def producer(queue: asyncio.Queue):
    for i in range(3):
        await queue.put(f"item-{i}")

async def consumer(queue: asyncio.Queue):
    while True:
        item = await queue.get()
        print(f"Processing {item}")
        queue.task_done()

async def async_main():
    queue = asyncio.Queue()
    await asyncio.gather(
        producer(queue),
        consumer(queue),
    )

# Lock for shared state
lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:
        counter += 1

threads = [threading.Thread(target=increment) for _ in range(10)]
for t in threads:
    t.start()
for t in threads:
    t.join()
print(f"Counter: {counter}")  # 10 (safe due to lock)
```

**Pitfalls:**
- GIL in Python threads means only one thread executes bytecode at a time; use multiprocessing for CPU work.
- Sharing mutable state between threads/processes without locks causes race conditions.
- asyncio requires all I/O to be async; mixing blocking I/O (time.sleep, requests) defeats the purpose.
- ProcessPoolExecutor pickles data; objects must be picklable.

---

#### Module 16: Logging — logging module, handlers, formatters, structlog

**Concept:**
Use `logging` module instead of `print()` for production. Control verbosity with log levels (DEBUG, INFO, WARNING, ERROR, CRITICAL). Use handlers and formatters for routing and formatting.

**Example:**
```python
import logging
import logging.handlers
from pathlib import Path

# Basic setup
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s",
)

logger = logging.getLogger(__name__)
logger.info("Application started")
logger.warning("This is a warning")
logger.error("An error occurred")

# Advanced setup with handlers
logger = logging.getLogger("myapp")
logger.setLevel(logging.DEBUG)

# Console handler
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_format = logging.Formatter("%(levelname)s: %(message)s")
console_handler.setFormatter(console_format)

# File handler
file_handler = logging.FileHandler("app.log")
file_handler.setLevel(logging.DEBUG)
file_format = logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s")
file_handler.setFormatter(file_format)

logger.addHandler(console_handler)
logger.addHandler(file_handler)

logger.debug("Debug message (console: no, file: yes)")
logger.info("Info message (console: yes, file: yes)")

# Rotation handler (for large logs)
rotating_handler = logging.handlers.RotatingFileHandler(
    "rotating.log",
    maxBytes=1024 * 1024,  # 1 MB
    backupCount=5,
)
logger.addHandler(rotating_handler)

# Structured logging with structlog (optional, for JSON output)
try:
    import structlog
    structlog.configure()
    struct_logger = structlog.get_logger()
    struct_logger.info("event", user_id=123, action="login")
except ImportError:
    pass
```

**Pitfalls:**
- Using `print()` in libraries; always use logging.
- Not setting log level appropriately; too verbose in production, too quiet in debug.
- Logging sensitive data (passwords, tokens); use formatters to redact.

---

#### Module 17: Stdlib Gems — collections, itertools, datetime, json, csv, sqlite3, subprocess

**Concept:**
The standard library has powerful utilities. Know `Counter`, `deque`, `defaultdict`, `itertools` functions, datetime operations, JSON/CSV handling, and subprocess for running external commands.

**Example:**
```python
from collections import Counter, deque, defaultdict
from itertools import chain, combinations, groupby
from datetime import datetime, timedelta
import json
import csv
import sqlite3
import subprocess

# Counter (frequency of items)
words = "the quick brown fox jumps over the lazy dog".split()
word_freq = Counter(words)
print(word_freq.most_common(3))  # [('the', 2), ...]

# deque (double-ended queue)
queue = deque([1, 2, 3])
queue.appendleft(0)
queue.append(4)
queue.popleft()  # O(1) operations on both ends

# defaultdict (default values for missing keys)
groups = defaultdict(list)
for letter, word in [("a", "apple"), ("a", "ant"), ("b", "bear")]:
    groups[letter].append(word)
# groups = {"a": ["apple", "ant"], "b": ["bear"]}

# itertools.chain (flatten)
nested = [[1, 2], [3, 4], [5, 6]]
flat = list(chain.from_iterable(nested))  # [1, 2, 3, 4, 5, 6]

# itertools.combinations
combos = list(combinations("ABC", 2))  # [('A', 'B'), ('A', 'C'), ('B', 'C')]

# itertools.groupby
data = [1, 1, 2, 2, 2, 3, 1]
for key, group in groupby(data):
    print(f"{key}: {list(group)}")

# datetime operations
now = datetime.now()
tomorrow = now + timedelta(days=1)
print(now.strftime("%Y-%m-%d %H:%M:%S"))

# JSON
data = {"name": "Govind", "scores": [95, 87, 92]}
json_str = json.dumps(data)
loaded = json.loads(json_str)

# CSV
with open("data.csv", "w", newline="") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "Alice", "age": 30})

with open("data.csv", "r") as f:
    reader = csv.DictReader(f)
    for row in reader:
        print(row)

# sqlite3
conn = sqlite3.connect(":memory:")  # In-memory DB
cursor = conn.cursor()
cursor.execute("CREATE TABLE users (id INTEGER, name TEXT)")
cursor.execute("INSERT INTO users VALUES (1, 'Govind')")
conn.commit()

cursor.execute("SELECT * FROM users")
for row in cursor.fetchall():
    print(row)
conn.close()

# subprocess (run external command)
result = subprocess.run(["echo", "Hello, World!"], capture_output=True, text=True)
print(result.stdout)  # Hello, World!
print(result.returncode)  # 0 (success)

# With timeout
try:
    result = subprocess.run(["sleep", "10"], timeout=2)
except subprocess.TimeoutExpired:
    print("Command timed out")
```

**Pitfalls:**
- `json.dumps()` not `json.dump()` for strings; `.dumps()` returns string, `.dump()` writes to file.
- `csv.DictWriter` requires `newline=""` in `open()` to avoid blank rows.
- `subprocess.run()` with `shell=True` is a security risk; avoid.

---

### Part B: Automation & Testing

#### Module 18: requests / httpx — Sync & Async API Clients

**Concept:**
`requests` is the standard HTTP library (blocking). `httpx` is modern, with async support and type hints. Use retries and timeouts to handle flaky networks.

**Example:**
```python
import requests
import httpx
from tenacity import retry, stop_after_attempt, wait_exponential

# Basic requests
response = requests.get("https://api.github.com/users/torvalds")
print(response.status_code)  # 200
print(response.json())  # Dict from JSON

# With headers and auth
headers = {"Accept": "application/vnd.github.v3+json"}
response = requests.get(
    "https://api.github.com/user",
    headers=headers,
    auth=("username", "token"),
)

# POST with JSON
payload = {"title": "Test Issue", "body": "This is a test"}
response = requests.post(
    "https://api.github.com/repos/user/repo/issues",
    json=payload,
    auth=("username", "token"),
)

# Timeout and retries
@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10),
)
def fetch_with_retry(url: str):
    return requests.get(url, timeout=5)

# httpx with async
async def fetch_multiple(urls: list[str]):
    async with httpx.AsyncClient() as client:
        tasks = [client.get(url) for url in urls]
        responses = await asyncio.gather(*tasks)
        return [r.json() for r in responses]

# httpx with retries
from httpx import Client
with httpx.Client() as client:
    response = client.get("https://api.example.com/data")
    print(response.status_code)

# Streaming responses
with requests.get("https://example.com/large-file.bin", stream=True) as r:
    for chunk in r.iter_content(chunk_size=8192):
        process_chunk(chunk)
```

**Pitfalls:**
- Not setting `timeout`; requests can hang indefinitely.
- Not closing sessions; use `with` statement or context managers.
- Mixing sync and async without care; use `requests` for sync, `httpx` for async.

---

#### Module 19: CLI Tools — argparse, click, typer

**Concept:**
Build command-line tools with argument parsing. `argparse` is built-in; `click` and `typer` are more user-friendly.

**Example:**
```python
# argparse (built-in)
import argparse

parser = argparse.ArgumentParser(description="QA automation tool")
parser.add_argument("--host", required=True, help="Server host")
parser.add_argument("--port", type=int, default=8080, help="Server port")
parser.add_argument("--verbose", action="store_true", help="Verbose output")
parser.add_argument("tests", nargs="+", help="Test files to run")

args = parser.parse_args()
print(f"Running tests on {args.host}:{args.port}")

# click (more ergonomic)
import click

@click.command()
@click.option("--host", required=True, help="Server host")
@click.option("--port", type=int, default=8080, help="Server port")
@click.option("--verbose", is_flag=True, help="Verbose output")
@click.argument("tests", nargs=-1, required=True)
def run_tests(host: str, port: int, verbose: bool, tests: tuple):
    """Run QA tests on a server."""
    click.echo(f"Running tests on {host}:{port}")
    for test in tests:
        click.echo(f"  - {test}")

# typer (modern, type-based)
import typer

app = typer.Typer()

@app.command()
def run(
    host: str = typer.Option(..., help="Server host"),
    port: int = typer.Option(8080, help="Server port"),
    verbose: bool = typer.Option(False, help="Verbose output"),
):
    """Run QA tests on a server."""
    typer.echo(f"Running tests on {host}:{port}")

if __name__ == "__main__":
    app()

# Command-line execution
# python script.py --host localhost --port 9000 --verbose test1.py test2.py
```

**Pitfalls:**
- Not handling missing required arguments; argparse will error.
- Not testing CLI parsing; use Click's `CliRunner` for testing.

---

#### Module 20: pytest — Discovery, Assertions, Fixtures, Parametrize

**Concept:**
pytest is the modern testing framework. Tests are functions starting with `test_`. Fixtures provide setup/teardown and dependencies. Parametrize runs the same test with multiple inputs.

**Example:**
```python
# test_example.py
import pytest
from pathlib import Path

def test_simple_assertion():
    assert 2 + 2 == 4

def test_string_matching():
    text = "hello, world"
    assert "world" in text
    assert text.startswith("hello")

def test_exception():
    with pytest.raises(ValueError, match="invalid"):
        raise ValueError("invalid value")

# Fixtures (setup/teardown)
@pytest.fixture
def temp_file(tmp_path):
    """Fixture that provides a temporary file."""
    file = tmp_path / "test.txt"
    file.write_text("test content")
    return file

def test_file_exists(temp_file):
    assert temp_file.exists()
    assert temp_file.read_text() == "test content"

# Fixture scopes
@pytest.fixture(scope="session")
def database():
    """Set up database once per test session."""
    db = setup_database()
    yield db
    teardown_database(db)

def test_db_query(database):
    result = database.query("SELECT * FROM users")
    assert len(result) > 0

# Parametrize (run test multiple times with different inputs)
@pytest.mark.parametrize("input,expected", [
    (2, 4),
    (3, 9),
    (4, 16),
])
def test_square(input, expected):
    assert input ** 2 == expected

# conftest.py (shared fixtures)
# pytest_plugins = ["plugin_name"]
# @pytest.fixture
# def shared_fixture():
#     return "shared"

# Running tests
# pytest test_example.py -v              # Verbose
# pytest test_example.py::test_simple    # Specific test
# pytest test_example.py -k "assertion"  # Keyword filter
# pytest -m "slow"                       # By marker
```

**Pitfalls:**
- Fixtures at function scope are reset per test; session scope persists.
- Order of parametrize arguments must match function signature.
- Test discovery requires `test_` prefix.

---

#### Module 21: pytest Advanced — Markers, xfail/skip, Plugins

**Concept:**
Mark tests (e.g., `@pytest.mark.slow`). Skip or xfail tests conditionally. Use plugins for parallel execution, HTML reports, and behavior-driven testing.

**Example:**
```python
import pytest

# Markers
@pytest.mark.slow
def test_slow_operation():
    import time
    time.sleep(5)

@pytest.mark.integration
def test_api_call():
    pass

# Skip tests
@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    pass

# Skip conditionally
import sys
@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix_feature():
    pass

# xfail (expected to fail)
@pytest.mark.xfail(reason="Known bug in library version 1.0")
def test_known_bug():
    assert False

# pytest configuration (pytest.ini or pyproject.toml)
# [pytest]
# markers =
#     slow: marks tests as slow
#     integration: marks tests as integration tests
# testpaths = tests
# python_files = test_*.py

# Running with markers
# pytest -m "slow"
# pytest -m "not slow"
# pytest --collect-only
# pytest -v --tb=short  # Shorter traceback

# Plugins
# pip install pytest-xdist pytest-html pytest-bdd pytest-cov

# pytest-xdist: parallel execution
# pytest -n auto  # Use all CPUs

# pytest-html: HTML report
# pytest --html=report.html

# pytest-bdd: Gherkin syntax
# from pytest_bdd import given, when, then
# @given("a user is logged in")
# def logged_in_user():
#     return User(name="Alice")

# pytest-cov: coverage report
# pytest --cov=mymodule --cov-report=html
```

**Pitfalls:**
- Not defining markers in config; pytest will warn about unknown marks.
- Using `pytest.mark.skip` in conditional; use `skipif` with condition.
- Mixing xfail and skip; xfail allows test to fail without failing suite.

---

#### Module 22: Mocking — unittest.mock, monkeypatch, responses

**Concept:**
Mock external dependencies (APIs, databases, files) to isolate tests. Use `unittest.mock.Mock`, `monkeypatch` fixture, and `responses` library for HTTP mocking.

**Example:**
```python
from unittest.mock import Mock, patch, MagicMock, call
from unittest.mock import mock_open
import pytest

# Basic Mock
mock_api = Mock()
mock_api.get_user.return_value = {"id": 1, "name": "Alice"}
result = mock_api.get_user(1)
assert result == {"id": 1, "name": "Alice"}
mock_api.get_user.assert_called_once_with(1)

# Mock with side_effect
mock_api.validate.side_effect = [True, False, ValueError("Invalid")]
assert mock_api.validate() is True
assert mock_api.validate() is False
with pytest.raises(ValueError):
    mock_api.validate()

# patch context manager
with patch("requests.get") as mock_get:
    mock_get.return_value.json.return_value = {"status": "ok"}
    # Call code that uses requests.get
    result = fetch_data()
    assert result == {"status": "ok"}

# patch as decorator
@patch("module.external_api")
def test_with_decorator(mock_api):
    mock_api.fetch.return_value = "data"
    assert use_api() == "data"

# monkeypatch fixture (pytest-specific)
def test_monkeypatch(monkeypatch, tmp_path):
    monkeypatch.setenv("API_KEY", "test-key")
    config = load_config()
    assert config.api_key == "test-key"
    
    monkeypatch.setattr("os.path.exists", lambda x: True)
    assert os.path.exists("any_path") is True

# responses library for HTTP mocking
import responses
@responses.activate
def test_http_mock():
    responses.add(
        responses.GET,
        "https://api.example.com/users/1",
        json={"id": 1, "name": "Alice"},
        status=200,
    )
    response = requests.get("https://api.example.com/users/1")
    assert response.json() == {"id": 1, "name": "Alice"}

# Mocking file I/O
@patch("builtins.open", new_callable=mock_open, read_data="test content")
def test_file_read(mock_file):
    with open("test.txt") as f:
        content = f.read()
    assert content == "test content"
    mock_file.assert_called_once_with("test.txt")

# MagicMock (more flexible)
mock_obj = MagicMock()
mock_obj.__getitem__.return_value = "value"
assert mock_obj["key"] == "value"

# Verify call order
mock = Mock()
mock.method_a()
mock.method_b()
assert mock.method_calls == [call.method_a(), call.method_b()]
```

**Pitfalls:**
- Mocking too much; only mock external dependencies, not your code.
- Not resetting mocks between tests; fixtures handle this automatically.
- Patching wrong location; patch where the object is used, not where it's defined.

---

#### Module 23: Playwright Python — Page Objects, Locators, Wait Strategies

**Concept:**
Playwright automates browsers (Chrome, Firefox, Safari). Use page objects to separate locators from logic. Wait for elements instead of sleeps.

**Example:**
```python
# Install: pip install playwright
# python -m playwright install
from playwright.sync_api import sync_playwright, expect

# Basic usage
with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    page.goto("https://example.com")
    
    # Locators
    title = page.locator("h1")
    print(title.text_content())
    
    # Interactions
    search_box = page.locator('input[name="q"]')
    search_box.fill("python automation")
    search_box.press("Enter")
    
    # Wait for element
    page.wait_for_selector('a[href="/python"]')
    
    # Assertions
    expect(page.locator("h1")).to_contain_text("Python")
    
    # Trace recording
    browser.new_context(record_video_dir="videos", record_har_path="trace.har")
    
    browser.close()

# Page Object Model
class LoginPage:
    def __init__(self, page):
        self.page = page
        self.username_input = page.locator('input[name="username"]')
        self.password_input = page.locator('input[name="password"]')
        self.login_button = page.locator('button:has-text("Login")')
    
    def login(self, username: str, password: str):
        self.username_input.fill(username)
        self.password_input.fill(password)
        self.login_button.click()
        self.page.wait_for_load_state("networkidle")

class HomePage:
    def __init__(self, page):
        self.page = page
        self.greeting = page.locator("h1")
    
    def get_greeting(self) -> str:
        return self.greeting.text_content()

# Test using page objects
def test_login_flow():
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto("https://example.com/login")
        
        login_page = LoginPage(page)
        login_page.login("testuser", "password123")
        
        home_page = HomePage(page)
        assert "Welcome" in home_page.get_greeting()
        
        browser.close()

# Async API
import asyncio
from playwright.async_api import async_playwright

async def async_test():
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page()
        await page.goto("https://example.com")
        title = await page.locator("h1").text_content()
        print(title)
        await browser.close()

asyncio.run(async_test())
```

**Pitfalls:**
- Sleeps instead of waits; always use `wait_for_selector()` or `expect()`.
- Hardcoding selectors in tests; use page objects.
- Not closing browser; use `with` or `async with`.

---

#### Module 24: Web Scraping — requests + BeautifulSoup, Playwright

**Concept:**
Scrape static HTML with `requests` + `BeautifulSoup`. Use Playwright for JavaScript-heavy sites. Always respect `robots.txt` and rate limits.

**Example:**
```python
import requests
from bs4 import BeautifulSoup
import time

# Static scraping
url = "https://example.com/products"
response = requests.get(url, headers={"User-Agent": "MyBot/1.0"})
soup = BeautifulSoup(response.content, "html.parser")

# Parse products
products = []
for item in soup.find_all("div", class_="product"):
    title = item.find("h2").text.strip()
    price = item.find("span", class_="price").text.strip()
    products.append({"title": title, "price": price})

# Pagination
def scrape_all_pages(base_url: str):
    results = []
    page = 1
    while True:
        url = f"{base_url}?page={page}"
        response = requests.get(url)
        soup = BeautifulSoup(response.content, "html.parser")
        
        items = soup.find_all("div", class_="item")
        if not items:
            break
        
        for item in items:
            results.append(item.text.strip())
        
        page += 1
        time.sleep(1)  # Rate limit
    
    return results

# Dynamic content with Playwright
from playwright.sync_api import sync_playwright

def scrape_dynamic(url: str):
    with sync_playwright() as p:
        browser = p.chromium.launch()
        page = browser.new_page()
        page.goto(url)
        
        # Wait for JS to render
        page.wait_for_selector(".dynamic-content")
        
        # Extract HTML after JS rendering
        content = page.content()
        soup = BeautifulSoup(content, "html.parser")
        
        results = [item.text for item in soup.find_all(".item")]
        browser.close()
        
        return results

# Respect robots.txt
import urllib.robotparser
def is_allowed(url: str) -> bool:
    rp = urllib.robotparser.RobotFileParser()
    rp.set_url(url.replace("example.com", "example.com/robots.txt"))
    rp.read()
    return rp.can_fetch("MyBot", url)
```

**Pitfalls:**
- Not respecting rate limits; add delays between requests.
- Ignoring robots.txt and TOS; may be illegal.
- Not handling pagination; loop until empty results.
- Using old HTML selectors; inspect live page to find current selectors.

---

#### Module 25: Automation Patterns — Page Objects, Data Factories, Faker

**Concept:**
Separate concerns in test automation. Page objects isolate UI logic. Data factories generate test data. Faker generates realistic synthetic data.

**Example:**
```python
from dataclasses import dataclass
from faker import Faker

# Data factory pattern
@dataclass
class User:
    name: str
    email: str
    age: int

class UserFactory:
    @staticmethod
    def create(name: str = "Alice", email: str = "alice@example.com") -> User:
        return User(name=name, email=email, age=30)
    
    @staticmethod
    def create_batch(count: int) -> list[User]:
        return [UserFactory.create(name=f"User{i}", email=f"user{i}@example.com") for i in range(count)]

# Faker for synthetic data
fake = Faker()

def generate_test_data():
    return {
        "name": fake.name(),
        "email": fake.email(),
        "phone": fake.phone_number(),
        "address": fake.address(),
        "company": fake.company(),
    }

# API testing with factories
class ApiUserFactory:
    @staticmethod
    def create_user(api_client):
        user_data = {
            "username": fake.user_name(),
            "email": fake.email(),
            "password": fake.password(),
        }
        return api_client.post("/users", json=user_data).json()

# Test using factories
def test_user_creation():
    user = UserFactory.create(name="TestUser")
    assert user.name == "TestUser"
    assert user.email == "alice@example.com"

# Bulk test data
users = UserFactory.create_batch(10)
assert len(users) == 10
```

**Pitfalls:**
- Hard-coding test data; use factories to make data reusable.
- Not cleaning up test data; factories should support cleanup.
- Over-parameterizing factories; keep common case simple.

---

#### Module 26: Packaging — pyproject.toml, hatch, poetry; PyPI

**Concept:**
Package Python projects for distribution. Use `pyproject.toml` (PEP 518/517) to declare metadata. `poetry` and `hatch` automate versioning and publishing. Upload to TestPyPI before PyPI.

**Example:**
```python
# pyproject.toml (modern standard)
# [build-system]
# requires = ["poetry-core"]
# build-backend = "poetry.core.masonry.api"
#
# [project]
# name = "qa-toolkit"
# version = "0.1.0"
# description = "QA automation and testing toolkit"
# authors = [{name = "Govind Divekar", email = "govind@example.com"}]
# license = {text = "MIT"}
# requires-python = ">=3.11"
# dependencies = [
#     "requests>=2.28.0",
#     "pytest>=7.0.0",
#     "playwright>=1.30.0",
# ]
#
# [project.optional-dependencies]
# dev = [
#     "black>=23.0.0",
#     "ruff>=0.1.0",
#     "mypy>=1.0.0",
#     "pytest-cov>=4.0.0",
# ]
#
# [project.urls]
# Repository = "https://github.com/user/qa-toolkit"
#
# [tool.poetry.scripts]
# qa-toolkit = "qa_toolkit.cli:main"

# Setup with poetry
# poetry new qa-toolkit
# cd qa-toolkit
# poetry add requests pytest playwright
# poetry add --group dev black ruff mypy pytest-cov
# poetry lock

# Build and publish to TestPyPI
# poetry build
# poetry config repositories.test-pypi https://test.pypi.org/legacy/
# poetry publish -r test-pypi
# poetry publish  # To real PyPI (requires API token)

# .env for PyPI token
# POETRY_PYPI_TOKEN_PYPI=pypi-...

# Entry point (CLI)
# [project.scripts]
# qa-toolkit = "qa_toolkit.cli:main"

# Example CLI module
def main():
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--version", action="version", version="qa-toolkit 0.1.0")
    args = parser.parse_args()

if __name__ == "__main__":
    main()

# After publishing, install from PyPI
# pip install qa-toolkit
```

**Pitfalls:**
- Not using `pyproject.toml`; it's the modern standard.
- Publishing directly to PyPI without testing; use TestPyPI first.
- Forgetting to update version; use semantic versioning (major.minor.patch).

---

### Part C: AI/ML & Advanced

#### Module 27: NumPy — Arrays, Broadcasting, dtypes, Vectorization

**Concept:**
NumPy powers numerical Python. Master n-dimensional arrays, slicing, broadcasting (implicit expansion), and vectorization (avoid Python loops).

**Example:**
```python
import numpy as np

# Create arrays
arr = np.array([1, 2, 3, 4, 5])
matrix = np.array([[1, 2, 3], [4, 5, 6]])
zeros = np.zeros((3, 3))
ones = np.ones((2, 4))
range_arr = np.arange(0, 10, 2)  # [0, 2, 4, 6, 8]
linspace = np.linspace(0, 1, 5)  # 5 evenly spaced values

# Array properties
print(arr.shape)  # (5,)
print(matrix.shape)  # (2, 3)
print(matrix.dtype)  # int64
print(matrix.ndim)  # 2

# Indexing and slicing
print(arr[0])  # 1
print(arr[1:3])  # [2, 3]
print(matrix[0, :])  # [1, 2, 3]
print(matrix[:, 1])  # [2, 5]

# Boolean indexing
mask = arr > 2
print(arr[mask])  # [3, 4, 5]

# Reshaping
reshaped = arr.reshape(5, 1)
flattened = matrix.flatten()

# Broadcasting (implicit expansion)
a = np.array([1, 2, 3])  # Shape (3,)
b = np.array([[1], [2], [3]])  # Shape (3, 1)
result = a + b  # Shape (3, 3) — b broadcasts to match a
# [[2, 3, 4], [3, 4, 5], [4, 5, 6]]

# Vectorized operations (fast)
x = np.arange(1000000)
result = x ** 2 + 2 * x + 1  # 1 million operations in C

# Avoid loops
# Bad: Python loop
slow = np.array([i**2 for i in range(1000)])
# Good: vectorized
fast = np.arange(1000) ** 2

# Statistical functions
data = np.array([1, 2, 3, 4, 5])
print(np.mean(data))  # 3.0
print(np.std(data))   # 1.41...
print(np.min(data))   # 1
print(np.max(data))   # 5

# Matrix operations
A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])
print(np.dot(A, B))  # Matrix multiplication
print(A @ B)  # @ operator also works
```

**Pitfalls:**
- Confusing element-wise operations with matrix operations; use `@` for matrix multiply.
- Not understanding broadcasting; leads to unexpected shapes.
- Creating arrays in loops; pre-allocate with `zeros` and fill.

---

#### Module 28: pandas — Series, DataFrame, indexing, groupby, merge, datetime

**Concept:**
pandas is the data manipulation library. DataFrames are table-like structures. Master `.loc`/`.iloc`, `.groupby()`, `.merge()`, and datetime handling.

**Example:**
```python
import pandas as pd
import numpy as np

# Series (1-D labeled array)
s = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
print(s['a'])  # 1
print(s > 2)   # Boolean series

# DataFrame (2-D table)
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [30, 25, 35],
    'salary': [50000, 60000, 70000],
})

# Indexing: .loc (label-based), .iloc (position-based)
print(df.loc[0, 'name'])  # 'Alice'
print(df.iloc[0, 0])      # 'Alice'
print(df.loc[df['age'] > 25])  # Rows where age > 25

# Column selection
print(df['name'])  # Series
print(df[['name', 'age']])  # DataFrame

# Adding columns
df['bonus'] = df['salary'] * 0.1
df['dept'] = pd.Categorical(['Sales', 'IT', 'HR'])

# Missing data
df['email'] = [None, 'bob@example.com', 'charlie@example.com']
print(df.isna())
df.fillna('unknown')
df.dropna()

# Apply functions
df['salary_grade'] = df['salary'].apply(lambda x: 'High' if x > 60000 else 'Low')

# groupby
grouped = df.groupby('dept')['salary'].agg(['mean', 'sum', 'count'])

# merge (join)
df2 = pd.DataFrame({'name': ['Alice', 'Bob'], 'bonus_pct': [0.1, 0.15]})
merged = df.merge(df2, on='name', how='left')

# Datetime
df['hire_date'] = pd.to_datetime(['2020-01-15', '2021-03-20', '2019-06-10'])
df['year'] = df['hire_date'].dt.year
df.sort_values('hire_date')

# CSV I/O
df.to_csv('data.csv', index=False)
df_loaded = pd.read_csv('data.csv')

# Parquet (efficient binary format)
df.to_parquet('data.parquet')
df_parquet = pd.read_parquet('data.parquet')

# Statistics
print(df.describe())
print(df.corr())
```

**Pitfalls:**
- Confusing `.loc` and `.iloc`; `.loc` uses labels, `.iloc` uses integer positions.
- Modifying a copy by accident; use `.copy()` to avoid SettingWithCopyWarning.
- Not converting to datetime; string dates don't enable date operations.

---

#### Module 29: Visualization — Matplotlib, Seaborn, Plotly

**Concept:**
Visualize data with plotting libraries. Matplotlib is low-level; Seaborn is high-level; Plotly is interactive.

**Example:**
```python
import matplotlib.pyplot as plt
import seaborn as sns
import plotly.express as px
import pandas as pd

# Matplotlib basics
x = [1, 2, 3, 4, 5]
y = [2, 4, 6, 8, 10]
plt.plot(x, y, marker='o')
plt.xlabel('X Label')
plt.ylabel('Y Label')
plt.title('Line Plot')
plt.show()

# Seaborn (statistical plots)
df = pd.DataFrame({
    'group': ['A', 'A', 'B', 'B', 'C', 'C'],
    'value': [10, 12, 15, 18, 20, 22],
})

sns.set_style("darkgrid")
plt.figure(figsize=(8, 6))
sns.barplot(data=df, x='group', y='value')
plt.show()

# Subplots
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
axes[0].plot([1, 2, 3], [1, 4, 9])
axes[1].scatter([1, 2, 3], [1, 4, 9])
plt.show()

# Scatter plot with Seaborn
df = pd.DataFrame({
    'x': np.random.randn(100),
    'y': np.random.randn(100),
    'category': np.random.choice(['A', 'B', 'C'], 100),
})
sns.scatterplot(data=df, x='x', y='y', hue='category')
plt.show()

# Plotly (interactive)
fig = px.scatter(
    df,
    x='x', y='y',
    color='category',
    title='Interactive Scatter Plot',
    hover_data={'x': ':.2f', 'y': ':.2f'},
)
fig.show()

# Histogram
plt.hist([1, 2, 2, 3, 3, 3, 4, 4, 5], bins=5)
plt.show()

# Box plot
sns.boxplot(data=df, y='y')
plt.show()
```

**Pitfalls:**
- Not setting figure size; use `figsize=(width, height)`.
- Overlapping plots without `plt.show()` between them.
- Using Matplotlib when Plotly would be clearer (interactive is often better).

---

#### Module 30: scikit-learn — Datasets, Pipelines, Estimators, CV, Tuning

**Concept:**
scikit-learn is the standard ML library. Learn train_test_split, Pipelines, estimators, cross-validation, and hyperparameter tuning.

**Example:**
```python
from sklearn import datasets
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

# Load dataset
iris = datasets.load_iris()
X, y = iris.data, iris.target

# Train-test split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Simple model
model = LogisticRegression(max_iter=200)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)
print(f"Accuracy: {accuracy_score(y_test, y_pred)}")

# Pipeline (preprocessor + model)
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', LogisticRegression(max_iter=200)),
])
pipeline.fit(X_train, y_train)
y_pred = pipeline.predict(X_test)

# Cross-validation
scores = cross_val_score(pipeline, X, y, cv=5, scoring='accuracy')
print(f"CV Scores: {scores.mean():.3f} +/- {scores.std():.3f}")

# GridSearchCV (hyperparameter tuning)
param_grid = {
    'classifier__C': [0.1, 1, 10],
    'classifier__penalty': ['l2'],
}
grid_search = GridSearchCV(pipeline, param_grid, cv=5)
grid_search.fit(X_train, y_train)
print(f"Best params: {grid_search.best_params_}")
print(f"Best score: {grid_search.best_score_}")

# Evaluate best model
y_pred = grid_search.predict(X_test)
print(f"Precision: {precision_score(y_test, y_pred, average='weighted')}")
print(f"Recall: {recall_score(y_test, y_pred, average='weighted')}")
print(f"F1: {f1_score(y_test, y_pred, average='weighted')}")

# Feature importance (for tree-based models)
rf_model = RandomForestClassifier(n_estimators=100)
rf_model.fit(X_train, y_train)
importances = rf_model.feature_importances_
print(f"Feature importances: {importances}")
```

**Pitfalls:**
- Not scaling features; use `StandardScaler` or `MinMaxScaler`.
- Leaking test data during preprocessing; put preprocessing in Pipeline.
- Not using cross-validation for reliable estimates; always validate.

---

#### Module 31: PyTorch Essentials — Tensors, autograd, nn.Module, DataLoader, GPU

**Concept:**
PyTorch is a deep learning framework. Learn tensor operations, automatic differentiation, building neural networks with `nn.Module`, and GPU usage.

**Example:**
```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# Tensors (GPU-accelerated arrays)
x = torch.tensor([1.0, 2.0, 3.0])
y = torch.tensor([[1.0, 2.0], [3.0, 4.0]])
print(x.shape)  # torch.Size([3])
print(y.shape)  # torch.Size([2, 2])

# GPU device
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
x = x.to(device)

# Automatic differentiation
x = torch.tensor([2.0], requires_grad=True)
y = x ** 2 + 3 * x + 1
y.backward()
print(x.grad)  # dy/dx = 2*x + 3 = 7.0

# Building a neural network
class SimpleNet(nn.Module):
    def __init__(self, input_size: int, hidden_size: int, output_size: int):
        super().__init__()
        self.fc1 = nn.Linear(input_size, hidden_size)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_size, output_size)
    
    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x

# Training loop
model = SimpleNet(input_size=10, hidden_size=64, output_size=2).to(device)
criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)

# Dummy data
X_train = torch.randn(100, 10)
y_train = torch.randint(0, 2, (100,))
dataset = TensorDataset(X_train, y_train)
loader = DataLoader(dataset, batch_size=32)

# Training
for epoch in range(10):
    for batch_X, batch_y in loader:
        batch_X, batch_y = batch_X.to(device), batch_y.to(device)
        
        # Forward pass
        outputs = model(batch_X)
        loss = criterion(outputs, batch_y)
        
        # Backward pass
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
    
    print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# Save and load weights
torch.save(model.state_dict(), "model.pth")
model.load_state_dict(torch.load("model.pth"))
```

**Pitfalls:**
- Not calling `optimizer.zero_grad()` before backward; gradients accumulate.
- Not moving data/model to device; GPU training requires `.to(device)`.
- Not setting `model.eval()` during inference; dropout and batch norm behave differently.

---

#### Module 32: TensorFlow/Keras — Intro, Comparison with PyTorch

**Concept:**
TensorFlow is another deep learning framework with Keras API. Know when to choose TensorFlow (production, distributed) vs PyTorch (research, flexibility).

**Example:**
```python
import tensorflow as tf
from tensorflow import keras

# Sequential API (simple models)
model = keras.Sequential([
    keras.layers.Dense(64, activation='relu', input_shape=(10,)),
    keras.layers.Dense(32, activation='relu'),
    keras.layers.Dense(2, activation='softmax'),
])

model.compile(
    optimizer='adam',
    loss='sparse_categorical_crossentropy',
    metrics=['accuracy'],
)

# Dummy data
X_train = tf.random.normal((100, 10))
y_train = tf.random.uniform((100,), minval=0, maxval=2, dtype=tf.int32)

# Training
model.fit(X_train, y_train, epochs=10, batch_size=32)

# Functional API (complex models)
inputs = keras.Input(shape=(10,))
x = keras.layers.Dense(64, activation='relu')(inputs)
x = keras.layers.Dense(32, activation='relu')(x)
outputs = keras.layers.Dense(2, activation='softmax')(x)
model_func = keras.Model(inputs=inputs, outputs=outputs)

# TensorFlow vs PyTorch
# TensorFlow: Production-focused, distributed training, mobile deployment
# PyTorch: Research-focused, dynamic computation graphs, easier debugging
```

**Pitfalls:**
- Mixing TensorFlow 1.x and 2.x syntax; use 2.x (eager execution).
- Not normalizing input data; TensorFlow is sensitive to scale.

---

#### Module 33: Hugging Face Transformers & Datasets

**Concept:**
Hugging Face provides pretrained transformers and datasets. Load models, tokenizers, and fine-tune on custom tasks.

**Example:**
```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification, pipeline
from datasets import load_dataset

# Pipeline (high-level API)
classifier = pipeline("sentiment-analysis", model="distilbert-base-uncased-finetuned-sst-2-english")
result = classifier("I love this!")
print(result)  # [{'label': 'POSITIVE', 'score': 0.999...}]

# Load model and tokenizer
model_name = "distilbert-base-uncased"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name, num_labels=2)

# Tokenization
text = "I love this product!"
inputs = tokenizer(text, return_tensors="pt")
# {'input_ids': tensor([[...]]),'attention_mask': tensor([[...]])}

# Forward pass
outputs = model(**inputs)
logits = outputs.logits

# Load dataset
dataset = load_dataset("sst2")
print(dataset["train"][0])  # {'sentence': '...', 'label': 0}

# Inference on batch
texts = ["I love this!", "This is terrible."]
inputs = tokenizer(texts, padding=True, truncation=True, return_tensors="pt")
outputs = model(**inputs)
predictions = outputs.logits.argmax(dim=-1)
```

**Pitfalls:**
- Loading large models without GPU; use smaller models or cloud inference.
- Not tokenizing correctly; always use the model's tokenizer.

---

#### Module 34: LangChain vs LlamaIndex vs Direct APIs; RAG Patterns

**Concept:**
LangChain and LlamaIndex abstract LLM workflows. Direct APIs (OpenAI, Ollama) give more control. Choose based on complexity.

**Example:**
```python
# Direct OpenAI API (simple)
import openai

openai.api_key = "sk-..."
response = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is Python?"},
    ],
)
print(response.choices[0].message.content)

# LangChain (for chains and memory)
from langchain.llms import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate

llm = OpenAI(api_key="sk-...")
prompt = PromptTemplate(
    input_variables=["language"],
    template="Explain {language} in 50 words.",
)
chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run("Python")

# LlamaIndex (for document indexing and retrieval)
from llama_index import VectorStoreIndex, SimpleDirectoryReader

documents = SimpleDirectoryReader("./data").load_data()
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine()
response = query_engine.query("What is automation?")

# Ollama (local inference)
# ollama pull mistral
# ollama serve
import requests

response = requests.post(
    "http://localhost:11434/api/generate",
    json={"model": "mistral", "prompt": "What is Python?"},
)
# Streaming response

# RAG (Retrieval-Augmented Generation)
# 1. Load documents
# 2. Embed and store in vector DB
# 3. Query: retrieve relevant docs, pass to LLM
```

**Pitfalls:**
- Over-engineering with LangChain for simple tasks; use direct API.
- Not chunking documents for RAG; large docs lose relevance.
- Using closed-source APIs for production; consider open models.

---

#### Module 35: Embeddings & Vector Search — sentence-transformers, FAISS

**Concept:**
Embeddings convert text to vectors. Use sentence-transformers for semantic similarity. FAISS indexes embeddings for fast search.

**Example:**
```python
from sentence_transformers import SentenceTransformer
import faiss
import numpy as np

# Load embedding model
model = SentenceTransformer('all-MiniLM-L6-v2')

# Embed documents
documents = [
    "Python is a programming language",
    "Machine learning is a subset of AI",
    "Automation testing is important",
    "Neural networks learn from data",
]
embeddings = model.encode(documents, convert_to_numpy=True)
print(embeddings.shape)  # (4, 384)

# FAISS index
dimension = embeddings.shape[1]
index = faiss.IndexFlatL2(dimension)
index.add(embeddings.astype('float32'))

# Query
query = "What is machine learning?"
query_embedding = model.encode([query], convert_to_numpy=True)
distances, indices = index.search(query_embedding.astype('float32'), k=2)
print(f"Top 2 documents: {[documents[i] for i in indices[0]]}")

# Cosine similarity
from sklearn.metrics.pairwise import cosine_similarity
similarity = cosine_similarity(query_embedding, embeddings)[0]
top_idx = np.argsort(similarity)[::-1][:2]
print(f"Most similar: {[documents[i] for i in top_idx]}")
```

**Pitfalls:**
- Using L2 distance instead of cosine for embeddings; use IndexFlatIP with normalized vectors.
- Mixing embedding models in production; stick to one.
- Not re-embedding when updating documents; rebuild index.

---

#### Module 36: Model Evaluation — sklearn metrics, HF evaluate

**Concept:**
Evaluate models with appropriate metrics. Classification: precision/recall/F1. Regression: RMSE/MAE. NLP: BLEU/ROUGE.

**Example:**
```python
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    confusion_matrix, roc_auc_score, classification_report
)
from evaluate import load

# Classification metrics
y_true = [0, 1, 1, 0, 1]
y_pred = [0, 1, 0, 0, 1]

print(f"Accuracy: {accuracy_score(y_true, y_pred)}")
print(f"Precision: {precision_score(y_true, y_pred)}")
print(f"Recall: {recall_score(y_true, y_pred)}")
print(f"F1: {f1_score(y_true, y_pred)}")
print(f"Confusion Matrix:\n{confusion_matrix(y_true, y_pred)}")
print(classification_report(y_true, y_pred))

# HF evaluate
accuracy = load("accuracy")
results = accuracy.compute(predictions=y_pred, references=y_true)
print(results)

# NLP metrics
bleu = load("bleu")
predictions = ["the cat is on the mat"]
references = [["the cat is on the mat"]]
results = bleu.compute(predictions=predictions, references=references)
```

**Pitfalls:**
- Using accuracy for imbalanced data; use F1 or ROC-AUC.
- Evaluating on training data only; always use held-out test set.
- Optimizing for wrong metric; understand business requirements first.

---

#### Module 37: Notebook vs Script — Jupyter, nbconvert, papermill

**Concept:**
Jupyter notebooks are for exploration and presentation. Convert to .py for production. Parametrize notebooks with papermill.

**Example:**
```python
# Running Jupyter
# pip install jupyter
# jupyter notebook
# or
# jupyter lab

# Convert notebook to script
# jupyter nbconvert --to script notebook.ipynb
# Outputs: notebook.py

# Convert to HTML report
# jupyter nbconvert --to html notebook.ipynb

# papermill (parametrize notebooks)
# pip install papermill
# papermill input.ipynb output.ipynb -p param1 value1 -p param2 value2

# In notebook, declare parameters in first cell with #parameters comment
# # parameters
# model_path = "model.pkl"
# batch_size = 32

# Run from Python
import papermill as pm
pm.execute_notebook(
    "input.ipynb",
    "output.ipynb",
    parameters={"model_path": "new_model.pkl", "batch_size": 64}
)

# Best practices
# Production: Use scripts (.py)
# Exploration: Use notebooks (.ipynb)
# Reports: Use notebooks, convert to HTML
# Automation: Use papermill or nbconvert
```

**Pitfalls:**
- Mixing exploration and production code; separate concerns.
- Not committing notebooks to git cleanly; use nbstripout to remove outputs.
- Using notebooks as source of truth; scripts are more maintainable.

---

#### Module 38: Productionizing — FastAPI, pydantic, Model Serving

**Concept:**
Serve models via REST API. FastAPI is modern and fast. Pydantic validates request/response schemas. OpenAPI docs auto-generated.

**Example:**
```python
from fastapi import FastAPI
from pydantic import BaseModel
import pickle
import numpy as np

# Load model (assume trained sklearn model)
with open("model.pkl", "rb") as f:
    model = pickle.load(f)

# Define request/response schemas
class PredictionRequest(BaseModel):
    features: list[float]

class PredictionResponse(BaseModel):
    prediction: float
    confidence: float

app = FastAPI(title="Model API", version="1.0.0")

@app.get("/health")
def health():
    return {"status": "ok"}

@app.post("/predict", response_model=PredictionResponse)
def predict(request: PredictionRequest):
    X = np.array([request.features])
    prediction = model.predict(X)[0]
    proba = model.predict_proba(X)[0].max()
    return PredictionResponse(prediction=float(prediction), confidence=float(proba))

# Run: uvicorn script:app --reload
# Docs: http://localhost:8000/docs
# OpenAPI schema: http://localhost:8000/openapi.json

# Testing
from fastapi.testclient import TestClient

client = TestClient(app)

def test_predict():
    response = client.post("/predict", json={"features": [1.0, 2.0, 3.0]})
    assert response.status_code == 200
    data = response.json()
    assert "prediction" in data
    assert "confidence" in data

def test_health():
    response = client.get("/health")
    assert response.status_code == 200
```

**Pitfalls:**
- Not validating input; Pydantic handles this automatically.
- Blocking I/O in endpoints; use async functions for better concurrency.
- Not testing the API; use TestClient.

---

## 4. Exercises

### Beginner (15 CodeWars-style katas)

1. Sum of digits
2. Count vowels
3. Find duplicate in list
4. Reverse string
5. Check palindrome
6. Filter even numbers
7. Flatten nested list
8. Find max element
9. Remove duplicates
10. Convert Celsius to Fahrenheit
11. Check if list is sorted
12. Merge two sorted lists
13. Calculate factorial
14. Count character frequency
15. Find common elements in two lists

### Intermediate

**(a) Build an API client library with httpx + retries + typed responses**
- Implement `APIClient` class with:
  - Base URL configuration
  - Automatic retries (exponential backoff)
  - Type-hinted response parsing
  - Support for GET, POST, PUT, DELETE
  - Context manager support
- Test with a public API (e.g., GitHub, JSONPlaceholder)
- Publish as package to TestPyPI

**(b) Write a pytest plugin that records slow tests**
- Create plugin that:
  - Hooks into test execution
  - Measures test duration
  - Logs tests slower than threshold (default 1s)
  - Outputs summary report
- Test the plugin on a test suite

**(c) Use pandas to clean a messy CSV (missing data, types)**
- Load a real-world CSV with:
  - Missing values (NaN, "", empty)
  - Mixed types (age as string, etc.)
  - Duplicates
  - Outliers
- Clean and validate data:
  - Fill missing values appropriately
  - Convert types
  - Remove duplicates
  - Document transformations
- Export cleaned CSV and summary report

**(d) Train a scikit-learn classifier on Titanic; cross-validate; tune with GridSearch**
- Load Titanic dataset
- Exploratory analysis
- Preprocessing pipeline
- Train RandomForestClassifier
- Cross-validate (5-fold)
- GridSearchCV for tuning
- Evaluate on holdout test set
- Interpret feature importances

### Advanced

**(a) Implement a small MNIST CNN in PyTorch and hit >98% test accuracy**
- Load MNIST dataset
- Build CNN with Conv2D, pooling, FC layers
- Training loop with validation
- Achieve >98% test accuracy
- Save/load weights
- Visualize misclassifications

**(b) Build a local sentence-transformers + FAISS RAG over a folder of PDFs**
- Read PDFs from folder
- Chunk documents (overlap-aware)
- Embed with sentence-transformers
- Index with FAISS
- Query interface
- Retrieve and format results

**(c) Async scraper that pulls 100 pages with rate limiting + retries**
- Async HTTP requests
- Rate limiting (delays between requests)
- Exponential backoff retries
- Concurrent batch processing
- Save results to JSON
- Error logging

**(d) FastAPI service serving a sklearn model with pydantic, OpenAPI, integration tests**
- Load pretrained model
- Define request/response schemas
- Implement `/predict` endpoint
- Add `/health` endpoint
- OpenAPI auto-docs
- Write integration tests with TestClient
- Docker containerization

### Capstone: "QA Automation & AI Toolkit" Package

Build a production-ready Python package with:

**(i) Typer CLI**
- Commands: `run-api-tests`, `run-ui-tests`, `generate-cases`, `health-check`
- Help text, subcommands, options
- Exit codes for CI integration

**(ii) Playwright-based UI smoke runner**
- Page object models
- Parametrized test data
- Wait strategies
- Screenshot/video on failure
- HTML report

**(iii) pytest-based API test runner**
- Fixtures for API client
- Request/response validation with pydantic
- Parametrized test cases
- HTML report with pytest-html

**(iv) LLM-powered test-case generator**
- Ollama (local) or OpenAI API
- Prompt engineering for edge cases
- Generate pytest test functions
- Save to file or database

**(v) CI on GitHub Actions**
- Lint (ruff, mypy)
- Test (pytest)
- Build package
- Publish to TestPyPI on tag

**(vi) Published to TestPyPI**
- Semantic versioning
- README with usage examples
- Installation: `pip install qa-automation-toolkit`

---

## 5. Additional Reading & Resources

### Books
- **Fluent Python** (Luciano Ramalho) — Mastery of idioms, decorators, descriptors, async
- **Python Cookbook** (David Beazley & Brian K. Jones) — Recipes for common patterns
- **Hands-On ML with Scikit-Learn, Keras, and TensorFlow** (Aurélien Géron) — ML workflows end-to-end

### Online
- [docs.python.org](https://docs.python.org/3/) — Official Python docs
- [Real Python](https://realpython.com) — In-depth tutorials
- [Corey Schafer YouTube](https://www.youtube.com/@CoreySchafer) — Comprehensive Python tutorials
- [Arjan Codes YouTube](https://www.youtube.com/@ArjanCodes) — Advanced patterns and best practices
- [Official pytest docs](https://docs.pytest.org/) — Testing framework
- [PyTorch tutorials](https://pytorch.org/tutorials/) — Deep learning
- [Full Stack Deep Learning](https://fullstackdeeplearning.com/) — ML in production

### Tooling
- [PyPI Package Index](https://pypi.org/) — Package repository
- [Python Packaging Guide](https://packaging.python.org/) — Packaging best practices
- [GitHub Actions Documentation](https://docs.github.com/en/actions) — CI/CD

---

## 6. Index

- **asyncio** — Module 15, async/await, gather, Queue
- **dataclasses** — Module 11, @dataclass, frozen, field
- **decorators** — Module 13, @wraps, @lru_cache, custom decorators
- **error handling** — Module 10, try/except/else/finally, exception chaining
- **FastAPI** — Module 38, REST API, pydantic, OpenAPI
- **fixtures** — Module 20, pytest fixtures, scope
- **generators** — Module 12, yield, generator expressions
- **imports** — Module 8, relative imports, packages, __name__
- **list comprehension** — Module 4, syntax, nested
- **logging** — Module 16, handlers, formatters, structlog
- **multiprocessing** — Module 15, CPU-bound parallelism
- **OOP** — Module 11, classes, inheritance, ABC, dunder methods
- **pandas** — Module 28, DataFrame, groupby, merge, datetime
- **Playwright** — Module 23, page objects, wait strategies, locators
- **pytest** — Modules 20–22, fixtures, parametrize, mocking, plugins
- **PyTorch** — Module 31, tensors, autograd, nn.Module, DataLoader
- **requests** — Module 18, HTTP client, retries, headers
- **scikit-learn** — Module 30, pipelines, cross-validation, GridSearchCV
- **type hints** — Module 7, Optional, Union, Callable, Generic
- **typing** — Module 7, List, Dict, Tuple, Type aliases
- **virtual environments** — Module 2, venv, poetry, uv, lockfiles

---

## 7. References

1. Python Enhancement Proposals (PEPs):
   - PEP 8: Style Guide for Python Code
   - PEP 484: Type Hints
   - PEP 492: Coroutines with async and await
   - PEP 517: A build-system independent format
   - PEP 604: Complementary Type Union Syntax

2. Official Documentation:
   - Python Standard Library: [docs.python.org/3/library/](https://docs.python.org/3/library/)
   - NumPy: [numpy.org](https://numpy.org)
   - pandas: [pandas.pydata.org](https://pandas.pydata.org)
   - scikit-learn: [scikit-learn.org](https://scikit-learn.org)
   - PyTorch: [pytorch.org](https://pytorch.org)
   - FastAPI: [fastapi.tiangolo.com](https://fastapi.tiangolo.com)
   - pytest: [pytest.org](https://docs.pytest.org/)
   - Playwright: [playwright.dev/python/](https://playwright.dev/python/)

3. Community:
   - Stack Overflow: #python, #python-unittest, #pytest, #pytorch
   - Reddit: r/learnprogramming, r/Python, r/MachineLearning
   - Discord: Python Discord, PyTorch, Hugging Face

4. Video Channels:
   - Corey Schafer: Python fundamentals, OOP, decorators
   - Arjan Codes: Design patterns, async, testing
   - Andreas Mueller: scikit-learn tutorials
   - Jeremy Howard: Fast.ai Deep Learning (practical)

---

**End of Learning Plan**

Estimated total time: 80–100 hours over 6 weeks (12–15 hours/week). Adjust pace based on experience and project deadlines. Focus on exercises and capstone to solidify learning.
