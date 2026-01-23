---
description: Generate a custom checklist for the current feature based on user requirements.
---

## Checklist Purpose: "Unit Tests for English"

**CRITICAL CONCEPT**: Checklists are **UNIT TESTS FOR REQUIREMENTS WRITING** - they validate the quality, clarity, and completeness of requirements in a given domain.

**NOT for verification/testing:**
- NOT "Verify the button clicks correctly"
- NOT "Test error handling works"
- NOT "Confirm the API returns 200"

**FOR requirements quality validation:**
- "Are visual hierarchy requirements defined for all card types?" (completeness)
- "Is 'prominent display' quantified with specific sizing/positioning?" (clarity)
- "Are hover state requirements consistent across all interactive elements?" (consistency)
- "Are accessibility requirements defined for keyboard navigation?" (coverage)
- "Does the spec define what happens when logo image fails to load?" (edge cases)

**Metaphor**: If your spec is code written in English, the checklist is its unit test suite.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## SDD Context

This command is part of Spec-Driven Development (SDD). For full methodology guidance, load the `sdd` skill.

**Key Artifacts:**
- Specifications: `.speckit/features/<feature>/spec.md`
- Plans: `.speckit/features/<feature>/plan.md`
- Tasks: `.speckit/features/<feature>/tasks.md`
- Templates: `.speckit/templates/checklist-template.md`

## Execution Steps

### 1. Setup

Run `.speckit/scripts/bash/check-prerequisites.sh --json` from repo root and parse JSON for FEATURE_DIR and AVAILABLE_DOCS list. All file paths must be absolute.

### 2. Clarify Intent (Dynamic)

Derive up to THREE initial contextual clarifying questions. They MUST:
- Be generated from the user's phrasing + extracted signals from spec/plan/tasks
- Only ask about information that materially changes checklist content
- Be skipped individually if already unambiguous in `$ARGUMENTS`

Generation algorithm:
1. Extract signals: feature domain keywords, risk indicators, stakeholder hints, explicit deliverables
2. Cluster signals into candidate focus areas (max 4) ranked by relevance
3. Identify probable audience & timing (author, reviewer, QA, release)
4. Detect missing dimensions: scope breadth, depth/rigor, risk emphasis, exclusion boundaries
5. Formulate questions from archetypes:
   - Scope refinement
   - Risk prioritization
   - Depth calibration
   - Audience framing
   - Boundary exclusion
   - Scenario class gap

Question formatting rules:
- If presenting options, generate a compact table with columns: Option | Candidate | Why It Matters
- Limit to A-E options maximum
- Never ask the user to restate what they already said

Defaults when interaction impossible:
- Depth: Standard
- Audience: Reviewer (PR) if code-related; Author otherwise
- Focus: Top 2 relevance clusters

### 3. Understand User Request

Combine `$ARGUMENTS` + clarifying answers:
- Derive checklist theme (e.g., security, review, deploy, ux)
- Consolidate explicit must-have items mentioned by user
- Map focus selections to category scaffolding
- Infer any missing context from spec/plan/tasks (do NOT hallucinate)

### 4. Load Feature Context

Read from FEATURE_DIR:
- spec.md: Feature requirements and scope
- plan.md (if exists): Technical details, dependencies
- tasks.md (if exists): Implementation tasks

**Context Loading Strategy:**
- Load only necessary portions relevant to active focus areas
- Prefer summarizing long sections into concise scenario/requirement bullets
- Use progressive disclosure: add follow-on retrieval only if gaps detected

### 5. Generate Checklist - Create "Unit Tests for Requirements"

- Create `FEATURE_DIR/checklists/` directory if it doesn't exist
- Generate unique checklist filename using short, descriptive name based on domain (e.g., `ux.md`, `api.md`, `security.md`)
- Number items sequentially starting from CHK001
- Each `/speckit-checklist` run creates a NEW file (never overwrites existing checklists)

