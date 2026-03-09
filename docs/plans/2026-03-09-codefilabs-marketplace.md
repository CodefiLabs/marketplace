# CodefiLabs Marketplace Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create the `codefilabs/marketplace` Claude Code plugin marketplace repo and add Claude Code plugin scaffold to `tailwind-for-agents`.

**Architecture:** Option A — thin marketplace index (`marketplace.json`) pointing to separate plugin repos. Plugin files coexist with npm package in the same repo, excluded from npm via `.npmignore`. Immediate scope: marketplace scaffold + tailwind-for-agents plugin files + tq `plugin.json`. openrouter-launch and tq hooks are deferred.

**Tech Stack:** JSON (marketplace.json, plugin.json, hooks.json), Markdown (SKILL.md, commands), Bash (hook scripts), Node.js (existing MCP server)

---

## Phase 1: Marketplace Repo

### Task 1: Initialize marketplace repo

**Files:**
- Create: `/Users/kk/Sites/codefi/marketplace/.claude-plugin/marketplace.json`
- Create: `/Users/kk/Sites/codefi/marketplace/README.md`

**Step 1: Create `.claude-plugin/marketplace.json`**

```json
{
  "name": "codefilabs",
  "owner": {
    "name": "CodefiLabs",
    "url": "https://github.com/codefilabs"
  },
  "metadata": {
    "description": "AI coding tools from CodefiLabs",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "tailwind-for-agents",
      "source": {
        "source": "url",
        "url": "https://github.com/codefilabs/tailwind-for-agents.git"
      },
      "description": "Resolve your Tailwind CSS theme into AI-friendly token maps. MCP server, skill, and hooks.",
      "version": "0.1.1",
      "strict": true
    },
    {
      "name": "openrouter-launch",
      "source": {
        "source": "url",
        "url": "https://github.com/codefilabs/openrouter-launch.git"
      },
      "description": "Launch AI coding tools with OpenRouter's 400+ model catalog in one command.",
      "version": "1.0.0",
      "strict": true
    },
    {
      "name": "tq",
      "source": {
        "source": "url",
        "url": "https://github.com/codefilabs/tq.git"
      },
      "description": "Task queue runner that spawns Claude AI tasks as independent tmux sessions.",
      "version": "1.0.0",
      "strict": true
    }
  ]
}
```

Note: `openrouter-launch` and `tq` URLs use the future `codefilabs/` org location. Update these once repos are migrated.

**Step 2: Create `README.md`**

```markdown
# CodefiLabs Claude Code Plugins

AI coding tools from [CodefiLabs](https://github.com/codefilabs).

## Add this marketplace

```bash
/plugin marketplace add codefilabs/marketplace
```

## Available plugins

| Plugin | Description | Install |
|--------|-------------|---------|
| [tailwind-for-agents](https://github.com/codefilabs/tailwind-for-agents) | Resolve your Tailwind CSS theme into AI-friendly token maps | `/plugin install tailwind-for-agents@codefilabs` |
| [openrouter-launch](https://github.com/codefilabs/openrouter-launch) | Launch AI coding tools with OpenRouter's 400+ model catalog | `/plugin install openrouter-launch@codefilabs` |
| [tq](https://github.com/codefilabs/tq) | Task queue runner for Claude AI — spawns prompts as tmux sessions | `/plugin install tq@codefilabs` |

## Plugin structure

Each plugin is its own GitHub repo. See individual repos for documentation.
```

**Step 3: Validate JSON**

Run: `python3 -m json.tool /Users/kk/Sites/codefi/marketplace/.claude-plugin/marketplace.json`
Expected: clean JSON output, no errors

**Step 4: Initialize git and commit**

```bash
cd /Users/kk/Sites/codefi/marketplace
git init
git add .claude-plugin/marketplace.json README.md docs/
git commit -m "feat: initial marketplace scaffold with 3 plugin entries"
```

---

## Phase 2: tailwind-for-agents Plugin Scaffold

All files are added to `/Users/kk/Sites/codefi/tailwind-for-agents/`. No existing files are modified except `.npmignore` and `bin/cli.js`.

Reference: `tailwind-for-agents/thoughts/shared/research/2026-03-09-plugin-refactor.md`

### Task 2: Update .npmignore

**Files:**
- Modify: `/Users/kk/Sites/codefi/tailwind-for-agents/.npmignore`

