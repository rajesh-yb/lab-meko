# Northstar Payments — Hold vs reject policy

Internal playbook for the risk operations team. Agents must retrieve this before recommending an action. Do not invent a stricter or looser rule.

Effective: 2026-04-01. Owner: Risk Ops.

## Decision table

| Situation | Action | Window / notes |
|---|---|---|
| Same vendor + same amount + invoice or payment reference within 90 days | **HOLD** | Auto-resolves in 48 hours if no matching payment clears. Rejection requires a human reviewer. |
| Confirmed account takeover: new device **and** password reset within 24h **and** payout destination changed | **REJECT** and freeze payouts | Escalate to ATO desk. Do not HOLD. |
| First-party / friendly-fraud claim under $500, first occurrence, device and geo match enrollment | **HOLD** and request evidence | Do not reject on the first claim. |
| Same merchant has **two or more chargebacks later won by the merchant**, device/geo consistent | **CLEAR** (low priority) | Treat as noisy false positive unless new ATO signals appear. |
| Amount ≥ $5,000 with any mismatch of device, geo, or payout change | **HOLD** and escalate | Dual control. |

## Hard rules

1. Duplicate-invoice holds are never converted to rejects by an agent. A human must reject.
2. Cite the row you used. If two rows could apply, take the **more severe** action and say so.
3. Store the case id, merchant, amount, and chosen action. Do not store PAN, CVV, full account numbers, or government IDs.
4. If policy and a prior memory conflict, follow **this policy**, then note the conflict for a reviewer.
