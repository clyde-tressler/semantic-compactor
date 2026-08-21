---
name: semantic-compactor
description: >-
  Institutional-memory discipline that guarantees no problem→solution outcome is ever lost
  across sessions, /compact operations, or agent restarts. Use this skill whenever: (1) a bug,
  symptom, or failure is being investigated — especially one that looks familiar or may have
  occurred before; (2) a bug is resolved or a design decision is made and its knowledge must be
  durably recorded; (3) the user mentions regressions, "we fixed this before", lost context,
  post-mortems, /compact losing information, or knowledge that "should have been written down";
  (4) setting up or auditing a repo's knowledge-capture structure (DECISIONS.md, SYMPTOMS.md,
  regression guards). Also trigger proactively at the moment any debugging effort concludes,
  even if the user doesn't ask — Resolve → Record is atomic.
---

# Semantic Compactor: Lossless Institutional Memory

The purpose of this skill is to make the normal development loop —
**propose → implement → debug → test → refine → repeat** —
have the property that **no outcome of problem → solution is ever lost**, without rote-recording
every word. It has two halves: a **triage protocol** (how to investigate a symptom without
re-solving solved problems) and a **capture architecture** (how to record knowledge so it
survives context loss).

---

## Part 1 — Triage protocol (BEFORE theorizing about any bug)

When a symptom appears, in this order:

1. **Search the record for the SYMPTOM, not the cause — and search git first.**
   ```bash
   git log --all --grep="<symptom words>" -i
   grep -ri "<symptom words>" SYMPTOMS.md docs/ 2>/dev/null
   ```
   Git is the most durable, highest-signal record available. Ten seconds here can save hours.

2. **Never assert that knowledge was lost until you've proven it absent** from git history,
   the symptom index, and the decision ledger. Absence in one corpus is not absence.
   "The fix must have been lost" is a hypothesis to falsify, not a premise.

3. **A symptom is not a defect identity.** When a symptom recurs, assume a **new cause**
   until proven otherwise. Identical visual/behavioral signatures routinely have distinct
   root causes. If the record shows a prior fix, verify the fix is still present and effective
   before concluding it regressed.

4. **Run the cheapest discriminating experiment before theorizing within a layer.**
   Bisect *between* layers (content vs. renderer vs. driver; app vs. library vs. OS) before
   exploring hypotheses *inside* one layer. Ask: "what two-minute observation would eliminate
   an entire branch?"

5. **Weight direct human observation heavily.** When the person with eyes on the system
   contradicts your theory, update the theory, not the report.

## Part 2 — Capture architecture

### Compression principle: record only what cannot be re-derived

| Class | Examples | Durability |
|---|---|---|
| Ephemeral | tool output, failed searches, reasoning traces | zero — discard |
| Derivable | current code, test counts, deploy state | **never record** — regenerate from source of truth; recording it produces lies when it goes stale |
| Irreversible | why a choice was made, what a symptom means, what was ruled out | **must be lossless** |

The irreversible residue of even a very long session is typically a few pages. That is the
entire journaling burden.

### The durability ladder

Push every piece of knowledge as far UP this ladder as it can go:

1. **Executable guard** (test / assertion / script refusal) — fails loudly, cannot be ignored
2. **Comment at the danger point** — read exactly when relevant
3. **Indexed document** keyed by how a future investigator would *search* — findable on purpose
4. **Prose in a log** — findable by luck
5. **Conversation / context window** — lost

Anything left at levels 4–5 at the end of a debugging session is a regression waiting to happen.
Important knowledge should live at multiple levels simultaneously (guard + comment + index entry).

### The four stores (each with exactly one job)

| Store | Nature | Content |
|---|---|---|
| `DECISIONS.md` | append-only, dated | why a choice was made, **including rejected alternatives** |
| `SYMPTOMS.md` | append-only, keyed by *observable* | symptom → known causes → discriminator → fix + commit ref → **what was ruled out** |
| Executable guards | test suite / scripts | every fix that *can* be asserted, *is* |
| Dashboard (`next_steps.md` or similar) | mutable, small | current state, explicitly derived, regularly reconciled |

**Critical rule: never mix append-only history with mutable state in one file.**
Dated facts never become false; state rots. Ledger ≠ dashboard.

