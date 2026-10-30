---
name: permission-gate
description: Generic human-in-the-loop (HITL) gate that pauses before dangerous agent actions and requires Allow, Deny, or Modify.
when-to-use: Before any action that deletes data, runs destructive shell, touches credentials, sends messages externally, moves money, force-pushes git, or reaches undeclared network targets. Load this skill whenever the agent is about to do something irreversible or high-impact.
---

# permission-gate

Stop once. Ask clearly. Proceed only after a human decision.

## Purpose

Agents move fast. Humans need a last look before irreversible or high-impact steps. This skill defines a **generic HITL permission gate**: classify the planned action, present a confirmation card when needed, and wait for **Allow**, **Deny**, or **Modify**.

## Classification

### Safe (no card — proceed)

- Read-only inspection: list files, read files, search, `git status` / `git diff` / `git log`
- Local edits the user already asked for in the current turn
- Installing or running tools that only affect the agent sandbox and do not egress secrets
- Generating drafts, summaries, or plans that are not yet applied externally

### Needs-confirm (card required)

Treat as **needs-confirm** if the planned step matches any of:

| Category | Examples |
| --- | --- |
| Delete / overwrite | `rm -rf`, truncate, wipe DB tables, overwrite large trees |
| Destructive shell | `chmod -R`, disk format, kill unrelated processes, `sudo` for system changes |
| Credentials | Read/write vault, `.env`, keys, tokens, password managers |
| Egress | First contact with a new host/API, upload of local files, webhook posts |
| Messaging | Send email, Slack, chat, social posts under the user's name |
| Payments | Checkout, transfer, Sponsors, wallet, payment API calls |
| Forceful git | `push --force`, rewrite protected branch, delete remote branch |

When in doubt, classify as **needs-confirm**.

## Confirmation card

Before a needs-confirm action, present a short card and **stop**. Do not execute the action until the human answers.

```text
PERMISSION GATE
───────────────
What:     <one sentence: the exact action>
Touches:  <paths | secrets | network targets | remotes>
Risk:     <low | medium | high — why>
───────────────
Reply: Allow | Deny | Modify <note>
```

Rules for the card:

1. **What** — name the concrete command or tool call, not a vague goal.
2. **Touches** — list paths, secret kinds (not values), domains, remotes, or account names.
3. **Risk** — one line on blast radius (data loss, money, public visibility, credential leak).
4. Keep the card under ~12 lines so it is scannable on mobile.

## Human responses

| Reply | Agent behavior |
| --- | --- |
| **Allow** | Run this action **once**. Do not treat Allow as blanket approval for later similar steps. |
| **Deny** | Stop. Do not retry the same action. Acknowledge and offer a safer alternative only if asked. |
| **Modify** | Re-propose with the human's constraints (narrower path, dry-run, different target). Show a new card. |

## Loop discipline

- One gate per risky action (or tightly coupled batch the human can understand as one step).
- After **Allow**, if a later step is again needs-confirm, show a **new** card.
- Never silently escalate from safe to needs-confirm mid-tool-chain without a card.
- Never invent "implied consent" from earlier turns or from memory.

## Anti-patterns

- Running the dangerous command "just to show what would happen"
- Burying the ask inside a long report so the user misses it
- Echoing secret values in the card (name the category only)
- Treating a previous Allow for a different path as approval for this one

## Minimal demo prompt (fake)

```text
Agent plans: rm -rf /workspace/project/dist /workspace/project/.cache
→ Show PERMISSION GATE card (What / Touches / Risk)
→ Wait for Allow | Deny | Modify
```

See `demos/scenarios/01-permission-gate.md` for a full walkthrough.

---

## 中文

**一句话：** 危险动作前通用的人机确认闸门 —— 先停、说清楚、等人选「允许 / 拒绝 / 修改」。

# permission-gate

停一次。问清楚。只有人做了决定再继续。

## 用途

Agent 跑得快，人需要在不可逆或高影响步骤前再看一眼。本技能定义**通用 HITL 权限闸门**：给计划中的动作分类，需要时出确认卡，等待 **允许（Allow）**、**拒绝（Deny）** 或 **修改（Modify）**。

## 分类

### 安全（不出卡 —— 可继续）

- 只读检查：列目录、读文件、搜索、`git status` / `git diff` / `git log`
- 用户本轮已明确要求的本地编辑
- 仅影响 Agent 沙箱、且不会把密钥送出网的工具安装/运行
- 生成尚未对外落地的草稿、摘要或计划

### 需确认（必须出卡）

计划步骤命中任一即视为 **需确认**：

| 类别 | 示例 |
| --- | --- |
| 删除 / 覆盖 | `rm -rf`、截断、清空表、大面积覆盖目录树 |
| 破坏性 shell | `chmod -R`、格式化磁盘、杀掉无关进程、用 `sudo` 改系统 |
| 凭证 | 读写保险库、`.env`、密钥、token、密码管理器 |
| 出网 | 首次访问新主机/API、上传本地文件、打 webhook |
| 发消息 | 以用户名义发邮件、Slack、聊天、社交媒体 |
| 支付 | 结账、转账、Sponsors、钱包、支付 API |
| 强力 git | `push --force`、改写受保护分支、删远端分支 |

拿不准时，一律按 **需确认** 处理。

## 确认卡

需确认动作执行前，出示短卡并 **停下**。人未回复前不得执行。

```text
PERMISSION GATE
───────────────
What:     <一句话：具体动作>
Touches:  <路径 | 密钥类别 | 网络目标 | 远端>
Risk:     <低 | 中 | 高 — 原因>
───────────────
Reply: Allow | Deny | Modify <说明>
```

出卡规则：

1. **What** —— 写具体命令或工具调用，不要只写模糊目标。
2. **Touches** —— 列路径、密钥类别（不要写值）、域名、远端或账户名。
3. **Risk** —— 一行说清爆炸半径（丢数据、花钱、公开可见、泄密）。
4. 整卡控制在约 12 行内，方便手机扫一眼。

## 人工回复

| 回复 | Agent 行为 |
| --- | --- |
| **允许（Allow）** | **只执行这一次**。不得把允许当成后续同类步骤的空白支票。 |
| **拒绝（Deny）** | 停下。不要重试同一动作。仅在被问及时再提更安全的替代。 |
| **修改（Modify）** | 按人的约束重提（更窄路径、干跑、换目标）。再出一张新卡。 |

## 循环纪律

- 每个风险动作一张闸（或人能当成一步理解的紧密批次）。
- **允许** 之后，若下一步仍需确认，必须出 **新卡**。
- 工具链中途从安全悄悄升到需确认而不出卡 —— 禁止。
- 不得用更早轮次或记忆发明「默示同意」。

## 反模式

- 跑危险命令「只是演示一下会怎样」
- 把确认问句埋在长报告里让人漏看
- 在卡里回显密钥值（只报类别）
- 把上次对不同路径的允许当成这次的批准

## 最小演示提示（假想）

```text
Agent 计划：rm -rf /workspace/project/dist /workspace/project/.cache
→ 出示 PERMISSION GATE 卡（What / Touches / Risk）
→ 等待 Allow | Deny | Modify
```

完整演练见 `demos/scenarios/01-permission-gate.md`。
