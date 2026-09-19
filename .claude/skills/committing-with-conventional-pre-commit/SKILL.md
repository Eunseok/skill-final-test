---
name: committing-with-conventional-pre-commit
description: Use when writing a git commit message in this repository — read the enforced convention from .pre-commit-config.yaml first, write a compliant message, and self-correct from hook rejections instead of bypassing them
---

# Committing in this repository (conventional-pre-commit)

This repository enforces [Conventional Commits](https://www.conventionalcommits.org)
via the `conventional-pre-commit` hook (Python-based, runs through the `pre-commit`
framework — not commitlint/Node). The hook's config in `.pre-commit-config.yaml` is
the contract: read it before writing a commit message, and trust the error output
when a commit is rejected.

## 1. Detect the enforced rules

Read `.pre-commit-config.yaml` at the repo root and find the `conventional-pre-commit`
hook entry. Its `args` list is the actual rule set — an empty or absent `args` means
the tool's built-in defaults apply (type is enforced, scope is fully optional).

```yaml
repos:
  - repo: https://github.com/compilerla/conventional-pre-commit
    rev: <tag>
    hooks:
      - id: conventional-pre-commit
        stages: [commit-msg]
        args: []
```

This repo's backend and client live in separate repositories, so a scope no longer
distinguishes "which repo" — that's already implied by which repo you're in. Scope is
therefore NOT enforced here (no `--force-scope`, no `--scopes` allow-list): use one when
it adds clarity within this repo (e.g. `feat(entity): ...`), skip it when the type +
description already say enough (e.g. `chore: update gitignore`).

Relevant `args` flags and what they control (for reference, if a future config change
turns any of these on — check the actual file, don't assume):

| Flag                | What it controls                                          |
| -------------------- | ---------------------------------------------------------- |
| (no args)            | Default types only: build chore ci docs feat fix perf refactor revert style test |
| `--force-scope`      | A scope in parentheses becomes required, not optional      |
| `--scopes=a,b,c`     | Only these scope values are allowed                        |
| positional type list | Overrides/extends the allowed type list                    |

If `.pre-commit-config.yaml` isn't found, or has no `conventional-pre-commit` hook,
the repo isn't enforcing this — fall back to the plain Conventional Commits spec
(type + optional scope + `:` + description) without assuming any team-specific scope list.

## 2. This repo's agreed convention

Types in use: `feat`, `fix`, `chore`, `docs`, `refactor`

Scope is optional here, not enforced. When one is useful, suggested values by layer:

- Spring/Java repo: `api`, `entity`, `service`, `config`, `security`
- Unity/C# repo: `ui`, `anim`, `audio`, `unity-meta`
- Domain/feature scopes (once genre is settled) are fine too, and read better in a
  CHANGELOG than layer names — e.g. `feat(shop): ...` over `feat(api): ...` for a
  user-facing change. Neither is required; use whichever helps a reader more.

Breaking changes: append `!` before the colon (`feat(api)!: ...`) and explain the
break in the description; a `BREAKING CHANGE:` footer is only needed for detail
beyond what the `!` + description already conveys.

## 3. Write the message

Format: `type(scope): description`

```
feat(shop): 상점 아이템 목록 조회 API 추가
fix(entity): 연관관계 매핑 오류 수정
chore(unity-meta): .meta 파일 정리
```

- Description in the language the team actually writes commits in (Korean is fine —
  Conventional Commits doesn't require English).
- No period at the end of the description.
- Body (if any): one blank line after the description, then free-form paragraphs.
- Footers (if any): one blank line after the body, `Token: value` or `Token #value`,
  e.g. `Refs: #123`.

## 4. Validate before committing

There's no non-interactive "dry run" flag on the hook itself — the honest check is
running it as part of a real commit attempt. If uncertain, check the message
manually against the type list in section 2 before running `git commit`.

## 5. Split unrelated changes before committing

A commit should represent one kind of change. Before writing a commit message,
check what's actually staged (`git diff --staged`, or `git status` if nothing is
staged yet). If the changes mix things that belong to different types or scopes —
e.g. a new endpoint plus an unrelated typo fix plus a `.gitignore` tweak — split
them into separate commits rather than writing one message that tries to cover
everything.

How to split:

1. Unstage everything if it's already staged in one lump: `git restore --staged .`
2. Stage one logical group at a time. Use `git add <specific files>` when whole
   files map cleanly to one change, or `git add -p` to interactively pick hunks
   within a file when a single file mixes unrelated edits.
3. Commit that group with its own conventional message.
4. Repeat for the next group until everything committed is intentional.

When NOT to split: a `feat` commit's own supporting changes (e.g. the entity plus
the repository plus the service method it needs) are one logical unit — don't
fragment a single feature into commits so small that none of them build or make
sense alone. The bar is "does this commit represent one coherent, reviewable
change", not "one file per commit".

If asked to just "commit this" and the staged/working changes clearly mix
unrelated concerns, propose the split (which files/hunks go where, with what
message) before running any `git commit` — don't silently pick one and commit
everything under it.

## 6. Self-correct when the hook rejects

Rejection output names the problem plainly, e.g.:

```
Conventional Commit......................................................Failed
[Bad Commit message] >> add a new feature

Your commit message does not follow Conventional Commits formatting
...
Conventional Commits start with one of the below types, followed by a colon...
```

- Fix only what the message flags (missing type, disallowed scope, wrong format) —
  don't rewrite unrelated parts of the message.
- Retry the commit with the corrected message.
- NEVER use `git commit --no-verify` to bypass the hook — fixing the message is
  always the correct action, and bypassing defeats the whole point of enforcing
  this for the team.
