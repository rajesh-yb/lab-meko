# Lab: YugabyteDB audit copilot

**Time:** about 90 minutes for the core path. Extra cases are optional.  
**Level:** first time with Meko, or a second lab after fraud ops.  
**What you build:** a **shared audit desk**. Not a live database.

Meko stores the playbook, last week’s lessons, and the “why” of each decision. Cursor is the investigator. The company in the story is **Helios Retail**.

Start with the [easy overview](README.md) if words like “datapack” or “CAB” are new. Case-by-case help is in [CASES.md](CASES.md).

## What this lab shows

| Meko piece | What you will see |
|---|---|
| Knowledge base | The hold / revoke / clear table |
| Memory | “This backup host is noisy” and “we are in a schema freeze” |
| New chat | Night-shift DBA recovers CASE-2204 without seeing day-shift chat |
| Promote | Backup lesson can become team knowledge |
| Trace | You can show *why* CASE-2204 was held |

You never connect to a real Yugabyte universe. Alerts are already scored. You only choose HOLD, REVOKE, or CLEAR and leave a trail.

This is the same Meko pattern as the [fraud lab](../fraud-copilot/LAB.md). Use that lab for a payments audience. Use this one for a database audience.

## You need

1. A [Meko](https://docs.mekodata.ai/quick-start/) account (Free is enough).
2. Meko MCP on in Cursor (`https://mcp.mekodata.ai/mcp`). Restart Cursor if the tools do not show.
3. Two Cursor chats for the core path (day-shift triage, then night-shift DBA).

**Quick check.** In this chat, send:

```text
List my Meko datapacks.
```

You should get a list from `datapack_list`. If nothing happens, Cursor Settings → MCP → turn `meko` on, then try again.

## Station 0 — Make a workspace (5 min)

In the Meko website:

1. Create a datapack named `yb-audit-lab` (or reuse that name if it exists).
2. Pin it so new saves go here.
3. Copy the datapack id. The agent needs it on searches.

Then in Cursor:

```text
Use datapack yb-audit-lab for this lab. Confirm its datapack_id. Do not create extra datapacks.
```

**Pass:** You get one id, and you can open that datapack in the website.

---

## Station 1 — Upload the playbook (10 min)

Uploading files is **only in the Meko website** today. Search from Cursor works after the upload finishes.

1. Website → datapack `yb-audit-lab` → Actions → **Add Knowledge**.
2. Upload:
   - `labs/yb-audit-copilot/knowledge/01-hold-vs-revoke-policy.md`
   - `labs/yb-audit-copilot/knowledge/02-typologies.md`
   - `labs/yb-audit-copilot/knowledge/03-case-note-template.md`
3. Wait until indexing is done (often under a minute).

Then in Cursor:

```text
Search the Meko knowledge base on datapack yb-audit-lab for the hold vs revoke rule on DDL during a schema freeze. Quote the policy row. Do not guess.
```

**Pass:** The answer says **HOLD**, **24 hours**, and a **human DBA** must approve or revert. If the agent invents a rule, wait and search again.

---

## Station 2 — Save last week’s lessons (10 min)

These sentences are **not** in the uploaded files. Paste this:

```text
Remember this in Meko memory for yb-audit-lab, verbatim:

Role yb_backup on universe prod-us-east-1 runs SELECT-only from host 10.8.4.12 against the replica between 02:00 and 03:30 UTC. Three consecutive nights in 2026 had the same pattern with no COPY and no DDL. Treat new yb_backup SELECT alerts from that host as noisy false positives unless COPY, DDL, or a new client address appears. Host 10.8.4.12 is the only backup allow-list address.

Universe prod-us-east-1 is under schema freeze through 2026-09-15. Role app_migrate ran an unapproved ALTER TABLE on public.orders last week with no CAB ticket. Default is HOLD 24h, never agent-rollback. A valid CAB ticket on the session that matches the object may CLEAR.

Do not store passwords or connection strings. Confirm with a memory search for yb_backup, app_migrate, and 10.8.4.12.
```

**Pass:** Search finds the backup host and the migrate role. If the write looked fine but search is empty, say so. Wait 30 seconds and search again.

---

## Station 3 — Day-shift triage, CASE-2204 (15 min)

Stay in **this** chat. Details: [CASES.md](CASES.md) → CASE-2204.

```text
You are the triage agent for Helios Retail (YugabyteDB YSQL audit).

Investigate the alert in labs/yb-audit-copilot/cases/alert-001-unapproved-ddl.json.

Required:
1. knowledgebase_search for hold vs revoke and typology A-DDL.
2. memory_search for app_migrate, schema freeze, and CASE-2204.
3. Recommend HOLD, REVOKE, or CLEAR. Cite the policy row.
4. Store your reasoning in Meko so a later DBA chat can resume — include why you did not roll back, the 24h window, and case id CASE-2204.
5. Do not store passwords, secrets, or PII bind values.

Reply with action, typology, policy quote, and what you stored.
```

**Pass:** Action **HOLD**, type **A-DDL**, and a clear line that **agents cannot roll back DDL**.

In the **same** chat, also run the superuser case so you see REVOKE:

```text
Investigate labs/yb-audit-copilot/cases/alert-002-superuser-grant.json the same way.
```

**Pass:** **REVOKE** (not HOLD). Type **A-PRIV**.

---

## Station 4 — Night shift, new chat (15 min)

This is the point of Meko. Open a **new** Cursor chat so CASE-2204 is not still on screen.

```text
You are the night-shift DBA. You did not see the triage chat.

Before writing anything, search Meko memory and the yb-audit-lab knowledge base for CASE-2204 / app_migrate / public.orders.

Then write a case note using the template in the knowledge base.

If Meko has no triage reasoning, say so — do not invent the 24h hold.

Do not re-score the audit event. Resume from stored context.
```

**Pass:** The note has CASE-2204, HOLD, 24h, and “a human DBA must revert” — found in Meko, not remembered from the old chat.

---

## Station 5 — Backup noise, CASE-0912 (15 min)

Same chat as night shift, or a new one. Details: [CASES.md](CASES.md) → CASE-0912.

```text
Investigate alert-003 (CASE-0912, yb_backup, SELECT on payments, host 10.8.4.12).
File: labs/yb-audit-copilot/cases/alert-003-backup-select.json

Search Meko memory and the knowledge base first.

Recommend HOLD, REVOKE, or CLEAR using the hold-vs-revoke table and any yb_backup memories.

If you CLEAR, say which prior nights you relied on. Store a one-line learning for the next shift.
```

**Pass:** **CLEAR** (or low-priority CLEAR), citing two or more quiet nights and host `10.8.4.12`. Without memory, many models HOLD. That contrast is the lesson.

---

## Station 6 — Share a lesson and open the trace (10 min)

In the Meko website, datapack `yb-audit-lab`:

1. **Learnings** (or Memories) — if you are owner, promote the `yb_backup` noisy-host fact. Promotion is one-way. The whole datapack can see it.
2. **Conversations** — open the CASE-2204 thread → **Open in Observe**.

You should see tool calls: `knowledgebase_search`, `memory_search`, `memory_add`.

In Cursor:

```text
Search the knowledge base for yb_backup noisy false positives. If promotion succeeded, it should appear as team knowledge, not only personal memory.
```

**Pass:** You can explain CASE-2204 HOLD from policy + memory, not from a guess.

---

## Optional: practice pack (004–010)

About 30–45 more minutes. Stay in the triage chat. For each file, use the prompt in [CASES.md](CASES.md).

Do at least these two so the traps are obvious:

1. `alert-006-backup-new-host.json` — same backup **role**, wrong **host** → **HOLD**
2. `alert-007-cab-approved-ddl.json` — freeze but ticket **CAB-1842** → **CLEAR**

Then finish 004, 005, 008, 009, 010 if you have time.

**Pass:** The agent quotes a policy row each time and does not CLEAR 006 or HOLD 007 without a reason.

---

## Done when you can say

1. The playbook lives in the datapack, not in the prompt.
2. Night shift got CASE-2204 from Meko after a new chat.
3. `yb_backup` on `10.8.4.12` is not treated like a first-time bulk read, and `10.8.4.99` is not treated like `10.8.4.12`.
4. There is a trace for the HOLD.

## Files

```text
labs/yb-audit-copilot/
  README.md              ← easy overview
  LAB.md                 ← you are here
  CASES.md               ← all 10 alerts in plain language
  EXPECTED.md            ← answer key
  knowledge/             ← Station 1 upload
  cases/                 ← JSON alerts
```

## If something fails

| What you see | Likely cause |
|---|---|
| Agent never calls Meko | MCP off; restart Cursor; say “Meko” in the prompt |
| Knowledge search empty | Upload not finished, or wrong datapack |
| Memory write OK, search empty | Wait 30s, or the write went to another datapack |
| Station 4 invents the 24h hold | New chat did not search memory first |
| Login / 403 on CLI | Use the API key in Cursor MCP settings |

More: [Meko troubleshooting](https://docs.mekodata.ai/).
