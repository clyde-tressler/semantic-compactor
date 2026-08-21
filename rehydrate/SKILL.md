---
name: rehydrate
description: Post-compaction state restore — re-derive the true working state from tracked files and live systems, verify every claim before acting on it, and resume at the verified resume point. The counterpart to semantic-compactor.
---

# Rehydrate: verified restore after compaction

The semantic-compactor's pre-compaction pass pushes the session's state into tracked files
(the dashboard, the ledgers, git). This skill is the other half: after a compaction (or at
the start of any session resuming prior work), **re-derive the state from those sources of
truth instead of trusting any summary** — including the harness-injected compaction summary,
which is a convenience copy, not a record.

The governing rule is the compactor's own: **a surviving note states intent, not fact.**
Every claim gets verified against its source of truth before anything acts on it.

## Procedure

### 1. Locate the state block

Read the project dashboard (`next_steps.md` or the project's equivalent mutable state file —
CLAUDE.md usually names it). The resume point is the most recent dated "CURRENT STATE"
block at the top. If no dashboard exists or the top block predates the last few commits,
say so — that itself is a finding (the compactor's pass was skipped or incomplete).

### 2. Extract and classify the claims

Walk the state block and sort every factual claim into one of four classes, each with its
own verifier:

| Claim class | Example | Source of truth |
|---|---|---|
| **Git claims** | "committed as `96bd364`", "wave-1 fixes are in HEAD" | `git log` / `git show` — confirm the commit exists, is on the expected branch, and touches what the note says it touches |
| **External-state claims** | "change set X is staged", "the fix is deployed", "the service is running" | The live system: cloud CLI, an HTTP probe, `systemctl` — whatever actually holds that state. If credentials/network make it uncheckable right now, mark it UNVERIFIED, never assume it |
| **Position claims** | "docket at question 2, awaiting ruling", "awaiting the deploy phrase" | The ledger (`DECISIONS.md` etc.): confirm the prior rulings the position implies are actually recorded, and the next one is not |
| **Queue claims** | "workstream Y is queued, not started" | Spot-check that the work genuinely isn't done — `git log --grep`, a glance at the named files. A "queued" item that already landed is a stale note about to cause duplicate work |

### 3. Reconcile and report

Produce a short reconciliation, one line per load-bearing claim:
**VERIFIED** (matches source of truth) / **CONTRADICTED** (source of truth says otherwise —
quote what it says) / **UNVERIFIED** (couldn't check, and why).

- A CONTRADICTED claim is surfaced to the user **before** anything acts on it, and the
  dashboard is corrected in the same breath (staleness is the dual of loss).
- An UNVERIFIED claim may be carried forward only if explicitly labeled as unverified
  wherever it is next used.
- Don't drown the user: verify everything, but report only contradictions, unverifiables,
  and a one-line "everything else checks out." A clean rehydrate should read as two or
  three sentences, not an audit report.

### 4. Resume

State the verified resume point in one plain sentence ("we are mid-docket at question N;
change set X is confirmed still staged") and continue the work from there. If the resume
point involves a question awaiting the user's answer, re-pose that question — compaction
may have eaten the last time it was asked.

## Anti-patterns

- **Trusting the injected summary over the tracked files.** The summary is lossy by
  construction; where it disagrees with the dashboard, neither wins — the underlying
  source of truth (git, the live system, the ledger) does.
- **Verifying by re-reading the same note.** A dashboard claim is not verified by the
  dashboard; it's verified by the system the claim is about.
- **Silent correction.** If the record was wrong, the user learns it was wrong and why —
  a quietly patched dashboard hides that the compaction pass has a failure mode.
- **Rehydrating into action.** Rehydrate ends at the verified resume point. It does not
  start executing queued workstreams — those still follow the normal approval rhythm.
