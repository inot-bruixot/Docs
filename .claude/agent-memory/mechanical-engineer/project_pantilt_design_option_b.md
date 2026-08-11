---
name: project-pantilt-design-option-b
description: Design Option B pan/tilt CAD model exists and its key part list/dimensions/output path
metadata:
  type: project
---

Design Option B (the fix for the over-simplified "Redesign A") has a finished CadQuery
build: `/home/inot/CLAUDE/mechanical_design/build_design_option_b.py` -> exports
`/home/inot/CLAUDE/mechanical_design/Design-Option_B.step` (both committed to the
`mechanical_design` git repo, commit `359dba5`).

**Why it exists:** the user disliked Redesign A
(`/home/inot/CLAUDE/Docs/PanTilt_Mini_28BYJ48_Redesign.pdf`, 110x126x85mm, 11 unique/19
total parts) because it merged/deleted parts and swapped bearing families away from the
original `ASSEMBLY_pan_tilt.STEP` (306.5x298.1x303.7mm, 17 unique/25 occurrences). Design
Option B (`/home/inot/CLAUDE/Docs/Design-Option_B.pdf`) keeps Redesign A's small size
class but restores the original's part-by-part structure: separate motor plate, separate
tilt hub, a real Cam Holder fork/pivot joint (not fused into the tilt platform), flanged
bearing families (608/625, downsized not swapped to plain press-fit), and 4 individual
spacer sleeves rather than bosses molded into neighboring parts.

**Result:** 16 unique parts / 22 occurrences, all valid solids, zero unintended boolean
interference (gear-tooth mesh at the two 20T/60T pinion-gear pairs excepted). Achieved
envelope 95 x 136 x 105mm (W x H x D) against a ~120 x 140 x 92mm target — width/height
landed close; depth overshot because the spec's mandatory "~40mm behind the tilt axis"
tilt-motor placement plus can radius (14mm) and M4 bolt-pattern half-width (~20mm)
structurally requires roughly that much protrusion behind the tilt axis, and the tilt axis
itself sits directly above the pan axis (Y=0) specifically to keep depth as tight as
possible. If depth must hit 92mm exactly, the tilt motor's 40mm offset or bolt pattern
would need to shrink, which conflicts with the PDF spec.

Key coordinates/params baked into the script (useful if resuming/iterating): pan axis at
world (0,0), pan gear z 8.3-14.3, Base2 top face z=15.3, pan motor can top z=37.3, tilt
axis at (y=0, z=105), tilt gear x=-3..3 (face width slot between Gear2Base's two towers,
gap 6.3mm), Cam Holder pivot pin axis along global Y at x≈31.5. Module 1.0 20T/60T gears,
CD=40mm both stages, bearing seat design follows the spec's shoulder/relief rule (see
[[feedback-cadquery-gotchas]] for the flange-recess detail that had to be added on top of
the spec's shoulder-only description to avoid real interference).

See [[feedback-cadquery-gotchas]] for reusable CadQuery bugs/fixes hit during this build,
and [[project-claude-env-file-instability]] for why the deliverable was committed to git
immediately after validation rather than left as loose files.
