# CLAUDE.md

Rules for contributing to this repository. Follow these exactly.

## GitHub org

The org is `devmonks-co`. The repo is `devmonks-co/ai-toolkit`. Use this everywhere — READMEs, plugin.json, install commands.

## Plugin structure

Plugins live at the repo root (not inside a `plugins/` folder):

```
devmonks-<name>/
├── .claude-plugin/plugin.json
├── README.md
├── LICENSE
├── commands/          ← slash commands (if any)
│   ├── README.md
│   └── <command>.md
└── skills/            ← skills (if any)
    └── <skill>/
        ├── SKILL.md
        └── reference.md (optional)
```

### Naming

- Plugin dirs: `devmonks-<name>` (e.g. `devmonks-git`, `devmonks-diagrams`)
- Command files: `<verb-or-noun>.md` (e.g. `commit.md`, `branch.md`)
- Skill dirs: short name matching the tool (e.g. `d2`)

### plugin.json

```json
{
  "name": "devmonks-<name>",
  "description": "One sentence, no period.",
  "version": "0.1.0",
  "author": { "name": "devmonks" },
  "homepage": "https://github.com/devmonks-co/ai-toolkit",
  "repository": "https://github.com/devmonks-co/ai-toolkit",
  "license": "MIT",
  "keywords": ["relevant", "terms"]
}
```

Bump `version` on every change.

### Command frontmatter

```yaml
---
description: What the command does (one line)
argument-hint: <args> (optional)
allowed-tools: minimal set needed
---
```

### Skill frontmatter

```yaml
---
description: What the skill does (one line)
when_to_use: When Claude should auto-trigger this
argument-hint: "[description]" (optional)
allowed-tools: minimal set needed
---
```

## Plugin README format

Every plugin gets a `README.md` at its root. Use this structure:

```markdown
# devmonks-<name>

One-line description.

## Prerequisites (only if needed)

## Install

From your terminal:

\`\`\`bash
claude plugin install devmonks-<name>@ai-toolkit
\`\`\`

Or from inside a Claude Code session:

\`\`\`
/plugin install devmonks-<name>@ai-toolkit
\`\`\`

## Usage

| Command / Skill | Purpose | Arguments |
|---|---|---|

## License

MIT
```

## Registering a new plugin

1. Create the plugin directory and files.
2. Add an entry to `.claude-plugin/marketplace.json`.
3. Add a row to the plugin table in the root `README.md`.
4. Update the repo structure tree in the root `README.md`.

## Commits

Use conventional commits: `type(scope): subject`. See `/commit` command for full spec. No AI co-author trailers.
