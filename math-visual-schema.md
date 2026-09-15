# `math-visual` — input schema

Reference for `pluginConfig["math-visual"]`. Companion to `math-visual-tutor-prompt.md`
(the how-to-use-it doc) — this is the exact shape, extracted from the validation and
rendering code in `math-visual-plugin.html`.

## Top level

```
{
  title?:    string,          // shown as a heading above the figure
  caption?:  string,          // shown under the title, smaller
  board?:    { ... },         // see below — defaults apply if omitted entirely
  params?:   [ { ... } ],     // named sliders, see below
  objects:   [ { ... } ],     // REQUIRED, non-empty — nothing else is required
  display?:  { ... }          // see below
}
```
Any other top-level key is rejected with an error.

## `board`

| Key | Type | Default | Notes |
|---|---|---|---|
| `type` | `"cartesian"` \| `"numberline"` \| `"polar"` | `"cartesian"` | `numberline` hides the y-axis and forces `grid:false`. `polar` is accepted but currently renders identically to `cartesian` — polar *curves* come from the `polar` object type, not this. |
| `x` | `[min, max]` | `[-5, 5]` | |
| `y` | `[min, max]` | *(auto-framed)* | Omit it and the plugin samples every function/point/scatter/histogram/bar on screen and picks a window that contains them, trimmed to the 2nd–98th percentile to survive a pole. |
| `aspect` | `"equal"` \| `"free"` | `"free"` | `"equal"` locks the unit circle to look like a circle — use for geometry. |
| `grid` | boolean | `true` | |
| `axes` | `false` \| `{xLabel?, yLabel?}` | `{xLabel:"x", yLabel:"y"}` | `false` removes the axis lines and ticks entirely — for a geometry/trig construction where the coordinate frame isn't part of the lesson. |
| `pan` | boolean | `false` | Enables pinch/drag pan and zoom. |

## `params[]` (sliders)

| Key | Type | Default |
|---|---|---|
| `name` | identifier, not `x`/`y`/`t`/`e`/`pi`/`PI` | — (required) |
| `label` | string | = `name` |
| `min`, `max` | number, `min < max` | — (required) |
| `step` | number | `(max-min)/100` |
| `value` | number | midpoint, clamped into range |
| `integer` | boolean | `false` |

Any `params` name becomes usable inside any expression or numeric field elsewhere in the
declaration (`"n"`, `"a*(x-h)^2+k"`, etc.) — that's what makes a slider live.

## `display`

| Key | Type | Default |
|---|---|---|
| `readouts` | boolean | `true` — the computed-value row below the figure |
| `legend` | boolean | `true` |
| `height` | `"s"` \| `"m"` \| `"l"` | `"m"` — 230 / 300 / 390 px |
| `hint` | string | auto ("Drag the slider…" / "Drag the points…") |

## `objects[]` — shared keys

Every object needs `type` (one of the 30 below). Everything else is optional unless noted:

| Key | Meaning |
|---|---|
| `id` | Lets other objects reference this one via `of`, `on`, `intersect`, or as a point in `points`/`foci`/`center`. |
| `label` | Surfaces differently per type — legend swatch, on-canvas name, or readout row — see below. |
| `color` | One of `primary·secondary·accent·violet·amber·teal·muted·error`. Omit it and colors auto-cycle through the first six in declaration order. |
| `style` | `"solid"` \| `"dashed"`. |
| `showValue` | Turns on the computed readout for types that have one. |
| `draggable` | Lets the student move a free point or glider. |

A **point reference** anywhere below (`points[]`, `foci`, `center`, `through`) is either a
literal `[x, y]` or the string `id` of a previously declared point. Any field marked *expr*
accepts a literal number **or** a JessieCode string that can reference `x`/`y`/`t` and any
declared param.

## Object types, by family

**Curves**

| Type | Fields |
|---|---|
| `function` | `expr` (in x), `domain?` [min,max] |
| `piecewise` | `pieces: [{expr, from, to}, ...]` |
| `parametric` | `x`, `y` (both in t), `tRange?` (default `[0, 2π]`) |
| `polar` | `r` (in t), `tRange?` |
| `implicit` | `expr` (in x,y) — curve where `expr(x,y) = 0` |
| `inequality` | `of` (a function/line id), `side?: "below"` (default above) |

