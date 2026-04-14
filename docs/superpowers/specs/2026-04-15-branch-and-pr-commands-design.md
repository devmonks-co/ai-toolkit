# `/branch` and `/pr` slash commands — design

**Status:** draft
**Date:** 2026-04-15
**Scope:**
- `plugins/devmonks-core/commands/branch.md`
- `plugins/devmonks-core/commands/pr.md`
- `plugins/devmonks-core/commands/README.md`

## Purpose

Two slash commands that take the friction out of the "start work → ship work" loop:

- `/branch` turns a short description or a well-formed branch name into a new local branch cut from current `HEAD`, pushed with upstream tracking.
- `/pr` turns a ready-to-ship branch into a GitHub pull request with a derived title, a concise body, and an explicit base branch — via the `gh` CLI.

Both commands follow the same propose → approve → execute pattern as `/commit`. They never drive git state machines (no stash, no rebase, no amend, no force-push) and never attribute the AI as a co-author.

## Non-goals

- Interactive per-step approval — one approval per run covers the whole plan.
- Fixing repository state the user didn't ask to be fixed (merges, rebases, conflicts, uncommitted changes).
- Ticket/issue linking, labels, reviewers, assignees — v1 scope leaves these to the user.
- Running tests, formatters, or type checks proactively — that is pre-commit's job. Commands react to hook failures, they don't pre-empt them.
- Reopening closed PRs or updating existing ones — v1 is create-only.
- Auto-fetch, auto-pull, auto-stash — all refused on principle.
- Single-command convenience (`/pr` that also creates the branch). Explicit separation mirrors `/commit`.

## Dependencies

- `git` ≥ 2.30
- `gh` CLI, authenticated against the target remote (`/pr` only)
- Shell tools assumed by Claude Code (bash, standard POSIX utilities)

The command markdown files declare a narrow `allowed-tools` list so approvals are predictable.

---

## `/branch`

### Purpose
Create a new local branch from current `HEAD` and push it with upstream tracking. Accepts either a well-formed `<type>/<slug>` branch name or a free-text description; in the free-text case, the command proposes a name and waits for approval before creating anything.

### Input

`/branch $ARGUMENTS`

Two shapes are recognized:

1. **Verbatim branch name** — starts with a known type prefix followed by `/` (e.g., `feat/add-user-auth`, `fix/login-redirect-loop`). The command validates the slug and uses the name as-is.
2. **Free-text description** — anything else (e.g., `add user auth`, `fix login redirect loop`). The command infers a type from the verbs, slugifies the rest, and proposes `<type>/<slug>` for approval.

If `$ARGUMENTS` is empty, refuse with a one-line usage hint.

### Allowed types

`feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert` — the same set `/commit` uses. No custom types in v1.

### Type inference rules (free-text input only)

Matched on the leading verb of the description, case-insensitive:

- `add`, `build`, `create`, `introduce`, `implement`, `ship` → `feat`
- `fix`, `repair`, `resolve`, `patch`, `correct` → `fix`
- `update`, `refactor`, `rename`, `rework`, `restructure`, `clean` → `refactor`
- `doc`, `docs`, `document`, `write` → `docs`
- `test`, `cover` → `test`
- `perf`, `optimize`, `speed up` → `perf`
- `chore`, `bump`, `upgrade` → `chore`

If no rule matches, the command asks the user to pick a type rather than guessing.

### Slug rules

- Lowercase
- `a-z0-9-` only (non-ASCII transliterated or dropped)
- Stopwords stripped: `a`, `an`, `the`, `to`, `for`, `of`, `in`, `on`, `and`, `or`
- Multiple hyphens collapsed to one, no leading/trailing hyphen
- Max length 50 characters (truncate at the last hyphen before the limit)

### Workflow

