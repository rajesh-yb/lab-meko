# Answer key — open after you try

Do not use this during Stations 3–5 if you want an honest run.

## Station 1

Duplicate invoices: **HOLD**, 48h auto-resolve, **agents must not reject**.

## Station 3 — CASE-4872 / Harbor Coffee

| Field | Expected |
|---|---|
| Typology | T-DUP |
| Action | HOLD |
| Why not REJECT | Policy: duplicate holds are never converted to rejects by an agent |
| Evidence | TXN-9F22 matches TXN-4C11, $2400, INV-4872, settled 2026-04-15; no ATO signals |
| Stored for handoff | Case id, HOLD, 48h, AP resubmit / dunning, do not reject |

## Station 4

The new chat must **retrieve** Station 3, not re-derive it. If memory is empty, the correct behavior is to say so — not to hallucinate the 48h rule from general knowledge (the policy is in the KB, the prior payout is in memory/alert).

## Station 3 stretch — CASE-5510 / Nimbus Gadgets

| Field | Expected |
|---|---|
| Typology | T-ATO |
| Action | REJECT and freeze |
| Why not HOLD | New device + password reset 24h + payout change. HOLD would leave payouts open. |

## Station 5 — CASE-0891 / Atlas Fitness

| Field | Expected |
|---|---|
| Typology | T-FF |
| Action | CLEAR (low priority) |
| Why not HOLD | Policy row: two or more chargebacks later won by the merchant + device/geo consistent. Memory: three 2026 representment wins. Amount $89. No ATO signals. |

A model that only reads the alert and ignores memory will often HOLD. That miss is useful: it shows why the seed memories exist.

## Station 6

Promotion should make Atlas Fitness searchable via `knowledgebase_search` for other members of the datapack. Traces should show search → decision → `memory_add` (or conversation capture) for CASE-4872.
