---
description: Open a PR from the current branch against a chosen base via the gh CLI
argument-hint: [--base <branch>] [--draft] [--title <text>]
allowed-tools: Bash(git status:*), Bash(git rev-parse:*), Bash(git rev-list:*), Bash(git log:*), Bash(git fetch:*), Bash(gh auth status:*), Bash(gh pr list:*), Bash(gh pr create:*), Bash(rm:*), Read, Write
---

## State

- !`git status --porcelain=v1 -b`
- !`git rev-parse --abbrev-ref HEAD 2>/dev/null || echo "(detached)"`
- !`git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "(no upstream)"`
- !`gh auth status 2>&1 | tail -n 5`

---

# /create-pr

Open a GitHub pull request from the current branch via `gh pr create`. Derive the title from the first commit on the branch; write a minimal Summary + Test plan body. Propose a plan, get one approval, execute. Never draft by default.

**Arguments:** `$ARGUMENTS` — all flags optional.

- `--base <branch>` — skip the base-branch prompt
- `--draft` — open as draft
- `--title <text>` — override the derived title

Unknown args are reported and ignored.

## Workflow

1. **Read the state above.** Parse `$ARGUMENTS` for supported flags; collect unknowns to report.
2. **Refuse** (one-line reason + next action) if any of:
   - Current branch is `main`, `master`, or `dev`
   - Working tree is not clean (uncommitted or staged changes)
   - Branch has no upstream configured
   - Branch is ahead of upstream — tell the user to `git push`
   - Branch is behind upstream — tell the user to `git pull`
   - `gh auth status` reports unauthenticated
   - `gh pr list --head <branch> --state open` returns an existing PR — print its URL and stop
   - Mid-merge / mid-rebase / conflict markers are present
3. **Resolve the base branch.**
   - If `--base <branch>` was passed, use it verbatim.
   - Otherwise ask the user explicitly: `Base branch? (dev, main, or type another)`. Accept any valid remote ref.
   - Refuse if the base equals the current branch.
4. **Fetch the base.** `git fetch origin <base>`.
5. **Draft the title.**
   - If `--title` was passed, use it verbatim.
   - Otherwise: `git log <base>..HEAD --reverse --format=%s | head -1` — the first commit subject.
   - If it matches conventional-commits shape (`type: subject` / `type(scope): subject`), use verbatim; otherwise show and offer an edit.
   - ≤72 chars (hard cap 100). Imperative mood. No ticket refs.
6. **Draft the body** — always this minimal shape:
   ```
   ## Summary

   - <1–3 bullets from commit subjects and the diff>

   ## Test plan

   - [ ] <2–4 actionable checks; placeholders if unclear>
   ```
7. **Present the plan** (see template below) and wait for approval. Accept edits (base, title, draft, body bullets). Do not execute until approved.
8. **Execute on approval.**
   - Write the body to a temp file (use the `Write` tool for multiline safety).
   - `gh pr create --base <base> --head <branch> --title <title> --body-file <tmpfile>` (+ `--draft` if requested).
   - Delete the temp file.
9. **Report** the PR URL on success, including any ignored unknown arguments. On failure, print `gh`'s stderr and stop — no retries.

## Refuse rules — explicit

Never:

- Push the current branch for the user (refuse; tell them to `git push`)
- Force-push or rebase
- Add reviewers, labels, assignees, or milestones
- Update, reopen, or close an existing PR
- Accept `--base` pointing at the current branch
- Write AI attribution anywhere in title or body (strip `Co-Authored-By: Claude ...` if injected)

## Plan presentation template

```
Proposed PR:

  base:    <base>
  head:    <current branch> (<short sha>, N commits ahead)
  title:   <title>
  draft:   yes|no

  body:
    <rendered body, indented>

Approve, reject, or tell me what to change.
```

## Example

Branch `feat/add-user-auth`, `/create-pr` (no args):

```
Base branch? (dev, main, or type another)
> dev
```

```
Proposed PR:

  base:    dev
  head:    feat/add-user-auth (3f91a2c, 4 commits ahead)
  title:   feat(auth): add user login
  draft:   no

  body:
    ## Summary

    - Add email/password login backed by the existing session store
    - Wire the login form to the new /auth/login endpoint

    ## Test plan

    - [ ] Log in with a valid credential, land on /dashboard
    - [ ] Log in with a bad password, see inline error

Approve, reject, or tell me what to change.
```
