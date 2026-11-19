# Subagent briefs

Read the brief you are about to dispatch, fill every `<angle bracket>`, and send the Header followed by everything
under the brief's heading. Briefs are plain text to paste, not instructions to you.

## Header — opens every brief

You are a fresh subagent in an unattended /spec-auto run. Nobody can answer a question: decide it by the ladder, or
report it as a block.

<The three floors, verbatim from SKILL.md.>

<"Deciding without the human", verbatim from SKILL.md: the classification, the ladder, the flag rule.>

Read CONTEXT.md and docs/adr/ first if they exist; use the glossary's terms and respect the ADRs. Write only the
files named below. Return the paths you wrote, a three-line summary, and every ASSUMPTION: line you added — unless
your persona or this brief names a report format, in which case use that.

## Domain

Plan: <plan path>. Read it whole.

Build the domain model the specs will speak. If present, read CONTEXT-FORMAT.md and ADR-FORMAT.md beside
domain-modeling/SKILL.md under ~/.claude/plugins/cache/mattpocock/ (Glob for them). In short:

- CONTEXT.md is a glossary and nothing else. Only terms specific to this project; one or two sentences each saying
  what the term IS; an `_Avoid_:` line naming the synonyms you rejected. Be opinionated: where the plan uses two
  words for one thing, pick one. Group under subheadings when clusters emerge. Implementation detail belongs in the
  specs.
- docs/adr/NNNN-<slug>.md for a plan decision only when all three hold: hard to reverse, surprising without
  context, the result of a real trade-off. One paragraph — context, decision, why — plus Considered Options only
  when a rejected alternative is worth remembering. Number on from the highest existing ADR.
- Extend existing files. An entry a human wrote stands; a conflict with it goes in your summary as an ASSUMPTION:.

