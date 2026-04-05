# Claude Setup Dashboard

A Claude Code skill that generates a visual dashboard of your Claude setup — agents, skills, hooks, and MCP servers — as a styled HTML file you can open in any browser.
<img width="1458" height="965" alt="Screenshot 2026-04-02 at 14 33 14" src="https://github.com/user-attachments/assets/85528d32-be24-447a-a0fc-c5dc14fff708" />

---

## What it generates

A single HTML file (`claude-code-capabilities-styled_1.html`) with five sections:

- **Agents** — all agents available to you, grouped by plugin or custom
- **Skills** — all slash commands (`/skill-name`), grouped and categorised
- **Hooks** — automated behaviors that run before/after tool use or on session events
- **MCP Servers** — connected tools and integrations (local, claude.ai connectors, plugins)
- **Knowledge Library** — explanatory notes about how Claude Code works (optional, see below)

The dashboard supports dark mode, light mode, and system preference, with a toggle in the header.

---

## Installation

1. Copy `SKILL.md` into your Claude skills folder:

```bash
mkdir -p ~/.claude/skills/dashboard-refresh
cp SKILL.md ~/.claude/skills/dashboard-refresh/SKILL.md
```

2. Restart Claude Code (or start a new session). The skill is picked up automatically.

---

## Usage

In any Claude Code session, run:

```
/dashboard-refresh
```

Claude will scan your live setup and write two files to `~/Desktop/Claude Capabilities Dashboard/`:

| File | Purpose |
|---|---|
| `claude-code-capabilities.md` | Plain text source — easy to read and version |
| `claude-code-capabilities-styled_1.html` | Styled visual dashboard — open in a browser |

If the folder doesn't exist yet, it's created automatically. **No existing files required** — works on first run.

---

## Knowledge Library (optional)

If you create a `~/.claude/knowledge/` folder, the dashboard will automatically include a **Knowledge Library** section listing all `.md` files inside it.

This folder is meant for explanatory notes about how Claude Code works — concepts, comparisons, mental models. Not rules (that's memory) or prompts (that's skills). Just reference notes you want to be able to look up.

```bash
mkdir -p ~/.claude/knowledge
```

Add any `.md` file with a `# Title` heading and it will appear in the dashboard on the next `/dashboard-refresh`. If the folder doesn't exist, the section is simply omitted — nothing breaks.

---

## Keeping it up to date

Re-run `/dashboard-refresh` any time you:
- Install a new plugin, add an MCP server, or create a custom skill
- Add files to `~/.claude/knowledge/`

Claude will diff against the existing dashboard and update only what changed.

---

## Notes

- MCP servers configured in Claude's developer settings must be added to the dashboard manually — they can't be auto-detected
- claude.ai connected connectors (Zapier, Notion, Gamma, etc.) are detected automatically from your session context.

<img width="1458" height="965" alt="Screenshot 2026-04-02 at 14 38 54" src="https://github.com/user-attachments/assets/d95a697b-7ce3-4af2-8500-595dfc9f85d9" />


