---
name: dashboard-refresh
description: Create or refresh the Claude Capabilities Dashboard. Scans agents, skills, hooks, and MCP servers from the live system and writes both the .md source and the styled HTML file. Works on first run (no existing files) or as an update.
---

# Dashboard Refresh

Create or update the Claude Code Capabilities Dashboard to reflect the current state of your setup.

## When to Use

Run `/dashboard-refresh` when:
- You want to create the dashboard for the first time (no files needed)
- You installed a new plugin, agent, or skill
- You added or removed MCP servers or hooks
- You added files to `~/.claude/knowledge/`

## Target Files

| File | Role |
|---|---|
| `~/Desktop/Claude Capabilities Dashboard/claude-code-capabilities.md` | Source of truth — written first |
| `~/Desktop/Claude Capabilities Dashboard/claude-code-capabilities-styled_1.html` | Styled HTML view |

---

## Step 1 — Detect mode: Create vs Refresh

Check whether the target files exist:

```
~/.../Claude Capabilities Dashboard/claude-code-capabilities.md
~/.../Claude Capabilities Dashboard/claude-code-capabilities-styled_1.html
```

- **Both exist** → Refresh mode: read the current `.md`, diff against live system, update only what changed
- **Missing or first run** → Create mode: scan everything from scratch, write both files fresh

If the directory doesn't exist, note this — you'll need to create it with the Write tool (which creates parent dirs automatically).

---

## Step 2 — Gather live system state

Scan these sources:

### Agents
- Custom agents: read all `~/.claude/agents/*.md` — extract `name` and `description` from frontmatter
- Plugin agents: read from the system-reminder "Available agent types" list. Group by plugin prefix (`everything-claude-code:`, etc.). Strip the prefix for the display name.

### Skills
- Custom skills: read all `~/.claude/skills/*/SKILL.md` — extract `name` and `description` from frontmatter
- Built-in skills: list skills in the system-reminder that have no plugin prefix (e.g. `update-config`, `loop`, `schedule`, `simplify`, `keybindings-help`, `claude-api`)
- Plugin skills: skills in the system-reminder with a plugin prefix. Group by prefix.

### Hooks
- Read `~/.claude/settings.json` — look for the `hooks` key
- Also check `~/.claude/settings.local.json` if it exists
- For each hook, note: event type (PreToolUse, PostToolUse, PreCompact, SessionStart, Stop, SessionEnd, UserPromptSubmit), matcher/trigger, and description

### MCP Servers
- Local servers: read `~/.claude/settings.json` → `mcpServers` key. These go under "Core · Local (Developer Settings)"
- claude.ai connectors: infer from `mcp__claude_ai_<Name>__*` tool names in the system-reminder. Each unique `<Name>` is a connector. These go under "Core · claude.ai Connected Connectors"
- Plugin MCPs: look for `mcp__plugin_<plugin>_<server>__*` tool prefixes in the system-reminder

### Knowledge Library (optional)
- Check if `~/.claude/knowledge/` exists
- If it does: list all `.md` files excluding `README.md`
- For each file, extract: the filename (slug) and the first `# ` heading as the title
- If the folder does not exist, skip this section entirely — do not create an empty section

---

## Step 3 — Write the .md file

Write `claude-code-capabilities.md` using today's date. Structure:

```markdown
# Claude Code Capabilities
> Generated: YYYY-MM-DD

---

## Agents

### Core (Custom)
| Agent | Description |
|---|---|
| `name` | description |

### Plugin: everything-claude-code (v1.9.0)
| Agent | Description |
|---|---|
...

> Invoke agents with `/agent:<name>` or via the Agent tool in conversation.

---

## Skills

### Core (Built-in)
| Skill | Description |
|---|---|
...

### Core (Custom)
| Skill | Description |
|---|---|
...

### Plugin: claude-mem (v10.6.3)
| Skill | Description |
|---|---|
...

### Plugin: everything-claude-code (v1.9.0)
#### Core Workflow
| Skill | Description |
...

> Invoke with `/skill-name` or `/plugin:skill-name`

---

## Hooks

### Plugin: everything-claude-code (v1.9.0)

#### PreToolUse
| Hook | Trigger | Description |
|---|---|---|
...

#### PostToolUse
...

#### Stop
...

#### SessionStart / SessionEnd / PreCompact
...

---

### Plugin: claude-mem (v10.6.3)
...

---

## MCP Servers

### Core (Local — Developer Settings)
| Name | Description |
|---|---|
...

### Core (claude.ai Connected Connectors)
| Name | Description |
|---|---|
...

### Plugin: everything-claude-code (v1.9.0)
| MCP Server | Key Tools |
|---|---|
...

## Knowledge Library
> Only include this section if `~/.claude/knowledge/` exists and contains .md files.

| File | Title |
|---|---|
| `filename` | First # heading from the file |

> Location: `~/.claude/knowledge/`
```

