---
name: ai-kaderskill
description: Runs the full Scope → Advisor Strategy → Wayfinder → To-Spec → To-Tickets → Triage → Implement → Review cycle for any feature or task in this project. Use whenever the user wants a structured, ticket-based build instead of one long unstructured implementation. After every phase, tells the user exactly what happens next and waits for their go-ahead before proceeding — never silently jumps ahead.
---

# AI-KaderSkill (Ticket Cycle)

You are running a fixed, ordered pipeline. Follow the phases below **in order, one at a time**. Do not skip a phase or merge two phases into one turn.

**Hard rule:** at the end of every phase, before doing anything else, output a message in exactly this shape:

```
✅ Phase [N] done: [one-line summary of what just happened]
👉 Next: [Phase N+1 name] — [one sentence on what it will do]
[If input is needed from the user, ask for it here. Otherwise ask: "Go ahead?"]
```

Then **stop and wait** for the user's reply before starting the next phase. Never proceed automatically.

---

## Phase 0 — Setup check (run once per repo, skip if already confirmed this session)

- Confirm the project folder is a Git repo linked to a remote (GitHub/GitLab/etc). **Default to creating a real GitHub repo for the project before doing anything else, if one doesn't already exist** — check `gh auth status` first; if authenticated, create the repo (`gh repo create`, ask the user for name/visibility/license rather than guessing) and push the initial commit, instead of leaving the project as a local-only sandbox. Only skip this and stay local if the user explicitly says so (e.g. a throwaway experiment) or no GitHub auth is available.
- Confirm a context file exists (e.g. `CONTEXT.md`) and read it if present. If it doesn't exist and the project has meaningful standing rules or conventions, offer to create one — ask the user what belongs in it rather than guessing.
- **Default tracker: GitHub Issues on that repo**, not local ticket files — tickets become issues (`gh issue create`), triage labels become real GitHub labels, and "close the ticket" in every later phase means closing the issue (`gh issue close`), not editing a `label:` line in a markdown file. Confirm this default with the user rather than assuming silently; fall back to local files or Linear only if the user prefers that or no GitHub repo is in play for this project. Either way, confirm that these five triage labels exist: `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`. Create any missing ones (`gh label create` for GitHub Issues).
- If any MCP servers or external tools are required for this project, confirm they're connected/toggled on before proceeding.
- **If the repo is public on GitHub, set up branch protection on the default branch** (`gh api .../branches/<default>/protection`): require pull requests with at least 1 approving review, disable force-pushes and branch deletion. Leave `enforce_admins` off so the project owner can keep pushing directly after an in-conversation review — protection exists to gate outside contributors and guard against accidental force-push/deletion, not to add ceremony to the owner's own already-reviewed changes. Skip this for a private/solo-only repo unless the user asks for it anyway.

## Phase 1 — Scope the task

Ask the user (if not already stated) what they want to work on. Based on the answer, decide:
- **Big / ambiguous** (spans many components, unresolved design questions, unclear approach) → go to Phase 2 (Advisor Strategy), then Phase 3 (Wayfinder).
- **Small / already clear** (one specific, well-understood change) → skip straight to Phase 4 (To-Spec).

State which path you're taking and why in the "Next" message.

## Phase 2 — Advisor Strategy (only if routing to Wayfinder)

- Run `/consult` (the strategic model-advisor command) with a short description of the task and why Phase 1 classified it as big/ambiguous.
- Present the resulting Consult Report to the user in full before spending any effort on Wayfinder's own decomposition — recommended model, reasoning, token strategy, and the suggested start prompt.
- This is advisory, not a gate: the user may switch models via `/model`, or just say to proceed with the current one. Never choose or switch the model on the user's behalf.
- Skipped entirely on the small/already-clear path — that path never sees Wayfinder-scale ambiguity in the first place.
- `/consult` is a personal command, not bundled with this skill — if it isn't available in this environment, say so plainly and skip straight to Phase 3 rather than asking the user to install a specific command.

## Phase 3 — Wayfinder (only if triggered above)

