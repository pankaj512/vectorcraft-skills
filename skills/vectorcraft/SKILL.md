---
name: vectorcraft
description: Create and edit professional vector illustrations in VectorCraft through its MCP server. Use for logos, icons, posters, badges, infographics, charts, UI graphics, spot illustrations, and any SVG/vector asset. Covers the design-to-draw-to-inspect-to-refine-to-export loop, MCP connection setup (remote and headless), tool routing, and core concepts. Triggers on vector art, illustration, logo, icon set, poster, SVG, "draw in VectorCraft", or "use the vectorcraft MCP". Pair with the companion skills vectorcraft-mcp, vectorcraft-design, vectorcraft-recipes, and vectorcraft-verify.
license: MIT OR Apache-2.0
metadata:
  author: ArtCraft community
  version: "1.0.0"
  tested-with: VectorCraft v0.24.x / vectorcraft-cli
---

# VectorCraft

VectorCraft is a free, open-source, Illustrator-like vector illustration app (Pen, Direct Selection, Pathfinder, Smart Guides, Appearance/Swatches/Layers). It runs on Windows, macOS, Linux, FreeBSD and web. Its MCP server — shipped inside `vectorcraft-cli` — lets a coding agent drive it programmatically. This skill teaches any agent how to use that server to produce polished vector artwork, not just valid shapes.

Use this skill as the **entry point / router**. It gives the mental model and the core workflow, then points to the focused companion skills for details.

## When to use

- Generate a logo, icon set, badge, poster, banner, infographic, chart, or spot illustration.
- Produce or edit SVG / PDF / EPS / PNG / vectorcraft deliverables for an app, website, or doc.
- Recolor, restyle, trace, or export existing artwork in VectorCraft.
- Any request that mentions VectorCraft, the vectorcraft MCP, or vector illustration as a deliverable.

## 1. Connect to the MCP server

The MCP server is `vectorcraft-cli mcp`. Two connection modes:

**Headless (recommended for agents — no GUI needed):**
```bash
vectorcraft-cli mcp --headless
```
Everything works except UI-only tools (`inspect_ui`, `ui_screenshot`, `type_text`, `answer_dialog`, `open_panel`, `screenshot` with `window:true`). That is expected and fine.

**Remote (live app, for watching edits in the UI):**
```bash
vectorcraft --control 7979          # in the app
vectorcraft-cli mcp --connect 127.0.0.1:7979
```
Plain `vectorcraft-cli mcp` tries `127.0.0.1:7979` first, then falls back to headless.

Register with your agent, e.g. for Claude Code:
```bash
claude mcp add vectorcraft -- vectorcraft-cli mcp
```
or JSON config:
```json
{"mcpServers":{"vectorcraft":{"command":"vectorcraft-cli","args":["mcp"]}}}
```

Tool names differ by client: `mcp__vectorcraft__<tool>` (Claude Code), `mcp_vectorcraft_<tool>` (Codex/Copilot-ish), or plain `<tool>` (some hosts). The tool **names and parameters are identical** — only the prefix differs. Use whatever your client exposes.

Confinement: `--automation-read-root` / `--automation-write-root` restrict file paths (repeatable, imply headless). Use **absolute paths** for all files.

## 2. Mental model (read this before drawing)

- **Artboard** = page. Points are in **1/72 in** (same as PostScript points); **y increases downward**; origin is the top-left of the first artboard. A default US Letter doc is 612×792.
- **Path** = anchors + segments. **Fill** is the interior, **stroke** is the outline (X toggles, Shift+X swaps, D resets to white fill / 1 pt black stroke in the GUI).
- **Objects → groups → layers** build a hierarchy. New objects become the current selection and are **drawn back to front** (later draws sit on top).
- **Live effects** live in the Appearance stack; `Object → Expand Appearance` bakes them in. `apply_effect` is non-destructive; `pathfinder` is destructive.
- Every edit is **undoable**. Returned `id`s are stable references — keep them. `command_batch` with `atomic:true` collapses many steps into one undo step.
- Keep the **selected tool** and **selection** in mind: paint/transform calls act on the selection or on explicit `ids`.

## 3. The core loop (follow this every time)

