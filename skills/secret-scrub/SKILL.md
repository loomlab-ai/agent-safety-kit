---
name: secret-scrub
description: Before commit, PR body, chat send, or paste to the web, scan for tokens, keys, and ID-like secrets; on hit redact or block and name the category only — never echo the secret.
when-to-use: Before git commit or push, before opening/updating a PR description, before sending chat/email/Slack, before pasting agent output into a browser form, or whenever staged diffs / outbound text may contain credentials.
---

# secret-scrub

Scan outbound text. Redact or block. Never repeat the secret.

## Purpose

Agents copy from logs, `.env` files, and tool output into commits, PRs, and messages. This skill defines **when to scan**, **what to look for**, and **how to respond** without leaking the matched value into the transcript.

## When to scan (mandatory)

Run a scrub pass immediately before:

1. `git commit` / staging a commit message that embeds file contents
2. Opening or editing a **PR / MR body** or review comment
3. **Sending** chat, email, Slack, or social posts
4. **Pasting** into a web form, issue tracker, or pastebin
5. Uploading logs or screenshots that may contain terminal output with env dumps

Also scan when the user asks to "share this config" or "paste the error."

## Pattern categories

Match on **category**, not on a single brittle regex. Suggested detectors (fixtures use obvious fakes):

| Category | Example fake fixtures (safe to put in docs) | Real-world shape (do not echo matches) |
| --- | --- | --- |
| Cloud / LLM API keys | `sk-test-EXAMPLEKEY000` | `sk-…`, vendor-prefixed tokens |
| GitHub tokens | `ghp_xxxEXAMPLE` | `ghp_`, `gho_`, `ghs_`, `github_pat_` |
| Private keys | `-----BEGIN EXAMPLE PRIVATE KEY-----` | PEM / OpenSSH private key blocks |
| AWS-like keys | `AKIAEXAMPLE000000000` | `AKIA…` access key IDs + secret pairs |
| Slack / Discord-like | `xoxb-example-token` | Bot/user tokens |
| Generic passwords | `password=EXAMPLE_ONLY` | `password=`, `passwd=`, `pwd=` assignments |
| Connection strings | `postgres://user:EXAMPLE@host/db` | URI with embedded credentials |
| ID-like numbers | `ID-0000-0000-EXAM` | National ID / passport-like patterns when clearly sensitive in context |

Tune patterns to your org; false positives should **block and ask**, not auto-send.

## On hit

1. **Do not** print the matched secret in full — not in the reply, not in the card, not in logs the user did not already have.
2. Name the **category** and **location** only, e.g. `GitHub token-like string in staged file .env:12`.
3. Choose one response:

| Mode | When | Behavior |
| --- | --- | --- |
| **Block** | Commit / send / paste about to go out | Stop the outbound action; offer to redact and retry |
| **Redact** | User asked to share a snippet | Replace match with `[REDACTED:<category>]` and show the scrubbed text |
| **Quarantine** | Diff contains secrets | Unstage or exclude those hunks; pair with **git-guard** |

```text
SECRET SCRUB
────────────
Action blocked: <commit | PR body | chat send | web paste>
Hits:   1) <category> in <path-or-field>:<line-or-span>
        2) …
Next:   Redact & retry | Deny (abort) | Modify <what to exclude>
────────────
```

## Fixtures for tests and demos

Use only obvious fakes in repositories and demos:

```text
sk-test-EXAMPLEKEY000
ghp_xxxEXAMPLE
AKIAEXAMPLE000000000
xoxb-example-token
-----BEGIN EXAMPLE PRIVATE KEY-----
password=EXAMPLE_ONLY
postgres://user:EXAMPLE@localhost:5432/app
```

Never commit live credentials "for realism."

## Coordination with other kit skills

- **git-guard** — path-level block for `.env` / keys; secret-scrub adds content-level scan of hunks and commit messages.
- **permission-gate** — scrub failures surface as a needs-confirm / block card before egress.
- **money-denylist** — payment tools may return receipts with masked PANs; still scrub raw card fields if present.

## Anti-patterns

- Quoting the secret "so the user can see what was found"
- Base64-wrapping a secret and calling it scrubbed
- Scanning only filenames and ignoring PR descriptions or chat drafts
- Auto-rewriting history to remove secrets without telling the human

## Minimal demo prompt (fake)

```text
Agent about to commit a file containing: ghp_xxxEXAMPLE
→ SECRET SCRUB blocks; reports category "GitHub token-like" at path:line
→ Offers Redact & retry | Deny | Modify — never prints ghp_xxxEXAMPLE in the card beyond the fixture docs
```

See `demos/scenarios/04-secret-scrub.md`.
