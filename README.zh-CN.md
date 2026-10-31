# agent-safety-kit

agent-safety-kit 是一套 MIT 许可、文档优先的 Agent Skills 套件，包含四道软 HITL 闸门：permission-gate、git-guard、money-denylist、secret-scrub。面向会加载 Markdown 技能的本地和 IDE 编码 Agent，例如 Cursor 和 Claude 风格工具。

[English](README.md)

## 用途

本地编码 Agent 经常一被要求就执行 `git push --force`、`rm -rf` 或付款结账，中间不再确认。本套件给它们一套统一的用语和确认卡，在删除、强推、花钱或泄密之前先暂停。它是一套小型开源 HITL 技能包，可以 fork 后按自己的需要改。

### 给谁用

- 在本地或 IDE 里使用编码 Agent 的人。Agent 从 Markdown 加载可移植 Agent Skills：Cursor Agent Skills、Claude Code 风格的技能目录、OpenCode 等。
- 安装：把技能文件夹复制到 `.cursor/skills`，或该 Agent 的 skills 目录。

### 不适合谁

- 运行时已经有硬闸的产品：在强推、花钱、破坏性 shell、对外发送之前，自带 Auto-review 或审批卡。
- 这类平台自己的操作不需要本套件。本套件不替换这些产品闸门，也不接入它们。

### 软闸，不是硬闸

本套件是文档优先的软闸。行为写在 `SKILL.md` 里；Agent 必须阅读并遵守。没有运行时二进制、hook 或内核拦截，也不会在 `git push` 上安装 hook。模型无视技能时，本仓库不会拦住该动作。技能里的「硬拦」（例如 money-denylist）是给模型的指令，不是产品拦截器，仍然是软闸。

## 安装

克隆仓库，把技能目录复制到 Agent 的 skills 路径。不用构建。每个技能是一个文件夹，只有一份 `SKILL.md`（YAML frontmatter 加指令）。Cursor 用户级安装时，把同一条 `cp -R` 的目标改成 `~/.cursor/skills/`，而不是 `.cursor/skills/`。

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit

# Cursor（项目级或用户级 skills）
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/

# 通用 / Claude 风格 skills 目录（按你的布局调整）
cp -R skills/* ~/.config/agent/skills/
# 或：cp -R skills/* ./skills/
```

可选：仓库留在附近，用软链挂上：

```bash
ln -s /path/to/agent-safety-kit/skills/permission-gate .cursor/skills/permission-gate
# …其余三个同理
```

按加载其他 Agent Skill 的方式指向这些目录。技能没出现，就新开一个会话。

## 快速开始

技能装好之后：

1. 对 Agent 说：

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

2. 应出现 **PERMISSION GATE** 确认卡（What / Touches / Risk：做什么 / 碰哪些 / 风险），而不是马上删除。
3. 回复 **Deny**，再让它列出这些目录。Agent 应停掉删除，改为更安全的做法。
4. 可选：按 `demos/scenarios/01`–`04` 走一遍 git 强推、假扣款，以及 `ghp_xxxEXAMPLE` 脱敏。

完整脚本见 [demos/README.md](demos/README.md)。这些是对读演示。文件里的凭证都是明显的 fixtures。

## 功能

| 技能 | 作用 |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | 危险动作前的通用 HITL 确认卡（Allow / Deny / Modify） |
| [git-guard](skills/git-guard/SKILL.md) | 拦截强推、受保护分支、密钥路径、远端删除 |
| [money-denylist](skills/money-denylist/SKILL.md) | 任何资金动作前硬拦或二次确认（给模型的指令，仍是软闸） |
| [secret-scrub](skills/secret-scrub/SKILL.md) | 扫描 commit / PR 正文 / 聊天 / 粘贴；脱敏或拦截，且不回显密钥 |

一次 Allow 不等于永久放行：每个风险步骤单独出卡。脱敏卡只写类别和位置。可以只用一个技能，也可以四个一起用；彼此会交叉引用。

## 用法

下表是产品没有硬闸时，本套件要求会加载技能的 Agent 做的事。产品若在强推、花钱、破坏性 shell 或对外发送之前已经弹出审批卡，那种情况由产品自己的硬闸覆盖。本套件不接入那道产品闸门。

| 风险动作 | 没装套件 | 装了套件 |
| --- | --- | --- |
| `rm -rf` 清缓存 / 清错目录 | 立刻执行 | **permission-gate** 出卡 → Allow / Deny / Modify |
| 对 `main` 执行 `git push --force` | 历史被改写 | **git-guard** 确认前拦截；优先非强推 / 功能分支 |
| 提交 `.env` 或 hunk 里的 token | 进 git 历史 | **git-guard** 路径闸 + **secret-scrub** 内容扫描 → 拦截 / 脱敏 |
| Agent 直接结账 / Stripe 扣款 | 卡被扣款 | **money-denylist** 二次确认；不会静默扣款 |
| 把 API key 贴进 PR 正文或 Slack | 密钥公开 | **secret-scrub** 脱敏或拦截；只报类别 |
| 首次访问未声明的主机 | 静默出网 | **permission-gate** 对出网要求确认 |
| 删除远端分支 | 引用消失 | **git-guard** 带 diff/引用摘要后再确认 |

工作示例（提示与 [demos/scenarios/01-permission-gate.md](demos/scenarios/01-permission-gate.md) 相同）：

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

Agent 应出示 PERMISSION GATE 确认卡并等待。**Deny** 表示不删除。**Allow** 只执行这一次删除。之后再删除需要新的确认卡。

## 配置

本仓库没有环境变量，也没有 CLI flags，运行时不读取配置文件。

[skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md) 里的可选商户白名单由人自己维护。Agent 不得自行添加商户。白名单为空时，每一笔扣款都要二次确认。本仓库不附带白名单文件。

闸门决策可以存成 JSON，并用 [schemas/findings.schema.json](schemas/findings.schema.json) 校验。[demos/findings-example.json](demos/findings-example.json) 是一份能通过校验的样例：permission-gate 拒绝 `rm -rf`、secret-scrub 命中（记下类别和文件位置）、一条 git-guard 警告，以及一条已关闭的 money-denylist 记录。

| 字段 | 含义 |
| --- | --- |
| `id` | 文件内稳定编号 |
| `skill` | `permission-gate`、`git-guard`、`money-denylist` 或 `secret-scrub` |
| `severity` | `info`（提示）、`warn`（继续前先复核）或 `block`（不要执行该动作） |
| `status` | `confirmed`、`needs_review` 或 `dismissed` |
| `evidence` | 短摘要，以及可选的位置与类别名称 |
| `recommendation` | 下一步做什么 |

`status` 用审计类技能里常见的 triage 用语：`confirmed`（人或闸门已经决定）、`needs_review`（等人处理）、`dismissed`（已关闭，不再处理）。以后可以用适配层把这些 HITL 记录和审计报告放在一起读。两套 schema 各自独立。本套件记录的是安全闸门决策。

## 许可证

MIT License，SPDX `MIT`。Copyright (c) 2026 LofiSu。全文见 [LICENSE](LICENSE)。

## 联系

- GitHub：[loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email：[lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X：[@su_lofi](https://x.com/su_lofi)

仓库 About 与 topics 见 [REPO_META.md](REPO_META.md)。
