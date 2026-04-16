# devmonks-git

Slash commands for atomic commits, branches, and pull requests.

## Install

From your terminal:

```bash
claude plugin install devmonks-git@ai-toolkit
```

Or from inside a Claude Code session:

```
/plugin install devmonks-git@ai-toolkit
```

## Usage

| Command | Purpose | Arguments |
|---|---|---|
| `/commit` | Turn the working tree into one or more atomic conventional commits | _(none)_ |
| `/branch` | Create a branch from current `HEAD` and push it with upstream tracking | `<type>/<slug>` or a short description |
| `/create-pr` | Open a PR from the current branch via the `gh` CLI | `[--base <branch>] [--draft] [--title <text>]` |

All commands follow a **propose -> approve -> execute** loop. See [commands/README.md](commands/README.md) for shared principles and how to add new commands.

## License

MIT
