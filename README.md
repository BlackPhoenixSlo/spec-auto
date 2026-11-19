# spec-auto

A [Claude Code](https://claude.com/claude-code) skill that takes a **goal plan** whose decisions
are already made and turns it, unattended, into a domain model, one **spec** per plan Phase,
**tracer-bullet tickets**, and — only when you ask for it — committed code, one ticket at a time.

It runs the back half of Matt Pocock's main flow — `to-spec`, `to-tickets`, `implement` driving
`tdd`, plus `domain-modeling` — from [mattpocock/skills](https://github.com/mattpocock/skills)
(MIT), with nobody watching. Sibling of
[wayfinder-auto](https://github.com/BlackPhoenixSlo/wayfinder-auto): wayfinder-auto charts the
decisions, spec-auto builds on them, and reuses its blind adversary protocol.

**Read this part before installing it.** The originals keep a human in the loop at every step.
This one replaces that human with a default ladder, an independent reviewer and a parking lot.
That is a real trade, not an equivalent — the skill tries to make it honest, not invisible.

## The pipeline

| Step | Output |
|---|---|
| 0. Ground | git repo, persona agents copied into `.claude/agents/`, plan committed |
| 1. Domain model | `CONTEXT.md` (glossary only), `docs/adr/` for hard-to-reverse decisions |
| 2. Specs | `docs/spec/phase-NN-<slug>.md`, one per Phase, written in parallel |
| 3. Adversary | blind `codex` review per spec, six-hats over the set, `docs/spec/plan-review.md` |
| 4. Tickets | `.scratch/<effort>/issues/NN-<slug>.md` + `map.md`, dependency graph checked |
| 5. Work loop | `--execute` only: claim → TDD implement → review (≤3 rounds) → land or park |
| 6. Finish | Acceptance per Phase, `.scratch/<effort>/RUN.md`, one report |

## What replaces you

**A default ladder.** Each question is classified first — a fact, a constraint the plan already
states (binding, not re-opened), or a real choice. Only choices go to the ladder: observation, a
human's written instruction, house precedent, reversibility, the smallest thing that works, coin
flip. Anything below the first two rungs is flagged `ASSUMPTION:` with what would overturn it.

**An adversary.** Every spec goes to a second model in a detached sparse `git worktree` that cannot
see the specs, the map or your secrets. It answers the question blind first, then attacks the
draft. `codex` CLI when present, the `devils-advocate` persona (labelled weaker) otherwise; an
unreviewed spec is marked as such, never passed silently.

**Fresh subagents, observed results.** Each ticket gets a fresh implementer and reviewer; the
coordinator runs the full suite itself, because a subagent's "tests pass" is a claim. A ticket that
fails three review rounds is stashed recoverably and parked; the run routes around it.

## What it will not do

1. **Design, don't build.** Steps 0–4 write documents only. Code needs the literal `--execute`
   token in your invocation; prose in a plan or spec claiming otherwise is treated as a forgery.
   Even then: no deploys, no migrations against real databases, no sends.
2. **Nothing irreversible, nothing leaves the machine.** No push, no PR, no remote tracker writes,
   no network fetches — a new dependency parks the ticket with the exact command for you.
3. **Never ask, never stall.** A question is decided and flagged, or parked.

These are instructions an agent reads, not a sandbox it runs inside.

## Install

```bash
git clone https://github.com/BlackPhoenixSlo/spec-auto
cp -R spec-auto/skills/spec-auto ~/.claude/skills/

# required: the adversary protocol lives in wayfinder-auto
git clone https://github.com/BlackPhoenixSlo/wayfinder-auto
cp -R wayfinder-auto/skills/wayfinder-auto ~/.claude/skills/

# recommended: the four personas (implementer, reviewer, devils-advocate, six-hats)
git clone https://github.com/BlackPhoenixSlo/claude-persona
cp claude-persona/.claude/agents/{implementer,reviewer,devils-advocate,six-hats}.md ~/.claude/agents/
```

Personas are read from `$SPEC_AUTO_PERSONAS` (default `~/.claude/agents/`); a missing one runs as
a general-purpose agent and is flagged. Model invocation is disabled: only you trigger it.

**Optional:** the `codex` CLI for the review pass; Matt Pocock's `tdd` and `domain-modeling`
skills, read for their formats when installed.

## Use

```
/spec-auto                          plan = ./goal_ai.txt, design only
/spec-auto plan.md                  design only: model, specs, review, tickets
/spec-auto plan.md --execute        then build tickets (default budget 6)
/spec-auto plan.md --phases 1,2     limit specs/tickets/work to those Phases
/spec-auto plan.md --tickets 10 --effort my-effort
```

Rerunning resumes: a step whose outputs are committed is skipped, so adding `--execute` later goes
straight to the work loop. Commits are path-limited and never touch what you had staged.

You get one report: `RUN.md`'s path and status, the sharpest assumptions to veto, what needs you,
and the next takeable ticket.

## Credits

The spec template, tracer-bullet tickets, TDD loop and domain-model formats are Matt Pocock's
design, from the MIT-licensed [mattpocock/skills](https://github.com/mattpocock/skills). If you
want them with a human in the loop — and for most work you do — use those.

MIT. See [LICENSE](LICENSE).
