---
description: Create atomic conventional commits from the current working tree
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git reset:*), Bash(git commit:*), Read, Grep, Glob
---

## State

- !`git status`
- !`git log -n 20 --oneline 2>/dev/null || echo "(no commits yet)"`

---

# /commit

Turn the current working tree into one or more atomic conventional commits. Propose a plan, get one approval, execute in order. Never attribute the AI as a co-author. Takes no arguments.

## Workflow

1. **Read the state above.** For file-level detail use `git diff -- <path>` or `git diff --staged -- <path>`.
2. **Refuse** (one-line reason + next action) if: no changes, the repo is mid-merge / mid-rebase / mid-cherry-pick / mid-revert (visible in `git status` banners), or tracked files contain conflict markers. Never drive git state machines on the user's behalf.
3. **Group atomically.** Partition the full tree — modifications, deletions, untracked files — into cohesive groups. File-level by default; escalate to hunk-level only when one file clearly mixes unrelated concerns. Co-located siblings (module+test, migration+seed, component+style) stay together. Ignore any pre-existing staging and re-group from scratch so the plan is reproducible.
4. **Draft each commit** per **Message format** below. Refuse to stage secret-shaped files (`.env*`, `*credentials*`, `*.pem`, `*.key`, `id_rsa*`, anything that looks like a live token); flag them and ask.
5. **Present the plan** using the template below and wait for explicit approval. If the user asks for edits, revise and re-present — do not execute until approved.
6. **Execute on approval.** For each group in order: `git reset` the index, stage only that group (`git add -- <paths>`, or `git add -p` for hunk-level), `git commit` via heredoc, `git status` to verify. Never `--amend`. Never `--no-verify`. Always new commits.
7. **Hook failure → bounded retry.** If a pre-commit hook rejects: read its output, attempt **one** auto-fix scoped strictly to files in the current group, re-stage, retry once. If the fix would touch any file outside the group, stop instead. If the retry still fails, stop cleanly — prior commits stay, the current group is left unstaged, later groups are untouched — and report exactly what happened.
8. **Report** a short summary: commits created (list subjects), groups skipped, files left uncommitted, secret-shaped files flagged.

## Message format

**Subject** — `type(scope): subject`, `type: subject`, or `type(scope)!: subject` for breaking changes.
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- Scope optional — derive it from the actual repo: the directory or module the changed files live in, matched against the scope style already used in recent `git log` entries above. Omit the scope on cross-cutting or trivial changes. Never invent filler scopes like `(repo)` or `(misc)`.
- ≤72 chars (hard cap 100). Imperative mood. Lowercase after the colon. No trailing period. No ticket or issue refs.

**Body** — always present, blank line after subject, one or two sentences (more only when genuinely needed), wrap at 72. Answers *why*, not *what*. No references to the conversation, PR, or reviewer. No AI self-attribution.

**Trailers** — never emit `Co-Authored-By: Claude ...` or any AI trailer; strip it if the environment injects one. `Signed-off-by:` only if recent `git log` shows the repo already uses DCO. No others.

**Breaking change** — both markers, always: `!` after the type/scope in the subject **and** a `BREAKING CHANGE:` footer in the body explaining impact and migration. Never one without the other.

**Verify before every `git commit`:** subject constraints hold, body is present and why-focused, no forbidden trailers, no secrets staged, the group is one logical change, breaking markers match.

## Examples

```
fix(commands): pin resolver path for /commit

The default resolver started returning absolute paths on macOS after
the 2026-Q1 update, which broke relative plan output.
```

```
feat(api)!: drop legacy /v1/users endpoint

Clients have been on /v2 since 2025-11 and the shim is the top source
of 500s in prod.

BREAKING CHANGE: /v1/users is removed. Migrate to /v2/users; response
shape is identical except `created_at` is ISO-8601.
```

## Plan presentation template

```
Proposed commits (N):

1. type(scope): subject
   files: path/a, path/b
   why: one-sentence body preview

2. type: subject
   files: path/c
   why: one-sentence body preview

Approve all, reject, or tell me what to change.
```

For hunk-level groups, use `path/x (hunks)` in the `files` line.
