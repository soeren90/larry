# Scheduled automations

The background half of the system: deterministic cron jobs that run whether or
not the operator is at the desk. Most are **token-free** — plain scripts, not
agent calls — so they're cheap, fast, and reliable. They write to the memory
vault and notify the operator; none of them take customer-facing action.

## Ordering matters

The jobs run in a deliberate sequence early each day:

```
1. History snapshot   ← make today's data durable first
2. External pre-check ← flag restricted-zone records
3. Resource routing   ← propose assignments
4. Morning briefing   ← reconcile everything + fold in the above
```

The briefing runs last so it can fold in what the earlier jobs found. The
history snapshot runs first so everything downstream reads a durable copy of
the day's data.

## The jobs

### Morning briefing (daily)
The anchor. Reconciles the day's data export with CRM, chat, and mail, and
posts a complete operational briefing to a team channel. Folds in the findings
of the pre-check and routing jobs.

### External-constraint pre-check (daily, before the briefing)
Filters open records and checks each location against a geospatial/regulatory
API, flagging anything inside a restricted zone. It's an **early warning**, not
an approval — the responsible person still verifies. Deterministic; results
cached.

### History snapshot (daily, first)
Imports the day's data export into a local SQLite database as a dated snapshot.
This is what makes the system answer time questions: *when* did a record change
status, what was the state six weeks ago. It also lets records survive a
rolling-window export and **rejects** a suspiciously filtered export (well below
the usual median) instead of importing bad data.

### Resource routing (daily)
The same matching engine as the on-demand routing agent, run proactively so its
suggestions are ready in the morning briefing.

### Weekly KPI report (weekly)
Builds a week-over-week report and an HTML visualization, drawing on the history
DB for figures that need a real "since last week" baseline.

### Learning loop (weekly)
Compares what the system proposed against what the human actually did, and
writes improvement proposals. Detailed in
[the-learning-loop.md](the-learning-loop.md).

### Vault maintenance (monthly)
Consolidates the memory vault. Cosmetic fixes directly; content changes as
proposals. Never renames, moves, or deletes files wired into scripts.

## Why token-free where possible

A daily job that calls a model on every run is expensive and non-deterministic.
Where the work is mechanical — import a CSV, check coordinates against a WMS,
compute deltas — a plain script is cheaper, faster, and gives the same answer
every time. The model is reserved for the work that genuinely needs judgment.