1. **Understand the brief.** Decide canvas size, output formats, subject, palette (3–5 colors), and visual hierarchy. For aesthetics, load `vectorcraft-design`.
2. **Inspect existing work** if editing: `doc_inspect {depth:0}` for a skeleton, then `doc_inspect {ids:[...]}` / `document.find` to target objects by name/type. Never edit blind.
3. **Set up the document.** `command_run` with `file.new` (or `open_file`). Add an artboard if needed.
4. **Draw back to front** — background, then main shapes, then details, then type. Batch independent shapes with `command_batch`; reference earlier steps via `$0.id`, `$last.path`. For filled shapes pass explicit `fill` and `stroke:"none"` so you don't inherit a black outline.
5. **Render and LOOK.** Call `render_preview {}` after every major step and **actually view the returned image** before continuing. This is non-negotiable for quality — text overflow, bad gradients, and stacking errors are invisible in JSON but obvious in the preview.
6. **Refine.** Fix composition, contrast, alignment, spacing, and typography (see `vectorcraft-design`). Use `transform`, `set_paint`, `apply_effect`, `pathfinder`.
7. **Save the source.** `save_file {path:"/abs/path/art.vectorcraft"}`.
8. **Export deliverables.** `export {path, format, scale}` for the formats you need (svg, png, pdf, eps, …). **Read the reply's `warnings`** (missing fonts, overflow, etc.).

For exact call signatures and parameters, load `vectorcraft-mcp`. For step-by-step build patterns, load `vectorcraft-recipes`. For verification and error recovery, load `vectorcraft-verify`.

## 4. Routing

| Need | Load |
|------|------|
| Exact MCP tool/parameter reference, paint & gradient syntax, command ids | `vectorcraft-mcp` |
| Make it look professional — composition, color, type, depth, critique | `vectorcraft-design` |
| Proven build recipes (poster, logo, icons, chart, badge, trace, multi-size export) | `vectorcraft-recipes` |
| Verify output, debug errors, headless/remote issues | `vectorcraft-verify` |

## 5. Minimal worked example (badge)

```jsonc
// 1. new document
{"tool":"command_run","params":{"id":"file.new","params":{"width":800,"height":800}}}

// 2. background + main shapes in one atomic batch
{"tool":"command_batch","params":{
  "atomic":true,
  "steps":[
    {"id":"draw_shape","params":{"shape":"rectangle","geometry":{"x":0,"y":0,"width":800,"height":800},"fill":"#0f172a","stroke":"none","name":"bg"}},
    {"id":"draw_shape","params":{"shape":"ellipse","geometry":{"cx":400,"cy":400,"r":300},"fill":{"gradient":{"kind":"linear","angle":-90,"stops":[{"offset":0,"color":"#f59e0b"},{"offset":1,"color":"#ef4444"}]}},"stroke":"none","name":"disc"}}
  ]
}}

// 3. look at it
{"tool":"render_preview","params":{"max_side":1024}}

// 4. save + export
{"tool":"save_file","params":{"path":"/abs/out/badge.vectorcraft"}}
{"tool":"export","params":{"path":"/abs/out/badge.svg","format":"svg"}}
{"tool":"export","params":{"path":"/abs/out/badge@2x.png","format":"png","scale":2}}
```

## 6. Golden rules

- **Look at the render.** JSON success ≠ visual correctness.
- **Explicit paint on draws.** Filled shape without outline → `fill` + `stroke:"none"`.
- **Use ids**, not fragile selection state, for edits.
- **Don't invent command ids.** Use `command_list {query:"..."}` to discover the real one.
- **Effect ids are not command ids.** Apply effects with `apply_effect {effect:"stylize.dropShadow"}`.
- **Read `warnings` on every export.**
- **Absolute paths only.**

## References

- `vectorcraft-mcp/SKILL.md` — exact tool & parameter reference
- `vectorcraft-design/SKILL.md` — design principles
- `vectorcraft-recipes/SKILL.md` — step-by-step recipes
- `vectorcraft-verify/SKILL.md` — verification & troubleshooting
- Official MCP docs: `vectorcraft://commands/reference` resource; wiki `https://craft.wiki/apps/vectorcraft/ai-agents/`
