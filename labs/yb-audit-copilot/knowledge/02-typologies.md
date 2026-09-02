# Helios Retail — YugabyteDB audit types (short list)

Pick one type before you write the case note. Use these codes in the note.

## A-DDL — Schema change without a ticket

Someone ran `CREATE`, `ALTER`, or `DROP` while production is frozen, or a migrate role ran DDL with no CAB ticket.

This is often a deploy that skipped the change board, not a hack.

Default: **HOLD** for 24 hours. If a valid CAB ticket matches the object: **CLEAR**.

## A-PRIV — Extra database power

Someone ran `GRANT yb_superuser` (or `CREATEDB` / `CREATEROLE`) to an app or CI role.

Default: **REVOKE** now and kill the session.

## A-BACKUP — Nightly backup read

`SELECT` only, from role `yb_backup`, from the known backup host, overnight.

If we have seen this on two or more quiet nights: **CLEAR**.

If the host is new, or we see `COPY` / DDL: **HOLD**.

## A-EXFIL — Bulk copy of data

`COPY` or `\copy` of customer or payment tables from a host that is not the backup host.

Default: **HOLD** and escalate. Do not CLEAR.

## A-AUTH — Login attack noise

Many failed logins to a powerful role from one IP in a short time.

Default: **HOLD** and escalate to the auth desk.