---

## Step 4 — Write the HTML file

Write the full HTML file. In **Refresh mode**, preserve the existing CSS/JS and only replace the `<div class="wrapper">...</div>` block. In **Create mode**, write the complete file from the template below.

### HTML Template

The complete file structure to use (or preserve) for the HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Claude Code Capabilities</title>
<link href="https://fonts.googleapis.com/css2?family=DM+Mono:wght@300;400;500&family=Syne:wght@400;600;700;800&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0a0a0f; --surface: #111118; --surface2: #1a1a24; --border: #2a2a3a;
    --accent: #7c6bff; --accent2: #ff6b9d; --accent3: #6bffd8; --accent4: #ffd96b;
    --text: #e8e8f0; --muted: #6b6b80;
    --tag-bg: rgba(124,107,255,0.12); --tag-border: rgba(124,107,255,0.3);
    --nav-bg: rgba(10,10,15,0.88);
    --h1-gradient: linear-gradient(135deg,#fff 0%,rgba(255,255,255,0.6) 100%);
    --table-cell: #c8c8d8;
  }
  [data-theme="light"] {
    --bg: #f4f4f9; --surface: #ffffff; --surface2: #ebebf4; --border: #d8d8e8;
    --accent: #6b5ce7; --accent2: #e8547a; --accent3: #00a87a; --accent4: #c98a00;
    --text: #1a1a2e; --muted: #7878a0;
    --tag-bg: rgba(107,92,231,0.1); --tag-border: rgba(107,92,231,0.25);
    --nav-bg: rgba(244,244,249,0.92);
    --h1-gradient: linear-gradient(135deg,#1a1a2e 0%,#4a4a7a 100%);
    --table-cell: #4a4a6a;
  }
  @media (prefers-color-scheme: light) {
    :root:not([data-theme="dark"]):not([data-theme="light"]) {
      --bg: #f4f4f9; --surface: #ffffff; --surface2: #ebebf4; --border: #d8d8e8;
      --accent: #6b5ce7; --accent2: #e8547a; --accent3: #00a87a; --accent4: #c98a00;
      --text: #1a1a2e; --muted: #7878a0;
      --tag-bg: rgba(107,92,231,0.1); --tag-border: rgba(107,92,231,0.25);
      --nav-bg: rgba(244,244,249,0.92);
      --h1-gradient: linear-gradient(135deg,#1a1a2e 0%,#4a4a7a 100%);
      --table-cell: #4a4a6a;
    }
  }
  * { margin:0; padding:0; box-sizing:border-box; }
  html { scroll-behavior: smooth; }
  body { background:var(--bg); color:var(--text); font-family:'DM Sans',sans-serif; font-weight:300; line-height:1.6; min-height:100vh; overflow-x:hidden; }
  body::before { content:''; position:fixed; top:-50%; left:-50%; width:200%; height:200%;
    background: radial-gradient(ellipse 60% 40% at 20% 20%,rgba(124,107,255,0.08) 0%,transparent 60%),
                radial-gradient(ellipse 50% 35% at 80% 70%,rgba(107,255,216,0.06) 0%,transparent 60%),
                radial-gradient(ellipse 40% 30% at 60% 10%,rgba(255,107,157,0.05) 0%,transparent 60%);
    pointer-events:none; z-index:0; }
  .wrapper { position:relative; z-index:1; max-width:1100px; margin:0 auto; padding:0 32px 80px; }
  header { padding:72px 0 56px; border-bottom:1px solid var(--border); margin-bottom:64px;
    display:flex; align-items:flex-end; justify-content:space-between; gap:32px; flex-wrap:wrap; }
  .eyebrow { font-family:'DM Mono',monospace; font-size:11px; letter-spacing:0.18em; text-transform:uppercase;
    color:var(--accent); margin-bottom:16px; display:flex; align-items:center; gap:10px; }
  .eyebrow::before { content:''; display:inline-block; width:24px; height:1px; background:var(--accent); }
  h1 { font-family:'Syne',sans-serif; font-size:clamp(36px,6vw,64px); font-weight:800;
    letter-spacing:-0.02em; line-height:1.05; background:var(--h1-gradient);
    -webkit-background-clip:text; -webkit-text-fill-color:transparent; background-clip:text; }
  .header-meta { font-family:'DM Mono',monospace; font-size:12px; color:var(--muted); text-align:right; line-height:2; }
  .header-meta strong { color:var(--text); font-weight:500; }
  nav { position:sticky; top:0; z-index:100; background:var(--nav-bg); backdrop-filter:blur(20px);
    border-bottom:1px solid var(--border); margin:0 -32px 64px; padding:0 32px; }
  .nav-inner { display:flex; gap:0; overflow-x:auto; scrollbar-width:none; }
  .nav-inner::-webkit-scrollbar { display:none; }
  nav a { font-family:'DM Mono',monospace; font-size:11px; letter-spacing:0.12em; text-transform:uppercase;
    color:var(--muted); text-decoration:none; padding:16px 20px; white-space:nowrap;
    border-bottom:2px solid transparent; transition:color 0.2s, border-color 0.2s; }
  nav a:hover { color:var(--text); border-bottom-color:var(--accent); }
  section { margin-bottom:80px; }
  .section-header { display:flex; align-items:center; gap:16px; margin-bottom:40px;
    padding-bottom:20px; border-bottom:1px solid var(--border); }
  .section-icon { width:40px; height:40px; border-radius:10px; display:flex; align-items:center;
    justify-content:center; font-size:18px; flex-shrink:0; }
  .icon-agents     { background:linear-gradient(135deg,rgba(124,107,255,0.3),rgba(124,107,255,0.1)); }
  .icon-skills     { background:linear-gradient(135deg,rgba(107,255,216,0.3),rgba(107,255,216,0.1)); }
  .icon-hooks      { background:linear-gradient(135deg,rgba(255,217,107,0.3),rgba(255,217,107,0.1)); }
  .icon-mcp        { background:linear-gradient(135deg,rgba(255,107,157,0.3),rgba(255,107,157,0.1)); }
  .icon-knowledge  { background:linear-gradient(135deg,rgba(107,255,216,0.2),rgba(124,107,255,0.2)); }
  h2 { font-family:'Syne',sans-serif; font-size:28px; font-weight:700; letter-spacing:-0.01em; }
  .section-count { margin-left:auto; font-family:'DM Mono',monospace; font-size:11px; color:var(--muted); letter-spacing:0.08em; }
  .subsection { margin-bottom:40px; }
  .subsection-label { font-family:'DM Mono',monospace; font-size:10px; letter-spacing:0.2em; text-transform:uppercase;
    color:var(--muted); margin-bottom:12px; display:flex; align-items:center; gap:8px; }
  .subsection-label::after { content:''; flex:1; height:1px; background:var(--border); }
  h3 { font-family:'Syne',sans-serif; font-size:15px; font-weight:600; color:var(--muted); margin-bottom:16px; }
  h4 { font-family:'Syne',sans-serif; font-size:13px; font-weight:600; color:var(--accent);
    margin:24px 0 10px; text-transform:uppercase; letter-spacing:0.08em; }
  .card-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(280px,1fr)); gap:10px; margin-bottom:16px; }
  .card { background:var(--surface); border:1px solid var(--border); border-radius:10px;
    padding:14px 16px; display:flex; flex-direction:column; gap:6px;
    transition:border-color 0.2s, background 0.2s; cursor:default; }
  .card:hover { border-color:rgba(124,107,255,0.4); background:var(--surface2); }
  .card-name { font-family:'DM Mono',monospace; font-size:12px; font-weight:500; color:var(--accent);
    white-space:nowrap; overflow:hidden; text-overflow:ellipsis; }
  /* color convention: agents = default purple, skills = green, knowledge = gold */
  /* agent `color` frontmatter does NOT map to card color — all agents use plain card-name (purple) */
  .card-name.green { color:var(--accent3); }
  .card-name.pink  { color:var(--accent2); }
  .card-name.gold  { color:var(--accent4); }
  .card-desc { font-size:12px; color:var(--muted); line-height:1.5; }
  .hook-table { width:100%; border-collapse:collapse; font-size:12.5px; margin-bottom:8px; }
  .hook-table thead tr { border-bottom:1px solid var(--border); }
  .hook-table th { font-family:'DM Mono',monospace; font-size:10px; letter-spacing:0.14em;
    text-transform:uppercase; color:var(--muted); text-align:left; padding:8px 12px 10px; font-weight:400; }
  .hook-table td { padding:10px 12px; border-bottom:1px solid rgba(42,42,58,0.5);
    vertical-align:top; color:var(--table-cell); }
  .hook-table tr:last-child td { border-bottom:none; }
  .hook-table tr:hover td { background:var(--surface2); }
  .hook-table td:first-child { font-weight:500; color:var(--text); white-space:nowrap; }
  .trigger-badge { display:inline-block; font-family:'DM Mono',monospace; font-size:10px;
    background:var(--tag-bg); border:1px solid var(--tag-border); color:var(--accent);
    border-radius:4px; padding:2px 6px; white-space:nowrap; }
  .trigger-badge.gold { background:rgba(255,217,107,0.1); border-color:rgba(255,217,107,0.3); color:var(--accent4); }
  .mcp-grid { display:grid; grid-template-columns:repeat(auto-fill,minmax(320px,1fr)); gap:14px; }
  .mcp-card { background:var(--surface); border:1px solid var(--border); border-radius:12px;
    padding:20px; transition:border-color 0.2s; }
  .mcp-card:hover { border-color:rgba(255,107,157,0.4); }
  .mcp-name { font-family:'Syne',sans-serif; font-size:15px; font-weight:700; margin-bottom:10px; color:var(--text); }
  .mcp-tools { display:flex; flex-wrap:wrap; gap:5px; }
  .tool-chip { font-family:'DM Mono',monospace; font-size:10px; background:rgba(255,107,157,0.1);
    border:1px solid rgba(255,107,157,0.25); color:var(--accent2); border-radius:4px; padding:3px 7px; }
  .invoke-hint { margin-top:24px; padding:14px 18px; background:var(--surface);
    border:1px solid var(--border); border-radius:8px; font-family:'DM Mono',monospace;
    font-size:12px; color:var(--muted); }
  .invoke-hint code { color:var(--accent3); background:rgba(107,255,216,0.1); border-radius:4px; padding:2px 6px; }
  .phase-label { font-family:'DM Mono',monospace; font-size:10px; letter-spacing:0.16em;
    text-transform:uppercase; display:inline-flex; align-items:center; gap:6px;
    margin:28px 0 12px; color:var(--accent4); }
  .phase-label::before { content:''; width:6px; height:6px; border-radius:50%; background:var(--accent4); }
  .theme-toggle { display:flex; align-items:center; gap:2px; background:var(--surface);
    border:1px solid var(--border); border-radius:8px; padding:3px; margin-top:12px; }
  .theme-btn { font-family:'DM Mono',monospace; font-size:10px; letter-spacing:0.08em; color:var(--muted);
    background:none; border:none; border-radius:5px; padding:5px 10px; cursor:pointer;
    transition:background 0.15s, color 0.15s; white-space:nowrap; }
  .theme-btn:hover { color:var(--text); }
  .theme-btn.active { background:var(--surface2); color:var(--accent); }
  @keyframes fadeUp { from { opacity:0; transform:translateY(20px); } to { opacity:1; transform:translateY(0); } }
  section { animation:fadeUp 0.5s ease both; }
  section:nth-child(1) { animation-delay:0.05s; }
  section:nth-child(2) { animation-delay:0.12s; }
  section:nth-child(3) { animation-delay:0.19s; }
  section:nth-child(4) { animation-delay:0.26s; }
