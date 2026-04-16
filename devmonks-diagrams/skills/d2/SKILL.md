---
description: Generates diagrams using D2 — system architecture, ERD, flowchart, sequence, class diagrams. Compiles to SVG/PNG with icon URL validation.
when_to_use: Triggers on requests to create, draw, visualize, or diagram system components, database schemas, flows, sequences, or class hierarchies. Example requests — "draw the system architecture", "create an ERD for these tables", "diagram the auth flow".
argument-hint: "[diagram description]"
allowed-tools: Bash Read Write Glob WebFetch
---

## State

- !`which d2 2>/dev/null && d2 --version 2>/dev/null || echo "D2_NOT_INSTALLED"`

---

# D2 Diagram Generation

Generates `.d2` diagram files and compiles them to SVG/PNG using the D2 CLI.

**Full syntax reference:** `${CLAUDE_SKILL_DIR}/reference.md` — read it before writing any `.d2` file.

**Official D2 docs:** https://d2lang.com/tour/intro

## Checklist

- [ ] D2 CLI is available (or tell user to install)
- [ ] Determine diagram type and read relevant section from reference
- [ ] Write `.d2` file
- [ ] Validate all icon URLs (curl check for non-2xx)
- [ ] Compile with `d2` and verify output
- [ ] Report output path to user

## Design Principles

Follow these rules to produce clean, professional diagrams. Read the "Design best practices" section in the reference for full details.

1. **Use ELK for anything with containers.** Set `layout-engine: elk` in `d2-config` whenever the diagram has nested groups. Only use dagre for flat graphs with no nesting.
2. **Max 2 levels of nesting.** Deeper nesting creates sprawl. Flatten by grouping peers at the same level.
3. **Never use `shape: cloud` as a container.** It bloats into an unreadable blob. Use a rectangle container with a dashed border and a label like "AWS" instead.
4. **Use classes for styling.** Define 2–3 classes (e.g., `client`, `service`, `storage`) and apply them. Avoid per-node inline styles.
5. **Keep connection labels short.** 1–2 words max (e.g., "HTTPS", "read/write", "jobs"). Long labels clutter the layout.
6. **Keep node labels short.** Remove redundant suffixes — "S3" not "S3 Bucket", "RDS" not "RDS Database" — unless the short name is ambiguous.
7. **Use `style.shadow: true` on leaf nodes** for depth. Use `style.border-radius: 8` on containers for a softer look.
8. **Use theme-id 3** (flagship) as the default. It produces clean, professional colors.

## Workflow

1. **Check D2 is installed.** If state shows `D2_NOT_INSTALLED`, tell the user: `brew install d2` (macOS) or `curl -fsSL https://d2lang.com/install.sh | sh -`. Stop.
2. **Read the reference.** Load `${CLAUDE_SKILL_DIR}/reference.md` — read the section matching the requested diagram type AND the "Design best practices" section.
3. **Plan the structure.** Before writing any D2, sketch the node hierarchy mentally:
   - Identify logical groups (max 2 nesting levels)
   - Choose layout engine (ELK for containers, dagre for flat)
   - Define 2–3 style classes
   - Map the connection flow left-to-right or top-to-bottom
4. **Write the `.d2` file** in the current working directory (or user-specified path). Use the syntax patterns from the reference. Start with `vars` / `classes` blocks, then nodes, then connections.
5. **Resolve icon URLs.** Read `${CLAUDE_SKILL_DIR}/icons.md` for the full resolution guide. Follow this priority chain for each icon:
   1. **Terrastruct** (`https://icons.terrastruct.com/`) — construct a URL from the category patterns in icons.md and validate
   2. **jsdelivr** (`https://cdn.jsdelivr.net/npm/@mdi/svg/svg/`) — good for generic icons (lock, email, database, etc.)
   3. **Official provider logo** — check the provider's brand/press page or `simple-icons`
   4. **Web search** — last resort, search for `<name> SVG icon`
   - **Validate every icon URL** by running: `curl -sL -o /dev/null -w '%{http_code}' "<url>"`
   - If non-2xx: move to the next source in the chain. If all fail, omit the icon and note it to the user.
6. **Compile.** Run `d2 <name>.d2 <name>.svg`. On failure: read the error, fix the `.d2` source, retry once. If it still fails, show the error and stop.
7. **Report.** Output file path, diagram type, and any icon issues found.

## Refuse

- If the diagram type cannot be expressed in D2, say so and suggest an alternative.
- Do not fabricate icon URLs. Follow the resolution priority chain in `icons.md`, and always validate with curl before compiling.
