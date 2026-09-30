# Project Board

Use the [Aura project board](https://github.com/users/mwilsoncoding/projects/6) as the planning view.

## Milestones

- Assign each Aura issue to the milestone matching its current phase when that phase is known. New issues may be created without a milestone while their phase or future placement is unresolved; the Project Manager must triage them into an appropriate milestone before work starts.
- The Project Manager may create a milestone when no existing milestone fits the agreed phase. Leave its due date unset until a target is agreed.
- Update the GitHub issue milestone whenever scope or phase changes, and keep the board's milestone value consistent with the issue.
- `Aura v0.0.1 Specification` covers the Wayfinder map and its decision tickets. Implementation work belongs in a separate milestone once that phase is agreed.

## Issue Planning

Use `/project-manager` when creating an issue, starting work, or maintaining project-board fields.

- The board tracks `Estimated start`, `Estimated completion`, `Priority`, and `Status`. Inspect the live schema for field types and valid options before setting values.
- On issue creation, use the skill to evaluate and set the estimated-start date and priority using the live board schema and existing priority scale.
- When an agent begins work, use the skill to fill the estimated-completion date with a forecast.
- Revisit priority and date forecasts when scope or dependencies change. Dates are forecasts, not commitments.
- Follow the skill's scope and schema checks. If required fields are inaccessible or missing, or a forecast cannot be supported, state the blocker and ask rather than guessing.