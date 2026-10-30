# agent-safety-kit

**HITL safety skills for Cursor / Claude-style agents — pause before delete, force-push, spend, or leak.**

A docs-first kit of four Agent Skills you copy into an agent's skills folder. No runtime binary: agents read `SKILL.md` and follow the gates.

## Skills

| Skill | Role |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | Generic HITL card before dangerous actions (Allow / Deny / Modify) |
| [git-guard](skills/git-guard/SKILL.md) | Gate force-push, protected branches, secret paths, remote deletes |
| [money-denylist](skills/money-denylist/SKILL.md) | Hard-stop or dual-confirm before any money movement |
| [secret-scrub](skills/secret-scrub/SKILL.md) | Scan commits / PR bodies / chat / pastes; redact or block without echoing secrets |

## Install

Copy the skill directories into your agent's skills path:

```bash
# Cursor (project or user skills)
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/

# Generic / Claude-style skills folders (adjust to your layout)
cp -R skills/* ~/.config/agent/skills/
# or: cp -R skills/* ./skills/
```

Each skill is a folder with a single `SKILL.md` (YAML frontmatter + instructions). Point your agent at those folders the same way you load any other Agent Skill. No build step.

Optional: keep this repo nearby and symlink:

```bash
ln -s /path/to/agent-safety-kit/skills/permission-gate .cursor/skills/permission-gate
# …repeat for the other three
```

## Without kit vs with kit

| Risky action | Without kit | With kit |
| --- | --- | --- |
| `rm -rf` project cache / wrong tree | Rens immediately | **permission-gate** card → Allow / Deny / Modify |
| `git push --force` to `main` | History rewritten | **git-guard** blocks until confirmed; prefer non-force / feature branch |
| Commit `.env` or a token in a hunk | Lands in git history | **git-guard** path gate + **secret-scrub** content scan → block / redact |
| Agent completes checkout / Stripe charge | Card charged | **money-denylist** dual-confirm; never silent |
| Paste API key into PR Body or Slack | Secret published | **secret-scrub** redacts or blocks; names category only |
| First call to an undeclared host | Silent egress | **permission-gate** needs-confirm for egress |
| Delete remote branch | Ref gone | **git-guard** gated with diff/ref summary |

## 30-second demo

1. Install the four skills (copy into `.cursor/skills` or your skills dir).
2. Ask the agent: *"Clean the build artifacts with `rm -rf dist .cache`."*
3. Expect a **PERMISSION GATE** card (What / Touches / Risk) — not an immediate delete.
4. Reply **Deny**, then ask it to list those dirs instead — agent stops the delete and follows the safer path.
5. Optional: walk `demos/scenarios/01`–`04` for git force-push, fake checkout, and `ghp_xxxEXAMPLE` scrub.

Full scripts: [demos/README.md](demos/README.md).

## Design notes

- **Docs-first** — behavior lives in Markdown skills agents already know how to load.
- **One Allow ≠ forever** — each risky step gets its own card.
- **Never echo secrets** — scrub cards name categories and locations only.
- **Composable** — use one skill or all four; they cross-link where useful.

## License

MIT ̩ 2026 LofiSu — see [LICENSE](LICENSE).

## Repo meta

Suggested GitHub About / topics: [REPO_META.md](REPO_META.md).
