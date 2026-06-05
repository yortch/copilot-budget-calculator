# GitHub Copilot Budget Sizing Tools

Tools and references for sizing GitHub Copilot Business and Enterprise budgets under the new usage-based billing model (AI Credits). Use these to help customers set User-Level Budgets (ULBs) and Enterprise budgets that maximize their promo credits without exceeding a desired spend ceiling.

## Contents

| File | Description |
|------|-------------|
| `copilot-budget-calculator.html` | Single-page interactive calculator. Open in any browser. Self-contained, no internet required. |
| `Copilot_Budget_Sizing_Worksheet.xlsx` | Excel sizing worksheet with four sheets: Sizing Calculator, Scenarios, Power Users, Setup Notes. |

## Quick Start

1. Open `copilot-budget-calculator.html` in any modern browser.
2. Enter the customer's Business and Enterprise seat counts. Total monthly license fees are calculated automatically.
3. Enter the customer's **Total Customer Budget** — the total dollar amount they're willing to spend per month, including license fees.
4. Set the **Budget set date** (defaults to the first of the current month). Budgets are effective from the day they're set and cannot look back, so a mid-month start only covers the remaining days of that first month.
5. Pick a ULB strategy (Conservative / Balanced / Moderate / Aggressive).
6. Compare the **Promo Period** (Jun 1 – Sep 1, 2026) and **Post-Promo Period** columns to see how their costs will shift when the promo ends.
7. Use the recommended Universal ULB and Enterprise Budget values when configuring GitHub admin settings.

For deeper sensitivity analysis or to model excluded cost centers for power users, use the Excel worksheet.

## Key Concepts

### The Four Budget Controls

| Control | What it caps | When active | Hard stop? |
|---------|--------------|-------------|------------|
| Universal ULB | Each user's total AI credit consumption | Always (pool + metered) | Always |
| Individual ULB | A specific user's consumption (overrides universal) | Always | Always |
| Cost center budget | A team's metered charges | Metered phase only | Only if "Stop usage" enabled |
| Enterprise budget | Total enterprise metered charges | Metered phase only | Only if "Stop usage" enabled |

### How Billing Flows

1. **User-level budget check** — if the user exceeded their ULB, block immediately (hard stop).
2. **Shared pool check** — if pool has credits, serve from the pool at no extra cost.
3. **Metered phase** — once the pool is exhausted, check cost center budget (if applicable), then enterprise budget. Block only if "Stop usage when budget limit is reached" is enabled.

### Lowest Remaining Headroom Wins

A user is blocked by whichever budget runs out first — not their personal ULB. If your ULBs collectively allow more consumption than your pool plus enterprise budget can cover, users will be blocked before reaching their personal limits.

## The Math

```
Pool ($)              = Business seats × Business credits × $0.01
                      + Enterprise seats × Enterprise credits × $0.01

License fees ($)      = Business seats × $19
                      + Enterprise seats × $39

Per-license share     = Pool / total seats

Universal ULB         = Per-license share × ULB multiplier

Max collective spend  = Universal ULB × total seats

Max overage exposure  = max(0, Max collective − Pool)

Enterprise budget     = min(max(0, Total Customer Budget − License fees), Max overage exposure)

Maximum monthly bill  = License fees + Enterprise budget
```

## Constants

| Plan | Cost/user/month | Promo credits | Post-promo credits |
|------|-----------------|---------------|--------------------|
| Copilot Business | $19 | 3,000 | 1,500 |
| Copilot Enterprise | $39 | 7,000 | 3,500 |

Credit price is fixed at **$0.01 per AI credit**. The promo period runs **June 1 through September 1, 2026**.

> **Note:** These constants and post-promo allowances are shown for reference. Confirm the current GitHub-published values against the latest [GitHub Copilot documentation](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing).

## Power Users — The Excluded Cost Center Pattern

There is **no way to exclude an individual user** from the enterprise budget. Setting a high ULB for a top developer does not let them bypass the enterprise cap — the lowest-headroom rule still applies.

To give power users true independent spending authority:

1. Create a cost center for the power user (or team).
2. Set the cost center budget to their desired cap.
3. **Enable cost center exclusion** so their charges do not roll up to the enterprise budget.
4. Set their individual ULB override accordingly.

After exclusion, the user is governed only by their cost center budget. The enterprise cap won't block them, and their spend won't eat into enterprise headroom that other teams rely on. Their cost center budget is **additive** to the bill ceiling.

## Setup Checklist (GitHub Enterprise admin)

- [ ] Enable the **AI credit paid usage** policy (required for any metered usage above the pool).
- [ ] Set the **Universal ULB** to the recommended value from the calculator.
- [ ] Set **Individual ULB overrides** for any named power users.
- [ ] Set the **Enterprise budget** to the recommended value.
- [ ] **Enable "Stop usage when budget limit is reached"** on the enterprise budget — **off by default**. Without it, charges accrue past the cap.
- [ ] (Optional) Create cost centers for any teams or power users needing independent authority. Enable cost center exclusion.
- [ ] Plan a re-tune at the promo → post-promo transition. The same ULB will produce more overage when the pool shrinks.

## Key Rules to Remember

- **Pool is shared** across the entire enterprise — not per user, not per license type.
- **ULBs are always hard stops** — there is no "continue past limit" toggle for them.
- **Enterprise and cost center budgets only apply after the pool is exhausted** (metered phase).
- **Cost center exclusion is the only way** to give a team or user spending authority beyond the enterprise cap.
- **A $0 budget at any level blocks usage immediately** for the users it applies to.
- **The enterprise budget is not a total monthly cap** — it only caps metered overage. Max bill = license fees + enterprise budget.
- **Budgets cannot look back** — they are effective from the day they are set. A mid-month start only governs the remaining days of that first month; earlier usage is not covered.

## References

- [Budgets for usage-based billing — GitHub Docs](https://docs.github.com/en/copilot/concepts/billing/budgets-for-usage-based-billing)
- [Getting started with budget controls — GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-billing-for-enterprise)
- [Optimizing your budget configuration — GitHub Docs](https://docs.github.com/en/copilot/how-tos/administer-copilot/manage-for-enterprise/manage-billing-for-enterprise/optimize-budget-configuration)

## Disclaimer

These tools are provided as sizing aids you can walk through with customers. Pricing, credit allowances, and feature availability change over time — always verify current values against official GitHub documentation before relying on them. Promo terms in particular are time-limited and may differ by region or contract.
