# Testing

## Locust for load testing

What is Locust?

Open-source load testing tool written in Python
Allows simulating thousands of concurrent users
Web interface for real-time monitoring
Supports distributed testing for larger scale.\
Once it's a microservice that handles with thousands requests, the load test is required.


# locustfile.py example
```python
from locust import HttpUser, task, between

class FastAPIUser(HttpUser):
    # Time between requests (1-5 seconds)
    wait_time = between(1, 5)
    
    # Base URL
    host = "http://localhost:5590"

    @task
    def get_data(self):
        # endpoint GET /data
        self.client.get("/data")

    @task
    def create_data(self):
        # endpoint POST /data
        payload = {
            "value": "test data"
        }
        self.client.post("/data", json=payload)
```

To run
```bash
$pip install locust
$locust -f locustfile.py
```

We also have unit tests with a initial 60% coverage.
Check the pyproject settings below:
```toml
[tool.pytest.ini_options]
addopts = "--cov=. --cov-report term-missing:skip-covered --cov-report xml:coverage.xml --cov-fail-under 60 --cov-config .coveragerc -p no:warnings"
python_files = "*tests.py *test_*.py *_tests.py"
```
It will force the **CI** fail if the coverage percentage was not met.

Strategy for the future:
- Add  e2e tests
- Add Integration tests starting with endpoints
- Increase unit tests coverage
