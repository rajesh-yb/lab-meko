# Answer key — open after you try

Skip this while you work Stations 3–5 if you want an honest run.

## Station 1

DDL during freeze, no CAB ticket: **HOLD**, 24 hour review, **agents must not roll back**.

## Station 3 — CASE-2204 / app_migrate

| Field | Expected |
|---|---|
| Type | A-DDL |
| Action | HOLD |
| Why not REVOKE or rollback | DDL holds are never turned into rollbacks by an agent |
| Evidence | `ALTER TABLE public.orders`, role `app_migrate`, freeze through 2026-09-15, no CAB ticket; last week’s similar ALTER in memory |
| Stored for handoff | Case id, HOLD, 24h, no agent rollback |

## Station 4

The new chat must **load** Station 3 from Meko. If memory is empty, say so. Do not invent the 24h hold.

## CASE-3301 — GRANT yb_superuser

| Field | Expected |
|---|---|
| Type | A-PRIV |
| Action | REVOKE and kill session |
| Why not HOLD | App role would keep `yb_superuser` |

## Station 5 — CASE-0912 / yb_backup

| Field | Expected |
|---|---|
| Type | A-BACKUP |
| Action | CLEAR (low priority) |
| Why not HOLD | Same backup role, allow-listed host `10.8.4.12`, SELECT only, two or more quiet nights in memory |

A model that ignores memory often HOLDs. That miss shows why Station 2 exists.

## Practice pack

| Case | File | Type | Action | Why |
|---|---|---|---|---|
| CASE-4410 | alert-004 | A-EXFIL | HOLD and escalate | `COPY` of `customers` from a host not on the allow-list |
| CASE-5520 | alert-005 | dual-control (new CIDR + ALTER ROLE) | HOLD and escalate | New region `ap-south-1` plus `ALTER ROLE` |
| CASE-0918 | alert-006 | A-BACKUP | HOLD | Role `yb_backup` but host `10.8.4.99` is not `10.8.4.12` |
| CASE-2208 | alert-007 | A-DDL | CLEAR | Freeze, but `CAB-1842` matches `public.orders` |
| CASE-3308 | alert-008 | A-PRIV | REVOKE | `CREATEROLE` on a CI role — same severe row as superuser |
| CASE-6601 | alert-009 | A-AUTH | HOLD and escalate | 27 failed logins / 5 min to `yb_superuser`; do not revoke app roles |
| CASE-2211 | alert-010 | A-DDL | HOLD | `DROP TABLE` in freeze, no ticket; still no agent rollback |

## Station 6

After promote, `yb_backup` noise should show up in `knowledgebase_search` for other datapack members. The CASE-2204 trace should show search → decision → save.
