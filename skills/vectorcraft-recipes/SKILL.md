---
name: vectorcraft-recipes
description: Proven, step-by-step build recipes for common VectorCraft artwork — posters, logos and marks, icon sets, badges, infographics/styled charts, trace-and-style, recoloring, and multi-size/multi-format export. Use when you want a reliable sequence of MCP calls for a specific deliverable. Companion to vectorcraft and vectorcraft-design.
license: MIT OR Apache-2.0
metadata:
  version: "1.0.0"
---

# VectorCraft recipes

Concrete sequences of MCP calls for common deliverables. Adapt coordinates/colors to your brief using `vectorcraft-design`. All paths are absolute; `command_batch` examples use `atomic:true` for one undo step. Render (`render_preview`) and look after each major step.

## Recipe 1 — Poster (hero graphic + title + subtitle)

Brief inputs: canvas (e.g. 1200×1600), palette, title, subtitle, motif.

```jsonc
// 1. document
{"tool":"command_run","params":{"id":"file.new","params":{"width":1200,"height":1600}}}

// 2. background + motif, back to front
{"tool":"command_batch","params":{"label":"poster base","atomic":true,"steps":[
  {"id":"draw_shape","params":{"shape":"rectangle","geometry":{"x":0,"y":0,"width":1200,"height":1600},"fill":"#0f172a","stroke":"none","name":"bg"}},
  {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":600,"cy":620,"r":380},"fill":{"gradient":{"kind":"radial","stops":[{"offset":0,"color":"#f59e0b"},{"offset":1,"color":"#b45309"}]}},"stroke":"none","name":"sun"}},
  {"id":"draw_shape","params":{"shape":"polygon","geometry":{"cx":600,"cy":1050,"r":520,"sides":3,"rotation":180},"fill":"#1e293b","stroke":"none","name":"mountain"}}
]}}

// 3. type hierarchy
{"tool":"add_text","params":{"text":"SUMMIT","x":600,"y":220,"size":150,"font":"Inter","color":"#f8fafc","justify":"center"}}
{"tool":"add_text","params":{"text":"Reach higher.","x":600,"y":300,"size":42,"font":"Inter","color":"#f59e0b","justify":"center"}}

// 4. inspect look, refine, save, export
{"tool":"render_preview","params":{"max_side":1024}}
{"tool":"save_file","params":{"path":"/abs/poster.vectorcraft"}}
{"tool":"export","params":{"path":"/abs/poster.png","format":"png","scale":2}}
{"tool":"export","params":{"path":"/abs/poster.pdf","format":"pdf"}}
```
Design notes: title on the upper third, focal motif center, accent color only on the subtitle. Gradients radial for a light source behind the subject.

## Recipe 2 — Logo mark (geometric boolean)

Goal: a clean, memorable mark built from primitives + booleans.

```jsonc
{"tool":"command_run","params":{"id":"file.new","params":{"width":512,"height":512}}}

{"tool":"command_batch","params":{"label":"mark","atomic":true,"steps":[
  {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":205,"cy":256,"r":140},"fill":"#0ea5e9","stroke":"none"}},
  {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":307,"cy":256,"r":140},"fill":"#0ea5e9","stroke":"none"}}
]}}
// subtract the front circle from the back one to make a crescent
{"tool":"pathfinder","params":{"operation":"minusFront","ids":["$0.id","$1.id"]}}
// or keep live: group then apply a pathfinder.* effect via apply_effect
```
Variants:
- **Ring / donut**: two concentric ellipses, `minusFront`.
- **Cut-out letter**: rectangle + path/`add_text` converted to outlines, `minusFront`.
- **Overlap blend**: `unite` two shapes for a single silhouette.
- **Spark/star**: `draw_shape star` with `radius1/radius2/points`, paired with a small offset ellipse, `unite`.

Finish: render, silhouette test at 64px, recolor to brand, then `export svg` + `export png` at 1x/2x.

## Recipe 3 — Icon set (consistent grid)

Goal: N icons with one stroke weight, one grid, one style.

```jsonc
{"tool":"command_run","params":{"id":"file.new","params":{"width":1024,"height":512}}}
// draw icons on a 24x24 conceptual grid, scaled up; use addon.arrange.grid for layout
{"tool":"command_run","params":{"id":"addon.arrange.grid","params":{
  "count":6,"columns":3,"spacing":32,"originX":80,"originY":80
}}}

// per icon: explicit fill + stroke:"none" OR consistent stroke and fill:"none"
{"tool":"draw_shape","params":{"shape":"rectangle","geometry":{"x":80,"y":80,"width":160,"height":160,"radius":32},"fill":"none","stroke":"#0f172a","strokeWidth":12,"cap":"round","join":"round"}}
```
Rules: same grid, same stroke weight (scaled), same corner radii, same optical padding. Build repeated elements with `transform {copy:true}` + `object.transformAgain`, or a radial/grid Repeat command. Recolor the whole set at once with `document.find` + `paint.setFill`. Export each icon via `export {selection:true, format:"svg"}` or one sheet + slice.

## Recipe 4 — Badge / emblem (from the official tutorial)

