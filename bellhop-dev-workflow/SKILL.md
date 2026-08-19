---
name: bellhop-dev-workflow
description: Setup and maintain the Bellhop development environment, including nested source paths, venv, testing, and CI monitoring.
---

## When to Use
Use this skill when starting work on the Bellhop project, running tests, linting, resolving "ModuleNotFoundError" or "AttributeError" in unit tests, or checking/debugging CI runs.

## Procedure

### 1. Environment Setup

1. **Working Directory**: Always work from the repository root: `/home/steve5805/bellhop/`.
2. **Nested Source Path**: Note that the primary source code is located in a nested directory: `bellhop/src/` (relative to the repo root). If imports fail, verify if the path should be `bellhop/bellhop/src/`.
3. **Virtual Environment**: Use the project's virtual environment for all Python operations:
   ```bash
   cd /home/steve5805/bellhop
   source venv/bin/activate
   # Or use the absolute path to the python executable
   ./venv/bin/python3 -m <module>
   ```
4. **PYTHONPATH**: When running scripts from outside the source root, explicitly set the `PYTHONPATH`:
   ```bash
   export PYTHONPATH=$(pwd)/bellhop/src
   # or
   export PYTHONPATH=$(pwd)/bellhop/bellhop/src
   ```

### 2. Linting and Formatting

Use Ruff for linting and formatting. Many errors can be fixed automatically:
```bash
./venv/bin/python3 -m ruff check . --fix
./venv/bin/python3 -m ruff format .
```

### 3. Running Tests

> [!CAUTION]
> **DO NOT RUN LOCAL BAZEL (`bazel test`)**. Local Bazel execution is extremely slow and takes too long. Never run `bazel test` locally.

- **Fast Python / Pytest Validation (PREFERRED)**: Use the virtual environment Python directly with `PYTHONPATH`:
  ```bash
  cd /home/steve5805/bellhop
  ./venv/bin/python3 -m pyright <file.py>
  PYTHONPATH=$(pwd)/src ./venv/bin/python3 -m pytest tests/unit/path/to/test.py
  ```
- **CI Presubmit Validation**: Push your branch and rely on GitHub Actions presubmit checks (`gh run list`) for full remote Bazel verification.

### 4. CI Monitoring

When asked to "check the presubmits," "verify the PR," or investigate remote build failures:

1. **Check Local Context**: Identify the current branch:
   ```bash
   git branch --show-current
   ```
2. **List Recent Runs**: Find failing runs for a branch or PR:
   ```bash
   gh run list --branch <branch_name> --limit 5
   ```
3. **View Run Details**: Get a summary of jobs and their statuses:
   ```bash
   gh run view <run_id>
   ```
4. **Inspect Failed Logs**: Retrieve logs for failing steps (e.g., Pyright errors, failing tests):
   ```bash
   gh run view <run_id> --log-failed
   ```
5. **Watch for Completion**: Wait for CI to finish after pushing changes:
   ```bash
   gh run watch <run_id> --exit-status
   ```
- **Tip**: Look for the `X` (failure) or `✓` (success) markers in `gh run list` output.
- You can provide the user with a direct link: `https://github.com/j-steve/bellhop/actions/runs/<run_id>`.

## Pitfalls and Fixes

- **Symptom**: `ModuleNotFoundError: No module named 'db'`
  - **Cause**: `PYTHONPATH` is not set or points to the wrong directory level.
  - **Fix**: Set `PYTHONPATH` to the directory containing the `db` package (e.g., `export PYTHONPATH=$(pwd)/bellhop/src`).

- **Symptom**: `ModuleNotFoundError: No module named 'google'`
  - **Cause**: Running with system `python3` instead of the project's virtual environment.
  - **Fix**: Use `bellhop/.venv/bin/python3` or `bellhop/venv/bin/python3`.

- **Symptom**: `AttributeError: <module 'tools.x'> does not have the attribute 'y'` during unit tests.
  - **Cause**: A function was refactored/moved to another module (e.g., `tools.utils`), but the unit test is still trying to patch it in the old location.
  - **Fix**: Update the `@patch` decorator in the test file to target the new location OR the location where the code under test imports the function from.

- **Symptom**: `PermissionDenied: 403 Missing or insufficient permissions` during unit tests.
  - **Cause**: Tests are attempting to call live Firestore/GCP services instead of using mocks.
  - **Fix**: Ensure all database-interacting functions are correctly patched with `AsyncMock` or `MagicMock`.

- **Symptom**: `docker.errors.DockerException: Error while fetching server API version`
  - **Cause**: Integration tests fail because the local environment has no Docker daemon.
  - **Fix**: Skip integration tests locally. Run only unit tests (`tests/unit/`). Use CI for integration verification.

- **Symptom**: Empty log output from `gh run view --log`
  - **Cause**: GitHub API is slow to provide logs for very fresh runs.
  - **Fix**: Wait a few seconds and retry, or use `gh run watch` to wait for completion.

## Verification
- `pytest` should report "passed" for all relevant tests.
- `ruff check .` should return no errors.
- You can pinpoint the exact failing CI step (e.g., `Run Pyright (Static Type Checking)`).