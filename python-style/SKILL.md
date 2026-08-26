---
name: python-style
description: Python-specific code style conventions and patterns, including PEP 8 standards, typing, and logging conventions.
---

# Python Style & Architectural Conventions

This skill provides comprehensive Python-specific code style standards and patterns. All Python code written or refactored within repositories governed by `code-writer-kit` must strictly adhere to these conventions in conjunction with the core `style_guide.md`.

---

## 1. Logging Convention

### Standard Pattern: `logger = logging.getLogger(__name__)`
Always instantiate module-level loggers using the standard lowercase identifier `logger` without any leading underscore:

```python
import logging

logger = logging.getLogger(__name__)
```

### Rationale & Rules
- **Stateful Instances vs. Static Constants**: Loggers are module-level instances/singletons with mutable runtime state and callable methods (`logger.info()`, `logger.debug()`), not immutable constant value literals.
- **Explicit Override of Uppercase Constant Rule**: This convention explicitly overrides the general uppercase naming rule for module-level constants.
- **Forbidden Antipatterns**:
  - `_LOGGER = logging.getLogger(__name__)` (DO NOT USE)
  - `LOGGER = logging.getLogger(__name__)` (DO NOT USE)
  - `_logger = logging.getLogger(__name__)` (DO NOT USE)
  - `logger = _LOGGER` or `logger = LOGGER` (DO NOT USE)

---

## 2. Constants: Private vs. Public

Define all module-level constants at the top of the file immediately following imports.

### Private Module Constants (`_ALL_CAPS`)
Use uppercase naming with a single leading underscore for internal/private constants not intended for public export from the module:
```python
_DEFAULT_TIMEOUT_SECONDS: float = 30.0
_MAX_RETRY_ATTEMPTS: int = 3
_MODEL_NAME: str = "gemini-2.5-pro"
_BUFFER_SIZE_BYTES: int = 4096
```

### Public Module Constants (`ALL_CAPS`)
Use uppercase naming without a leading underscore for constants explicitly designed for public API export:
```python
DEFAULT_CONFIG_PATH: str = "/etc/antigravity/config.json"
MAX_PAYLOAD_SIZE: int = 10485760
VERSION: str = "1.0.0"
```

---

## 3. Strict Type Hints & Modern Syntax

Maintain 100% static typing compliance across all functions, methods, and class attributes.

### Modern Union & Generic Syntax
Use modern Python 3.10+ typing syntax directly rather than importing from `typing`:
- Use `int | str` instead of `Union[int, str]`.
- Use `str | None` instead of `Optional[str]`.
- Use standard built-in generic collections: `list[str]`, `dict[str, object]`, `set[int]`, `tuple[str, ...]`.

### Avoid `Any`
Never use `Any` or untyped signatures. Use concrete types, specific interfaces, type variables (`TypeVar`), or `object` where generic data structures are handled.

### Forward References & Top-Down Ordering
When structuring modules with caller-before-callee ordering, place `from __future__ import annotations` as the first line of the file to prevent forward-reference errors in type annotations:

```python
from __future__ import annotations

import logging
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mypackage.models import ComplexRecord

logger = logging.getLogger(__name__)
```

---

## 4. Top-Down Sequential Ordering (Caller Before Callee)

Organize module contents so that files read logically from top to bottom:
1. **Module Docstring & Future Imports**: `from __future__ import annotations` and module overview.
2. **Imports**: Standard library, third-party packages, local application modules.
3. **Constants & Module Logger**: Private/public constants and `logger = logging.getLogger(__name__)`.
4. **Public Classes & Functions**: High-level public entry points, services, and APIs.
5. **Private Helpers**: Placed immediately below the specific caller function that invokes them.
6. **Shared Private Helpers**: Placed immediately below the last caller in the group that uses them.

---

## 5. Modularity & Function Complexity

### Single Responsibility & Line Limits
- Every function and method must have a single, well-defined responsibility.
- Keep function logic concise, typically `<= 25–30 executable lines` (excluding docstrings, type annotations, and blank lines).
- Decompose multi-step workflows into focused, private helper functions placed immediately below the caller.
- Do not artificially fragment coherent, readable algorithms solely to satisfy line limits; balance modularity with readability.

