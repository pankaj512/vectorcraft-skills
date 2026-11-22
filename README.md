# VectorCraft Skills for Coding Agents

A portable, [agentskills.io](https://agentskills.io/specification)-compliant skill collection that teaches any coding agent to produce **professional vector illustrations** with [VectorCraft](https://craft.wiki/apps/vectorcraft/overview/) through its MCP server.

These skills work with **OpenCode, Codex, Claude Code, GitHub Copilot, Gemini CLI**, and any other agent that reads `SKILL.md` files. They encode the official VectorCraft MCP tool signatures plus the design and workflow knowledge that is *not* in the documentation (learned from public community skills and design practice).

## What's inside

```
vectorcraft-skills/
├── skills/
│   ├── vectorcraft/          # Router / entry point: when to use, MCP setup, core loop
│   ├── vectorcraft-mcp/      # Exact tool signatures, paint/gradient/effect syntax, command ids
│   ├── vectorcraft-design/   # Composition, color, typography, depth, design critique  ← the "looks good" part
│   ├── vectorcraft-recipes/  # Tested build recipes: poster, logo, icons, badge, chart, trace, export
│   └── vectorcraft-verify/   # Visual/structure/export verification and error recovery
├── README.md                 # This documentation
├── LICENSE                   # MIT OR Apache-2.0
└── .gitignore
```

Each skill is a single `SKILL.md` with frontmatter (`name`, `description`, `license`, `metadata`) and a concise, step-by-step body under ~500 lines, following progressive disclosure. Start with `vectorcraft`; it routes to the others.

## Prerequisites

1. **VectorCraft** and **`vectorcraft-cli`** installed and on your `PATH` (the MCP server ships inside the CLI).
2. Verify: `vectorcraft-cli info`.
3. Decide on a connection mode:
   - **Headless** (agents, no GUI): `vectorcraft-cli mcp --headless`
   - **Remote** (watch edits live in the app):
     ```bash
     vectorcraft --control 7979
     vectorcraft-cli mcp --connect 127.0.0.1:7979
     ```
   - Plain `vectorcraft-cli mcp` tries `127.0.0.1:7979`, then falls back to headless.
4. Use **absolute paths** for all files. Optionally confine access with `--automation-read-root` / `--automation-write-root`.

## Install

First, clone this repository:

```bash
git clone <repo-url> vectorcraft-skills
cd vectorcraft-skills
```

Then choose one of the options below.

### Option A — `npx skills add` (agentskills.io tooling)

If you use the `skills` CLI:
```bash
npx skills add <owner>/<repo> --skill vectorcraft
```
This reads the standard `skills/<name>/SKILL.md` layout directly.

### Option B — manual copy

Copy the `skills/` subdirectories into your agent's skills folder, e.g.:
```bash
cp -R vectorcraft-skills/skills/vectorcraft      ~/.config/opencode/skills/
cp -R vectorcraft-skills/skills/vectorcraft-mcp  ~/.config/opencode/skills/
# ...repeat for the other four
```
GitHub Copilot (VS Code) reads `AGENTS.md` plus `.agents/skills/` in the workspace — copy the five `skills/*` dirs into `.agents/skills/` at your project root.

## Register the MCP server (once per agent)

Claude Code:
```bash
claude mcp add vectorcraft -- vectorcraft-cli mcp
```
Generic JSON (works for most agents — OpenCode, Codex, Copilot, Gemini CLI):
```json
{
  "mcpServers": {
    "vectorcraft": {
      "command": "vectorcraft-cli",
      "args": ["mcp"]
    }
  }
}
```
For headless-only environments use `"args": ["mcp", "--headless"]`.

## Quickstart (what the agent will do)

Once installed, just ask naturally, e.g.:

- *"Create a bold flat poster titled SUMMIT with a sun-and-mountain motif, export SVG and 2x PNG."*
- *"Design a 6-icon set for a weather app with a consistent 2px stroke."*
- *"Make a logo mark from overlapping circles using a boolean cutout."*
- *"Recolor this illustration to a teal-and-amber palette and export for web."*

The agent should:
1. Load the `vectorcraft` skill (auto-triggered by the description).
2. Plan brief → palette → hierarchy, then `file.new`.
3. Build back-to-front, batching shapes; paint explicitly (`stroke:"none"` when needed).
4. Call `render_preview` and **look** after each major step.
5. Refine using `vectorcraft-design` principles.
6. `save_file` to `.vectorcraft`, then `export` deliverables and read `warnings`.

## Tool naming note

Tool prefixes vary by client: `mcp__vectorcraft__render_preview` (Claude Code), `mcp_vectorcraft_render_preview` (some clients), or `render_preview` (others). The tool names and parameters are identical — use whatever your client exposes.

## Design & workflow philosophy

These skills emphasize the things docs omit:
- **Visual verification is mandatory** — render and look, don't trust JSON.
- **One focal point, 60–30–10 color, consistent grid/spacing, restrained depth.**
- **Use ids and `command_batch atomic:true`** for reliable, undoable edits.
- **Discover before inventing** — `command_list {query}`, never guess command ids.
- **Read warnings** on every export.

## Version / compatibility

Written against VectorCraft v0.24.x (955 engine commands, 118 effects, 8 MCP prompts). Older versions may lack newer commands; `vectorcraft-verify` explains how to detect and work around that.

## License

Dual-licensed `MIT OR Apache-2.0`, matching VectorCraft. See [LICENSE](LICENSE).

Community skill sources that informed this collection: hermes-pro/storytold-skills, midasmatterhorn/artcraft-skills, and other public VectorCraft skills on GitHub. Thanks to the ArtCraft team and those authors.
