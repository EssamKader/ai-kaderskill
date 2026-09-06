# ai-kaderskill

A structured, ticket-based development workflow for [Claude Code](https://claude.com/claude-code),
packaged as a reusable **skill**. Instead of one long unstructured conversation turning
directly into code, ai-kaderskill runs every feature or task through a fixed pipeline:

```
Setup → Scope → Advisor Strategy → Wayfinder → To-Spec → To-Tickets → Triage → Implement → Review
```

At the end of every phase, it tells you exactly what just happened and what happens next,
and waits for your go-ahead — it never silently jumps ahead, and it never invents a design
decision on your behalf when the honest answer is "ask the user."

## Why

AI coding agents are good at writing code and bad at knowing when to stop and ask. This
skill forces a checkpoint after every phase of the work — scoping, spec-writing, ticket
breakdown, triage, implementation, and review — so you stay the one making the actual design
calls, while the agent still does the actual typing, ticket-writing, and delegation.

## Install

Copy the `ai-kaderskill/` folder from this repo into your own `.claude/skills/` directory:

```
.claude/skills/ai-kaderskill/SKILL.md
```

Then invoke it in Claude Code with `/ai-kaderskill` (or just describe a task — Claude Code
picks it up automatically when it matches).

## How it works

- **Phase 0 (Setup)**: confirms your project is a real Git repo with a real tracker (GitHub
  Issues by default — tickets become issues you open and close with real labels; falls back
  to local files or Linear only if you say so), and that a `CONTEXT.md` glossary exists if
  the project needs one.
- **Phase 1 (Scope)**: decides whether what you're asking for is small and clear (skip
  straight to spec) or big/ambiguous (needs Wayfinder first).
- **Phase 2 (Advisor Strategy)**: only for ambiguous work, right before Wayfinder — runs a
  `/consult`-style model advisor on the task so you know which model tier suits the ambiguity
  you're about to spend effort resolving, before that effort is spent. Purely advisory, and
  skipped entirely on the small/clear path.
- **Phase 3 (Wayfinder)**: only for ambiguous work — breaks the ambiguity into decision
  tickets (Research/Grill/Prototype/Routine/Manual) and resolves them one at a time, asking
  you directly for anything only you can decide.
- **Phase 4 (To-Spec)**: turns resolved decisions into a spec of user stories, confirmed with
  you before anything gets built.
- **Phase 5 (To-Tickets)**: splits the spec into tracer-bullet tickets with explicit
  dependencies.
- **Phase 6 (Triage)**: labels every ticket — ready for an agent, ready for a human, missing
  info, or out of scope — never guesses.
- **Phase 7 (Implement)**: delegates the actual coding to a subagent, never writes it inline.
- **Phase 8 (Review)**: runs a real review step before anything is closed; failures go back
  to the same subagent with specific comments, not a silent rewrite.
- **Versioning & Release**: a merge is not a deploy. Closed tickets accumulate on the default
  branch until you deliberately cut a version tag and GitHub Release — that tag, not the
  branch's HEAD, is the only thing treated as safe to install or deploy. Public repos also get
  branch protection by default (PRs + 1 approval for anyone but the owner, no force-push/delete).

## See it in action

[**ClashFlag**](https://github.com/EssamKader/ClashFlag), a pyRevit clash-detection add-in,
was built end-to-end using this exact skill. Its `tickets/` and `specs/` folders are the
full, unedited record — every design decision, every review round (including the ones that
failed), and the real debugging sessions where things went wrong before they went right.
If you want to see what this workflow actually produces in practice, that repo is the proof.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) and the [Code of Conduct](CODE_OF_CONDUCT.md).

## License

[MIT](LICENSE)
