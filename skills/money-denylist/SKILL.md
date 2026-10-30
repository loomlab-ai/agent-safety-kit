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

---

## 中文

**一句话：** 购物、结账、转账、Sponsors、钱包、支付 API 前硬拦或二次确认；可选商户白名单；绝不静默动钱。

# money-denylist

没有人的明确同意 —— 最好是两次 —— 钱不能动。

## 用途

能购物、打赏或调支付 API 的 Agent，离真扣款只差一次误点。本技能定义资金动作 **黑名单**、可选 **商户白名单**，以及 **二次确认** 协议。静默或「好心代买」一律禁止。

## 黑名单（一律闸）

下列任一属资金动作，**未经确认不得执行**：

| 类别 | 示例 |
| --- | --- |
| 购物 / 结账 | 下单、确认购物车、一键购买、「立即购买」 |
| 转账 | 银行/钱包转出、P2P、电汇、加密货币转账 |
| 平台打赏 | GitHub Sponsors、Patreon、Ko-fi 类打赏、应用内打赏 |
| 支付 API | Stripe / PayPal / Square / Adyen 的 charge、capture、payout |
| 计费变更 | 开通付费订阅、升级套餐、增加付费席位（会触发扣款时） |
| Agent 发起的退款 / 拒付 | 仅在用户明确要求 + 确认后 |

只读价格、帮用户填好购物车由其自行结账、或草拟心愿单 —— **在不调用扣款 API 的前提下** 可以做。仍须声明：购物车已填好，**尚未**结账。

## 可选商户白名单

维护用户预批准的少量商户或 SKU（名称 + 域名 + 可选金额上限）。行为：

| 情形 | 所需确认 |
| --- | --- |
| 商户 **不在** 白名单 | **二次确认**（见下） |
| 在白名单且金额 ≤ 上限 | 资金卡上单次 **允许** |
| 在白名单但金额 > 上限 | 二次确认 |
| 白名单为空 / 未设置 | 每笔扣款都走二次确认 |

白名单由人配置。未经单独明确请求，Agent 不得自行把商户加进白名单。

## 资金卡（第一次确认）

```text
MONEY DENYLIST
──────────────
Action:   <扣款 | 转账 | 赞助 | 订阅 | …>
Merchant: <名称 / 域名 / "unknown">
Amount:   <币种 + 数字，或「未知 — 结账页回读」>
Allowlist:<是（未超上限） | 是（已超上限） | 否>
──────────────
Reply: Allow | Deny | Modify <说明>
```

## 二次确认协议

需要二次确认时：

1. 出示 **MONEY DENYLIST** 卡。等待 **允许**。
2. 用一行重述金额 + 商户：`再次确认：向 <商户> 扣款 <金额>？`
3. 等待第二次明确的 **允许 / Yes / Confirm**。
4. 然后才可调用支付工具或点击结账。

任一步 **拒绝** 即永久中止本次尝试。**修改** 则带着新参数从第 1 步重来。

## 硬性规则

1. **绝不** 静默动钱 —— 禁止「我已经帮你下单了」。
2. **绝不** 在没有用户保险库/购买流程 + 本闸门的情况下使用已存卡片。
3. 金额要到最后一页才知道时：停在那一页，把金额读回卡上，再确认。
4. 打赏与 Sponsors 也算钱，哪怕金额很小。
5. 「用户说买牛奶」只授权购物 **意图**，不是空白支票 —— 扣款前仍须出示金额。

## 反模式

- 因为购物车「很明显」就直接结账
- 把上次购买当成再下单的长期授权而不出新卡
- 在卡上四舍五入或省略币种 / 税 / 运费
- 回显完整卡号（需要时最多 last4）

## 最小演示提示（假想）

```text
Agent 计划：调用 stripe.charges.create amount=4999 currency=usd
→ 出示 MONEY DENYLIST 卡；商户不在白名单 → 二次确认
→ 等待 Allow，再等第二次 Confirm，否则停止
```

见 `demos/scenarios/03-money-denylist.md`。
