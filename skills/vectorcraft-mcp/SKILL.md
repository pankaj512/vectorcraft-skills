---
name: vectorcraft-mcp
description: Exact reference for the VectorCraft MCP server tools — parameters, paint and gradient syntax, effects, command ids, batching, coordinates, export formats, and verification. Use when you need the precise JSON shape for a VectorCraft MCP call, when a tool call failed, or when discovering/choosing engine commands. Companion to the vectorcraft skill; load it before writing MCP calls you are unsure about.
license: MIT OR Apache-2.0
metadata:
  version: "1.0.0"
  tested-with: VectorCraft v0.24.x (955 engine commands, 118 effects)
---

# VectorCraft MCP reference

Precise signatures for the VectorCraft MCP server (`vectorcraft-cli mcp`). Tool prefixes vary by client (`mcp__vectorcraft__`, `mcp_vectorcraft_`, or none); the tool names/params below do not. Based on the official MCP doc and public skill audits for v0.24.x.

## Tool name aliases

Many tools have two accepted names. Use either:

| Purpose | Names |
|---|---|
| List commands | `command_list` / `list_commands` |
| Run one command | `command_run` / `run_command` |
| Run many steps | `command_batch` |
| Inspect document | `doc_inspect` / `inspect_document` |
| Apply an effect | `apply_effect` / `effect.apply` |
| Set fill | `set_paint` / `paint.setFill` (see below — params differ!) |
| Preview | `render_preview` |
| Screenshot | `screenshot`, `ui_screenshot` |
| Inspect UI | `ui_inspect` / `inspect_ui` |

> Warning: `set_paint` and `paint.setFill` are similar but not identical in parameter shape. Prefer the **named paint commands via `command_run`** (`paint.setFill`, `paint.setStroke`, `object.setProps`, `transparency.set`) when you need their full option sets; use the convenience `set_paint` for the common `{fill?, stroke?, strokeWidth?, ids?}` case.

## Discovery & navigation

```jsonc
{"tool":"command_list","params":{"query":"gradient"}}          // find a command by keyword
{"tool":"command_list","params":{"menu":"Object","brief":true}} // browse a menu, short output
{"tool":"command_list","params":{"filter":"text","enabledOnly":true}}
{"tool":"command_run","params":{"id":"<command.id>","params":{ /* exact params */ }}}
```
- Unknown parameter keys are **rejected** with a "did you mean" hint. Read the hint.
- Discover first; never guess a command id. `command_list` + the `vectorcraft://commands/reference` resource are authoritative.

```jsonc
{"tool":"doc_inspect","params":{}}                          // whole doc (small docs)
{"tool":"doc_inspect","params":{"depth":0}}                 // skeleton of a large doc
{"tool":"doc_inspect","params":{"ids":["id1"],"childLimit":20,"depth":2}}
{"tool":"command_run","params":{"id":"document.find","params":{"name":"Hero"}}}
{"tool":"command_run","params":{"id":"document.node","params":{"id":"id1","summary":true}}}
```
- Opacity appears as `opacity` (0–1) and `opacityPct` (0–100). Tools take the **0–100** form.

## Drawing

### draw_shape
```jsonc
{"tool":"draw_shape","params":{
  "shape":"rectangle",              // rectangle | ellipse | polygon | star | line
  "geometry":{"x":0,"y":0,"width":200,"height":100,"radius":16}, // rect (+corner radius)
  "fill":"#3b82f6","stroke":"none","strokeWidth":2,
  "dash":[],"cap":"round","join":"round","opacity":100,
  "name":"Card","effects":[]
}}
```
Geometry by shape:
- `rectangle` / `ellipse`: `{x,y,width,height}`; rect adds `radius`; ellipse accepts `anchors` (2–1000).
- `polygon`: `{cx,cy,radius,sides}`.
- `star`: `{cx,cy,radius1,radius2,points}`.
- `line`: `{x1,y1,x2,y2}`.
- `rotation` is degrees **clockwise** (0 = point/corner up).
- Omitted paint **inherits current new-art appearance** (often a fill + thin black stroke). For a filled shape with no outline, pass `fill` and `stroke:"none"`. Lines start with no fill and a visible stroke.
- Fill/stroke/opacity/effects are applied in the **same atomic operation** as the draw. An invalid paint or effect can cause the shape to draw as nothing.

### draw_path
```jsonc
{"tool":"draw_path","params":{
  "points":[[0,0],[100,50],[200,0]],  // or [{x,y,in:[..],out:[..],smooth:true},..]
  "closed":false,
  "fill":"none","stroke":"#111827","strokeWidth":4,
  "cap":"round","join":"round","name":"Curve"
}}
// or SVG path data:
{"tool":"draw_path","params":{"d":"M0 0 C 50 -40, 150 -40, 200 0","stroke":"#111827","fill":"none"}}
```
- `points` = `[[x,y],…]` or `[{x,y,in?,out?,smooth?},…]`; `d` = SVG path data (mutually exclusive with `points`).
- `dash`, `opacity`, `name`, `effects` also accepted.

