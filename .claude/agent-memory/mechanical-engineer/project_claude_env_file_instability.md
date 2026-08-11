---
name: project-claude-env-file-instability
description: This environment has spontaneously lost uncommitted files mid-task (2026-08-11) — commit deliverables to git promptly
metadata:
  type: project
---

On 2026-08-11, while building [[project-pantilt-design-option-b]], a large set of files
vanished from disk between two consecutive tool calls with no action of mine that should
have caused it:
- A loose (non-git-tracked) script at `/home/inot/CLAUDE/build_design_option_b.py`
  (~270 lines, built up over several successful `Edit` calls) disappeared entirely.
- A freshly-exported, freshly-verified 12MB STEP file at
  `/home/inot/CLAUDE/mechanical_design/Design-Option_B.step` (untracked, just written)
  also disappeared.
- Even **pre-existing, git-tracked** files in the `mechanical_design` repo
  (`ASSEMBLY_pan_tilt.STEP`, `pan_tilt_geared_28byj48.step`) showed up as deleted in
  `git status` at the same time — though these were recoverable via `git checkout -- .`
  since they were committed. The untracked new work was not recoverable at all.

**Why this matters:** this is not something I did (no `rm`, no destructive git command
was run) — it looks like an external reset/cleanup of the working tree that this project
folder is exposed to, independent of tool calls. It has only been observed once so far,
so treat it as a real but unconfirmed-frequency risk, not a certainty on every session.

**How to apply:** for any nontrivial CAD build or multi-step file-generation task in this
project, once a script/output has been validated (passes `.isValid()`, interference
checks, etc.), `git add` + `git commit` it into whichever tracked repo it belongs in
(`mechanical_design/` or `Docs/`) right away rather than leaving it as a loose uncommitted
file — do this before moving on to the next unrelated step, not only at the very end of
the task. If a file mysteriously vanishes mid-task, first check `git status --short` in
the relevant repo — a `git checkout -- .` may restore anything that was already committed.
