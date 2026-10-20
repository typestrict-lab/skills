# typestrict skills

Skills that put AI assistants to work with [typestrict](https://typestrict.ai): they prepare the data, check and fix
the training proposal, train a decision model that beats Jev, follow it to the result, and put it to work.

| Skill | What it does |
| --- | --- |
| [typestrict-prepare-data](typestrict-prepare-data/SKILL.md) | Checks and shapes the user's data before training. |
| [typestrict-train](typestrict-train/SKILL.md) | Uploads, answers the review, starts the training (asking before spending), follows it and downloads the model. |
| [typestrict-integrate](typestrict-integrate/SKILL.md) | Calls the trained model from an app or service. |

They use the typestrict MCP server, remote (`https://api.typestrict.ai/mcp`, sign in with your typestrict account) or
local (`typestrict mcp` from the [typestrict CLI](https://github.com/typestrict-lab/cli)), and the CLI itself:

```bash
claude mcp add --transport http typestrict https://api.typestrict.ai/mcp   # or your assistant's setup:
                                                                           # https://docs.typestrict.ai/agents
```

## Install

```bash
git clone https://github.com/typestrict-lab/skills.git
```

| Assistant | Copy the `typestrict-*` folders into |
| --- | --- |
| Claude Code | `~/.claude/skills/` (every project) or `.claude/skills/` |
| Codex, Cursor, GitHub Copilot, Antigravity, opencode | `.agents/skills/` in the project, or `~/.agents/skills/` |
| Devin Desktop and Devin CLI | `.devin/skills/` |

```bash
mkdir -p .agents/skills && cp -r skills/typestrict-* .agents/skills/
```

**Claude.ai and Claude Desktop:** zip a skill folder (the zip holds `typestrict-train/SKILL.md`) and upload it in
**Customize → Skills**.

**ChatGPT:** add the skills' text to the project instructions. Setup for each assistant:
[docs.typestrict.ai/agents](https://docs.typestrict.ai/agents).

## License

Apache-2.0.
