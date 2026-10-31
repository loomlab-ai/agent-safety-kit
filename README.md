# agent-safety-kit

agent-safety-kit is a docs-first MIT-licensed Agent Skills kit of soft HITL gates (permission-gate, git-guard, money-denylist, and secret-scrub) for people running local and IDE Markdown skill coding agents such as Cursor and Claude-style tools, with the checks written only in SKILL.md.

[简体中文](README.zh-CN.md)

## Purpose

Local coding agents often run `git push --force`, `rm -rf`, or a payment checkout as soon as they are asked, with no second check. agent-safety-kit gives them a shared vocabulary and permission cards so they pause before delete, force-push, spend, or leak. It is a small open-source HITL skill pack you can fork and adapt. The checks are instructions in `SKILL.md`. There is no runtime binary, hook, or kernel interceptor. If the model ignores the skill, this repository does not hard-block the action.

### Who this is for

- People running local and IDE coding agents that load portable Agent Skills from Markdown: Cursor Agent Skills, Claude Code–style skill folders, OpenCode, and similar.
- Install by copying the skill folders into `.cursor/skills` or that agent's skills directory.

### Who this is not for

- Products that already enforce hard gates in the runtime: built-in Auto-review or approval cards before force-push, spend, destructive shell, or outbound send.
- Those platforms do not need this kit for their own operations. This kit does not replace their product gates and does not plug into them.

### Soft gates, not hard gates

This kit is docs-first soft gates. Behavior lives in `SKILL.md`; the agent must read the skill and obey it. A skill line that says "hard-stop" (for example in money-denylist) is an instruction to the model, not a product interceptor.

## Quick start

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/
```

1. Ask the agent:

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

2. Expect a **PERMISSION GATE** card (What / Touches / Risk), not an immediate delete.
3. Reply **Deny**, then ask it to list those directories. The agent stops the delete and follows the safer path.
4. Optional: walk `demos/scenarios/01`–`04` for git force-push, a fake checkout, and a `ghp_xxxEXAMPLE` scrub.

Full scripts: [demos/README.md](demos/README.md).

## Features

- **permission-gate** — generic HITL card before dangerous actions (Allow / Deny / Modify).
- **git-guard** — gate force-push, protected branches, secret paths, and remote deletes.
- **money-denylist** — the skill tells the agent to hard-stop or dual-confirm before any money movement. That line is still a soft gate.
- **secret-scrub** — scan commits, PR bodies, chat, and pastes; redact or block without echoing secrets.
- Docs-first soft gates in `SKILL.md`. This kit is not a substitute for product Auto-review, and it does not install a hook on `git push`.
- Structured findings JSON at [schemas/findings.schema.json](schemas/findings.schema.json), with a sample at [demos/findings-example.json](demos/findings-example.json).
- Table-read demos under [demos/](demos/README.md). Credentials in those files are obvious fixtures.

| Skill | Role |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | Generic HITL card before dangerous actions (Allow / Deny / Modify) |
| [git-guard](skills/git-guard/SKILL.md) | Gate force-push, protected branches, secret paths, remote deletes |
| [money-denylist](skills/money-denylist/SKILL.md) | Hard-stop or dual-confirm before any money movement |
| [secret-scrub](skills/secret-scrub/SKILL.md) | Scan commits / PR bodies / chat / pastes; redact or block without echoing secrets |

One Allow is not forever: each risky step gets its own card. Scrub cards name categories and locations only. Use one skill or all four; they cross-link where useful.

## Install

Clone the repository, then copy the skill directories into the agent's skills path. No build step. Each skill is a folder with a single `SKILL.md` (YAML frontmatter plus instructions). For a user-level Cursor install, use the same `cp -R` into `~/.cursor/skills/` instead of `.cursor/skills/`.

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit

# Cursor (project or user skills)
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/

# Generic / Claude-style skills folders (adjust to your layout)
cp -R skills/* ~/.config/agent/skills/
# or: cp -R skills/* ./skills/
```

Optional: keep this repo nearby and symlink:

```bash
ln -s /path/to/agent-safety-kit/skills/permission-gate .cursor/skills/permission-gate
# …repeat for the other three
```

