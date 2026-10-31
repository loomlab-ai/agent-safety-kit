# agent-safety-kit

[English](README.md)

四个 Markdown 技能，给 Cursor 和 Claude 用。force-push、`rm -rf`、花钱、把密钥发出去，这些事它会先问你。

只靠 SKILL.md 约定；模型不听就拦不住。

## 技能

| 技能 | 干什么 |
| --- | --- |
| [permission-gate](skills/permission-gate/SKILL.md) | 删除、危险 shell、访问没提过的主机之前，先给你一张卡：Allow / Deny / Modify |
| [git-guard](skills/git-guard/SKILL.md) | force-push、改受保护分支、提交密钥文件、删远端分支，先问一声 |
| [money-denylist](skills/money-denylist/SKILL.md) | 下单、转账、调支付 API 之前先停（hard-stop），或再确认一次（dual-confirm） |
| [secret-scrub](skills/secret-scrub/SKILL.md) | 扫 commit、PR 正文、聊天、粘贴。中了就打码或拦住，密钥原文不打出来 |

装一个也行，四个一起也行。**Allow** 只管这一下。下一步还得再问。

## 安装

```bash
git clone https://github.com/loomlab-ai/agent-safety-kit.git
cd agent-safety-kit
mkdir -p .cursor/skills
cp -R skills/permission-gate skills/git-guard skills/money-denylist skills/secret-scrub .cursor/skills/
```

上面那条装进当前项目。每个项目都用，把目录换成 `~/.cursor/skills/`。

Claude Code、OpenCode，或者别的自己带 skills 目录的：四个文件夹原样放进去。

不想复制就链上：`ln -s`，把 `skills/<name>` 指到 `.cursor/skills/`。

聊天里看不到技能，新开一个对话。

## 试一次

1. 四个都装上。
2. 跟它说：`Clean the build artifacts. Something like rm -rf dist .cache is fine.`
3. 动手删之前，你会看到一张 **PERMISSION GATE** 卡（What / Touches / Risk）。
4. 回 **Deny**。再让它把那两个目录列出来。
5. force-push、假扣款、`ghp_xxxEXAMPLE` 的脚本在 [demos/README.md](demos/README.md)。照着聊就行，token 是假的。

## 什么时候会问

| 你让它做 | 没装 | 装了 |
| --- | --- | --- |
| `rm -rf` 清缓存，或清错目录 | 直接删 | **permission-gate** 出卡：Allow / Deny / Modify |
| `git push --force` 到 `main` | 历史改掉了 | **git-guard** 先问。去掉 `--force`，或者换条分支 |
| 提交 `.env`，或 hunk 里夹着 token | 密钥进了 git | **git-guard** 看路径，**secret-scrub** 看内容 |
| 结账，或 Stripe 扣款 | 钱扣了 | **money-denylist** 要 dual-confirm，不会自己扣 |
| PR 或 Slack 里冒出 API key | 密钥公开了 | **secret-scrub** 打码或拦住，只告诉你是哪一类 |
| 去连一个你没提过的主机 | 请求发出去了 | **permission-gate** 先问再连 |
| 删远端分支 | 分支没了 | **git-guard** 把 diff 和 ref 拿出来，然后等你 |

## 配置

没有环境变量，没有命令行参数，没有配置文件。

商户白名单归你管，见 [skills/money-denylist/SKILL.md](skills/money-denylist/SKILL.md)。agent 不能自己加商户。名单空着，每笔都要 dual-confirm。仓库里没有这份名单。

别的工具要查这次决定，就存成 JSON。[`schemas/findings.schema.json`](schemas/findings.schema.json) 是格式，[`demos/findings-example.json`](demos/findings-example.json) 是一份例子：`rm -rf` 被 Deny，密钥命中但 token 没写进去，一条 git 警告，一条扣款记录关掉了。

| 字段 | 什么意思 |
| --- | --- |
| `id` | 这个文件里的编号 |
| `skill` | `permission-gate`、`git-guard`、`money-denylist`、`secret-scrub` |
| `severity` | `info` 记一笔，`warn` 先看一眼，`block` 别做 |
| `status` | `confirmed` 已经定了，`needs_review` 还等你，`dismissed` 关掉了 |
| `evidence` | 发生了什么。有路径或类别就附上 |
| `recommendation` | 接下来做什么 |

## 许可证

MIT License，SPDX `MIT`。Copyright (c) 2026 LofiSu。全文见 [LICENSE](LICENSE)。

## 联系

- GitHub：[loomlab-ai/agent-safety-kit](https://github.com/loomlab-ai/agent-safety-kit)
- Email：[lofisuchat@gmail.com](mailto:lofisuchat@gmail.com)
- X：[@su_lofi](https://x.com/su_lofi)

About 和 topics 在 [REPO_META.md](REPO_META.md)。
