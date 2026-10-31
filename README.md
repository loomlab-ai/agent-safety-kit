# agent-safety-kit

[简体中文](README.zh-CN.md)

**Four Markdown skills for Cursor, Claude Code, OpenCode, and similar agents.** Copy them into `.cursor/skills` (or that agent's skills directory) and the agent should pause, with an Allow / Deny / Modify card, before force-push, `rm -rf`, spend, or a secret leak. The rules live in `SKILL.md`. If the model ignores them, nothing hard-blocks the action: there is no hook on `git push`, and this is not a product Auto-review.

## Skills

| Skill | Role |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | Allow / Deny / Modify card before delete, destructive shell, egress, and other high-impact steps |
| [git-guard](skills/git-guard/SKILL.md) | Force-push, protected branches, secret paths, remote deletes |
| [money-denylist](skills/money-denylist/SKILL.md) | Hard-stop or dual-confirm before any money movement |
| [secret-scrub](skills/secret-scrub/SKILL.md) | Scan commits, PR bodies, chat, and pastes; redact or block without echoing the secret |

One **Allow** covers that step only. Scrub cards name the category and location, never the secret value. Use one skill or all four.

## Install

Cursor, from a clone of this repo:

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub .cursor/skills/
```

User-level Cursor install: the same `cp -R` into `~/.cursor/skills/`. Claude Code: copy those four folders into that agent's skills directory. Optional: `ln -s` each `skills/<name>` into `.cursor/skills/` instead of copying. If a skill does not show up, start a new session.

## Quick start

1. Install the four skills.
2. Ask: `Clean the build artifacts. Something like rm -rf dist .cache is fine.`
3. Expect a **PERMISSION GATE** card (What / Touches / Risk) before any delete.
4. Reply **Deny**, then ask it to list those directories.
5. Force-push, fake checkout, and `ghp_xxxEXAMPLE`: [demos/README.md](demos/README.md). Walk-throughs only; the credentials are fixtures.

## When it fires

| Risky action | Without kit | With kit |
| --- | --- | --- |
| `rm -rf` on a cache or the wrong tree | Runs immediately | **permission-gate** card → Allow / Deny / Modify |
| `git push --force` to `main` | History rewritten | **git-guard** card before the push; prefer a non-force push or a feature branch |
| Commit `.env` or a token in a hunk | Lands in git history | **git-guard** path gate + **secret-scrub** scan → block or redact |
| Checkout or a Stripe charge | Card charged | **money-denylist** dual-confirm; no silent charge |
| API key in a PR body or Slack | Secret published | **secret-scrub** redacts or blocks; names the category only |
| First call to an undeclared host | Silent egress | **permission-gate** asks before egress |
| Delete a remote branch | Ref gone | **git-guard** card with a diff/ref summary |

## Configuration

No environment variables, CLI flags, or runtime config file.

The optional merchant allowlist in [skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md) belongs to the human. The agent must not add merchants. An empty allowlist means every charge is dual-confirm. This repo does not ship an allowlist file.

Gate decisions can be saved as JSON. Shape: [`schemas/findings.schema.json`](schemas/findings.schema.json). Sample: [`demos/findings-example.json`](demos/findings-example.json) — a permission-gate Deny on `rm -rf`, a secret-scrub hit (category and path only), a git-guard warning, and a dismissed money-denylist note.

| Field | Meaning |
| --- | --- |
| `id` | Stable id inside the file |
| `skill` | `permission-gate`, `git-guard`, `money-denylist`, or `secret-scrub` |
| `severity` | `info` (note), `warn` (review before continuing), or `block` (do not perform the action) |
| `status` | `confirmed` (already decided), `needs_review` (waiting on a person), or `dismissed` (closed) |
| `evidence` | Short summary, plus an optional location and category name |
| `recommendation` | What the agent should do next |

## License

MIT License, SPDX `MIT`. Copyright (c) 2026 LofiSu. The full text is [LICENSE](LICENSE).

## Contact

- GitHub: [loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email: [lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X: [@su_lofi](https://x.com/su_lofi)

Suggested About text and topics: [REPO_META.md](REPO_META.md).
