# Publish checklist — agent-safety-kit

Report from the `github-launch` skill. Status is `pass`, `fail`, or `skip`. This file does not block `git push`.

| Item | Status | Note |
| --- | --- | --- |
| Intake recorded (kind, audience, languages, owner) | pass | Agent skill kit; local/IDE Markdown skill agents (Cursor / Claude-style); Markdown/docs; owner loomlab-ai; copyright LofiSu |
| Secrets scan | pass | No live credentials or secret files; fixture rows in skills/secret-scrub/SKILL.md document detector categories and were ignored |
| Personal data scan | pass | No phone, home address, private email, or home-directory path; public contacts are only the ones named for this launch |
| Dead demos removed or kept on purpose | pass | Kept demos/ and findings-example.json; README still presents them; hosts and tokens are fixtures |
| License headers match LICENSE, or none required | pass | No SPDX-License-Identifier headers in the tree; MIT does not require per-file headers |
| `.gitignore` covers secrets and build output | pass | Added .env, .env.* (keep .env.example), *.pem, *.key, id_rsa, id_ed25519, credentials.json, .DS_Store, and build dirs |
| Large binaries reviewed | pass | No tracked file is 1 MB or larger |
| LICENSE present; SPDX matches the README | pass | Existing MIT LICENSE kept unchanged; README SPDX id is MIT; copyright LofiSu |
| `README.md` and `README.zh-CN.md` share structure and facts | pass | Same heading order, commands, flags, and URLs; Chinese file uses the required headings |
| No double-brace uppercase placeholders left in published files | pass | Placeholder scan of README files, LICENSE, docs/, and this checklist is clean |
| About description is 160 characters or fewer | pass | Docs-first Agent Skills kit: HITL permission gate, git-guard, money denylist, and secret-scrub for Cursor / Claude-style agents. (128) |
| 5–10 topics | pass | agent-skills, hitl, safety, permission-gate, git-guard, secret-scrub, llm, cursor, claude, ai-agents |
| CI workflow matches the stack | pass | Left .github/workflows/ci.yml; it checks README, LICENSE, the four skills, the findings schema, and the demo |
| Pages decision recorded; the agent did not enable Pages | pass | Added docs/index.html and empty docs/.nojekyll; did not call the Pages API; no docs/index.md |
| Homepage URL recorded; set only after the site loads | pass | Proposed https://loomlab-ai.github.io/agent-safety-kit/ ; homepage left unset |
| `llms.txt` written; FAQ and product noun present | skip | user declined llms.txt |
| Push | pass | Branch cursor/github-launch-packaging-4b40; normal push only; no force-push and no history rewrite; visibility unchanged |

## Coordinator

This file does not install a hook and does not reject `git push`.

- About (128 characters), already on the private repo; this run did not call `gh repo edit --description`: `Docs-first Agent Skills kit: HITL permission gate, git-guard, money denylist, and secret-scrub for Cursor / Claude-style agents.`
- Topics, already on the repo; this run did not call `gh repo edit --add-topic`: `agent-skills`, `hitl`, `safety`, `permission-gate`, `git-guard`, `secret-scrub`, `llm`, `cursor`, `claude`, `ai-agents`.
- Homepage stays unset until `https://loomlab-ai.github.io/agent-safety-kit/` loads. Then: `gh repo edit --homepage https://loomlab-ai.github.io/agent-safety-kit/`
- Visibility stays private here. After merge: `gh repo edit --visibility public`
- Pages, in the browser: Repository → Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch `main`, folder `/docs` → Save. This run did not call the Pages API.
