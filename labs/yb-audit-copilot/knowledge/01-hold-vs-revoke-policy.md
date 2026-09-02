# Helios Retail — YugabyteDB audit policy

This is the official playbook. The agent must search for it in Meko before it recommends an action. Do not make up a stricter or looser rule.

Effective: 2026-06-01. Owner: Platform Security. Scope: YSQL audit events on production universes.

Plain meaning of the three actions:

- **HOLD** — Pause. Do not undo the SQL. A human reviews it (usually within 24 hours).
- **REVOKE** — Take the extra privilege away now and kill that session. Do not wait.
- **CLEAR** — Mark as noise. No incident. Still write a short note.

## Decision table

| Situation (in plain language) | Action | Notes |
|---|---|---|
| Someone changed the schema (`CREATE` / `ALTER` / `DROP`) during a freeze, and there is **no** CAB ticket | **HOLD** | Review in **24 hours**. The agent must **not** roll back the change. A human DBA approves or reverts. |
| Someone changed the schema during a freeze, but the session has a **valid CAB ticket** that matches the object | **CLEAR** | Write down the ticket id (example: CAB-1842). |
| Someone gave an **app or CI role** extra power: `yb_superuser`, `CREATEDB`, or `CREATEROLE` | **REVOKE** and kill the session | Send to the privilege desk. Do not HOLD. |
| First time we see `COPY` of a customer or payments table from a host that is **not** on the backup allow-list | **HOLD** and escalate | Two people must review. Do not CLEAR. |
| Known backup role, **known** backup host, `SELECT` only, and this has happened on **two or more** quiet nights | **CLEAR** (low priority) | Noise, unless we also see `COPY`, DDL, or a new host. |
| Known backup role, but the host is **new** (not on the allow-list) | **HOLD** | Ask for proof. Do not CLEAR just because the role is named `yb_backup`. |
| New region or new client network **and** `ALTER ROLE` or `GRANT` | **HOLD** and escalate | Two people must review. |
| 20 or more failed logins in 5 minutes to `yb_superuser` from one IP | **HOLD** and escalate | Auth desk. Do not revoke application roles for this. |

## Hard rules

1. During a freeze, a HOLD on DDL is never turned into a rollback by an agent. A human DBA must revert or approve.
2. Always quote the table row you used. If two rows fit, pick the **more severe** action and say so.
3. Save the case id, universe, database role, statement type, and action. Do **not** save passwords, secret connection strings, or SQL bind values that contain personal data.
4. If this policy and an old memory disagree, follow **this policy**, then tell a reviewer about the conflict.