</style>
</head>
<body>
<div class="wrapper">

  <header>
    <div class="header-left">
      <div class="eyebrow">Configuration Export</div>
      <h1>Claude Code<br>Capabilities</h1>
    </div>
    <div class="header-meta">
      <strong>Generated</strong> YYYY-MM-DD<br>
      <strong>Agents</strong> N · <strong>Skills</strong> N<br>
      <strong>Hooks</strong> N · <strong>MCP Servers</strong> N
      <div class="theme-toggle">
        <button class="theme-btn" data-mode="dark" onclick="setTheme('dark')">🌙 Dark</button>
        <button class="theme-btn" data-mode="light" onclick="setTheme('light')">☀️ Light</button>
        <button class="theme-btn" data-mode="system" onclick="setTheme('system')">💻 System</button>
      </div>
    </div>
  </header>

  <nav>
    <div class="nav-inner">
      <a href="#agents">Agents</a>
      <a href="#skills">Skills</a>
      <a href="#hooks">Hooks</a>
      <a href="#mcp">MCP Servers</a>
      <a href="#knowledge">Knowledge</a>
    </div>
  </nav>

  <!-- AGENTS SECTION -->
  <section id="agents">
    <div class="section-header">
      <div class="section-icon icon-agents">🤖</div>
      <h2>Agents</h2>
      <span class="section-count">N agents</span>
    </div>

    <!-- one .subsection per group, e.g.: -->
    <div class="subsection">
      <div class="subsection-label">Core · Custom</div>
      <div class="card-grid">
        <div class="card">
          <div class="card-name">agent-name</div>
          <div class="card-desc">description</div>
        </div>
      </div>
    </div>

    <div class="subsection">
      <div class="subsection-label">Plugin · everything-claude-code v1.9.0</div>
      <div class="card-grid">
        <!-- one .card per agent -->
      </div>
    </div>

    <div class="invoke-hint">Invoke agents with <code>/agent:&lt;name&gt;</code> or via the Agent tool in conversation.</div>
  </section>

  <!-- SKILLS SECTION -->
  <section id="skills">
    <div class="section-header">
      <div class="section-icon icon-skills">⚡</div>
      <h2>Skills</h2>
      <span class="section-count">N skills</span>
    </div>

    <div class="subsection">
      <div class="subsection-label">Core · Built-in</div>
      <div class="card-grid">
        <!-- card-name green for skills -->
        <div class="card"><div class="card-name green">skill-name</div><div class="card-desc">description</div></div>
      </div>
    </div>

    <div class="subsection">
      <div class="subsection-label">Core · Custom</div>
      <div class="card-grid">
        <!-- each custom skill gets its own .card — never nest subsection-label inside card-grid -->
        <div class="card"><div class="card-name green">skill-name</div><div class="card-desc">description</div></div>
      </div>
    </div>

    <!-- plugin skills: use h4 for sub-categories within a plugin -->
    <div class="subsection">
      <div class="subsection-label">Plugin · everything-claude-code v1.9.0</div>
      <h4>Core Workflow</h4>
      <div class="card-grid"><!-- cards --></div>
      <h4>Code Quality &amp; Review</h4>
      <div class="card-grid"><!-- cards --></div>
      <!-- etc. -->
    </div>

    <div class="invoke-hint">Invoke with <code>/skill-name</code> or <code>/plugin:skill-name</code></div>
  </section>

  <!-- HOOKS SECTION -->
  <section id="hooks">
    <div class="section-header">
      <div class="section-icon icon-hooks">🪝</div>
      <h2>Hooks</h2>
      <span class="section-count">N hooks</span>
    </div>

    <div class="subsection">
      <div class="subsection-label">Plugin · everything-claude-code v1.9.0</div>

      <div class="phase-label">PreToolUse</div>
      <table class="hook-table">
        <thead><tr><th>Hook</th><th>Trigger</th><th>Description</th></tr></thead>
        <tbody>
          <tr><td>Hook name</td><td><span class="trigger-badge">Bash</span></td><td>description</td></tr>
        </tbody>
      </table>

      <div class="phase-label">PostToolUse</div>
      <table class="hook-table"><!-- ... --></table>

      <div class="phase-label">Stop</div>
      <table class="hook-table"><!-- ... --></table>

      <!-- lifecycle hooks use .gold badge -->
      <div class="phase-label">SessionStart / SessionEnd / PreCompact</div>
      <table class="hook-table">
        <thead><tr><th>Hook</th><th>Phase</th><th>Description</th></tr></thead>
        <tbody>
          <tr><td>Hook name</td><td><span class="trigger-badge gold">SessionStart</span></td><td>description</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <!-- MCP SECTION -->
  <section id="mcp">
    <div class="section-header">
      <div class="section-icon icon-mcp">🔌</div>
      <h2>MCP Servers</h2>
      <span class="section-count">N servers</span>
    </div>

    <div class="subsection">
      <div class="subsection-label">Core · Local (Developer Settings)</div>
      <div class="mcp-grid">
        <div class="mcp-card">
          <div class="mcp-name">Server Name</div>
          <div class="card-desc" style="margin-bottom:12px">description</div>
          <div class="mcp-tools">
            <span class="tool-chip">tool-name</span>
          </div>
        </div>
      </div>
    </div>

    <div class="subsection">
      <div class="subsection-label">Core · claude.ai Connected Connectors</div>
      <div class="mcp-grid"><!-- mcp-cards --></div>
    </div>

    <div class="subsection">
      <div class="subsection-label">Plugin · everything-claude-code v1.9.0</div>
      <div class="mcp-grid"><!-- mcp-cards --></div>
    </div>
  </section>

  <!-- KNOWLEDGE SECTION — only render if ~/.claude/knowledge/ exists and has .md files -->
  <section id="knowledge">
    <div class="section-header">
      <div class="section-icon icon-knowledge">📚</div>
      <h2>Knowledge Library</h2>
      <span class="section-count">N notes</span>
    </div>

    <div class="subsection">
      <div class="subsection-label">~/.claude/knowledge</div>
      <div class="card-grid">
        <div class="card">
          <div class="card-name gold">filename-slug</div>
          <div class="card-desc">Title extracted from first # heading</div>
        </div>
      </div>
    </div>

    <div class="invoke-hint">Explanatory notes on how Claude Code works — concepts, comparisons, mental models. Not rules (memory) or prompts (skills).</div>
  </section>

