---
description: Create a new local branch from HEAD and push it with upstream tracking
argument-hint: <type>/<slug> OR a short description
allowed-tools: Bash(git status:*), Bash(git rev-parse:*), Bash(git branch:*), Bash(git ls-remote:*), Bash(git checkout:*), Bash(git push:*), Bash(git log:*), Read
---

## State

- !`git status --porcelain=v1 -b`
- !`git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "(detached)"`
- !`git log -n 5 --oneline 2>/dev/null || echo "(no commits yet)"`

---

# /branch

Create a new local branch from current `HEAD` and push it with upstream tracking. Accepts either a well-formed `<type>/<slug>` name or a free-text description; in the free-text case, propose a name and wait for approval.

**Arguments:** `$ARGUMENTS`

## Workflow

1. **Read the state above.** If `$ARGUMENTS` is empty, refuse with a one-line usage hint (`/branch <type>/<slug>` or `/branch <short description>`) and stop.
2. **Refuse** (one-line reason + next action) if the repo is mid-merge / mid-rebase / mid-cherry-pick / mid-revert, tracked files contain conflict markers, or the repo has no commits yet.
3. **Classify the input.**
   - If `$ARGUMENTS` starts with a known type followed by `/` (see **Allowed types**), treat the whole string as a verbatim branch name and validate the slug portion per **Slug rules**.
   - Otherwise, treat it as a free-text description. Infer the type per **Type inference** and build the slug per **Slug rules**, yielding `<type>/<slug>`.
4. **Uniqueness checks.** Run `git branch --list <name>` and `git ls-remote --heads origin <name>`. Refuse if either returns a hit — never overwrite.
5. **Present the plan** using the template below and wait for explicit approval. If the user asks for edits (`use fix`, `shorter slug`, `different name`), revise and re-present. Do not execute until approved.
6. **Execute on approval.**
   - `git checkout -b <name>` (uncommitted changes carry over — never stash for the user)
   - `git push -u origin <name>`
7. **Report** a one-line summary: branch created, upstream set, short HEAD sha.

## Allowed types

`feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert` — same set `/commit` uses.

## Type inference (free-text input only)

Match on the leading verb, case-insensitive. If nothing matches, ask the user to pick a type rather than guessing.

- `add`, `build`, `create`, `introduce`, `implement`, `ship` → `feat`
- `fix`, `repair`, `resolve`, `patch`, `correct` → `fix`
- `update`, `refactor`, `rename`, `rework`, `restructure`, `clean` → `refactor`
- `doc`, `docs`, `document`, `write` → `docs`
- `test`, `cover` → `test`
- `perf`, `optimize`, `speed up` → `perf`
- `chore`, `bump`, `upgrade` → `chore`

## Slug rules

- Lowercase, `a-z0-9-` only (non-ASCII stripped)
- Drop stopwords: `a`, `an`, `the`, `to`, `for`, `of`, `in`, `on`, `and`, `or`
- Collapse hyphens, no leading/trailing hyphens
- Max 50 chars (truncate at the last hyphen before the cap)

## Refuse rules — explicit

Never:

- Stash uncommitted changes on the user's behalf
- Run `git fetch` or `git pull` before creating the branch
- Create a branch from anywhere other than current `HEAD`
- Overwrite an existing branch (local or remote)
- Force-push under any circumstance
- Add AI attribution anywhere

## Plan presentation template

```
Proposed branch:

  name:   <type>/<slug>
  from:   <short sha> (<current branch>)
  push:   origin/<name> (new, tracking)

Approve, reject, or tell me what to change.
```

## Examples

Input: `/branch feat/add-user-auth`

```
Proposed branch:

  name:   feat/add-user-auth
  from:   a3f1c9d (main)
  push:   origin/feat/add-user-auth (new, tracking)

Approve, reject, or tell me what to change.
```

Input: `/branch fix login redirect loop`

```
Proposed branch:

  name:   fix/login-redirect-loop
  from:   a3f1c9d (main)
  push:   origin/fix/login-redirect-loop (new, tracking)

Approve, reject, or tell me what to change.
```

