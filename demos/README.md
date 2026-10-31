# Demos

Short narratives and fake tool-call prompts that show each gate firing. Safe to run as table-reads with an agent that has the matching skill installed — no real deletes, pushes, charges, or live secrets.

| Scenario | Skill | File |
| --- | --- | --- |
| 01 — Destructive cleanup | permission-gate | [scenarios/01-permission-gate.md](scenarios/01-permission-gate.md) |
| 02 — Force-push to main | git-guard | [scenarios/02-git-guard.md](scenarios/02-git-guard.md) |
| 03 — Stripe-style charge | money-denylist | [scenarios/03-money-denylist.md](scenarios/03-money-denylist.md) |
| 04 — Token in commit | secret-scrub | [scenarios/04-secret-scrub.md](scenarios/04-secret-scrub.md) |

Machine-readable sample of those gates (permission-gate Deny, secret-scrub hit, plus a git-guard warning and a dismissed money note): [findings-example.json](findings-example.json). Document shape: [../schemas/findings.schema.json](../schemas/findings.schema.json).

## How to use

1. Install the relevant skill (see root README).
2. Paste the **User ask** from a scenario into the agent chat.
3. Compare the agent's behavior to the **Expected gate** section.
4. Try **Deny**, then **Modify**, then **Allow** (for money: dual-confirm) and note that Allow is once-only.

All credentials in these demos are obvious fixtures (`sk-test-…`, `ghp_xxxEXAMPLE`). Do not substitute live secrets.

---

## 中文

# 演示

短叙事 + 假想工具调用提示，用来展示各闸门如何触发。可与已安装对应技能的 Agent 对读演练 —— 不做真实删除、推送、扣款，也不使用真实密钥。

| 场景 | 技能 | 文件 |
| --- | --- | --- |
| 01 — 破坏性清理 | permission-gate | [scenarios/01-permission-gate.md](scenarios/01-permission-gate.md) |
| 02 — 强推 main | git-guard | [scenarios/02-git-guard.md](scenarios/02-git-guard.md) |
| 03 — Stripe 风格扣款 | money-denylist | [scenarios/03-money-denylist.md](scenarios/03-money-denylist.md) |
| 04 — 提交里夹 token | secret-scrub | [scenarios/04-secret-scrub.md](scenarios/04-secret-scrub.md) |

上述闸门的机器可读样例（permission-gate 拒绝、secret-scrub 命中，另附一条 git-guard 警告和一条已关闭的资金记录）：[findings-example.json](findings-example.json)。文档形状见 [../schemas/findings.schema.json](../schemas/findings.schema.json)。

## 怎么用

1. 安装对应技能（见根目录 README）。
2. 把场景里的 **用户请求（User ask）** 贴进 Agent 对话。
3. 对照 **预期闸门（Expected gate）** 看 Agent 行为。
4. 依次试 **拒绝（Deny）**、**修改（Modify）**、**允许（Allow）**（资金场景要二次确认）；注意「允许」只对这一次生效。

演示里的凭证全是一眼假的夹具（`sk-test-…`、`ghp_xxxEXAMPLE`）。不要换成真实密钥。