**Step 1: Append plugin scaffold entries**

Current `.npmignore` contains `test/` and `.github/`. Append:

```
.claude-plugin/
skills/
hooks/
commands/
.mcp.json
thoughts/
```

**Step 2: Verify npm pack excludes plugin files**

Run: `npm pack --dry-run --prefix /Users/kk/Sites/codefi/tailwind-for-agents 2>&1 | grep -E 'claude-plugin|skills/|hooks/|commands/|\.mcp'`
Expected: no matches (plugin files excluded)

**Step 3: Commit**

```bash
cd /Users/kk/Sites/codefi/tailwind-for-agents
git add .npmignore
git commit -m "chore: exclude claude code plugin files from npm publish"
```

---

### Task 3: Add plugin.json manifest

**Files:**
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/.claude-plugin/plugin.json`

**Step 1: Create `.claude-plugin/plugin.json`**

```json
{
  "name": "tailwind-for-agents",
  "version": "0.1.1",
  "description": "Resolve your Tailwind CSS theme into AI-friendly token maps. MCP server, skill, and hooks.",
  "author": {
    "name": "codefilabs",
    "url": "https://github.com/codefilabs/tailwind-for-agents"
  },
  "homepage": "https://github.com/codefilabs/tailwind-for-agents",
  "repository": "https://github.com/codefilabs/tailwind-for-agents",
  "license": "MIT",
  "keywords": ["tailwindcss", "design-tokens", "mcp", "figma"]
}
```

**Step 2: Validate JSON**

Run: `python3 -m json.tool /Users/kk/Sites/codefi/tailwind-for-agents/.claude-plugin/plugin.json`
Expected: clean output

**Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add claude code plugin manifest"
```

---

### Task 4: Add MCP server wiring

**Files:**
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/.mcp.json`

**Step 1: Create `.mcp.json`**

```json
{
  "mcpServers": {
    "tailwind-for-agents": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/src/mcp/server.js"]
    }
  }
}
```

**Step 2: Smoke-test MCP server starts**

Run: `timeout 2 node /Users/kk/Sites/codefi/tailwind-for-agents/src/mcp/server.js 2>&1 || true`
Expected: process starts (may hang waiting for stdin — that's correct), or exits cleanly. No import errors.

**Step 3: Commit**

```bash
git add .mcp.json
git commit -m "feat: wire mcp server for claude code plugin"
```

---

### Task 5: Add skill

**Files:**
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/skills/tailwind-for-agents/SKILL.md`

**Step 1: Create `skills/tailwind-for-agents/SKILL.md`**

Adapt from `~/.claude/skills/tailwind-tokens/SKILL.md`. Key changes:
- Skill name: `tailwind-for-agents`
- Prefer MCP tools (`get_tokens`, `lookup`) over CLI when in a Claude session
- CLI reference: `node ${CLAUDE_PLUGIN_ROOT}/bin/cli.js`
- Output path: `.tailwind-tokens/` (after Task 8 fix lands)

```markdown
---
name: tailwind-for-agents
description: >-
  Resolve the project's Tailwind CSS theme into AI-friendly token lookup tables mapping exact
  pixel values to class names and hex colors to utilities. Use at the START of any Figma-to-code
  session so agents have exact resolved values for THIS project rather than assuming Tailwind
  defaults. Triggers when: starting Figma-to-code work, before implementing a design, when the
  pixel-perfect-validator needs correct Tailwind class suggestions, when asked to "generate tailwind
  tokens", "build token lookup table", or "resolve tailwind theme". Also triggers on "design tokens",
  "tailwind config", or any pixel-perfect frontend implementation task.
version: 0.1.1
---

# tailwind-for-agents

Resolve the project's FULL Tailwind config into lookup tables at `.tailwind-tokens/`.

**Why this matters**: Tailwind's defaults live inside `node_modules/tailwindcss` (or the standalone
binary), not in the project's config file. Projects can override any value, so `p-4` might not be
`16px` in this project. tailwind-for-agents finds the actual values.

## Preferred: MCP tools (when already in a Claude session)

If the `tailwind-for-agents` MCP server is connected, prefer these tools:

- **`get_tokens`** — resolve full theme into structured token data (json or markdown)
- **`lookup`** — look up a px/hex/weight value → Tailwind class

## CLI (when MCP is not available)

```bash
# Resolve and write .tailwind-tokens/tokens.{md,json}
node ${CLAUDE_PLUGIN_ROOT}/bin/cli.js --project /absolute/path/to/project

