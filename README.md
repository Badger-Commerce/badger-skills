# Badger Commerce Agent Skills

Claude Code plugin providing agent skills for [Badger Commerce](https://www.bdgr.co.uk) — domain model guide, workflow reference, component building, taxonomy design, and site management via MCP tools.

## Installation

As a Claude Code plugin:

```bash
claude plugin install badger-skills
```

Or into a project with [`skills`](https://www.npmjs.com/package/skills), which copies them into
`.agents/skills/` (symlinked from `.claude/skills/`) and records them in `skills-lock.json`. This is
how badger-commerce consumes them:

```bash
npx skills add Badger-Commerce/badger-skills --skill badger-mcp -a claude-code
```

## Skills

| Skill | Description | Invocable |
|-------|-------------|-----------|
| `badger-mcp` | Domain model guide, tool map, and common workflows for all Badger Commerce MCP tools | `/badger-mcp` |
| `animation-scene` | Build GSAP-powered animation scenes — SaaS-style reveals, mock-browser demos, scroll-scrub effects, composed from a JSON spec with server-resolved bindings | auto |
| `hero` | Add and configure Hero section extensions with 5 preset layouts | auto |
| `json-components` | Build and edit freeform JSON components — full component catalog, semantic style tokens, data binding | auto |
| `taxonomy` | Design and construct product taxonomies — common patterns for apparel, food, retail, and charity shops | auto |
| `category-trees` | Build and run category trees (`/c/` pages) — backings, cross-cutting pages, product breadcrumbs, replaced collections, and moving a collection-based shop onto a tree | auto |
| `search-tuning` | Diagnose search ("why doesn't X show for Y"), close zero-result gaps, manage synonyms and review AI suggestions | auto |

Start with `/badger-mcp` for an overview of the platform and how the tools fit together. The other skills are triggered automatically when you're working in their domain.

## Editing skills

This repo is the only place skills are edited. The copies in a consuming project
(`.agents/skills/`, `.claude/skills/`) are generated: an edit there is lost on the next refresh and
never reaches anyone else.

1. Branch from `main`, change `skills/<skill>/SKILL.md`, and open a PR. Check every tool, action,
   parameter, extension and config key you mention against badger-commerce's `development` branch
   (see `CLAUDE.md`).
2. After the PR merges, refresh each consuming project from its root:

   ```bash
   npx skills update -p -y
   ```

   A skill the project doesn't have yet (e.g. a new one) is added with
   `npx skills add Badger-Commerce/badger-skills --skill <name> -a claude-code`.
3. A new skill also needs a row in the table above.

## Prerequisites

These skills are designed to work with the Badger Commerce MCP server, which provides tools for managing extensions, media, products, collections, and site configuration.

## License

Copyright (c) Badger Commerce Limited, a subsidiary of Kedos Consulting Limited.