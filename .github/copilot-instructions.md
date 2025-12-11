# Python Code Review Guidelines for GitHub Copilot

This document provides comprehensive instructions for GitHub Copilot when reviewing Python code in this repository.  All code suggestions, reviews, and generated code must adhere to these standards.

---

## Table of Contents

1. [Code Formatting and Spacing Rules](#code-formatting-and-spacing-rules)
2. [Variable and Constant Placement](#variable-and-constant-placement)
3. [Naming Conventions](#naming-conventions)
4. [Function Design](#function-design)
5. [DRY Principle](#dry-principle)
6. [General Python Best Practices](#general-python-best-practices)
7. [Code Review Checklist](#code-review-checklist)

---

## Code Formatting and Spacing Rules

### 1. Blank Lines Before Control Flow Statements

**REQUIRED:** Add a blank line before `if`, `for`, `while`, and `try` blocks to improve readability.

```python
# ✅ CORRECT
def process_data(items):
    result = []

    for item in items:
        processed = transform(item)

        if processed. is_valid:
            result. append(processed)

    return result


# ❌ INCORRECT
def process_data(items):
    result = []
    for item in items:
        processed = transform(item)
        if processed.is_valid:
            result.append(processed)
    return result
```

### 2. No Blank Lines Before Continuation Blocks

**REQUIRED:** Do NOT add blank lines before `else`, `elif`, `except`, and `finally` blocks. They should immediately follow the closing of the previous block.

```python
# ✅ CORRECT
def fetch_data(url):

    try:
        response = requests.get(url)
    except ConnectionError:
        log_error("Connection failed")
        return None
    except TimeoutError:
        log_error("Request timed out")
        return None
    finally:
        cleanup_resources()

    if response.status_code == 200:
        return response. json()
    elif response.status_code == 404:
        return None
    else:
        raise ApiError(response.status_code)


# ❌ INCORRECT
def fetch_data(url):

    try:
        response = requests.get(url)

    except ConnectionError:
        log_error("Connection failed")
        return None

    except TimeoutError:
        log_error("Request timed out")
        return None

    finally:
        cleanup_resources()

    if response.status_code == 200:
        return response.json()

    elif response.status_code == 404:
        return None

    else:
        raise ApiError(response.status_code)
```

---

## Variable and Constant Placement

### 3. Constants and Global Variables at File Beginning

**REQUIRED:** All constants and global variables must be defined at the beginning of the file, immediately after imports.

```python
# ✅ CORRECT
import os
import logging
from typing import List, Dict

# Constants
MAX_RETRY_ATTEMPTS = 3
DEFAULT_TIMEOUT_SECONDS = 30
API_BASE_URL = "https://api.example.com"
SUPPORTED_FILE_EXTENSIONS = [". py", ".json", ".yaml"]

# Global configuration
logger = logging.getLogger(__name__)


def process_request(endpoint: str) -> Dict:
    """Process an API request."""
    full_url = f"{API_BASE_URL}/{endpoint}"
    # ... implementation


# ❌ INCORRECT
import os
import logging


def process_request(endpoint: str):
    MAX_RETRY_ATTEMPTS = 3  # Should be at file level
    API_BASE_URL = "https://api.example.com"  # Should be at file level
    full_url = f"{API_BASE_URL}/{endpoint}"
    # ... implementation
```

### Constant Naming Convention

- Use `UPPER_SNAKE_CASE` for all constants
- Group related constants together with comments

```python
# Database Configuration
DATABASE_HOST = "localhost"
DATABASE_PORT = 5432
DATABASE_NAME = "production_db"
DATABASE_POOL_SIZE = 10

# API Configuration
API_VERSION = "v2"
API_RATE_LIMIT = 100
API_TIMEOUT_SECONDS = 60
```

---

## Naming Conventions

### 4. Meaningful Variable Names

**REQUIRED:** Variable names must be meaningful and descriptive. The following are strictly prohibited:

- Single character variable names (except for well-established conventions like `i` in simple loops, which should still be avoided when possible)
- Variable names containing numbers
- Abbreviations that are not universally understood

```python
# ✅ CORRECT
def calculate_order_total(order_items:  List[OrderItem]) -> float:
    """Calculate the total price for all items in an order."""
    subtotal = 0.0
    discount_amount = 0.0

    for order_item in order_items: 
        item_price = order_item.unit_price * order_item.quantity
        subtotal += item_price

        if order_item.has_discount:
            discount_amount += calculate_item_discount(order_item)

    total_price = subtotal - discount_amount
    return total_price


# ❌ INCORRECT
def calc(items):
    t = 0.0
    d = 0.0

    for i in items:
        p = i.price * i.qty
        t += p

        if i.disc:
            d += calc_disc(i)

    t2 = t - d  # Using numbers in variable names
    return t2
```

### Variable Naming Guidelines

| Type | Convention | Example |
|------|------------|---------|
| Local variables | `snake_case` | `user_count`, `total_price` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_CONNECTIONS`, `API_KEY` |
| Class names | `PascalCase` | `UserAccount`, `DataProcessor` |
| Private variables | `_snake_case` | `_internal_cache`, `_connection_pool` |
| Protected variables | `__snake_case` | `__secret_key` |

---

## Function Design

### 5. Meaningful Function Names

**REQUIRED:** Function names must clearly describe what the function does.  The name should answer "What does this function do?"

```python
# ✅ CORRECT
def validate_user_credentials(username: str, password: str) -> bool:
    """Validate user login credentials against the database."""
    pass


def calculate_shipping_cost(weight: float, destination: str) -> float:
    """Calculate shipping cost based on package weight and destination."""
    pass


def fetch_user_profile_by_email(email: str) -> Optional[UserProfile]:
    """Retrieve a user profile from the database using their email address."""
    pass


def convert_temperature_celsius_to_fahrenheit(celsius: float) -> float:
    """Convert a temperature value from Celsius to Fahrenheit."""
    pass


# ❌ INCORRECT
def validate(u, p):  # Unclear what is being validated
    pass


def calc(w, d):  # What is being calculated?
    pass


def get_data(e):  # What data? From where?
    pass


def convert(c):  # Convert what to what?
    pass
```

### Function Design Principles

1. **Single Responsibility:** Each function should do one thing and do it well
2. **Verb-first naming:** Start function names with action verbs (`get_`, `calculate_`, `validate_`, `process_`, `fetch_`, `create_`, `update_`, `delete_`)
3. **Consistent naming patterns:** Use consistent prefixes across the codebase
   - `get_*` / `fetch_*` - Retrieve data
   - `create_*` / `build_*` - Create new objects
   - `update_*` / `modify_*` - Modify existing data
   - `delete_*` / `remove_*` - Remove data
   - `validate_*` / `check_*` - Validation functions
   - `calculate_*` / `compute_*` - Calculations
   - `is_*` / `has_*` / `can_*` - Boolean checks

---

## DRY Principle

### 6. Don't Repeat Yourself (DRY)

**REQUIRED:** Code must follow the DRY principle.  Duplicate code should be refactored into reusable functions, classes, or modules.

```python
# ✅ CORRECT - Reusable function for common logic
def format_api_response(data: Dict, status: str, message: str) -> Dict:
    """Format a standardized API response."""
    return {
        "status": status,
        "message": message,
        "data": data,
        "timestamp": datetime.utcnow().isoformat()
    }


def get_user_endpoint(user_id: int) -> Dict:
    user_data = fetch_user_by_id(user_id)
    return format_api_response(user_data, "success", "User retrieved successfully")


def get_orders_endpoint(user_id:  int) -> Dict:
    orders_data = fetch_orders_by_user(user_id)
    return format_api_response(orders_data, "success", "Orders retrieved successfully")


# ❌ INCORRECT - Repeated code
def get_user_endpoint(user_id: int) -> Dict:
    user_data = fetch_user_by_id(user_id)
    return {
        "status": "success",
        "message": "User retrieved successfully",
        "data": user_data,
        "timestamp":  datetime.utcnow().isoformat()
    }


def get_orders_endpoint(user_id: int) -> Dict:
    orders_data = fetch_orders_by_user(user_id)
    return {
        "status": "success",
        "message": "Orders retrieved successfully",
        "data": orders_data,
        "timestamp": datetime.utcnow().isoformat()  # Same structure repeated! 
    }
```

### DRY Strategies

1. **Extract common logic into functions**
2. **Use base classes for shared behavior**
3. **Create utility modules for reusable operations**
4. **Use decorators for cross-cutting concerns**
5. **Leverage configuration files for repeated values**

```python
# Using decorators to avoid repeating error handling
def handle_api_errors(func):
    """Decorator to handle common API errors."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs):

        try:
            return func(*args, **kwargs)
        except ConnectionError:
            logger.error("Connection failed")
            return {"status": "error", "message": "Service unavailable"}
        except ValidationError as validation_error:
            logger.warning(f"Validation failed: {validation_error}")
            return {"status": "error", "message": str(validation_error)}

    return wrapper


@handle_api_errors
def create_user(user_data: Dict) -> Dict:
    # No need to repeat error handling here
    validated_data = validate_user_data(user_data)
    return user_repository.create(validated_data)


@handle_api_errors
def update_user(user_id:  int, user_data: Dict) -> Dict:
    # Same error handling applied automatically
    validated_data = validate_user_data(user_data)
    return user_repository.update(user_id, validated_data)
```

---

## General Python Best Practices

### Docstrings

All functions and classes must have docstrings:

```python
def calculate_compound_interest(
    principal: float,
    annual_rate: float,
    years: int,
    compounds_per_year: int = 12
) -> float:
    """
    Calculate compound interest for a given principal amount. 

    Args:
        principal:  The initial investment amount.
        annual_rate: The annual interest rate (as a decimal, e.g., 0.05 for 5%).
        years: The number of years for the investment.
        compounds_per_year: How many times interest compounds per year.

    Returns:
        The final amount after compound interest is applied.

    Raises:
        ValueError:  If principal or years is negative. 

    Example:
        >>> calculate_compound_interest(1000, 0.05, 10)
        1647.01
    """

    if principal < 0 or years < 0:
        raise ValueError("Principal and years must be non-negative")

    return principal * (1 + annual_rate / compounds_per_year) ** (compounds_per_year * years)
```

### Error Handling

Use specific exceptions and provide meaningful error messages:

```python
class UserNotFoundError(Exception):
    """Raised when a user cannot be found in the database."""
    pass


class InvalidCredentialsError(Exception):
    """Raised when user credentials are invalid."""
    pass


def authenticate_user(username: str, password: str) -> User:
    """Authenticate a user and return their profile."""
    user = user_repository.find_by_username(username)

    if user is None:
        raise UserNotFoundError(f"No user found with username: {username}")

    if not verify_password(password, user.password_hash):
        raise InvalidCredentialsError("Invalid password provided")

    return user
```

---

## Code Review Checklist

When reviewing Python code, verify the following:

### Formatting
- [ ] Blank line exists before `if`, `for`, `while`, and `try` blocks
- [ ] No blank line before `else`, `elif`, `except`, and `finally` blocks
- [ ] Consistent indentation (4 spaces)
- [ ] Lines do not exceed 100 characters

### Variables and Constants
- [ ] All constants are defined at the beginning of the file
- [ ] Constants use `UPPER_SNAKE_CASE`
- [ ] No single-character variable names
- [ ] No numbers in variable names
- [ ] Variable names are meaningful and self-documenting

### Functions
- [ ] Function names clearly describe their purpose
- [ ] Functions follow single responsibility principle
- [ ] Docstrings are present and complete

### DRY Compliance
- [ ] No duplicated code blocks
- [ ] Common logic is extracted into reusable functions
- [ ] Utility functions are used where appropriate
- [ ] Configuration values are centralized

### General Quality
- [ ] Proper error handling with specific exceptions
- [ ] Appropriate logging is implemented
- [ ] No hardcoded values (use constants instead)
- [ ] Code is testable and modular

---

## Summary of Rules

| Rule # | Description |
|--------|-------------|
| 1 | Add blank lines before `if`, `for`, `while`, and `try` blocks |
| 2 | No blank lines before `else`, `elif`, `except`, and `finally` blocks |
| 3 | Constants and global variables at file beginning |
| 4 | Meaningful variable names (no single chars, no numbers) |
| 5 | Function names must describe their task |
| 6 | Follow DRY principle - no code duplication |

---

*These guidelines ensure consistent, readable, and maintainable Python code across the project.*
