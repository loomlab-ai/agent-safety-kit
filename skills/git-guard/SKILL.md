---
name: git-guard
description: Confirm or block force-push, protected-branch rewrites, secret commits, and remote/branch deletion; summarize diffs before risky git ops.
when-to-use: Before any non-routine git write — especially push --force, branch -D on remotes, amending pushed history, or commits that may include .env / keys. Also when reviewing a staged diff that looks like it contains secrets.
---

# git-guard

Make destructive and secret-leaking git operations impossible to do silently.

## Purpose

Agents often treat git as "just another shell." Force-pushes, protected-branch rewrites, and accidental secret commits are hard to undo. This skill defines what is **routine** (allowlist) versus **gated**, and how to summarize risk before the human decides.

## Allowlist (no card for routine local work)

These are allowed without a confirmation card when they match the user's current request:

- `git status`, `git diff`, `git log`, `git show`, `git blame`
- `git add` of non-secret paths (see Secret paths below)
- `git commit` of already-reviewed, non-secret staged content
- `git branch` (list / create local), `git checkout` / `git switch` to existing local branches
- `git fetch` (read-only remote sync)
- `git pull` / `git rebase` onto the current tracking branch when the user explicitly asked to update — still prefer a short summary of incoming commits

If any allowlisted command would also stage or commit a secret path, **upgrade to gated**.

## Gated operations (confirmation required)

| Operation | Why it is gated |
| --- | --- |
| `git push --force` / `--force-with-lease` | Rewrites remote history; can erase teammates' work |
| Push or amend on a **protected** branch (`main`, `master`, `release/*`, or repo-configured) | Bypasses normal review |
| Commit or stage of secret-like paths | Leaks credentials into history |
| `git push` that deletes a remote branch (`:branch` or `--delete`) | Drops shared refs |
| `git remote remove` / rewrite of remote URLs | Can redirect push/pull to a hostile host |
| `git filter-branch` / `git filter-repo` / history rewrite tools | Irreversible without backups |
| `git reset --hard` / `git clean -fdx` on a dirty tree the user did not explicitly ask to discard | Data loss |

## Secret paths (always gate before add/commit)

Block or require confirm before staging/committing paths matching:

- `.env`, `.env.*`, `*.pem`, `*.key`, `id_rsa*`, `*.p12`, `*.pfx`
- `credentials.json`, `service-account*.json`, `*secret*`, `*token*` (case-insensitive basename heuristics)
- Known tool credential dirs (e.g. `.aws/credentials`, `.ssh/`) when they appear in the diff

Prefer pairing with **secret-scrub** for content-level scans of staged hunks.

## Diff summary card

Before a gated git op, show:

```text
GIT GUARD
─────────
Command:  <exact git invocation>
Branch:   <local> → <remote/branch if any>
Summary:  <N files, +adds/-dels; call out secret-like paths>
Risk:     <force / protected / secrets / delete remote / history rewrite>
─────────
Reply: Allow | Deny | Modify <note>
```

Rules:

1. Prefer `git diff --stat` (and name secret-like files explicitly).
2. Never paste secret values into the card — file names and categories only.
3. **Allow** applies to this command once. A later force-push needs a new card.
4. **Deny** stops; do not fall back to an equally destructive alternative without a new card.
5. **Modify** examples: drop force flag, push to a feature branch, unstage `.env`, open a PR instead of pushing to `main`.

## Protected branch heuristic

Treat as protected unless the repo config says otherwise:

- `main`, `master`, `trunk`, `production`, `prod`, `release`, `release/*`

If the agent can read branch protection rules from the host (GitHub/GitLab API), prefer those over the heuristic.

## Anti-patterns

- `git push --force` "because rebase looked clean"
- Amending a commit that was already pushed without saying so on the card
- Squashing secrets out of the index but leaving them in earlier commits without warning
- Renaming a secret file so the basename heuristic misses it — still run **secret-scrub** on content

## Minimal demo prompt (fake)

```text
Agent plans: git push --force origin main
→ Show GIT GUARD card with branch, stat summary, risk=force+protected
→ Wait for Allow | Deny | Modify
```

See `demos/scenarios/02-git-guard.md`.

---

## 中文

