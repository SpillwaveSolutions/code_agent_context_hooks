---
description: Create or update the project constitution from interactive or provided principle inputs, ensuring all dependent templates stay in sync.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## SDD Context

This command is part of Spec-Driven Development (SDD). For full methodology guidance, load the `sdd` skill.

**Key Artifacts:**
- Constitution: `.speckit/constitution.md`
- Templates: `.speckit/templates/`

## Goal

You are updating the project constitution at `.speckit/constitution.md`. This file is a TEMPLATE containing placeholder tokens in square brackets (e.g. `[PROJECT_NAME]`, `[PRINCIPLE_1_NAME]`). Your job is to (a) collect/derive concrete values, (b) fill the template precisely, and (c) propagate any amendments across dependent artifacts.

## Execution Steps

### 1. Load Constitution Template

Load the existing constitution template at `.speckit/constitution.md`.
- Identify every placeholder token of the form `[ALL_CAPS_IDENTIFIER]`
- **IMPORTANT**: The user might require less or more principles than the ones used in the template. If a number is specified, respect that - follow the general template. You will update the doc accordingly.

### 2. Collect/Derive Values for Placeholders

- If user input (conversation) supplies a value, use it
- Otherwise infer from existing repo context (README, docs, prior constitution versions if embedded)
- For governance dates: `RATIFICATION_DATE` is the original adoption date (if unknown ask or mark TODO), `LAST_AMENDED_DATE` is today if changes are made
- `CONSTITUTION_VERSION` must increment according to semantic versioning rules:
  - MAJOR: Backward incompatible governance/principle removals or redefinitions
  - MINOR: New principle/section added or materially expanded guidance
  - PATCH: Clarifications, wording, typo fixes, non-semantic refinements
- If version bump type ambiguous, propose reasoning before finalizing

### 3. Draft Updated Constitution Content

- Replace every placeholder with concrete text (no bracketed tokens left except intentionally retained template slots)
- Preserve heading hierarchy and comments can be removed once replaced
- Ensure each Principle section: succinct name line, paragraph (or bullet list) capturing non-negotiable rules, explicit rationale if not obvious
- Ensure Governance section lists amendment procedure, versioning policy, and compliance review expectations

### 4. Consistency Propagation Checklist

- Read `.speckit/templates/plan-template.md` and ensure any "Constitution Check" or rules align with updated principles
- Read `.speckit/templates/spec-template.md` for scope/requirements alignment - update if constitution adds/removes mandatory sections or constraints
- Read `.speckit/templates/tasks-template.md` and ensure task categorization reflects new or removed principle-driven task types
- Read any runtime guidance docs (e.g., `README.md`, `docs/quickstart.md`). Update references to principles changed

### 5. Produce Sync Impact Report

Prepend as an HTML comment at top of the constitution file after update:
- Version change: old -> new
- List of modified principles (old title -> new title if renamed)
- Added sections
- Removed sections
- Templates requiring updates (updated / pending) with file paths
- Follow-up TODOs if any placeholders intentionally deferred

### 6. Validation Before Final Output

- No remaining unexplained bracket tokens
- Version line matches report
- Dates ISO format YYYY-MM-DD
- Principles are declarative, testable, and free of vague language ("should" -> replace with MUST/SHOULD rationale where appropriate)

### 7. Write Completed Constitution

Write the completed constitution back to `.speckit/constitution.md` (overwrite).

### 8. Output Final Summary

Output a final summary to the user with:
- New version and bump rationale
- Any files flagged for manual follow-up
- Suggested commit message (e.g., `docs: amend constitution to vX.Y.Z (principle additions + governance update)`)

## Formatting & Style Requirements

- Use Markdown headings exactly as in the template (do not demote/promote levels)
- Wrap long rationale lines to keep readability (<100 chars ideally)
- Keep a single blank line between sections
- Avoid trailing whitespace

## Behavior Rules

- If the user supplies partial updates (e.g., only one principle revision), still perform validation and version decision steps
- If critical info missing (e.g., ratification date truly unknown), insert `TODO(<FIELD_NAME>): explanation` and include in the Sync Impact Report under deferred items
- Do not create a new template; always operate on the existing `.speckit/constitution.md` file

## Next Steps

After updating the constitution, common next steps include:
- `/speckit-specify` - Build a specification based on the updated constitution
- Review and update templates in `.speckit/templates/` if flagged

## Context

$ARGUMENTS
