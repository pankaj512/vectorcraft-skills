# Coding Standards

## Skill Format

- Use one agent skill per `skills/<name>/SKILL.md`.
- Keep a skill concise and under the existing progressive-disclosure pattern.
- Put frontmatter before the Markdown body.
- Include `name`, `description`, `license`, and relevant `metadata` in frontmatter.
- Use the router skill for routing and point to focused companion skills for exact details.
- Preserve compatibility with agent clients that read `SKILL.md` directly.

## Markdown

- Use ATX headings and short sections.
- Prefer lists and tables for instructions and reference matrices.
- Use fenced code blocks for commands and JSON examples.
- Keep examples copyable and use absolute paths for VectorCraft file operations.
- State tool-name aliases when a host varies MCP prefixes.
- Use the existing names: `vectorcraft`, `vectorcraft-mcp`, `vectorcraft-design`, `vectorcraft-recipes`, and `vectorcraft-verify`.

## VectorCraft Workflow

- Discover engine commands before running unfamiliar command ids.
- Inspect existing documents before editing.
- Draw back to front and batch independent operations when useful.
- Render and visually inspect after every major edit.
- Use stable object ids for follow-up edits.
- Read export warnings and verify SVG/PNG/PDF deliverables.
- Keep examples accurate for the documented VectorCraft CLI and MCP versions.
- When a tool signature is uncertain, mark it as a TODO and verify it against the MCP command reference.

## Naming

- Skill directory names use kebab-case.
- Frontmatter names match the skill directory name.
- Command and effect ids retain their exact case and dotted notation.
- Use descriptive recipe headings and filenames.

## Error Handling

- Do not present JSON success as visual or export success.
- Document the concrete recovery step for CLI, MCP, command, effect, font, or export failures.
- Distinguish headless limitations from remote/UI-only failures.
- Use absolute paths and preserve automation-root confinement requirements.

## Testing

No test runner or test command is currently configured, so testing is opt-in. Do not describe a gate that the repository does not have.

Validation for skill changes should be evidence-based:
- Check frontmatter and Markdown structure.
- Verify links and relative skill references.
- Check that documented MCP tool names and parameters match the command reference.
- Run a real VectorCraft preview/export when the change affects workflows or examples.
- Run `vectorcraft-cli info` when CLI compatibility is relevant.

There is no package build or lint command in this repository.

## Code Quality

- Remove inaccurate or duplicated documentation.
- Keep instructions action-oriented and avoid speculative abstractions.
- No commented-out content.
- Preserve MIT OR Apache-2.0 licensing.

## Browser Verification

No browser test command is configured. Browser or live VectorCraft verification is optional and should be used when the change affects rendering or exported visuals.

## Comments

Write code and documentation that explains itself. Comment only non-obvious reasoning or constraints.

## Writing

- No em dashes (U+2014) in generated content: docs, comments, commit messages, READMEs, specs.
- Use commas, parentheses, or a hyphen instead of em or en dashes.
