# Contributing to ai-kaderskill

Thanks for considering a contribution to this workflow.

## Ways to contribute

- **Report friction**: if you tried the skill on a real project and a phase's instructions
  were ambiguous, produced the wrong behavior, or a hard rule got skipped, open an issue
  describing exactly what happened.
- **Propose a phase change**: if you think a phase should work differently (e.g. how Triage
  labels are chosen, how Wayfinder decision types are picked), open an issue explaining the
  scenario that motivates the change before sending a PR — `SKILL.md` is followed literally
  by an agent, so small wording changes can have large behavioral effects, and it helps to
  agree on the *behavior* first.
- **Share a worked example**: if you use this skill on your own project and are willing to
  make (part of) its ticket/spec trail public, we'd love to link to it as a second example
  alongside [ClashFlag](https://github.com/EssamKader/ClashFlag).

## Making changes to SKILL.md

- Keep the fixed phase order and the "stop and wait after every phase" hard rule — those are
  the core discipline this skill exists to enforce; changes that quietly remove a checkpoint
  should explain why in the PR description.
- Prefer precise, unambiguous instructions over general guidance — remember this file is
  read and followed by an LLM agent, not a human skimming for the gist.
- If you change a phase's behavior, update this README's "How it works" summary to match.

## Code of Conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). By participating, you
agree to abide by it.
