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
- If a helper method/function is called by multiple other methods (e.g., Method A and Method C both call Method B), define all caller methods first, and define the shared helper method afterwards.
  - E.g., if Method A and Method C call Method B, define them in the order:
    1. Method A (caller)
    2. Method C (caller)
    3. Method B (callee helper)

> [!CRITICAL]
> **Caller-Before-Callee Audit Step**:
> Before completing any file edit or refactoring, ALWAYS inspect line numbers of functions in the modified file. Verify that any helper function (e.g., `_parse_timestamp`, `_helper_fn`) is placed strictly **BELOW** its caller function (`get_...`, `fetch_...`). Never leave a helper function defined above its caller.

## 3. Handle Exceptions Cleanly (Never Log and Throw)
When an exception occurs, you must EITHER log it OR throw/propagate it. Never do both.
- Logging and throwing creates duplicate, noisy log trails for a single error.
- If you throw/propagate the exception, the upstream caller or global handler is responsible for logging it.
- If you catch the exception and log it, handle it gracefully and return an appropriate default (or raise a different, higher-level error, but do not log the original error if you are wrapping/re-raising it).

## 4. Define Constants at the Class or File Level
Constants must be defined in `ALL_CAPS`.
- Place them outside of methods, either at the class level or the file level (global).
- By default, constants should be private (e.g., prefixing them with an underscore in Python, like `_MAX_RETRIES` or `_DEFAULT_TIMEOUT`), unless there is an explicit need for them to be public.

## 5. Keep Methods Small and Modular (Extract Sub-blocks)
Methods should remain focused, small, and highly readable. Do not inline long, nested blocks (such as complex API download loops, file parsers, or multi-step operations) into existing handlers or endpoints.
- Extract nested or complex sub-blocks into well-named private helper functions or classes.
- Place the extracted helpers immediately after the calling function (following the "Caller Before Callee" sequential rule).
- This keeps the cyclomatic complexity of main methods low and facilitates easier unit testing.

**Bad:**
```python
@router.post("/process")
async def process_handler(data: RequestData):
    # Main logic...
    if data.items:
        # Long, nested block of 30 lines performing complex calculations, 
        # API calls, and GCS uploads
        for item in data.items:
            ...
    return {"status": "ok"}
```

**Good:**
```python
@router.post("/process")
async def process_handler(data: RequestData):
    # Main logic...
    processed_paths = await _process_items(data.items)
    return {"status": "ok"}

async def _process_items(items: list[Item]) -> list[str]:
    # Extracted helper logic
    ...
```

## 6. Don't Repeat Yourself (DRY)
Avoid duplicate blocks of code, even short 4-5 line utility snippets, across functions or classes in the same file.
- If a block of code (like defensive input parsing, data transformations, or specific format validations) is used more than once, extract it into a helper function.
- Place the shared helper function sequentially after the first caller, or at the top of the helper section, and reuse it everywhere.

## 7. Google Application Default Credentials (ADC) Scopes
When fetching Google Application Default Credentials (ADC) for signing GCS URLs or making other IAM-based credential operations, always explicitly pass the `https://www.googleapis.com/auth/cloud-platform` scope.
- Define this scope as a private file-level constant (in `ALL_CAPS` with a leading underscore, like `_CLOUD_PLATFORM_SCOPE = "https://www.googleapis.com/auth/cloud-platform"`).
- This ensures the generated access token contains the necessary OAuth scopes to successfully authenticate IAM signBytes API requests.
- Do not attempt to introspect the credentials object dynamically (using `getattr` or `cast`) to extract the service account email or to check for placeholders like `"default"`. Instead, retrieve the service account email directly from configuration settings (like `settings.SERVICE_ACCOUNT_EMAIL`).

## 8. No Legacy Code Fallbacks or Schema Degradation
Never add legacy fallbacks, optional schema fields for missing data, dummy defaults, or silent try/except wrappers in application code to accommodate invalid legacy database records.
- **Fix the Data, Not the Code**: Keep application schemas, models, and type constraints strict and clean. If existing records in database/storage are missing required fields or violate schema constraints, update/migrate the legacy database records directly.
- Do not dilute schema validation or add defensive compatibility code to mask corrupt or incomplete database records.