</div>

<script>
  function setTheme(mode) {
    if (mode === 'system') {
      document.documentElement.removeAttribute('data-theme');
      localStorage.removeItem('theme');
    } else {
      document.documentElement.setAttribute('data-theme', mode);
      localStorage.setItem('theme', mode);
    }
    document.querySelectorAll('.theme-btn').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.mode === mode);
    });
  }
  (function() {
    const saved = localStorage.getItem('theme');
    const mode = saved || 'system';
    if (saved) document.documentElement.setAttribute('data-theme', saved);
    document.querySelectorAll('.theme-btn').forEach(btn => {
      btn.classList.toggle('active', btn.dataset.mode === mode);
    });
  })();
</script>
</body>
</html>
```

Fill in the placeholder content (`N`, `YYYY-MM-DD`, card lists, etc.) from the data gathered in Step 2.

---

## Step 5 — Counts & summary line

Update the header stats line in the HTML:
```html
<strong>Agents</strong> 26 · <strong>Skills</strong> 130+<br>
<strong>Hooks</strong> 23 · <strong>MCP Servers</strong> 9
```
Count actual items from Step 2. Use `N+` notation if a section has many items from a plugin (e.g. `130+` skills).

---

## Step 6 — Verify

After writing both files confirm:
- Date updated to today in both files
- Counts in HTML header match actual totals
- All agents/skills/hooks/MCPs from Step 2 are present
- Knowledge section present if `~/.claude/knowledge/` exists, absent if it doesn't
- In Refresh mode: no items removed unless they genuinely no longer exist

---

## Notes

- Plugin versions can be found in the plugin's package.json or manifest. If unknown, omit the version.
- "Core · Local (Developer Settings)" entries must be maintained manually — do not auto-remove them.
- claude.ai connectors come from `mcp__claude_ai_<Name>__*` tool prefixes. Each unique `<Name>` is one connector.
- If the target folder doesn't exist, the Write tool will create it automatically.
- Knowledge Library section is optional and plugin-independent — include it only if `~/.claude/knowledge/` exists. Any user can create this folder to get the section.
