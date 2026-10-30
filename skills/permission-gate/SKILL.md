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
