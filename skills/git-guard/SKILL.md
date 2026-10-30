---
name: git-guard
description: Confirm or block force-push, protected-branch rewrites, secret commits, and remote/branch deletion; summarize diffs before risky git ops.
when-to-use: Before any non-routine git write — especially push --force, branch -D on remotes, amending pushed history, or commits that may include .env / keys. Also when reviewing a staged diff that looks like it contains secrets.
---

# git-guard

Make destructive and secret-leaking git operations impossible to do silently.

## Purpose

Agents often treat git as "just another shell." Force-pushes, protected-branch rewrites, and accidental secret commits are hard to undo. This skill defines what is **routine** (allowlist) versus **gated**, and how to summarize risk before the human decides.

## Allowlist (no card for routine local work)

These are allowed without a confirmation card when they match the user's current request:

- `git status`, `git diff`, `git log`, `git show`, `git blame`
- `git add` of non-secret paths (see Secret paths below)
- `git commit` of already-reviewed, non-secret staged content
- `git branch` (list / create local), `git checkout` / `git switch` to existing local branches
- `git fetch` (read-only remote sync)
- `git pull` / `git rebase` onto the current tracking branch when the user explicitly asked to update — still prefer a short summary of incoming commits

If any allowlisted command would also stage or commit a secret path, **upgrade to gated**.

## Gated operations (confirmation required)

| Operation | Why it is gated |
| --- | --- |
| `git push --force` / `--force-with-lease` | Rewrites remote history; can erase teammates' work |
| Push or amend on a **protected** branch (`main`, `master`, `release/*`, or repo-configured) | Bypasses normal review |
| Commit or stage of secret-like paths | Leaks credentials into history |
| `git push` that deletes a remote branch (`:branch` or `--delete`) | Drops shared refs |
| `git remote remove` / rewrite of remote URLs | Can redirect push/pull to a hostile host |
| `git filter-branch` / `git filter-repo` / history rewrite tools | Irreversible without backups |
| `git reset --hard` / `git clean -fdx` on a dirty tree the user did not explicitly ask to discard | Data loss |

## Secret paths (always gate before add/commit)

Block or require confirm before staging/committing paths matching:

- `.env`, `.env.*`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `*.pfx`
- `credentials.json`, `service-account*.json`, `*secret*`, `*token*` (case-insensitive basename heuristics)
- Known tool credential dirs (e.g. `.aws/credentials`, `.ssh/`) when they appear in the diff

Prefer pairing with **secret-scrub** for content-level scans of staged hunks.

## Diff summary card

Before a gated git op, show:

```text
GIT GUARD
─────────
Command:  <exact git invocation>
Branch:   <local> → <remote/branch if any>
Summary:  <N files, +adds/-dels; call out secret-like paths>
Risk:     <force / protected / secrets / delete remote / history rewrite>
─────────
Reply: Allow | Deny | Modify <note>
```

Rules:

1. Prefer `git diff --stat` (and name secret-like files explicitly).
2. Never paste secret values into the card — file names and categories only.
3. **Allow** applies to this command once. A later force-push needs a new card.
4. **Deny** stops; do not fall back to an equally destructive alternative without a new card.
5. **Modify** examples: drop force flag, push to a feature branch, unstage `.env`, open a PR instead of pushing to `main`.

## Protected branch heuristic

Treat as protected unless the repo config says otherwise:

- `main`, `master`, `trunk`, `production`, `prod`, `release`, `release/*`

If the agent can read branch protection rules from the host (GitHub/GitLab API), prefer those over the heuristic.

## Anti-patterns

- `git push --force` "because rebase looked clean"
- Amending a commit that was already pushed without saying so on the card
- Squashing secrets out of the index but leaving them in earlier commits without warning
- Renaming a secret file so the basename heuristic misses it — still run **secret-scrub** on content

## Minimal demo prompt (fake)

```text
Agent plans: git push --force origin main
→ Show GIT GUARD card with branch, stat summary, risk=force+protected
→ Wait for Allow | Deny | Modify
```

See `demos/scenarios/02-git-guard.md`.
