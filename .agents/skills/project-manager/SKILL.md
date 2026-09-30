---
name: project-manager
description: Maintain delivery milestones, issue fields, date forecasts, priorities, and estimation calibration.
disable-model-invocation: true
---

# Project Manager

Manage delivery planning for work with sufficiently clear scope. Wayfinder resolves open scope and design decisions; Project Manager organizes and forecasts the agreed work through delivery. Use this skill to maintain milestones and project fields, prioritize and group issues, or compare estimates with completed work.

## Workflow

### 1. Establish the planning scope

- Read `AGENTS.md` for the repository, project board, and current milestone. Read `docs/agents/issue-tracker.md` and follow `/gh` for GitHub operations. Read `docs/agents/triage-labels.md` when assigning triage or readiness labels.
- Identify the requested repository, board, milestone, issue set, and requested mode: milestone or field maintenance, prioritization, scheduling, or estimate calibration. Keep queries and changes within that scope.
- If a decision about goals, requirements, or solution scope is still open, surface it for resolution through Wayfinder before scheduling the affected work.
- Treat an explicit request to maintain fields or prioritize the board as authorization for changes within the named scope. When the intended issues, values, or tradeoffs are ambiguous, present the proposed changes and get direction first. For analysis-only requests, leave tracker data unchanged.

Done when the repository, board, work scope, and requested outcome are unambiguous; otherwise report the specific missing information or access.

### 2. Read the current plan

- Inspect the target milestone, its issues, dependencies, relevant project items, and linked pull requests before proposing changes.
- Discover field names, types, options, and item membership from the live project schema. Map the requested estimated-start, estimated-completion, priority, and status concepts to the fields that actually exist.
- The Aura board tracks `Estimated start`, `Estimated completion`, `Priority`, and `Status`; verify their live types and options before setting values.
- Keep each GitHub issue's milestone assignment consistent with the milestone shown on the project board. Treat a milestone due date as a milestone-level target, separate from per-issue date forecasts.
- If the board schema is inaccessible, report the access gap and request access or the field names; do not infer that a field is missing. If the estimated-start, estimated-completion, or requested priority field is absent from an accessible schema, ask the human to create it or confirm an existing field mapping, then re-read the schema before proceeding. Ask before any other board-schema change; do not guess field names or values.

Done when the current values and exact editable fields for every target are verified, or the blocker is stated clearly.

### 3. Prioritize and schedule

- Rank only work in the agreed scope. Use the project's existing priority scale and explain each placement with the evidence that matters: milestone goals, user impact, urgency, dependencies, risk reduction, and readiness.
- Put prerequisite work ahead of work it blocks. Group issues by meaningful milestone, dependency chain, or readiness state so a person or agent can select a coherent next task.
- Use the repository's readiness labels and native dependency relationships where applicable. Load their local conventions rather than inventing a parallel vocabulary.
- Set date forecasts only when scope, dependencies, and available capacity support them. Schedule dependent work after its blockers; when capacity or a real constraint is unknown, state the assumption or ask instead of presenting a guess as a commitment.
- Treat estimated start and completion dates as forecasts, not guarantees or milestone due dates. Keep forecasts at the precision supported by the project fields.

Done when every in-scope issue is either ranked and grouped with a reason, or explicitly marked as blocked or not ready, and each proposed date has a stated basis.

### 4. Maintain the tracker

- Make only the requested milestone, issue, and project-field changes. Do not change project field schema.
- New issues may remain unmilestoned while their phase or future placement is unresolved. Triage each such issue into an appropriate milestone before work starts. The Project Manager may create a milestone when no existing milestone matches the agreed phase; leave its due date unset until a target is agreed.
- Preserve the original estimate baseline when a forecast changes: record the previous dates, new forecast, change date, and reason in the issue history before replacing the current values. Distinguish that baseline from the latest forecast.
- Apply the repository's milestone convention to every in-scope issue and keep the project-board milestone value in sync. Leave milestone due dates unset when no target date has been agreed.
- Use `/gh` for mutations and read each changed issue or project item back to verify the intended values.

Done when every requested mutation has been read back and matches the intended plan; report any partial update or failed verification.

### 5. Calibrate estimates against actuals

- Use completed, comparable issues with a preserved estimate baseline. Read issue state and history, linked pull-request creation and merge times, and issue-linked git commits. Keep the evidence source for each actual date visible in the analysis.
- Define actual start from a reliable recorded transition to in-progress when available; otherwise use the earliest attributable pull request or substantive linked commit. Define actual completion from the issue's closed time when closure represents accepted work; otherwise use the merge time of the last required linked pull request.
- Mark actual dates unavailable when the evidence cannot support them. Issue creation time is not a start date; rewritten commit author dates are not reliable work-start evidence.
- Compare baseline estimated start and completion with actual start and completion at calendar-day precision. Report start and completion drift, estimated versus actual elapsed duration, sample size, exclusions, and evidence quality.
- Calibrate future forecasts from repeated patterns in comparable work. Show the direction and size of any observed bias with its sample count; keep one-off or sparse evidence as a caveat instead of turning it into a correction.
- Preserve original estimates after completion. Use changes to improve future forecasts, not to make past estimates appear accurate.

Done when every included issue has a supported baseline and actual, or a stated exclusion, and the resulting trend is reported with its sample size and limitations.

## Report

Summarize the scope, milestone state, prioritized groups and rationale, forecast assumptions, verified field changes, and calibration evidence relevant to the request. Name unresolved blockers and distinguish estimates from commitments.