# Aura

## Project direction

Treat [README.md](README.md) as the source of truth for Aura's goals and non-goals. Aura is an exploratory language and compiler project; its stated influences are BEAM/OTP, Perceus reference counting, Koka effect handling, Elixir-like set-theoretic types, and Verse. Treat those influences as research leads, not settled design requirements. Make unresolved tradeoffs explicit before turning them into implementation constraints.

Keep the Rust implementation dependency-free: use only the standard library and add no external crates.

In the following priority order:

1. Make it work.
2. Make it fast.
3. Make it readable.

## Workflow

- Use a dedicated git worktree for each independent agent or sub-agent task that needs a separate checkout. Prefer worktrees over switching branches in a shared checkout, so parallel work does not overwrite or disturb the human's or another agent's state. Check `git worktree list` before creating one, and leave other worktrees and their changes untouched.
- Use [`/wayfinder`](.agents/skills/wayfinder/SKILL.md) early and often for substantial, exploratory, or multi-session work. Keep its map and decision tickets on the configured issue tracker, and resolve decisions before treating the route as implementation-ready. Follow the skill's one-ticket-per-session rule.
- Use [`/grill-me`](.agents/skills/grill-me/SKILL.md) when ambiguity in scope, semantics, or design could change the direction of the work. Clarify that ambiguity before committing it to a Wayfinder decision or implementation.
- Use [`/gh`](.agents/skills/gh/SKILL.md) for GitHub-hosted context and operations. `gh` is available in the devcontainer/Codespaces environment; establish the repository and authentication context before repository-scoped operations, and verify mutations.
- Use [`/research`](.agents/skills/research/SKILL.md) when a design decision depends on external facts; prefer first-party documentation, papers, and source code. Use [`/domain-modeling`](.agents/skills/domain-modeling/SKILL.md) when language concepts or terminology are being defined, and [`/codebase-design`](.agents/skills/codebase-design/SKILL.md) when module boundaries or interfaces are in question.
- Use [`/implement`](.agents/skills/implement/SKILL.md) for work with a clear spec or resolved ticket. Use [`/tdd`](.agents/skills/tdd/SKILL.md) for behavior changes at agreed public seams, [`/diagnosing-bugs`](.agents/skills/diagnosing-bugs/SKILL.md) for reported failures, and [`/code-review`](.agents/skills/code-review/SKILL.md) when reviewing changes.

## Upstream references

Use these sources to compare behavior and design; record which version or revision informed a decision when that matters.

- [Koka source](https://github.com/koka-lang/koka) and [Koka documentation](https://koka-lang.github.io/koka/doc/book.html)
- [Erlang/OTP source](https://github.com/erlang/otp) and [OTP Design Principles](https://www.erlang.org/doc/system/design_principles.html)
- [Elixir source](https://github.com/elixir-lang/elixir) and [Elixir typespecs and behaviours](https://elixir-lang.org/getting-started/typespecs-and-behaviours.html)
- [Rust source](https://github.com/rust-lang/rust) and [The Rust Reference](https://doc.rust-lang.org/reference/)
- [Perceus: Garbage Free Reference Counting with Reuse](https://arxiv.org/abs/2004.03570)
- [Semantic Subtyping](https://doi.org/10.1145/1040305.1040316), a foundation for reasoning about set-theoretic types
- [Set-theoretic types research by Giuseppe Castagna](https://www.irif.fr/~gc/)
- [Verse language book](https://verselang.github.io/book/00_overview/)

## Agent skills

### Issue tracker

Issues live in GitHub Issues. See `docs/agents/issue-tracker.md`.

### Triage labels

Use the default triage labels. See `docs/agents/triage-labels.md`.

### Domain docs

Use the single-context layout. See `docs/agents/domain.md`.