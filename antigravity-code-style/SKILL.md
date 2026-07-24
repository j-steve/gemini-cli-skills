---
name: antigravity-code-style
description: Generic code writing and styling guidelines for Antigravity AI.
---

# Antigravity Code Writing Guidelines

Follow these guidelines when writing, refactoring, or modifying code in any programming language.

## 1. Do Not Create Unnecessary Intermediary Variables
Avoid creating temporary variables to alias simple, direct expressions (such as attribute lookups, dictionary lookups, or method calls that are already short and highly readable on their own).
- Do not assign a short expression to a local variable if the expression can be used directly without compromising clarity.
- Avoid split assignment-and-copy patterns (e.g., creating a local variable to hold a dictionary or list just to call `.copy()` or `.model_dump()` on it on the next line). Chain the calls directly.

**Bad:**
```python
raw_states = response.json()
return {item["id"]: item for item in raw_states}

original_runbook = state.get("runbook", {})
clean_runbook = original_runbook.copy()
```

**Good:**
```python
return {item["id"]: item for item in response.json()}

clean_runbook = state.get("runbook", {}).copy()
```


*Exceptions:*
- Assigning to a variable to give a snappy name to a very long-winded expression to keep lines short and readable.
- Assigning to a variable to correct a poorly-named or misleading upstream function/method (e.g., renaming `getJaspter()` to `price`).

## 2. Order Classes, Methods, and Functions Sequentially
Classes, functions, and methods in a file should be ordered sequentially from top to bottom based on dependencies and call sequence:
- Define parent entities, primary models, and public APIs first at the top of the file.
- Subordinate classes, helper models, private helpers, or data sub-structures referenced by the parent entity must be listed immediately *after* it.
- A public method/function should be defined first.
- Private helper functions or subordinate methods called by the public method should be listed immediately *after* it.
- If `method1` calls `method2` and then `method3`, the order of definition in the file should be:
  1. `method1`
  2. `method2`
  3. `method3`

## 3. Handle Exceptions Cleanly (Never Log and Throw)
When an exception occurs, you must EITHER log it OR throw/propagate it. Never do both.
- Logging and throwing creates duplicate, noisy log trails for a single error.
- If you throw/propagate the exception, the upstream caller or global handler is responsible for logging it.
- If you catch the exception and log it, handle it gracefully and return an appropriate default (or raise a different, higher-level error, but do not log the original error if you are wrapping/re-raising it).

## 4. Define Constants at the Class or File Level
Constants must be defined in `ALL_CAPS`.
- Place them outside of methods, either at the class level or the file level (global).
- By default, constants should be private (e.g., prefixing them with an underscore in Python, like `_MAX_RETRIES` or `_DEFAULT_TIMEOUT`), unless there is an explicit need for them to be public.
