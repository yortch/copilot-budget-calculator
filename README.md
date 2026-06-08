# GitHub Copilot Budget Sizing Calculator

Interactive single-page calculator for sizing GitHub Copilot Business and Enterprise budgets under the usage-based billing model (AI Credits). Use it to help customers set [User-Level Budgets (ULBs)](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing#user-level-budget) and [Enterprise budgets](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing#enterprise-budget) that maximize their promo credits without exceeding a desired spend ceiling.

## Quick Start

1. Open `index.html` in any modern browser — no server or internet connection required.
2. Enter the customer's **Number of Copilot Business seats** and **Number of Copilot Enterprise seats**. **Total Monthly License Fees** are calculated automatically.
3. Enter the customer's **Total Customer Budget** — the total dollar amount they are willing to spend per month, including license fees.
4. Set the **Budget set date** (defaults to the first day of the current month). Budgets are effective from the day they are set and [cannot look back](https://docs.github.com/en/billing/concepts/budgets-and-alerts#your-first-billing-cycle-after-creating-a-budget), so a mid-month start only covers the remaining days of that first month.
5. Pick a **ULB strategy** (Limited / Conservative / Balanced / Moderate / Aggressive). **Balanced (2.0×)** is the default starting point.
6. Compare the **Promo Period** (Jun 1 – Sep 1, 2026) and **Post-Promo Period** (after Sep 1, 2026) columns to see how costs shift when the promo ends.
7. If the budget set date is not the first of the month, the **Prorated configuration** section appears with a date-sensitive first-month enterprise budget.
8. Use the recommended **Universal ULB** and **Enterprise budget** values when configuring GitHub Enterprise admin settings.

## Key Concepts

### The Four Budget Controls

| Control | What it caps | When active | Hard stop? |
|---------|--------------|-------------|------------|
| Universal ULB | Each user's total AI credit consumption | Always (pool + metered) | Always |
| Individual ULB | A specific user's consumption (overrides universal) | Always | Always |
| Cost center budget | A team's metered charges | Metered phase only | Only if "Stop usage" enabled |
| Enterprise budget | Total enterprise metered charges | Metered phase only | Only if "Stop usage" enabled |

### How Billing Flows

1. **User-level budget check** — if the user has exceeded their ULB, block immediately (hard stop).
2. **Shared pool check** — if the pool has credits, serve from the pool at no extra cost.
3. **Metered phase** — once the pool is exhausted, check cost center budget (if applicable), then enterprise budget. Blocks usage only if "Stop usage when budget limit is reached" is enabled.

### Lowest Remaining Headroom Wins

A user is blocked by whichever budget runs out first — not their personal ULB. If your ULBs collectively allow more consumption than your pool plus enterprise budget can cover, users will be blocked before reaching their personal limits.

### Total Customer Budget vs. Enterprise Budget

The calculator works from a **Total Customer Budget** — the combined ceiling for license fees *plus* metered overage. The enterprise budget entered in GitHub admin settings is derived as:

```
Enterprise budget = min(max(0, Total Customer Budget − License fees), Max overage exposure)
```

If the Total Customer Budget does not cover the monthly license fees, the enterprise budget is shown as **N/A**. Do **not** set the enterprise budget to $0 in that case — a $0 budget blocks all metered usage immediately.

## The Math

```
Pool ($)              = Business seats × Business credits × $0.01
                      + Enterprise seats × Enterprise credits × $0.01

License fees ($)      = Business seats × $19
                      + Enterprise seats × $39

Per-license share     = Pool ($) / total seats

Universal ULB ($/user) = Per-license share × ULB multiplier

Available avg ULB     = Universal ULB / $0.01  (credits/user ceiling)

Max collective spend  = Universal ULB × total seats

Max overage exposure  = max(0, Max collective − Pool)

Enterprise budget     = min(max(0, Total Customer Budget − License fees), Max overage exposure)

Maximum monthly bill  = License fees + Enterprise budget

Prorated ent. budget  = Enterprise budget × (days remaining in month / days in month)
```

## ULB Strategies

| Strategy | Multiplier | Description |
|----------|------------|-------------|
| Limited | 1.0× | ULB equals the blended per-license pool share — **not recommended**; heavy users get blocked even while the pool has unspent capacity from lighter users |
| Conservative | 1.5× | Modest headroom above the pool share |
| **Balanced** | **2.0×** | **Default — good starting point for most teams** |
| Moderate | 2.5× | More headroom; enterprise budget governs total spend |
| Aggressive | 3.0× | Maximum headroom; enterprise budget is the primary governor |

## Constants

| Plan | Cost/user/month | Promo credits (Jun 1 – Sep 1, 2026) | Post-promo credits |
|------|-----------------|--------------------------------------|--------------------|
| Copilot Business | $19 | 3,000 | 1,900 |
| Copilot Enterprise | $39 | 7,000 | 3,900 |

Credit price is fixed at **$0.01 per AI credit**.

> **Note:** These constants are shown for reference. Always confirm current values against the latest [GitHub Copilot billing documentation](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing) before relying on them.

## Power Users — The Excluded Cost Center Pattern

There is **no way to exclude an individual user** from the enterprise budget. Setting a high ULB for a top developer does not let them bypass the enterprise cap — the lowest-headroom rule still applies.

To give Power Users true independent spending authority:

1. Create a cost center for the Power User (or their team).
2. Set the cost center budget to their desired cap.
3. **Enable cost center exclusion** so their charges do not roll up to the enterprise budget.
4. Set their individual ULB override accordingly.

After exclusion, the user is governed only by their cost center budget. The enterprise cap will not block them, and their spend will not consume enterprise headroom that other teams rely on. Their cost center budget is **additive** to the bill ceiling.

## Setup Checklist (GitHub Enterprise admin)

- [ ] Enable the **AI credit paid usage** policy — required for any metered usage above the pool.
- [ ] Set the **Universal ULB** to the recommended value from the calculator.
- [ ] Set **Individual ULB overrides** for any named Power Users.
- [ ] Set the **[Enterprise budget](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing#enterprise-budget)** to the recommended value (use the prorated value if setting mid-month).
- [ ] **Enable "Stop usage when budget limit is reached"** on the enterprise budget — **off by default**. Without it, charges accrue past the cap.
- [ ] (Optional) Create cost centers for Power Users needing independent authority. Enable cost center exclusion.
- [ ] Plan a re-tune at the promo → post-promo transition (Sept 1, 2026). The same ULB will produce more overage when the pool shrinks.

## Key Rules to Remember

- **Pool is shared** across the entire enterprise — not per user, not per license type.
- **ULBs are always hard stops** — there is no "continue past limit" toggle for ULBs.
- **Enterprise and cost center budgets only apply after the pool is exhausted** (metered phase).
- **Cost center exclusion is the only way** to give a team or user spending authority beyond the enterprise cap.
- **A $0 budget at any level blocks usage immediately** for the users it applies to. If the Total Customer Budget is below license fees, do not set the enterprise budget to $0 — mark it as N/A.
- **The enterprise budget is not a total monthly cap** — it only caps metered overage. Max bill = license fees + enterprise budget.
- **[Budgets cannot look back](https://docs.github.com/en/billing/concepts/budgets-and-alerts#your-first-billing-cycle-after-creating-a-budget)** — they are effective from the day they are set. A mid-month start only governs the remaining days of that first month.
- **"Available average ULB" is a ceiling, not a guarantee** — if the enterprise budget ceiling is hit first, users are blocked before reaching their personal ULB (lowest headroom wins).

## References

- [Budgets for usage-based billing — GitHub Docs](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing)
  - [Enterprise budget](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing#enterprise-budget)
  - [User-level budget (ULB)](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing#user-level-budget)
- [Your first billing cycle after creating a budget — GitHub Docs](https://docs.github.com/en/billing/concepts/budgets-and-alerts#your-first-billing-cycle-after-creating-a-budget)
- [Manage billing for enterprise — GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-billing-for-enterprise)

## Disclaimer

⚠ This tool is a sizing aid you can walk through with customers. Pricing, credit allowances, promo terms, and feature availability change over time and may differ by region or contract. **Always verify every number against the latest official [GitHub Copilot billing documentation](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing) before relying on it. This tool is a sizing aid, not a quote.**