### Data Encapsulation & Callee Self-Sufficiency
- Prefer having helper and child functions derive or fetch their own internal dependencies directly from the primary configuration or context object (e.g., `cfg`), rather than having caller functions orchestrate intermediate data fetches solely to pass them down as discrete arguments.
- Pushing data retrieval down into the callee keeps caller orchestration concise and decoupled, and narrows function signatures to only essential domain parameters unless explicit separation of concerns or testability dictates otherwise.

**Anti-pattern:**
```python
def sync_dataset(dataset_id: str, cfg: PipelineConfig) -> SyncSummary:
    # Caller unnecessarily extracts and derives parameters solely consumed by the child helper
    endpoint_url = f"{cfg.base_url}/datasets/{dataset_id}/sync"
    auth_headers = {"Authorization": f"Bearer {cfg.api_key}"}
    timeout_seconds = cfg.timeout_seconds

    records = _fetch_remote_records(endpoint_url, auth_headers, timeout_seconds)
    return _persist_records(dataset_id, records)


def _fetch_remote_records(
    endpoint_url: str,
    auth_headers: dict[str, str],
    timeout_seconds: int,
) -> list[Record]:
    ...
```

**Preferred:**
```python
def sync_dataset(dataset_id: str, cfg: PipelineConfig) -> SyncSummary:
    # Caller stays clean and orchestrates high-level workflow
    records = _fetch_remote_records(dataset_id, cfg)
    return _persist_records(dataset_id, records)


def _fetch_remote_records(dataset_id: str, cfg: PipelineConfig) -> list[Record]:
    # Child helper encapsulates its own URL construction and header derivation
    endpoint_url = f"{cfg.base_url}/datasets/{dataset_id}/sync"
    auth_headers = {"Authorization": f"Bearer {cfg.api_key}"}
    ...
```

---

## 6. Error Visibility & Custom Exceptions

### Custom Domain Exceptions
Derive all application-specific exceptions from a base application exception. Never raise bare `Exception` or generic `ValueError` when a domain-specific exception is appropriate:

```python
class AppException(Exception):
    """Base exception for all application errors."""


class ConfigurationError(AppException):
    """Raised when configuration validation fails."""


class NetworkTimeoutError(AppException):
    """Raised when an external service request times out."""
```

### Transparent Propagation
- Allow unexpected exceptions to propagate naturally rather than swallowing them with empty `except:` or generic `except Exception: pass` blocks.
- Avoid arbitrary fallback defaults that mask failures; fail explicitly and cleanly.
- Use `raise ... from exc` to preserve original traceback chains when re-raising exceptions.

---

## 7. Context-Rich Documentation (Google-Style Docstrings)

Provide Google-style docstrings on all classes, public functions, and public methods. Docstrings must provide additive operational context, invariants, and design rationale:

```python
def fetch_user_profile(user_id: str, timeout_seconds: float = _DEFAULT_TIMEOUT_SECONDS) -> dict[str, object]:
    """Retrieves user profile data from the remote identity service.

    Connects via the authenticated identity gateway and deserializes
    the cached user profile. If the cache is cold, triggers a backfill.

    Args:
        user_id: Unique alphanumeric identifier of the target user.
        timeout_seconds: Maximum time to wait for gateway response.

    Returns:
        Dictionary mapping profile attribute keys to raw values.

    Raises:
        UserNotFoundError: If the provided user_id does not exist.
        NetworkTimeoutError: If the remote identity gateway fails to respond.
    """
```

---

## 8. Direct Expressions & Inline Flow

- Favor direct `return`, `yield`, or `pass` expressions over intermediate variable aliases.
- Reserve local variables strictly for values reused multiple times or complex intermediate computations.
- Use list, dict, and set comprehensions for simple, clear transformations; fall back to explicit loops for complex logic.

---

## 9. Targeted Verification vs. Monolithic Test Suites

- Fast static checks (`ruff check`, `ruff format`, `pyright`, `mypy`) and isolated, target-specific unit tests (e.g., testing a single target or file) are permitted and encouraged to catch regressions and type mismatches before handoff.
- NEVER run full-repo monolithic test suites (e.g. `bazel test //...`) or heavy end-to-end integration suites locally; remote CI presubmits handle full regression testing.
