---
name: execute
description: Implement an approved plan file step by step, verify each step, and hand off a report for the user's review. Manual only.
argument-hint: "[path to plan file] [step numbers]"
disable-model-invocation: true
---

# Execute an approved plan: you implement, the user reviews

Arguments: $ARGUMENTS

The plan is the contract. Implement exactly what it says, verify it, and give the user what they need to review your work.

## Never
- Change the plan's steps, scope, or "Done when" on your own.
- Add features, refactors, files, or dependencies the plan doesn't ask for.
- Commit or push unless the user asks.
- Say something works without having run the check.

## 0. Load the plan
- If a path was given, use it. Otherwise look in the plans directory (the one named in the project's CLAUDE.md on a line like `Plans directory: <path>`, else `docs/plans/`) for files with `Status: approved` or `Status: in-progress`. One match: use it. Several: list them and ask. None: say there's no approved plan, suggest sharing a plan for review or running `/discovery`, and stop.
- If the plan's status is anything else, stop and ask.
- If step numbers were given, do only those steps. Otherwise do every unchecked step, in order.
- Read the whole plan before touching code. Run `git status`; if there are uncommitted changes unrelated to the plan, mention them and ask before continuing.
- Set `Status: in-progress` in the plan file.

## 1. For each step
1. Implement only what the step says, following the project's existing conventions.
2. Verify with the step's `Check:` plus the project's relevant tests, lint, or build (for example `npm test`, `mvn test`, `sf apex run test`). If a check can't run here, say so.
3. If verification fails, fix it within the step's scope, at most two attempts. Still failing: stop (section 2).
4. Tick the step's checkbox in the plan and add one line to the Execution log: date, step, files changed, verification result.

## 2. Stop and ask when
- A decision isn't covered by the plan: a new dependency, an API or data model change, where a piece of logic lives.
- The plan doesn't match the code: a field or class doesn't exist, or a step is wrong.
- Verification still fails after two attempts.
- The next action is destructive or hard to undo: deleting data, migrations on shared data, deploying to a shared org or environment, force-pushing.

When you stop: explain what you found, give the options with their trade-offs, add it to the Execution log, and wait. Don't choose for the user.

## 3. Handoff report
When the steps are done, or you stopped, set `Status: in-review` and reply in the user's language with:
1. **Steps:** each step as done, partial, or not started, with the files changed.
2. **Verification:** the commands you ran and their results.
3. **Done when:** each criterion as met (with evidence), not met, or needs a manual check.
4. **Deviations:** anything that differs from the plan, and why.
5. **Review first:** the two or three places most likely to be wrong, with `file:line`.
6. **Not verified or unsure:** say it plainly.

## 4. After the user's review
- Corrections: apply only what the user asked for, verify, log them, and report again in the same format, covering only what changed.
- Approval: set `Status: done` and add the date to the Execution log.
