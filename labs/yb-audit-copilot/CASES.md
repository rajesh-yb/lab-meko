# All cases, in easy language

There are **10 alerts**. They are fake. None of them contain passwords.

Use the same method every time:

1. Search the **knowledge base** for the playbook and the type code.
2. Search **memory** for this universe, this role, this case id.
3. Pick HOLD, REVOKE, or CLEAR. Quote the table row.
4. Save the reason in Meko. Do not save secrets.

Core lab uses **001, 002, 003**. Practice pack is **004–010**.

---

## Quick map

| File | Case | What happened | Type | Action |
|---|---|---|---|---|
| [alert-001-unapproved-ddl.json](cases/alert-001-unapproved-ddl.json) | CASE-2204 | Flyway added a column during freeze. No ticket. | A-DDL | **HOLD** |
| [alert-002-superuser-grant.json](cases/alert-002-superuser-grant.json) | CASE-3301 | CI gave `yb_superuser` to the app role. | A-PRIV | **REVOKE** |
| [alert-003-backup-select.json](cases/alert-003-backup-select.json) | CASE-0912 | Nightly backup `SELECT` from the known host. | A-BACKUP | **CLEAR** |
| [alert-004-copy-export.json](cases/alert-004-copy-export.json) | CASE-4410 | Laptop copied the customers table. | A-EXFIL | **HOLD** + escalate |
| [alert-005-new-cidr-alter-role.json](cases/alert-005-new-cidr-alter-role.json) | CASE-5520 | New region + `ALTER ROLE` on the app login. | A-PRIV / dual-control | **HOLD** + escalate |
| [alert-006-backup-new-host.json](cases/alert-006-backup-new-host.json) | CASE-0918 | Role `yb_backup`, but a **new** IP. | A-BACKUP | **HOLD** |
| [alert-007-cab-approved-ddl.json](cases/alert-007-cab-approved-ddl.json) | CASE-2208 | Index during freeze, ticket **CAB-1842**. | A-DDL | **CLEAR** |
| [alert-008-createrole-grant.json](cases/alert-008-createrole-grant.json) | CASE-3308 | CI role got `CREATEROLE`. | A-PRIV | **REVOKE** |
| [alert-009-login-burst.json](cases/alert-009-login-burst.json) | CASE-6601 | 27 failed logins to `yb_superuser`. | A-AUTH | **HOLD** + escalate |
| [alert-010-drop-table-freeze.json](cases/alert-010-drop-table-freeze.json) | CASE-2211 | `DROP TABLE` during freeze. No ticket. | A-DDL | **HOLD** (no agent rollback) |

---

## Core cases (Stations 3–5)

### CASE-2204 — Unapproved column (HOLD)

**File:** `cases/alert-001-unapproved-ddl.json`

Flyway ran as `app_migrate` and added `ship_note` on `public.orders`. Production is frozen until 2026-09-15. There is no CAB ticket.

This is the handoff case. Day shift stores *why it is HOLD*. Night shift opens a **new chat** and must read that from Meko.

**Why not REVOKE?** You are not taking a privilege away. You are not allowed to undo DDL yourself.

**Copy-paste prompt**

```text
You are the triage agent for Helios Retail (YugabyteDB YSQL audit).

Investigate labs/yb-audit-copilot/cases/alert-001-unapproved-ddl.json.

Search the yb-audit-lab knowledge base and memory first.
Recommend HOLD, REVOKE, or CLEAR. Cite the policy row.
Store reasoning for CASE-2204 (24h window, no agent rollback).
Do not store passwords or PII bind values.
```

### CASE-3301 — Superuser grant (REVOKE)

**File:** `cases/alert-002-superuser-grant.json`

Someone on the CI host ran `GRANT yb_superuser TO app_orders`.

If you only HOLD, the app role **keeps** superuser. Policy says take it away now.

**Copy-paste prompt**

```text
Investigate labs/yb-audit-copilot/cases/alert-002-superuser-grant.json.
Search knowledge and memory first. Recommend HOLD, REVOKE, or CLEAR. Cite the row.
```

### CASE-0912 — Known backup (CLEAR)

