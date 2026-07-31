# semantic-compactor

**Lossless compaction of session knowledge for AI coding agents — so no problem→solution outcome is ever lost to
context compaction, session resets, or agent handoffs.**

Agentic development is development where the developer has amnesia. Context windows compact,
sessions end, and the hard-won knowledge from a debugging session — the root cause, the
discriminating experiments, everything that was ruled out — evaporates. Human teams get
institutional memory for free through people. Agent loops get none: **the repo has to be the
team memory.**

This skill packages a set of countermeasures, derived from real post-mortems of agent
misdiagnoses and validated against real compaction runs.

## What's in the box

```
semantic-compactor/
├── SKILL.md                            # the skill: triage protocol + capture architecture
├── references/
│   ├── templates.md                    # SYMPTOMS.md / DECISIONS.md templates + commit trailer convention
│   ├── verifier-pattern.md             # automated detectors for human-only failure classes
│   └── compaction-checklist.md         # pre-compaction pass with all-PASS acceptance bar
└── agents/
    ├── regression-triage.md            # subagent: symptom-first triage BEFORE theorizing
    └── cold-start-auditor.md           # subagent: verifies the record is actually lossless
```

## Core ideas

**Triage protocol** — when a symptom appears: search the record for the *symptom* (git first)
before theorizing about the cause; never assert knowledge was "lost" until proven absent;
treat a recurring symptom as a new cause until proven otherwise; run the cheapest
discriminating experiment before exploring within a layer.

**Capture architecture** — when a problem is resolved:

- *Record only what cannot be re-derived.* Sessions partition into ephemeral (drop),
  derivable (never record — copies rot into lies), and irreversible (the why, the
  discriminators, the ruled-out branches). The irreversible residue of a huge session is a
  few pages. That's the whole journaling burden.
- *The durability ladder.* Executable guard > comment at the danger point > indexed doc >
  prose log > conversation. Push everything as high as it goes.
- *Four stores, one job each.* `DECISIONS.md` (append-only why), `SYMPTOMS.md` (append-only,
  keyed by observable), executable guards, and a small mutable dashboard. Never mix ledger
  and dashboard.
- *Three habits.* Resolve→Record atomically; a `Symptom:` trailer on every fix commit (makes
  git a self-maintaining symptom index); productize-or-it-regresses (if a script could have
  detected the symptom, write it before closing the session).

**Verification** — borrowed from compiler theory's translation validation: a cold-start
audit spawns a context-free agent with only the repo and asks it the questions a future
maintainer would need answered. Every question it can't answer is knowledge trapped in a
conversation that no longer exists. The audit simulates a *fresh clone* (gitignored files
treated as absent) and flags stale claims that `HEAD` contradicts — because a surviving
artifact that lies is as dangerous as a lost one.

## Install

**Claude Code** — as a skill:

```bash
mkdir -p ~/.claude/skills
cp -r semantic-compactor ~/.claude/skills/
```

Per-project subagents:

```bash
mkdir -p .claude/agents
cp semantic-compactor/agents/*.md .claude/agents/
```

Other agent frameworks: `SKILL.md` is plain markdown with YAML frontmatter; adapt the
`description` triggering to your harness's convention.

## Bootstrap a repo

1. Create `DECISIONS.md` and `SYMPTOMS.md` from `references/templates.md`; seed `SYMPTOMS.md`
   with every currently-known symptom.
2. Adopt the `Symptom:` commit trailer.
3. For failure classes only a human can currently detect (rendered output, audio, hardware),
   build a verifier per `references/verifier-pattern.md` — and prove it in both directions.
4. Run one cold-start audit to baseline how lossy your record already is.

## When NOT to use this

Throwaway prototypes and short-lived projects. The architecture is an investment against
recurrence across context loss; if nothing lives long enough to recur, keep only the
`Symptom:` trailer (it's free) and skip the rest.

## License

MIT
