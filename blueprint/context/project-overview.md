# VectorCraft Skills - Project Overview

<!-- blueprint:source-hash f9cf33c385094493377d16d96b0d96eafb0e37579e0bc4a293729637b25834d4 -->

Portable Agent Skills documents that help coding agents create, edit, verify, and export professional VectorCraft illustrations.

## Problem

Coding agents need reliable, repository-native guidance for using the VectorCraft MCP server. The project packages that guidance into portable skills that cover routing, exact MCP usage, design quality, recipes, and verification.

## Users

- Developers installing reusable VectorCraft skills in OpenCode, Codex, Claude Code, GitHub Copilot, Gemini CLI, or compatible agents.

## Usage model

- Documentation and skill files are public project assets.
- The project stores no application data and runs no application service.
- No redesign of the existing skill structure is planned.
- Scale, tenancy, authentication, security, and compliance requirements are not established.

## Features

1. **VectorCraft router skill** - Exists and routes vector-art requests to the appropriate companion skill.
2. **MCP reference** - Exists and documents exact tools, parameters, paint, gradients, effects, batching, and exports.
3. **Design guidance** - Exists and covers composition, spacing, color, typography, depth, icons, logos, and final critique.
4. **Build recipes** - Exists and provides patterns for posters, logos, icons, badges, charts, tracing, recoloring, and exports.
5. **Visual verification** - Exists and covers structure, warnings, exports, and CLI/MCP recovery.
6. **Portable installation documentation** - Exists and explains MCP setup and installation across coding-agent clients.

## Data model

There is no persisted application data model. The project manages Markdown documents and frontmatter metadata:

### Skill

- `name` (string) - Frontmatter identifier matching the skill directory name.
- `description` (string) - Agent routing and trigger description.
- `license` (string) - MIT OR Apache-2.0 licensing declaration.
- `metadata` (object) - Optional author, version, compatibility, or testing information.
- `body` (Markdown) - Instructions for routing or a focused VectorCraft workflow.

Skills reside at `skills/<name>/SKILL.md`. There are no stored records, relationships, or database constraints.

## Tech stack

- **Markdown** - Canonical format for the skill collection and project documentation.
- **Agent Skills `SKILL.md` layout** - Portable structure for compatible coding agents.
- **VectorCraft CLI** - Launches the MCP server in headless or remote mode.
- **VectorCraft MCP server** - Supplies drawing, inspection, effect, transform, and export capabilities.
- **GitHub Actions** - Releases archived skills and tagged releases on changes to `main`.

## Monetization

No monetization plan.

## UI/UX

- Keep instructions concise and portable.
- Use progressive disclosure from the router to focused companion skills.
- Emphasize hierarchy, composition, explicit paint, visual preview, and export verification.
- Preserve ATX headings, short sections, lists, tables, fenced commands, and copyable JSON examples.

## Deployment

- Distribution is a Git repository containing `skills/<name>/SKILL.md`.
- Installation supports `npx skills add` or copying skill directories into an agent's skills folder.
- `.github/workflows/release.yml` archives `skills/` and creates a tagged GitHub release.
- No application build, server, database, environment variables, web host, or health check are configured.

## Open questions

- > TODO: Confirm the minimum supported VectorCraft CLI and MCP versions.
- > TODO: Supply the canonical repository owner and URL for clone and `npx skills add` examples.
