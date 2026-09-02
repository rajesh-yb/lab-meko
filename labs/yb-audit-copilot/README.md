# YugabyteDB audit copilot — start here

This lab teaches **Meko**, not YugabyteDB itself.

You do **not** connect to a real cluster. You get fake audit alerts (JSON files). Your Cursor agent reads the company playbook from Meko, remembers what last week’s on-call learned, and writes a case note. A second chat (night shift) must **look that up** instead of guessing.

## The story (30 seconds)

Helios Retail runs orders and payments on **YugabyteDB** (the Postgres-compatible API is called **YSQL**).

Security tools already fired an alert. Your job is only:

1. Match the alert to a **type** (schema change, extra privilege, backup noise, data copy, login burst).
2. Pick **HOLD**, **REVOKE**, or **CLEAR** using the playbook.
3. Save *why* in Meko so the next person does not start from zero.

## Words we use

| Word | Plain meaning |
|---|---|
| Universe | One YugabyteDB production cluster. Ours is `prod-us-east-1`. |
| YSQL | SQL API (Postgres-like) on that cluster. |
| Role | Database login name, such as `app_migrate` or `yb_backup`. |
| DDL | Schema change: `CREATE`, `ALTER`, `DROP`. |
| CAB ticket | Change-advisory-board approval, like `CAB-1842`. Needed during a freeze. |
| Schema freeze | No schema changes in production unless there is a CAB ticket. Freeze ends 2026-09-15. |
| HOLD | Wait. Do not undo the SQL. A human reviews it. |
| REVOKE | Remove the extra privilege now and kill the session. |
| CLEAR | This is expected noise. Not an incident. |
| Datapack | A Meko workspace. This lab uses one named `yb-audit-lab`. |
| Knowledge base | Team playbook you **upload** in the Meko portal. |
| Memory | Facts from last week’s shift. Not in the uploaded files until someone promotes them. |

## Two paths

| Path | Time | What you do |
|---|---|---|
| **Core lab** | about 90 minutes | Stations 0–6 in [LAB.md](LAB.md). One HOLD handoff, one REVOKE, one CLEAR. |
| **Practice pack** | extra 30–45 minutes | All 10 cases in [CASES.md](CASES.md). Same playbook, more situations. |

Open **[LAB.md](LAB.md)** next. Keep **[CASES.md](CASES.md)** beside you. Open **[EXPECTED.md](EXPECTED.md)** only after you try.

## What is in this folder

```text
README.md      ← you are here (easy overview)
LAB.md         ← click-by-click steps and copy-paste prompts
CASES.md       ← every alert in plain language
EXPECTED.md    ← answer key (after you try)
knowledge/     ← upload these three files into Meko
cases/         ← ten JSON alerts
```
