# agent-safety-kit

[English](README.md)

**四份 Markdown 技能，给 Cursor、Claude Code、OpenCode 或同类 Agent 用。** 拷进 `.cursor/skills`（或该 Agent 的 skills 目录）之后，Agent 应在 force-push、`rm -rf`、花钱或泄密之前停下，并给出 Allow / Deny / Modify 确认卡。规则写在 `SKILL.md` 里。模型不遵守时没有硬拦截：不会在 `git push` 上挂 hook，也不是产品自带的 Auto-review。

## 技能

| 技能 | 作用 |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | 删除、破坏性 shell、出网等高影响步骤前的 Allow / Deny / Modify 确认卡 |
| [git-guard](skills/git-guard/SKILL.md) | force-push、受保护分支、密钥路径、远端删除 |
| [money-denylist](skills/money-denylist/SKILL.md) | 任何资金动作前先停（hard-stop），或二次确认（dual-confirm） |
| [secret-scrub](skills/secret-scrub/SKILL.md) | 扫描 commit、PR 正文、聊天和粘贴；脱敏或拦截，不回显密钥 |

一次 **Allow** 只覆盖这一步。脱敏卡只写类别和位置，不写密钥本身。可以只用一个，也可以四个一起用。

## 安装

Cursor，从本仓库克隆：

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub .cursor/skills/
```

Cursor 用户级目录：把同一条 `cp -R` 拷到 `~/.cursor/skills/`。Claude Code：把同样四个文件夹拷到该 Agent 的 skills 目录。可选：用 `ln -s` 把各个 `skills/<name>` 链到 `.cursor/skills/`，不必复制。技能没出现，就新开一个会话。

## 快速开始

1. 装上这四个技能。
2. 对 Agent 说：`Clean the build artifacts. Something like rm -rf dist .cache is fine.`
3. 删除之前，应先出现 **PERMISSION GATE** 确认卡（What / Touches / Risk）。
4. 回复 **Deny**，再让它列出这些目录。
5. force-push、假扣款、`ghp_xxxEXAMPLE`：[demos/README.md](demos/README.md)。按脚本走一遍即可，凭证都是假的。

## 何时触发

| 风险动作 | 没装 | 装上 |
| --- | --- | --- |
| `rm -rf` 清缓存，或清错目录 | 立刻执行 | **permission-gate** 确认卡 → Allow / Deny / Modify |
| 对 `main` 执行 `git push --force` | 历史被改写 | **git-guard** 先给确认卡；优先普通 push 或功能分支 |
| 提交 `.env`，或 hunk 里的 token | 进 git 历史 | **git-guard** 路径检查 + **secret-scrub** 扫描 → 拦截或脱敏 |
| 结账或 Stripe 扣款 | 卡被扣款 | **money-denylist** dual-confirm；不会静默扣款 |
| API key 出现在 PR 正文或 Slack | 密钥公开 | **secret-scrub** 脱敏或拦截；只报类别 |
| 第一次访问未声明的主机 | 静默出网 | **permission-gate** 出网前先问 |
| 删除远端分支 | 引用消失 | **git-guard** 确认卡，附上 diff / ref 摘要 |

## 配置

没有环境变量、CLI 参数，也没有运行时配置文件。

[skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md) 里的可选商户白名单由人自己维护。Agent 不得自行加商户。白名单为空时，每一笔扣款都要 dual-confirm。本仓库不附带白名单文件。

闸门决策可以存成 JSON。格式见 [`schemas/findings.schema.json`](schemas/findings.schema.json)。样例见 [`demos/findings-example.json`](demos/findings-example.json)：permission-gate 拒绝一次 `rm -rf`、secret-scrub 命中（只记类别和路径）、一条 git-guard 警告，以及一条已关闭的 money-denylist 记录。

| 字段 | 含义 |
| --- | --- |
| `id` | 文件内稳定编号 |
| `skill` | `permission-gate`、`git-guard`、`money-denylist` 或 `secret-scrub` |
| `severity` | `info`（提示）、`warn`（继续前先复核）或 `block`（不要执行） |
| `status` | `confirmed`（已决定）、`needs_review`（等人处理）或 `dismissed`（已关闭） |
| `evidence` | 短摘要，以及可选的位置和类别名 |
| `recommendation` | 下一步做什么 |

## 许可证

MIT License，SPDX `MIT`。Copyright (c) 2026 LofiSu。全文见 [LICENSE](LICENSE)。

## 联系

- GitHub：[loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email：[lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X：[@su_lofi](https://x.com/su_lofi)

建议的 About 与 topics：[REPO_META.md](REPO_META.md)。
