# Ledger Templates

Both files are **append-only**. Never edit or delete past entries; append amendments instead.
Never put current state ("not yet deployed", "TODO") in these files — that belongs in the
mutable dashboard.

---

## SYMPTOMS.md template

Keyed on **what an observer sees**, not on the fix. A future investigator arrives with a
symptom in hand and must be able to find the entry by searching those words.

```markdown
# Symptom Index

Append-only. Keyed by observable. Search this file (and `git log --all --grep`) BEFORE
theorizing about any bug.

---

## Symptom: <observable, in operator's words — e.g. "blank region on the display", "audio dropout after resume">

**Known causes** (a recurring symptom may gain new causes over time — list all):

### Cause 1: <root cause> — fixed in <commit sha>
- **Mechanism**: <one or two sentences>
- **Discriminator**: <the cheap experiment that distinguishes this cause from the others,
  e.g. "does it survive a page change? If yes, not content-layer">
- **Fix**: <what was changed> (<commit sha>)
- **Guard**: <test name / script, or "none — see gap below">

### Cause 2: ...

**Ruled out** (pure gold for the next recurrence — most of an investigation's cost is here):
- <hypothesis> — eliminated by <observation/experiment>, <date>
- ...
```

---

## DECISIONS.md template

```markdown
# Decision Ledger

Append-only, dated. Records WHY. Never edited — amend by appending a new dated entry that
references the old one.

---

## <YYYY-MM-DD> — <decision title>

**Decision**: <what was decided>
**Context**: <the forces at play>
**Rejected alternatives**:
- <alternative> — rejected because <reason>
**Consequences**: <what this commits us to>
**Amends**: <link/date of earlier decision, if any>
```

---

## Commit trailer convention

Every commit that fixes a defect carries a trailer line:

```
Symptom: <observable, in the words someone would search for>
```

Multiple symptoms → multiple trailer lines. This makes
`git log --all --grep="<symptom>" -i` a reliable, zero-maintenance symptom index.
