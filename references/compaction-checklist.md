# Pre-Compaction Checklist

Run before any context compaction, session end, or agent handoff. Goal: **nothing of value
depends on the chat surviving** — and "written down" only counts if a stranger with a fresh
clone can find it.

This checklist encodes three failure modes discovered empirically (not theorized):
the **gitignore trap**, **stale distractors**, and **surviving-artifact staleness** — the dual
of knowledge loss, where an artifact outlives the session and actively misleads.

## The checklist

1. **Sweep the session.** Sort every candidate fact:
   - EPHEMERAL (mattered only in-conversation) → drop
   - DERIVABLE (recoverable from code/tests/`git log`) → drop; a copy would rot into a lie
   - IRREVERSIBLE (decisions, root causes, ruled-out branches, why-it's-built-this-way,
     **corrections the human made to the agent's assumptions**) → keep

2. **Place each IRREVERSIBLE fact** at the highest rung of the durability ladder it fits:
   executable guard → code comment at the danger point → `SYMPTOMS.md` (defects, keyed by
   observable, with discriminators and ruled-out list) → `DECISIONS.md` (decisions with
   rejected alternatives).

3. **Check the gitignore trap.** For every fact, confirm its home is a **tracked** file.
   Agent-config files (`CLAUDE.md`, `.claude/*`), device images, and local scripts are often
   gitignored — knowledge whose *only* home is gitignored vanishes on a fresh clone and isn't
   durable. Add a tracked echo (for gitignored scripts, the echo is a tracked entry explaining
   the mechanism, not necessarily the script itself).

4. **Update `COLDSTART_QUESTIONS.md`.** One line per non-obvious fact: the question a
   context-free investigator would ask → pointer to where the answer now lives. Writing the
   index is itself a gap detector — a question you can't point anywhere is a missing artifact.

5. **Remove stale claims.** Deletion is part of recording. A context-free reader cannot tell
   a stale note from a live one; a wrong claim misleads worse than a missing one. Sweep the
   docs you touched for statements that `HEAD` now contradicts.

6. **Sweep the auto-injected artifacts — not just the tracked docs.** Anything the harness
   re-surfaces into fresh context after compaction (plan files, loaded backlogs, memory rules)
   must be reconciled against `HEAD`: mark completed items DONE *in the artifact itself*.
   Resolve→Record applies to plans, not only to ledgers. A surviving plan that describes
   finished work as pending will cause a future agent to re-do it.

7. **Cold-read verification** (translation validation). Dispatch a fresh, zero-memory agent
   per `agents/cold-start-auditor.md`. **Tell it which paths are gitignored and have it treat
   them as absent** — a cold read on your working copy otherwise false-PASSes on files a
   fresh clone won't have. It answers every question from the repo alone: PASS / WEAK / FAIL.

8. **Acceptance bar:** proceed only when the cold read is **all-PASS** *and* every
   IRREVERSIBLE fact has a **tracked** home. WEAK counts as a gap. Fix, re-run, then commit
   and push the capture artifacts, then compact.

## The verify-before-act rule (for the agent on the OTHER side of compaction)

A plan or backlog states **intent, not completion**. "Derivable, don't record" is correct
*only if the reader re-derives before acting* — so make that binding: before acting on any
surviving plan item, confirm its status against the code and `git log`. One grep beats
re-doing finished work. This rule belongs in the project's tracked agent instructions so it
survives the compaction it guards against.
