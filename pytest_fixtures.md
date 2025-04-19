# 🧪 Pytest Fixtures Cheat Sheet

## 📌 What are Fixtures?

Fixtures in `pytest` are used to set up and tear down test dependencies. They help you write clean, modular, and reusable test setup code.

---

## ✅ Basic Fixture

```python
import pytest

@pytest.fixture
def sample_user():
    return {"username": "alice", "email": "alice@example.com"}

def test_user_has_email(sample_user):
    assert "email" in sample_user
```

---

## 🔁 How Fixtures Work

- Run once per test by default.
- Isolated across tests unless otherwise scoped.

---

## 🧼 Setup and Teardown with `yield`

```python
@pytest.fixture
def db():
    conn = connect_to_database()
    yield conn
    conn.close()
```

- Code before `yield`: setup
- Code after `yield`: teardown

---

## 🧬 Fixture Scope

Control how often fixtures are run.

```python
@pytest.fixture(scope="function")  # Default
@pytest.fixture(scope="module")
@pytest.fixture(scope="class")
@pytest.fixture(scope="session")
```

| Scope     | Applies To             |
|-----------|------------------------|
| function  | Each test function     |
| class     | Once per test class    |
| module    | Once per module        |
| session   | Once per entire test run |

---

## ⚙️ Composing Fixtures

Fixtures can depend on other fixtures:

```python
@pytest.fixture
def base_url():
    return "https://api.example.com"

@pytest.fixture
def user_api(base_url):
    return f"{base_url}/users"
```

---

## 🔂 Parametrized Fixtures

Run the same fixture with different values:

```python
@pytest.fixture(params=[1, 2, 3])
def number(request):
    return request.param

def test_even(number):
    assert number % 2 == 0
```

---

## 🔄 Autouse Fixtures

Automatically use a fixture in every test:

```python
@pytest.fixture(autouse=True)
def run_before_every_test():
    print("Runs before every test")
```

---

## 🧠 Accessing Fixture Metadata

Use the `request` object to inspect or interact with test data.

```python
@pytest.fixture
def info(request):
    print(f"Running: {request.function.__name__}")
    yield
```

---

## 🧩 Global Fixtures with `conftest.py`

Define fixtures globally by placing them in `conftest.py` in the test directory.

```python
# conftest.py
import pytest

@pytest.fixture
def default_user():
    return {"name": "Alice"}
```

Use it in any test without imports.

---

## 🔥 Real-World Use Cases

### Temp File
```python
import tempfile

@pytest.fixture
def temp_file():
    with tempfile.NamedTemporaryFile(delete=False) as f:
        f.write(b"Hello")
        return f.name
```

---

### API Mocking
```python
from unittest.mock import patch

@pytest.fixture
def mock_requests():
    with patch("requests.get") as mock:
        yield mock
```

---

## 📝 Summary Table

| Feature              | Use Case                              |
|----------------------|----------------------------------------|
| `@pytest.fixture`     | Declare a reusable setup block         |
| `yield`              | Add teardown logic                     |
| `scope`              | Change how often it runs               |
| `params=[...]`       | Run with multiple inputs               |
| `autouse=True`       | Always run, no need to declare         |
| `conftest.py`        | Global fixtures, no imports needed     |
| `request`            | Access test metadata in fixtures       |
