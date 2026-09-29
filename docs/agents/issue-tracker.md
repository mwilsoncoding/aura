# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations; it infers the repo from `git remote -v`.

## Conventions

- Create: `gh issue create --title "..." --body "..."`
- Read: `gh issue view <number> --comments`
- List: `gh issue list` with appropriate state and label filters
- Comment: `gh issue comment <number> --body "..."`
- Label: `gh issue edit <number> --add-label "..."` or `--remove-label "..."`
- Close: `gh issue close <number> --comment "..."`

PRs as a triage request surface: no.

## Wayfinding operations

A Wayfinder map is an issue labelled `wayfinder:map`. Tickets are child issues linked as GitHub sub-issues; if unavailable, use a task list on the map and `Part of #<map>` on each child. Ticket labels are `wayfinder:research`, `wayfinder:prototype`, `wayfinder:grilling`, or `wayfinder:task`.

Use native GitHub issue dependencies for blocking. Add a blocker through `repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by`, passing the blocker's numeric database ID. If dependencies aren't available, use a `Blocked by: #<n>` line in the child body.

The frontier is open child tickets with no open blockers and no assignee. Claim with `gh issue edit <n> --add-assignee @me`. Resolve by commenting with the answer, closing the ticket, then adding a context pointer to the map's Decisions-so-far.