# agent-safety-kit

agent-safety-kit 是一套采用 MIT License 的文档优先 Agent Skills 软闸（permission-gate、git-guard、money-denylist、secret-scrub），给在本地和 IDE 里运行 Markdown 技能的编码代理使用（如 Cursor 与 Claude 风格工具），检查只写在 SKILL.md 里。

[English](README.md)

## 用途

本地编码代理经常一被要求就执行 `git push --force`、`rm -rf` 或付款结账，中间不再确认。agent-safety-kit 给它们一套共用说法和确认卡，让它们在删除、强推、花钱或泄密之前先停一下。它是一份可以 fork 再改的小型开源 HITL 技能包。检查是 `SKILL.md` 里的指令。没有运行时二进制，没有 hook，也没有内核拦截。模型如果无视技能，本仓库不会硬拦该动作。

### 给谁用

- 在本地和 IDE 里、从 Markdown 加载可移植 Agent Skills 的人：Cursor Agent Skills、Claude Code 风格的技能目录、OpenCode，以及同类工具。
- 安装方式：把技能文件夹复制到 `.cursor/skills`，或该代理的 skills 目录。

### 不是给谁

- 运行时已经有硬闸的产品：强推、花钱、破坏性 shell、对外发送之前，自带 Auto-review 或审批卡。
- 这类平台自己的操作不需要本套件。本套件不替换它们的产品闸门，也不接入那些闸门。

### 软闸，不是硬闸

本套件是文档优先的软闸。行为写在 `SKILL.md` 里，代理必须阅读并遵守。技能里写的「硬拦」（例如 money-denylist）是给模型的指令，不是产品拦截器。

## 快速开始

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub \
  .cursor/skills/