# Optional flags
--output /custom/output/dir   # default: <project>/.tailwind-tokens/
--strategy v3-node|v4-node|v4-cli   # auto-detected by default
--format json|markdown|both   # default: both
```

### Outputs

- `<project>/.tailwind-tokens/tokens.json` — machine-readable token data
- `<project>/.tailwind-tokens/tokens.md` — human-readable lookup tables

## What the markdown contains

1. **Source Files** — `@path/to/file` references to every config, theme, and build output file
2. **Quick Reference Tables** — px→class and hex→class lookup tables for spacing, border-radius,
   font-size, font-weight, colors, breakpoints, and CSS custom properties

## Resolution strategies (auto-detected)

| Strategy | Triggers when |
|----------|--------------|
| `v3-node` | `node_modules/tailwindcss` + v3 in package.json |
| `v4-node` | `node_modules/tailwindcss` + v4 in package.json |
| `v4-cli` | No node_modules tailwindcss; `tailwindcss-rails` gem or binary in PATH |

## Figma-to-code workflow

Run before design implementation so agents have the truth, not training-data guesses:

```bash
node ${CLAUDE_PLUGIN_ROOT}/bin/cli.js --project $CLAUDE_PROJECT_DIR
```

Then reference `.tailwind-tokens/tokens.md` in every design-to-code prompt.
```

**Step 2: Commit**

```bash
git add skills/
git commit -m "feat: add tailwind-for-agents claude code skill"
```

---

### Task 6: Add hooks

**Files:**
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/hooks/hooks.json`
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/hooks/scripts/flag-css-changes.sh`
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/hooks/scripts/regenerate-tokens.sh`

**Step 1: Create `hooks/hooks.json`**

```json
{
  "description": "Auto-regenerate .tailwind-tokens/ when Tailwind source files change during a session",
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/scripts/flag-css-changes.sh\"",
        "timeout": 5
      }]
    }],
    "Stop": [{
      "hooks": [{
        "type": "command",
        "command": "bash \"${CLAUDE_PLUGIN_ROOT}/hooks/scripts/regenerate-tokens.sh\"",
        "timeout": 60
      }]
    }]
  }
}
```

**Step 2: Create `hooks/scripts/flag-css-changes.sh`**

Sets a flag file when a CSS/Tailwind config file is written. Reads tool context from stdin JSON.

```bash
#!/usr/bin/env bash
set -euo pipefail
command -v jq >/dev/null 2>&1 || exit 0
INPUT=$(cat)
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name // empty')
[[ "$TOOL_NAME" != "Write" && "$TOOL_NAME" != "Edit" ]] && exit 0
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
[[ -z "$FILE_PATH" ]] && exit 0
if echo "$FILE_PATH" | grep -qE '\.(css|scss)$|tailwind\.config\.(js|ts|mjs|cjs)$'; then
  PROJECT_DIR="${CLAUDE_PROJECT_DIR:-$(cd "$(dirname "$0")/../.." && pwd)}"
  mkdir -p "$PROJECT_DIR/.claude"
  touch "$PROJECT_DIR/.claude/tailwind_tokens_needed"
fi
exit 0
```

**Step 3: Create `hooks/scripts/regenerate-tokens.sh`**

Runs the CLI at session end if the flag file exists.

```bash
#!/usr/bin/env bash
set -euo pipefail
PROJECT_DIR="${CLAUDE_PROJECT_DIR:-$(cd "$(dirname "$0")/../.." && pwd)}"
FLAG="$PROJECT_DIR/.claude/tailwind_tokens_needed"
[[ ! -f "$FLAG" ]] && exit 0
rm -f "$FLAG"
echo "[tailwind-for-agents] Regenerating .tailwind-tokens/..."
node "${CLAUDE_PLUGIN_ROOT}/bin/cli.js" --project "$PROJECT_DIR" 2>&1
echo "[tailwind-for-agents] Done."
exit 0
```

**Step 4: Make scripts executable**

```bash
chmod +x /Users/kk/Sites/codefi/tailwind-for-agents/hooks/scripts/flag-css-changes.sh
chmod +x /Users/kk/Sites/codefi/tailwind-for-agents/hooks/scripts/regenerate-tokens.sh
```

**Step 5: Validate hooks.json**

Run: `python3 -m json.tool /Users/kk/Sites/codefi/tailwind-for-agents/hooks/hooks.json`
Expected: clean output

**Step 6: Commit**

```bash
git add hooks/
git commit -m "feat: add auto-regenerate hooks for tailwind token changes"
```

---

### Task 7: Add slash command

**Files:**
- Create: `/Users/kk/Sites/codefi/tailwind-for-agents/commands/tailwind-tokens.md`

**Step 1: Create `commands/tailwind-tokens.md`**

```markdown
---
name: tailwind-tokens
description: Resolve the current project's Tailwind CSS theme into token lookup tables (.tailwind-tokens/tokens.md and tokens.json)
argument-hint: "[--strategy v3-node|v4-node|v4-cli] [--format json|markdown|both]"
---

