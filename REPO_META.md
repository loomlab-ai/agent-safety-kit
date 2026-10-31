# Suggested GitHub repository metadata

Packaging record for the public launch. This file does not change visibility, the homepage, or the live About field and topics. See [.github/PUBLISH_CHECKLIST.md](.github/PUBLISH_CHECKLIST.md). After merge, the coordinator sets About and topics. Do not call the GitHub API from the docs change.

## About (description)

160 characters or fewer. Exact string for the GitHub About field (116 characters):

```
Ask-before-act Agent Skills: pause before force-push, rm -rf, spend, or secret leaks. Soft HITL for Cursor / Claude.
```

## Website

Leave empty until GitHub Pages loads `https://loomlab-ai.github.io/agent-safety-kit/`. Then:

```bash
gh repo edit --homepage https://loomlab-ai.github.io/agent-safety-kit/
```

## Topics

Set these eight. Remove `llm` and `ai-agents` if they are still on the repo.

```
agent-skills
hitl
safety
permission-gate
git-guard
secret-scrub
cursor
claude
```

## Visibility

Visibility stays unchanged in this docs change. When the coordinator publishes:

```bash
gh repo edit --visibility public
```

---

## 中文

# GitHub 仓库元信息

公开上架的打包记录。这里不改可见性、homepage，也不改线上 About 和 topics。见 [.github/PUBLISH_CHECKLIST.md](.github/PUBLISH_CHECKLIST.md)。合并后由协调人设置 About 和 topics。文档改动不要调用 GitHub API。

## About（简介）

不超过 160 个字符。GitHub About 只用下面这一句英文（116 个字符）：

```
Ask-before-act Agent Skills: pause before force-push, rm -rf, spend, or secret leaks. Soft HITL for Cursor / Claude.
```

## 网站

在 GitHub Pages 能打开 `https://loomlab-ai.github.io/agent-safety-kit/` 之前保持留空。之后执行：

```bash
gh repo edit --homepage https://loomlab-ai.github.io/agent-safety-kit/
```

## Topics

设成下面八个。如果仓库上还有 `llm` 和 `ai-agents`，去掉。

```
agent-skills
hitl
safety
permission-gate
git-guard
secret-scrub
cursor
claude
```

## 可见性

这次文档改动不改可见性。协调人公开时执行：

```bash
gh repo edit --visibility public
```
