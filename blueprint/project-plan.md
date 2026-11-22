# Project Plan

> One of the two planning docs you provide. Use as much detail as the project
> needs, including rationale, constraints, examples, edge cases, and explicit
> exclusions that should guide later feature work. Draft it directly, develop it
> through any AI conversation, or optionally run `/discovery` for a guided deep
> planning session. The content is always yours to direct. When it is filled in,
> run `/overview` to generate the project overview from this plus `build-plan.md`.

## 1. Problem - What problem are we solving?

Coding agents need repository-native VectorCraft knowledge to create professional vector illustrations through the VectorCraft MCP server. This project packages that knowledge as portable, Agent Skills-compatible instructions.

## 2. Users - Who is this for?

Developers who want to install and use VectorCraft skills with coding agents such as OpenCode, Codex, Claude Code, GitHub Copilot, and Gemini CLI.

## 3. Features - What does the MVP need?

- A router skill that determines when to use VectorCraft and routes to focused companion skills.
- An exact MCP tool and parameter reference.
- Composition, color, typography, depth, and critique guidance.
- Tested build recipes for common vector artwork.
- Visual, structure, export, and error-recovery verification guidance.
- Installation and compatibility documentation for supported coding agents.

## 4. Data - What are we storing?

The project stores no application data. It stores Markdown skill documents, frontmatter metadata, examples, and licensing information.

## 5. Tech - What stack are we using?

Markdown and the agentskills.io `SKILL.md` layout. The skills integrate with VectorCraft through `vectorcraft-cli` and its MCP server.

> TODO (confirm): Confirm the minimum supported VectorCraft CLI and MCP versions.

## 6. Monetize - How will this make money?

No monetization plan.

## 7. UI/UX - How should this look and feel?

Documentation should be concise, portable, and easy for coding agents to follow. Skills should use progressive disclosure, with the router directing agents to the smallest relevant companion skill. Vector artwork guidance should emphasize composition, clear hierarchy, explicit paint, visual preview, and export verification.

## 8. Deployment - Where and how will this ship?

The project ships as a Git repository containing reusable `skills/<name>/SKILL.md` files. Users may install it with `npx skills add`, or copy selected skill directories into an agent's skills folder.

There is no application build, server, database, environment-variable configuration, or deployed web service.

> TODO (confirm): Supply the canonical repository owner and URL for clone and `npx skills add` examples.

## 9. Usage model and constraints (optional)

No redesign is planned. The existing structure and skill split are intentional.
