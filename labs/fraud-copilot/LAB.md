# Lab: Fraud investigation copilot

**Time:** 90 minutes  
**Level:** First hands-on with Meko  
**What you build:** Not a real-time scorer. A **shared investigation desk** — policy in the knowledge base, prior cases in memory, handoff with reasoning, and an audit trail.

Meko is the data layer. Your Cursor agent is the investigator. The fictional company is **Northstar Payments**.

## Why this use case fits Meko

| Meko primitive | What the lab does with it |
|---|---|
| Knowledge base | Hold vs reject policy, typologies, case-note template |
| Memory | Prior merchant learnings that should change the next shift |
| Conversation + handoff | Triage stores *why*; a new chat retrieves it before writing the note |
| Learnings / promote | One confirmed pattern becomes team-visible |
| Traces | You can show *why* CASE-4872 was held |

This is the same pattern as Meko’s own examples: [knowledge handoff](https://docs.mekodata.ai/examples/meko-enhances-teaming/), [collective learning](https://docs.mekodata.ai/examples/meko-enhances-teaming/), and [auditable traces](https://docs.mekodata.ai/examples/trace-and-resume/).

What this lab is **not**: scoring every payment through MCP. Alerts are already scored. You decide HOLD / REJECT / CLEAR and leave a trail.

## You need

1. A [Meko](https://docs.mekodata.ai/quick-start/) account (Free tier is enough).
2. Meko MCP connected in Cursor (`https://mcp.mekodata.ai/mcp`). This workspace already has a `meko` server in Cursor MCP settings — restart Cursor if tools do not appear.
3. About 90 minutes and two chat threads (triage, then investigator).

**Sanity check (2 minutes).** In this chat, send:

```text
List my Meko datapacks.
```

You should see a list from `datapack_list`. If the agent cannot call Meko tools, open Cursor Settings → MCP, confirm `meko` is enabled, then retry.

## Station 0 — Datapack (5 min)

In the Meko portal:

1. Create a datapack named `fraud-ops-lab` (or pin an existing one and use that name).
2. Pin it as the active datapack so saves land here.
3. Keep the datapack id handy. Agents pass `datapack_id` on tool calls.

Then in Cursor:

```text
Use datapack fraud-ops-lab for this lab. Confirm its datapack_id. Do not create extra datapacks.
```

**Pass:** The agent reports one datapack id and you can open that datapack in the portal.

---

## Station 1 — Shared playbook (10 min)

Knowledge-base **upload is portal-only** on Cloud today. MCP search works after upload.

1. Portal → Datapack `fraud-ops-lab` → Actions → **Add Knowledge**.
2. Upload these three files (PDF/TXT/MD/JSON; Free cap 5 MB each):
   - `labs/fraud-copilot/knowledge/01-hold-vs-reject-policy.md`
   - `labs/fraud-copilot/knowledge/02-typologies.md`
   - `labs/fraud-copilot/knowledge/03-case-note-template.md`
3. Wait until indexing finishes (usually under a minute for these files).

In Cursor:

```text
Search the Meko knowledge base on datapack fraud-ops-lab for the hold vs reject rule on duplicate invoices. Quote the policy row. Do not guess.
```

**Pass:** The answer quotes **HOLD**, **48 hours**, and **human must reject**. If it invents a rule, indexing is not ready — wait and search again.

---

## Station 2 — Seed private memory (10 min)

These facts are **not** in the uploaded docs. They are what last week’s shift learned. Paste:

```text
Remember this in Meko memory for fraud-ops-lab, verbatim:

Atlas Fitness (merchant M-0777) had three cardholder "not recognized" disputes in 2026. All three were won by the merchant on representment. Device and geo matched enrollment each time. Treat new Atlas Fitness disputes under $100 with a known device as noisy false positives unless ATO signals appear.

Harbor Coffee Co (merchant M-1044) had a true duplicate payout on 2026-04-15 for $2400 invoice INV-4872. AP resubmits after vendor dunning. Default is HOLD 48h, never agent-reject.

Do not store PAN or account numbers. Confirm with a memory search for Atlas Fitness and Harbor Coffee.
```

**Pass:** A follow-up search returns both merchants. If the write looked successful but search is empty, say so — do not claim it saved.

---

## Station 3 — Triage CASE-4872 (15 min)

Stay in **this** chat. Open `labs/fraud-copilot/cases/alert-001-duplicate-invoice.json` and paste:

```text
You are the triage agent for Northstar Payments.

Investigate the alert in labs/fraud-copilot/cases/alert-001-duplicate-invoice.json.

Required:
1. knowledgebase_search for hold vs reject and typology T-DUP.
2. memory_search for Harbor Coffee and INV-4872.
3. Recommend HOLD, REJECT, or CLEAR. Cite the policy row.
4. Store your reasoning in Meko so a later investigator chat can resume — include why you did not REJECT, the 48h window, and case id CASE-4872.
5. Do not store PAN or full account numbers.

Reply with action, typology, policy quote, and what you stored.
```

**Pass:** Action is **HOLD**, typology **T-DUP**, explicit “agents cannot reject duplicates.”

---

## Station 4 — Handoff in a **new** chat (15 min)

This is the Meko payoff. Open a **new Cursor chat** so the model does not still have Station 3 in context.

Paste:

```text
You are the night-shift investigator. You did not see the triage chat.

Before writing anything, search Meko memory and the fraud-ops-lab knowledge base for CASE-4872 / Harbor Coffee / INV-4872.

Then write a case note using the template in the knowledge base.

If Meko has no triage reasoning, say so — do not invent the 48h hold.

Do not re-score the payment. Resume from stored context.
```

**Pass:** The note mentions CASE-4872, HOLD, 48h, and that reject needs a human — retrieved from Meko, not from this thread.

Optional stretch: also run `alert-002-account-takeover.json` in the original chat. Policy says **REJECT** (ATO bundle). Confirms the agent is following the table, not “always HOLD.”

---

## Station 5 — Collective learning (15 min)

In a **new** or the investigator chat, paste `labs/fraud-copilot/cases/alert-003-noisy-chargeback.json` and:

```text
Investigate alert-003 (CASE-0891, Atlas Fitness, $89, not_recognized).

Search Meko memory and the knowledge base first.

Recommend HOLD, REJECT, or CLEAR using the hold-vs-reject table and any Atlas Fitness memories.

If you CLEAR, say which prior cases you relied on. Store a one-line learning for the next shift.
```

**Pass:** Action is **CLEAR** (or low-priority CLEAR), citing two-or-more won representments and known device. A first-time agent with an empty memory would HOLD — that contrast is the lab.

---

## Station 6 — Promote and traces (10 min)

In the Meko portal, datapack `fraud-ops-lab`:

1. **Learnings** (or Memories) — promote the Atlas Fitness noisy-FP fact to shared knowledge if you are owner/maintainer. Promotion is one-way and team-visible.
2. **Conversations** — open the CASE-4872 thread → **Open in Observe** / decision trace.

Confirm you can see tool calls (`knowledgebase_search`, `memory_search`, `memory_add`) under that conversation.

In Cursor you can also ask:

```text
Search the knowledge base for Atlas Fitness noisy false positives. If promotion succeeded, it should appear as team knowledge, not only personal memory.
```

**Pass:** You can point at a trace for CASE-4872 and explain HOLD from policy + prior match, not from model vibe.

---

## Done when

You can tell a colleague, with the portal open:

1. Policy lives in the datapack knowledge base, not in a prompt.
2. Night shift recovered CASE-4872 reasoning from Meko after a new chat.
3. Atlas Fitness did not get the same treatment as a first-time dispute.
4. A decision trace exists for the hold.

## Files

```text
labs/fraud-copilot/
  LAB.md                 ← you are here
  knowledge/             ← upload in Station 1
  cases/                 ← paste into Stations 3–5
  EXPECTED.md            ← answer key (open after you try)
```

## If something fails

| Symptom | Likely cause |
|---|---|
| Agent never calls Meko tools | MCP server off; restart Cursor; mention “Meko” in the prompt |
| Knowledge search empty | Upload not finished, or wrong datapack |
| Memory write OK, search empty | Indexing lag, or write went to another datapack — search again in 30s |
| Station 4 invents the hold | New chat did not search; make it call `memory_search` before answering |
| OAuth / 403 on CLI clients | Use the API key in Cursor MCP settings, not OAuth |

More: [Meko troubleshooting](https://docs.mekodata.ai/).