**File:** `cases/alert-003-backup-select.json`

Role `yb_backup`, host `10.8.4.12`, `SELECT` on `payments`, overnight. Memory says this happened three quiet nights already.

If memory is empty, a careful agent would **HOLD**. That is the point of seeding memory in Station 2.

**Copy-paste prompt**

```text
Investigate labs/yb-audit-copilot/cases/alert-003-backup-select.json
(CASE-0912, yb_backup, host 10.8.4.12).

Search Meko memory and the knowledge base first.
Recommend HOLD, REVOKE, or CLEAR.
If you CLEAR, say which prior nights you used. Store a one-line learning for the next shift.
```

---

## Practice pack (004–010)

Use the same prompt shape. Change only the file path. Search Meko first. Do not invent rules.

```text
You are the triage agent for Helios Retail (YugabyteDB YSQL audit).

Investigate labs/yb-audit-copilot/cases/<FILE>.

Search the yb-audit-lab knowledge base and memory first.
Recommend HOLD, REVOKE, or CLEAR. Cite the policy row.
Store the case id and why. Do not store passwords or PII bind values.
```

### CASE-4410 — Copy of customers (HOLD + escalate)

**File:** `cases/alert-004-copy-export.json`

An analyst laptop (`198.51.100.23`) ran `COPY` of `public.customers`. That host is not the backup box.

Looks like data leaving the database. Two people should look. Do not CLEAR.

### CASE-5520 — New region + ALTER ROLE (HOLD + escalate)

**File:** `cases/alert-005-new-cidr-alter-role.json`

The cluster is in `us-east-1`. The session is from `ap-south-1`. The SQL is `ALTER ROLE app_orders WITH LOGIN`.

Two signals together: new network **and** a role change. Policy wants dual control, not a silent CLEAR.

### CASE-0918 — Backup role, wrong host (HOLD)

**File:** `cases/alert-006-backup-new-host.json`

The role is still `yb_backup`, but the IP is `10.8.4.99`, not `10.8.4.12`.

This is the trap. Do **not** CLEAR because the name looks like the backup job. Policy: known role + **new** host → HOLD.

### CASE-2208 — DDL with a ticket (CLEAR)

**File:** `cases/alert-007-cab-approved-ddl.json`

Same freeze, same migrate role, but `cab_ticket` is `CAB-1842` and it matches `public.orders`.

This is the opposite of CASE-2204. Freeze + ticket that matches the object → CLEAR. Write the ticket id in the note.

### CASE-3308 — CREATEROLE (REVOKE)

**File:** `cases/alert-008-createrole-grant.json`

Not `yb_superuser`. Still extra power (`CREATEROLE` on `ci_deploy`). Policy lists this next to superuser. REVOKE and kill the session.

### CASE-6601 — Failed login burst (HOLD + escalate)

**File:** `cases/alert-009-login-burst.json`

27 failed logins in five minutes to `yb_superuser` from `203.0.113.40`. No successful login.

Send to the auth desk. Do **not** revoke `app_orders` or `ci_deploy` for this alert.

### CASE-2211 — DROP TABLE during freeze (HOLD)

**File:** `cases/alert-010-drop-table-freeze.json`

`DROP TABLE public.orders_old` with no ticket. It looks worse than adding a column. The action is still **HOLD**. The agent still must not roll back. A human DBA decides.

---

## Teaching contrast (use this in a demo)

| Pair | Same-looking thing | Different action | Lesson |
|---|---|---|---|
| 2204 vs 2208 | DDL in a freeze | HOLD vs CLEAR | The CAB ticket changes the row. |
| 0912 vs 0918 | Role `yb_backup` | CLEAR vs HOLD | The **host** matters, not only the role name. |
| 3301 vs 3308 | GRANT of extra power | Both REVOKE | Superuser and CREATEROLE use the same severe row. |
| 2204 vs 2211 | DDL, no ticket | Both HOLD | DROP is not a special “agent undo” path. |
| 0912 vs 4410 | Read vs copy of data | CLEAR vs HOLD | `SELECT` from the backup host is not the same as `COPY` from a laptop. |
