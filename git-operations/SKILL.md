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
1. **Submit PR without Local Tests:** Do NOT execute the project's unit test suite locally. Instead, stage the relevant modified files and push the changes to submit the PR directly.
2. **Monitor CI Action:** After submitting the PR, monitor the resulting GitHub Action/CI run to ensure all pre-merge checks pass successfully. If any check fails, coordinate with the user or debug based on CI logs.
3. **Initialize & Stage:** If in a new directory, run `git init`. Otherwise, stage the relevant modified files.
4. **Commit & Link Issues:** Generate a concise, descriptive commit message. If a relevant GitHub issue is known or being worked on, automatically append the issue reference to the commit message using standard closing keywords (e.g., `Fixes #123` or `Refs #45`).
5. **Push to Origin:** Push to the default `origin`. If no remote origin is set, ask the user for the repository URL, set it via `git remote add origin <url>`, and then push.
6. **Summarize:** Provide a direct summary of what was staged, committed, pushed, and link the GitHub PR for monitoring the CI results.
