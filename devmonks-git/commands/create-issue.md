---
description: Create a GitHub issue on a specified repo with a type label and description
argument-hint: [--repo <owner/repo>] [--type <bug|feat|task|docs>] <title or description>
allowed-tools: Bash(gh auth status:*), Bash(gh issue create:*), Bash(gh label list:*), Bash(gh repo view:*), Bash(git rev-parse:*), Bash(git remote:*), Read, Write
---

## State

- !`gh auth status 2>&1 | tail -n 5`
- !`git remote get-url origin 2>/dev/null || echo "(no git remote)"`
- !`gh repo view --json nameWithOwner -q .nameWithOwner 2>/dev/null || echo "(not a gh repo)"`

---

# /create-issue

Create a GitHub issue from a short description. Draft everything from what the user gives you — do not interrogate them upfront. Present the plan, let them edit inline, then execute on approval.

**Arguments:** `$ARGUMENTS`

- `--repo <owner/repo>` — target repo. If omitted, infer from git remote. If that fails, ask once.
- `--type <bug|feat|task|docs>` — issue type. If omitted, infer from description. If ambiguous, ask once.
- Everything else is the issue title/description.

## Workflow

1. **Check auth.** If `gh auth status` reports unauthenticated, refuse: tell user to run `gh auth login` and stop.
2. **Parse `$ARGUMENTS`.** Extract `--repo`, `--type`, remaining text as raw description.
3. **Resolve repo.**
   - Use `--repo` if given.
   - Otherwise use `nameWithOwner` from state.
   - Otherwise ask once: `Target repo? (e.g. owner/repo)` — do not proceed without it.
4. **Resolve type.** Use `--type` if given. Otherwise infer per **Type inference**. If genuinely ambiguous (no keywords match), ask once: `Issue type? bug / feat / task / docs`.
5. **Draft the title.** Use the raw description verbatim if ≤72 chars. Otherwise summarise to ≤72 chars imperative.
6. **Draft the body.** Use the template for the resolved type (see **Body templates**). Fill every field you can from the description. For fields you cannot fill, write a short italicised placeholder like `_add detail here_` — do NOT ask the user for them before presenting the plan.
7. **Resolve label.** Run `gh label list --repo <repo>`. Map type per **Label mapping**. If label missing, note it — do not ask.
8. **Present the plan** (see **Plan template**) and wait for approval. Accept inline edits to any field. Do not execute until approved.
9. **Execute on approval.** Write body to temp file with `Write`. Run `gh issue create --repo <repo> --title <title> --body-file <tmpfile> [--label <label>]`. Delete temp file. Report the issue URL. On failure print `gh`'s stderr and stop.

## Type inference

First match wins. If nothing matches at all, ask once.

- `crash`, `broken`, `error`, `fail`, `exception`, `not working`, `bug` → `bug`
- `add`, `implement`, `support`, `introduce`, `build`, `new`, `feature` → `feat`
- `doc`, `docs`, `readme`, `guide`, `document` → `docs`
- anything else → `task`

## Body templates

Keep bodies short. One sentence per field. Use `_add detail here_` as placeholder for unknowns — never leave a field blank, never ask before drafting.

### bug

```
## Problem

<one sentence from the description, or _add detail here_>

## Expected outcome

<one sentence, or _add detail here_>

## Steps to reproduce

1. <inferred from description, or _add step_>
2. <inferred from description, or _add step_>

## Environment

- Platform: <inferred or _web / mobile / desktop_>
- App version: <inferred or _add version_>
- OS / Browser: <inferred or _add OS and browser_>
```

### feat

```
## Problem / Goal

<one sentence from the description, or _add detail here_>

## Expected outcome

<one sentence, or _add detail here_>
```

### task

```
## Brief

<one sentence from the description, or _add detail here_>

## Scope

<one sentence, or _add detail here_>

## Deliverables

- [ ] <inferred from description, or _add deliverable_>
```

### docs

```
## Problem

<one sentence from the description, or _add detail here_>

## Expected outcome

<one sentence, or _add detail here_>
```

## Label mapping

| Type   | Label name      |
|--------|-----------------|
| `bug`  | `bug`           |
| `feat` | `enhancement`   |
| `task` | `task`          |
| `docs` | `documentation` |

## Refuse rules

Never:

- Execute without explicit approval
- Ask multiple questions before presenting the plan — draft first, ask never or once
- Create labels that do not exist in the repo
- Add assignees, milestones, or projects unless the user asks
- Retry a failed `gh issue create` — print the error and stop
- Add AI attribution anywhere in title or body

## Plan template

```
Proposed issue:

  repo:   <owner/repo>
  type:   <type>
  title:  <title>
  label:  <label> (exists) | (not found — will be omitted)

  body:
    <body preview, indented>

Approve, reject, or tell me what to change.
```
