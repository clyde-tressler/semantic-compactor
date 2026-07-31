# The Verifier Pattern

*("Productize-or-it-regresses")*

A **verifier** is an executable guard for a failure class whose only prior detector was a
human observing the system — rendered screens, audio output, hardware behavior, LED states,
physical actuation. If the only detector for a failure class is a human, the failure class is
effectively untested no matter how large the test suite is: a test asserting "the bad CSS is
gone" passes happily while the screen is broken. Build the machine eye.

## Rule

**When a debugging session ends, ask: "could a script have detected this symptom?"
If yes, write the script before closing the session.** A fix without a detector is a fix
that will regress silently. This is the executable-guard rung of the durability ladder
applied to observable output.

## Design requirements

Every verifier must satisfy all four:

### 1. Detect via the discriminator, not via "badness"

Don't look for generic wrongness — encode the *signature that separates the artifact from
legitimate content*. Example: an uninitialized render region is near-100% uniform in color,
while legitimate dark content (photos, dense 2D barcodes) is a fine-grained mix; thresholding
on region uniformity flags the artifact with no false positives on content. The discriminator you recorded in `SYMPTOMS.md`
is usually exactly the thing to mechanize — this is the discriminator promoted from
documentation (ladder level 3) to executable guard (level 1).

### 2. Tri-state exit codes — observation failure must never masquerade as a pass

```
exit 0 — observed, healthy
exit 1 — observed, anomaly detected
exit 2 — could not observe (capture/instrumentation failure)
```

A verifier that returns success when it couldn't capture is worse than no verifier: it
converts "unknown" into "healthy". Callers must treat exit 2 as its own failure to
investigate, not as a pass.

### 3. Point at the record on failure

On exit 1, the verifier's output names the symptom in operator's words and references the
`SYMPTOMS.md` entry (and known-cause commits). The detector and the index stay linked, so
detection immediately routes to accumulated knowledge instead of restarting an investigation.

### 4. Run at state transitions

Wire the verifier into the checklists where the system's state changes: after flash/deploy,
after each test scenario in a rehearsal, in CI where feasible. A verifier that exists but
isn't invoked is documentation, not a guard (ladder level 3 pretending to be level 1).

## Prove it in both directions

Before trusting a verifier, demonstrate on real hardware/output:
- healthy state → exit 0
- fix deliberately stripped/reverted → exit 1 with correct localization

An unproven verifier is a hypothesis. The strip-the-fix test also doubles as confirmation
that the guard actually covers the recorded cause.

## The closed loop

Each artifact in this skill covers a distinct failure mode of the *record itself*:

| Artifact | Failure mode it eliminates |
|---|---|
| Symptom-first diagnosis rule | misdiagnosis ("same symptom → same cause", "the fix was lost") |
| `SYMPTOMS.md` | knowledge unfindable (filed under an answer, not a question) |
| `Symptom:` commit trailer | index rot (git self-maintains the index at zero upkeep) |
| Verifier | undetectable regression (human-only failure classes) |
| Cold-start test | silent lossiness (proves the record answers a context-free investigator) |

The loop is closed only when all five exist. Dropping any one reopens the corresponding
failure mode.