**Points & lines**

| Type | Fields |
|---|---|
| `point` | `at: [x,y]`, **or** `on: id` (+ `at` = angle in radians if on a circle, x if on a function), **or** `intersect: [idA, idB]` (+ `which?`). `showValue` → `(x, y)` readout. An `intersect` point always gets its coordinate readout, regardless of `showValue`. |
| `segment` / `line` / `ray` / `vector` | `points: [ref, ref]`. `showValue` → `length` readout (segment) or `slope` readout (the other three). Only `line`/`ray` paint an on-canvas name — `segment`/`vector` don't, since it collides with vertex labels on a small figure. |
| `polygon` | `points: [ref, ref, ref, ...]` (≥3). `showValue` → `area` readout (computed, unsigned). |
| `angle` | `points: [armA, vertex, armB]`, `radius?` (default 0.55). Always reports the non-reflex (≤180°) measure. `showValue` (default true) controls the on-canvas degree label; `label` additionally adds a readout. |

**Round shapes**

| Type | Fields |
|---|---|
| `circle` | `center: ref`, **either** `radius` (*expr*, default 1) **or** `through: ref`. `showCenter?` (default true). `showValue` → `circumference` readout. |
| `arc` | `points: [center, start, end]`. |
| `ellipse` | `foci: [ref, ref]`, `sum?` (*expr*, default 4). |

**Calculus** — all reference a function via `of`

| Type | Fields |
|---|---|
| `tangent` | `of`, `at` (*expr*), `draggable?` (default true). Readout: `f′(x) = ...`, tracking the (possibly dragged) point. |
| `secant` | `of`, `from`, `to` (*expr*). Readout: average rate of change between two draggable gliders. |
| `derivative` | `of`, `domain?`. Draws `f′` as its own curve (numeric derivative). |
| `riemann` | `of`, `from`, `to`, `n` (*expr*), `method?: left\|right\|middle\|trapezoidal\|lower\|upper` (default middle). Readout: the computed sum. |
| `integral` | `of`, `from`, `to`. Shades the region; readout is the exact definite integral. |
| `asymptote` | Either `at` (*expr*) + `orientation?: "vertical"` (else horizontal), **or** `expr` for a curved asymptote. Always dashed/muted. |

**Data & statistics**

| Type | Fields |
|---|---|
| `scatter` | `data: [[x,y], ...]`. `showValue` → `n` readout. |
| `bestfit` | `of` (a scatter id), `degree?` (default 1). Readout: the fitted line/polynomial, plus r² always. |
| `histogram` | `data: [numbers]`, `binWidth?` (*expr*, default 1), `from?` (*expr*, default floor to bin). Bins are `[lo, hi)`. Readout: `n`. |
| `bar` | `categories: [strings]`, `values: [numbers]` (same length). |
| `boxplot` | `data: [numbers]` (≥4), `at?`, `width?` (default 0.8), `orientation?: "vertical"` (else horizontal). Readout: the computed five-number summary. |

**Number line**

| Type | Fields |
|---|---|
| `interval` | `from`, `to` (*expr*), `at?` (offset), `closed?: boolean \| [boolLeft, boolRight]` (hollow vs filled endpoints). |
| `tick` | `at` (*expr*), `label?`. |

**Annotation**

| Type | Fields |
|---|---|
| `label` | `at: [x,y]`, `text?`, `anchor?: left\|middle\|right`. Free text — the only object with nothing computed. |

## What's enforced, not just typed

- Unknown top-level/board/display keys, unknown object `type`s, and duplicate `id`s are all
  rejected.
- `of` is checked against the real kind it needs (a function for `tangent`/`riemann`/etc., a
  scatter for `bestfit`) before anything renders.
- A declaration that fails any of this renders a plain error card naming exactly what's
  wrong — never a picture drawn on best-effort guesses.
