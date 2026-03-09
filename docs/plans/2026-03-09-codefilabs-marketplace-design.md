# Design: CodefiLabs Claude Code Marketplace

**Date**: 2026-03-09
**Status**: Approved

## Overview

A Claude Code plugin marketplace for the `@codefilabs` GitHub org. Users add it once and install any Codefi plugin via Claude Code's plugin system.

```bash
/plugin marketplace add codefilabs/marketplace
/plugin install tailwind-for-agents@codefilabs
/plugin install openrouter-launch@codefilabs
/plugin install tq@codefilabs
```

## Architecture

Option A: separate repos + thin marketplace index.

Each plugin is its own GitHub repo (`github.com/codefilabs/<name>`). The marketplace repo is a pure index — no plugin code lives in it. npm packages and Claude Code plugin files coexist in the same repo; plugin files are excluded from npm via `.npmignore`.

## Marketplace Repo

**Repo:** `github.com/codefilabs/marketplace`
**Local:** `/Users/kk/Sites/codefi/marketplace/`

```
marketplace/
├── .claude-plugin/
│   └── marketplace.json
└── README.md
```

`marketplace.json` lists all three plugins pointing to their respective `codefilabs/` GitHub repos.

## Plugin Contract

Every plugin repo adds these on top of its existing npm/CLI structure:

```
<plugin-repo>/
├── .claude-plugin/
│   └── plugin.json          # name, description, author, homepage
├── .mcp.json                # if the plugin has an MCP server
├── skills/<plugin-name>/
│   └── SKILL.md
├── commands/
│   └── <name>.md
├── hooks/
│   └── hooks.json           # if the plugin has hooks
└── .npmignore               # excludes plugin files from npm publish
```

## Per-Plugin Status

| Plugin | Skills | Commands | MCP | Hooks | Work Needed |
|--------|--------|----------|-----|-------|-------------|
| `tailwind-for-agents` | needs | needs | has `src/mcp/server.js` | needs | fully specced in `tailwind-for-agents/thoughts/shared/research/2026-03-09-plugin-refactor.md` |
| `openrouter-launch` | needs | needs | no | no | plugin scaffold (separate design session) |
| `tq` | has `skills/` | needs | future | needs | `plugin.json` + command + hooks (separate design session) |

## Immediate Scope

1. Create `marketplace/` repo scaffold (`.claude-plugin/marketplace.json` + README)
2. Implement `tailwind-for-agents` plugin files from research doc
3. Placeholder entries for `openrouter-launch` and `tq` in `marketplace.json`

## Deferred

- `openrouter-launch` plugin scaffold design
- `tq` hooks design and command design
- `tq` and `openrouter-launch` GitHub org migration (`vibeathon-us`/`kevnk` → `codefilabs`)
- MCP server for `tq`
- Downstream consumer updates (figma-to-code, figma-mcp, 3 figma agents) after tailwind-for-agents plugin lands
- Removal of old `~/.claude/skills/tailwind-tokens/` monolith skill
