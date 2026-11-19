---
name: prune
description: Prune a steering file (CLAUDE.md, AGENTS.md, REVIEW.md, a SKILL.md, agent defs, memory files) line by line with three tests — single source of truth, sediment, no-op — and cut what fails. Use when asked to prune/trim/slim/de-bloat a steering file, when one has grown past ~100 lines, or as a periodic pruning pass.
disable-model-invocation: true
---

# Prune a steering file

Steering files only grow: adding feels cheap and safe, removing feels like stamping on someone's work. Every line costs context load and dilutes the lines that matter. Pruning = going back through and cutting, on a schedule.

Default verdict for a line: **cut**. A line stays only if it passes all three tests.

## The three tests (ask of every line)

| Test | Question | Fails when |
|---|---|---|
| 1. Single source of truth | Does this fact live anywhere else? | Same fact in another file, or twice in this one |
| 2. Sediment | Is this still true? | Library swapped, convention changed, guarded bug fixed, path/flag gone |
| 3. No-op | Does this change behaviour vs the default? | Model already does it, or it does nothing in this file's context |

### 1. Single source of truth
Duplication costs maintenance (edit N places), context (pay tokens N times) and prominence (duplicated fact outweighs its real importance, drowns unduplicated ones). Agent can't tell which copy is live. A fact with three homes is a fact you no longer control.
- Fix: keep the one authoritative copy (closest to the code/owner), replace others with a pointer (`see X`) or delete.
- Check across: root + nested CLAUDE.md/AGENTS.md, `~/.claude*/CLAUDE.md`, REVIEW.md mirrors, skills, agent defs, memory files, docs/, README.

### 2. Sediment
Once-true lines dragging the agent toward a world that no longer exists. Dangerous because it was correct once, or is correct only for some tasks.
- Verify against the repo now: `grep`/`ls` every named file, function, flag, script, command, path. Gone or renamed → cut or fix.
- Rule guarding a fixed bug → check git log / code; fixed → cut.
- True but narrow (applies to one subsystem/task) → move behind a pointer (nested CLAUDE.md, skill, doc) instead of always-loaded context.

### 3. No-op
True, on-topic, still worth zero tokens because the agent didn't need telling ("write a detailed commit message", "write clean code", "be careful", "test your changes").
- Test: delete it, would behaviour change? No → cut.
- Unsure on a load-bearing line → run the task with and without it and compare; no diff → cut.
- Keep: project-specific facts the model can't guess, deviations from default, hard constraints with a reason.

## Procedure

1. **Scope.** List target file(s). `git status` — skip files another session is editing. Note line/token count before.
2. **Inventory homes.** Collect every other steering file the target's facts could also live in (test 1 needs them).
3. **Line pass.** For each line/bullet emit one row: `line | SSOT | sediment | no-op | verdict (keep/cut/move/pointer/fix) | evidence`. Evidence for sediment = grep/ls result, not a guess.
4. **Edit.** Apply cuts. Merge duplicates into the authoritative home. Move narrow-but-true into pointers. Don't reword surviving lines beyond what's needed; keep owner's voice.
5. **Mirrors.** If the file has a required mirror (e.g. `REVIEW.md` ↔ `.claude/REVIEW.md`), keep them identical.
6. **Report.** Before/after line counts, table of cuts grouped by test, anything moved and where, anything kept despite doubt (and why). User-owned global files (`~/.claude*/CLAUDE.md`, memory): propose the diff, apply only cuts that are clear-cut; flag judgement calls.

## Guardrails
- Never cut a hard safety/prod/irreversibility rule as a "no-op" — those stay even if the model usually complies.
- Never cut something just because it's long; long-but-unique-and-true gets tightened, not deleted.
- One pass, one commit per repo; message lists counts (`prune: CLAUDE.md 180→70 lines; 40 dup, 35 sediment, 35 no-op`).
