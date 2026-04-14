# ai-toolkit

AI developer tooling used across the devmonks engineering team — plugins, rules, prompts, and MCP servers for Claude Code, Cursor, and Copilot.

## What this is

A shared repository of AI productivity tools for devmonks engineers. It ships **behavior** — commands, skills, hooks, agents, and MCP servers — not engineering rules. Stack conventions, commit formats, and code standards live in the devmonks handbook.

The goal: everyone on the team gets the same upgraded AI assistant experience with one install command.

## Claude Code concepts

A short glossary of the building blocks Claude Code exposes. Pick the right one for the job using the rule of thumb below.

| Concept | What it is | Who triggers it | Lives in |
|---|---|---|---|
| **CLAUDE.md** | Static rules loaded every turn | Automatic | Repo or project root |
| **Slash command** | A prompt template users invoke manually | User types `/name` | `commands/*.md` |
| **Skill** | Procedural know-how Claude self-applies when relevant | Claude (based on description) | `skills/<name>/SKILL.md` |
| **Subagent** | Isolated Claude instance with its own context and tools | Claude (via Agent tool) | `agents/*.md` |
| **Hook** | Shell command the harness runs on lifecycle events (`PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `Stop`, …) | Event-driven, deterministic | `hooks/hooks.json` |
| **Plugin** | A bundle of the above, distributable via marketplace | Installed once, persists | `plugins/<name>/` |
| **MCP server** | External process exposing tools or data to Claude | Tool call at runtime | Standalone, referenced in config |
| **Marketplace** | A repo listing installable plugins | `/plugin marketplace add owner/repo` | `.claude-plugin/marketplace.json` |

### Rule of thumb — which one should I use?

- **Always-on rule** → `CLAUDE.md`
- **User-initiated workflow** → slash command
- **Self-triggered know-how** → skill
- **Deterministic automation on an event** → hook
- **Parallel or isolated work** → subagent
- **New data source or external tool** → MCP server
- **Sharing any of the above with the team** → wrap it in a plugin

**Key distinction:** skills are *probabilistic* (Claude decides when to apply), hooks are *deterministic* (the harness always runs them), commands are *manual* (the user types them). The same task can often be built three ways — pick based on who triggers it.

## Repository structure

```
ai-toolkit/
├── README.md
├── LICENSE
├── .claude-plugin/
│   └── marketplace.json        ← Claude Code marketplace entry
└── plugins/                     ← Claude Code plugins
    └── devmonks-git/
        ├── .claude-plugin/plugin.json
        └── commands/           ← /commit, /branch, /create-pr
```

## Plugin layout

Plugins are split by **stack/persona**, not by content type. A React-only engineer installs a different set than a backend engineer, and neither pulls in tooling they will not use. Claude's context stays focused on what that person actually does.

| Plugin | Scope | Ship when |
|---|---|---|
| `devmonks-git` | Git & GitHub workflow — universal | **shipped** |
| `devmonks-frontend` | React / Next.js / web UI helpers | first frontend command lands |
| `devmonks-backend` | API / DB / server helpers | first backend command lands |
| `devmonks-mobile` | Flutter / iOS / Android helpers | first mobile command lands |
| `devmonks-devops` | CI / Docker / deploy / infra helpers | first devops command lands |

Rules of thumb:

- **Do not create empty placeholder plugins.** A plugin directory exists only once it has real content.
- **`devmonks-git` is the one universal plugin** — every engineer installs it. All other plugins are opt-in.
- **Split by persona, not by topic purity.** Do not split `devmonks-git-commit` from `devmonks-git-branch` — they serve one workflow. But do split `devmonks-frontend` from `devmonks-backend` — those serve different workflows.
- **Start as one plugin, split later.** When a plugin grows past ~8–10 items and clear sub-personas emerge, split it. Renaming and moving is cheap.

### Planned additions (tool-agnostic dirs)

As the toolkit grows beyond Claude Code, new vendor directories will be added alongside `plugins/`:

```
├── cursor/            ← Cursor rules (.cursor/rules/*.mdc templates)
├── copilot/           ← GitHub Copilot instructions templates
├── mcp-servers/       ← vendor-agnostic MCP servers
└── shared/            ← prompts and templates usable across tools
```

## Installation

### Claude Code

```bash
/plugin marketplace add devmonks-co/ai-toolkit
/plugin install devmonks-git@ai-toolkit
```

Cursor, Copilot, and MCP server setup will be documented here as those directories land.

## External resources

Curated list of plugins, skills, and agents we may adopt from the community. Verify each link before adopting anything — the ecosystem moves fast.

### Official
- [Claude Code documentation](https://docs.claude.com/claude-code)
- [Anthropic skills repository](https://github.com/anthropics/skills)
- [Model Context Protocol servers](https://github.com/modelcontextprotocol/servers)

### Plugins & marketplaces
_Add as discovered._

### Skills & agents worth borrowing
_Add as discovered._

### MCP servers to evaluate
_Add as discovered._

## License

MIT — see [LICENSE](LICENSE).
