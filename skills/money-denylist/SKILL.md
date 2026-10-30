---
name: money-denylist
description: Hard-stop or dual-confirm before shopping, checkout, transfers, Sponsors, wallets, or payment APIs; optional merchant allowlist; never move money silently.
when-to-use: Before any purchase, cart checkout, money transfer, tip/Sponsor, wallet send, subscription start that bills, or call to a payment API. Also when a tool result offers a "Pay" / "Buy now" / "Confirm order" step.
---

# money-denylist

Money never moves without an explicit human yes — preferably two.

## Purpose

Agents that can shop, tip, or call payment APIs are one mis-click from real charges. This skill defines a **denylist** of money-moving actions, an optional **merchant allowlist**, and a **dual-confirm** protocol. Silent or "helpful" purchases are forbidden.

## Denylist (always gate)

Any of the following is a money-moving action and **must not run** without confirmation:

| Class | Examples |
| --- | --- |
| Shopping / checkout | Place order, confirm cart, one-click buy, "Buy now" |
| Transfers | Bank/wallet send, P2P payment, wire, crypto transfer |
| Platform money | GitHub Sponsors, Patreon, Ko-fi-style tips, in-app tips |
| Payment APIs | Stripe / PayPal / Square / Adyen charge, capture, payout |
| Billing changes | Start paid subscription, upgrade plan, add paid seat (when it triggers charge) |
| Refunds / chargebacks initiated by the agent | Only with explicit user request + confirm |

Reading prices, filling a cart the user will check out themselves, or drafting a wishlist can be allowed **if** no charge API is invoked. Still announce that the cart was filled and that checkout was **not** completed.

## Optional merchant allowlist

Maintain a small allowlist of merchants or SKUs the user has pre-approved (name + domain + optional max amount). Behavior:

| Situation | Required confirmations |
| --- | --- |
| Merchant **not** on allowlist | **Dual-confirm** (see below) |
| Merchant on allowlist, amount ≤ max | Single **Allow** on a money card |
| Merchant on allowlist, amount > max | Dual-confirm |
| Allowlist empty / unset | Treat every charge as dual-confirm |

Allowlist entries are configuration the human owns. The agent must not add merchants to the allowlist without a separate explicit request.

## Money card (first confirm)

```text
MONEY DENYLIST
──────────────
Action:   <charge | transfer | sponsor | subscribe | …>
Merchant: <name / domain / "unknown">
Amount:   <currency + number, or "unknown — will read back at checkout">
Allowlist:<yes (within max) | yes (over max) | no>
──────────────
Reply: Allow | Deny | Modify <note>
```

## Dual-confirm protocol

When dual-confirm is required:

1. Show the **MONEY DENYLIST** card. Wait for **Allow**.
2. Re-state amount + merchant in one line: `Confirm again: charge <amount> to <merchant>?`
3. Wait for a second explicit **Allow** / **Yes** / **Confirm**.
4. Only then invoke the payment tool or click checkout.

**Deny** at either step aborts permanently for this attempt. **Modify** restarts from step 1 with new parameters.

## Hard rules

1. **Never** silent money moves — no "I went ahead and ordered it."
2. **Never** use a stored card without the user's vault/purchase flow and this gate.
3. If amount is unknown until the last page, stop at that page, read amount back on the card, then confirm.
4. Tips and Sponsors count as money, even if small.
5. "The user said buy milk" authorizes shopping **intent**, not a blank check — still show amount before charge.

## Anti-patterns

- Completing checkout because the cart was "obvious"
- Treating a previous purchase as standing approval for a reorder without a new card
- Rounding or omitting currency / tax / shipping on the card
- Echoing full card numbers (last4 only, if needed)

## Minimal demo prompt (fake)

```text
Agent plans: call stripe.charges.create amount=4999 currency=usd
→ Show MONEY DENYLIST card; merchant not allowlisted → dual-confirm
→ Wait for Allow, then second Confirm, else stop
```

See `demos/scenarios/03-money-denylist.md`.