**CORE PRINCIPLE - Test the Requirements, Not the Implementation:**
Every checklist item MUST evaluate the REQUIREMENTS THEMSELVES for:
- **Completeness**: Are all necessary requirements present?
- **Clarity**: Are requirements unambiguous and specific?
- **Consistency**: Do requirements align with each other?
- **Measurability**: Can requirements be objectively verified?
- **Coverage**: Are all scenarios/edge cases addressed?

**Category Structure** - Group items by requirement quality dimensions:
- Requirement Completeness
- Requirement Clarity
- Requirement Consistency
- Acceptance Criteria Quality
- Scenario Coverage
- Edge Case Coverage
- Non-Functional Requirements
- Dependencies & Assumptions
- Ambiguities & Conflicts

**HOW TO WRITE CHECKLIST ITEMS:**

WRONG (Testing implementation):
- "Verify landing page displays 3 episode cards"
- "Test hover states work on desktop"

CORRECT (Testing requirements quality):
- "Are the exact number and layout of featured episodes specified?" [Completeness]
- "Is 'prominent display' quantified with specific sizing/positioning?" [Clarity]
- "Are hover state requirements consistent across all interactive elements?" [Consistency]
- "Are keyboard navigation requirements defined for all interactive UI?" [Coverage]

**ITEM STRUCTURE:**
- Question format asking about requirement quality
- Focus on what's WRITTEN (or not written) in the spec/plan
- Include quality dimension in brackets [Completeness/Clarity/Consistency/etc.]
- Reference spec section `[Spec §X.Y]` when checking existing requirements
- Use `[Gap]` marker when checking for missing requirements

**Traceability Requirements:**
- MINIMUM: >=80% of items MUST include at least one traceability reference
- Each item should reference: spec section `[Spec §X.Y]`, or use markers: `[Gap]`, `[Ambiguity]`, `[Conflict]`, `[Assumption]`

**Content Consolidation:**
- Soft cap: If raw candidate items > 40, prioritize by risk/impact
- Merge near-duplicates checking the same requirement aspect
- If >5 low-impact edge cases, create one item aggregating them

**PROHIBITED** - These make it an implementation test:
- Any item starting with "Verify", "Test", "Confirm", "Check" + implementation behavior
- References to code execution, user actions, system behavior
- "Displays correctly", "works properly", "functions as expected"

**REQUIRED PATTERNS** - These test requirements quality:
- "Are [requirement type] defined/specified/documented for [scenario]?"
- "Is [vague term] quantified/clarified with specific criteria?"
- "Are requirements consistent between [section A] and [section B]?"
- "Can [requirement] be objectively measured/verified?"

### 6. Structure Reference

Generate the checklist following the canonical template in `.speckit/templates/checklist-template.md` for title, meta section, category headings, and ID formatting. If template is unavailable, use: H1 title, purpose/created meta lines, `##` category sections containing `- [ ] CHK### <requirement item>` lines.

### 7. Report

Output full path to created checklist, item count, and summarize:
- Focus areas selected
- Depth level
- Actor/timing
- Any explicit user-specified must-have items incorporated

## Example Checklist Types & Sample Items

**UX Requirements Quality:** `ux.md`
- "Are visual hierarchy requirements defined with measurable criteria? [Clarity, Spec §FR-1]"
- "Is the number and positioning of UI elements explicitly specified? [Completeness, Spec §FR-1]"
- "Are accessibility requirements specified for all interactive elements? [Coverage, Gap]"

**API Requirements Quality:** `api.md`
- "Are error response formats specified for all failure scenarios? [Completeness]"
- "Are rate limiting requirements quantified with specific thresholds? [Clarity]"
- "Is versioning strategy documented in requirements? [Gap]"

**Security Requirements Quality:** `security.md`
- "Are authentication requirements specified for all protected resources? [Coverage]"
- "Is the threat model documented and requirements aligned to it? [Traceability]"
- "Are security failure/breach response requirements defined? [Gap, Exception Flow]"

## Next Steps

After generating a checklist, common next steps include:
- Review the checklist and update the spec to address gaps
- `/speckit-analyze` - Run consistency analysis
- `/speckit-implement` - Proceed with implementation

## Context

$ARGUMENTS