### add_text
```jsonc
{"tool":"add_text","params":{
  "text":"Hello","x":100,"y":200,"size":48,"font":"Inter",
  "color":"#111827","justify":"left"
}}
// area type (wraps): width/height + fit
{"tool":"add_text","params":{"text":"Long copy…","x":100,"y":100,"width":300,"height":200,"fit":"shrinkText","fitMinPercent":60}}
// type on a path
{"tool":"add_text","params":{"text":"CURVED","path":"$last.path","mode":"onPath","pathEffect":"rainbow"}}
```
- Point type: `x` = **left edge** (with `justify:center|right`, `x` centers/ends the text on that coordinate); `y` = **baseline**.
- `fit`: `none` | `autoHeight` | `shrinkText`. Overflow returns `"overflow":true` — but area type can also **silently clip** text; always check the preview.
- `mode`: `area` | `onPath`. `pathEffect`: `rainbow` | `skew` | `3dRibbon` | `stairStep` | `gravity`.
- `justify`: `left` | `center` | `right` | `auto` | `justifyAll`. `vertical:true` for vertical text.
- Missing fonts fall back silently (often to Source Sans 3); confirm via export `warnings`.

## Paint, gradients, color

Accepted color forms:
- `"#rrggbb"` hex, `"none"`, `[r,g,b]` (0–1 floats), `{c,m,y,k}`, `{gray:0.5}`, `{swatch:"SwatchName"}`.
- Alpha hex (`#rrggbbaa`) is **rejected for fill/stroke** — use object opacity or `transparency.set`. Effect colors (e.g. Outer Glow) DO accept `#rrggbbaa`.

Gradient:
```jsonc
{"fill":{"gradient":{
  "kind":"linear",          // linear | radial | conic
  "angle":-90,              // -90 = stop 0 at top; 90 = bottom-to-top; 0 = left-to-right
  "stops":[{"offset":0,"color":"#f59e0b"},{"offset":1,"color":"#ef4444"}]
}}}
```
- One gradient spanning several objects: `paint.setFill {ids:[...], gradient:{...}, span:"selection"}` (via `command_run id:"paint.setFill"`).

Common paint/appearance commands (via `command_run`):
- `paint.setFill` — `{ids?, color:"#rrggbb"}`, `{ids?, gradient:{…}, span?}`, or `{ids?, none:true}`.
- `paint.setStroke` — `{ids?, color:"#rrggbb"}` (note: `paint.setStroke {color:"none"}` **fails** — use `object.setProps` / set stroke width to 0, or draw with `stroke:"none"`).
- `object.setProps` — `{ids?, opacity:0..100, blend:"Multiply", ...}`.
- `transparency.set` — `{ids?, opacity:0..100, blend:"Multiply"}`.
- `set_paint` convenience tool — `{fill?, stroke?, strokeWidth?, ids?}` → `{applied, objects}`; a returned note explains when only new-art paint changed.

## Effects

```jsonc
{"tool":"apply_effect"}                                  // no effect => catalogue with defaults (118 effects)
{"tool":"apply_effect","params":{"effect":"stylize.dropShadow","params":{"dx":0,"dy":6,"blur":12,"color":"#00000055"},"ids":["id1"]}}
```
- Common effect ids: `stylize.outerGlow`, `stylize.dropShadow`, `blur.gaussian`, `threeD.extrude`, `addon.color.halftone`, `warp.arc`. Discover all via `apply_effect {}`.
- **Effect ids are not command ids.** Apply with `apply_effect`, never `command_run {id:"stylize.dropShadow"}`.
- Effects are non-destructive (Appearance stack). To bake in: `command_run {id:"object.expandAppearance"}`.

## Shape operations & transforms

```jsonc
// pathfinder — destructive, acts back-to-front
{"tool":"pathfinder","params":{"operation":"minusFront","ids":["back","front"]}}
// operations: unite | minusFront | intersect | exclude | divide | trim | merge | crop | outline | minusBack

// transform — order: move -> rotate -> scale -> reflect -> shear
{"tool":"transform","params":{"ids":["id1"],"dx":10,"rotate":30,"scale":1.2,"origin":"center"}}
// rotate degrees are COUNTER-CLOCKWISE; shear positive moves the top to the right; copy:true duplicates first.
```
- For a **live (editable) boolean**, group the objects and apply a `pathfinder.*` effect via `apply_effect` instead of the destructive `pathfinder` tool.
- Repeat patterns: `command_run {id:"transform.copy", params:{...}}` + `object.transformAgain`, or the radial/grid/mirror Repeat commands, or `addon.arrange.grid`.
- Group/arrange/layer: `object.group`, `object.ungroup`, `object.arrange`, `layer.new`, `object.lock`.

## Data, type & layout helpers

