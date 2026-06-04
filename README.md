# ITONICS Skills for AI Agents

Skills for Claude Code, Codex CLI, Gemini CLI, Cursor, Windsurf, OpenCode and other AI agents that operate against an [ITONICS Innovation](https://www.itonics-innovation.com/) tenant.

## Skills

| Skill | Description |
|-------|-------------|
| **[itonics-cli](itonics-cli/SKILL.md)** | Primary CLI — elements, types, files, attachments, watches, likes |

## Install

### 1 — Install the CLI

The `itonics` binary ships from its own repo: [`itonics/itonics-cli`](https://github.com/itonics/itonics-cli).

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

Env-var alternative if you prefer not to write a config file:

```bash
export ITONICS_DOMAIN=https://your-tenant.itonics.io
export ITONICS_API_KEY=your-api-key
export ITONICS_SPACE=your-space-uri
```

### 2 — Add the skill

#### Claude Code (manual)

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
├── SKILL.md            # umbrella skill (lists all sub-skills)
├── LICENSE
└── itonics-cli/
    └── SKILL.md        # documents the `itonics` CLI for AI agents
```

Each top-level directory is one skill. New skills can be dropped in as sibling directories, each with their own `SKILL.md`.

## License

MIT — see [LICENSE](LICENSE).
