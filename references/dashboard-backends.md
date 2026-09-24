# Dashboard Backends

The dashboard is one of the four stores: **mutable, small, explicitly derived, regularly
reconciled** — current state and queued work, never history. The core skill and the
`rehydrate` companion speak only the *contract* below; this file holds the concrete
procedures per backend. Open it when you actually operate a dashboard.

## The contract

Every backend implements four operations:

| Operation | Meaning |
|---|---|
| **Read** | Recover the current state and open queue, freshest first |
| **Queue** | Record a genuinely pending work item — one per workstream, not one per keystroke |
| **Update** | Record an in-flight item's state change when it changes hands (e.g. "staged, awaiting deploy phrase") |
| **Close** | Mark an item done the moment its work lands |

## Declaring a backend

One line in the repo's CLAUDE.md:

```
Dashboard: file next_steps.md   # the default when undeclared
Dashboard: jira PROJ            # a Jira project key, via the Atlassian connector
```

If no declaration exists, use the `file` backend, and ask once whether another backend
should be adopted; record the answer in CLAUDE.md either way.

## Invariants (all backends)

- **Mutable, derivable state only.** Irreversible facts (root causes, decisions,
  ruled-out branches) go in the tracked ledgers (`DECISIONS.md` / `SYMPTOMS.md`); a
  dashboard item may *point* at a ledger entry but must never be a fact's only home.
- **Resolve → Record applies here too.** A completed item left open is a stale note that
  will cause a future agent to re-do the work. Close items the moment work lands.
- **Reconcile before compaction.** The pre-compaction checklist's dashboard step brings
  the dashboard to truth against `HEAD` using this file's per-backend procedure.
- **Verify before acting.** A dashboard claim is verified against the system it is about
  (git, the live service, the ledger) — never by re-reading the dashboard itself.

## Backend: `file` (default)

A small tracked markdown file, `next_steps.md` unless the declaration names another path.
Portable: zero dependencies, works on any harness, survives on a fresh clone.

- **Read** — the most recent dated "CURRENT STATE" block at the top of the file.
- **Queue** — a bullet under a QUEUED section; date it.
- **Update** — rewrite the state block with a fresh date; never append history (that
  belongs in the ledgers or git).
- **Close** — delete the item or mark it DONE in place.
- **Pre-compaction reconciliation** — sweep the file for any claim `HEAD` now
  contradicts; correct or delete it. Deletion is part of recording.

## Backend: `jira` (Atlassian connector)

The repo's Jira project, worked through the Atlassian connector
(`mcp__claude_ai_Atlassian__*` tools). The declaration names the **project key**.
Requires the connector; if it is unavailable in a session, say so and fall back to
read-only reasoning — never mirror the board into a file (one dashboard, not two).

- **Read** — `searchJiraIssuesUsingJql` with
  `project = <KEY> AND statusCategory != Done ORDER BY updated DESC`, then the recent
  comments on the in-progress issues. The resume point is the most recently updated
  in-progress issue and its last state-change comment.
- **Queue** — `createJiraIssue`, one issue per workstream.
- **Update** — `transitionJiraIssue` for status moves, plus a short
  `addCommentToJiraIssue` comment when state changes hands.
- **Close** — `transitionJiraIssue` to Done the moment the work lands.
- **Pre-compaction reconciliation** — transition every issue whose work has landed; file
  issues for work genuinely queued but unrecorded; correct any status or comment `HEAD`
  contradicts; and confirm no issue comment is the *only* home of an irreversible fact —
  if one is, move the fact to the tracked ledger and leave a pointer in the comment.

Why Jira fits this store: it is mutable by design, shared with humans, survives a fresh
clone, and its statuses can't silently diverge inside a file nobody re-opened. The
external-store caveat is the flip side: the board lives outside git, so nothing
irreversible may live only there.

## Adding a backend

Any store that satisfies the contract and invariants qualifies — GitHub Issues and Linear
are natural candidates. Add a section here with the four operations and the
reconciliation procedure, and use the declaration form `Dashboard: <backend> <locator>`.
