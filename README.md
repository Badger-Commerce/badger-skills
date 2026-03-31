# Badger Commerce Agent Skills

Claude Code plugin providing agent skills for [Badger Commerce](https://www.bdgr.co.uk) — domain model guide, workflow reference, component building, taxonomy design, and site management via MCP tools.

## Installation

```bash
claude plugin install badger-skills
```

## Skills

| Skill | Description | Invocable |
|-------|-------------|-----------|
| `badger-mcp` | Domain model guide, tool map, and common workflows for all Badger Commerce MCP tools | `/badger-mcp` |
| `hero` | Add and configure Hero section extensions with 5 preset layouts | auto |
| `json-components` | Build and edit freeform JSON components — full component catalog, semantic style tokens, data binding | auto |
| `taxonomy` | Design and construct product taxonomies — common patterns for apparel, food, retail, and charity shops | auto |

Start with `/badger-mcp` for an overview of the platform and how the tools fit together. The other skills are triggered automatically when you're working in their domain.

## Prerequisites

These skills are designed to work with the Badger Commerce MCP server, which provides tools for managing extensions, media, products, collections, and site configuration.

## License

Copyright (c) Badger Commerce Limited, a subsidiary of Kedos Consulting Limited.