1. **Inspect state.** Run `git status --porcelain=v1 -b`, `git rev-parse --abbrev-ref HEAD`, `git branch --list <proposed>`, `git ls-remote --heads origin <proposed>`.
2. **Refuse** on any of the following, with a one-line reason and the next action the user should take:
   - Mid-merge, mid-rebase, mid-cherry-pick, mid-revert (visible in `git status` banners)
   - Tracked files contain conflict markers
   - Proposed branch already exists locally
   - Proposed branch already exists on `origin`
   - Slug fails validation
   - `$ARGUMENTS` is empty
3. **Draft the name.** If input is a verbatim `<type>/<slug>`, validate it. Otherwise, run inference + slug rules to build `<type>/<slug>`.
4. **Present the plan** (see template below) and wait for explicit approval. If the user asks for edits (`use fix instead`, `shorter slug`, etc.), revise and re-present — do not execute until approved.
5. **Execute on approval:**
   1. `git checkout -b <name>` — uncommitted changes carry over, which is normal `checkout -b` behavior. The command does not stash.
   2. `git push -u origin <name>` — sets upstream tracking.
6. **Report** a one-line summary: branch created, tracking set, ready.

### Refuse rules — explicit

The command NEVER:

- Stashes uncommitted changes on the user's behalf
- Runs `git fetch` or `git pull` before creating the branch
- Creates a branch from anywhere other than current `HEAD`
- Overwrites an existing branch (local or remote)
- Force-pushes under any circumstance

### Plan presentation template

```
Proposed branch:

  name:   <type>/<slug>
  from:   <current HEAD short sha> (<current branch name>)
  push:   origin/<name> (new, tracking)

Approve, reject, or tell me what to change.
```

### Examples

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

Input: `/branch refactor the whole auth module for clarity`
```
Proposed branch:

  name:   refactor/whole-auth-module-clarity
  from:   a3f1c9d (main)
  push:   origin/refactor/whole-auth-module-clarity (new, tracking)

Approve, reject, or tell me what to change.
```

---

## `/pr`

### Purpose
Open a GitHub pull request from the current branch against an explicit base, via `gh pr create`. Derives the title from the first commit subject on the branch. Fills the body from `.github/PULL_REQUEST_TEMPLATE.md` if present, otherwise a minimal Summary + Test plan. Never draft by default.

### Input

`/pr $ARGUMENTS`

Supported flags:

- `--base <branch>` — skip the base-branch prompt and use this remote ref (e.g., `--base main`)
- `--draft` — open the PR as a draft (opt-in only; never the default)
- `--title <text>` — override the derived title entirely
- (Anything else is ignored with a warning — no silent acceptance)

All flags are optional. Running `/pr` with no arguments is the normal path.

### Preconditions (refuse if violated)

- Current branch is not `main`, `master`, or `dev`
- Working tree is clean (no uncommitted tracked changes, no staged changes)
- Branch has an upstream configured and is in sync with it. If ahead (unpushed commits) → refuse, tell the user to `git push`. If behind → refuse, tell the user to pull. The command never pushes on the user's behalf.
- `gh` CLI is installed and `gh auth status` reports authenticated
- No existing open PR from this branch — if one exists, print its URL and stop (no create, no update)
- Mid-merge / mid-rebase / conflict markers → refuse, same as `/commit`

### Workflow

1. **Inspect state.**
   - `git status --porcelain=v1 -b`
   - `git rev-parse --abbrev-ref HEAD`
   - `git rev-parse --abbrev-ref --symbolic-full-name @{u}` (upstream check)
   - `git rev-list --left-right --count @{u}...HEAD` (ahead/behind check)
   - `gh auth status`
   - `gh pr list --head <branch> --state open --json url,number`
   - Check for `.github/PULL_REQUEST_TEMPLATE.md` (also `docs/pull_request_template.md`, `.github/pull_request_template.md` — the locations `gh` honors)
2. **Refuse** on any precondition violation with a one-line reason.
3. **Resolve the base branch.**
   - If `--base <branch>` was passed, use it verbatim.
   - Otherwise, prompt the user explicitly. Show `dev` and `main` as hints if they exist on the remote. Accept any valid remote ref.
   - Run `git fetch origin <base>` once the base is known — only then, and only to make the PR diff accurate.