**一句话：** 强推、改受保护分支、提交密钥路径、删远端引用前必须确认，并先给出 diff 风险摘要。

# git-guard

让破坏性、会泄密的 git 操作无法静默完成。

## 用途

Agent 常把 git 当成「又一条 shell」。强推、改写受保护分支、误提交密钥很难挽回。本技能划分 **常规（白名单）** 与 **需闸门**，并在人决定前汇总风险。

## 白名单（常规本地工作不出卡）

与用户当前请求一致时，下列操作可不出确认卡：

- `git status`、`git diff`、`git log`、`git show`、`git blame`
- 对非密钥路径的 `git add`（见下方密钥路径）
- 对已审阅、无密钥暂存内容的 `git commit`
- `git branch`（列出 / 新建本地）、切到已有本地分支的 `git checkout` / `git switch`
- `git fetch`（只读同步远端）
- 用户明确要求更新时，对当前跟踪分支的 `git pull` / `git rebase` —— 仍建议短摘要即将进来的提交

若白名单命令同时会暂存或提交密钥路径，**升级为需闸门**。

## 需闸门操作（必须确认）

| 操作 | 为何要闸 |
| --- | --- |
| `git push --force` / `--force-with-lease` | 改写远端历史；可能抹掉队友的工作 |
| 向 **受保护** 分支推送或 amend（`main`、`master`、`release/*` 或仓库配置） | 绕过正常评审 |
| 暂存或提交疑似密钥路径 | 把凭证写进历史 |
| 删除远端分支的 `git push`（`:branch` 或 `--delete`） | 丢掉共享引用 |
| `git remote remove` / 改写远端 URL | 可能把推拉导向恶意主机 |
| `git filter-branch` / `git filter-repo` / 历史改写工具 | 无备份则不可逆 |
| 用户未明确要求丢弃时，对脏工作区执行 `git reset --hard` / `git clean -fdx` | 丢数据 |

## 密钥路径（add/commit 前一律闸）

匹配下列路径时，暂存/提交前拦截或要求确认：

- `.env`、`.env.*`、`*.pem`、`*.key`、`id_rsa*`、`*.p12`、`*.pfx`
- `credentials.json`、`service-account*.json`、`*secret*`、`*token*`（文件名不区分大小写的启发式）
- 出现在 diff 中的已知凭证目录（如 `.aws/credentials`、`.ssh/`）

内容级扫描建议与 **secret-scrub** 搭配，扫暂存 hunk。

## Diff 摘要卡

需闸门的 git 操作前出示：

```text
GIT GUARD
─────────
Command:  <完整 git 调用>
Branch:   <本地> → <远端/分支（若有）>
Summary:  <N 个文件，+/- 行；点名疑似密钥路径>
Risk:     <强推 / 受保护 / 密钥 / 删远端 / 改写历史>
─────────
Reply: Allow | Deny | Modify <说明>
```

规则：

1. 优先用 `git diff --stat`（并显式点名疑似密钥文件）。
2. 卡里绝不贴密钥值 —— 只报文件名与类别。
3. **允许** 只对这一次命令生效。之后再强推需新卡。
4. **拒绝** 即停；不得在不出新卡的情况下改用同等破坏力的替代。
5. **修改** 示例：去掉 force、推到功能分支、取消暂存 `.env`、开 PR 而不是直接推 `main`。

## 受保护分支启发式

除非仓库配置另有说明，下列视为受保护：

- `main`、`master`、`trunk`、`production`、`prod`、`release`、`release/*`

若 Agent 能从托管方（GitHub/GitLab API）读到分支保护规则，以规则为准，优先于启发式。

## 反模式

- 「rebase 看起来很干净」就 `git push --force`
- amend 已推送过的提交却不在卡上说明
- 从 index 挤掉密钥却不提醒它们仍在更早提交里
- 改名躲避文件名启发式 —— 仍应对内容跑 **secret-scrub**

## 最小演示提示（假想）

```text
Agent 计划：git push --force origin main
→ 出示含分支、stat 摘要、risk=force+protected 的 GIT GUARD 卡
→ 等待 Allow | Deny | Modify
```

见 `demos/scenarios/02-git-guard.md`。