```jsonc
// gradient rounded square + circle + star + wave + type, grouped
{"tool":"command_batch","params":{"label":"badge","atomic":true,"steps":[
  {"id":"draw_shape","params":{"shape":"rectangle","geometry":{"x":120,"y":120,"width":520,"height":520,"radius":96},"fill":{"gradient":{"kind":"linear","angle":-90,"stops":[{"offset":0,"color":"#f59e0b"},{"offset":1,"color":"#ef4444"}]}},"stroke":"none"}},
  {"id":"draw_shape","params":{"shape":"star","geometry":{"cx":380,"cy":380,"radius1":150,"radius2":70,"points":5},"fill":"#ffffff","stroke":"none","opacity":90}},
  {"id":"draw_path","params":{"d":"M180 520 Q 380 600 580 520","fill":"none","stroke":"#ffffff","strokeWidth":10,"cap":"round"}}
]}}
{"tool":"add_text","params":{"text":"CREW","x":380,"y":360,"size":72,"font":"Inter","color":"#0f172a","justify":"center"}}
{"tool":"command_run","params":{"id":"object.group","params":{"ids":["$0.id","$1.id","$2.id"]}}}
```
Note: place a watermark/logo at low opacity (e.g. `object.setProps {opacity:12}`), lock the background so `select.all` won't grab it.

## Recipe 5 — Infographic / styled chart

```jsonc
{"tool":"create_graph","params":{"type":"column","x":120,"y":140,"width":800,"height":520,
  "csv":"Quarter,Revenue\nQ1,120\nQ2,180\nQ3,150\nQ4,240"}}
// graph draws grey bars; find series groups and recolor
{"tool":"command_run","params":{"id":"document.find","params":{"type":"group","name":"series"}}}
{"tool":"command_run","params":{"id":"paint.setFill","params":{"ids":["$last.id"],"color":"#0ea5e9"}}}
// emphasize one bar
{"tool":"command_run","params":{"id":"paint.setFill","params":{"ids":["<bar4-id>"],"color":"#f59e0b"}}}
```
Rules: sort bars by value, emphasize one with accent, faint gridlines, label directly. Legend sits to the right of the plot. Recolor via `document.find` (not guessed ids).

## Recipe 6 — Trace and style a raster

```jsonc
{"tool":"open_file","params":{"path":"/abs/photo.png"}}
{"tool":"command_run","params":{"id":"imageTrace.make","params":{"mode":"blackWhite","threshold":180}}}
{"tool":"command_run","params":{"id":"imageTrace.expand"}}
// restyle the traced paths
{"tool":"command_run","params":{"id":"paint.setFill","params":{"color":"#111827"}}}
```
Then simplify paths, recolor, add a shadow via `apply_effect`, export vector. Image Trace converts bitmaps to editable vector paths — quality depends on source resolution/contrast.

## Recipe 7 — Recolor an existing illustration

```jsonc
{"tool":"doc_inspect","params":{"depth":0}}
{"tool":"command_run","params":{"id":"document.find","params":{"fill":"#oldcolor"}}}
{"tool":"command_run","params":{"id":"paint.setFill","params":{"ids":["$last.id"],"color":"#newcolor"}}}
// or map a whole palette via variables / Swatches if the doc uses them
```
Preserve contrast ratios after recoloring (`vectorcraft-design` §3). For a global tint across several objects, one gradient with `span:"selection"` keeps cohesion.

## Recipe 8 — Multi-size / multi-format export set

```jsonc
{"tool":"save_file","params":{"path":"/abs/art.vectorcraft","compress":true}}
{"tool":"export","params":{"path":"/abs/art.svg","format":"svg"}}
{"tool":"export","params":{"path":"/abs/art.pdf","format":"pdf","options":{"advanced":{"outlineText":true}}}}
{"tool":"export","params":{"path":"/abs/art@1x.png","format":"png","scale":1}}
{"tool":"export","params":{"path":"/abs/art@2x.png","format":"png","scale":2}}
{"tool":"export","params":{"path":"/abs/art@3x.png","format":"png","scale":3}}
```
Read every reply's `warnings`. For web, also export `webp` (quality ~85). For print, `eps`/`pdf` with outlined text. Use the `export-set` MCP prompt if your client exposes prompts: it takes formats, directory, scale.

## Recipe 9 — Many shapes efficiently ( parametric )

```jsonc
{"tool":"command_batch","params":{"label":"pattern","atomic":true,"steps":[
  {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":100,"cy":100,"r":40},"fill":"#38bdf8","stroke":"none"}},
  {"id":"transform.copy","params":{"ids":["$0.id"],"dx":120}},
  {"id":"object.transformAgain"},
  {"id":"object.transformAgain"}
]}}
// or parametric add-ons:
{"tool":"command_run","params":{"id":"addon.pathtools.shape.create","params":{ /* ... */ }}}
{"tool":"command_run","params":{"id":"addon.arrange.grid","params":{"count":12,"columns":4,"spacing":24}}}
{"tool":"command_run","params":{"id":"addon.cleanup.check","params":{}}}
```
Discover exact add-on params with `command_list {query:"addon."}` before use — v0.24.x ships `addon.pathtools.*`, `addon.arrange.*`, `addon.cleanup.*`, `addon.color.*`.

## Verification for every recipe

1. `render_preview {}` and **look** after each major step.
2. `doc_inspect {ids:[...]}` to confirm structure/ids.
3. Export, check `bytes > 0`, desired `format`, and empty/well-understood `warnings`.
4. Small-size silhouette test for logos/icons.

## References

- `vectorcraft/SKILL.md`, `vectorcraft-design/SKILL.md`, `vectorcraft-mcp/SKILL.md`, `vectorcraft-verify/SKILL.md`
