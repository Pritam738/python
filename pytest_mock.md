# Mocking in pytest: A Comprehensive Guide

This guide covers how to effectively use mocking in pytest for testing code that depends on external services or APIs.

## Introduction to Mocking in pytest

When testing code that makes API calls or interacts with external services, mocking allows you to replace those external dependencies with controlled test doubles. pytest offers two main approaches to mocking:

1. Using Python's built-in `unittest.mock` library
2. Using the `pytest-mock` plugin which provides a `mocker` fixture

## Installing pytest-mock

```bash
pip install pytest-mock
```

## Example: Mocking an Internal API Call

Let's work with a typical example where a service calls an internal API:

```python
# app/weather_service.py
import requests

class WeatherService:
    def __init__(self, api_url="https://internal-api.company.com"):
        self.api_url = api_url
    
    def get_forecast(self, city):
        """Get weather forecast by calling our internal weather API"""
        url = f"{self.api_url}/weather/{city}"
        response = requests.get(url, headers={"Authorization": "Bearer secret-token"})
        if response.status_code == 200:
            return response.json()
        else:
            response.raise_for_status()

# app/weather_app.py
from app.weather_service import WeatherService

def get_temperature(city):
    """Get just the temperature for a city"""
    service = WeatherService()
    forecast = service.get_forecast(city)
    return forecast.get("temperature")
```

## Approach 1: Using pytest-mock

The `pytest-mock` plugin provides a `mocker` fixture that integrates well with pytest:

```python
# tests/test_weather_app.py
import pytest
from app.weather_app import get_temperature

def test_get_temperature(mocker):
    # Create a mock for the WeatherService.get_forecast method
    mock_get_forecast = mocker.patch('app.weather_service.WeatherService.get_forecast')
    
    # Define what the mock should return when called
    mock_get_forecast.return_value = {"temperature": 25, "conditions": "sunny"}
    
    # Call the function we're testing
    result = get_temperature("London")
    
    # Assert the result matches what we expect
    assert result == 25
    
    # Verify the mock was called with the correct argument
    mock_get_forecast.assert_called_once_with("London")
```

### Key Features of mocker.patch

- `mocker.patch()` replaces the specified object with a mock during the test
- After the test, the original object is automatically restored
- The mock has built-in tracking of calls and arguments
- Assertion methods like `assert_called_once_with()` verify the mock was used correctly

## Approach 2: Using unittest.mock Directly

You can also use Python's built-in `unittest.mock` library:

```python
# tests/test_weather_app_unittest.py
import pytest
from unittest.mock import patch
from app.weather_app import get_temperature

@patch('app.weather_service.WeatherService.get_forecast')
def test_get_temperature_unittest(mock_get_forecast):
    # Define what the mock should return when called
    mock_get_forecast.return_value = {"temperature": 25, "conditions": "sunny"}
    
    # Call the function we're testing
    result = get_temperature("London")
    
    # Assert the result matches what we expect
    assert result == 25
    
    # Verify the mock was called with the correct argument
    mock_get_forecast.assert_called_once_with("London")
```

## Mocking at Different Levels

### Mocking a Method in Your Own Class

As shown above, you can mock specific methods:

```python
mocker.patch('app.weather_service.WeatherService.get_forecast')
```

### Mocking the HTTP Library (requests)

Often it's better to mock the external library that makes the actual calls:

```python
# tests/test_weather_service.py
import pytest
from app.weather_service import WeatherService

def test_get_forecast(mocker):
    # Create a mock for requests.get
    mock_response = mocker.Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = {"temperature": 25, "conditions": "sunny"}
    mock_get = mocker.patch('requests.get', return_value=mock_response)
    
    # Use the actual service but with mocked requests
    service = WeatherService()
    result = service.get_forecast("London")
    
    # Verify the result
    assert result == {"temperature": 25, "conditions": "sunny"}
    
    # Verify that requests.get was called with the right parameters
    mock_get.assert_called_once_with(
        "https://internal-api.company.com/weather/London", 
        headers={"Authorization": "Bearer secret-token"}
    )
```

## Reusable Mocks with pytest Fixtures

Fixtures allow you to reuse mocks across multiple tests:

```python
# tests/test_with_fixtures.py
import pytest
from app.weather_app import get_temperature

@pytest.fixture
def mock_weather_service(mocker):
    """Fixture that mocks the WeatherService"""
    mock = mocker.patch('app.weather_service.WeatherService.get_forecast')
    mock.return_value = {"temperature": 25, "conditions": "sunny"}
    return mock

def test_get_temperature_with_fixture(mock_weather_service):
    # The mock is already set up by the fixture
    result = get_temperature("London")
    assert result == 25
    mock_weather_service.assert_called_once_with("London")
```

