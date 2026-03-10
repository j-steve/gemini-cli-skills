# Gemini CLI Skills

A collection of useful general-purpose skills for the Gemini CLI to enhance your development workflow.

## Installation

### Install the entire suite
To install all skills in this repository as an extension:
```bash
gemini extensions install https://github.com/j-steve/gemini-cli-skills.git
```

### Install specific skills
To install individual skills (e.g., git-operations):
```bash
gemini skills install https://github.com/j-steve/gemini-cli-skills.git --path git-operations
```

## Available Skills

Here is the current catalog of skills available in this repository:

| Skill Name | What it does | Why you should use it (The Pitch) |
| :--- | :--- | :--- |
| **`github-issue-creator`** | Guides the agent to create well-formatted GitHub issues with appropriate markdown and default labels (like `bug`). | **Tired of messy, unstructured bug reports?** Let this skill do the heavy lifting. It ensures every issue is readable, actionable, and properly categorized right out of the gate! |
| **`skill-publisher`** | Instructs the agent to automatically publish generally applicable, non-project-specific skills to this central repository and updates this README. | **Keep your agent's knowledge base centralized and up-to-date!** No more manually copying skills around. Just build, let the agent push it here, and instantly share it across your workspaces. |
| **`git-operations`** | Streamlines your git and GitHub workflow with automated syncing, testing, and committing. | **Never break the build again!** Enforces pre-commit testing and handles repetitive git tasks, keeping your commit history clean and your code stable. |
