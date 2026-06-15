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

Create a focused GitHub issue. No assumptions — if information needed to fill the body is missing, ask before proposing. Propose a plan, get one approval, execute.

**Arguments:** `$ARGUMENTS`

- `--repo <owner/repo>` — target repo. If omitted, infer from git remote. If inference fails, ask.
- `--type <bug|feat|task|docs>` — issue type. If omitted, infer from description or ask.
- Everything else is the issue title/description.

## Workflow

1. **Check auth.** If `gh auth status` reports unauthenticated, refuse: tell the user to run `gh auth login` and stop.
2. **Parse `$ARGUMENTS`.** Extract `--repo`, `--type`, and the remaining text as the raw input.
3. **Resolve the repo.**
   - Use `--repo` if given.
   - Otherwise use the `nameWithOwner` from state if present.
   - Otherwise ask — do not proceed without a confirmed repo.
   - Verify with `gh repo view <repo>`. If it fails, refuse with the error and stop.
4. **Resolve the type.** Use `--type` if given. Otherwise infer per **Type inference**. If ambiguous, ask: `Issue type? bug / feat / task / docs` — do not guess.
5. **Draft the title.** ≤72 chars, imperative, lowercase after the colon. If the raw input is already a clean title, use it verbatim. Otherwise derive one and show it for confirmation.
6. **Check for missing info.** Before drafting the body, identify what the body template requires (see **Body templates**) and what the user has not provided. Ask for any missing required fields in a single grouped question — do not assume or invent values.
7. **Draft the body** using the template for the resolved type (see **Body templates**). Keep every line short and factual. No filler, no padding.
8. **Resolve the label.** Run `gh label list --repo <repo>`. Map type to label per **Label mapping**. If the label does not exist, note it in the plan — do not create it.
9. **Present the plan** (see **Plan template**) and wait for explicit approval. Accept edits. Do not execute until approved.
10. **Execute on approval.** Write the body to a temp file with `Write`. Run `gh issue create --repo <repo> --title <title> --body-file <tmpfile> [--label <label>]`. Delete the temp file. Report the issue URL. On failure, print `gh`'s stderr and stop.

## Type inference

First match wins. If nothing matches, ask.

- `crash`, `broken`, `error`, `fail`, `exception`, `not working`, `bug` → `bug`
- `add`, `implement`, `support`, `introduce`, `build`, `new`, `feature` → `feat`
- `doc`, `docs`, `readme`, `guide`, `document` → `docs`
- anything else → `task`

## Body templates

Use exactly the template for the resolved type. Do not add extra sections. Do not fill a field with a placeholder if the value is unknown — ask the user instead.

### bug

```
## Problem

<one sentence: what is broken>

## Expected outcome

<one sentence: what should happen instead>

## Steps to reproduce

1. <step>
2. <step>
3. <step>

## Environment

- Platform: <web | mobile | desktop>
- App version: <e.g. 1.4.2>
- OS / Browser: <e.g. iOS 17 / Chrome 124>
```

Required fields: problem, expected outcome, at least 2 reproduce steps, platform, app version, OS/Browser. Ask for any that are missing.

### feat

```
## Problem / Goal

<one sentence: what need or gap this addresses>

## Expected outcome

<one sentence: what done looks like>
```

Required fields: problem/goal, expected outcome. Ask for any that are missing.

### task

```
## Brief

<one sentence: what this task is>

## Scope

<one sentence: what is in scope; optionally one sentence on what is explicitly out>

## Deliverables

- [ ] <item>
- [ ] <item>
```

Required fields: brief, scope, at least one deliverable. Ask for any that are missing.

### docs

```
## Problem

<one sentence: what is missing or wrong in the docs>

## Expected outcome

<one sentence: what the docs should cover or fix>
```

Required fields: problem, expected outcome. Ask for any that are missing.

## Label mapping

| Type   | Label name       |
|--------|------------------|
| `bug`  | `bug`            |
| `feat` | `enhancement`    |
| `task` | `task`           |
| `docs` | `documentation`  |

## Refuse rules

Never:

- Execute without explicit approval
- Assume or invent values for missing body fields — ask instead
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
