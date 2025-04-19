# Understanding Mocks and Test Doubles in Python

This document provides a comprehensive overview of mocking and test doubles in Python, with a focus on the `unittest.mock` library.

## Test Doubles: Mocks vs. Stubs

Test doubles are objects that replace real dependencies during testing. The two most common types are:

### Stubs

**Purpose**: Provide canned answers to calls made during the test.

**Characteristics**:
- Focus on state verification (input → output)
- Passive - simply return pre-programmed responses
- No verification of how they were used
- Used when you just need to simulate certain return values

**Example**:
```python
# Simple stub implementation
class DatabaseStub:
    def get_user(self, user_id):
        return {"id": user_id, "name": "Test User"}
```

### Mocks

**Purpose**: Record interactions and verify they happened correctly.

**Characteristics**:
- Focus on behavior verification (interactions)
- Active - track calls and verify expectations
- Verify method calls, arguments, and call frequency
- Used when testing interactions between components

**Example**:
```python
# Using unittest.mock
with patch('module.Database') as mock_db:
    mock_db.get_user.return_value = {"id": 123, "name": "Test User"}
    
    system_under_test.process_user(123)
    
    # Verify interaction
    mock_db.get_user.assert_called_once_with(123)
```

## Python's `unittest.mock` Library

The `unittest.mock` library provides a powerful implementation that can function as both a stub and a mock.

### Key Components

1. **Mock Objects**: Generic mock objects that can replace any Python object
2. **patch()**: A decorator or context manager for replacing objects with mocks

### Basic Example

```python
import requests

def get_weather(city):
    response = requests.get(f"https://api.weather.com/{city}")
    return response.json()

# Test with mock
from unittest.mock import patch

@patch('requests.get')
def test_get_weather(mock_get):
    # Stub aspect - provide canned responses
    mock_response = mock_get.return_value
    mock_response.json.return_value = {'temp': 25}
    
    result = get_weather("London")
    
    # State verification
    assert result == {'temp': 25}
    
    # Mock aspect - verify interactions
    mock_get.assert_called_once_with("https://api.weather.com/London")
```

### Important Mock Attributes and Methods

When working with mock objects, these are the most commonly used attributes and methods:

#### Configuration
- `mock_obj.return_value`: Sets what the mock returns when called
- `mock_obj.side_effect`: Sets function to call or exception to raise when called

#### Tracking Calls
- `mock_obj.called`: Boolean indicating if mock was called
- `mock_obj.call_count`: Number of times the mock was called
- `mock_obj.call_args`: Arguments from the last call
- `mock_obj.call_args_list`: List of arguments from all calls

#### Assertion Methods
- `mock_obj.assert_called()`: Verify called at least once
- `mock_obj.assert_called_once()`: Verify called exactly once
- `mock_obj.assert_called_with(*args, **kwargs)`: Verify last call args
- `mock_obj.assert_called_once_with(*args, **kwargs)`: Verify called once with args
- `mock_obj.assert_not_called()`: Verify never called
- `mock_obj.assert_has_calls(calls)`: Verify sequence of calls

## Best Practices for Using Mocks

1. **Mock at the right level**: Mock at the boundaries of your system (external APIs, databases)
2. **Use proper import paths**: When using `@patch()`, target where the dependency is looked up
3. **Avoid over-mocking**: Don't mock everything; focus on external dependencies
4. **Reset mocks when needed**: Use `reset_mock()` if reusing mocks across tests

## Common Pitfalls

### Incorrect Patch Path
```python
# weather.py
import requests
def get_weather(city): 
    return requests.get(f"https://api.weather.com/{city}").json()

# test_weather.py - INCORRECT
@patch('requests.get')  # Won't work!
def test_wrong(mock_get):
    # ...

# test_weather.py - CORRECT
@patch('weather.requests.get')  # Patch where it's imported
def test_correct(mock_get):
    # ...
```

### Mocking Chain of Method Calls
```python
# Setting up a chain of method calls
mock_obj.method.return_value.another_method.return_value = 'result'

# More readable with an intermediate variable
mock_result = mock_obj.method.return_value
mock_result.another_method.return_value = 'result'
```

## When to Use Mocks vs. Stubs

- **Use stubs** when you only care about controlling indirect inputs to your test
- **Use mocks** when you need to verify how your code interacts with its dependencies
- In practice with `unittest.mock`, you often use both aspects together

By understanding the differences between mocks and stubs, you can write more effective and targeted tests that focus on exactly what needs to be tested.
