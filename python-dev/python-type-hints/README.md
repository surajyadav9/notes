# Python Type Hinting, Checking, and Data Validation

Python is not a statically typed language, meaning you do not need to specify data types when creating variables or functions, and these types can change throughout your script. While this flexibility is a benefit, it can also lead to bugs that are difficult to track down. To manage this, developers use **type hinting, type checking, and data validation**—three concepts that work with types but serve entirely different purposes. 

## 1. Type Hinting (Annotations)
**Type hinting is the process of adding expected type information to variables, function parameters, and return values**. 

*   **Benefits:** It creates **self-documenting code** that anyone can read and understand immediately, and it provides your IDE with autocomplete suggestions. 
*   **Drawbacks/Trade-offs:** It can sometimes be excessive or redundant (such as hinting a variable that is obviously an f-string). If you update a function's return type, you must also manually update the type hint for any variables expecting that return value.
*   **Runtime Behavior:** **Python completely ignores type hints at runtime**. They are strictly metadata. If you assign the wrong data type, Python will not throw an error or warn you at runtime.

```python
# Example of type hinting function parameters and a return type
def create_user(first_name: str, last_name: str, age: int) -> dict:
    return {"first_name": first_name, "last_name": last_name, "age": age}

# Example of type hinting a variable assignment
user: dict = create_user("John", "Doe", 38)
```
*Note: If you call `create_user` with a string for the age (e.g., `"thirty-eight"`), the code will still run perfectly fine without throwing any errors from Python*.

## 2. Type Checking
**Type checking is a static analysis process that occurs *before* your code runs** to ensure you are sticking to your specified types. 

*   **Tools:** This is not built into Python. It requires an external tool like **MyPy** or **Ty** (from Astral), which can be integrated directly into IDEs like VS Code as an extension.
*   **Runtime Behavior:** It helps catch mismatched types early in development, but **it does not prevent your code from executing**. If your IDE flags an error, you can still run the script successfully. Furthermore, static type checkers cannot guard against dynamic data (like APIs or user input) because they don't know what that data will contain until runtime.

```python
# If using a type checker like MyPy, the following line triggers a warning in your IDE:
# "create_user has incompatible type str, expected int"
user: dict = create_user("John", "Doe", "38")

# However, Python will STILL execute this code without crashing.
```

## 3. Data Validation
**Data validation happens at runtime and actively verifies that data meets specific requirements**. 

*   **Capabilities:** It goes beyond basic type checking. It can validate values, formatting (like valid email addresses), and ranges. If validation fails, it raises an error and stops bad data from entering your application. 
*   **Use Case:** It is highly recommended for **dynamic, external data** (e.g., API payloads, user input, environment variables). It is usually unnecessary for simple local scripts with hard-coded inputs.
*   **Modern Tools:** While validation used to be written manually, modern libraries like **Pydantic** (used extensively by frameworks like FastAPI) utilize your existing type hints to validate data automatically. Pydantic can even **coerce and cast data**—such as automatically converting the string `"38"` into the integer `38`.

### Manual Validation Example
Before modern tools, developers wrote boilerplate code to check types and raise errors manually.
```python
def create_user(first_name: str, last_name: str, age: int) -> dict:
    # Manual validation checking the instance type
    if not isinstance(age, int):
        raise TypeError("age must be an integer")
    return {"first_name": first_name, "last_name": last_name, "age": age}
```

### Pydantic Validation Example
With Pydantic, you can use a decorator to validate the inputs automatically based on the function's type hints.
```python
from pydantic import validate_call

# The validate_call decorator handles the boilerplate validation automatically
@validate_call
def create_user(first_name: str, last_name: str, age: int) -> dict:
    return {"first_name": first_name, "last_name": last_name, "age": age}

# Passing completely invalid data will now raise a detailed "ValidationError" at runtime.
```

## Recommended Best Practices
1.  **Use type hints moving forward.** You don't have to overhaul old codebases completely; just add hints gradually function by function as you work.
2.  **Use an IDE-integrated type checker** (like MyPy) to get early warnings as you write your code.
3.  **Do not go overboard with data validation.** Reserve strict data validation (like Pydantic) specifically for guarding your application against **external sources** and critical mistakes.


<br><br>
# Python Type Hints: From Basic Annotations to Advanced Generics

Type hints allow you to add type information to your Python code, which helps you write **self-documenting code, catch bugs earlier, and get better IDE completions**. Python itself does not enforce these types at runtime; to catch errors, you must use a dedicated type checker like **MyPy**. 

### Basic Variables and Functions
While you can add type hints to simple variables, it is generally unnecessary unless the variable is complex or its type is not immediately obvious. Type hinting is most beneficial when applied to function parameters and return values.

```python
# Simple variable annotation
age: int = 38

# Function parameter and return type annotations
def create_user(first_name: str, last_name: str) -> dict:
    return {"first_name": first_name, "last_name": last_name}
```

