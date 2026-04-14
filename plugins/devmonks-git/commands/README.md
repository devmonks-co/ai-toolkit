# devmonks-git commands

Slash commands shipped by the `devmonks-git` plugin. Each one turns a repetitive git or GitHub workflow into a propose → approve → execute loop.

## Shipped

| Command | Purpose | Arguments |
|---|---|---|
| [`/commit`](commit.md) | Turn the working tree into one or more atomic conventional commits | _(none)_ |
| [`/branch`](branch.md) | Create a branch from current `HEAD` and push it with upstream tracking | `<type>/<slug>` or a short description |
| [`/create-pr`](create-pr.md) | Open a PR from the current branch via the `gh` CLI | `[--base <branch>] [--draft] [--title <text>]` |

## Shared principles

All commands follow the same contract:

- **Propose → approve → execute.** Draft a plan, wait for approval, then touch state.
- **Refuse, don't fix.** Unusual git state (merges, rebases, conflicts, dirty trees) is the user's call. Stop and report.
- **No AI attribution.** Never emit `Co-Authored-By: Claude ...` in commits, branches, or PR content. Strip it if injected.
- **Terse output.** One line per step, no narration.
- **Bounded retries only.** On hook or CLI failure, print the error and stop.

## Adding a new command

1. Drop the markdown file in this directory.
2. Give it a clear `description` and a narrow `allowed-tools` list in the frontmatter.
3. Follow the shared principles above.
4. Add a row to the **Shipped** table.
5. Bump `version` in `plugin.json` so end users see the update (`/plugin marketplace update` uses the version field to detect changes).
