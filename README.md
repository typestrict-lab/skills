# typestrict skills

Skills that teach AI assistants to work with [typestrict](https://typestrict.ai): prepare data, train a decision model
that beats Jev (or the credit comes back), follow it to the result, and put it to work.

| Skill | What it does |
| --- | --- |
| [typestrict-prepare-data](typestrict-prepare-data/SKILL.md) | Checks and shapes the user's data before training. |
| [typestrict-train](typestrict-train/SKILL.md) | Uploads, answers the review, starts the training (asking before spending), follows it and downloads the model. |
| [typestrict-integrate](typestrict-integrate/SKILL.md) | Calls the trained model from an app or service. |

They use the [typestrict CLI](https://github.com/typestrict-lab/cli) and its MCP server:

```bash
curl -fsSL https://typestrict.ai/install.sh | sh
typestrict login                          # an API key from https://typestrict.ai/settings#api-keys
claude mcp add typestrict -- typestrict mcp   # or your assistant's setup: https://docs.typestrict.ai/agents
```

## Install

**Claude Code**

```bash
git clone https://github.com/typestrict-lab/skills.git
cp -r skills/typestrict-* ~/.claude/skills/        # for every project
# or: cp -r skills/typestrict-* .claude/skills/    # for this project only
```

**Claude.ai and Claude Desktop:** zip a skill folder and upload it in **Settings → Capabilities → Skills**.

**Other assistants** (Codex, Cursor, Copilot, Windsurf, Gemini CLI): skills are plain Markdown. Copy the folders into
your project and point your instruction file (`AGENTS.md`, `.cursor/rules`, `.github/copilot-instructions.md`,
`.windsurf/rules`, `GEMINI.md`) at them, or follow the setup for your assistant at
[docs.typestrict.ai/agents](https://docs.typestrict.ai/agents).

## License

Apache-2.0.
