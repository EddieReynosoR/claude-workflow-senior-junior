---
name: discovery
description: Build a feature plan with the user when they have none yet. Claude reads the codebase, asks about scope, proposes a plan, and iterates until the user approves it. Manual only.
argument-hint: "[feature description]"
disable-model-invocation: true
disallowed-tools: Edit Write NotebookEdit
---

# Discovery: from a feature idea to a plan the user approves

The user is the lead and decides. They have a feature but no plan yet. Your job: understand the problem, propose a plan, and iterate until the user approves it. No code in this phase.

Feature: $ARGUMENTS

If the feature is empty or unclear, ask the user to describe it in two or three sentences and stop.

## Rules for the whole phase
- No code and no file edits until the user approves the plan. Reading the codebase is expected.
- Every choice in your proposal is a proposal: show the alternative you rejected and why.
- Mark anything you assumed instead of confirmed with **(assumption)**.
- Don't push for approval. The user decides when the plan is ready.
- Reply in the user's language.

## Steps
1. **Context.** Read the parts of the codebase the feature touches: related models or objects, existing patterns, tests, conventions. Note what already exists that the plan can reuse. Summarize in a few lines, not a tour.
2. **Scope questions.** Ask only what you can't infer from the code: what version one must do, what "done" looks like, constraints (security, data, integrations, deadlines), who uses it. At most 5 questions, numbered, in one message. Then stop and wait for answers.
3. **Propose the plan** in the plan file format below, sections Problem through What I don't know (leave Review notes and Execution log empty). Each step must be small enough to implement and verify on its own, with a `Check:` line. Put every key choice in the Decisions table with its alternative.
4. **Ask for validation.** End by naming the part you are least sure about, and ask the user to approve, change, or reject parts of the plan.
5. **Iterate.** Apply only the changes the user asks for. If a change conflicts with another part of the plan, say so.
6. **On approval** (the user says approved, aprobado, me sirve, or equivalent):
   - Save the plan following "Saving the plan" below, with `Status: approved` and `Origin: discovery`. Record the assumptions the user confirmed or corrected under Review notes.
   - Reply with the file path and tell the user they can run `/execute <path>` when ready. Do not start implementing.

## Saving the plan
- Directory: the one named in the project's CLAUDE.md on a line like `Plans directory: <path>`; otherwise `docs/plans/`.
- File name: `YYYY-MM-DD-<feature-slug>.md`. If the file already exists, ask before overwriting.
- Content: the format below, filled with the plan exactly as approved. No extra steps, scope, or features.

## Plan file format
```markdown
# Plan: <feature name>

Status: approved
<!-- Status flow: approved → in-progress → in-review → done -->
Origin: <own plan + review | discovery>
Approved: <YYYY-MM-DD>

## Problem
<What is wrong or missing, and for whom.>

## Done when
- [ ] <Observable, testable criterion>

## Edge cases
- <Input or state> → <expected behavior>

## Approach
<The chosen approach in a few lines.>

### Steps
- [ ] 1. <Small step that can be implemented and verified on its own> — Check: <how to verify it>

## Decisions
| Decision | Chosen | Alternative | Why |
|---|---|---|---|

## What I don't know
- <Unknown> → <how it gets resolved, or "accepted risk">

## Review notes
<From a review: accepted and rejected suggestions by ID, with the reason for each rejection.>
<From discovery: assumptions the user confirmed or corrected.>

## Execution log
<!-- Filled by /execute: date, step, files, verification result, stops and the user's decisions. -->
```
