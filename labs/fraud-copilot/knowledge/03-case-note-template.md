# Case note template

Use this structure when writing the investigation note. Keep it short enough that a night-shift agent can resume without re-reading the raw alert.

```
Case: <CASE-ID>
Typology: <T-DUP | T-ATO | T-FF | other>
Action: <HOLD | REJECT | CLEAR | ESCALATE>
Policy row: <quote the hold-vs-reject row>
Evidence:
- <what you retrieved from the alert>
- <what you retrieved from Meko memory>
- <what you retrieved from the knowledge base>
Why not the alternative:
- <rejected action and why>
Next step:
- <who owns it, by when>
Redaction:
- No PAN, CVV, full account numbers, or government IDs stored.
```
