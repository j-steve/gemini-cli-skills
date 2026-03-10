---
name: git-operations
description: Comprehensive git and GitHub workflow management, including syncing, testing, and pushing.
---

# Git and GitHub Operations

You handle all version control, pre-commit testing, and GitHub operations for the user.

## Pre-flight
* **Always sync first:** Before modifying or writing any code, check if the current directory is git-tracked. If it is, execute a `git pull` to sync the local repository with the remote before proceeding.

## Authentication
1. **Check for Token:** Before running any remote command that requires authentication, check `.gemini/.env` (workspace) or `~/.gemini/.env` (global) for the `GITHUB_TOKEN` variable.
2. **Prompt and Store:** If the token is missing from both files, explicitly ask the user for it. Once provided, append `GITHUB_TOKEN=<provided_token>` to the `~/.gemini/.env` file.

## Instructions
1. **Test Before Commit:** Before staging or committing any changes, locate and execute the project's unit test suite. If any tests fail, halt the commit process immediately. Output the test failures and ask the user how to proceed.
2. **Initialize & Stage:** If in a new directory, run `git init`. If all tests pass in an existing directory, stage the relevant modified files.
3. **Commit & Link Issues:** Generate a concise, descriptive commit message. If a relevant GitHub issue is known or being worked on, automatically append the issue reference to the commit message using standard closing keywords (e.g., `Fixes #123` or `Refs #45`).
4. **Push to Origin:** Push to the default `origin`. If no remote origin is set, ask the user for the repository URL, set it via `git remote add origin <url>`, and then push.
5. **Summarize:** Provide a fast, direct summary of the test results and what was synced, committed, or pushed.