```jsonc
{"tool":"create_graph","params":{"type":"column","x":80,"y":80,"width":600,"height":400,"csv":"Category,Value\nQ1,120\nQ2,180"}}
// 9 types: column, stacked column, bar, stacked bar, line, area, scatter, pie, radar
// bars default black/grey — recolor via document.find on series groups + paint.setFill; legend sits right of plot.

{"tool":"text_wrap","params":{"ids":["id1"],"offset":6,"invert":false,"release":false}}
{"tool":"undo","params":{}}   {"tool":"redo","params":{}}
```

## Batching (important for speed & quality)

```jsonc
{"tool":"command_batch","params":{
  "label":"build badge","atomic":true,"stop_on_error":true,
  "steps":[
    {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":400,"cy":400,"r":300},"fill":"#ef4444","stroke":"none"}},
    {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":400,"cy":400,"r":180},"fill":"#ffffff","stroke":"none"}},
    {"id":"pathfinder","params":{"operation":"minusFront","ids":["$0.id","$1.id"]}}
  ]
}}
```
- Max **256 steps**. A step `id` may be a raw tool name (`draw_shape`/`draw_path`) with that tool's params, or any engine command id.
- Reference earlier results with `$N.path` / `$last.path` and ids with `$N.id` / `$last.id`.
- `atomic:true` = one undo step (great for "one logical change"). A failed batch keeps earlier successful steps unless you undo.
- Read the reply: `completed`, `failed`, `results`, `created`.

## Files, documents & export

```jsonc
{"tool":"open_file","params":{"path":"/abs/in.svg","importer":"svg"}}  // importer optional
{"tool":"save_file","params":{"path":"/abs/art.vectorcraft","compress":true,"version":4}}
{"tool":"export","params":{"path":"/abs/art.svg","format":"svg"}}
{"tool":"export","params":{"path":"/abs/art@2x.png","format":"png","scale":2}}
{"tool":"export","params":{"path":"/abs/art.pdf","format":"pdf","options":{"advanced":{"outlineText":true}}}}
{"tool":"export","params":{"path":"/abs/art.jpg","format":"jpg","options":{"quality":85}}}
```
- Formats: `svg`, `svgz`, `pdf`, `ai`, `eps`, `dxf`, `emf`, `wmf`, `hpgl`, `png`, `jpg`, `webp`, `gif`, `png8`, `tiff`, `bmp`, `tga`, `psd`, `txt`, `vectorcraft`, `template`.
- `export` options: `{quality}` (JPEG/WebP), `{ppi, background}` (raster), `{advanced:{outlineText:true}}` (PDF).
- No `path` → bytes returned as `dataBase64`. **Always read `warnings`** (missing fonts, clipped text, etc.).

## Coordinates, picking & UI tools

- Units: points (1/72 in). y down. Origin = top-left of first artboard. New doc 612×792 (US Letter).
- New objects become the selection; keep returned `id`s. All edits undoable.
- GUI-like control:
  - `select_tool {tool:"pen"}` — tool keys: `v` select, `a` direct, `p` pen, `m` rect, `l` ellipse, `t` type, etc.
  - `pointer_gesture {events:[{kind:"down",x,y},{kind:"drag",x,y},{kind:"up",x,y}], tool?}` — kinds: down, drag, up, move, doubleclick, tripleclick; `mods` for modifiers; `backquote`.
  - `press_key {key, mods?}`; `invoke_menu {command, params?}`.
  - **Remote only:** `type_text {text}`, `answer_dialog {set:[{field,value?}], action:"confirm"|"cancel"}`, `open_panel {panel}`, `ui_screenshot`, `ui_inspect`, `screenshot {window:true}`. These fail in headless mode (expected — use `--connect`).

## Resources

- `vectorcraft://document`, `vectorcraft://document/json`
- `vectorcraft://commands`, `vectorcraft://commands/reference`
- Templates: `vectorcraft://object/{id}`, `/command/{id}`, `/effect/{id}`, `/swatch/{name}`
- Prompts (8): poster, icon-set, recolor, trace-and-style, export-set, check-spelling, variables-bind, path-tools.

## Operational notes

- Engine has **955 commands** at v0.24.1; `addon.*` families add parametric shapes, arrange/grid, cleanup checks, halftone, path tools, etc.
- A ~2 s busy guard protects against overlapping calls; long operations emit progress tokens and can be cancelled via `notifications/cancelled`.
- Read/write roots (`--automation-*-root`) confine paths; use absolute paths. Errors look like `automation path rejected: outside the write root ...`.

## Pitfalls (quick list)

- Wrong app on port 7979 → "unknown command": use `--headless` or the right `--connect`.
- Misspelt param keys are rejected — read the "did you mean" hint.
- `paint.setStroke {color:"none"}` fails — draw with `stroke:"none"` instead.
- `add_text` `x` is the left edge; `y` is the baseline.
- Area type can clip text silently — verify with `render_preview`.
- Gradient angle direction is easy to flip (`-90` top, `90` bottom→top).
- Missing fonts fall back silently — check export warnings.
- `select.all` grabs unlocked backgrounds — lock them or target by `id`.
- Pathfinder is destructive and back-to-front.
- Never invent command ids or treat effect ids as command ids.
- UI tools fail in headless mode — by design.