Point the agent at those folders the same way you load any other Agent Skill. If a skill does not appear, start a new session.

## Usage

The table is what this kit asks a skill-loading agent to do when a product hard gate is absent. A product that already shows an approval card before force-push, spend, destructive shell, or outbound send covers that case with its own hard gate. This kit does not plug into that product gate.

| Risky action | Without kit | With kit |
| --- | --- | --- |
| `rm -rf` project cache / wrong tree | Runs immediately | **permission-gate** card → Allow / Deny / Modify |
| `git push --force` to `main` | History rewritten | **git-guard** blocks until confirmed; prefer non-force / feature branch |
| Commit `.env` or a token in a hunk | Lands in git history | **git-guard** path gate + **secret-scrub** content scan → block / redact |
| Agent completes checkout / Stripe charge | Card charged | **money-denylist** dual-confirm; never silent |
| Paste API key into PR body or Slack | Secret published | **secret-scrub** redacts or blocks; names category only |
| First call to an undeclared host | Silent egress | **permission-gate** needs-confirm for egress |
| Delete remote branch | Ref gone | **git-guard** gated with diff/ref summary |

Working example (same prompt as [demos/scenarios/01-permission-gate.md](demos/scenarios/01-permission-gate.md)):

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

The agent should show a PERMISSION GATE card and wait. **Deny** means no delete. **Allow** runs that delete once. A later delete needs a new card.

## Configuration

This repository has no environment variables and no CLI flags. Nothing in the tree reads a config file at runtime.

The optional merchant allowlist described in [skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md) is notes the human owns. The agent must not add merchants by itself. An empty allowlist means every charge is dual-confirm. This repo does not ship an allowlist file.

Gate decisions can be saved as JSON and checked against [schemas/findings.schema.json](schemas/findings.schema.json). [demos/findings-example.json](demos/findings-example.json) is a valid sample: a permission-gate Deny on `rm -rf`, a secret-scrub hit that records the category and the file location, a git-guard warning, and a dismissed money-denylist note.

| Field | Meaning |
| --- | --- |
| `id` | Stable id inside the file |
| `skill` | `permission-gate`, `git-guard`, `money-denylist`, or `secret-scrub` |
| `severity` | `info` (note), `warn` (review before continuing), or `block` (do not perform the action) |
| `status` | `confirmed`, `needs_review`, or `dismissed` |
| `evidence` | Short summary, plus an optional location and category name |
| `recommendation` | What the agent should do next |

`status` uses triage words common to audit-style skills: `confirmed` (a person or the gate already decided), `needs_review` (waiting on a person), `dismissed` (closed with no further action). A later adapter can place these HITL records beside an audit report. The schemas stay independent. This kit records safety-gate decisions.

## FAQ

### Who is agent-safety-kit for?

People running local and IDE coding agents that load Markdown skills (Cursor, Claude-style skill folders, OpenCode, and similar). Copy the skill folders into the agent's skills directory. Products that already enforce hard approval before force-push, spend, destructive shell, or outbound send use their own runtime gate.

### Is agent-safety-kit a hard gate that replaces Auto-review?

agent-safety-kit is a docs-first soft gate. The checks live in `SKILL.md`. Auto-review and similar product cards are hard gates in a product runtime. This kit does not replace those gates and does not plug into them. If the model skips the skill, this repository does not block the action.

### What if the model ignores the skill?

Nothing in this kit intercepts the shell, git, or a payment API. The skill works when the agent reads it and follows it. Treat the cards as a shared procedure, not as a kernel or a required status check.

### Does this kit block git push?

It does not install a hook, a required check, or a permission rule that rejects `git push`. **git-guard** tells the agent to show a card before force-push, protected-branch writes, secret paths, and remote deletes. The agent can still ignore that instruction. Publishing this repository does not add a push blocker.

## License

MIT License, SPDX `MIT`. Copyright (c) 2026 LofiSu. The full text is [LICENSE](LICENSE).

## Contact

- GitHub: [loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email: [lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X: [@su_lofi](https://x.com/su_lofi)

Repository About text and topics: [REPO_META.md](REPO_META.md).
