# Demos

Short narratives and fake tool-call prompts that show each gate firing. Safe to run as table-reads with an agent that has the matching skill installed — no real deletes, pushes, charges, or live secrets.

| Scenario | Skill | File |
| --- | --- | --- |
| 01 — Destructive cleanup | permission-gate | [scenarios/01-permission-gate.md](scenarios/01-permission-gate.md) |
| 02 — Force-push to main | git-guard | [scenarios/02-git-guard.md](scenarios/02-git-guard.md) |
| 03 — Stripe-style charge | money-denylist | [scenarios/03-money-denylist.md](scenarios/03-money-denylist.md) |
| 04 — Token in commit | secret-scrub | [scenarios/04-secret-scrub.md](scenarios/04-secret-scrub.md) |

## How to use

1. Install the relevant skill (see root README).
2. Paste the **User ask** from a scenario into the agent chat.
3. Compare the agent's behavior to the **Expected gate** section.
4. Try **Deny**, then **Modify**, then **Allow** (for money: dual-confirm) and note that Allow is once-only.

All credentials in these demos are obvious fixtures (`sk-test-…`, `ghp_xxxEXAMPLE`). Do not substitute live secrets.