4. **Draft the title.**
   - If `--title` was passed, use it verbatim.
   - Otherwise take the first commit subject on the branch: `git log <base>..HEAD --reverse --format=%s | head -1`. Use it verbatim if it matches conventional commits shape (`type(scope): subject` or `type: subject`). If not, show it and let the user edit before approving.
   - Hard cap 72 chars (100 absolute). Imperative mood. No ticket refs.
5. **Draft the body.**
   - If a template file exists, load it and fill the first two sections it contains:
     - Any heading matching `/summary/i` → populate with 1–3 bullets derived from commit subjects between `<base>..HEAD`
     - Any heading matching `/test plan|testing/i` → populate with 2–4 checkbox lines (`- [ ] <action>`). Derive from changed paths where possible; otherwise leave placeholders.
     - Leave every other template section untouched (user fills them).
   - If no template exists, use this fallback:
     ```
     ## Summary

     - <bullet 1>
     - <bullet 2>

     ## Test plan

     - [ ] <action 1>
     - [ ] <action 2>
     ```
6. **Present the plan** (see template below) and wait for explicit approval. Accept edits like `change base to main`, `shorter title`, `make it draft`, `add bullet about X`.
7. **Execute on approval:** `gh pr create --base <base> --head <branch> --title <title> --body-file <tmp>` (plus `--draft` if requested). Write the body to a temp file so multiline content round-trips cleanly.
8. **Report** the PR URL on success. On failure, print `gh`'s stderr verbatim and stop — no retry loop.

### Refuse rules — explicit

The command NEVER:

- Pushes the current branch for the user (refuses if not up to date; user runs `git push` themselves, then re-runs `/pr`)
- Force-pushes or rebases
- Adds reviewers, labels, assignees, or milestones
- Updates or reopens an existing PR
- Accepts `--base` pointing at the current branch
- Adds AI co-authorship anywhere in the title or body
- Writes a `Co-Authored-By: Claude ...` trailer to any commit

### Plan presentation template

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

### Examples

Branch `feat/add-user-auth`, no template, `/pr` (no args):
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
    - [ ] Existing session users are not forced to re-auth

Approve, reject, or tell me what to change.
```

Branch `fix/login-redirect-loop`, template exists, `/pr --base main --draft`:
```
Proposed PR:

  base:    main
  head:    fix/login-redirect-loop (9c12aa1, 1 commit ahead)
  title:   fix: stop redirect loop on expired session
  draft:   yes

  body:
    ## Summary

    - Stop redirecting expired sessions through /login when the next
      param already points at /login

    ## Test plan

    - [ ] Expire a session, confirm no redirect loop

    ## Risk

    (user fills)

Approve, reject, or tell me what to change.
```

---

## Shared principles (mirrors `/commit`)

- **Propose → approve → execute.** Never act before the user sees and okays the plan.
- **Refuse, don't fix.** Unusual git state is the user's call, not the command's.
- **No AI attribution.** Never in branch names, commit messages, PR titles, or PR bodies. Strip `Co-Authored-By: Claude ...` if the environment injects it.
- **Terse output.** One line per step, no narration.
- **Bounded retries only.** On hook or `gh` failure, print the error and stop. No exponential backoff, no second guesses.

## Commands README

A new file at `plugins/devmonks-core/commands/README.md` documents every command in the plugin:

- Name, one-line purpose, supported arguments, flags, and a minimal example
- Table rows, not long prose — think `man`-page summary, not a tutorial
- Linked from the main repo `README.md` under a future "What's shipped" section

Initial rows: `/commit`, `/branch`, `/pr`. New commands add a row to the same table as they land.

## Future (not v1)

- `/pr update` to amend an existing PR's title/body/base
- Issue/ticket linking via `--issue <id>` (`gh pr create --body` can take `Closes #123` trailers)
- `--reviewers`, `--labels`, `--assignees` flags
- A `/ship` command that composes `/commit` → `/branch` (if on main) → `/pr` behind one approval
- Author-prefix branch naming (`saiful/feat/...`) if the team ever needs ownership namespacing on shared repos
