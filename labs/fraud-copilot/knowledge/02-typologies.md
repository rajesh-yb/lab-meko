# Northstar Payments — Fraud typologies (lab excerpt)

Short reference for investigators. Match the alert to a typology before writing the case note.

## T-DUP — Duplicate invoice / double presentment

- Same merchant (or vendor name variant) and same amount as a prior settled payment inside 90 days.
- Often an accounts-payable resubmit, not a stolen card.
- Default action: HOLD 48h per hold-vs-reject policy.

## T-ATO — Account takeover

- New device or impossible travel, recent credential reset, then payout or beneficiary change.
- Default action: REJECT and freeze. ATO desk owns the case after freeze.

## T-FF — Friendly fraud / first-party dispute

- Cardholder claims “not me” but device, IP geo, and enrollment data align.
- First claim under $500: HOLD and request evidence.
- Repeat claims on a merchant that later wins representment: CLEAR as noisy FP.

## T-MULE — Money mule / pass-through

- New account, rapid inbound then outbound, unrelated counterparties.
- Out of scope for this lab. If suspected, HOLD and escalate; do not CLEAR.
