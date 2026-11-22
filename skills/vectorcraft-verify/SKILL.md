---
name: vectorcraft-verify
description: Verify VectorCraft MCP output and troubleshoot failures — render_preview visual checks, doc_inspect structure checks, export reply validation (bytes/warnings), vectorcraft-cli info, headless vs remote issues, and a catalog of common errors with fixes. Use after building artwork, when a tool call fails, or when exported art looks wrong. Companion to vectorcraft and vectorcraft-mcp.
license: MIT OR Apache-2.0
metadata:
  version: "1.0.0"
---

# VectorCraft verification & troubleshooting

"Success" in a tool reply does not mean the artwork is correct. This skill covers how to prove it and how to recover when things go wrong.

## 1. Verify visually (most important)

After every major build step:
```jsonc
{"tool":"render_preview","params":{"max_side":1024}}
```
- **Actually look at the returned image.** Check focal point, spacing, alignment, contrast, text fit, gradient direction, stacking order, stray shapes.
- For multi-artboard docs, pass `artboard` (0-based) to render each one.
- For icons/logos, render small (`max_side:64`–`128`) — the silhouette must still read.
- `screenshot {path, scale, artboard}` for a full-resolution raster to disk; `ui_screenshot` / `ui_inspect` only work in **remote** mode.

A prettier but slower alternative: zoom/pan with `pointer_gesture` and `press_key` in the live app (remote mode) to inspect details.

## 2. Verify structure

```jsonc
{"tool":"doc_inspect","params":{"depth":0}}                    // skeleton: layers, groups, object counts
{"tool":"doc_inspect","params":{"ids":["id1","id2"],"childLimit":20,"depth":3}}
{"tool":"command_run","params":{"id":"document.find","params":{"name":"Hero"}}}
{"tool":"command_run","params":{"id":"document.compare","params":{ /* ... */ }}} // newer versions
```
- Confirm expected objects exist, are in the right layer, and nothing extra was created.
- Opacity fields: `opacity` (0–1) vs `opacityPct` (0–100) — tools use `opacityPct`.
- Use `id`s from create replies for targeted edits; avoid relying on current selection.

## 3. Verify exports

For each export, check the reply:
- `path` / `format` match what you asked.
- `bytes > 0` (or `dataBase64` present if no path).
- **`warnings`** — read them every time. Common ones:
  - Missing font → text silently substituted (often Source Sans 3). Pick an installed font or outline text.
  - Area text overflow → increase the box, reduce size, or use `fit:"shrinkText"`.
  - Unsupported feature for chosen format → change format or flatten.
- Spot-check: open the exported SVG/PNG or re-open it via `open_file` to confirm it is valid.

## 4. CLI sanity checks

```bash
vectorcraft-cli info                 # version, connected app, capabilities
vectorcraft-cli commands             # engine command list
vectorcraft-cli commands --query "text wrap"   # discovery (if supported)
vectorcraft-cli mcp --headless       # start headless server
vectorcraft-cli mcp --connect 127.0.0.1:7979   # attach to live app
```
Confirm the version matches your skill expectations (v0.24.x). Some commands/params are version-gated; older builds may lack newer commands (`svgFilter.*`, `text.spelling.check`, `type.dynamicText`, `assets.add3D`, `document.compare`, `edit.newDocumentFromState`).

## 5. Common errors & fixes

| Symptom | Likely cause | Fix |
|---|---|---|
| `unknown command ...` on connect | Connected to wrong app on port 7979, or command id is invented | Use `vectorcraft-cli mcp --headless`, or discover the real id with `command_list {query}` |
| Tool rejected param `"did you mean ..."` | Misspelt/unknown parameter key | Follow the hint; check `vectorcraft-mcp` signatures |
| Shape draws nothing | Invalid paint or effect in the same draw call | Fix paint/effect (hex 6-digit, valid effect id), redraw |
| `paint.setStroke {color:"none"}` fails | Command doesn't accept `none` on stroke | Draw with `stroke:"none"`, or remove stroke via `object.setProps` |
| Shape has unexpected black outline | Omitted paint inherited new-art appearance | Pass explicit `fill` and `stroke:"none"` on draws |
| Text clipped / missing | Area type overflow, missing font, or wrong baseline (`y`) | Enlarge box / `fit:"shrinkText"`; check font; remember `y` is baseline, `x` is left edge |
| Gradient points wrong way | Angle misunderstanding | `-90`=top, `90`=bottom→top, `0`=left→right; use `span:"selection"` for multi-shape gradients |
| `select.all` grabbed background | Background not locked | Lock background layers, or target objects by `id` |
| Pathfinder result unexpected | Destructive, acts back-to-front | Order objects correctly; use live `pathfinder.*` effect on a group for editability |
| Effect id rejected by `command_run` | Effect ids ≠ command ids | Use `apply_effect {effect:"stylize.dropShadow", params:{...}}` |
| `"automation path rejected: outside the write root"` | Path outside confinement roots, or relative path | Use an absolute path under the configured `--automation-*-root` |
| UI tools fail (`inspect_ui`, `type_text`, `open_panel`, `answer_dialog`, `ui_screenshot`, `window:true`) | Running headless | Expected — use `--connect` to a live app, or avoid UI-only tools |
| Wrong app/hanging on 7979 | Port conflict | Set `VECTORCRAFT_CONTROL_PORT`, or use `--headless` |
| `overflow:true` from `add_text` | Text doesn't fit box | Increase width/height, reduce `size`, set `fit:"shrinkText"` and `fitMinPercent` |
| Exported art looks different | Font substitution, flat vs appearance, color profile | Outline text in PDF (`options.advanced.outlineText:true`), expand appearance, check CMYK/RGB |
| Calls return "busy" / cancelled | ~2 s busy guard or cancellation | Serialize calls; wait for progress tokens to finish; don't overlap writes |
| Batch partially applied | `stop_on_error` default true keeps earlier steps | Inspect `failed` in reply; fix step; undo if needed; `atomic:true` for single undo |

## 6. Recovery workflow

1. **Stop and inspect** — `render_preview` + `doc_inspect {depth:0}`.
2. **Identify** the smallest failing element (use ids / `document.find`).
3. **Undo** if a batch went wrong: `undo {}` — note `atomic:true` batches undo as one step.
4. **Fix minimally** with `set_paint` / `transform` / re-draw that one object; avoid regenerating everything.
5. **Re-render** and re-verify exports.
6. If an external file caused it (bad importer/format), try `open_file` with an explicit `importer`, or `vectorcraft-cli convert IN OUT` first.

## 7. Acceptance checklist (final)

- [ ] Visual preview matches the brief; focal point and hierarchy clear.
- [ ] `doc_inspect` shows expected structure, no stray objects.
- [ ] `.vectorcraft` source saved.
- [ ] All requested export formats produced, `bytes > 0`, warnings understood.
- [ ] Small-size test passed (logos/icons).
- [ ] Fonts present or text outlined for print.

## References

- `vectorcraft/SKILL.md`, `vectorcraft-mcp/SKILL.md`, `vectorcraft-design/SKILL.md`, `vectorcraft-recipes/SKILL.md`
- Official docs: `vectorcraft://commands/reference`, wiki `https://craft.wiki/apps/vectorcraft/ai-agents/`