```

1. 对代理说：

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

2. 应看到 **PERMISSION GATE** 确认卡（做什么 / 碰哪些 / 风险），而不是立刻删除。
3. 回复 **Deny**，再让它列出那些目录。代理应停下删除并走更安全的路径。
4. 可选：按 `demos/scenarios/01`–`04` 走 git 强推、假扣款，以及 `ghp_xxxEXAMPLE` 脱敏演示。

完整脚本见 [demos/README.md](demos/README.md)。

## 功能

- **permission-gate** — 危险动作前的通用 HITL 确认卡（Allow / Deny / Modify）。
- **git-guard** — 拦截强推、受保护分支、密钥路径和远端删除。
- **money-denylist** — 技能要求代理在任何资金动作前硬拦或二次确认。这一条仍然是软闸。
- **secret-scrub** — 扫描 commit、PR 正文、聊天和粘贴；脱敏或拦截，且不回显密钥。
- 文档优先的软闸，写在 `SKILL.md` 里。本套件不代替产品 Auto-review，也不在 `git push` 上安装 hook。
- 结构化发现 JSON 见 [schemas/findings.schema.json](schemas/findings.schema.json)，样例见 [demos/findings-example.json](demos/findings-example.json)。
- 对读演示在 [demos/](demos/README.md)。其中的凭证都是一眼可辨的夹具。

| 技能 | 作用 |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | 危险动作前的通用 HITL 确认卡（Allow / Deny / Modify） |
| [git-guard](skills/git-guard/SKILL.md) | 拦截强推、受保护分支、密钥路径、远端删除 |
| [money-denylist](skills/money-denylist/SKILL.md) | 任何资金动作前硬拦或二次确认 |
| [secret-scrub](skills/secret-scrub/SKILL.md) | 扫描 commit / PR 正文 / 聊天 / 粘贴；脱敏或拦截，且不回显密钥 |

一次 Allow 不等于永久放行：每个风险步骤各自出卡。脱敏卡只报类别和位置。可以只用一个技能，也可以四个一起用；彼此有交叉引用。

## 安装

克隆仓库，再把技能目录复制到代理的 skills 路径。无需构建。每个技能是一个只含 `SKILL.md`（YAML 头加指令）的文件夹。Cursor 用户级安装时，把同一条 `cp -R` 的目标换成 `~/.cursor/skills/`，而不是 `.cursor/skills/`。

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

可选：仓库留在附近，用软链挂上：

```bash
ln -s /path/to/agent-safety-kit/skills/permission-gate .cursor/skills/permission-gate
# …repeat for the other three
```

按加载其他 Agent Skill 的方式指向这些目录。如果技能没有出现，新开一轮会话。

## 用法

下表是：没有产品硬闸时，本套件要求会加载技能的代理怎么做。如果产品在强推、花钱、破坏性命令或对外发送之前已经弹出审批卡，那种情况由产品自己的硬闸覆盖。本套件不接入那道产品闸门。

| 风险动作 | 没装套件 | 装了套件 |
| --- | --- | --- |
| `rm -rf` 清缓存 / 清错目录 | 立刻执行 | **permission-gate** 出卡 → Allow / Deny / Modify |
| 对 `main` 执行 `git push --force` | 历史被改写 | **git-guard** 确认前拦截；优先非强推 / 功能分支 |
| 提交 `.env` 或 hunk 里的 token | 进 git 历史 | **git-guard** 路径闸 + **secret-scrub** 内容扫描 → block / redact |
| Agent 直接结账 / Stripe 扣款 | 卡被扣款 | **money-denylist** 二次确认；绝不静默 |
| 把 API key 贴进 PR 正文或 Slack | 密钥公开 | **secret-scrub** 脱敏或拦截；只报类别 |
| 首次访问未声明的主机 | 静默出网 | **permission-gate** 对出网要求确认 |
| 删除远端分支 | 引用消失 | **git-guard** 带 diff/引用摘要后再确认 |

工作示例（与 [demos/scenarios/01-permission-gate.md](demos/scenarios/01-permission-gate.md) 中的提示相同）：

```text
Clean the build artifacts. Something like rm -rf dist .cache is fine.
```

代理应出示 PERMISSION GATE 确认卡并等待。**Deny** 表示不删除。**Allow** 只执行这一次删除。之后再删除需要新的确认卡。

## 配置

本仓库没有环境变量，也没有 CLI 标志。目录里没有任何文件会在运行时读取配置。

[skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md) 里的可选商户白名单是人自己保管的笔记。代理不得自行把商户加进去。白名单为空时，每一笔扣款都要二次确认。本仓库不附带白名单文件。

闸门决策可以存成 JSON，并用 [schemas/findings.schema.json](schemas/findings.schema.json) 校验。[demos/findings-example.json](demos/findings-example.json) 是一份能通过校验的样例：permission-gate 对 `rm -rf` 的拒绝、secret-scrub 的一次命中（记下类别和文件位置）、一条 git-guard 警告，以及一条已关闭的 money-denylist 记录。

| 字段 | 含义 |
| --- | --- |
| `id` | 文件内稳定编号 |
| `skill` | `permission-gate`、`git-guard`、`money-denylist` 或 `secret-scrub` |
| `severity` | `info`（提示）、`warn`（继续前先看一眼）或 `block`（不要执行该动作） |
| `status` | `confirmed`、`needs_review` 或 `dismissed` |
| `evidence` | 短摘要，以及可选的位置与类别名称 |
| `recommendation` | 下一步做什么 |

`status` 采用审计风格技能里常见的分流词：`confirmed`（人或闸门已经做出决定）、`needs_review`（等人处理）、`dismissed`（关闭，不再动作）。以后可以用适配层把这些 HITL 记录和审计报告放在一起读。两套 schema 各自独立。本套件记录的是安全闸门决策。

## 常见问题

### agent-safety-kit 给谁用？

给在本地和 IDE 里运行、会加载 Markdown 技能的编码代理（Cursor、Claude 风格技能目录、OpenCode，以及同类工具）。把技能文件夹复制到代理的 skills 目录。已经在强推、花钱、破坏性 shell 或对外发送之前强制审批的产品，用它们自己的运行时硬闸。

### agent-safety-kit 是能代替 Auto-review 的硬闸吗？

agent-safety-kit 是文档优先的软闸。检查写在 `SKILL.md` 里。Auto-review 和同类产品卡片是产品运行时里的硬闸。本套件不替换那些闸门，也不接入它们。模型如果跳过技能，本仓库不会拦住动作。

### 模型如果无视技能会怎样？

本套件不拦截 shell、git 或支付 API。技能只有在代理阅读并遵守时才起作用。把确认卡当成共用流程，而不是内核或必需的状态检查。

### 这套技能会拦住 git push 吗？

它不安装 hook，不添加必需检查，也不添加会拒绝 `git push` 的权限规则。**git-guard** 要求代理在强推、改写受保护分支、密钥路径和远端删除之前出卡。代理仍可能无视该指令。发布本仓库不会加上推送拦截器。

## 许可证

MIT License，SPDX `MIT`。Copyright (c) 2026 LofiSu。全文见 [LICENSE](LICENSE)。

## 联系

- GitHub：[loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email：[lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X：[@su_lofi](https://x.com/su_lofi)

仓库 About 文案与 topics：[REPO_META.md](REPO_META.md)。
