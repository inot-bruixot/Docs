---
name: feedback-cadquery-gotchas
description: Reusable CadQuery (this env's version 2.7.0) bugs/fixes found while building complex multi-part assemblies
metadata:
  type: feedback
---

Hit and fixed three real CadQuery gotchas while building
[[project-pantilt-design-option-b]]. Apply these on any future CadQuery script in this
project (or any script using `OCP.BOPAlgo` directly, or `Workplane("XZ")` for
Y-axis extrusions).

**1. `BOPAlgo_Builder().Shape()` returns a bare `cq.Shape`, not `Solid`/`Compound` — breaks `.cut()`/`.union()`.**
When fusing many shapes with `OCP.BOPAlgo.BOPAlgo_Builder` (needed for fast fuse of 20-60
gear-tooth solids — plain chained `.union()` is too slow), wrapping the result as
`cq.Shape(builder.Shape())` produces an object whose `.cut()` call fails with
`ValueError: Cannot find a solid on the stack or in the parent chain` — `findSolid()`
requires an actual `Solid`/`Compound` subclass. Fix: `cq.Shape.cast(builder.Shape())`
instead — it inspects the underlying `TopAbs_ShapeEnum` and returns the correct wrapper.
Why: `BOPAlgo_Builder` always emits `TopAbs_COMPOUND`, and `cq.Shape()`'s bare constructor
doesn't do type dispatch the way `.cast()` does.

**2. `Workplane.newObject([shape])` also breaks `.cut()` — don't wrap a `Solid`/`Compound` in `newObject` before booleans.**
Even a properly-typed `cq.Solid` object failed the same `findSolid` error when re-wrapped
via `base_wp.newObject(base_wp.vals())` before calling `.cut()` on it. Fix: just use the
original `Workplane` reference directly (`.cut()` doesn't mutate it — safe to reuse the
same blank across multiple independent cuts, e.g. one gear blank used to cut both the pan
and tilt gear's different bore patterns).

**3. `Workplane("XZ").circle(r).extrude(L)` extrudes in **-Y**, not +Y — opposite of the more commonly reached-for `"YZ"` plane (which extrudes +X as expected).**
Verified empirically: `Workplane("XZ").circle(2).extrude(5)` gives Y bbox `(-5, 0)`, while
`Workplane("YZ").circle(2).extrude(5)` gives X bbox `(0, 5)`. Building anything meant to
span `y0..y0+length` via the XZ plane and translating by `y0` silently produces geometry
shifted a full `length` in the wrong direction — this caused three *real* false-negative
interference results (looked like solid overlap between a pivot-pin spacer and its holder,
and between a tilt-motor's double-D shaft stub and its mounting bracket) that were only
caught by running `.intersect()` boolean checks and inspecting the clash bounding box, not
by `.isValid()` (both solids were individually valid — just mispositioned). Fix: either
extrude negative (`.extrude(-length)` then translate to the *start* coordinate) or write a
small `y_cyl(r, length, x, y0, z)` helper that does this internally — cheaper than
re-deriving the sign every time a part needs a horizontal-axis-Y feature.

**General takeaway:** for this class of task (many interlocking cylindrical bores/shafts
built by rotating/translating a canonical local-Z-axis shape onto different world axes),
always run the boolean `.intersect()` interference sweep across *every* adjacent part pair
before trusting the geometry — `.isValid()` alone does not catch sign/direction errors
like #3, which produce a perfectly valid but wrongly-located solid.
