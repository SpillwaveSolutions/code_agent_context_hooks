---
description: Convert existing tasks into actionable, dependency-ordered GitHub issues for the feature based on available design artifacts.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## SDD Context

This command is part of Spec-Driven Development (SDD). For full methodology guidance, load the `sdd` skill.

**Key Artifacts:**
- Tasks: `.speckit/features/<feature>/tasks.md`

## Goal

Convert the completed tasks.md file into GitHub issues that can be tracked and assigned in the repository.

## Execution Steps

### 1. Initialize Context

Run `.speckit/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks` from repo root and parse FEATURE_DIR and AVAILABLE_DOCS list. All paths must be absolute.

### 2. Extract Tasks Path

From the executed script, extract the path to **tasks.md**.

### 3. Verify GitHub Remote

Get the Git remote by running:

```bash
git config --get remote.origin.url
```

**CRITICAL**: ONLY PROCEED TO NEXT STEPS IF THE REMOTE IS A GITHUB URL

If the remote is not a GitHub URL, abort and inform the user that this command only works with GitHub repositories.

### 4. Create GitHub Issues

For each task in the list, use the GitHub CLI (`gh`) to create a new issue in the repository that matches the Git remote.

For each task:
- Extract task ID, title, and description from tasks.md
- Include any dependencies or phase information
- Add appropriate labels if defined (e.g., `enhancement`, `bug`, `documentation`)
- Link related tasks as dependencies in the issue body

Example issue creation:
```bash
gh issue create --title "TASK-001: Implement feature X" --body "Description from tasks.md..."
```

**CRITICAL**: UNDER NO CIRCUMSTANCES EVER CREATE ISSUES IN REPOSITORIES THAT DO NOT MATCH THE REMOTE URL

### 5. Report Results

After creating issues:
- List all created issues with their GitHub URLs
- Note any tasks that could not be converted
- Summarize total issues created

## Behavior Rules

- Always verify the GitHub remote before creating any issues
- Never create issues in a repository that doesn't match the local remote
- If tasks.md doesn't exist, instruct user to run `/speckit-tasks` first
- Respect any existing issue numbering or linking conventions in the repository

## Next Steps

After creating issues, common next steps include:
- Review and refine issues in GitHub
- Assign issues to team members
- `/speckit-implement` - Begin implementation of tasks

## Context

$ARGUMENTS
