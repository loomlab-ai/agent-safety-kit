---
name: secret-scrub
description: Before commit, PR body, chat send, or paste to the web, scan for tokens, keys, and ID-like secrets; on hit redact or block and name the category only — never echo the secret.
when-to-use: Before git commit or push, before opening/updating a PR description, before sending chat/email/Slack, before pasting agent output into a browser form, or whenever staged diffs / outbound text may contain credentials.
---

# secret-scrub

Scan outbound text. Redact or block. Never repeat the secret.

## Purpose

Agents copy from logs, `.env` files, and tool output into commits, PRs, and messages. This skill defines **when to scan**, **what to look for**, and **how to respond** without leaking the matched value into the transcript.

## When to scan (mandatory)

Run a scrub pass immediately before:

1. `git commit` / staging a commit message that embeds file contents
2. Opening or editing a **PR / MR body** or review comment
3. **Sending** chat, email, Slack, or social posts
4. **Pasting** into a web form, issue tracker, or pastebin
5. Uploading logs or screenshots that may contain terminal output with env dumps

Also scan when the user asks to "share this config" or "paste the error."

## Pattern categories

Match on **category**, not on a single brittle regex. Suggested detectors (fixtures use obvious fakes):

| Category | Example fake fixtures (safe to put in docs) | Real-world shape (do not echo matches) |
| --- | --- | --- |
| Cloud / LLM API keys | `sk-test-EXAMPLEKEY000` | `sk-…`, vendor-prefixed tokens |
| GitHub tokens | `ghp_xxxEXAMPLE` | `ghp_`, `gho_`, `ghs_`, `github_pat_` |
| Private keys | `-----BEGIN EXAMPLE PRIVATE KEY-----` | PEM / OpenSSH private key blocks |
| AWS-like keys | `AKIAEXAMPLE000000000` | `AKIA…` access key IDs + secret pairs |
| Slack / Discord-like | `xoxb-example-token` | Bot/user tokens |
| Generic passwords | `password=EXAMPLE_ONLY` | `password=`, `passwd=`, `pwd=` assignments |
| Connection strings | `postgres://user:EXAMPLE@host/db` | URI with embedded credentials |
| ID-like numbers | `ID-0000-0000-EXAM` | National ID / passport-like patterns when clearly sensitive in context |

Tune patterns to your org; false positives should **block and ask**, not auto-send.

## On hit

1. **Do not** print the matched secret in full — not in the reply, not in the card, not in logs the user did not already have.
2. Name the **category** and **location** only, e.g. `GitHub token-like string in staged file .env:12`.
3. Choose one response:

| Mode | When | Behavior |
| --- | --- | --- |
| **Block** | Commit / send / paste about to go out | Stop the outbound action; offer to redact and retry |
| **Redact** | User asked to share a snippet | Replace match with `[REDACTED:<category>]` and show the scrubbed text |
| **Quarantine** | Diff contains secrets | Unstage or exclude those hunks; pair with **git-guard** |

```text
SECRET SCRUB
────────────
Action blocked: <commit | PR body | chat send | web paste>
Hits:   1) <category> in <path-or-field>:<line-or-span>
        2) …
Next:   Redact & retry | Deny (abort) | Modify <what to exclude>
────────────
```

## Fixtures for tests and demos

Use only obvious fakes in repositories and demos:

```text
sk-test-EXAMPLEKEY000
ghp_xxxEXAMPLE
AKIAEXAMPLE000000000
xoxb-example-token
-----BEGIN EXAMPLE PRIVATE KEY-----
password=EXAMPLE_ONLY
postgres://user:EXAMPLE@localhost:5432/app
```

Never commit live credentials "for realism."

## Coordination with other kit skills

- **git-guard** — path-level block for `.env` / keys; secret-scrub adds content-level scan of hunks and commit messages.
- **permission-gate** — scrub failures surface as a needs-confirm / block card before egress.
- **money-denylist** — payment tools may return receipts with masked PANs; still scrub raw card fields if present.

## Anti-patterns

- Quoting the secret "so the user can see what was found"
- Base64-wrapping a secret and calling it scrubbed
- Scanning only filenames and ignoring PR descriptions or chat drafts
- Auto-rewriting history to remove secrets without telling the human

## Minimal demo prompt (fake)

```text
Agent about to commit a file containing: ghp_xxxEXAMPLE
→ SECRET SCRUB blocks; reports category "GitHub token-like" at path:line
→ Offers Redact & retry | Deny | Modify — never prints ghp_xxxEXAMPLE in the card beyond the fixture docs
```

See `demos/scenarios/04-secret-scrub.md`.

---

## 中文

