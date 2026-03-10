---
name: skill-publisher
description: Automatically publishes non-project-specific skills to the central skill repository. Use this skill whenever a new skill is created or an existing skill is modified.
---

# Skill Publisher

Whenever you create or modify a skill, determine if it is project-specific. If it is generally applicable (or can be made so), you must publish it to the central skill repository.

## Publishing Workflow

1.  **Check Repository**:
    -   Check if the directory `~/gemini-cli-skills` exists.
    -   If it does not exist, clone the repository: `git clone https://github.com/j-steve/gemini-cli-skills ~/gemini-cli-skills`.
    -   If it does exist, navigate to it and sync it with remote: `git -C ~/gemini-cli-skills pull`.

2.  **Copy the Skill**:
    -   Copy the skill's source directory (containing `SKILL.md` and any assets/scripts) into `~/gemini-cli-skills/`. For example: `cp -r ~/skill-name ~/gemini-cli-skills/`

3.  **Commit and Push**:
    -   Stage the changes: `git -C ~/gemini-cli-skills add .`
    -   Commit the changes with a descriptive message: `git -C ~/gemini-cli-skills commit -m "Add/Update <skill-name> skill"`
    -   Push to GitHub. Use the `GITHUB_TOKEN` from `~/.gemini/.env` if authentication is needed. Example:
        ```bash
        export GITHUB_TOKEN=$(grep "GITHUB_TOKEN" ~/.gemini/.env | cut -d'=' -f2)
        git -C ~/gemini-cli-skills push https://x-access-token:$GITHUB_TOKEN@github.com/j-steve/gemini-cli-skills.git main
        ```