Files: CONTEXT.md, docs/adr/*.
Done when every decision the plan states is accounted for: an ADR, or named in your summary with the test it fails.

## Spec — one per Phase, in parallel

Plan: <plan path>. Read it whole; decisions outside your Phase's section bind you too.
Your Phase: <label> — <title>. File: docs/spec/phase-<NN>-<slug>.md.
Other Phases are being specced in parallel: refer to them by label, and take nothing about them beyond what the
plan says.

Explore the repo enough to place this Phase in the code that exists. Then fix the seams before you write: where
this Phase is tested. Prefer existing seams to new ones and the highest seam possible; the fewer the better, and
the ideal is one.

Fill the template. Every section is present; a genuinely empty one reads `None — <reason>`.

<spec-template>
# Phase <NN> — <title>

**Objective.** One sentence: what is true when this Phase is done.

## Problem Statement
The problem, from the user's perspective.

## Solution
The solution, from the user's perspective.

## User Stories
A long, numbered list, exhaustive over the Phase:
1. As a <actor>, I want <feature>, so that <benefit>

## Implementation Decisions
- **Owns.** The modules this Phase builds or modifies, by name.
- **Interfaces.** What each owned module exposes, and what changes.
- **Schema.** Tables, fields, constraints — or None.
- **Contracts.** APIs, events and formats that cross a boundary.
- Every other architectural decision and specific interaction.
Decisions are stated in prose and module names, because file paths and code go stale. The one exception is a
schema or type shape that states a decision more precisely than prose; trim it to the decision.

## Testing Decisions
The seams, by module and public interface. A good test here verifies external behaviour, never implementation.
Prior art: similar tests already in the repo, or None.

## Acceptance
A fenced sh block of commands that all succeed once the Phase is done. A step only a human can take is a
`# manual:` comment.

## Depends on
Each Phase that must land first, and what it provides — or None.

## Out of Scope
Cut (YAGNI): each item with its reason.

## Further Notes
Open questions, each already decided by the ladder and flagged ASSUMPTION:.
</spec-template>

File: docs/spec/phase-<NN>-<slug>.md.
Done when every section is filled, every plan line about this Phase is covered by a user story or listed under Out
of Scope, and the Acceptance block is runnable shell.

## Adversary — one per spec, in parallel

Spec: docs/spec/phase-<NN>-<slug>.md. Workspace: <WS>, blind to docs/spec/. Plan inside it: <WS>/<plan>.
Source inside it: <paths, or "none — docs-only review">.

Read ~/.claude/skills/wayfinder-auto/reference/adversary.md and follow its invocation, preflight
and postflight exactly, with your own `mktemp -d` for the prompt and output files. Its Phase 1 is the blind call
here, and its Phase 2 the draft call.

Blind call prompt:
  An unattended run is specifying Phase <label> — <title> of the plan at <plan>. Read the plan, CONTEXT.md,
  docs/adr/ and <source paths>. Before you see any spec: what are the real alternatives for this Phase, which
  requirements does the plan leave out, at which seam should it be tested, and what evidence would falsify its
  premise? Put a file:line, or a command and its output, behind every claim.

Draft call prompt:
  Here is the spec written for Phase <label>, below the line. Where is it wrong — not where could it be polished?
  Look for decisions that contradict the plan or an ADR, user stories no seam tests, an Acceptance block that would
  pass while the Phase is broken, scope the plan never asked for, and a Depends on that is missing or false. Put a
  file:line, or a command and its output, behind every claim.
  ---
  <the spec, without ## Review>

Fallback — codex missing, unauthenticated, or failing postflight: run the same two calls with the devils-advocate
persona, confined to <WS>, and label the review `weaker (devils-advocate)`. If that fails too, the spec is
unreviewed: write a `## Review` that says so and mark it needs-human. Silence is never agreement.

Reconcile into the spec. `## Review` opens with the reviewer label (`codex` or `weaker (devils-advocate)`) and the
date, then one line per finding from both calls: **accept** — rewrite the decision in its own section, since
accepting a caveat is not accepting a finding — **partial**, or **reject**, each with its reason. Accepting every
finding is legitimate; so is rejecting every one. A material objection still unanswered by evidence after the draft
call is **needs-human**: both positions, and the evidence that would settle it.

File: the spec.
Done when every finding from both calls has a line.

## Six hats

Read <plan>, every docs/spec/phase-*.md with its ## Review, CONTEXT.md and docs/adr/. Put the hats on the set as a
whole: the cut between Phases, their order, what the set misses, and what it builds that the plan never asked for.
Report in your persona's format.

## Plan review

Six-hats report, verbatim:
<report>

1. Reconcile each bullet as the Adversary brief does — accept, partial or reject, with a reason — into the
   `## Review` of each spec it touches; an accept rewrites the decision. A bullet about the set as a whole goes only
   to plan-review.md.
2. Write docs/spec/plan-review.md:
   - `## Workstreams` — table `| Wave | Phase | Spec | Depends on |`; a Phase sits in the first wave after every
     Phase it depends on.
   - `## Adversary` — one line per spec: reviewer label; accepted, partial and rejected counts; any needs-human.
   - `## Six hats` — the reconcile lines for bullets about the set.
   - `## Cut (YAGNI)` — the specs' Out of Scope, deduplicated, each with its reason.
   - `## Not yet specified` — plan lines no spec covers.
   - `## Open questions` — every ASSUMPTION: across the specs, sharpest first, each with what falls if it is wrong.
   - `## Needs the human` — every needs-human finding, by spec.

Files: the specs, docs/spec/plan-review.md.
Done when every six-hats bullet has a reconcile line and every spec appears under Adversary.

## Tickets — one per spec, serially

Spec: docs/spec/phase-<NN>-<slug>.md, its ## Review included. Effort: <effort>. Existing tickets:
.scratch/<effort>/issues/ — read their titles and Blocked by lines, and number yours on from the highest.

Break the spec into tracer bullets:
- Each ticket cuts a narrow but complete path through every layer the Phase touches — schema, API, UI, tests.
  Vertical, never one layer.
- Each is demoable or verifiable on its own, and fits one fresh context window.
- Prefactoring comes first. A wide refactor runs expand → migrate in batches → contract, each its own ticket.

Blocked by: the tickets in this spec that gate it and, for each Phase in the spec's Depends on, the earlier tickets
that provide what this one needs. A ticket resting on a needs-human decision in ## Review gets
`Status: parked — needs-human: <decision>`.

<ticket-template>
# <NN>: <Ticket title>

Spec: docs/spec/phase-<NN>-<slug>.md
Covers: user stories <n, n, n>
Seams: <the Testing Decisions seams this ticket tests at>
Blocked by: <NN: title, …> — or None (can start immediately)
Status: ready-for-agent

**What to build:** the end-to-end behaviour this ticket makes work, from the user's perspective — not a
layer-by-layer list.

- [ ] Acceptance criterion
</ticket-template>

Titles and bodies speak the glossary and describe behaviour; file paths and code snippets go stale, so leave them to
the implementer.

Files: .scratch/<effort>/issues/<NN>-<slug>.md.
Done when every user story in the spec appears in some ticket's Covers: line.

## Implement

Ticket: .scratch/<effort>/issues/<NN>-<slug>.md. Spec: <spec path>. Floor 1 is lifted for this ticket's code and
tests, and nothing else.

The spec's Implementation Decisions bind you. Its Testing Decisions are the pre-agreed seams — already reviewed, so
they need no confirmation — and the only place tests go.

Work /tdd (read tdd/SKILL.md under ~/.claude/plugins/cache/mattpocock/ if present):
- Red before green: one failing test at a seam, then the least code that passes it. One vertical slice per cycle.
- Tests verify behaviour through public interfaces. Expected values come from the spec or a worked example, never
  recomputed the way the code computes them.
- Run the typecheck and the single test file regularly, and the full suite once at the end.
- Stay inside the ticket. Refactoring belongs to the review.

Decide an ambiguity by the ladder and flag it. Report a denied tool, a missing credential or a needed network fetch
as a block. Leave your changes in the working tree; the coordinator commits.

Return your persona's report, plus the full-suite command.

## Review

Ticket: <ticket path>. Spec: <spec path>. Round <n> of 3. Prior findings, verbatim: <findings, or none>.

Review the ticket's change: `git status`, `git diff HEAD`, and every untracked file `git status` lists. Beyond your
bar, two things block approval: a test at a seam the spec's Testing Decisions do not name, and an acceptance
criterion in the ticket with no test behind it.

## Fix

Send the Implement brief, filled, with this appended:

Reviewer findings, verbatim:
<findings>

Failing test output, verbatim (tail):
<output, or none>

Close each finding, or dispute it with file:line evidence. Return your persona's report, then one line per finding:
closed or disputed.
