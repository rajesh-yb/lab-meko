# Case note template

Write a short note so the next DBA can continue without opening the raw alert again.

```
Case: <CASE-ID>
Typology: <A-DDL | A-PRIV | A-BACKUP | A-EXFIL | A-AUTH | other>
Action: <HOLD | REVOKE | CLEAR | ESCALATE>
Policy row: <quote the hold-vs-revoke row>
Evidence:
- <what you got from the alert>
- <what you got from Meko memory>
- <what you got from the knowledge base>
Why not the alternative:
- <the action you did not take, and why>
Next step:
- <who owns it, and by when>
Redaction:
- No passwords, secret connection strings, or personal data in bind values.
```
