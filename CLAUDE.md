# CLAUDE.md

Agent skills for Badger Commerce. Each skill is `skills/<name>/SKILL.md`; the README's table lists
them all.

## This repo is the source of truth

- Edit skills here, and only here, through a PR to `main` (merge commits). Never edit the copies a
  project imports (`.agents/skills/`, `.claude/skills/`, listed in its `skills-lock.json`): they are
  overwritten on the next refresh.
- Consumers refresh with `npx skills update -p -y` from the project root after the PR merges; a skill
  the project doesn't have yet is added with
  `npx skills add Badger-Commerce/badger-skills --skill <name> -a claude-code`.
- A new skill gets a row in the README's table.
- Keep each skill's voice and structure. Skills complement the MCP server's own instructions
  (`spring.ai.mcp.server.instructions` in badger-commerce `web/src/main/resources/application.yml`);
  don't repeat them.

## Verify a skill against badger-commerce `development`

Skills describe the platform as it is on badger-commerce's `development` branch. Before a PR, check
every MCP tool name, action, parameter, result field, extension name and property, slot, config key
and theme or preset name the change mentions. Don't document behaviour you haven't found in the code.

From a badger-commerce checkout:

```bash
git -c 'credential.https://gitlab.com.helper=!/opt/homebrew/bin/glab auth git-credential' \
  fetch https://gitlab.com/badger-commerce/badger-commerce.git development
git show FETCH_HEAD:<path>
```

Where to look:

| What | Where |
|------|-------|
| MCP tools: names, descriptions, actions, params | `ai/src/main/java/uk/co/kedos/badger/settbuilder/ai/mcp/tools/*.java` (`@McpTool`, `@McpToolParam`, the `*_ACTIONS` lists, result records) |
| MCP behaviour, tool by tool | `docs/features/mcp-oauth2.md` ("Tool Conventions") |
| Extensions: name (`getName()`), properties, default slot and priority | the extension class, e.g. `commerce-core/.../extensions/content/CallToActionExtension.java` |
| jsonComponent catalog, tokens, validation | `docs/features/json-component.md` and the validator/renderer classes |
| Hero, Ridge and other themes | `docs/features/extension-system.md`, `docs/features/*-theme.md`, `ThemeDefaults.java` |
| Taxonomies, category trees, search | `docs/features/product-taxonomy.md`, `catalogue-seo.md`, `search-synonyms.md`, `search-ranking.md` |
| Config keys | `common/.../ConfigKeyDefaults.java` |

Cite the files you checked in the PR description.