**Tracked-home rule:** a fact whose *only* home is a gitignored file (`CLAUDE.md`,
`.claude/*`, local scripts, device images) is not durable — it vanishes on a fresh clone.
Every irreversible fact needs a home in a **tracked** file; gitignored homes get a tracked
echo.

**Staleness is the dual of loss.** This architecture armors knowledge that would *die* with
the session; it must equally armor artifacts that *survive* the session and lie. Two rules:
(a) deletion is part of recording — when closing a session, remove or mark-done any claim
that `HEAD` now contradicts, in every artifact that will be re-surfaced (docs, checklists,
and especially auto-injected plan files); (b) **verify-before-act** — a surviving plan states
intent, not completion, so confirm any plan item's status against code/`git log` before
acting on it. "Derivable, don't record" is only safe when re-derivation before action is
enforced.

**Before any compaction or handoff**, run the full pre-compaction pass in
`references/compaction-checklist.md` — sweep, place, tracked-home check, question index,
stale-claim removal, auto-injected-artifact reconciliation, cold read, all-PASS acceptance
bar. **After the compaction**, the counterpart skill `rehydrate` restores state the safe
way: it re-derives the working state from the tracked files and live systems, verifying
every claim before anything acts on it, rather than trusting the injected summary.

`SYMPTOMS.md` is the store most likely to be missing from a repo — debugging knowledge is
otherwise homeless and scatters into commits, comments, and transcripts, recoverable only by
luck. Templates for both ledgers: `references/templates.md`.

### The three habits

1. **Resolve → Record, atomically.** The moment a bug is solved or a decision made, write the
   artifact *before* moving to the next task. Journal at state transitions, not continuously —
   that's how you get journaling without transcription. Treat an unrecorded fix as an
   unfinished fix.

2. **`Symptom:` trailer in every fix commit.** One line describing the *observable*, in the
   words an operator would use to describe it:
   ```
   Fix first-paint race in the renderer

   Symptom: blank region on the display after startup
   ```
   This makes git itself a self-maintaining symptom index — `git log --grep` on the symptom
   then always works. Near-zero cost.

3. **Productize-or-it-regresses.** When a debugging session ends, ask: "could a script have
   detected this symptom?" If yes, write the **verifier** before closing the session —
   especially for failure classes whose only prior detector was a human (rendered output,
   audio, hardware behavior). Design requirements — discriminator-based detection, tri-state
   exit codes (observation failure must never masquerade as a pass), failure output that
   points at the `SYMPTOMS.md` entry, invocation wired into state-transition checklists, and
   proof in both directions (healthy → pass, fix stripped → fail) — are in
   `references/verifier-pattern.md`, along with the closed-loop table showing which record
   failure mode each artifact eliminates.

### The guarantee mechanism: the cold-start test

Rules are aspirational; this is verifiable. Periodically (or after major debugging sessions),
run a **context-free audit**: a fresh agent with only the repo, asked the key questions —
"why does this odd construct exist?", "what causes symptom X and how do I discriminate the
causes?", "what has already been ruled out?" If it can't answer, the record is lossy — and you
found out cheaply instead of at hour three of a re-investigation. This is a falsifiable test
of losslessness. Full procedure and question-generation guidance: `agents/cold-start-auditor.md`.

---

## Deployment modes

- **As a standing skill**: apply Part 1 automatically whenever debugging begins; apply Part 2
  automatically whenever debugging ends or a decision is made.
- **As a subagent** (Claude Code): install `agents/regression-triage.md` as a project agent
  (`.claude/agents/`). Dispatch it when a symptom appears; it runs the triage protocol and
  reports whether the symptom is known, before any main-line theorizing happens.
- **As an audit**: run the cold-start test from `agents/cold-start-auditor.md` on demand or on
  a schedule.

## Bootstrap checklist (new or existing repo)

1. Create `DECISIONS.md` and `SYMPTOMS.md` from `references/templates.md`; seed `SYMPTOMS.md`
   with every currently-known symptom.
2. Audit existing docs: move any derivable state out of append-only files; move any history
   out of mutable dashboards.
3. For each known past fix, check whether an executable guard exists; write the missing ones —
   with special attention to output the tests can't see but a human can (rendered screens,
   audio, hardware behavior). If the only detector for a failure class is a human, build a
   verifier for it per `references/verifier-pattern.md`, and prove it in both directions.
4. Adopt the `Symptom:` commit trailer going forward.
5. Run one cold-start test to baseline the record's losslessness.
