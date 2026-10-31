# Suggested GitHub repository metadata

Packaging record for the public launch. This change does not set visibility or the homepage. See [.github/PUBLISH_CHECKLIST.md](.github/PUBLISH_CHECKLIST.md).

## About (description)

160 characters or fewer. This text is already the repository description (128 characters):

```
Docs-first Agent Skills kit: HITL permission gate, git-guard, money denylist, and secret-scrub for Cursor / Claude-style agents.
```

## Website

Leave empty until GitHub Pages loads `https://loomlab-ai.github.io/agent-safety-kit/`. Then:

```bash
gh repo edit --homepage https://loomlab-ai.github.io/agent-safety-kit/
```

## Topics

```
agent-skills
hitl
safety
permission-gate
git-guard
secret-scrub
llm
cursor
claude
ai-agents
```

## Visibility

The repository stays private in this packaging change. After merge, the coordinator runs:

```bash
gh repo edit --visibility public
```

---

## 中文

# GitHub 仓库元信息

公开上架的打包记录。本变更不设置可见性，也不设置 homepage。见 [.github/PUBLISH_CHECKLIST.md](.github/PUBLISH_CHECKLIST.md)。

## About（简介）

不超过 160 个字符。下面这段已经是仓库简介（128 个字符）：

```
Docs-first Agent Skills kit: HITL permission gate, git-guard, money denylist, and secret-scrub for Cursor / Claude-style agents.
```

## 网站

在 GitHub Pages 能打开 `https://loomlab-ai.github.io/agent-safety-kit/` 之前保持留空。之后执行：

```bash
gh repo edit --homepage https://loomlab-ai.github.io/agent-safety-kit/
```

## Topics

与英文同一组：

```
agent-skills
hitl
safety
permission-gate
git-guard
secret-scrub
llm
cursor
claude
ai-agents
```

## 可见性

本次打包保持私有。合并后由协调人执行：

```bash
gh repo edit --visibility public
```