### Optionals and Unions
When an argument is optional and defaults to `None`, you should use a **union** to indicate it can be multiple types. In newer Python versions, this is done using the pipe character (`|`). In older code (prior to Python 3.10), you will see this written using `Optional` from the `typing` module.

```python
from typing import Optional

# Older syntax
def create_user(first_name: str, age: Optional[int] = None) -> dict: ...

# Modern syntax
def create_user(first_name: str, age: int | None = None) -> dict: ...
```

### Type Aliases and `NewType`
When dealing with complex dictionary return types, the annotations can become cluttered. You can simplify your code by creating a **type alias**. Python 3.12 introduced the `type` keyword specifically for this purpose. 

If you have data types that share the exact same underlying structure but mean entirely different things—such as RGB colors and HSL colors both being represented as tuples of three integers—you should use **`NewType`**. This prevents developers from accidentally passing an HSL value into a function expecting an RGB value, as the type checker will flag it.

```python
from typing import NewType

# Type Alias (Python 3.12+ syntax)
type User = dict[str, str | int | None]

# NewType definition
RGB = NewType('RGB', tuple[int, int, int])
HSL = NewType('HSL', tuple[int, int, int])

def set_color(color: RGB | None = None):
    print(f"Color set to: {color}")

# Explicitly cast the tuple to the RGB NewType
set_color(RGB((255, 255, 255)))
```

### `TypedDict` and Dataclasses
A standard dictionary type alias applies type checking broadly to any value in the dictionary. If you need to specify exact types for specific keys, you should inherit from **`TypedDict`**. If you are creating objects from scratch rather than relying on standard dictionaries, it is often better to use a **`dataclass`**, which utilizes type hints to automatically generate methods like `__init__`.

```python
from typing import TypedDict
from dataclasses import dataclass

# TypedDict for checking specific dictionary keys
class UserDict(TypedDict):
    first_name: str
    last_name: str
    age: int | None

# USAGE: Create it exactly like a normal dictionary
my_user_dict: UserDict = {
    "first_name": "John",
    "last_name": "Doe",
    "age": 30
}

# Dataclass approach for creating objects
@dataclass
class User:
    first_name: str
    last_name: str
    age: int | None = None

# USAGE: Create it using class syntax (instantiation)
my_user_obj = User(
    first_name="Jane", 
    last_name="Doe", 
    age=28
)

# Access values using dot notation
print(my_user_obj.first_name)

# Because 'age' has a default value of None in the declaration, you can omit it when creating the object:
another_user = User(first_name="Bob", last_name="Smith")
print(another_user.age) # Outputs: None
```

**When to choose which?** If you are working with existing dictionary data (like parsing a JSON API response), TypedDict is excellent for adding strict type checking to those dictionaries. If you are building objects from scratch and want full class functionality (like dot notation access or the ability to add methods later), @dataclass is usually the better approach.

### Generics (`Any` vs. `TypeVar`)
When writing reusable helper functions (like choosing a random item from a list), you want the function to accept different types. Using the `Any` generic works, but it causes the IDE to lose track of the object's type, breaking autocomplete. 

```python
from typing import Any
import random

# Using 'Any' to indicate the list can contain items of any type,
# and the function will return a value of any type.
def random_choice(items: list[Any]) -> Any:
    return random.choice(items)

# Creating lists of different types
my_users = [{"name": "Corey"}, {"name": "Jane"}]
my_emails = ["a@b.com", "c@d.com"]

# Both of these will work without MyPy throwing errors
random_user = random_choice(my_users)
random_email = random_choice(my_emails)
```

Instead, you should use **`TypeVar`** (or the new Python 3.12 generics syntax). This explicitly connects the input type to the output type, ensuring that if you pass a list of strings, the type checker knows a string is returned.

```python
from typing import TypeVar

# Older generic syntax using TypeVar
T = TypeVar('T')

def random_choice(items: list[T]) -> T:
    return random.choice(items)

# USAGE:
my_emails = ["a@b.com", "c@d.com"]
my_integers = [4,5,6,7]

# The type checker sees a list of strings goes in, so a string must come out.
random_email = random_choice(my_emails) 
print(random_email.upper()) # IDE successfully autocompletes string methods!

# The type checker sees a list of ints goes in, so an int must come out.
random_number = random_choice(my_integers)

# Python 3.12+ Generic Syntax
def random_choice[T](items: list[T]) -> T: ...
```

### Third-Party Stubs
Many third-party libraries (such as `requests`) do not have built-in type hints, meaning MyPy will treat their types as `Any`. To fix this and restore type checking capabilities, you need to install **stub packages**, which usually follow the naming convention `types-<package_name>`.

```bash
# Installing stubs for the requests library
pip install types-requests
```

### Best Practices
When introducing type hints to your workflow:
1. **Start small**: You don't have to type-hint everything at once. Add them gradually to new functions or critical parts of your codebase.
2. **Inputs should be generic**: Try to accept the most flexible types possible (e.g., an iterable instead of a strict list).
3. **Outputs should be specific**: Provide detailed, structured return types (like moving from untyped dictionaries to a `dataclass`) so the rest of your application knows exactly what data it is working with.