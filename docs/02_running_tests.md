
# Running Tests

Zuspec projects typically have Python unit tests. You can check by looking for tests/unit. 

Zuspec projects contain a Python virtual environment in packages/python. You can run
tests using the following command:

```
% PYTHONPATH=$(pwd)/src ./packages/python/bin/pytest -s tests/unit
```

You can change the test specification from "tests/unit" to focus on specific tests.
