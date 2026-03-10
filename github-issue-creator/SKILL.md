---
name: github-issue-creator
description: Guidelines for creating well-formatted GitHub issues, including bug reports with markdown formatting, and applying appropriate default tags. Use whenever instructed to create a GitHub issue or bug report.
---

# GitHub Issue Creator

When instructed to create a GitHub issue or bug report, follow these guidelines to ensure the issue is clear, well-formatted, and appropriately tagged.

## Default GitHub Tags

Always apply appropriate labels when creating an issue. The default labels in standard GitHub repositories are:

- `bug`: Something isn't working
- `enhancement`: New feature or request
- `documentation`: Improvements or additions to documentation
- `duplicate`: This issue or pull request already exists
- `good first issue`: Good for newcomers
- `help wanted`: Extra attention is needed
- `invalid`: This doesn't seem right
- `question`: Further information is requested
- `wontfix`: This will not be worked on

**Always tag a bug report with the `bug` label.**

## Formatting Bug Reports

Structure the issue description clearly using markdown. A standard bug report format should include the following sections:

### Describe the Bug
A clear and concise description of what the bug is.

### Expected Behavior
A clear and concise description of what you expected to happen.

### Actual Behavior
A clear and concise description of what actually happened. Include error messages, stack traces, or relevant logs inside markdown code blocks.

### Steps to Reproduce
A numbered list of steps to reproduce the behavior, if applicable.

## Tools & Authentication
If `create_issue` or `update_issue` tools fail due to an authentication error, you can use the GitHub API via `curl`. To do so, load the `GITHUB_TOKEN` from the `.gemini/.env` file.

Example of creating an issue with `curl`:
```bash
export GITHUB_TOKEN=$(grep "GITHUB_TOKEN" ~/.gemini/.env | cut -d'=' -f2)
curl -X POST -H "Authorization: token $GITHUB_TOKEN" \
     -H "Accept: application/vnd.github.v3+json" \
     -d '{"title":"Issue Title","body":"Markdown body","labels":["bug"]}' \
     https://api.github.com/repos/owner/repo/issues
```
