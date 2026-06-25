---
name: agent-skills
description: ITONICS skills for AI agents. CLI, elements, files, attachments, watches and likes on the ITONICS Innovation platform.
metadata:
  version: "0.1.0"
  author: itonics
  repository: https://github.com/itonics/agent-skills
  tags: itonics,innovation,cli,odata,api
---

# ITONICS Skills

Skills for Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode and other AI agents operating against an [ITONICS Innovation](https://www.itonics-innovation.com/) tenant.

## Core skills

| Skill | Description |
|-------|-------------|
| **[itonics-cli](itonics-cli/SKILL.md)** | Primary CLI — elements, types, files, attachments, watches, likes |

## No terminal? Use the MCP connector

For clients without a CLI (**Claude.ai, Claude Desktop, Cursor, ChatGPT**), the hosted [itonics-mcp](https://github.com/itonics/itonics-mcp) connector exposes the same surface as self-describing MCP tools — no skill file or local config needed. Add a custom connector / remote MCP server and paste the Streamable HTTP endpoint `https://itonics-mcp.com/mcp` (include the `/mcp` path), or for Claude Code run `claude mcp add --transport http itonics https://itonics-mcp.com/mcp`. Auth runs through a browser OAuth flow. For per-type **property-write encoding**, the connector reuses [itonics-cli/SKILL.md → Property writes](itonics-cli/SKILL.md#property-writes).

## Install

### 1 — Install the CLI

The `itonics` binary ships from its own repo, [`itonics/itonics-cli`](https://github.com/itonics/itonics-cli):

```bash
# Homebrew (macOS / Linux)
brew tap itonics/itonics https://github.com/itonics/itonics-cli
brew install itonics
```

```powershell
# Scoop (Windows)
scoop bucket add itonics https://github.com/itonics/itonics-cli
scoop install itonics
```

Then log in:

```bash
itonics login   # prompts for domain, space, API key; saves to ~/.config/itonics/config.toml (chmod 0600)
itonics whoami  # confirm
```

### 2 — Add the skill(s)

#### Claude Code (manual)

Clone this repo and symlink each skill into `~/.claude/skills/`:

```bash
git clone https://github.com/itonics/agent-skills.git
ln -s "$PWD/agent-skills/itonics-cli" ~/.claude/skills/itonics-cli
```

#### `skills` package runtime

```bash
npx skills add itonics/agent-skills --skill itonics-cli --full-depth -y
```

## Layout

```
agent-skills/
├── README.md
├── SKILL.md            # this umbrella skill
├── LICENSE
└── itonics-cli/
    └── SKILL.md        # documents the `itonics` CLI for AI agents
```

Each top-level directory is one skill. New skills can be dropped in as sibling directories, each with their own `SKILL.md`.

## License

MIT — see [LICENSE](LICENSE).