## Advanced Mocking Techniques

### Loading Mock Responses from Files

For complex API responses, you can load them from JSON files:

```python
# tests/test_complex_response.py
import json
import os
import pytest
from app.weather_service import WeatherService

def test_get_forecast_complex(mocker):
    # Load mock response from JSON file
    fixture_path = os.path.join(os.path.dirname(__file__), 'fixtures', 'weather_response.json')
    with open(fixture_path) as f:
        mock_data = json.load(f)
    
    # Set up the mock
    mock_response = mocker.Mock()
    mock_response.status_code = 200
    mock_response.json.return_value = mock_data
    mocker.patch('requests.get', return_value=mock_response)
    
    # Test with the mock
    service = WeatherService()
    result = service.get_forecast("London")
    
    # Assertions
    assert result["temperature"] == mock_data["temperature"]
```

### Mocking Multiple Calls with Different Responses

You can use `side_effect` to return different values on different calls:

```python
def test_multiple_calls(mocker):
    # Create responses for different cities
    london_data = {"temperature": 20, "conditions": "rainy"}
    paris_data = {"temperature": 25, "conditions": "sunny"}
    
    # Create mock response objects
    london_response = mocker.Mock()
    london_response.status_code = 200
    london_response.json.return_value = london_data
    
    paris_response = mocker.Mock()
    paris_response.status_code = 200
    paris_response.json.return_value = paris_data
    
    # Configure requests.get to return different responses based on the URL
    def get_side_effect(url, **kwargs):
        if "London" in url:
            return london_response
        elif "Paris" in url:
            return paris_response
        raise ValueError(f"Unexpected URL: {url}")
    
    mock_get = mocker.patch('requests.get', side_effect=get_side_effect)
    
    # Test with different cities
    service = WeatherService()
    london_result = service.get_forecast("London")
    paris_result = service.get_forecast("Paris")
    
    assert london_result["temperature"] == 20
    assert paris_result["temperature"] == 25
    assert mock_get.call_count == 2
```

### Simulating Errors and Exceptions

You can configure your mock to raise exceptions:

```python
def test_api_error(mocker):
    # Set up the mock to raise an exception
    mock_response = mocker.Mock()
    mock_response.status_code = 404
    mock_response.raise_for_status.side_effect = requests.exceptions.HTTPError("404 Not Found")
    mocker.patch('requests.get', return_value=mock_response)
    
    # Test error handling
    service = WeatherService()
    with pytest.raises(requests.exceptions.HTTPError):
        service.get_forecast("NonExistentCity")
```

## Comparing unittest.mock and pytest-mock

| Feature | unittest.mock | pytest-mock |
|---------|--------------|------------|
| Integration with pytest | Requires manually patching | Integrates with pytest fixtures |
| Cleanup | Manual or using context managers | Automatic after test completion |
| Syntax | Uses decorators or context managers | Uses the `mocker` fixture |
| Scope control | Limited | Can use pytest fixture scopes |

## Best Practices for Mocking in pytest

1. **Mock at the right level**: Usually at the boundaries of your system
2. **Use descriptive names** for your mocks and fixtures
3. **Be specific with assertions**: Verify calls with `assert_called_with` rather than just checking `called`
4. **Don't over-mock**: Only mock what's necessary for the test
5. **Reset mocks when needed**: Use `reset_mock()` if reusing mocks
6. **Mock the import path**: Make sure to patch the object where it's imported, not where it's defined
7. **Use fixtures** for reusable mock setup

## Troubleshooting Common Issues

### Mock Not Being Used

If your patched object isn't being used, check:
- The import path in `patch()` matches where the code looks up the object
- The mock is being created before the code that uses it runs

```python
# INCORRECT: patching in the wrong place
@patch('requests.get')  # This won't work if get_temperature imports from weather_service
def test_incorrect(mock_get):
    result = get_temperature("London")  # Uses the real requests.get!

# CORRECT: patch where the object is looked up
@patch('app.weather_service.requests.get')
def test_correct(mock_get):
    result = get_temperature("London")  # Uses the mock
```

### Complex Method Chains

For mocking chains of method calls, use intermediate variables for clarity:

```python
# Hard to read:
mock_get.return_value.json.return_value = {"temperature": 25}

# More readable:
mock_response = mock_get.return_value
mock_response.json.return_value = {"temperature": 25}
```

By following these techniques and best practices, you can effectively use mocking in pytest to test code that interacts with APIs and external services.