Run `node ${CLAUDE_PLUGIN_ROOT}/bin/cli.js --project $CLAUDE_PROJECT_DIR $ARGUMENTS` and report the output summary.

On success, confirm the output files written and their paths.
On failure, show the full error output so the user can diagnose the strategy or configuration issue.
```

**Step 2: Commit**

```bash
git add commands/
git commit -m "feat: add /tailwind-tokens slash command"
```

---

### Task 8: Fix cli.js output directory default

**Files:**
- Modify: `/Users/kk/Sites/codefi/tailwind-for-agents/bin/cli.js:54`

**Step 1: Find the current default**

Run: `grep -n 'tailwind-for-agents\|tailwind-tokens\|outputDir' /Users/kk/Sites/codefi/tailwind-for-agents/bin/cli.js`

**Step 2: Update the default output directory**

Change:
```js
const outputDir = getArg('--output') || path.join(projectRoot, '.tailwind-for-agents');
```

To:
```js
const outputDir = getArg('--output') || path.join(projectRoot, '.tailwind-tokens');
```

**Step 3: Run existing tests to verify nothing broke**

Run: `npm test --prefix /Users/kk/Sites/codefi/tailwind-for-agents`
Expected: all tests pass

**Step 4: Smoke-test CLI produces correct output dir**

Run: `node /Users/kk/Sites/codefi/tailwind-for-agents/bin/cli.js --project /Users/kk/Sites/codefi/tailwind-for-agents 2>&1 | tail -5`
Expected: output mentions `.tailwind-tokens/` not `.tailwind-for-agents/`

**Step 5: Commit**

```bash
git add bin/cli.js
git commit -m "fix: standardize output dir to .tailwind-tokens/ (matches README)"
```

---

## Phase 3: tq Plugin Manifest

### Task 9: Add plugin.json to tq

**Files:**
- Create: `<tq-local-repo>/.claude-plugin/plugin.json`

Note: Find tq's local repo path with `cat ~/.claude/plugins/known_marketplaces.json` or check GitHub at `github.com/kevnk/tq`. If working locally, the repo should be at a path shown by the known_marketplaces config.

**Step 1: Create `.claude-plugin/plugin.json`**

```json
{
  "name": "tq",
  "version": "1.0.0",
  "description": "Task queue runner that spawns Claude AI tasks as independent tmux sessions.",
  "author": {
    "name": "codefilabs",
    "url": "https://github.com/codefilabs/tq"
  },
  "homepage": "https://github.com/codefilabs/tq",
  "repository": "https://github.com/codefilabs/tq",
  "license": "MIT",
  "keywords": ["claude", "tmux", "task-queue", "automation"]
}
```

**Step 2: Validate JSON**

Run: `python3 -m json.tool <tq-path>/.claude-plugin/plugin.json`
Expected: clean output

**Step 3: Commit**

```bash
git add .claude-plugin/plugin.json
git commit -m "feat: add claude code plugin manifest"
```

---

## Deferred (separate tasks)

- `openrouter-launch` plugin scaffold (needs skill/command design)
- `tq` hooks design and implementation
- `tq` slash command
- Org migrations: `kevnk/tq` → `codefilabs/tq`, `vibeathon-us/openrouter-launch` → `codefilabs/openrouter-launch`
- Update marketplace.json URLs once repos are migrated to `codefilabs/`
- Remove old `~/.claude/skills/tailwind-tokens/` monolith after plugin is confirmed working
- Downstream consumer updates (figma-to-code, figma-mcp, 3 figma agents)
