# agent-safety-kit

[简体中文](README.zh-CN.md)

Four Markdown skills that make Cursor, Claude, and similar agents ask before they force-push, wipe a tree, spend money, or dump secrets. Soft only — if the model ignores the skill, nothing hard-stops it.

## Skills

| Skill | What it does |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | Allow / Deny / Modify before a delete, a destructive shell command, or a host it has never called |
| [git-guard](skills/git-guard/SKILL.md) | Asks before force-push, a protected-branch rewrite, a secret path, or a remote-branch delete |
| [money-denylist](skills/money-denylist/SKILL.md) | Hard-stop or dual-confirm before checkout, a transfer, or a payment API |
| [secret-scrub](skills/secret-scrub/SKILL.md) | Scans a commit, PR body, chat, or paste. Redacts or blocks. Never prints the secret |

Install one, or install all four. **Allow** covers that step only. The next risky step asks again.

## Install

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub .cursor/skills/
```

That copy is for this project. For every project, use `~/.cursor/skills/` instead.

Claude Code or OpenCode: put the same four folders in that agent's skills directory.

Symlink if you'd rather not copy: `ln -s` each `skills/<name>` into `.cursor/skills/`.

Skill still missing? Start a new chat.

## Try it

1. Install the four skills.
2. Say: `Clean the build artifacts. Something like rm -rf dist .cache is fine.`
3. You get a **PERMISSION GATE** card (What / Touches / Risk) before anything is deleted.
4. Reply **Deny**. Then tell it to list those directories.
5. Force-push, a fake charge, and `ghp_xxxEXAMPLE`: [demos/README.md](demos/README.md). Read the script with the agent. The tokens are fake.

## When it asks

| You asked for | No skills | With skills |
| --- | --- | --- |
| `rm -rf` on a cache, or the wrong tree | Deletes now | **permission-gate** card: Allow / Deny / Modify |
| `git push --force` to `main` | Rewrites history | **git-guard** asks first. Push without `--force`, or use a branch |
| Commit `.env`, or a token in a hunk | Secret lands in git | **git-guard** catches the path. **secret-scrub** catches the text |
| Checkout, or a Stripe charge | Your card is charged | **money-denylist** wants dual-confirm. It does not charge on its own |
| An API key in a PR or in Slack | The key goes public | **secret-scrub** redacts or blocks, and names the category, not the value |
| A call to a host you never named | The call goes out | **permission-gate** asks before the call |
| Delete a remote branch | The ref is gone | **git-guard** shows the diff and the ref, then waits |

## Config

No env vars. No CLI flags. No config file.

You keep the merchant allowlist. See [skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md). The agent does not add merchants. Empty list? Every charge needs dual-confirm. No allowlist file in this repo.

Save the decision as JSON if another tool should read it. Schema: [`schemas/findings.schema.json`](schemas/findings.schema.json). Example: [`demos/findings-example.json`](demos/findings-example.json). One `rm -rf` Deny, one secret hit with the token left out, one git warning, one dismissed money note.

| Field | Meaning |
| --- | --- |
| `id` | Stable id in the file |
| `skill` | `permission-gate`, `git-guard`, `money-denylist`, or `secret-scrub` |
| `severity` | `info` (a note), `warn` (look before you go on), `block` (don't do it) |
| `status` | `confirmed` (decided), `needs_review` (waiting on you), `dismissed` (closed) |
| `evidence` | What happened. Add a path or a category when you have one |
| `recommendation` | What to do next |

## License

MIT License, SPDX `MIT`. Copyright (c) 2026 LofiSu. The full text is [LICENSE](LICENSE).

## Contact

- GitHub: [loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email: [lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X: [@su_lofi](https://x.com/su_lofi)

About text and topics: [REPO_META.md](REPO_META.md).