- Open one root issue/ticket labeled `wayfinder:map`.
- Break the ambiguity into decision tickets, each tagged by type: Research, Grill, Prototype, Routine, or Manual.
- Resolve them one at a time. For **Grill** tickets, ask the user directly — don't guess. For **Research** tickets, investigate and write a short markdown summary before moving on.
- Do not proceed to Phase 4 until every decision ticket under `wayfinder:map` is resolved or explicitly deferred by the user.

## Phase 4 — To-Spec

- Convert the resolved decisions (from Wayfinder, or from the Phase 1 conversation if Wayfinder was skipped) into a spec document of user stories: *"As [role], I want [goal], so that [reason]."*
- Save it to the project (e.g. `specs/<short-name>.md`) and post it in full for the user to confirm before moving on.

## Phase 5 — To-Tickets

- Split the confirmed spec into tracer-bullet tickets. Each ticket must declare its own blocking dependencies on other tickets explicitly.
- Open them as real tickets on the configured tracker.
- Every new ticket starts labeled `needs-triage`.

## Phase 6 — Triage

For each open ticket, assign one label:
- `ready-for-agent` — fully clear, no open questions, no design decision pending.
- `ready-for-human` — clear, but the work itself isn't something an agent should do (a design call, physical/manual work, a business decision).
- `needs-info` — missing a specific detail; ask the user for exactly that detail.
- `wontfix` — explicitly out of scope; confirm with the user before applying.
- Leave as `needs-triage` if genuinely undetermined.

Only tickets labeled `ready-for-agent` move to Phase 7.

## Phase 7 — Implement (delegate the actual coding)

For the next `ready-for-agent` ticket:
- Do **not** write the implementation yourself.
- Delegate it to an implementer subagent. If one doesn't exist yet for this project, create it at `.claude/agents/implementer.md` the first time it's needed — ask the user what model tier and tools it should have, don't assume. Whatever model tier they choose, also include `effort: medium` in the generated frontmatter alongside the `model:` line (e.g. `model: sonnet` / `effort: medium`) — this is the project-wide default for every subagent this skill generates, not just a one-off choice for the first one.
- Enforce any standing project rules from `CONTEXT.md` regardless of which model implements.

## Phase 8 — Review

- Run `/code-review` (or the project's configured review step) on the returned change before anything is committed.
- If it fails, delegate the fix back to the same subagent with the specific review comments — don't fix it yourself.
- On pass: close the ticket, and go back to Phase 6 to triage/pick the next ticket. **Closing a ticket does not by itself cut a release** — see "Versioning & Release" below.
- If no `ready-for-agent` or `needs-triage` tickets remain, tell the user the cycle is complete and summarize what was closed.

## Versioning & Release (applies whenever this project's code is deployed anywhere outside the repo itself — an installed extension, a published package, a running service)

A merge to the default branch means "the code exists," not "this is safe to deploy." Keep those two separate, deliberate steps:

- Maintain a `CHANGELOG.md` at the project root, one entry per release, listing which tickets/issues it closes.
- Only cut a version tag (semantic versioning, e.g. `v0.5.0`) and a GitHub Release once a batch of closed tickets has actually been verified and the user wants to deploy — never tag automatically just because a ticket closed; ask first, same as any other hard-to-reverse/public action.
- Treat the tagged release, not the default branch's HEAD, as the only thing that's "safe to install/deploy" — when copying code to a live install (a deployed extension, a running service, etc.), deploy from a tagged commit, not from whatever HEAD happens to be at the time.
- For any code that can't be executed or unit-tested in this environment (e.g. IronPython/pyRevit code with no live host available, or any other host-dependent runtime), require a standalone verification write-up — a mock-object simulation proving the logic — before a ticket touching that logic can close in Phase 8 (Review). This is the actual safety net when the real runtime isn't reachable, not optional polish.

## Throughout

- If context usage is climbing, run `/compact` instead of ending the session. Preserve: everything in `CONTEXT.md`, and any decisions made in this run's Wayfinder phase.
- Never invent a decision on the user's behalf for a Grill-type or design question — always ask.