**一句话：** 提交、PR 正文、发聊天或往网页粘贴前扫描 token/密钥/类证件号；命中则脱敏或拦截，只报类别，永不回显密钥。

# secret-scrub

扫描即将外发的文本。脱敏或拦截。绝不复述密钥本身。

## 用途

Agent 常从日志、`.env`、工具输出抄进 commit、PR 和消息。本技能规定 **何时扫描**、**扫什么**、以及如何在 **不把匹配值泄漏进对话** 的前提下响应。

## 何时扫描（强制）

在下列动作 **即将发生前** 立刻跑一轮脱敏：

1. `git commit` / 会嵌入文件内容的提交说明
2. 打开或编辑 **PR / MR 正文** 或评审评论
3. **发送** 聊天、邮件、Slack 或社交媒体
4. **粘贴** 到网页表单、issue 追踪器或 pastebin
5. 上传可能含环境变量转储的日志或截图

用户说「分享这份配置」「把报错贴出去」时也要扫。

## 模式类别

按 **类别** 匹配，不要死磕单条脆弱正则。建议检测器（文档夹具用一眼假的样本）：

| 类别 | 假想夹具示例（可写进文档） | 真实形态（匹配后勿回显） |
| --- | --- | --- |
| 云 / LLM API Key | `sk-test-EXAMPLEKEY000` | `sk-…`、厂商前缀 token |
| GitHub token | `ghp_xxxEXAMPLE` | `ghp_`、`gho_`、`ghs_`、`github_pat_` |
| 私钥 | `-----BEGIN EXAMPLE PRIVATE KEY-----` | PEM / OpenSSH 私钥块 |
| 类 AWS 密钥 | `AKIAEXAMPLE000000000` | `AKIA…` 访问密钥 ID + 密钥对 |
| 类 Slack / Discord | `xoxb-example-token` | 机器人/用户 token |
| 通用密码 | `password=EXAMPLE_ONLY` | `password=`、`passwd=`、`pwd=` 赋值 |
| 连接串 | `postgres://user:EXAMPLE@host/db` | 内嵌凭证的 URI |
| 类证件号 | `ID-0000-0000-EXAM` | 在上下文中明显敏感的身份证/护照类模式 |

按组织调参；误报应 **拦住并询问**，不要自动发出。

## 命中后

1. **不要** 完整打印匹配到的密钥 —— 回复里不要、卡里不要、用户原本没有的日志里也不要。
2. 只报 **类别** 与 **位置**，例如：`暂存文件 .env:12 中有疑似 GitHub token`。
3. 选一种响应：

| 模式 | 何时 | 行为 |
| --- | --- | --- |
| **拦截（Block）** | 即将 commit / 发送 / 粘贴 | 停下外发；提议脱敏后重试 |
| **脱敏（Redact）** | 用户要求分享片段 | 把匹配换成 `[REDACTED:<类别>]` 并展示脱敏后文本 |
| **隔离（Quarantine）** | diff 含密钥 | 取消暂存或排除那些 hunk；与 **git-guard** 搭配 |

```text
SECRET SCRUB
────────────
Action blocked: <commit | PR 正文 | 发聊天 | 网页粘贴>
Hits:   1) <类别> 于 <路径或字段>:<行或区间>
        2) …
Next:   脱敏后重试 | 拒绝（中止） | 修改 <排除什么>
────────────
```

## 测试与演示夹具

仓库与演示里只用一眼假的样本：

```text
sk-test-EXAMPLEKEY000
ghp_xxxEXAMPLE
AKIAEXAMPLE000000000
xoxb-example-token
-----BEGIN EXAMPLE PRIVATE KEY-----
password=EXAMPLE_ONLY
postgres://user:EXAMPLE@localhost:5432/app
```

绝不为了「真实感」提交活凭证。

## 与套件其他技能的配合

- **git-guard** —— 路径级拦 `.env` / 密钥；secret-scrub 补充对 hunk 与提交说明的内容级扫描。
- **permission-gate** —— 脱敏失败在出网前表现为需确认 / 拦截卡。
- **money-denylist** —— 支付工具可能返回打码 PAN；若仍有原始卡字段，照样脱敏。

## 反模式

- 引用密钥「好让用户看到找到了什么」
- Base64 包一层就宣称已脱敏
- 只扫文件名，忽略 PR 描述或聊天草稿
- 不告知人就自动改写历史清密钥

## 最小演示提示（假想）

```text
Agent 即将提交含 ghp_xxxEXAMPLE 的文件
→ SECRET SCRUB 拦截；在 path:line 报类别「疑似 GitHub token」
→ 提供 脱敏后重试 | 拒绝 | 修改 —— 卡上除文档夹具外不再打印 ghp_xxxEXAMPLE
```

见 `demos/scenarios/04-secret-scrub.md`。
