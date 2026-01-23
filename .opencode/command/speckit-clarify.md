---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec.
---

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## SDD Context

This command is part of Spec-Driven Development (SDD). For full methodology guidance, load the `sdd` skill.

**Key Artifacts:**
- Specifications: `.speckit/features/<feature>/spec.md`
- Templates: `.speckit/templates/spec-template.md`

## Goal

Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

**Note**: This clarification workflow is expected to run (and be completed) BEFORE invoking `/speckit-plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but must warn that downstream rework risk increases.

## Execution Steps

### 1. Initialize Context

Run `.speckit/scripts/bash/check-prerequisites.sh --json --paths-only` from repo root once. Parse minimal JSON payload fields:
- `FEATURE_DIR`
- `FEATURE_SPEC`
- (Optionally capture `IMPL_PLAN`, `TASKS` for future chained flows.)

If JSON parsing fails, abort and instruct user to re-run `/speckit-specify` or verify feature branch environment.

### 2. Perform Ambiguity & Coverage Scan

Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing.

**Functional Scope & Behavior:**
- Core user goals & success criteria
- Explicit out-of-scope declarations
- User roles / personas differentiation

**Domain & Data Model:**
- Entities, attributes, relationships
- Identity & uniqueness rules
- Lifecycle/state transitions
- Data volume / scale assumptions

**Interaction & UX Flow:**
- Critical user journeys / sequences
- Error/empty/loading states
- Accessibility or localization notes

**Non-Functional Quality Attributes:**
- Performance (latency, throughput targets)
- Scalability (horizontal/vertical, limits)
- Reliability & availability (uptime, recovery expectations)
- Observability (logging, metrics, tracing signals)
- Security & privacy (authN/Z, data protection, threat assumptions)
- Compliance / regulatory constraints (if any)

**Integration & External Dependencies:**
- External services/APIs and failure modes
- Data import/export formats
- Protocol/versioning assumptions

**Edge Cases & Failure Handling:**
- Negative scenarios
- Rate limiting / throttling
- Conflict resolution (e.g., concurrent edits)

**Constraints & Tradeoffs:**
- Technical constraints (language, storage, hosting)
- Explicit tradeoffs or rejected alternatives

**Terminology & Consistency:**
- Canonical glossary terms
- Avoided synonyms / deprecated terms

**Completion Signals:**
- Acceptance criteria testability
- Measurable Definition of Done style indicators

**Misc / Placeholders:**
- TODO markers / unresolved decisions
- Ambiguous adjectives ("robust", "intuitive") lacking quantification

### 3. Generate Prioritized Questions

Generate (internally) a prioritized queue of candidate clarification questions (maximum 5). Apply these constraints:
- Maximum of 10 total questions across the whole session
- Each question must be answerable with EITHER:
  - A short multiple-choice selection (2-5 distinct, mutually exclusive options), OR
  - A one-word / short-phrase answer (explicitly constrain: "Answer in <=5 words")
- Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation
- Ensure category coverage balance: attempt to cover the highest impact unresolved categories first
- Exclude questions already answered, trivial stylistic preferences, or plan-level execution details

### 4. Sequential Questioning Loop (Interactive)

- Present EXACTLY ONE question at a time
- For multiple-choice questions:
  - **Analyze all options** and determine the **most suitable option** based on best practices, common patterns, risk reduction, and alignment with project goals
  - Present your **recommended option prominently** at the top with clear reasoning
  - Format as: `**Recommended:** Option [X] - <reasoning>`
  - Render all options as a Markdown table:

  | Option | Description |
  |--------|-------------|
  | A | <Option A description> |
  | B | <Option B description> |
  | Short | Provide a different short answer (<=5 words) |

  - Add: `You can reply with the option letter (e.g., "A"), accept the recommendation by saying "yes" or "recommended", or provide your own short answer.`

- For short-answer style (no meaningful discrete options):
  - Provide your **suggested answer** based on best practices and context
  - Format as: `**Suggested:** <your proposed answer> - <brief reasoning>`
  - Then output: `Format: Short answer (<=5 words). You can accept the suggestion by saying "yes" or "suggested", or provide your own answer.`

- After the user answers:
  - If the user replies with "yes", "recommended", or "suggested", use your previously stated recommendation/suggestion
  - Otherwise, validate the answer maps to one option or fits the <=5 word constraint
  - Record it in working memory and move to the next queued question

- Stop asking further questions when:
  - All critical ambiguities resolved early
  - User signals completion ("done", "good", "no more")
  - You reach 5 asked questions

### 5. Integration After EACH Accepted Answer

- Maintain in-memory representation of the spec plus the raw file contents
- For the first integrated answer in this session:
  - Ensure a `## Clarifications` section exists (create it after the overview section if missing)
  - Under it, create a `### Session YYYY-MM-DD` subheading for today
- Append a bullet line immediately after acceptance: `- Q: <question> -> A: <final answer>`
- Apply the clarification to the most appropriate section(s):
  - Functional ambiguity -> Update Functional Requirements
  - User interaction -> Update User Stories or Actors subsection
  - Data shape -> Update Data Model
  - Non-functional constraint -> Add/modify Non-Functional section
  - Edge case -> Add under Edge Cases / Error Handling
  - Terminology conflict -> Normalize term across spec
- Save the spec file AFTER each integration

### 6. Validation

Performed after EACH write plus final pass:
- Clarifications session contains exactly one bullet per accepted answer
- Total asked (accepted) questions <= 5
- Updated sections contain no lingering vague placeholders
- Markdown structure valid
- Terminology consistency maintained

### 7. Write Updated Spec

Write the updated spec back to `FEATURE_SPEC`.

### 8. Report Completion

After questioning loop ends:
- Number of questions asked & answered
- Path to updated spec
- Sections touched (list names)
- Coverage summary table with each taxonomy category status
- If any Outstanding or Deferred remain, recommend whether to proceed to `/speckit-plan` or run `/speckit-clarify` again later

## Behavior Rules

- If no meaningful ambiguities found, respond: "No critical ambiguities detected worth formal clarification." and suggest proceeding
- If spec file missing, instruct user to run `/speckit-specify` first
- Never exceed 5 total asked questions
- Avoid speculative tech stack questions unless the absence blocks functional clarity
- Respect user early termination signals ("stop", "done", "proceed")

## Next Steps

After clarification, common next steps include:
- `/speckit-plan` - Build the technical plan from the clarified spec
- `/speckit-specify` - Further refine the specification if major gaps remain

## Context

$ARGUMENTS
