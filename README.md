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
| `rm -rf` project cache / wrong tree | Runs immediately | **permission-gate** card → Allow / Deny / Modify |
| `git push --force` to `main` | History rewritten | **git-guard** blocks until confirmed; prefer non-force / feature branch |
| Commit `.env` or a token in a hunk | Lands in git history | **git-guard** path gate + **secret-scrub** content scan → block / redact |
| Agent completes checkout / Stripe charge | Card charged | **money-denylist** dual-confirm; never silent |
| Paste API key into PR body or Slack | Secret published | **secret-scrub** redacts or blocks; names category only |
| First call to an undeclared host | Silent egress | **permission-gate** needs-confirm for egress |
| Delete remote branch | Ref gone | **git-guard** gated with diff/ref summary |

## 30-second demo

1. Install the four skills (copy into `.cursor/skills` or your skills dir).
2. Ask the agent: *“Clean the build artifacts with `rm -rf dist .cache`.”*
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

MIT © 2026 LofiSu — see [LICENSE](LICENSE).

## Repo meta

Suggested GitHub About / topics: [REPO_META.md](REPO_META.md).

---

## 中文

# agent-safety-kit

**面向 Cursor / Claude 类 Agent 的人机确认（HITL）安全技能包 —— 删除、强推、花钱、泄密之前先停住。**

文档优先的四件套 Agent Skills：复制进 Agent 的 skills 目录即可。无运行时二进制，Agent 阅读 `SKILL.md` 并按闸门执行。

## 技能一览

| 技能 | 作用 |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | 危险动作前的通用 HITL 确认卡（允许 / 拒绝 / 修改） |
| [git-guard](skills/git-guard/SKILL.md) | 拦截强推、受保护分支改写、密钥路径提交、远端删除 |
| [money-denylist](skills/money-denylist/SKILL.md) | 任何资金动作前硬拦或二次确认 |
| [secret-scrub](skills/secret-scrub/SKILL.md) | 扫描 commit / PR 正文 / 聊天 / 粘贴；脱敏或拦截，且不回显密钥 |

## 安装

把技能目录复制到 Agent 的 skills 路径：

```bash
# Cursor（项目级或用户级 skills）
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/

# 通用 / Claude 风格 skills 目录（按你的布局调整）
cp -R skills/* ~/.config/agent/skills/
# 或：cp -R skills/* ./skills/
```

每个技能就是一个带 `SKILL.md`（YAML 头 + 指令）的文件夹。按加载其他 Agent Skill 的方式指向这些目录即可，无需构建。

可选：仓库留在附近，用软链挂上：

```bash
ln -s /path/to/agent-safety-kit/skills/permission-gate .cursor/skills/permission-gate
# …其余三个同理
```

## 没装套件 vs 装了套件

| 风险动作 | 没装套件 | 装了套件 |
| --- | --- | --- |
| `rm -rf` 清缓存 / 清错目录 | 立刻执行 | **permission-gate** 出卡 → 允许 / 拒绝 / 修改 |
| 对 `main` 执行 `git push --force` | 历史被改写 | **git-guard** 确认前拦截；建议非强推 / 功能分支 |
| 把 `.env` 或 hunk 里的 token 提交进去 | 进 git 历史 | **git-guard** 路径闸 + **secret-scrub** 内容扫描 → 拦截 / 脱敏 |
| Agent 直接下单 / 调 Stripe 扣款 | 卡被扣款 | **money-denylist** 二次确认；绝不静默扣款 |
| 把 API Key 贴进 PR 或 Slack | 密钥公开 | **secret-scrub** 脱敏或拦截；只报类别 |
| 首次访问未声明的主机 | 静默出网 | **permission-gate** 对出网要求确认 |
| 删除远端分支 | 引用消失 | **git-guard** 带 diff/引用摘要再确认 |

## 30 秒演示

1. 安装四个技能（复制到 `.cursor/skills` 或你的 skills 目录）。
2. 对 Agent 说：*「用 `rm -rf dist .cache` 清一下构建产物。」*
3. 应看到 **PERMISSION GATE** 确认卡（做什么 / 碰哪些 / 风险），而不是立刻删除。
4. 回复 **拒绝（Deny）**，再让它改成列出那些目录 —— Agent 应停下删除并走更安全的路径。
5. 可选：按 `demos/scenarios/01`–`04` 走强推、假扣款、以及 `ghp_xxxEXAMPLE` 脱敏演示。

完整脚本见 [demos/README.md](demos/README.md)。

## 设计要点

- **文档优先** —— 行为写在 Agent 本来就会加载的 Markdown 技能里。
- **一次允许 ≠ 永久放行** —— 每个风险步骤各自出卡。
- **永不回显密钥** —— 脱敏卡只报类别与位置。
- **可组合** —— 可单用也可四件齐上；彼此有交叉引用。

## 许可证

MIT © 2026 LofiSu — 见 [LICENSE](LICENSE)。

## 仓库元信息

GitHub About / topics 建议见 [REPO_META.md](REPO_META.md)。
